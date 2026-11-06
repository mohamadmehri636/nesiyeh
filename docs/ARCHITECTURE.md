# معماری کلان نسیه

## 1. الگوی معماری
Modular Monolith با Laravel؛ مرزبندی ماژول‌ها در سطح دامنه و سرویس‌ها انجام می‌شود. در MVP استفاده از Microservices یا SPA مستقل توجیه ندارد.

ماژول‌های هدف:
- Identity & Access: کاربران، OTP، نشست‌ها، نقش‌ها و مجوزها
- Business & Branch: کسب‌وکار، شعبه و عضویت کاربران
- Customer: پرونده مشتری در محدوده کسب‌وکار
- Ledger: حساب و تراکنش‌های مالی تغییرناپذیر
- Audit: ثبت رخدادهای حساس و مدیریتی
- Notification: اعلان درون‌برنامه‌ای؛ SMS و Push در مراحل بعد

## 2. Multi-Tenancy
- Business مرز اصلی Tenant است.
- عضویت کاربر در جدول واسط business_user تعریف می‌شود؛ Role به Membership تعلق دارد.
- یک کاربر می‌تواند در چند Business نقش متفاوت داشته باشد.
- دسترسی شعبه از طریق business_user_branch کنترل می‌شود.
- تمام Queryها، Policyها و عملیات نوشتن باید Business context را بررسی کنند.
- شناسه غیرقابل حدس به‌تنهایی جایگزین مجوزدهی نیست؛ کنترل IDOR الزامی است.
- قیود و کلیدهای خارجی مرکب باید از ارجاع بین Businessها جلوگیری کنند.

## 3. Ledger
مانده حساب از دفتر تراکنش‌ها محاسبه می‌شود:

    balance = SUM(DEBIT) - SUM(CREDIT)

مبلغ مثبت و integer ریال است (BIGINT). هر تراکنش پس از ثبت immutable است. اصلاح با REVERSAL یا ADJUSTMENT انجام می‌شود؛ حذف فیزیکی تراکنش ممنوع است.

انواع تراکنش:
- SALE
- PAYMENT
- PAYMENT_REFUND
- SALE_RETURN
- REVERSAL
- ADJUSTMENT

جهت‌ها: DEBIT و CREDIT.

روش پرداخت: CASH، CARD، BANK_TRANSFER، OTHER. برای PAYMENT و PAYMENT_REFUND الزامی و برای سایر انواع NULL است.

قواعد ارجاع:
- PAYMENT_REFUND، SALE_RETURN و REVERSAL باید به تراکنش مرجع در همان Business و Account ارجاع دهند.
- هر تراکنش حداکثر یک‌بار قابل REVERSAL است.
- خود REVERSAL قابل Reverse مجدد نیست.
- ADJUSTMENT توضیح الزامی دارد و فقط نقش‌های مجاز مانند OWNER یا ACCOUNTANT آن را ثبت می‌کنند.

## 4. داده و چرخه عمر
- Account به ازای (business_id, customer_id) یکتا است.
- Branch code و customer_code در محدوده Business یکتا هستند.
- شماره تماس مشتری اختیاری و در صورت وجود در همان Business یکتا است.
- شماره مشتری و شماره تراکنش در سطح Business مستقل و atomic تولید می‌شوند؛ MAX(id)+1 ممنوع است.
- UUID برای شناسه عمومی منابع استفاده می‌شود.
- Business، Branch، Customer و User غیرفعال/Soft Delete می‌شوند.
- Account، Transaction و Audit Log حذف فیزیکی نمی‌شوند.
- روابط مالی با ON DELETE RESTRICT محافظت می‌شوند.

## 5. احراز هویت و اعلان
OTP برای مالک کسب‌وکار و کاربران در معماری از ابتدا لحاظ می‌شود. OTP در DB به‌صورت hash، دارای TTL، rate limit، محدودیت تلاش و مصرف یک‌باره است. Provider توسعه محلی از SMS تولید جداست و کد تستی در Production مطلقاً فعال نمی‌شود.

Sanctum برای مدیریت نشست/token استفاده می‌شود؛ OTP provider مسئول اثبات مالکیت شماره تلفن است. اعلان درون‌برنامه‌ای در MVP؛ SMS و FCM برای مراحل بعد.

## 6. کلاینت‌ها
- Web: Blade + Tailwind CSS + Alpine.js
- Android: Kotlin + Jetpack Compose، Clean Architecture + MVVM، Retrofit، Room، Hilt و Coroutines
- Android فقط از REST API نسخه‌دار استفاده می‌کند و هرگز مستقیم به PostgreSQL متصل نمی‌شود.
- داده Ledger روی Server مرجع نهایی است؛ Room صرفاً cache و داده موقت است.
