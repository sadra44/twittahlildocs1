# قراردادهای وب‌سرویس، ساختار API و پاکت‌های تبادل داده
## REST API Specifications, JSON Contracts & Endpoints Catalog

این سند مشخصات قراردادهای تبادل داده در لایه وب‌سرویس، اندپوئینت‌های فید ۳ تبی، بخش کاوش، دستیار هوش مصنوعی و پنل ادمین را تدوین می‌کند.

---

## ۱. ساختار استاندارد پاکت پاسخ و خطا

### ساختار پاسخ موفق:
```json
{
  "success": true,
  "data": {},
  "meta": {
    "cursor": "eyJjcmVhdGVkX2F0IjoiMjAyNi0xMC0wOVQxMDo0MDowMFoifQ==",
    "has_more": true
  },
  "error": null
}
```

### ساختار پاسخ خطا:
```json
{
  "success": false,
  "data": null,
  "meta": null,
  "error": {
    "code": "VALIDATION_FAILED",
    "message": "متن توییت حاوی لینک غیرمجاز تلگرام است.",
    "field": "content"
  }
}
```

---

## ۲. مشخصات اندپوئینت‌های کلیدی سامانه

### الف) ماژول انتشار توییت و دستیار هوش مصنوعی (`/api/v1/posts`)

#### ۱. بررسی دستیار نگارش هوش مصنوعی (مخصوص کاربران تاییدشده)
* **مسیر:** `POST /api/v1/posts/ai-assistant-check`
* **هدر:** `Authorization: Bearer <TOKEN>` (نشان آبی/طلایی/سبز الزامی)
* **بدنه درخواست:**
```json
{
  "content": "به نظر میرسد ایران خودرو به زودی از زیان انباشته خارج شود."
}
```
* **پاسخ موفق (HTTP 200):**
```json
{
  "success": true,
  "data": {
    "has_suggestions": true,
    "suggested_content": "به نظر می‌رسد $خودرو (ایران خودرو) به زودی از زیان انباشته خارج شود.",
    "suggested_symbols": ["خودرو"],
    "corrected_typos": true
  }
}
```

#### ۲. انتشار توییت جدید
* **مسیر:** `POST /api/v1/posts`
* **بدنه درخواست:**
```json
{
  "content": "شکست خط روند نزولی و تثبیت در محدوده حمایتی. $فولاد",
  "stock_ids": ["9f323c8a-78b1-419b-a320-b88301ec711a"],
  "attachments": [
    {
      "url": "https://storage.boursino.ir/files/foolad-chart.webp",
      "file_type": "image/webp",
      "file_name": "foolad-chart.webp",
      "size_bytes": 450000
    }
  ],
  "reply_to_id": null
}
```
* **پاسخ موفق (HTTP 201 Created):** بازگرداندن آبجکت توییت ذخیره‌شده.

---

### ب) ماژول فیدهای خانه و کاوش (`/api/v1/feeds`)

#### ۱. فید خانه با ۳ تب
* **مسیر:** `GET /api/v1/feeds/home?tab=for-you&cursor=...&limit=20`
* **پارامتر `tab`:** `for-you` (مخصوص شما) | `all` (همه) | `following` (دنبال‌شده).

#### ۲. فید بخش کاوش
* **مسیر:** `GET /api/v1/feeds/explore?sort=views&timeframe=24h&cursor=...&limit=20`
* **پارامتر `sort`:** `views` (پربازدیدترین‌ها) | `comments` (پربحث‌ترین‌ها) | `likes` (بیشترین پسندها).
* **پارامتر `timeframe`:** `24h` | `48h` | `7d`.

---

### ج) ماژول نمادها و خلاصه هوش مصنوعی (`/api/v1/stocks`)

#### ۱. دریافت تابلوی کمکی و خلاصه هوش مصنوعی سهم
* **مسیر:** `GET /api/v1/stocks/:symbol`
* **پاسخ موفق (HTTP 200):**
```json
{
  "success": true,
  "data": {
    "symbol": "فولاد",
    "name_fa": "فولاد مبارکه اصفهان",
    "sector": "فلزات اساسی",
    "market_strip": {
      "last_price": 5420,
      "closing_price": 5380,
      "price_change_pct": 2.3,
      "status": "AUTHORIZED"
    },
    "ai_discussion_digest": {
      "is_available": true,
      "headline": "اجماع عمومی متمایل به رشد با تثبیت روی حمایت ۵۲۰ تومان",
      "pros": ["تقاضای سنگین در بورس کالا", "حمایت تکنیکال موج ۴"],
      "cons": ["ابهام در نرخ سوخت گاز صنایع"],
      "updated_at": "2026-10-09T08:00:00Z"
    }
  }
}
```

#### ۲. لیست توییت‌های متصل به نماد
* **مسیر:** `GET /api/v1/stocks/:symbol/tweets?verified_only=true&cursor=...`
* **پارامتر `verified_only`:** اگر `true` باشد تنها توییت‌های کاربران با نشان آبی، طلایی و سبز فیلتر می‌شوند.

---

### د) ماژول جستجوی جامع (`/api/v1/search`)

* **مسیر:** `GET /api/v1/search?q=ایران خودرو`
* **پاسخ موفق:** تطبیق آنی نام ثبتی «ایران خودرو» و بازگرداندن کارت نماد `$خودرو` همراه با توییت‌ها و کاربران مرتبط.

---

### ه) ماژول مدیریت ادمین و تنظیمات پویا (`/api/v1/admin`)

* **ویرایش اطلاعات کاربر:** `PATCH /api/v1/admin/users/:id` (ویرایش نام، هندل، تلفن، کلمه عبور و رده).
* **مدیریت توییت:** `PATCH /api/v1/admin/posts/:id` (ویرایش متن، مخفی‌سازی، پین سراسری).
* **تنظیمات پویا و کدهای رنگی تم:** `GET /api/v1/config/theme` و `PUT /api/v1/admin/config/theme` (تنظیم HEX رنگ‌های روز و شب).
* **ویرایش متن قوانین:** `PUT /api/v1/admin/config/terms`.
* **تست کنسول هوش مصنوعی:** `POST /api/v1/admin/ai/test-prompt` و `POST /api/v1/admin/ai/trigger-morning-brief`.
