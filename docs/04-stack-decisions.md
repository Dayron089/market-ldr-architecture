# 04 — Stack decisions

> Почему Grok, почему openclaw, почему Telegram, почему git-as-memory. Каждый выбор — trade-off, и здесь они зафиксированы.

---

## Модель: Grok 4.1 Fast (xAI Direct)

### Почему Grok 4.1 Fast, а не Claude / GPT-4

| Параметр | Grok 4.1 Fast | Claude 3.5 Sonnet | GPT-4o |
|---|---|---|---|
| Cost (input, cached) | $0.05/1M | $0.30/1M (cache write) / $0.03 (cache read) | нет prompt caching |
| Cost (input, no cache) | $0.20/1M | $3.00/1M | $2.50/1M |
| Context | 131K | 200K | 128K |
| Tool calling | да | да | да |
| Quality для маркетингового тонo | хорошо | очень хорошо | хорошо |

Решающий фактор: **prompt caching**. У market-ldr 14 файлов knowledge base = ~15-20K токенов immutable контекста. Без cache — каждый турн стоит ~$0.06. С cache — ~$0.001. Это разница между "$10/день" и "$0.20/день" при средней нагрузке.

xAI Direct даёт самый дешёвый cached input на рынке ($0.05/1M). Cache hit rate в проде = 96%.

### Почему xAI Direct, а не OpenRouter

OpenRouter — agregator с наценкой ~10-20% и **без поддержки prompt caching**. Для агента, у которого 96% контекста immutable — это бессмысленный overpay.

Конфиг: xAI Direct primary, OpenRouter — fallback на случай 503.

### Почему non-reasoning вариант (`grok-4-1-fast-non-reasoning`)

Маркетинговый агент не нуждается в reasoning chain. Каждое решение — это "выбери hook из 4 вариантов под закон 9". Reasoning добавляет ~3-5x latency и стоимость без улучшения качества выходов. Тест проводили — non-reasoning выдаёт идентичные сценарии.

`temperature: 0.3` — низкая для consistency сценариев. Не 0.0 (нужна вариативность хуков).

## Runtime: openclaw

### Почему не LangChain / CrewAI / AutoGen

| Фреймворк | Проблема для market-ldr |
|---|---|
| LangChain | Слишком много абстракций. Агент = кастомный pipeline, нужен low-level контроль над контекстом |
| CrewAI | Multi-agent ориентирован, у нас 1 агент. Overkill |
| AutoGen | То же. Плюс плотная связность с Microsoft stack |
| **openclaw** | File-system-based: knowledge = .md файлы, скилы = .md инструкции, никаких abstract pipelines |

openclaw — это runtime который **читает .md файлы как контекст** и даёт shell + git tools агенту. Минимализм, прозрачность.

Главное преимущество: **дебаг**. Если агент странно ответил — открываешь его файлы, читаешь как обычный markdown, видишь что в памяти. В LangChain — лезешь в trace через UI.

### Что openclaw даёт из коробки

- Telegram channel (whitelist, dm policy)
- Cron scheduler (в одном процессе)
- Multi-LLM provider routing (xai, openai, ...)
- Skills (markdown + scripts)
- Web search tool
- Bash tool (для скилов)
- Compaction режим safeguard

Вся инфраструктура — в одном `openclaw.json` конфиге (~100 строк).

## Канал: Telegram

### Почему не web-app / Slack / Discord

Web-app — нужен auth, UI, hosting, маршрутизация. Слишком тяжело для агента который работает с **одним юзером**.

Slack / Discord — workspace-oriented, для команд. Здесь команда = 1 человек.

Telegram:
- Single user whitelist в одну строку конфига (`allowFrom: [admin_id]`)
- Mobile push из коробки
- Файлы / голосовые / форматирование code blocks
- Bot API стабильный

Латентность: ~200ms от Telegram до openclaw, ~800ms total с Grok response = терпимо для "ассистента в чате".

## Persistence: git ветка

### Почему не Postgres / Redis / S3

См. подробно [`docs/02-self-modification.md`](02-self-modification.md). Кратко:
- Postgres — overkill для 5 .md файлов с обновлением 1-3 раза в день
- Redis — не для долговременного хранения
- S3 — нет diff/history без extra работы
- **git** — встроено, бесплатно, версионируется, дебаг через `git log`

Trade-off: write throughput низкий (1 commit/push цикл = ~2-5 секунд). Не подойдёт для агента которому нужны write 100x/секунду. Для маркетингового агента который пишет "одобрено: S22" 5 раз в день — идеально.

## Хостинг: Railway

### Почему не AWS / GCP / Fly.io

AWS / GCP — нужен IAM / VPC / load balancers. У агента нет горизонтального масштабирования (1 контейнер).

Fly.io — отличный, но Railway уже используется для FORELDR backend. Один dashboard, один билл.

Railway даёт:
- Auto-deploy из git push на main
- Persistent env vars (для GITHUB_TOKEN, XAI_API_KEY)
- SIGTERM hook на restart → можем сохранить state в git
- $5/мес базовый план хватает с запасом

Без attached Volumes (платные) — всё state идёт в git ветку.

## Cron: openclaw native

### Почему не GitHub Actions / cron в отдельном worker

GitHub Actions:
- Запуск через minutes-precision не гарантирован
- Нет state между запусками без artifacts/cache hacks
- Bot не может сам обновить cron-jobs.json без нового PR

External cron worker (например на Railway):
- Дополнительный сервис → дополнительный $5/мес
- Нужно общаться с агентом по сети
- Лишняя точка отказа

openclaw native scheduler:
- В одном процессе с агентом
- `cron-jobs.json` — конфиг с crontab + payload (полный prompt)
- При SIGTERM сохраняется как часть state
- Бот может сам редактировать cron-jobs.json (теоретически)

## Stack summary

```
┌─────────────────────────────────────┐
│  openclaw runtime                   │
│  ├── xAI Grok 4.1 Fast (primary)    │
│  ├── OpenAI embeddings              │
│  ├── Telegram channel               │
│  ├── Cron scheduler                  │
│  ├── Bash + Web search tools        │
│  └── Skills (4 .md packages)        │
└─────────────────────────────────────┘
              │
              ▼
┌─────────────────────────────────────┐
│  Docker container                    │
│  ├── Knowledge base (immutable)      │
│  └── Memory (mutable, git-synced)    │
└─────────────────────────────────────┘
              │
              ▼
┌─────────────────────────────────────┐
│  Railway hosting                     │
│  ├── Auto-deploy from main branch    │
│  ├── SIGTERM hook → git push         │
│  └── ~$5/mo                          │
└─────────────────────────────────────┘
```

Total стоимость:
- Railway hosting: ~$5/мес
- xAI Grok (96% cache hit): ~$2-5/мес при текущей нагрузке
- OpenAI embeddings: ~$0.50/мес
- **Итого: ~$8-12/мес**

Для сравнения — джуниор маркетолог-фрилансер начинается от $500/мес и не работает в 3 ночи.
