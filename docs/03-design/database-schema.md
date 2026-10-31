# طرح پایگاه داده — دادرس

| | |
|---|---|
| نام پروژه | دادرس (Dadress) |
| نسخهٔ سند | ۱.۱ |
| تاریخ | ۱۴۰۵ |
| تهیه‌کننده | سینا نوروزی، احسان رستم خانی، محمد مانی امینی، سالار هادی، سعید بابائی |
| وضعیت | پیش‌نویس |

---

## ۱. مقدمه

این سند ساختار پایگاه داده (PostgreSQL) سامانهٔ دادرس را تعریف می‌کند.
نام جدول‌ها و ستون‌ها انگلیسی و به‌صورت snake_case است.

---

## ۲. ERD

`mermaid
erDiagram
    users ||--o| lawyer_profiles : has
    lawyer_profiles ||--o{ verification_documents : uploads
    lawyer_profiles ||--o{ lawyer_specialties : has
    specialties ||--o{ lawyer_specialties : classifies
    users ||--o{ legal_questions : asks
    specialties ||--o{ legal_questions : about
    legal_questions ||--o{ answers : has
    lawyer_profiles ||--o{ answers : writes
    users ||--o{ case_requests : sends
    lawyer_profiles ||--o{ case_requests : receives
    users ||--o{ reviews : writes
    lawyer_profiles ||--o{ reviews : receives
    users ||--o{ reports : files
    lawyer_profiles ||--o{ availability_slots : offers
    availability_slots ||--o| appointments : bookedAs
    users ||--o{ appointments : books
    lawyer_profiles ||--o{ appointments : receives
    users ||--o{ payments : pays
    appointments ||--o{ payments : paidBy
    case_requests ||--o{ payments : paidBy
    subscriptions ||--o{ payments : paidBy
    lawyer_profiles ||--o{ subscriptions : buys
    case_requests ||--o{ case_documents : has
    case_requests ||--o{ messages : has
    case_requests ||--o{ case_events : tracks
    users ||--o{ case_documents : uploads
    users ||--o{ messages : sends

    users {
        int id PK
        string full_name
        string email UK
        string phone UK
        string password_hash
        string role
        datetime created_at
    }
    lawyer_profiles {
        int id PK
        int user_id FK
        string license_number UK
        text bio
        string city
        int experience_years
        string verification_status
        float average_rating
        int review_count
    }
    verification_documents {
        int id PK
        int lawyer_id FK
        string file_path
        datetime uploaded_at
    }
    specialties {
        int id PK
        string name UK
    }
    lawyer_specialties {
        int lawyer_id FK
        int specialty_id FK
    }
    legal_questions {
        int id PK
        int client_id FK
        int specialty_id FK
        string title
        text body
        string status
        datetime created_at
    }
    answers {
        int id PK
        int question_id FK
        int lawyer_id FK
        text body
        datetime created_at
    }
    case_requests {
        int id PK
        int client_id FK
        int lawyer_id FK
        string title
        text description
        string status
        datetime created_at
    }
    reviews {
        int id PK
        int client_id FK
        int lawyer_id FK
        int question_id FK
        int case_request_id FK
        int rating
        text comment
        boolean is_visible
        datetime created_at
    }
    availability_slots {
        int id PK
        int lawyer_id FK
        datetime start_time
        datetime end_time
        string consult_type
        boolean is_booked
    }
    appointments {
        int id PK
        int client_id FK
        int lawyer_id FK
        int slot_id FK
        string consult_type
        string status
        datetime created_at
    }
    payments {
        int id PK
        int user_id FK
        int appointment_id FK
        int case_request_id FK
        int subscription_id FK
        string purpose
        decimal amount
        decimal commission_amount
        string status
        string gateway_ref
        datetime created_at
    }
    subscriptions {

int id PK
        int lawyer_id FK
        string plan
        date start_date
        date end_date
        boolean is_active
    }
    case_documents {
        int id PK
        int case_request_id FK
        int uploader_id FK
        string file_name
        string file_path
        datetime uploaded_at
    }
    messages {
        int id PK
        int case_request_id FK
        int sender_id FK
        text body
        boolean is_read
        datetime sent_at
    }
    case_events {
        int id PK
        int case_request_id FK
        string status
        text note
        datetime created_at
    }
    reports {
        int id PK
        int reporter_id FK
        string target_type
        int target_id
        text reason
        string status
        datetime created_at
    }
`

---

## ۳. جدول‌ها

| جدول | توضیح |
|------|-------|
| users | کاربران (موکل، وکیل، مدیر) |
| lawyer_profiles | پروفایل حرفه‌ای وکلا |
| verification_documents | مدارک بارگذاری‌شدهٔ وکلا |
| specialties | تخصص‌ها |
| lawyer_specialties | رابطهٔ چندبه‌چند وکیل و تخصص |
| legal_questions | سؤال‌های حقوقی |
| answers | پاسخ وکلا |
| case_requests | درخواست‌های پرونده |
| reviews | نمره و نظرات |
| reports | گزارش‌های کاربران |
| availability_slots | بازه‌های آزاد تقویم وکیل |
| appointments | نوبت‌های رزروشده |
| payments | تراکنش‌ها و کمیسیون |
| subscriptions | اشتراک ویژهٔ وکلا |
| case_documents | مدارک پرونده |
| messages | پیام‌های چت پرونده |
| case_events | تاریخچهٔ پیشرفت پرونده |

