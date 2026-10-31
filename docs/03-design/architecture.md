# سند معماری — دادرس

| | |
|---|---|
| نام پروژه | دادرس (Dadress) |
| نسخهٔ سند | ۱.۱ |
| تاریخ | ۱۴۰۵ |
| تهیه‌کننده | سینا نوروزی، احسان رستم خانی، محمد مانی امینی، سالار هادی، سعید بابائی |
| وضعیت | پیش‌نویس |

---

## ۱. مقدمه

### ۱.۱ هدف سند
این سند معماری نرم‌افزار پروژهٔ دادرس را تعریف می‌کند.
هدف آن ایجاد یک درک مشترک از ساختار سیستم، تصمیم‌های کلیدی و
محدودیت‌ها است.

### ۱.۲ دامنه
- در دامنه: معماری سطح بالا، لایه‌ها، Endpointها، امنیت.
- خارج از دامنه: جزئیات پیاده‌سازی، کد.

### ۱.۳ ذی‌نفعان
| ذی‌نفع | نقش |
|--------|------|
| تیم توسعه | پیاده‌سازی |
| استاد درس | ارزیابی |
| مشتری فرضی (دفتر وکالت / موکلان) | تأیید |

---

## ۲. نمای کلی معماری

### ۲.۱ سبک معماری
Monolith + Layered Architecture

- یک برنامهٔ واحد
- سه لایهٔ اصلی: Presentation, Business, Data
- مناسب تیم کوچک (۳-۴ نفر)
- ساده برای شروع، قابل گسترش

### ۲.۲ دیاگرام لایه‌ها

┌─────────────────────────────────────────┐
│            Presentation Layer           │
│               React (SPA)               │
└─────────────────┬───────────────────────┘
                  │ HTTP/REST
┌─────────────────▼───────────────────────┐
│             Business Layer              │
│          Node.js + Express              │
│  (Auth, Lawyers, Questions, Reviews)    │
└─────────────────┬───────────────────────┘
                  │ SQL (Prisma)
┌─────────────────▼───────────────────────┐
│               Data Layer                │
│               PostgreSQL                │
└─────────────────────────────────────────┘
### ۲.۳ جریان داده

flowchart LR
    U["Browser"] -->|HTTPS| F["React SPA"]
    F -->|"REST + JWT"| B["Express API"]
    B -->|Prisma| D[("PostgreSQL")]
    B -->|Files| S["File Storage"]
    B -->|Payment| G["Payment Gateway - Sandbox"]
    F <-->|WebSocket| B
| عنصر در دیاگرام | توضیح |
|-----------------|-------|
| Browser | مرورگر کاربر (موکل، وکیل یا مدیر) |
| React SPA | رابط کاربری (Frontend) |
| Express API | سرور و منطق برنامه (Backend) |
| PostgreSQL | پایگاه داده |
| File Storage | فضای ذخیره‌سازی مدارک پرونده |
| Payment Gateway | درگاه پرداخت (آزمایشی در نسخهٔ اولیه) |
| WebSocket | ارتباط زندهٔ چت بین موکل و وکیل |

---

## ۳. لایه‌ها

### ۳.۱ Presentation Layer (Frontend)

| مورد | انتخاب |
|------|--------|
| فناوری | React 18 |
| Routing | React Router |
| State | Redux Toolkit |
| UI | Tailwind CSS |
| Build | Vite |

### ۳.۲ Business Layer (Backend)

| مورد | انتخاب |
|------|--------|
| فناوری | Node.js 20 + Express 4 |
| ORM | Prisma |
| Validation | Zod |
| Auth | JWT |
| Logging | Winston |

ماژول‌های Business: auth، lawyers، questions، case-requests، appointments، payments، documents، chat، subscriptions، reviews، admin.

### ۳.۳ Data Layer (Database)

| مورد | انتخاب |
|------|--------|
| فناوری | PostgreSQL 16 |
| Migration | Prisma Migrate |
| Backup | روزانه |
| Hosting | Railway |

