# 03 — Skills pipeline (как 4 скила собираются в конвейер)

> Skills — это openclaw-расширения, которые добавляют агенту новые инструменты. У market-ldr их 4. По отдельности — просто helpers. Вместе — daily/weekly конвейер от боли пользователя в reddit до готового сценария.

---

## 4 скила

| Skill | Что добавляет | Когда вызывается |
|---|---|---|
| **reddit-scraper** | Парсинг Reddit без API ключа (через old.reddit.com) | Daily morning + по запросу "найди свежие боли" |
| **content-research** | Web search по нише AI companion / TikTok trends | Daily morning + при генерации сценариев |
| **competitor-analysis-report** | SWOT vs Replika / CharAI / Candy AI / Nomi / Lovix.ai | Weekly Mondays + по запросу "сравни с..." |
| **b2c-marketing** | 500M+ views playbook для short-form video | Reference при генерации hooks/CTA |

## Daily workflow

Cron `morning-trends` запускается в 22:30 (= утро в timezone Димы, GMT+10).

```
┌──────────────────────────────────────────────────┐
│  22:30 cron-trigger                              │
└──────────────┬───────────────────────────────────┘
               ▼
┌──────────────────────────────────────────────────┐
│  STEP 1: reddit-scraper                          │
│  Парсит:                                         │
│  ── r/Replika (Segment A)                        │
│  ── r/CharacterAI (Segment B)                    │
│  ── r/LongDistance (Segment C)                   │
│  ── r/socialanxiety (Segment D)                  │
│  ── r/CandyAI (Segment E)                        │
│  ── r/lonely (Segment F)                         │
│  Output: 30-50 свежих постов с метаданными       │
└──────────────┬───────────────────────────────────┘
               ▼
┌──────────────────────────────────────────────────┐
│  STEP 2: content-research                        │
│  Web search:                                     │
│  ── "AI companion app TikTok" (last week)         │
│  ── "AI girlfriend trend"                         │
│  ── site:tiktok.com конкуренты                    │
│  Output: 3-5 трендов / новостей за неделю        │
└──────────────┬───────────────────────────────────┘
               ▼
┌──────────────────────────────────────────────────┐
│  STEP 3: scenario generation (внутренний)        │
│  ── Reddit pain → mapping на 6 сегментов         │
│  ── Tренды → mapping на 3 pillars                 │
│  ── 48 законов → выбор формата/hook              │
│  ── Прогон через winning-patterns.md (что зашло) │
│  Output: 2-3 сценария по формату из CLAUDE.md   │
└──────────────┬───────────────────────────────────┘
               ▼
┌──────────────────────────────────────────────────┐
│  STEP 4: Telegram delivery                       │
│  "Доброе утро! Вот 2-3 сценария на сегодня:"    │
│  + ждём "да/нет" от Димы                        │
└──────────────┬───────────────────────────────────┘
               ▼
┌──────────────────────────────────────────────────┐
│  STEP 5: persistence                             │
│  ── Approved → memory/winning-patterns.md         │
│  ── Rejected → memory/rejected.md (с причиной)   │
│  ── git commit + push на market-ldr-data         │
└──────────────────────────────────────────────────┘
```

## Weekly workflow

Cron `weekly-review` запускается в воскресенье 23:00.

```
┌──────────────────────────────────────────────────┐
│  STEP 1: competitor-analysis-report              │
│  ── FORELDR vs Replika vs CharAI vs Candy        │
│  ── Что нового у конкурентов за неделю            │
│  ── SWOT, прайсинг, фичи                          │
│  Output: 1-page report                            │
└──────────────┬───────────────────────────────────┘
               ▼
┌──────────────────────────────────────────────────┐
│  STEP 2: pattern recognition                     │
│  Читает winning-patterns.md + rejected.md        │
│  Считает:                                         │
│  ── approval rate по сегментам                   │
│  ── approval rate по pillars                     │
│  ── approval rate по hook patterns               │
│  Output: агрегированная статистика               │
└──────────────┬───────────────────────────────────┘
               ▼
┌──────────────────────────────────────────────────┐
│  STEP 3: b2c-marketing review                    │
│  ── На какой phase мы (Content-Market Fit?)      │
│  ── Hook formulas — какие работают для нас       │
│  ── CTA strategy — что менять                    │
└──────────────┬───────────────────────────────────┘
               ▼
┌──────────────────────────────────────────────────┐
│  STEP 4: MEMORY.md update                        │
│  Distillation: только устойчивые insights        │
│  ── New learnings                                │
│  ── Drop patterns с low confidence               │
│  ── Update segment insights                       │
│  Cap: 200 lines (если больше — compress)         │
└──────────────┬───────────────────────────────────┘
               ▼
┌──────────────────────────────────────────────────┐
│  STEP 5: Telegram delivery                        │
│  Воскресный отчёт Диме                           │
│  + рекомендации на след. неделю                  │
└──────────────────────────────────────────────────┘
```

## Почему скилы устроены как .md а не .py

openclaw skills — это **markdown файлы с инструкциями + опционально scripts/**. Например `reddit-scraper/SKILL.md`:

```markdown
# reddit-scraper

## Description
Parse Reddit without API key, by scraping old.reddit.com.

## When to use
- User asks for recent Reddit posts
- Daily morning trend scan
- Looking for user pain points

## Usage
Call the script:
\`\`\`bash
python3 ./scripts/reddit_scraper.py --subreddit Replika --search "memory" --time week --limit 20 --json
\`\`\`

## Examples
[Дальше — примеры выходов и как их интерпретировать]
```

Агент читает SKILL.md как обычный документ, понимает когда вызывать, и запускает скрипт через bash tool. **Никакой "function calling" / tool registry.** Скил = инструкция + опциональный binary.

Это упрощает добавление новых: написал markdown — скил готов.

## Композиция скилов

Самое важное — что скилы **не изолированы**. Reddit-scraper выдаёт сырые данные, content-research добавляет контекст, competitor-analysis даёт permanent comparison, b2c-marketing задаёт frame. Агент комбинирует:

> "Reddit показывает что в r/Replika всплеск жалоб про update (reddit-scraper). На TikTok тренд: 'they broke my AI' (content-research). Lovix.ai тоже только что выкатил апдейт с регрессом (competitor-analysis). По b2c-marketing playbook фаза 4 = trend exploitation. → Сценарий: Reddit-quote hook про обнуление памяти, для Segment A + E одновременно."

Это и есть **inteliigence** — не один скил, а связь между ними. Knowledge compounds.

## Пример композиции в одной сессии

См. [`examples/03-reddit-pain-to-tiktok.md`](../examples/03-reddit-pain-to-tiktok.md) — полный pipeline от reddit поста до готового сценария с записью в память.
