# 01 — Anatomy агента

> Как market-ldr устроен изнутри. Что immutable, что мутирует, как кусочки собираются в один контекст для LLM.

---

## Три слоя контекста

При каждом турне (запрос Димы или cron-trigger) openclaw собирает контекст из трёх слоёв:

```
┌─────────────────────────────────────────────────────┐
│  SLOT 1: Identity (immutable, малый, всегда грузится) │
│  ── CLAUDE.md       (system prompt, ~5K токенов)      │
│  ── IDENTITY.md     (имя, vibe, ~200 токенов)         │
│  ── SOUL.md         (характер, границы, ~500 токенов) │
│  ── HEARTBEAT.md    (что проверять при старте)        │
└─────────────────────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────┐
│  SLOT 2: Knowledge (immutable, большой, по нужде)     │
│  ── 48-LAWS-FULL.md       (~6K)                       │
│  ── AUDIENCE-SEGMENTS.md  (~3K)                       │
│  ── SCENARIOS.md          (~8K — 30+ сценариев)       │
│  ── STRATEGY.md           (~5K)                       │
│  ── FUNNEL.md             (~4K)                       │
│  ── CONTENT-PLAYBOOK.md   (~4K)                       │
│  ── MARKET-INTEL-2026-03.md  (~3K)                    │
│  ── AGENTS.md             (~3K)                       │
└─────────────────────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────┐
│  SLOT 3: Memory (mutable, агент пишет сам)            │
│  ── MEMORY.md                  (агрегированные insights) │
│  ── memory/winning-patterns.md (одобренные сценарии)   │
│  ── memory/rejected.md         (отклонённые + причины) │
│  ── memory/YYYY-MM-DD.md       (дневные заметки)       │
└─────────────────────────────────────────────────────┘
```

## Что immutable / что мутирует

**Immutable** (правит только человек):
- `CLAUDE.md` — system prompt. Менять = менять роль агента
- Knowledge base (14 файлов) — это домен-знания. Если меняется стратегия / сегменты / сценарии-чемпионы, человек обновляет руками

**Mutable** (агент пишет сам через git push):
- `MEMORY.md` — distilled insights. Бот сам сжимает память каждую неделю
- `memory/` — сырьё фидбека. Каждое "да/нет" Димы → запись
- `HEARTBEAT.md` — может обновлять при добавлении новых проверок

## Зачем такое разделение

**Если всё immutable** — агент не учится. Каждая сессия = чистый лист.

**Если всё mutable** — агент дрифтует. Через месяц забудет что он про FORELDR / превратится в general AI assistant из-за compaction.

Разделение immutable + mutable = **долгосрочная стабильность личности + накопление опыта**.

## Anatomy одного запроса

Дима пишет "сделай сценарий для Night Owl сегмента".

```
1. openclaw грузит SLOT 1 (Identity, всегда)
   → CLAUDE.md, IDENTITY.md, SOUL.md, HEARTBEAT.md

2. openclaw читает HEARTBEAT.md
   → "проверь что MEMORY.md свежий, проверь pending tasks"

3. openclaw грузит SLOT 3 (Memory)
   → MEMORY.md, winning-patterns.md, rejected.md

4. Агент решает что нужно из SLOT 2 (Knowledge)
   → "Night Owl = Segment F → читаю AUDIENCE-SEGMENTS.md (только секцию F)"
   → "Сценарий → читаю SCENARIOS.md (для inspiration), 48-LAWS-FULL.md"

5. Агент генерит сценарий через Grok 4.1 Fast
   → System prompt (SLOT 1) кэшируется (96% hit rate)
   → Knowledge (SLOT 2) кэшируется тоже
   → Только пользовательский запрос + memory delta — не кэшируются

6. Агент возвращает сценарий
7. Дима говорит "да" → агент пишет в winning-patterns.md → git push
```

## Стоимость одного турна

| Compoнент | Tokens | Cached? | $ (xAI Direct) |
|---|---|---|---|
| Identity (SLOT 1) | ~6K | ✅ 96% | $0.00003 |
| Knowledge selected (SLOT 2) | ~10K avg | ✅ 89% | $0.00007 |
| Memory (SLOT 3) | ~2K | partial | $0.00006 |
| User prompt | ~200 | ❌ | $0.00004 |
| Output (scenario) | ~600 | ❌ | $0.00060 |
| **Total** | ~18.8K | — | **~$0.0008** |

То есть полный сценарий с памятью + 14 файлов knowledge → меньше цента.

## Что бы сломалось без слоёв

Если бы system prompt = "ты маркетолог FORELDR, делай сценарии под TikTok" (один абзац вместо 14 файлов):

- ❌ Не знает 6 сегментов → дженерик "AI girlfriend pain"
- ❌ Не знает 48 законов → не может объяснить почему сценарий сильный
- ❌ Не помнит что одобрено → повторяет отклонённые идеи
- ❌ Не знает конкурентов → даёт generic positioning
- ❌ Не знает воронку → CTA в стиле "click the link"

Иначе говоря: получится тот же ChatGPT с пустым промптом. Эксклюзив агента — в knowledge + memory, не в LLM.
