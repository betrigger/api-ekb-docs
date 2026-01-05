# Документація структури бази даних

**Дата генерації:** 2025-01-05  
**СУБД:** PostgreSQL  
**Загальна кількість таблиць:** 22

---

## Зміст

1. [applications](#applications)
2. [tbl_test_uyp](#tbl-test-uyp)
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
14. [contragent](#contragent)
15. [order_decisions](#order-decisions)
16. [events](#events)
17. [relationships](#relationships)
18. [audit_log](#audit-log)
19. [skill_groups](#skill-groups)
20. [orders](#orders)
21. [action_types](#action-types)
22. [default_first_actions](#default-first-actions)

---

## applications

**Призначення:** Основна таблиця заявок клієнтів на банківські послуги. 

**Бізнес-процес:**
• Клієнт подає заявку через різні канали (онлайн, відділення, контакт-центр)
• Система автоматично створює перше завдання (order) для відповідного департаменту
• Заявка має тип (відкриття рахунку, кредит, депозит), статус та пріоритет
• Прив'язується до клієнта з EKB через ekb_client_id
• Має унікальний application_number для відстеження

**API ендпоінти:** POST /applications, GET /applications, GET /applications/stats

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

**Check Constraints:** 7

---

## tbl_test_uyp

**Призначення:** Тестова таблиця для УЮП (управління якістю процесів). Тестування, перевірка прав доступу.

**Кількість колонок:** 2

### Колонки

| # | Назва | Тип даних | Довжина | Nullable | Default |
|---|-------|-----------|---------|----------|----------|
| 1 | `id` | integer | - | ✗ | nextval('tbl_test_uyp_id_seq'::regclass) |
| 2 | `name` | text | - | ✓ | - |

### Обмеження

**Primary Key:**
- `tbl_test_uyp_pkey`

**Check Constraints:** 1

---

## application_types

**Призначення:** Довідник типів заявок. Визначає класифікацію та маршрутизацію заявок.

**Типи:** Відкриття поточного рахунку, депозиту, кредиту, випуск карток, закриття рахунку.

Кожен type має default_first_action - який департамент отримує першу заявку.

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

**Check Constraints:** 5

---

## application_statuses

**Призначення:** Довідник статусів заявок. Життєвий цикл: Нова → В роботі → На перевірці → Очікування клієнта → Схвалена/Відхилена.

**Поля:** color (jsonb) для UI, is_final (чи кінцевий статус).

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

**Check Constraints:** 6

---

## application_priorities

**Призначення:** Довідник пріоритетів заявок. Низький (5-7 днів), Середній (2-3 дні), Високий (1 день), Критичний (день звернення).

Впливає на SLA та порядок в черзі завдань.

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

**Check Constraints:** 5

---

## clients

**Призначення:** Базова таблиця клієнтів банку. 

**Типи (type - jsonb):** individual, entrepreneur, legal

**Статуси:** active, blocked, closed

Батьківська таблиця для profiles. Пов'язана з companies через company_clients.

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

**Check Constraints:** 1

---

## profiles

**Призначення:** Персональні дані фізичних осіб. ПІБ, дата народження, стать, мови спілкування, сімейний стан.

**API:** Пошук за ПІБ (full_name, name_words), датою народження.

**Зв'язок:** client_id → clients.id (один клієнт - один профіль)

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

**Check Constraints:** 1

---

## contacts

**Призначення:** Контактна інформація клієнтів.

**Типи:** phone (mobile/home/work), email, messenger (telegram/viber), social.

**Важливі поля:** is_primary (основний), is_verified (підтверджено), verified_at, flags (do_not_call, marketing_allowed).

Використовується для пошуку клієнтів, верифікації, нотифікацій.

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

**Check Constraints:** 1

---

## companies

**Призначення:** Довідник компаній-партнерів або клієнтів-юрособ. Реквізити, ЄДРПОУ, роботодавці клієнтів.

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

**Check Constraints:** 2

---

## company_clients

**Призначення:** Зв'язок між компаніями та клієнтами (багато-до-багатьох).

**Сценарії:** Співробітники компанії, бенефіціари, керівники, відносини з роботодавцем.

**Поля:** external_id (ID з ЄКБ/1C), data (роль, посада, частка володіння).

Всі документи, адреси, контакти прив'язані до company_client_id.

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

**Check Constraints:** 1

---

## addresses

**Призначення:** Адреси клієнтів зі структурованим зберіганням.

**Типи:** registration, residence, work, mailing.

**Структура:** country/region/city/district/street/building/apartment (всі jsonb).

**Додатково:** location (геолокація), valid (період актуальності), used_for (delivery/correspondence).

**Кількість колонок:** 23

### Колонки

| # | Назва | Тип даних | Довжина | Nullable | Default |
|---|-------|-----------|---------|----------|----------|
| 1 | `id` | uuid | - | ✗ | gen_random_uuid() |
| 2 | `company_client_id` | uuid | - | ✓ | - |
| 3 | `type` | text | - | ✓ | - |
| 4 | `is_primary` | boolean | - | ✓ | - |
| 5 | `used_for` | jsonb | - | ✓ | - |
| 6 | `status` | text | - | ✓ | - |
| 7 | `description` | text | - | ✓ | - |
| 8 | `source` | text | - | ✓ | - |
| 9 | `country` | jsonb | - | ✓ | - |
| 10 | `region` | jsonb | - | ✓ | - |
| 11 | `city` | jsonb | - | ✓ | - |
| 12 | `district` | jsonb | - | ✓ | - |
| 13 | `street` | jsonb | - | ✓ | - |
| 14 | `building` | jsonb | - | ✓ | - |
| 15 | `apartment` | text | - | ✓ | - |
| 16 | `postal_code` | text | - | ✓ | - |
| 17 | `location` | jsonb | - | ✓ | - |
| 18 | `valid` | jsonb | - | ✓ | - |
| 19 | `data` | jsonb | - | ✓ | - |
| 20 | `created_at` | timestamp without time zone | - | ✓ | - |
| 21 | `updated_at` | timestamp without time zone | - | ✓ | - |
| 22 | `verified_at` | timestamp without time zone | - | ✓ | - |
| 23 | `external_type_id` | character varying | 10 | ✓ | - |

### Обмеження

**Primary Key:**
- `addresses_pkey`

**Foreign Keys:**
- `addresses_company_client_id_fkey`

**Check Constraints:** 1

---

## documents

**Призначення:** Реєстр документів клієнтів.

**Типи:** passport, id_card, inn, foreign_passport, contract, application_form.

**Статуси:** new → verification → verified/rejected, expired.

**Поля:** channel (mobile_app/branch/email), signature (електронні підписи), data (серія, номер, дати).

**Кількість колонок:** 13

### Колонки

| # | Назва | Тип даних | Довжина | Nullable | Default |
|---|-------|-----------|---------|----------|----------|
| 1 | `id` | uuid | - | ✗ | gen_random_uuid() |
| 2 | `company_client_id` | uuid | - | ✓ | - |
| 3 | `type` | text | - | ✗ | - |
| 4 | `status` | text | - | ✓ | - |
| 5 | `channel` | text | - | ✓ | - |
| 6 | `source` | text | - | ✓ | - |
| 7 | `owner` | jsonb | - | ✓ | - |
| 8 | `signature` | jsonb | - | ✓ | - |
| 9 | `data` | jsonb | - | ✓ | - |
| 10 | `is_primary` | boolean | - | ✓ | false |
| 11 | `created_at` | timestamp without time zone | - | ✓ | - |
| 12 | `updated_at` | timestamp without time zone | - | ✓ | - |
| 13 | `verified_at` | timestamp without time zone | - | ✓ | - |

### Обмеження

**Primary Key:**
- `documents_pkey`

**Foreign Keys:**
- `documents_company_client_id_fkey`

**Check Constraints:** 2

---

## document_files

**Призначення:** Файли документів. Метадані та посилання на storage (S3/MinIO).

**API:** POST /documents/{documentId}/files, GET /documents/{documentId}/files

**Required:** name, size, mime_type, storage_url. Hash для перевірки цілісності.

**Кількість колонок:** 8

### Колонки

| # | Назва | Тип даних | Довжина | Nullable | Default |
|---|-------|-----------|---------|----------|----------|
| 1 | `id` | uuid | - | ✗ | gen_random_uuid() |
| 2 | `document_id` | uuid | - | ✓ | - |
| 3 | `name` | text | - | ✗ | - |
| 4 | `size` | bigint | - | ✓ | - |
| 5 | `mime_type` | text | - | ✓ | - |
| 6 | `storage_url` | text | - | ✓ | - |
| 7 | `hash` | jsonb | - | ✓ | - |
| 8 | `created_at` | timestamp without time zone | - | ✓ | now() |

### Обмеження

**Primary Key:**
- `document_files_pkey`

**Foreign Keys:**
- `document_files_document_id_fkey`

**Check Constraints:** 2

---

## contragent

**Призначення:** Контрагенти зі старої Agile Bank. Історичні дані для перегляду та міграції в clients/profiles.

**Кількість колонок:** 4

### Колонки

| # | Назва | Тип даних | Довжина | Nullable | Default |
|---|-------|-----------|---------|----------|----------|
| 1 | `contragentid` | integer | - | ✗ | - |
| 2 | `contragentsiteid` | integer | - | ✗ | - |
| 3 | `contragentsname` | character varying | 38 | ✗ | - |
| 4 | `contragenttypeid` | character | 1 | ✗ | - |

### Обмеження

**Check Constraints:** 4

---

## order_decisions

**Призначення:** Довідник рішень по завданнях. Визначає наступні кроки.

**Рішення:** approve, reject, request_documents, escalate, send_to_legal, send_to_finmon, verify.

**Логіка:** Якщо finalizes_application=true → завершує заявку. Якщо creates_order_for → створює новий order.

**API:** GET /order_decisions/{id}

**Кількість колонок:** 10

### Колонки

| # | Назва | Тип даних | Довжина | Nullable | Default |
|---|-------|-----------|---------|----------|----------|
| 1 | `id` | uuid | - | ✗ | gen_random_uuid() |
| 2 | `skill_group_id` | uuid | - | ✗ | - |
| 3 | `code` | text | - | ✗ | - |
| 4 | `name` | text | - | ✗ | - |
| 5 | `creates_order_for` | uuid | - | ✓ | - |
| 6 | `finalizes_application` | boolean | - | ✗ | false |
| 7 | `is_rejection` | boolean | - | ✗ | false |
| 8 | `is_active` | boolean | - | ✗ | true |
| 9 | `sort_order` | integer | - | ✗ | 0 |
| 10 | `creates_with_action_type_id` | uuid | - | ✓ | - |

### Обмеження

**Primary Key:**
- `order_decisions_pkey`

**Foreign Keys:**
- `fk_order_decisions_skill_group`
- `fk_order_decisions_creates_for`
- `fk_order_decisions_action_type`

**Check Constraints:** 8

---

## events

**Призначення:** Журнал подій та комунікацій. Timeline всіх взаємодій.

**Типи:** call, email, sms, meeting, status_change, document_uploaded.

**Напрямок:** incoming (від клієнта), outgoing (до клієнта).

Використовується для timeline клієнта, історії дзвінків, нотифікацій.

**Кількість колонок:** 14

### Колонки

| # | Назва | Тип даних | Довжина | Nullable | Default |
|---|-------|-----------|---------|----------|----------|
| 1 | `id` | uuid | - | ✗ | gen_random_uuid() |
| 2 | `company_client_id` | uuid | - | ✓ | - |
| 3 | `external_id` | text | - | ✓ | - |
| 4 | `direction` | text | - | ✓ | - |
| 5 | `type` | jsonb | - | ✓ | - |
| 6 | `category` | text | - | ✓ | - |
| 7 | `status` | text | - | ✓ | - |
| 8 | `description` | text | - | ✓ | - |
| 9 | `channel` | jsonb | - | ✓ | - |
| 10 | `data` | jsonb | - | ✓ | - |
| 11 | `created_at` | timestamp without time zone | - | ✓ | - |
| 12 | `updated_at` | timestamp without time zone | - | ✓ | - |
| 13 | `notification_settings` | jsonb | - | ✓ | '{}'::jsonb |
| 14 | `read` | boolean | - | ✓ | false |

### Обмеження

**Primary Key:**
- `events_pkey`

**Foreign Keys:**
- `events_company_client_id_fkey`

**Check Constraints:** 1

---

## relationships

**Призначення:** Зв'язки між клієнтами.

**Типи:** family (spouse/child), business (partner/beneficiary), reference, trust (опікун).

**Використання:** KYC/AML перевірка, бенефіціарні власники, сімейні зв'язки для кредитування.

**Кількість колонок:** 8

### Колонки

| # | Назва | Тип даних | Довжина | Nullable | Default |
|---|-------|-----------|---------|----------|----------|
| 1 | `id` | uuid | - | ✗ | gen_random_uuid() |
| 2 | `company_client_id` | uuid | - | ✓ | - |
| 3 | `status` | text | - | ✓ | - |
| 4 | `related_entity` | jsonb | - | ✓ | - |
| 5 | `relation_type` | jsonb | - | ✓ | - |
| 6 | `valid` | jsonb | - | ✓ | - |
| 7 | `created_at` | timestamp without time zone | - | ✓ | - |
| 8 | `updated_at` | timestamp without time zone | - | ✓ | - |

### Обмеження

**Primary Key:**
- `relationships_pkey`

**Foreign Keys:**
- `relationships_company_client_id_fkey`

**Check Constraints:** 1

---

## audit_log

**Призначення:** Журнал аудиту всіх змін. Відповідність НБУ та SOX.

**Фіксує:** Хто (actor), коли (timestamp), що змінив (table/field), старе/нове значення, звідки (IP/user_agent).

Використання: розслідування інцидентів, регуляторні вимоги, відновлення даних.

**Кількість колонок:** 17

### Колонки

| # | Назва | Тип даних | Довжина | Nullable | Default |
|---|-------|-----------|---------|----------|----------|
| 1 | `id` | uuid | - | ✗ | gen_random_uuid() |
| 2 | `client_id` | uuid | - | ✓ | - |
| 3 | `company_client_id` | uuid | - | ✓ | - |
| 4 | `actor_id` | text | - | ✓ | - |
| 5 | `actor_type` | text | - | ✓ | - |
| 6 | `action` | text | - | ✓ | - |
| 7 | `target_table` | text | - | ✓ | - |
| 8 | `field_name` | text | - | ✓ | - |
| 9 | `old_value` | text | - | ✓ | - |
| 10 | `new_value` | text | - | ✓ | - |
| 11 | `data` | jsonb | - | ✓ | - |
| 12 | `source_ip` | text | - | ✓ | - |
| 13 | `user_agent` | text | - | ✓ | - |
| 14 | `timestamp` | timestamp without time zone | - | ✓ | - |
| 15 | `comment` | text | - | ✓ | - |
| 16 | `transaction_id` | uuid | - | ✓ | - |
| 17 | `severity` | text | - | ✓ | - |

### Обмеження

**Primary Key:**
- `audit_log_pkey`

**Foreign Keys:**
- `audit_log_client_id_fkey`
- `audit_log_company_client_id_fkey`

**Check Constraints:** 1

---

## skill_groups

**Призначення:** Департаменти банку для маршрутизації завдань.

**Департаменти:** contact_center, financial_monitoring, legal, underwriting, operations, back_office.

**API:** Фільтрація заявок за assigned_to_group, статистика по департаментах.

**Кількість колонок:** 5

### Колонки

| # | Назва | Тип даних | Довжина | Nullable | Default |
|---|-------|-----------|---------|----------|----------|
| 1 | `id` | uuid | - | ✗ | gen_random_uuid() |
| 2 | `code` | text | - | ✗ | - |
| 3 | `name` | text | - | ✗ | - |
| 4 | `is_active` | boolean | - | ✗ | true |
| 5 | `sort_order` | integer | - | ✗ | 0 |

### Обмеження

**Primary Key:**
- `skill_groups_pkey`

**Unique:**
- `skill_groups_code_key`

**Check Constraints:** 5

---

## orders

**Призначення:** Робочі завдання для співробітників. Workflow обробки заявок.

**Lifecycle:** Створення → Призначення департаменту/співробітнику → Виконання → Вибір рішення → Завершення.

**Статуси:** pending, in_progress, completed, cancelled.

**API:** GET /orders (список завдань), POST /orders (створення), фільтри по application_id, skill_group_id, assigned_to.

**Кількість колонок:** 11

### Колонки

| # | Назва | Тип даних | Довжина | Nullable | Default |
|---|-------|-----------|---------|----------|----------|
| 1 | `id` | uuid | - | ✗ | gen_random_uuid() |
| 2 | `application_id` | uuid | - | ✗ | - |
| 3 | `skill_group_id` | uuid | - | ✗ | - |
| 4 | `decision_id` | uuid | - | ✓ | - |
| 5 | `assigned_to` | uuid | - | ✓ | - |
| 6 | `assigned_to_name` | text | - | ✓ | - |
| 7 | `status` | text | - | ✗ | 'pending'::text |
| 8 | `comment` | text | - | ✓ | - |
| 9 | `created_at` | timestamp without time zone | - | ✗ | CURRENT_TIMESTAMP |
| 10 | `completed_at` | timestamp without time zone | - | ✓ | - |
| 11 | `action_type_id` | uuid | - | ✓ | - |

### Обмеження

**Primary Key:**
- `orders_pkey`

**Foreign Keys:**
- `fk_orders_application`
- `fk_orders_skill_group`
- `fk_orders_decision`
- `fk_orders_action_type`

**Check Constraints:** 6

---

## action_types

**Призначення:** Типи дій для завдань.

**Дії:** call_client, verify_documents, kyc_check, aml_screening, credit_scoring, open_account, issue_card.

**API:** GET /action_types?type_id=...&skill_group_id=... (доступні дії)

**Кількість колонок:** 8

### Колонки

| # | Назва | Тип даних | Довжина | Nullable | Default |
|---|-------|-----------|---------|----------|----------|
| 1 | `id` | uuid | - | ✗ | gen_random_uuid() |
| 2 | `type_id` | uuid | - | ✗ | - |
| 3 | `skill_group_id` | uuid | - | ✗ | - |
| 4 | `code` | text | - | ✗ | - |
| 5 | `name` | text | - | ✗ | - |
| 6 | `description` | text | - | ✓ | - |
| 7 | `is_active` | boolean | - | ✗ | true |
| 8 | `sort_order` | integer | - | ✗ | 0 |

### Обмеження

**Primary Key:**
- `action_types_pkey`

**Unique:**
- `uq_action_types_code`

**Foreign Keys:**
- `fk_action_types_type`
- `fk_action_types_skill_group`

**Check Constraints:** 7

---

## default_first_actions

**Призначення:** Автоматична маршрутизація перших дій.

**Логіка:** Заявка певного типу → автоматично створює перший order з action_type для skill_group.

**Приклад:** "Відкриття рахунку" → "Дзвінок клієнту" → "Контакт-центр"

**Кількість колонок:** 3

### Колонки

| # | Назва | Тип даних | Довжина | Nullable | Default |
|---|-------|-----------|---------|----------|----------|
| 1 | `type_id` | uuid | - | ✗ | - |
| 2 | `action_type_id` | uuid | - | ✗ | - |
| 3 | `skill_group_id` | uuid | - | ✗ | - |

### Обмеження

**Primary Key:**
- `default_first_actions_pkey`

**Foreign Keys:**
- `fk_default_first_actions_type`
- `fk_default_first_actions_action`
- `fk_default_first_actions_skill_group`

**Check Constraints:** 3

---

