# 02 — Память между перезапусками (git как persistence)

> Бот живёт в Docker контейнере на Railway. Контейнер пересоздаётся при каждом деплое. Без persistence — каждый рестарт стирает память. С persistence — память переживает деплои, рестарты, и даже миграции на другой хост.
>
> Решение: git ветка как база данных.

---

## Почему git, а не Postgres / Redis

| Критерий | Postgres / Redis | git |
|---|---|---|
| Стоимость | +$5-15/мес сервис | $0 (тот же репо) |
| Версионирование | manual snapshots | каждое изменение = коммит |
| История | требует table design | `git log` показывает всё |
| Откат | восстановление из бэкапа | `git checkout <hash>` |
| Дебаг "что сломал бот" | сложно | `git diff` |
| Доступ из агента | требует SDK + connection pool | `git add && git commit && git push` |
| Подходит для текстовых .md | overkill | идеально |

Для агента, у которого память = 5-10 текстовых файлов с обновлением раз в день — git **идеален**.

## Архитектура

```
GitHub repo (Dayron089/ldr.git)
├── branch: main                  ← код приложения, deploy, Railway watches this
└── branch: market-ldr-data       ← ТОЛЬКО память бота, Railway игнорирует
```

Две ветки в одном репо. Они **никогда не мержатся**:
- `main` — код, обновляется человеком
- `market-ldr-data` — данные, обновляется ботом

Деплой кода (push в `main`) **не задевает** память бота. Бот пушит в свою ветку, человек — в свою.

## Lifecycle одной сессии

```
1. Container starts
   ├── git fetch origin market-ldr-data
   ├── git checkout origin/market-ldr-data -- memory/ MEMORY.md HEARTBEAT.md
   └── openclaw gateway up

2. Бот работает (Дима пишет, cron triggers)
   ├── Читает knowledge base (immutable, из Docker image)
   ├── Читает memory/ (mutable, из git pull)
   └── При одобрении сценария → пишет в memory/winning-patterns.md

3. Railway отправляет SIGTERM (deploy / scale / restart)
   └── trap SIGTERM:
       ├── git add memory/ MEMORY.md HEARTBEAT.md (НЕ -A!)
       ├── git commit -m "Auto-save on shutdown <timestamp>"
       └── git push origin market-ldr-data

4. Container dies, новый стартует → шаг 1
```

## Из entrypoint.sh (упрощённо)

```bash
# Setup
cd /home/appuser/.openclaw/workspace-market-ldr
git init -b market-ldr-data
git remote add origin "https://${GITHUB_TOKEN}@github.com/Dayron089/ldr.git"
git fetch origin market-ldr-data --depth=1

# Restore mutable files only
git checkout origin/market-ldr-data -- memory/ skills/ MEMORY.md HEARTBEAT.md

# Auto-save on shutdown
save_workspace() {
  cd /home/appuser/.openclaw/workspace-market-ldr
  git add memory/ skills/ MEMORY.md HEARTBEAT.md
  git diff --cached --quiet || git commit -m "Auto-save on shutdown $(date -u +%FT%TZ)"
  git push origin market-ldr-data
}
trap save_workspace SIGTERM SIGINT

# Run
openclaw --profile market-ldr gateway --port 21872 &
wait $!
```

## Что добавляет правило `git add` без `-A`

Knowledge base файлы (`CLAUDE.md`, `48-LAWS-FULL.md`, etc.) копируются в Docker image на build time. После старта контейнера git считает их "untracked" в новой ветке. Если бы `git add -A` — бот закоммитил бы knowledge base на свою ветку, и при следующем рестарте ИЗ ВЕТКИ pulled бы свою версию вместо свежей из Docker image.

`git add memory/ skills/ MEMORY.md HEARTBEAT.md` — точечно, только мутируемые файлы. Knowledge base всегда приходит из Docker image (свежая, под контролем человека).