جزئیات جدول‌ها در [database-schema.md](database-schema.md).

---

## ۴. تصمیم‌های معماری (ADR)

### ADR-01: چرا Monolith؟

تصمیم: معماری Monolith

دلیل:
- تیم کوچک (۳-۴ نفر)
- پیچیدگی کم
- سرعت توسعه بالاتر
- استقرار ساده

### ADR-02: چرا React؟

تصمیم: React برای Frontend

دلیل:
- محبوب و مستند
- اکوسیستم بزرگ
- SPA برای تجربهٔ بهتر در جست‌وجو و مقایسهٔ وکلا

### ADR-03: چرا PostgreSQL؟

تصمیم: PostgreSQL برای Database

دلیل:
- داده‌های رابطه‌ای (کاربر، وکیل، سؤال، نظر)
- پشتیبانی از جست‌وجوی متنی و JSON
- رایگان و اوپن‌سورس

### ADR-04: چرا نمره و نظر فقط برای خدمت‌گیرندگان؟

تصمیم: ثبت نظر فقط برای موکلی که پاسخ یا پروندهٔ پذیرفته‌شده از همان وکیل دارد.

دلیل:
- جلوگیری از نظرات جعلی (NFR-07)
- افزایش اعتماد کاربران به نمرات
- هزینهٔ پیاده‌سازی کم (بررسی در لایهٔ Business)

### ADR-05: چرا وکلا باید تأیید شوند؟

تصمیم: وکیل تا تأیید مدارک توسط مدیر در نتایج جست‌وجو دیده نمی‌شود.

دلیل:
- صحت اطلاعات حرفه‌ای (NFR-06)
- کاهش ریسک اطلاعات نادرست

### ADR-06: چرا درگاه پرداخت خارجی (Redirect)؟

تصمیم: پرداخت از طریق انتقال کاربر به درگاه؛ اطلاعات کارت هرگز در سامانهٔ ما ذخیره نمی‌شود. در نسخهٔ اولیه از درگاه آزمایشی (Sandbox) استفاده می‌شود.

دلیل:
- امنیت (NFR-09)
- پیچیدگی کم برای تیم کوچک
- امکان آزمون بدون پول واقعی

### ADR-07: چرا Socket.IO برای چت؟

تصمیم: چت متنی خصوصی با Socket.IO (WebSocket با جایگزین Polling).

دلیل:
- پیام‌ها بدون بارگذاری مجدد صفحه برسند
- یکپارچه با Node.js
- ذخیرهٔ پیام‌ها در PostgreSQL برای سابقهٔ گفت‌وگو

### ADR-08: چرا مدارک در فضای ذخیره‌سازی جدا؟

تصمیم: فایل‌ها در فضای ذخیره‌سازی (دیسک سرور یا سرویس شیء) و فقط مسیر آن‌ها در پایگاه داده.

دلیل:
- کوچک‌ماندن پایگاه داده
- امکان کنترل دسترسی در API
- امکان مهاجرت به سرویس ابری

---

## ۵. REST API

### ۵.۱ Endpointهای احراز هویت

| Method | Endpoint | توضیح |
|--------|----------|-------|
| POST | /api/auth/register | ثبت‌نام (موکل یا وکیل) |
| POST | /api/auth/login | ورود |
| POST | /api/auth/refresh | تمدید توکن |

### ۵.۲ Endpointهای عمومی (وکلا)

| Method | Endpoint | توضیح |
|--------|----------|-------|
| GET | /api/lawyers | فهرست وکلای تأییدشده |
| GET | /api/lawyers/search | جست‌وجو و فیلتر (تخصص، شهر، نمره) |
| GET | /api/lawyers/:id | پروفایل وکیل |
| GET | /api/lawyers/:id/reviews | نظرات دربارهٔ وکیل |
| GET | /api/specialties | فهرست تخصص‌ها |
| GET | /api/lawyers/recommend | پیشنهاد وکیل بر اساس موضوع |

### ۵.۳ Endpointهای موکل

