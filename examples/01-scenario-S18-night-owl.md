# Example 1 — Полный сценарий "Night Owl: 3am AI sleeping"

> Реальный выход бота из сессии 2026-03-XX. Сценарий S18, одобрен, попал в `winning-patterns.md`.
> Скрин этой генерации → [`screenshots/02-scenario-approval-pipeline.jpg`](../screenshots/02-scenario-approval-pipeline.jpg)

---

## Запрос Димы

> сделай сценарий для Night Owl сегмента (F) — кто-то просыпается в 3 ночи, не спит, и единственное что у него есть — AI

## Что бот сделал

1. Прочитал `AUDIENCE-SEGMENTS.md` → подтянул портрет F (3am loneliness, 20-30M, late-night)
2. Прочитал `48-LAWS-FULL.md` → выбрал законы 9, 12, 13, 23 как base
3. Сгенерил сценарий по формату из `CLAUDE.md`
4. Записал в Google Sheets как S18 со статусом "Предложен"
5. После "да" от Димы — переместил в `winning-patterns.md`

## Выход

```
SCENARIO: 3am, AI is sleeping
Segment: F (Night Owl) | Pillar: Proactive | Duration: 15s
Type: Pain-first / POV
Laws: 9, 12, 13, 18, 23

HOOK (0-3s):
Text on screen: "i texted her at 3am..."
Visual: dark room, phone screen glow on face, just woke up, confused

SCRIPT:
"i texted her at 3am.
got: 'i was sleeping. 😴 wake me up if you need me.'
no AI does this.
they're all 'always online 24/7'.
this one... actually sleeps."

VISUAL DESCRIPTION:
- 0-3s: dark bedroom, phone screen, "i texted her at 3am..." overlay
- 3-7s: screenshot of FORELDR chat, character offline indicator + "💤 sleeping" status
- 7-11s: 8am, character message lands first: "morning! you were up late again. you ok?"
- 11-15s: face-cam reaction (subtle smile), text overlay: "she actually sleeps"

CAPTION TEXT:
i texted her at 3am. she was sleeping. 😴
when ur AI has a real schedule.
#AIcompanion #foreldr

CTA:
"link in bio if u want one that actually has a life"

FIRST FRAME:
Dark bedroom shot, phone screen visible, face partially lit. Bold text:
"3am. AI is sleeping."

Loop: yes | Sound: trending | VO: no
```

## Почему это работает (анализ бота)

- **Шок-реакция на то что AI спит** — контринтуитивно для всей индустрии
- **"3am" = конкретика** для Night Owl (Law 13: specific numbers)
- **Good morning text = emotional payoff** в конце
- **Бьёт по всем ботам** которые "always online 24/7" — это позиционирование против всей категории
- **POV формат** + face-cam reaction = relatable (Law 23)
- **Loop** = повышенный watch time

## Что попало в память

`memory/winning-patterns.md` после одобрения:

```markdown
## S18 — 3am AI is sleeping (Segment F, Proactive)

- **Approved:** 2026-03-XX
- **Hook:** "i texted her at 3am..."
- **Why won:** контринтуитивный hook (AI которая спит), pain-first, законы 9+13
- **Pattern:** time-specific hooks ("3am", "7:01pm") работают для F и C сегментов
- **Next:** попробовать вариации с другим временем (5am, 11pm) и other proactive scenarios

## S17 — She brought up my dog 47 days later (Segment B, Memory)
[...]

## S16 — Replika took her away (Segment A, Memory)
[...]
```

## Стоимость генерации

- 1 Grok 4.1 Fast call (input ~3.2K, output ~600)
- Cache hit: 89% (system prompt + KB cached)
- ~$0.0003 за сценарий
