# مدل داده، دیاگرام روابط و اسکیما پایگاه داده
## Data Model, Relational Schema & Database Design

این سند ساختار جداول پایگاه داده رابطه‌ای (PostgreSQL)، کلیدهای خارجی، محدودیت‌ها، انوم‌ها و ایندکس‌های بهینه‌سازی‌شده را طبق آخرین تصمیمات معماری تدوین می‌کند.

---

## ۱. دیاگرام روابط موجودیت‌ها (Entity-Relationship Diagram)

```text
  ┌──────────────┐          1:N          ┌──────────────────────┐
  │    users     ├──────────────────────►│        posts         │
  └──────┬───────┘                       └──────────┬───────────┘
         │                                          │
         ├──────────────┬──────────────┐ ┌──────────┴───────────┐
         │ 1:1          │ 1:N          │ │ M:N                  │ 1:N
         ▼              ▼              ▼ ▼                      ▼
  ┌──────────────┐┌──────────────┐┌──────────────┐      ┌──────────────┐
  │user_settings ││ user_follows ││ post_stocks  │      │    likes     │
  │ (تنظیمات)    ││(دنبال‌کردن)  ││(اتصال به سهم)│      │    reposts   │
  └──────────────┘└──────────────┘└──────┬───────┘      │   bookmarks  │
                                         │              └──────────────┘
                                         │ M:1
                                         ▼
                                  ┌──────────────┐
                                  │    stocks    │
                                  │ (بانک نمادها)│
                                  └──────────────┘
```

---

## ۲. تعاریف انواع داده شمارشی (Custom PostgreSQL ENUMs)

```sql
-- رده‌های کاربری و نشان‌های اعتبار
CREATE TYPE user_tier_enum AS ENUM (
  'NORMAL_USER',
  'VERIFIED_ANALYST',
  'FINANCIAL_INSTITUTION',
  'OFFICIAL_ISSUER',
  'MARKET_BOT',
  'ADMIN_MODERATOR'
);

-- وضعیت‌های اعتبارسنجی مدارک
CREATE TYPE verification_status_enum AS ENUM (
  'PENDING',
  'UNDER_REVIEW',
  'APPROVED',
  'REJECTED'
);

-- انواع اعلان‌های سیستمی
CREATE TYPE notification_type_enum AS ENUM (
  'LIKE',
  'COMMENT',
  'MENTION',
  'REPOST',
  'FOLLOW',
  'VERIFICATION_RESULT'
);

-- دلایل گزارش تخلف
CREATE TYPE report_reason_enum AS ENUM (
  'SIGNAL_SELLING',
  'HARASSMENT',
  'MISINFORMATION',
  'SPAM'
);
```

---

## ۳. تعریف جداول اصلی پایگاه داده (DDL Schemas)

### ۱. جدول کاربران (`users`)
```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  phone_number VARCHAR(15) UNIQUE NOT NULL,
  handle VARCHAR(32) UNIQUE NOT NULL,
  display_name VARCHAR(64) NOT NULL,
  password_hash VARCHAR(255),                  -- هش پسورد جهت ورود بدون پیامک
  tier user_tier_enum NOT NULL DEFAULT 'NORMAL_USER',
  is_verified BOOLEAN NOT NULL DEFAULT FALSE,
  bio VARCHAR(200),
  avatar_url VARCHAR(512),
  banner_url VARCHAR(512),
  website_url VARCHAR(256),
  is_banned BOOLEAN NOT NULL DEFAULT FALSE,
  is_shadowbanned BOOLEAN NOT NULL DEFAULT FALSE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_users_handle ON users(handle);
CREATE INDEX idx_users_phone ON users(phone_number);
```

### ۲. جدول نمادهای بورسی (`stocks`)
```sql
CREATE TABLE stocks (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  symbol VARCHAR(20) UNIQUE NOT NULL,          -- مثال: فولاد
  name_fa VARCHAR(100) NOT NULL,                -- مثال: فولاد مبارکه اصفهان
  isin VARCHAR(12) UNIQUE,                      -- کد ۱۲ رقمی بورسی
  market_category VARCHAR(50) NOT NULL,         -- بورس / فرابورس
  sector VARCHAR(80) NOT NULL,                  -- فلزات اساسی
  last_price BIGINT NOT NULL DEFAULT 0,         -- آخرین معامله (ریال)
  closing_price BIGINT NOT NULL DEFAULT 0,      -- قیمت پایانی (ریال)
  price_change_pct NUMERIC(6, 2) NOT NULL DEFAULT 0.0,
  market_status VARCHAR(30) NOT NULL DEFAULT 'AUTHORIZED',
  last_market_sync TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_stocks_symbol ON stocks(symbol);
CREATE INDEX idx_stocks_name_fa ON stocks(name_fa); -- جستجو با نام کامل شرکت
CREATE INDEX idx_stocks_sector ON stocks(sector);
```

