# Apify Validation — як провалідувати LinkedIn-аудиторію, зібрану з Apollo

Це мозок ФАЗИ 6. Apollo дає список, але дані можуть бути **застарілі**. Перш ніж писати цепочку — валідуємо через Apify. Логіка: знайти потрібний actor у Apify Store → показати студенту → запропонувати запустити.

> **Канал — тільки LinkedIn.** Валідуємо LinkedIn-сигнали, не email.
> **Принцип:** не пиши дотик у мертвого/неактивного ліда. Валідація піднімає reply rate у 3-4 рази.

---

## Чому Apollo-аудиторію треба валідувати (сказати студенту)

| Ризик у Apollo-даних | Що відбувається | Чим валідуємо в Apify |
|---|---|---|
| Людина пішла з компанії | дотик б'є повз, "хто це?" | LinkedIn profile scraper → поточна посада |
| Профіль неактивний | інвайт висить місяцями, 0 reply | linkedin profile posts → активність ≥5/30д |
| Немає свіжого тригера | дотик без приводу = шаблон | profile posts → шукати pain/hiring пост |

---

## 3 виміри валідації (LinkedIn) → який actor шукати

| Вимір | Що перевіряємо | Пошук у Apify Store (keyword) | Поріг / вихід |
|---|---|---|---|
| **Активність** ⭐ | ≥5 подій (пости/лайки/коментарі) за 30 днів | `linkedin profile posts` | active / passive |
| **Актуальність посади** | чи людина досі на цій ролі/в компанії | `linkedin profile scraper` | match / changed |
| **Свіжий тригер** | пост про біль / найм / зміну за 30 днів | `linkedin profile posts` | є тригер / немає |

> Для outbound вирішальний вимір — **активність**. Якщо бюджет/час обмежені — валідуй хоча б її.

---

## Як скіл знаходить endpoint у Apify (динамічно — НЕ хардкодити ID)

**Крок 1 — пошук у Apify Store** (публічний API, токен не потрібен для пошуку):
```bash
curl -s "https://api.apify.com/v2/store?search=linkedin+profile+posts&limit=5" \
  | jq '.data.items[] | {name: .name, username: .username, runs: .stats.totalRuns, price: .currentPricingInfo.pricePerUnitUsd}'
```
→ Показати студенту 2-3 топ-actors з кількістю запусків + ціною. Рекомендувати з найбільшими `totalRuns` (перевірений).

**Крок 2 — запропонувати запуск** (потрібен Apify token студента):
```bash
curl -s -X POST "https://api.apify.com/v2/acts/{ACTOR_ID}/runs?token=$APIFY_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{ "profileUrls": ["https://linkedin.com/in/...","..."] }'   # поля залежать від actor
```
→ Спершу прочитати input schema actor'а: `GET https://api.apify.com/v2/acts/{ACTOR_ID}/input-schema`. Не вгадувати поля.

**Крок 3 — забрати результат:**
```bash
curl -s "https://api.apify.com/v2/acts/{ACTOR_ID}/runs/last/dataset/items?token=$APIFY_TOKEN"
```

---

## Перевірені actors для старту (станом на 2026 — все одно звірити в Store)

- **Активність / пости:** `harvestapi/linkedin-profile-posts` — останні пости, лайки, реакції.
- **Профіль / посада:** шукати `linkedin profile scraper` → брати топ за `totalRuns`.

> ⚠️ ID actors у Store змінюються. Завжди спершу `?search=` → бери живий, з реальними запусками. Не посилай студента на мертвий actor.

---

## Простий шлях (fallback — без ручного Apify)

Якщо у студента вже є наш скіл **`linkedin-enrich`** — він обгортає весь цей Apify-пайплайн (profigent.ai → Supabase) одним запуском:
- Передати список LinkedIn URL → `linkedin-enrich` дістає активність → скорить → пише в таблицю.
- Тоді ФАЗА 6 = просто виклик `linkedin-enrich`, а не ручні curl.

Рекомендувати цей шлях новачкам; ручний Apify API — для тих, хто хоче контролю.

---

## Що скіл виводить після валідації

Оновити `outreach/leads.md`: додати колонки `active (y/n)` · `still_at_company (y/n)` · `trigger`.
Відсіяти: passive + посада changed → у `parked`. Решта → в цепочку (ФАЗА 7).

**Сказати студенту цифру:** з 20 Apollo-лідів після валідації зазвичай залишається 12-15 "живих". Саме по них пишемо цепочку. Решта — даремні дотики, якщо не відсіяти.

---

## Cost / етика

- Apify тарифікується за запуск/результат — попередити: валідація 20 профілів ≈ $1-3 залежно від actor.
- Тільки публічні LinkedIn-дані. Не масштабувати скрейпінг особистих даних поза B2B legitimate interest.
- Не запускати actor без явного "так" студента (показати ціну ДО запуску).