| Method | Endpoint | توضیح |
|--------|----------|-------|
| POST | /api/questions | ثبت سؤال حقوقی |
| GET | /api/questions/mine | سؤال‌های من و پاسخ‌ها |
| POST | /api/case-requests | درخواست پذیرش پرونده |
| GET | /api/case-requests/mine | وضعیت درخواست‌های من |
| POST | /api/reviews | ثبت نمره و نظر |
| POST | /api/reports | گزارش یک نظر یا پروفایل |

### ۵.۴ Endpointهای وکیل

| Method | Endpoint | توضیح |
|--------|----------|-------|
| PUT | /api/lawyer/profile | ایجاد و ویرایش پروفایل |
| POST | /api/lawyer/documents | بارگذاری مدارک |
| GET | /api/lawyer/questions | سؤال‌های مرتبط با تخصص من |
| POST | /api/questions/:id/answers | پاسخ به سؤال |
| GET | /api/lawyer/case-requests | درخواست‌های پرونده |
| PUT | /api/case-requests/:id/status | قبول یا رد درخواست |

### ۵.۵ Endpointهای مدیر

| Method | Endpoint | توضیح |
|--------|----------|-------|
| GET | /api/admin/lawyers/pending | وکلای در انتظار تأیید |
| PUT | /api/admin/lawyers/:id/verification | تأیید یا رد مدارک |
| GET | /api/admin/reports | فهرست گزارش‌ها |
| PUT | /api/admin/reports/:id | رسیدگی به گزارش |
| PUT | /api/admin/reviews/:id/visibility | مخفی یا نمایش نظر |

### ۵.۶ Endpointهای نوبت، پرداخت و اشتراک

| Method | Endpoint | توضیح |
|--------|----------|-------|
| GET | /api/lawyers/:id/slots | بازه‌های آزاد وکیل |
| PUT | /api/lawyer/slots | مدیریت تقویم و زمان‌های آزاد (وکیل) |
| POST | /api/appointments | رزرو نوبت (موکل) |
| GET | /api/appointments/mine | نوبت‌های من |
| POST | /api/payments | شروع پرداخت و انتقال به درگاه |
| GET | /api/payments/callback | بازگشت از درگاه و تأیید پرداخت |
| POST | /api/lawyer/subscription | خرید اشتراک ویژه (وکیل) |

### ۵.۷ Endpointهای مدارک، چت و پیشرفت پرونده

| Method | Endpoint | توضیح |
|--------|----------|-------|
| POST | /api/case-requests/:id/documents | بارگذاری مدرک (موکل) |
| GET | /api/case-requests/:id/documents | فهرست مدارک (موکل و وکیل همان پرونده) |
| GET | /api/case-requests/:id/messages | تاریخچهٔ چت |
| POST | /api/case-requests/:id/messages | ارسال پیام (علاوه بر Socket.IO) |
| GET | /api/case-requests/:id/events | تاریخچهٔ پیشرفت پرونده |
| POST | /api/case-requests/:id/events | ثبت وضعیت جدید پرونده (وکیل) |

### ۵.۸ Endpointهای گزارش مالی (مدیر)

| Method | Endpoint | توضیح |
|--------|----------|-------|
| GET | /api/admin/payments | فهرست تراکنش‌ها |
| GET | /api/admin/finance/summary | جمع تراکنش‌ها و کمیسیون |

### ۵.۹ نمونه Response

(مقادیر نمونه به انگلیسی نوشته شده‌اند.)

{
  "id": 12,
  "fullName": "Reza Mohammadi",
  "city": "Tehran",
  "specialties": ["Family Law", "Real Estate"],
  "experienceYears": 12,
  "verificationStatus": "VERIFIED",
  "averageRating": 4.6,
  "reviewCount": 38
}
---

## ۶. امنیت

### ۶.۱ احراز هویت
- JWT برای API
- bcrypt برای رمزنگاری گذرواژه
- Refresh Token برای تمدید

