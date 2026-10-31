# دیاگرام کلاس — دادرس

| | |
|---|---|
| نام پروژه | دادرس (Dadress) |
| نسخهٔ سند | ۱.۱ |
| تاریخ | ۱۴۰۵ |
| تهیه‌کننده | سینا نوروزی، احسان رستم خانی، محمد مانی امینی، سالار هادی، سعید بابائی |
| وضعیت | پیش‌نویس |

---

## ۱. مقدمه

این سند کلاس‌های اصلی دامنه و رابطهٔ بین آن‌ها را نشان می‌دهد.
این نسخه اولیه است و در Sprintها تکمیل می‌شود.

---

## ۲. دیاگرام کلاس

`mermaid
classDiagram
    class User {
        +int id
        +string fullName
        +string email
        +string phone
        +string passwordHash
        +Role role
        +register()
        +login()
    }

    class LawyerProfile {
        +int id
        +string licenseNumber
        +string bio
        +string city
        +int experienceYears
        +VerificationStatus verificationStatus
        +float averageRating
        +updateProfile()
        +recalculateRating()
    }

    class Specialty {
        +int id
        +string name
    }

    class VerificationDocument {
        +int id
        +string filePath
        +DateTime uploadedAt
    }

    class LegalQuestion {
        +int id
        +string title
        +string body
        +QuestionStatus status
        +DateTime createdAt
    }

    class Answer {
        +int id
        +string body
        +DateTime createdAt
    }

    class CaseRequest {
        +int id
        +string title
        +string description
        +RequestStatus status
        +DateTime createdAt
        +accept()
        +reject()
    }

    class Review {
        +int id
        +int rating
        +string comment
        +bool isVisible
        +DateTime createdAt
    }

    class AvailabilitySlot {
        +int id
        +DateTime startTime
        +DateTime endTime
        +ConsultType type
        +bool isBooked
    }

    class Appointment {
        +int id
        +ConsultType type
        +AppointmentStatus status
        +DateTime createdAt
        +cancel()
    }

    class Payment {
        +int id
        +PaymentPurpose purpose
        +decimal amount
        +decimal commissionAmount
        +PaymentStatus status
        +string gatewayRef
        +DateTime createdAt
    }

    class CaseDocument {
        +int id
        +string fileName
        +string filePath
        +DateTime uploadedAt
    }

    class Message {
        +int id
        +string body
        +bool isRead
        +DateTime sentAt
    }

    class CaseEvent {
        +int id
        +CaseStatus status
        +string note
        +DateTime createdAt
    }

    class Subscription {
        +int id
        +string plan
        +Date startDate
        +Date endDate
        +bool isActive
    }

    class Report {
        +int id
        +string targetType
        +int targetId
        +string reason
        +ReportStatus status
    }

    class Role {
        <<enumeration>>
        CLIENT
        LAWYER
        ADMIN
    }

    class VerificationStatus {
        <<enumeration>>
        PENDING
        VERIFIED
        REJECTED
    }

    class RequestStatus {
        <<enumeration>>
        PENDING
        ACCEPTED
        REJECTED
    }

    class CaseStatus {
        <<enumeration>>
        IN_REVIEW
        COURT_SESSION
        CLOSED
    }

    class PaymentStatus {
        <<enumeration>>
        PENDING
        PAID
        FAILED
        REFUNDED
    }

    User "1" --> "0..1" LawyerProfile : has
    User --> Role
    LawyerProfile --> VerificationStatus
    LawyerProfile "1" --> "0..*" VerificationDocument : uploads
    LawyerProfile "0..*" -- "1..*" Specialty : expertIn
    User "1" --> "0..*" LegalQuestion : asks
    LegalQuestion "0..*" --> "1" Specialty : about
    LegalQuestion "1" --> "0..*" Answer : has
    LawyerProfile "1" --> "0..*" Answer : writes
    User "1" --> "0..*" CaseRequest : sends
    CaseRequest "0..*" --> "1" LawyerProfile : sentTo

CaseRequest --> RequestStatus
    User "1" --> "0..*" Review : writes
    LawyerProfile "1" --> "0..*" Review : receives
    User "1" --> "0..*" Report : files
    LawyerProfile "1" --> "0..*" AvailabilitySlot : offers
    AvailabilitySlot "1" --> "0..1" Appointment : bookedAs
    User "1" --> "0..*" Appointment : books
    LawyerProfile "1" --> "0..*" Appointment : receives
    User "1" --> "0..*" Payment : pays
    Appointment "1" --> "0..*" Payment : paidBy
    CaseRequest "1" --> "0..*" Payment : paidBy
    Subscription "1" --> "0..*" Payment : paidBy
    LawyerProfile "1" --> "0..*" Subscription : buys
    CaseRequest "1" --> "0..*" CaseDocument : has
    CaseRequest "1" --> "0..*" Message : has
    CaseRequest "1" --> "0..*" CaseEvent : tracks
    User "1" --> "0..*" CaseDocument : uploads
    User "1" --> "0..*" Message : sends
    CaseEvent --> CaseStatus
    Payment --> PaymentStatus
`

