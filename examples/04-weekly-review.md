# Example 4 — Еженедельный обзор (cron Sun 23:00)

> Каждое воскресенье в 23:00 бот сам читает `winning-patterns.md` и `rejected.md`, считает паттерны, обновляет `MEMORY.md` и приходит в Telegram с отчётом.

---

## Cron payload

```json
{
  "name": "weekly-review",
  "schedule": { "kind": "cron", "expr": "0 23 * * 0" },
  "payload": {
    "kind": "agentTurn",
    "message": "Еженедельный обзор. Прочитай memory/winning-patterns.md и memory/rejected.md. Подведи итоги: сколько сценариев предложено, одобрено, отклонено за неделю. Какие сегменты и pillars работают лучше. Рекомендации на следующую неделю. Обнови MEMORY.md."
  }
}
```

## Что бот сам сделал

1. Прочитал оба файла памяти
2. Распарсил даты, посчитал агрегаты
3. Нашёл паттерны (какие сегменты/pillars/hooks побеждают)
4. Обновил `MEMORY.md`
5. Запушил изменения в ветку `market-ldr-data`
6. Прислал отчёт в Telegram

## Выход в Telegram

```
📊 Weekly Review — Week 11 (8-14 Mar 2026)

ИТОГИ НЕДЕЛИ
- Предложено: 18 сценариев
- Одобрено: 11 (61%)
- Отклонено: 7 (39%)
- A/B вариаций сделано: 3 серии (15 вариантов)

ТОП ПО APPROVAL RATE
1. Pillar Memory — 5/6 одобрено (83%)
2. Segment B (CharAI Amnesiac) — 4/4 одобрено (100%)
3. Reddit-quote hooks — 3/3 одобрено (100%)

ХУЖЕ ВСЕГО
- Pillar Trust — 2/5 одобрено (40%) — Дима жалуется что "trust 0->100" звучит абстрактно. Нужно конкретнее: до/после поведение в одном кадре.
- Segment D (Introvert) — 1/3 — кажется hook'и слишком "soft", нужно agressive pain.
- POV формат без face-cam — 0/2 — face-cam обязателен (Law 23).

REJECTED PATTERNS (что не повторять)
- "Try this app" — слишком sales-y, нарушает Law 16
- Список 5+ фич в одном видео — нарушает Law 15
- CTA "click the link" без контекста — нарушает Law 16

NEW LEARNINGS (записал в MEMORY.md)
- Reddit-quote hooks → высокий save rate (Law 38), стабильный winner
- Female actor в B сегменте +31% CTR — попробовать на A и F
- "47 days" / "3am" / другие конкретные числа = базовый паттерн (Law 13 в действии)

РЕКОМЕНДАЦИИ НА СЛЕД. НЕДЕЛЮ
1. Удвоить Reddit-pain-mining: 2 раза в день вместо 1
2. Запустить вариации S18 с female actor (V4 показал +31%)
3. Pillar Trust — нужны "до/после" сценарии, а не абстрактные обещания
4. Segment D — обновить портрет в AUDIENCE-SEGMENTS.md, текущий слишком мягкий

Sheets обновлены, MEMORY.md закоммичен. Готов к следующей неделе.
```

## Что попало в память

`MEMORY.md` (delta после этой недели):

```markdown
## Segment Insights

### Segment B (CharAI Amnesiac) — 100% approval rate
- Reddit-quote hooks работают идеально (3/3)
- Conversational language, не маркетинг
- Specific number'ы обязательны ("47 days later")

### Segment D (Introvert) — нужна перенастройка
- Текущие портреты слишком "мягкие"
- Aggressive pain работает лучше soft pain
- TODO: переписать AUDIENCE-SEGMENTS.md → D

## Hook Patterns (рост confidence)

| Pattern | Approval rate | Sample size | Status |
|---|---|---|---|
| Reddit-quote | 100% | 3 | Stable winner |
| Time-specific ("3am", "5am") | 80% | 5 | Strong |
| Comparison hook ("Replika: X. CharAI: Y. FORELDR:...") | 67% | 3 | Promising |
| Question hook | 33% | 3 | Weak — drop unless format changes |
```

## Зачем это нужно

Это не отчёт. Это система **которая знает что работает на этой неделе** и предлагает следующий шаг сама. Без бота — папка из 100 google docs которую никто не перечитывает. С ботом — knowledge compounds, агент стартует следующую неделю с контекстом всей истории.
