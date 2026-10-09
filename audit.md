# Pre-Implementation Audit Report: BourseTweet Product Blueprint

This document presents a comprehensive, critical audit of the BourseTweet (بورسینو) platform blueprint, as requested. The objective is to identify contradictions, missing specifications, ambiguities, and implementation risks before any application code is written.

---

## A. Overall Assessment

The provided product blueprint is highly structured, professionally organized, and clear about the overarching goals of the platform. The explicit separation of concerns, the decision to decouple market data from social interactions, and the RTL-first Next.js architecture represent a solid technical foundation.

**Strongest Areas:**
- **Product Vision & Core Loop:** The emphasis on user-generated content without automated noise from CADAL (کدال) is clear and well-reasoned.
- **Architectural Modularity:** The domain boundaries are well-defined (Identity, Posts, Stocks, Feed, AI).
- **Decision Logs (ADRs):** Explicitly documenting rejected features (like TradingView integration or sentiment analysis) prevents scope creep and developer assumptions.

**Biggest Risks:**
- **Missing or Underspecified Features:** Several core social platform features, particularly around mentions (`@handle`), content moderation lifecycle, and file uploads, lack explicit flows.
- **Data Model Discrepancies:** There are mismatches between the Entity-Relationship Diagram and the provided DDL schema (e.g., Bookmarks).
- **AI & Edge Cases:** The Gemini AI integration is well-scoped, but edge cases regarding token limits, spam filtering, and multiple-tag overlaps remain ambiguous.
- **Concurrency & Market Status:** Handling real-world Iranian market behaviors (e.g., halted stocks) and high-concurrency edits needs clarification.

---

## B. Critical Blockers

These issues must be resolved before backend or database implementation begins, as they fundamentally affect the data structure and core workflows.

1. **Handling of Suspended/Banned Accounts (Data Lifecycle):**
   - *Issue:* The `users` table includes `is_banned` and `is_shadowbanned` flags, but there is no explicit policy on what happens to a user's existing posts, replies, and likes when they are banned. Do they disappear from all feeds (`soft delete`), or do they remain visible with a "Banned" badge?
   - *Risk:* Without a defined policy, developers might implement incomplete bans where a banned user's content continues to populate the "Explore" or "For You" feeds, causing moderation crises.
2. **Mentions (`@handle`) and Social Graph:**
   - *Issue:* The `notification_type_enum` includes `MENTION`, but there is no specification on how mentions are parsed in the composer, whether there is a limit on mentions per tweet, or if they generate links.
   - *Risk:* Developers might not implement `@handle` extraction, leaving the `MENTION` notification useless, or they might allow an unlimited number of mentions (enabling spam).
3. **Missing "Bookmarks" Schema:**
   - *Issue:* The ER diagram in `04-technical-architecture/02-data-model-and-database-schema.md` shows a `bookmarks` relationship, and `03-features-and-functional-specs/05-interactions-notifications-bookmarks-spec.md` mentions bookmarks in its title, but there is no DDL table for bookmarks in the database schema.
   - *Risk:* Developers will skip implementing bookmarks entirely or create an unoptimized schema.
4. **Institutional Accounts Multiple Operators:**
   - *Issue:* Institutional (`FINANCIAL_INSTITUTION`) and Issuer (`OFFICIAL_ISSUER`) KYC specs mention a single representative/admin phone number.
   - *Risk:* Corporate accounts typically require multiple social media managers. If the system binds the account strictly to one OTP phone number, it becomes a major operational bottleneck for institutions.

---

## C. Important Gaps and Flaws

1. **File Upload Architecture (Orphaned Files):**
   - *Gap:* The database schema stores `attachments` as a `JSONB` array within the `posts` table. There is no central `media` or `files` table.
   - *Flaw:* If a user uploads a 10MB PDF, but abandons the tweet before hitting "Publish", the file remains in the S3 bucket forever. There is no garbage collection mechanism or "pending uploads" state.
2. **AI Digest & Multiple Cashtags:**
   - *Gap:* If a tweet contains multiple cashtags (e.g., `$خودرو` and `$خساپا`), does the AI Digest (Gemini) for *both* stocks process this tweet? Does this risk diluting the digest for one stock if the tweet is primarily about the other?
3. **Stock Status Updates (Halted/Suspended Symbols):**
   - *Gap:* The Iranian stock market frequently halts symbols (ممنوع-متوقف). The DB has `market_status` (`AUTHORIZED`), but there is no mention of how UI handles a halted stock. Should users still be able to tweet about a halted stock?
4. **Rate Limiting & Spam on OTP:**
   - *Gap:* OTP login is standard, but the spec does not define the rate limit for OTP requests. In Iran, SMS costs are high, and unprotected OTP endpoints are prime targets for SMS bombing bots.

