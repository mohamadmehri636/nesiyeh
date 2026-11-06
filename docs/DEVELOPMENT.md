# راهنمای توسعه

## پیش‌نیازها
- PHP 8.5 با افزونه‌های موردنیاز Laravel و Composer
- PostgreSQL 16
- Git
- Node.js LTS و npm برای build دارایی‌های Web
- Android Studio هنگام شروع توسعه کلاینت Android

## راه‌اندازی Backend
1. در ریشه مخزن اجرا کنید: composer create-project laravel/laravel:^13.0 backend
2. فایل backend/.env.example را به backend/.env کپی و تنظیمات PostgreSQL را تکمیل کنید.
3. دیتابیس و کاربر PostgreSQL را مطابق تنظیمات محلی ایجاد کنید.
4. از پوشه backend اجرا کنید: php artisan key:generate
5. سپس php artisan serve و تست‌ها: php artisan test

این Foundation فقط قراردادها و فایل‌های راه‌اندازی را نگه می‌دارد؛ کد Laravel و vendor را Composer دریافت می‌کند. هیچ رمز یا فایل .env واقعی را commit نکنید.

## قواعد تغییر
- قبل از افزودن migration، قواعد دامنه و Multi-Tenant constraints را بررسی کنید.
- هر قابلیت باید با تست Unit/Feature همراه باشد.
- برای عملیات مالی، تست‌های concurrency، tenant isolation، immutability و idempotency الزامی است.
- پس از پایان مرحله، دستورات تست و نتیجه واقعی را ثبت کنید.
- تا تأیید تست‌های مرحله جاری، وارد مرحله بعد نشوید.
