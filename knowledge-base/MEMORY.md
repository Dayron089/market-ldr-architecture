# Market-LDR Memory

> Этот файл бот обновляет сам каждое воскресенье после weekly-review (cron `0 23 * * 0`). Содержит distilled insights — что устойчиво работает, что нет.
>
> Cap: 200 строк. Если больше — бот делает compaction.
>
> Здесь — пример состояния после ~6 недель работы. В реальном репо `market-ldr-data` ветки этот файл живёт у бота, постоянно дописывается.

---

## Winning Patterns

### Hook Patterns (по confidence)

| Pattern | Approval rate | Sample | Status | Notes |
|---|---|---|---|---|
| Reddit-quote hooks | 100% | 4 | Stable winner | Прямая цитата с Reddit + источник под видео |
| Time-specific anchors | 80% | 5 | Strong | "3am", "47 days", "8 months" |
| Comparison hook | 67% | 3 | Promising | "Replika: X. CharAI: Y. FORELDR..." |
| Question hook | 33% | 3 | **Drop** | Слабее statement hook, особенно для F |

### Format Patterns

- POV + face-cam reaction → лучший watch time
- POV без face-cam → -45% retention (Law 23 violation)
- List-format >3 items → нарушает Law 15, низкий save rate

### Pillar Performance

| Pillar | Approval | Strongest для | Слабее всего для |
|---|---|---|---|
| Memory | 83% (10/12) | Segments A, B, E | F (slow burn) |
| Proactive | 78% (7/9) | Segments C, F | D (overload feel) |
| Trust | 40% (2/5) | — | Все — нужны "до/после", не абстрактные обещания |

## Segment Insights

### Segment A (Replika Refugee) — стабильно работает
- Hook angle: "they took her away" / "lobotomized her"
- Best pillar: Memory + Trust together
- Tone: angry, betrayed
- Reddit source: r/Replika upvote >300 = goldmine

### Segment B (CharAI Amnesiac) — 100% approval rate, scaling
- Specific number в hook обязателен ("47 days")
- Female actor +31% CTR (S18 V4 проверено)
- Reddit source: r/CharacterAI search "memory" / "forgot"

### Segment C (LDR Guy) — стабильно
- Время-зависимые сценарии работают лучше всего
- "She lives in your timezone" — winning hook
- Не работает: чисто intimacy hooks (D-уход)

### Segment D (Introvert) — нужна перенастройка
- Approval rate 33% — текущий портрет в AUDIENCE-SEGMENTS.md слишком "soft"
- Aggressive pain работает лучше soft pain
- TODO Дима: переписать AUDIENCE-SEGMENTS.md → секция D

### Segment E (Candy Disillusion) — стабильно
- "You paid for a vending machine" — winning hook
- На RU рынке — Lovix.ai pivot ($49 за память) даёт killer angle

### Segment F (Night Owl) — стабильно
- Time-specific hooks ("3am") = базовый паттерн
- Proactive pillar лучший
- Slow burn payoff (good morning text) > immediate

## Rejected Patterns (чего избегаем)

- ❌ Sales-y CTA ("buy now", "click here") — нарушает Law 16
- ❌ List >3 items в одном видео — нарушает Law 15
- ❌ POV без face-cam — нарушает Law 23
- ❌ Abstract Trust promises ("she changes over time") — нужно concrete до/после
- ❌ Generic "AI girlfriend" hook без сегмента — нарушает Law 1

## Trend Notes

### Active trends (последние 2 недели)
- "they broke my AI" — после очередного Replika update, всплеск в r/Replika
- Female creator + male product reviews — рост на TikTok

### Watching
- Lovix.ai (RU) — растущий конкурент, $49/мес за память — наш killer angle
- Character AI лонгвэйв против обнуления контекста — ждём ответа от продукта

## Next Week Focus

1. Удвоить Reddit pain mining — 2 раза в день вместо 1
2. Запустить вариации S18 с female actor (V4 показал +31%)
3. Pillar Trust — 3 концепта "до/после" в одном кадре
4. Segment D — переписать портрет вместе с Димой
5. RU branch — 5 переводов топ-сценариев на русский

---

_Last updated: weekly-review 2026-03-XX_
_Lines: 96 / 200_