---

## ۳. شرح کلاس‌ها

| کلاس | مسئولیت |
|------|---------|
| User | حساب کاربری و نقش (موکل، وکیل، مدیر) |
| LawyerProfile | اطلاعات حرفه‌ای وکیل و میانگین نمره |
| Specialty | حوزهٔ تخصصی (خانواده، ملکی، کیفری، ...) |
| VerificationDocument | مدارک بارگذاری‌شده برای تأیید وکیل |
| LegalQuestion | سؤال حقوقی موکل (مشاورهٔ سریع) |
| Answer | پاسخ وکیل به سؤال |
| CaseRequest | درخواست پذیرش پرونده برای یک وکیل |
| Review | نمره و نظر موکل دربارهٔ وکیل |
| Report | گزارش کاربر دربارهٔ محتوای نامناسب |
| AvailabilitySlot | بازهٔ آزاد تقویم وکیل |
| Appointment | نوبت رزروشدهٔ مشاوره (حضوری یا تلفنی) |
| Payment | تراکنش پرداخت و کمیسیون پلتفرم |
| CaseDocument | مدرک بارگذاری‌شدهٔ پرونده |
| Message | پیام چت خصوصی در چارچوب پرونده |
| CaseEvent | تغییر وضعیت پیشرفت پرونده |
| Subscription | اشتراک ویژهٔ وکیل برای رتبهٔ بالاتر |

---

## ۴. قوانین دامنه

- فقط وکیلِ با وضعیت `VERIFIED` در نتایج جست‌وجو نمایش داده می‌شود
- `Review` فقط وقتی ثبت می‌شود که موکل از آن وکیل `Answer` یا `CaseRequest` پذیرفته‌شده داشته باشد
- نمرهٔ هر `Review` بین ۱ تا ۵ است
- `averageRating` پس از هر نظر جدید یا مخفی‌شدن نظر دوباره محاسبه می‌شود
- هر `AvailabilitySlot` حداکثر یک `Appointment` فعال دارد
- نوبت فقط پس از پرداخت موفق `CONFIRMED` می‌شود
- وکیل با `Subscription` فعال در نتایج جست‌وجو بالاتر نمایش داده می‌شود
- فقط موکل و وکیل همان پرونده به `CaseDocument` و `Message` دسترسی دارند

---

## پیوست الف — تاریخچهٔ تغییرات

| نسخه | تاریخ | تغییرات | تهیه‌کننده |
|-------|-------|----------|--------------|
| ۱.۰ | ۱۴۰۵ | نسخهٔ اولیه | سینا نوروزی، احسان رستم خانی، محمد مانی امینی، سالار هادی |
| ۱.۱ | ۱۴۰۵ | افزودن کلاس‌های نوبت، پرداخت، مدارک، چت و اشتراک | سینا نوروزی، احسان رستم خانی، محمد مانی امینی، سالار هادی |
