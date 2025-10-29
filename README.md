# AppVinka (modesoud)

اپلیکیشن اندرویدی برای ضبط مداوم صدا و ضبط اتوماتیک تماس‌ها با استفاده از Quasar و Capacitor.

## ویژگی‌ها

- **ضبط مداوم صدا**: ضبط صدا تا زمان توقف دستی یا خاموش شدن گوشی
- **ضبط اتوماتیک تماس‌ها**: ضبط تماس‌ها از زمان زنگ خوردن تا پایان تماس
- **رابط کاربری فارسی**: کنترل‌های ساده شروع/توقف ضبط
- **ذخیره‌سازی محلی**: فایل‌های ضبط شده در دستگاه ذخیره می‌شوند
- **تنظیمات کیفیت**: امکان تنظیم کیفیت ضبط صدا

## تکنولوژی‌ها

- **Quasar v2.18.5**: فریمورک Vue.js برای اپلیکیشن‌های موبایل
- **Vue 3**: با Composition API
- **Capacitor**: پلتفرم توسعه native موبایل
- **TypeScript**: برای تایپ امن کد
- **Web Audio API**: برای ضبط صدا

## دسترسی‌های اندروید

اپلیکیشن نیاز به دسترسی‌های زیر دارد:
- `RECORD_AUDIO`: برای ضبط صدا
- `READ_PHONE_STATE`: برای تشخیص وضعیت تماس
- `WRITE_EXTERNAL_STORAGE`: برای ذخیره فایل‌ها
- `FOREGROUND_SERVICE`: برای ضبط در background
- `WAKE_LOCK`: برای جلوگیری از sleep دستگاه

## نصب و راه‌اندازی

### پیش‌نیازها

- Node.js (نسخه 16 یا بالاتر)
- Yarn یا npm
- Android Studio (برای توسعه اندروید)
- دستگاه اندرویدی یا emulator

### نصب وابستگی‌ها

```bash
yarn install
# یا
npm install
```

### اجرای اپلیکیشن در حالت توسعه

```bash
quasar dev
```

### ساخت اپلیکیشن برای تولید

```bash
quasar build
```

### sync با اندروید

```bash
npx cap sync android
```

### باز کردن پروژه در Android Studio

```bash
npx cap open android
```

## ساختار پروژه

```
src/
├── pages/
│   └── IndexPage.vue          # صفحه اصلی با کنترل‌های ضبط
├── components/
│   ├── EssentialLink.vue
│   └── ExampleComponent.vue
├── layouts/
│   └── MainLayout.vue
├── stores/
│   └── example-store.ts
├── router/
│   ├── routes.ts
│   └── index.ts
├── i18n/                      # پشتیبانی چندزبانه
├── boot/                      # فایل‌های راه‌اندازی
└── assets/                    # منابع استاتیک

android/                       # پروژه native اندروید
├── app/
│   ├── src/main/
│   │   ├── AndroidManifest.xml
│   │   └── java/
│   └── build.gradle
└── ...
```

## تنظیمات Capacitor

فایل `capacitor.config.ts` شامل تنظیمات اصلی Capacitor است:
- `appId`: شناسه منحصر به فرد اپلیکیشن
- `appName`: نام اپلیکیشن
- `webDir`: پوشه خروجی build

## توسعه بیشتر

### ضبط تماس واقعی
برای ضبط تماس‌ها نیاز به پلاگین سفارشی Capacitor دارید که با native Android API کار کند.

### ضبط در background
برای ضبط مداوم در background نیاز به foreground service در اندروید دارید.

### بهبود UI
- نمایش waveform صدا
- کنترل‌های پیشرفته ضبط
- تنظیمات کیفیت بیشتر

## رفع اشکال

### خطاهای کامپایل
اگر با خطاهای TypeScript مواجه شدید، مطمئن شوید که تمام وابستگی‌ها نصب شده‌اند:

```bash
yarn install
```

### مشکلات sync با اندروید
اگر sync با اندروید شکست خورد، مطمئن شوید که Android Studio نصب است و SDK به‌روز است.

### دسترسی‌های اندروید
اگر اپلیکیشن دسترسی‌های لازم را ندارد، فایل `AndroidManifest.xml` را چک کنید.

## لایسنس

این پروژه تحت لایسنس MIT منتشر شده است.
