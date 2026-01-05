# Документація структури бази даних

**Дата генерації:** 2025-01-05  
**СУБД:** PostgreSQL  
**Загальна кількість таблиць:** 22

---

## Зміст

### Основні таблиці
1. [applications](#applications)
3. [application_types](#application-types)
4. [application_statuses](#application-statuses)
5. [application_priorities](#application-priorities)
6. [clients](#clients)
7. [profiles](#profiles)
8. [contacts](#contacts)
9. [companies](#companies)
10. [company_clients](#company-clients)
11. [addresses](#addresses)
12. [documents](#documents)
13. [document_files](#document-files)
15. [order_decisions](#order-decisions)
16. [events](#events)
17. [relationships](#relationships)
18. [audit_log](#audit-log)
19. [skill_groups](#skill-groups)
20. [orders](#orders)
21. [action_types](#action-types)
22. [default_first_actions](#default-first-actions)

### Тимчасові таблиці міграції
2. [tbl_test_uyp](#tbl-test-uyp)
14. [contragent](#contragent)

---

## applications

**Призначення:** Основна таблиця заявок клієнтів. Містить заявки на відкриття рахунків, отримання послуг банку. Зберігає базову інформацію про клієнта, тип заявки, статус обробки та пріоритет.

**Кількість колонок:** 11

### Колонки

| # | Назва | Тип даних | Довжина | Nullable | Default |
|---|-------|-----------|---------|----------|----------|
| 1 | `id` | uuid | - | ✗ | gen_random_uuid() |
| 2 | `type_id` | uuid | - | ✗ | - |
| 3 | `status_id` | uuid | - | ✗ | - |
| 4 | `priority_id` | uuid | - | ✗ | - |
| 5 | `ekb_client_id` | uuid | - | ✓ | - |
| 6 | `client_name` | text | - | ✓ | - |
| 7 | `client_tax_id` | text | - | ✓ | - |
| 8 | `client_phone` | text | - | ✓ | - |
| 9 | `created_at` | timestamp without time zone | - | ✗ | CURRENT_TIMESTAMP |
| 10 | `updated_at` | timestamp without time zone | - | ✗ | CURRENT_TIMESTAMP |
| 11 | `application_number` | integer | - | ✗ | nextval('application_number_seq'::regclass) |

### Обмеження

**Primary Key:**
- `applications_pkey`

**Foreign Keys:**
- `fk_applications_type`
- `fk_applications_status`
- `fk_applications_priority`
- `fk_applications_client`

---

## tbl_test_uyp

**Призначення:** Тестова таблиця для УЮП (управління якістю процесів). Використовується для тестування та налагодження.

**Кількість колонок:** 2

### Колонки

| # | Назва | Тип даних | Довжина | Nullable | Default |
|---|-------|-----------|---------|----------|----------|
| 1 | `id` | integer | - | ✗ | nextval('tbl_test_uyp_id_seq'::regclass) |
| 2 | `name` | text | - | ✓ | - |

### Обмеження

**Primary Key:**
- `tbl_test_uyp_pkey`

---

## application_types

**Призначення:** Довідник типів заявок. Визначає класифікацію заявок (наприклад: відкриття рахунку, кредитування, депозит). Використовується для маршрутизації заявок до відповідних департаментів.

**Кількість колонок:** 6

### Колонки

| # | Назва | Тип даних | Довжина | Nullable | Default |
|---|-------|-----------|---------|----------|----------|
| 1 | `id` | uuid | - | ✗ | gen_random_uuid() |
| 2 | `code` | text | - | ✗ | - |
| 3 | `name` | text | - | ✗ | - |
| 4 | `description` | text | - | ✓ | - |
| 5 | `is_active` | boolean | - | ✗ | true |
| 6 | `sort_order` | integer | - | ✗ | 0 |

### Обмеження

**Primary Key:**
- `application_types_pkey`

**Unique:**
- `application_types_code_key`

---

## application_statuses

**Призначення:** Довідник статусів заявок. Відображає поточний стан обробки заявки (нова, в роботі, схвалена, відхилена, завершена). Містить кольорове кодування для UI та прапорець фінального статусу.

**Кількість колонок:** 7

### Колонки

| # | Назва | Тип даних | Довжина | Nullable | Default |
|---|-------|-----------|---------|----------|----------|
| 1 | `id` | uuid | - | ✗ | gen_random_uuid() |
| 2 | `code` | text | - | ✗ | - |
| 3 | `name` | text | - | ✗ | - |
| 4 | `color` | jsonb | - | ✓ | - |
| 5 | `is_final` | boolean | - | ✗ | false |
| 6 | `is_active` | boolean | - | ✗ | true |
| 7 | `sort_order` | integer | - | ✗ | 0 |

### Обмеження

**Primary Key:**
- `application_statuses_pkey`

**Unique:**
- `application_statuses_code_key`

---

## application_priorities

**Призначення:** Довідник пріоритетів заявок. Визначає терміновість обробки (низький, середній, високий, критичний).

**Кількість колонок:** 5

### Колонки

| # | Назва | Тип даних | Довжина | Nullable | Default |
|---|-------|-----------|---------|----------|----------|
| 1 | `id` | uuid | - | ✗ | gen_random_uuid() |
| 2 | `code` | text | - | ✗ | - |
| 3 | `name` | text | - | ✗ | - |
| 4 | `sort_order` | integer | - | ✗ | 0 |
| 5 | `is_active` | boolean | - | ✗ | true |

### Обмеження

**Primary Key:**
- `application_priorities_pkey`

**Unique:**
- `application_priorities_code_key`

---

## clients

**Призначення:** Базова таблиця клієнтів банку. Містить загальну інформацію про клієнта (фізична/юридична особа), статус клієнта. Використовується як батьківська таблиця для profiles.

**Кількість колонок:** 6

### Колонки

| # | Назва | Тип даних | Довжина | Nullable | Default |
|---|-------|-----------|---------|----------|----------|
| 1 | `id` | uuid | - | ✗ | gen_random_uuid() |
| 2 | `status` | text | - | ✓ | - |
| 3 | `type` | jsonb | - | ✓ | - |
| 4 | `created_at` | timestamp without time zone | - | ✓ | - |
| 5 | `updated_at` | timestamp without time zone | - | ✓ | - |
| 6 | `data` | jsonb | - | ✓ | - |

### Обмеження

**Primary Key:**
- `clients_pkey`

---

## profiles

**Призначення:** Персональні дані фізичних осіб. Зберігає ПІБ, дату народження, стать, сімейний стан, мови спілкування. Пов'язана з clients через client_id.

**Кількість колонок:** 12

### Колонки

| # | Назва | Тип даних | Довжина | Nullable | Default |
|---|-------|-----------|---------|----------|----------|
| 1 | `id` | uuid | - | ✗ | gen_random_uuid() |
| 2 | `client_id` | uuid | - | ✓ | - |
| 3 | `first_name` | text | - | ✓ | - |
| 4 | `middle_name` | text | - | ✓ | - |
| 5 | `last_name` | text | - | ✓ | - |
| 6 | `previous_last_name` | text | - | ✓ | - |
| 7 | `birth` | jsonb | - | ✓ | - |
| 8 | `gender` | text | - | ✓ | - |
| 9 | `languages` | jsonb | - | ✓ | - |
| 10 | `marital_status` | jsonb | - | ✓ | - |
| 11 | `created_at` | timestamp without time zone | - | ✓ | - |
| 12 | `updated_at` | timestamp without time zone | - | ✓ | - |

### Обмеження

**Primary Key:**
- `profiles_pkey`

**Foreign Keys:**
- `profiles_client_id_fkey`

---

## contacts

**Призначення:** Контактна інформація клієнтів. Зберігає телефони, email, месенджери та інші канали зв'язку. Підтримує верифікацію контактів та позначення основного способу зв'язку.

**Кількість колонок:** 15

### Колонки

| # | Назва | Тип даних | Довжина | Nullable | Default |
|---|-------|-----------|---------|----------|----------|
| 1 | `id` | uuid | - | ✗ | gen_random_uuid() |
| 2 | `company_client_id` | uuid | - | ✓ | - |
| 3 | `type` | text | - | ✓ | - |
| 4 | `subtype` | text | - | ✓ | - |
| 5 | `status` | text | - | ✓ | - |
| 6 | `name` | text | - | ✓ | - |
| 7 | `description` | text | - | ✓ | - |
| 8 | `value` | text | - | ✓ | - |
| 9 | `is_primary` | boolean | - | ✓ | - |
| 10 | `is_verified` | boolean | - | ✓ | - |
| 11 | `flags` | jsonb | - | ✓ | - |
| 12 | `created_at` | timestamp without time zone | - | ✓ | - |
| 13 | `updated_at` | timestamp without time zone | - | ✓ | - |
| 14 | `verified_at` | timestamp without time zone | - | ✓ | - |
| 15 | `external_type_id` | text | - | ✓ | - |

### Обмеження

**Primary Key:**
- `contacts_pkey`

**Foreign Keys:**
- `contacts_company_client_id_fkey`

---

## companies

**Призначення:** Довідник компаній-партнерів банку або клієнтів-юридичних осіб.

**Кількість колонок:** 5

### Колонки

| # | Назва | Тип даних | Довжина | Nullable | Default |
|---|-------|-----------|---------|----------|----------|
| 1 | `id` | uuid | - | ✗ | gen_random_uuid() |
| 2 | `name` | text | - | ✗ | - |
| 3 | `status` | text | - | ✓ | - |
| 4 | `created_at` | timestamp without time zone | - | ✓ | - |
| 5 | `updated_at` | timestamp without time zone | - | ✓ | - |

### Обмеження

**Primary Key:**
- `companies_pkey`

---

## company_clients

**Призначення:** Зв'язок між компаніями та клієнтами. Реалізує відношення багато-до-багатьох. Зберігає external_id для синхронізації з зовнішніми системами.

**Кількість колонок:** 8

### Колонки

| # | Назва | Тип даних | Довжина | Nullable | Default |
|---|-------|-----------|---------|----------|----------|
| 1 | `id` | uuid | - | ✗ | gen_random_uuid() |
| 2 | `company_id` | uuid | - | ✓ | - |
| 3 | `client_id` | uuid | - | ✓ | - |
| 4 | `external_id` | text | - | ✓ | - |
| 5 | `status` | text | - | ✓ | - |
| 6 | `data` | jsonb | - | ✓ | - |
| 7 | `created_at` | timestamp without time zone | - | ✓ | - |
| 8 | `updated_at` | timestamp without time zone | - | ✓ | - |

### Обмеження

**Primary Key:**
- `company_clients_pkey`

**Unique:**
- `company_clients_link`

**Foreign Keys:**
- `company_clients_company_id_fkey`
- `company_clients_client_id_fkey`

---

## addresses

**Призначення:** Адреси клієнтів. Зберігає юридичні та фактичні адреси у структурованому вигляді (країна, регіон, місто, вулиця, будинок). Підтримує геолокацію та валідацію адрес.

**Кількість колонок:** 1

### Колонки

| # | Назва | Тип даних | Довжина | Nullable | Default |
|---|-------|-----------|---------|----------|----------|
| 1 | `id` | uuid | - | ✗ | gen_random_uuid() |

### Обмеження

**Primary Key:**
- `addresses_pkey`

---

## documents

**Призначення:** Документи клієнтів. Зберігає паспорти, ID-картки, довідки, договори. Містить інформацію про підпис, канал отримання документа, статус верифікації.

**Кількість колонок:** 1

### Колонки

| # | Назва | Тип даних | Довжина | Nullable | Default |
|---|-------|-----------|---------|----------|----------|
| 1 | `id` | uuid | - | ✗ | gen_random_uuid() |

### Обмеження

**Primary Key:**
- `documents_pkey`

---

## document_files

**Призначення:** Файли документів. Зберігає скановані копії, фото документів. Містить метадані файлу (розмір, MIME-тип, хеш для перевірки цілісності).

**Кількість колонок:** 1

### Колонки

| # | Назва | Тип даних | Довжина | Nullable | Default |
|---|-------|-----------|---------|----------|----------|
| 1 | `id` | uuid | - | ✗ | gen_random_uuid() |

### Обмеження

**Primary Key:**
- `document_files_pkey`

---

## contragent

**Призначення:** Таблиця контрагентів зі старої банківської системи. Містить історичні дані, використовується при міграції.

**Кількість колонок:** 1

### Колонки

| # | Назва | Тип даних | Довжина | Nullable | Default |
|---|-------|-----------|---------|----------|----------|
| 1 | `contragentid` | integer | - | ✗ | - |
---

## order_decisions

**Призначення:** Довідник можливих рішень по завданнях. Визначає які дії може виконати співробітник (схвалити, відхилити, запросити додаткові документи, створити наступне завдання).

**Кількість колонок:** 1

### Колонки

| # | Назва | Тип даних | Довжина | Nullable | Default |
|---|-------|-----------|---------|----------|----------|
| 1 | `id` | uuid | - | ✗ | gen_random_uuid() |

### Обмеження

**Primary Key:**
- `order_decisions_pkey`

---

## events

**Призначення:** Журнал подій та повідомлень. Фіксує всі важливі події у системі (дзвінки, листи, зміни статусів). Використовується для відображення timeline клієнта та налаштування нотифікацій.

**Кількість колонок:** 1

### Колонки

| # | Назва | Тип даних | Довжина | Nullable | Default |
|---|-------|-----------|---------|----------|----------|
| 1 | `id` | uuid | - | ✗ | gen_random_uuid() |

### Обмеження

**Primary Key:**
- `events_pkey`

---

## relationships

**Призначення:** Зв'язки між сутностями. Зберігає відношення між клієнтами (родинні зв'язки, бенефіціари, довірені особи). Містить період дії зв'язку.

**Кількість колонок:** 1

### Колонки

| # | Назва | Тип даних | Довжина | Nullable | Default |
|---|-------|-----------|---------|----------|----------|
| 1 | `id` | uuid | - | ✗ | gen_random_uuid() |

### Обмеження

**Primary Key:**
- `relationships_pkey`

---

## audit_log

**Призначення:** Журнал аудиту системи. Фіксує всі зміни даних: хто, коли, що змінив. Зберігає старе та нове значення полів. Використовується для відповідності вимогам НБУ та внутрішнього контролю.

**Кількість колонок:** 1

### Колонки

| # | Назва | Тип даних | Довжина | Nullable | Default |
|---|-------|-----------|---------|----------|----------|
| 1 | `id` | uuid | - | ✗ | gen_random_uuid() |

### Обмеження

**Primary Key:**
- `audit_log_pkey`

---

## skill_groups

**Призначення:** Групи навичок / департаменти. Визначає підрозділи банку (контакт-центр, фінмоніторинг, юридичний відділ, андерайтинг). Використовується для маршрутизації завдань.

**Кількість колонок:** 1

### Колонки

| # | Назва | Тип даних | Довжина | Nullable | Default |
|---|-------|-----------|---------|----------|----------|
| 1 | `id` | uuid | - | ✗ | gen_random_uuid() |

### Обмеження

**Primary Key:**
- `skill_groups_pkey`

---

## orders

**Призначення:** Робочі завдання для співробітників. Створюються автоматично або вручну на основі заявок. Містять призначення виконавцю, рішення, коментарі. Зберігають історію обробки заявки.

**Кількість колонок:** 1

### Колонки

| # | Назва | Тип даних | Довжина | Nullable | Default |
|---|-------|-----------|---------|----------|----------|
| 1 | `id` | uuid | - | ✗ | gen_random_uuid() |

### Обмеження

**Primary Key:**
- `orders_pkey`

---

## action_types

**Призначення:** Типи дій для завдань. Визначає конкретні операції, які виконує співробітник (перевірка документів, дзвінок клієнту, KYC-перевірка).

**Кількість колонок:** 1

### Колонки

| # | Назва | Тип даних | Довжина | Nullable | Default |
|---|-------|-----------|---------|----------|----------|
| 1 | `id` | uuid | - | ✗ | gen_random_uuid() |

### Обмеження

**Primary Key:**
- `action_types_pkey`

---

## default_first_actions

**Призначення:** Дефолтні перші дії для типів заявок. Визначає який тип завдання автоматично створюється при надходженні заявки певного типу.

**Кількість колонок:** 1

### Колонки

| # | Назва | Тип даних | Довжина | Nullable | Default |
|---|-------|-----------|---------|----------|----------|
| 1 | `type_id` | uuid | - | ✗ | - |

### Обмеження

**Primary Key:**
- `default_first_actions_pkey`

---