### ۳. جدول توییت‌ها (`posts`)
```sql
CREATE TABLE posts (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  content TEXT NOT NULL,
  reply_to_id UUID REFERENCES posts(id) ON DELETE SET NULL,
  quote_post_id UUID REFERENCES posts(id) ON DELETE SET NULL,
  attachments JSONB DEFAULT '[]'::jsonb, -- آرایه اشیاء: {url, file_type, file_name, size_bytes}
  likes_count INT NOT NULL DEFAULT 0,
  comments_count INT NOT NULL DEFAULT 0,
  reposts_count INT NOT NULL DEFAULT 0,
  views_count INT NOT NULL DEFAULT 0,    -- شمارنده بازدید جهت فیلتر پربازدیدترین‌ها
  is_edited BOOLEAN NOT NULL DEFAULT FALSE,
  is_pinned BOOLEAN NOT NULL DEFAULT FALSE,
  is_flagged BOOLEAN NOT NULL DEFAULT FALSE,
  admin_edited_at TIMESTAMPTZ,
  admin_edited_by UUID REFERENCES users(id) ON DELETE SET NULL,
  admin_edit_reason TEXT,
  deleted_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_posts_user_created ON posts(user_id, created_at DESC) WHERE deleted_at IS NULL;
CREATE INDEX idx_posts_created_at ON posts(created_at DESC) WHERE deleted_at IS NULL;
CREATE INDEX idx_posts_views ON posts(views_count DESC);
CREATE INDEX idx_posts_search_gin ON posts USING gin(to_tsvector('simple', content));
```

### ۴. جدول پیوند توییت به نمادها (`post_stocks`)
```sql
CREATE TABLE post_stocks (
  post_id UUID NOT NULL REFERENCES posts(id) ON DELETE CASCADE,
  stock_id UUID NOT NULL REFERENCES stocks(id) ON DELETE CASCADE,
  PRIMARY KEY (post_id, stock_id)
);

CREATE INDEX idx_post_stocks_stock_id ON post_stocks(stock_id);
```

### ۵. جدول تنظیمات کاربر (`user_settings`)
```sql
CREATE TABLE user_settings (
  user_id UUID PRIMARY KEY REFERENCES users(id) ON DELETE CASCADE,
  theme VARCHAR(10) NOT NULL DEFAULT 'SYSTEM', -- 'LIGHT', 'DARK', 'SYSTEM'
  favorite_sectors TEXT[] NOT NULL DEFAULT '{}',
  favorite_symbols TEXT[] NOT NULL DEFAULT '{}',
  muted_words TEXT[] NOT NULL DEFAULT '{}',
  muted_symbols TEXT[] NOT NULL DEFAULT '{}',
  notify_likes BOOLEAN NOT NULL DEFAULT TRUE,
  notify_replies BOOLEAN NOT NULL DEFAULT TRUE,
  notify_mentions BOOLEAN NOT NULL DEFAULT TRUE,
  notify_follows BOOLEAN NOT NULL DEFAULT TRUE,
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

### ۶. جدول تنظیمات پویای پرتال و رنگ‌های سیستم (`system_configs`)
```sql
CREATE TABLE system_configs (
  key VARCHAR(64) PRIMARY KEY,
  value JSONB NOT NULL,                 -- شامل تم‌های رنگی روز/شب، قوانین و غیره
  description TEXT,
  updated_by UUID REFERENCES users(id) ON DELETE SET NULL,
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

### ۷. جداول هوش مصنوعی Gemini (`ai_stock_summaries` و `ai_morning_briefs`)
```sql
CREATE TABLE ai_stock_summaries (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  stock_id UUID NOT NULL REFERENCES stocks(id) ON DELETE CASCADE,
  summary_payload JSONB NOT NULL,
  post_count_analyzed INT NOT NULL,
  model_version VARCHAR(50) NOT NULL DEFAULT 'gemini-1.5-flash',
  expires_at TIMESTAMPTZ NOT NULL,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_ai_summaries_stock_expires ON ai_stock_summaries(stock_id, expires_at DESC);

CREATE TABLE ai_morning_briefs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  brief_date DATE UNIQUE NOT NULL DEFAULT CURRENT_DATE,
  tweet_id UUID REFERENCES posts(id) ON DELETE SET NULL,
  content TEXT NOT NULL,
  tagged_symbols TEXT[] NOT NULL DEFAULT '{}',
  tokens_used INT NOT NULL DEFAULT 0,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

### ۸. جدول دنبال‌کردن اشخاص (`user_follows`) و نمادها (`stock_follows`)
```sql
CREATE TABLE user_follows (
  follower_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  following_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  PRIMARY KEY (follower_id, following_id)
);

CREATE TABLE stock_follows (
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  stock_id UUID NOT NULL REFERENCES stocks(id) ON DELETE CASCADE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  PRIMARY KEY (user_id, stock_id)
);
```

### ۹. تعاملات: لایک (`likes`)، بازنشر (`reposts`)، اعلان‌ها (`notifications`)
```sql
CREATE TABLE likes (
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  post_id UUID NOT NULL REFERENCES posts(id) ON DELETE CASCADE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  PRIMARY KEY (user_id, post_id)
);

CREATE TABLE reposts (
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  post_id UUID NOT NULL REFERENCES posts(id) ON DELETE CASCADE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  PRIMARY KEY (user_id, post_id)
);

CREATE TABLE notifications (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  recipient_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  actor_id UUID REFERENCES users(id) ON DELETE SET NULL,
  event_type notification_type_enum NOT NULL,
  target_post_id UUID REFERENCES posts(id) ON DELETE CASCADE,
  is_read BOOLEAN NOT NULL DEFAULT FALSE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_notifications_recipient ON notifications(recipient_id, is_read, created_at DESC);
```