## Что бот пишет сам (с примерами)

### `memory/winning-patterns.md`

```markdown
## S22 — "She forgot we got married" (A, Memory)
- Approved: 2026-03-12
- Hook: "she forgot we got married." (Reddit-quote)
- Why won: language not ours (Law 11), specific numbers (Law 13)
- Pattern: Reddit-quote hooks + Memory pillar = 100% approval rate так далеко

## S18 — 3am AI is sleeping (F, Proactive)
- Approved: 2026-03-10
- Hook: "i texted her at 3am..."
- Why won: контринтуитивно (AI которая спит), Law 9 (first 3 sec) + Law 13 (specific time)
```

### `memory/rejected.md`

```markdown
## S15 — "Try this app" (drop)
- Rejected: 2026-03-08
- Reason: слишком sales-y, нарушает Law 16 (CTA = recommendation, not sales)
- Pattern to avoid: явный sales pitch в CTA

## S20 — "5 features in 60s" (drop)
- Rejected: 2026-03-13
- Reason: нарушает Law 15 (one video = one idea = one segment)
- Pattern to avoid: list-format с >3 items
```

### `MEMORY.md` (после еженедельного review)

```markdown
# Market-LDR Memory

## Winning Patterns
- Reddit-quote hooks → 100% approval rate (3/3)
- Time-specific anchors ("3am", "47 days", "8 months") → 80% approval rate
- Female actor + male product → +31% CTR (S18 V4)

## Rejected Ideas (что не повторять)
- Sales-y CTA — нарушает Law 16
- List >3 items в одном видео — нарушает Law 15
- POV без face-cam — нарушает Law 23

## Segment Insights
- B (CharAI Amnesiac) — 100% approval rate, scaling up
- D (Introvert) — 33% approval rate, портрет требует переписки

## Hook Patterns Confidence
| Pattern | Approval | n | Status |
|---|---|---|---|
| Reddit-quote | 100% | 3 | Stable winner |
| Time-specific | 80% | 5 | Strong |
| Comparison | 67% | 3 | Promising |
| Question | 33% | 3 | Drop |
```

## Доказательство что это работает

`git log origin/market-ldr-data` показывает реальные коммиты:

```
e3a2c1f Auto-save on shutdown 2026-03-15T23:04:12Z
        ── memory/winning-patterns.md (+12 lines)
        ── MEMORY.md (rewritten, weekly review delta)

b8f4d92 Auto-save on shutdown 2026-03-14T22:48:33Z
        ── memory/rejected.md (+4 lines)

a51c7e8 Auto-save on shutdown 2026-03-13T23:01:55Z
        ── memory/winning-patterns.md (+8 lines)
        ── memory/2026-03-13.md (new file)
```

Author всех коммитов — `MarketLDR Bot <market-ldr-bot@foreldr.com>`. Не Дима. Это машина пишет сама.

## Что ломается

- **GitHub rate limit на push** — каждые 2-3 минуты push не страшен, но если бы были pushes каждую секунду — упёрлись бы. На практике 1-3 push/день.
- **Race condition если 2 контейнера одновременно** — у нас 1 контейнер на Railway, не сценарий. Если бы scale → нужен lock через Redis.
- **GitHub token rotation** — токен в env vars, ротация = redeploy с новым GITHUB_TOKEN.

## Альтернативы которые рассматривались

| Альтернатива | Почему отказались |
|---|---|
| Postgres | Overkill для 5 текстовых файлов с обновлением 1-3 раза в день |
| Redis | То же. Плюс Redis — не для долговременного хранения |
| S3 / R2 bucket | Нет встроенного версионирования и diff |
| SQLite в Volume | Railway Volumes стоят денег + один failure point |
| Notion API | Сторонняя зависимость, latency, rate limits |
| **git** | Бесплатно, версионируется, история, дебаг через git log |

git выиграл по соотношению "сложность / возможности".
