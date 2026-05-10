# Skills Integration — FORELDR Marketing Pipeline

## Installed Skills
1. **reddit-scraper** — парсинг Reddit без API ключа
2. **content-research** — ресёрч трендов + генерация контента
3. **competitor-analysis-report** — конкурентный анализ (SWOT, фичи, цены)
4. **b2c-marketing** — органический рост через short-form video (500M+ views playbook)

## How They Work Together

### 🔄 Daily Workflow (утренний cron)

```
1. REDDIT SCRAPER → собираем свежие жалобы и боли
   - r/Replika, r/CharacterAI, r/replika_ai, r/lonely, r/LongDistance
   - Поиск: "AI girlfriend forgot", "character AI memory", "AI companion"
   - Выход: raw pain points, формулировки, trending complaints

2. CONTENT RESEARCH → ищем что залетает в нише
   - web_search: "AI companion app TikTok", "AI girlfriend trend"
   - Проверяем конкурентов через site:reddit.com, site:tiktok.com
   - Выход: 3-5 трендов / новостей за неделю

3. SCENARIO GENERATION → используя данные из п.1-2
   - Сопоставляем найденные боли с нашими 3 Pillars
   - Матчим с сегментами A-F из AUDIENCE-SEGMENTS.md
   - Применяем 48 Laws из 48-LAWS-FULL.md
   - Генерируем 2-3 сценария по формату из CLAUDE.md

4. APPROVAL → Дима подтверждает/отклоняет
   - Approved → memory/winning-patterns.md
   - Rejected → memory/rejected.md (с причиной)
```

### 📊 Weekly Workflow (понедельник cron)

```
1. COMPETITOR ANALYSIS REPORT
   - FORELDR vs Replika vs Character AI vs Candy AI vs Nomi vs Kindroid
   - Фичи, цены, SWOT, позиционирование
   - Что нового у конкурентов за неделю

2. B2C MARKETING REVIEW
   - Какие форматы сработали (из winning-patterns)
   - На каком этапе мы: Phase 3 (Content-Market Fit) / Phase 4 (Trends) / Phase 5 (Conversion)
   - Рекомендации по следующим шагам

3. PATTERN ANALYSIS
   - Какие сегменты/pillars/hooks побеждают
   - Что отклонено и почему — паттерны
   - Рекомендация фокуса на следующую неделю
```

### 🎯 Target Subreddits (для reddit-scraper)

**Primary (наши сегменты живут тут):**
- r/Replika — Segment A (Replika Refugees)
- r/CharacterAI — Segment B (CharAI Amnesiacs)
- r/LongDistance — Segment C (LDR Guys)
- r/socialanxiety — Segment D (Introverts)
- r/CandyAI — Segment E (Candy Disillusioned)
- r/lonely — Segment F (Night Owls)

**Secondary (тренды и индустрия):**
- r/AICompanions
- r/ArtificialIntelligence
- r/singularity

### 📐 Content-Research Brand Config

При генерации контента через content-research, использовать voice из CLAUDE.md:
- Scripts in ENGLISH (international audience)
- Analysis/discussion in RUSSIAN (Дима)
- UGC aesthetic: casual, imperfect, authentic
- Pain before solution. Always.
- One video = one idea = one segment

### 🏋️ B2C Marketing Integration

Маппинг фаз b2c-marketing на наш проект:
- **Phase 3 (Content-Market Fit)** = то, где мы сейчас
- Hook формулы из b2c-marketing → адаптировать под наши 6 сегментов
- CTA Strategy → "She actually remembers" / link in bio
- Tracking → memory/b2c-marketing-log.md (создать когда начнём постить)

## Script Paths

```bash
# Reddit
REDDIT=/root/.openclaw/workspace-market-ldr/skills/reddit-scraper/scripts/reddit_scraper.py

# Quick commands
python3 $REDDIT --subreddit Replika --sort hot --limit 10
python3 $REDDIT --subreddit CharacterAI --search "memory" --limit 10 --json
python3 $REDDIT --search "AI girlfriend app" --time week --limit 15
```
