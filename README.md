# 🛡️ Prop Discipline Sentinel (MT5 to Telegram)

[![Platform: MetaTrader 5](https://img.shields.io/badge/Platform-MetaTrader%205-blue.svg)](https://www.metatrader5.com)
[![Language: MQL5](https://img.shields.io/badge/Language-MQL5-orange.svg)]()
[![Prop-Firm Safe: Read--Only](https://img.shields.io/badge/Prop--Firm-Safe%20(Read--Only)-green.svg)]()
[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)]()

یک اکسپرت مانیتورینگ **فقط‌خواندنی (Read-Only)** برای متاتریدر ۵ جهت مدیریت ریسک، پیشگیری از نقض قوانین دروداون در حساب‌های پراپ‌فرم و مهار خطاهای رفتاری (ترید انتقامی و استاپ‌کشی).

---

## 📌 ویژگی‌های کلیدی

- **پایش لحظه‌ای جابه‌جایی و عقب‌کشیدن حد ضرر (SL Tampering):** محاسبه خودکار میزان دلاریِ ریسک تحمیل‌شده و ارسال اخطار فوری به تلگرام.
- **توقف با توالی باخت (Consecutive Losses):** ممانعت از ترید پس از ۲ باخت متوالی روزانه و صدور دستور خروج از بازار.
- **تایمر وقفه اجباری روانی (Cooldown Timer):** هشدار ترید شتاب‌زده (Revenge Alert) در صورت ورود زیر ۱۰ دقیقه پس از باخت قبلی.
- **پایش سقف افت مجاز و تارگت سود روزانه:** هشدار آنی با لمس سقف ضرر مجاز روزانه یا تارگت خروج روزانه.
- **پنل هوشمند روی چارت (On-Chart HUD):** نمایش زنده سشن بازار (لندن/نیویورک)، نوسان ATR، و محاسبه دقیق لات‌سایز برای ریسک مشخص (۰.۵٪).
- **پروتکل تعهد آگاهانه (Pre-Trade 30s Pop-Up):** نمایش پاپ‌آپ اخطار تفکر ۳۰ ثانیه‌ای روی صفحه متاتریدر به محض کلیک روی Buy/Sell.

---

## 🚀 راهنمای نصب و راه‌اندازی سریع

### ۱. فعال‌سازی وب‌ریکوئست در متاتریدر ۵
1. در متاتریدر ۵ کلیدهای `Ctrl + O` را بزنید و به تب **Expert Advisors** بروید.
2. تیک **Allow WebRequest for listed URL** را فعال کنید.
3. آدرس زیر را به لیست اضافه کرده و OK را بزنید:
   ```text
   [https://api.telegram.org](https://api.telegram.org)
