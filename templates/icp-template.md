---
type: icp
created: [YYYY-MM-DD]
status: draft
---

# ICP — [назва сегмента]

## Хто компанія
- **Індустрія / вертикаль:** [1-3]
- **Розмір:** [1-10 / 11-50 / 51-200 / 201+]
- **Гео:** [країни / міста]
- **Стадія / виручка:** [bootstrapped / seed / Series A+ / —]

## Хто людина
- **ЛПР (підписує чек):** [посада]
- **Інфлюенсер (рекомендує):** [посада]
- **Seniority:** [c_suite / vp / director / manager]

## Біль і тригер
- **Топ-1 біль (verbatim):** "[словами клієнта]"
- **Buying signal (гаряче):** [найм ролі / раунд / зміна посади / пост про біль / зміна стеку]
- **Стек = ознака клієнта:** [HubSpot / Salesforce / Webflow / …]

## Оффер і докази
- **Оффер (1 речення):** [що пропонуємо]
- **Головне заперечення ICP:** [«у нас вже є команда» / «дорого» / …]
- **Proof point з цифрою:** [кейс + метрика + джерело]

## Apollo-фільтри (готові до вставки)
```json
{
  "person_seniorities": ["[c_suite/vp/...]"],
  "organization_num_employees_ranges": ["[11,50]"],
  "person_locations": ["[geo]"],
  "currently_using_any_of_technology_uids": ["[tech]"],
  "organization_job_postings_titles": ["[роль, яку наймають — intent]"]
}
```

> ⚠️ Для hiring-сигналу — endpoint `organizations_job_postings.posted_titles`, НЕ `q_organization_job_titles`.
> Канал — LinkedIn: нам потрібен LinkedIn URL ліда, не email.