---

## D. Missing or Underspecified Features

- **For Public/Unauthenticated Visitors:** Can unauthenticated users view a stock page, the "All" feed, or the Explore page? The spec is entirely silent on unauthenticated access boundaries.
- **For Registered Users:** How does a user delete their entire account (a common privacy requirement)? Is it a soft or hard delete?
- **For Administrators:** How does the admin panel handle appeals for rejected KYC or banned accounts?
- **Content Moderation:** Who reviews flagged tweets? Is there an automated threshold (e.g., hide post after 50 reports) before an admin intervenes?

---

## E. Documentation Conflicts

1. **Tweet Character Limits vs. DB Schema:**
   - *Conflict:* The specs state verified users can post up to 1500 characters, and institutions up to 2500 characters. However, the database schema defines `content TEXT NOT NULL` and relies solely on frontend/API validation. While `TEXT` can hold it, the API spec does not specify backend validation rules for these varying limits based on `user_tier_enum`.
2. **Missing Notification Type:**
   - *Conflict:* The `05-interactions-notifications-bookmarks-spec.md` spec mentions "Quote Tweet" notifications, but `notification_type_enum` only has `REPOST`, `LIKE`, `COMMENT`, `MENTION`, `FOLLOW`, `VERIFICATION_RESULT`. There is no `QUOTE` enum.

---

## F. Implementation Risks

- **15-Minute Edit Window Concurrency:** If a user edits a post 14 minutes in, but their internet drops and the request reaches the server at minute 16, does it fail? What if someone replies to the post while it is being edited?
- **Market Data TSETMC Outages:** TSETMC API is notoriously unstable. The caching strategy (Redis) is mentioned, but what happens if the background worker cannot fetch data for 48 hours? Does the UI show a "Data Unavailable" state or stale prices from two days ago?
- **AI Digest Token Limits:** If a highly discussed stock (like `$شستا`) has 500 tweets in 24 hours, passing all 500 tweets to Gemini will exceed token limits and cost a fortune. The spec says "based on 5 to 10 tweets" but doesn't specify *how* those 5-10 tweets are selected from the 500 (e.g., most liked? most viewed? random?).

---

## G. Questions for Me

Please provide clarity on the following prioritized questions so we can finalize the implementation plan:

1. **Unauthenticated Access:** Should unregistered/logged-out visitors be able to read public tweets, the Explore page, and Stock pages, or is the entire platform locked behind an OTP wall?
   - *Why it matters:* This fundamentally changes routing, SSR strategy in Next.js, and API rate limiting.
2. **Banned/Suspended Users:** When an account is banned, should their existing tweets be hidden from the platform, or left visible with a "Banned" indicator?
   - *Why it matters:* Affects database queries for all feeds and stock pages.
3. **AI Selection Logic:** For the AI Stock Digest, if a stock has hundreds of tweets in 24 hours, how should the backend select the 5-10 tweets to send to Gemini?
   - *Options:* (A) Top by Likes, (B) Top by Views, (C) Only from Verified users, (D) Chronologically latest.
4. **Institutional Accounts:** Will corporate accounts be operated by a single person (one OTP number), or do we need a "Team/Multi-User" feature for institutions?
   - *Why it matters:* If team access is needed later, the database architecture needs an `organization_members` table now, otherwise it will require a massive refactor.
5. **Orphaned File Attachments:** How should we handle files uploaded during tweet composition if the user never clicks "Publish"?
   - *Options:* (A) Cron job to delete unlinked files after 24h, (B) Implement a `media` table to track upload state.
6. **Mentions & Limits:** Do we officially support `@mentions`? If so, is there a limit on how many users can be mentioned in one tweet to prevent spam?
7. **Bookmarks:** Are Bookmarks officially part of the MVP? (They are in the DB diagram and title, but missing from DB schema).
8. **Account Deletion:** Do we need a "Delete Account" button for users in the MVP to comply with privacy standards?

---

## H. Non-Blocking Technical Decisions

*These will be handled by the development team and do not require your immediate input, but are noted for completeness:*
- **Search Pagination:** The API will use cursor-based pagination for search results to ensure performance over OFFSET/LIMIT.
- **File Upload Flow:** We will use Pre-signed URLs for S3 uploads directly from the frontend to save backend bandwidth.
- **SMS Rate Limiting:** We will implement a strict Redis-based rate limit for OTP requests (e.g., max 3 requests per 5 minutes per IP/Phone) to prevent financial abuse.

---

## I. Audit Coverage

- **Documents Reviewed:** All 26 markdown files in the `bourse-social-docs` repository, including Strategy, UX, Technical Architecture, Features, AI Specs, and Roadmap.
- **Incomplete Parts:** None. The audit was conducted thoroughly across all provided documentation.
