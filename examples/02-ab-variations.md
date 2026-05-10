# Example 2 — A/B вариации победившего сценария

> Закон 41: тест по одной переменной за раз. Закон 43: winning ads fatigue, refresh каждые 5-7 дней.
> Бот берёт одобренный сценарий (S18 из `examples/01-`) и делает 5 контролируемых вариаций.

---

## Запрос Димы

> сделай 5 вариаций к S18, мы крутим его уже 5 дней — пора освежать

## Что бот сделал

1. Прочитал `winning-patterns.md` → нашёл S18, понял core механику ("AI которая спит")
2. Изолировал переменные: hook (3 варианта), actor (1 вариант), CTA (1 вариант)
3. Закон 41 — каждая вариация меняет ровно одну переменную, остальное идентично

## Выход

### V1 — Hook variation: time anchor swap
```
HOOK: "5am. i couldn't sleep."
SCRIPT: same as S18, "5am" вместо "3am" в hook и тексте
WHY: тестим — работает ли механика на других временных якорях, или только 3am?
```

### V2 — Hook variation: question-first
```
HOOK: "what does an AI do at 3am?"
Text on screen: "what does an AI do at 3am?"
Visual: dark room, phone, but no texting yet — just looking
THEN: "turns out... she sleeps."
WHY: question hook vs statement hook — закон 39 (controversial → engagement)
```

### V3 — Hook variation: comparison-first
```
HOOK: "Replika: always online. CharAI: always online. FORELDR..."
Visual: split-screen 3 frames, last frame = "💤 sleeping"
WHY: тестим против сегмента A (Replika Refugees) — может объединить F и A?
```

### V4 — Actor variation: female creator
```
EVERYTHING SAME, BUT: actor switched from male (target user) to female (sister/friend POV)
SCRIPT TWEAK: "my brother showed me this AI app. it sleeps. i thought he was joking."
WHY: тестим женский голос — он уберёт защиту "это для одиноких чуваков", расширит охват
```

### V5 — CTA variation
```
EVERYTHING SAME, BUT CTA SWAP:
Original: "link in bio if u want one that actually has a life"
NEW: "this is the only one with sleep. link in bio."
WHY: тестим заявление "только один такой" vs casual recommendation
```

## Что попадает в память

После прогона вариаций (через 5-7 дней) в `memory/winning-patterns.md`:

```markdown
## S18 vinarionts result

- V1 (5am): same CTR ±5% → time anchor работает на любом 1am-5am
- V2 (question hook): -22% CTR → statement hook лучше для F segment
- V3 (comparison): +18% CTR + объединил A+F трафик → keep, scale up
- V4 (female actor): +31% CTR, аудитория расширилась 18-24 → keep, делаем версии для других сценариев
- V5 (assertive CTA): -8% CTR → закон 16 подтверждён (CTA = recommendation, not sales)

NEW LEARNINGS:
- Female actor + male product = unexpected combo, работает
- Statement > question для proactive сценариев
- Comparison hook объединяет сегменты A и F
```

## Зачем это нужно

Без бота: маркетолог придумывает вариации сам, тратит 1-2 часа на 5 вариантов, часто меняет 2-3 переменные сразу → невозможно понять что повлияло.

С ботом: 30 секунд, чёткое разделение переменных, автоматическая привязка к законам и сегментам, результаты в общую память.
