---
client: profigent
lesson: 3
type: skill-spec
title: "Lesson 3 Apollo Outreach Skill — SPEC"
status: spec (to build / package as downloadable, equivalent of Lesson_2 zip)
stack: Apollo MCP + Claude Code + Apify (activity layer)
---

# Lesson 3 Skill — Apollo Outreach (SPEC — рушій)

Це специфікація **рушія** для Lesson 3 — 4 технічні команди, які працюють з Apollo MCP.

> **Архітектура:** студент взаємодіє з обгорткою `outreach-architect` (інтерактивний скіл-консультант, головна роздатка Lesson 3). Вона під капотом викликає ці 4 команди. Цей SPEC = двигун; `outreach-architect/SKILL.md` = водій. Пакуються разом у `profigent-lesson_3.zip`.

Студент додає скіл у свій Claude Project (з L1 + L2) і отримує 4 команди outreach з коробки.

**Передумова у студента:** Apollo MCP залогінений (`/mcp` → Apollo → connected) + voice-profile.json з L1.

---

## Команди скіла

> Канал — тільки LinkedIn. Email-команд (`apollo:sequence`) у скілі немає.

| Команда | Що робить | Endpoint / залежність |
|---|---|---|
| `/apollo:enrich-lead` | Один контакт → LinkedIn URL, посада, job history, tech stack | `apollo_people_match` |
| `/apollo:prospect` | ICP-речення → ланцюжок job_postings → companies → people → табличка лідів | `organizations_job_postings` + `mixed_companies_search` + `mixed_people_api_search` |
| `/apollo:verify` | Список лідів → 3-layer ICP-фільтр → fit_score + timing + action (JSON) | LLM + (опц.) Apify активність |

---

## 1. `/apollo:prospect` (зірка модуля)

**Вхід:** ICP одним-двома реченнями (industry / size / role / geo / tech / signal).

**Кроки (Claude виконує сам через MCP):**
1. Якщо в ICP є hiring-сигнал → `apollo_organizations_job_postings` з `posted_titles`.
2. `apollo_mixed_companies_search` — hard-фільтри (size, industry, geo, tech `currently_using_any_of_technology_uids`).
3. Перетин списків компаній (1 ∩ 2).
4. `apollo_mixed_people_api_search` по company_ids + `person_seniorities`.
5. Output: Markdown табличка `Name | Title | Company | LinkedIn | Hiring Signal`.

**⚠️ Hard rule:** ніколи не плутати `q_organization_job_titles` (компанії, де роль ВЖЕ Є) з `organizations_job_postings.posted_titles` (компанії, які ЗАРАЗ наймають = intent). Завжди друге для buying signal.

**⚠️ Credits:** глибокий enrich спалює кредити Apollo — показувати лічильник до/після. Нам потрібен LinkedIn URL, не email.

---

## 2. `/apollo:verify` (AI-фільтр)

**Вхід:** список лідів з `/apollo:prospect` (або CSV).

**Промпт-ядро:**
```
Ти — ICP qualifier. Для кожного ліда оціни:
1. fit_score (0-10) проти ICP: {{ICP в 2 реченнях}}
2. signals[]: hiring / funding / tech change / new role
3. timing: hot / warm / cold
4. reason: 1 речення
5. action: invite / like-post / comment / skip
Поверни JSON масив, відсортований за fit_score desc.
```

**Activity layer (опційно, через існуючий скіл `linkedin-enrich`):**
- Apify actor `harvestapi/linkedin-profile-posts` → останні пости/лайки/реакції.
- Поріг `≥5 подій / 30 днів = active`. Active → response rate ×3-4.
- Інтеграція: `linkedin-enrich` пише активність у Supabase → `/apollo:verify` читає як Layer 3.

---

## 3. Валідація через Apify (перед цепочкою)

- Перевірити LinkedIn-сигнали зібраних лідів: активність ≥5/30д · актуальність посади · свіжий тригер.
- Пошук actor: `GET https://api.apify.com/v2/store?search=...` → топ за `totalRuns`.
- Fallback — скіл `linkedin-enrich` (один запуск).
- Відсіяти мертвих → цепочку пишемо тільки по живих.

---

## 4. Touchpoint Loop — два рівні (тільки LinkedIn)

**🟢 Tier 1 — Default (скіл драфтить, людина відправляє):**
- `/apollo:prospect` → `/apollo:verify` → Apify-валідація → для кожного HOT-ліда Claude генерує персоналізований інвайт (≤200 знаків, 1 конкретна деталь).
- Студент копіює → вставляє в LinkedIn → тисне send вручну.
- Цепочка: invite → like → DM → comment → DM (без email).
- **Ризик банy: ~0.**

**🔴 Tier 2 — Advanced (full-auto, окремий модуль, на свій ризик):**
- Apollo webhook `invite_accepted` → n8n → Claude DM → Unipile send.
- Apollo webhook `invite_not_accepted` + 3d → Apify пости → авто-лайк → авто-коментар.
- Ліміт 80-100 інвайтів/тиждень + warm-up обов'язкові.
- **НЕ включати в базовий скіл.** Документувати окремо як advanced add-on.

---

## Залежні / суміжні скіли (вже існують у проекті)

- `linkedin-enrich` — Apify → Supabase enrichment + activity scoring (Layer 3 для verify).
- `outreach-collector` — мульти-клієнтський збирач аудиторії (згадати як next step).
- Команди `/outreach strategy|sequence|pipeline|log|report` з CLAUDE.md — куди приземляються результати prospect/verify.

---

## Чек-лист пакування (для викладача)

- [ ] Зібрати 3 команди (enrich / prospect / verify) як рушій під `outreach-architect`
- [ ] Прописати MCP-залежність (Apollo) у README скіла
- [ ] Вшити промпт-ядро `/apollo:verify` (Slide 14 лекції)
- [ ] Hard-rule про `job_postings` vs `job_titles` — у SKILL.md великим
- [ ] Канал — тільки LinkedIn (без email/Apollo Sequences)
- [ ] Tier 1 = default; Tier 2 — окремий файл `advanced-fullauto.md`
- [ ] Запакувати `outreach-architect` + цей SPEC + handout у `profigent-lesson_3.zip` → у чат групи (як Lesson_2)