### ۶.۲ مجوزدهی
- Role-Based Access Control (RBAC)
- نقش‌ها: Client, Lawyer, Admin

| نقش | دسترسی |
|-----|--------|
| Client | ثبت سؤال، درخواست پرونده، نمره و گزارش |
| Lawyer | مدیریت پروفایل، پاسخ به سؤال، مدیریت درخواست‌ها |
| Admin | تأیید مدارک، رسیدگی به گزارش‌ها، مدیریت کاربران |

### ۶.۳ امنیت API
- HTTPS اجباری
- Rate Limiting (۱۰۰ درخواست در دقیقه)
- CORS محدود
- Input Validation با Zod

### ۶.۴ پرداخت و فایل‌ها
- اطلاعات کارت در سامانه ذخیره نمی‌شود (Redirect به درگاه)
- تأیید پرداخت فقط از سمت سرور و با کد پیگیری درگاه انجام می‌شود
- فایل‌ها: فقط PDF/JPG/PNG، حداکثر ۱۰ مگابایت، نام فایل تصادفی
- دانلود مدرک فقط برای موکل و وکیل همان پرونده

### ۶.۵ حریم خصوصی
- سؤال‌ها و پرونده‌ها فقط برای وکیل مرتبط و صاحب سؤال قابل مشاهده است (NFR-03)
- مدارک وکلا فقط برای مدیر قابل دانلود است
- دادهٔ حساس در لاگ‌ها ثبت نمی‌شود

---

## ۷. مقیاس‌پذیری

### ۷.۱ محدودیت‌های فعلی
- Monolith → مقیاس‌پذیری محدود
- یک سرور → تک نقطهٔ شکست

### ۷.۲ مسیر رشد
1. فاز ۱ (MVP): Monolith + یک سرور
2. فاز ۲: Load Balancer + چند سرور
3. فاز ۳: Database Replication
4. فاز ۴: جدا کردن سرویس‌های پرمصرف (جست‌وجو، اعلان)

---

## ۸. ریسک‌ها و بدهی فنی

| ریسک | احتمال | اثر | راهکار |
|------|--------|-----|---------|
| نظرات جعلی | متوسط | بالا | ADR-04 + گزارش و مخفی‌سازی |
| اطلاعات نادرست وکیل | متوسط | بالا | تأیید مدارک (ADR-05) |
| نفوذ امنیتی | کم | بالا | OWASP |
| افشای اطلاعات پرونده | کم | بالا | RBAC + HTTPS |
| از دست دادن داده | کم | بالا | Backup روزانه |
| پرداخت تکراری یا ناموفق | متوسط | بالا | ثبت تراکنش، بررسی وضعیت از سمت سرور |
| فایل مخرب | کم | بالا | بررسی نوع و حجم، ذخیره خارج از مسیر عمومی |
| بدهی فنی | بالا | متوسط | Refactor در Sprint 3 |

---

## ۹. فناوری‌ها

| لایه | فناوری |
|------|--------|
| Frontend | React 18 + Vite + Tailwind |
| Backend | Node.js 20 + Express 4 |
| Database | PostgreSQL 16 |
| ORM | Prisma |
| Auth | JWT + bcrypt |
| Realtime | Socket.IO |
| Upload | Multer |
| Payment | درگاه آزمایشی (Sandbox) |
| Testing | Jest + Supertest |
| CI/CD | GitHub Actions |
| Deploy | Railway |

---

## پیوست الف — تاریخچهٔ تغییرات

| نسخه | تاریخ | تغییرات | تهیه‌کننده |
|-------|-------|----------|--------------|
| ۱.۰ | ۱۴۰۵ | نسخهٔ اولیه | سینا نوروزی، احسان رستم خانی، محمد مانی امینی، سالار هادی |
| ۱.۱ | ۱۴۰۵ | افزودن پرداخت، چت، مدارک، نوبت‌دهی و ADR-06 تا ADR-08 | سینا نوروزی، احسان رستم خانی، محمد مانی امینی، سالار هادی |