---

## ۴. جزئیات جدول‌های مهم

### ۴.۱ users

| ستون | نوع | قید |
|------|-----|-----|
| id | INT | PK، Auto |
| full_name | VARCHAR(100) | NOT NULL |
| email | VARCHAR(150) | UNIQUE |
| phone | VARCHAR(15) | UNIQUE |
| password_hash | VARCHAR(255) | NOT NULL |
| role | VARCHAR(10) | CLIENT / LAWYER / ADMIN |
| created_at | TIMESTAMP | DEFAULT now() |

### ۴.۲ lawyer_profiles

| ستون | نوع | قید |
|------|-----|-----|
| id | INT | PK |
| user_id | INT | FK → users، UNIQUE |
| license_number | VARCHAR(30) | UNIQUE |
| bio | TEXT | |
| city | VARCHAR(50) | |
| experience_years | INT | ≥ 0 |
| verification_status | VARCHAR(10) | PENDING / VERIFIED / REJECTED |
| average_rating | NUMERIC(2,1) | DEFAULT 0 |
| review_count | INT | DEFAULT 0 |

### ۴.۳ reviews

| ستون | نوع | قید |
|------|-----|-----|
| id | INT | PK |
| client_id | INT | FK → users |
| lawyer_id | INT | FK → lawyer_profiles |
| question_id | INT | FK → legal_questions، می‌تواند خالی باشد |
| case_request_id | INT | FK → case_requests، می‌تواند خالی باشد |
| rating | INT | بین ۱ تا ۵ (CHECK) |
| comment | TEXT | اختیاری |
| is_visible | BOOLEAN | DEFAULT true |
| created_at | TIMESTAMP | DEFAULT now() |

قانون: حداقل یکی از `question_id` یا `case_request_id` باید مقدار داشته باشد (NFR-07).

### ۴.۴ payments

| ستون | نوع | قید |
|------|-----|-----|
| id | INT | PK |
| user_id | INT | FK → users |
| appointment_id | INT | FK، اختیاری |
| case_request_id | INT | FK، اختیاری |
| subscription_id | INT | FK، اختیاری |
| purpose | VARCHAR(15) | CONSULT / CASE_FEE / SUBSCRIPTION |
| amount | NUMERIC(12,0) | > 0 |
| commission_amount | NUMERIC(12,0) | سهم پلتفرم |
| status | VARCHAR(10) | PENDING / PAID / FAILED / REFUNDED |
| gateway_ref | VARCHAR(60) | کد پیگیری درگاه |

قانون: دقیقاً یکی از `appointment_id`، `case_request_id` یا `subscription_id` باید مقدار داشته باشد.
اطلاعات کارت هرگز ذخیره نمی‌شود (NFR-09).

### ۴.۵ case_requests (وضعیت‌ها)

| وضعیت | معنی |
|-------|------|
| PENDING | در انتظار بررسی وکیل |
| ACCEPTED | پذیرفته‌شده |
| REJECTED | ردشده |
| IN_REVIEW | در حال بررسی |
| COURT_SESSION | جلسهٔ دادگاه |
| CLOSED | مختومه |

تاریخچهٔ تغییر وضعیت در `case_events` نگهداری می‌شود.

---

## ۵. شاخص‌ها (Index)

| جدول | ستون | دلیل |
|------|------|------|
| lawyer_profiles | city، verification_status | سرعت جست‌وجو (NFR-04) |

| lawyer_specialties | specialty_id | فیلتر بر اساس تخصص |
| reviews | lawyer_id | نمایش نظرات پروفایل |
| legal_questions | specialty_id، status | نمایش سؤال‌ها به وکلای مرتبط |
| availability_slots | lawyer_id، start_time، is_booked | نمایش بازه‌های آزاد |
| messages | case_request_id، sent_at | بارگذاری تاریخچهٔ چت |
| payments | status، created_at | گزارش مالی مدیر (US-20) |

---

## ۶. نرمال‌سازی

- رابطهٔ وکیل و تخصص با جدول واسط lawyer_specialties جدا شده است (۳NF)
- میانگین نمره (average_rating) عمداً در lawyer_profiles ذخیره می‌شود تا هر بار محاسبه نشود (Denormalization برای کارایی)

---

## پیوست الف — تاریخچهٔ تغییرات

| نسخه | تاریخ | تغییرات | تهیه‌کننده |
|-------|-------|----------|--------------|
| ۱.۰ | ۱۴۰۵ | نسخهٔ اولیه | سینا نوروزی، احسان رستم خانی، محمد مانی امینی، سالار هادی |
| ۱.۱ | ۱۴۰۵ | افزودن نوبت، پرداخت، اشتراک، مدارک، چت و پیشرفت پرونده | سینا نوروزی، احسان رستم خانی، محمد مانی امینی، سالار هادی |
