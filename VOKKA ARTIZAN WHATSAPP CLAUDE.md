# CLAUDE.md — Vokka Project Context

## What is Vokka?

Vokka is a WhatsApp-based custom tailoring marketplace for the Nigerian market. Buyers send a photo of any outfit on WhatsApp, get matched with skilled artisans who competitively quote, pay securely via escrow, and have the finished piece delivered to their door — all without visiting a tailor.

**Website:** vokka.tech
**Business email:** hello@vokka.tech
**GitHub:** github.com/abd010x/vokka-v3
**CAC:** Registered
**NDPC:** Pending registration (₦25,000 small business fee)

---

## Repository Structure

```
vokka-v3/
├── CLAUDE.md                               ← this file
├── README.md
├── .gitignore
├── frontend/
│   └── public/
│       └── index.html                      ← landing page (deployed to Vercel)
├── docs/
│   ├── artizan-technical-architecture.md   ← system design v1.1 with Claude AI layer
│   └── artizan-whatsapp-scripts.md         ← 18 complete conversation flows
└── backend/                                ← (to be built)
    ├── src/
    │   ├── server.js                       ← Express server + webhook endpoint
    │   ├── handlers/                       ← State machine handlers
    │   ├── services/
    │   │   ├── claude-nlp.js               ← Claude API NLP intent parser
    │   │   ├── claude-vision.js            ← Claude API image recognition
    │   │   ├── whatsapp.js                 ← BSP message sending
    │   │   ├── payment.js                  ← Bank transfer / wallet logic
    │   │   └── delivery.js                 ← In-house delivery coordination
    │   ├── models/                         ← Sequelize/Prisma models
    │   ├── jobs/                           ← BullMQ job processors
    │   └── utils/
    ├── package.json
    └── .env.example
```

---

## Tech Stack (Final Decisions)

| Layer | Technology | Status |
|-------|-----------|--------|
| **Frontend** | Static HTML, Fraunces + Plus Jakarta Sans, Lucide icons | ✅ Built, deploy to Vercel |
| **Backend** | Node.js + Express | 🔲 To build |
| **Database** | PostgreSQL | 🔲 Railway managed |
| **Cache/State** | Redis | 🔲 Railway managed |
| **Job Queue** | BullMQ | 🔲 To build |
| **Hosting** | Railway (backend + Redis + PostgreSQL + workers) | 🔲 To set up |
| **Frontend Hosting** | Vercel (hobby plan, connected to GitHub abd010x) | ✅ Ready |
| **WhatsApp** | WhatsApp Business API via BSP (360dialog or Twilio) | 🔲 Pending phone number + Meta verification |
| **AI** | Claude API (claude-sonnet-4-6) — NLP + image recognition | 🔲 To integrate |
| **Banking** | Direct bank transfer (PAUSED: 9PSB for later) | 🔲 Manual for now |
| **Delivery** | In-house coordination (PAUSED: Kwik API for later) | 🔲 Manual for now |
| **Images** | Cloudinary | 🔲 To set up |

---

## Brand Identity

### Name
- Product name: **Vokka** (lowercase in logo: `vokka.`)
- The terracotta-colored dot after the name is part of the logo

### Colors
```css
--parchment: #F8F3EB;       /* page background */
--parchment-deep: #EFE7DA;  /* section backgrounds */
--espresso: #1C1410;         /* primary text */
--cocoa: #3D2B22;            /* dark sections, footer */
--terracotta: #B85C38;       /* primary accent, CTAs */
--terracotta-light: #F5DDD3;
--terracotta-dark: #943E22;
--adire: #1B5E4B;            /* section labels, secondary accent */
--adire-light: #D4EAE2;
--brass: #C79B4A;            /* highlights, gold accents */
--brass-light: #F7EDD5;
--stone: #9C8B7A;            /* secondary text */
--sand: #D6CBBE;             /* borders, dividers */
```

### Typography
- **Display/headings:** Fraunces (Google Fonts) — variable serif, organic handcrafted feel
- **Body text:** Plus Jakarta Sans (Google Fonts) — clean, modern, slightly rounded

### Icons
- **Library:** Lucide exclusively — `lucide-react` for React, Lucide CDN for static HTML
- **Style:** Outline/stroke at 24px default
- **Stroke width:** 1.5px for body elements, 2px for nav/toolbar
- **Color:** Inherits from parent via `currentColor`
- **NEVER** mix with other icon libraries (no Font Awesome, no Heroicons, no emoji as icons)

### Woven Divider
A repeating horizontal pattern using terracotta → adire → brass, evoking textile/weaving patterns.

---

## Core Business Logic

### Service Fee
- **20% service fee** added ON TOP of the artisan's quoted price
- Artisan receives their full quoted amount — fee is never deducted from them
- Example: Artisan quotes ₦18,000 → Buyer pays ₦18,000 + ₦3,600 (20%) + delivery = ₦21,600 + delivery

### Payment Flow (Escrow)
```
Buyer pays full amount → held in escrow
  ↓
40% of artisan quote → disbursed immediately to artisan (production advance)
  ↓
Artisan produces garment
  ↓
Delivery to buyer
  ↓
Buyer reviews artisan + Vokka (semi-compulsory)
  ↓
60% of artisan quote → released to artisan AFTER review
```

**Critical rule:** Review happens BEFORE the 60% release. This makes the review functionally compulsory — the artisan's payment is held until the buyer reviews.

**Review nudge message:** "Your artisan is waiting to be paid — a quick review releases their payment."

**Auto-approval:** If buyer doesn't respond within 24 hours after delivery, order is auto-approved. Review nudge sent at 24hrs and 48hrs. After 48hrs, payment auto-releases even without review.

### Payment Method (Current — MVP)
- Direct bank transfer to Vokka's bank account
- No MFB/wallet integration yet (9PSB paused for later)
- Manual reconciliation of payments initially

### Delivery (Current — MVP)
- In-house coordination (no Kwik API yet, paused for later)
- Vokka coordinates between artisan and buyer
- Delivery cost based on distance and quantity, charged to buyer at checkout
- Flat zone pricing for Lagos:
  - Same-area: ₦1,000 - ₦1,500
  - Mainland → Mainland: ₦1,500 - ₦2,500
  - Mainland ↔ Island: ₦2,500 - ₦4,000
  - Island → Island: ₦1,200 - ₦2,000

### Matching Engine
- Buyer sends design image → Claude Vision identifies garment type + details
- Artisans matched by: garment type, location (area in Lagos), price tier
- Maximum 7 artisans matched per order
- **15-minute quote window** — send results when either all respond or time expires
- Artisans see buyer's budget range but NOT each other's quotes (blind bidding)
- Minimum 1 quote to proceed; if zero, widen match or retry

### Measurements
- Collected at FIRST ORDER (not during onboarding — reduces friction)
- Saved permanently, reused for all future orders
- Four options:
  1. AI quick estimate (Beta) — send full-body photo, Claude Vision estimates
  2. WhatsApp video call (Fastest) — agent guides measurement via video
  3. Agent home visit (Most accurate) — free, trained agent visits
  4. Manual input — buyer enters own measurements

### Dispute Resolution
- Buyer has 24 hours after delivery to approve or raise concern
- Dispute types: fit issue, quality issue, wrong design, damaged
- Resolution paths:
  - **Minor fix:** Artisan re-does at no cost, re-delivered free
  - **Major failure:** 60% + service fee refunded to buyer. 40% advance is non-refundable (materials already used)
  - **Delivery damage:** Vokka covers replacement delivery
- Three-strike policy: 3 major disputes → artisan suspension review
- Resolution SLA: acknowledged within 2 hours, resolved within 48 hours

### Repeat Orders
- Default: every order goes through the matching flow (competitive quotes)
- Exception: buyer can explicitly request a previous artisan → direct single quote
- Repeat orders skip onboarding and measurements (already saved)

---

## Account Types

### Personal Account (Buyers)
- Onboarding: name → gender → location (area in Lagos)
- Phone auto-captured from WhatsApp
- Wallet for payments (later, when MFB integrated)

### Business Account (Brands/Artisans)
- Onboarding: brand name → location → specialties (multi-select garment types) → price range → portfolio (3-5 photos) → weekly capacity
- Manual verification by Vokka team (24hr review)
- Starting with 10 curated artisans before opening to buyers
- Artisans informed during onboarding: final 60% is tied to buyer completing review

---

## AI Intelligence Layer (Claude API)

### NLP — Natural Language Understanding
- Model: claude-sonnet-4-6
- Uses Claude's tool-use feature with structured `parse_message` tool
- System prompt trained on: Nigerian Pidgin, Yoruba-English mix, currency shorthand (20k = ₦20,000), garment vocabulary (agbada, kaftan, senator), fabric types (ankara, aso-oke, adire)
- Extracts: intent, order fields (fabric, budget, timeline, occasion), selected options, star ratings, production status keywords, artisan names for re-orders
- **Hybrid approach:** Claude feeds structured data INTO the state machine. If all fields extracted from one message, state skips ahead. If partial, asks for remaining fields only.
- **Fallback:** If Claude API down or confidence < 0.5, revert to rigid numbered-option prompts

### Image Recognition — Design Analysis
- Model: claude-sonnet-4-6 with vision
- Uses `analyze_design` tool to extract: garment_type, detected_fabric, style_details, complexity, gender_target
- Pre-fills order fields from image analysis, confirms with buyer
- Reduces order flow from 8 messages to 2-3

### Body Scan (Beta)
- Model: claude-sonnet-4-6 with vision
- Estimates measurements from full-body photo
- Returns values with ± margin of error
- Clearly flagged as beta estimate, recommends video call or agent for precision

### Voice Notes (PAUSED — Version 2)
- Not implemented in v1
- Future: Whisper API for speech-to-text → feed into NLP handler

---

## Conversation State Machine

State stored in Redis: `state:{phone_number}` → JSON payload

### Buyer States
```
NEW_USER → ONBOARD_ACCOUNT_TYPE → ONBOARD_NAME → ONBOARD_GENDER → ONBOARD_LOCATION → IDLE

IDLE → (sends image) → ORDER_DETAILS → ORDER_FABRIC* → ORDER_BUDGET* → ORDER_TIMELINE* → ORDER_OCCASION*
  → MEASURE_CHOICE (first order only) → MEASURE_INPUT → MEASURE_CONFIRM
  → AWAITING_QUOTES (15 min timer) → QUOTE_SELECT → PAYMENT_CONFIRM → IDLE

DELIVERED → APPROVAL_PENDING (24hr timer)
  → REVIEW_ARTISAN → REVIEW_PLATFORM → IDLE (60% released)
  → DISPUTE_TYPE → DISPUTE_PHOTO → DISPUTE_PENDING

* = skipped if NLP extracted from previous message
```

### Brand States
```
ONBOARD_BRAND_NAME → ONBOARD_BRAND_LOCATION → ONBOARD_BRAND_SPECIALTIES
  → ONBOARD_BRAND_PRICE_RANGE → ONBOARD_BRAND_PORTFOLIO → ONBOARD_BRAND_CAPACITY
  → PENDING_VERIFICATION → IDLE

IDLE → QUOTE_RESPOND → IDLE
PRODUCTION → status keywords (cutting/sewing/finishing/ready) → IDLE
```

---

## Database Tables

- **users** — id, phone, type (buyer/brand), name, gender, location, wallet_id, status, rating
- **brand_profiles** — user_id, brand_name, specialties[], price range, portfolio_urls[], capacity, verified
- **measurements** — user_id, method, chest/waist/hips/shoulder/arm/dress/neck/inseam, raw_data
- **orders** — buyer_id, brand_id, status, design details, pricing, payment tracking, delivery tracking
- **quotes** — order_id, brand_id, price, days, status, expires_at (15 min)
- **reviews** — order_id, reviewer_id, artisan_rating, artisan_comment, platform_rating
- **disputes** — order_id, type, description, photo_urls, status, resolution
- **wallet_transactions** — user_id, order_id, type, amount, reference, status
- **conversation_history** — phone, role, content, message_type (for Claude NLP context)

Full SQL schemas in `docs/artizan-technical-architecture.md`

---

## Async Jobs (BullMQ)

- **match_artisans** — triggered when buyer completes order details + measurements
- **collect_quotes** — fires 15 min after matching, sends results to buyer
- **process_payment** — debits buyer, holds escrow, disburses 40% advance
- **book_delivery** — triggered when artisan marks "ready" (manual coordination for now)
- **release_payment** — triggered when buyer completes review, releases 60%
- **auto_approve** — 24hr delayed job after delivery confirmation
- **review_nudge** — 24hr and 48hr reminders after delivery

---

## WhatsApp Message Templates (Need Meta Approval)

1. **order_status_update** (Utility) — production status notifications
2. **delivery_notification** (Utility) — delivery ETA alerts
3. **review_reminder** (Utility) — review nudge with payment release messaging
4. **quote_request** (Utility) — sent to artisans with order details
5. **payment_received** (Utility) — 40% advance confirmation to artisans

All utility templates (highest approval rate). Marketing templates deferred.

---

## Launch Sequence

1. ✅ CAC registered
2. ✅ Business email (hello@vokka.tech)
3. ✅ GitHub repo (abd010x/vokka-v3)
4. ✅ Vercel connected to GitHub
5. 🔲 Deploy landing page to Vercel at vokka.tech
6. 🔲 Get dedicated phone number (fresh SIM)
7. 🔲 Set up Meta Business Manager + submit verification
8. 🔲 Register with NDPC (₦25,000)
9. 🔲 Write privacy policy + terms of service
10. 🔲 Set up Railway (backend + PostgreSQL + Redis)
11. 🔲 Choose BSP (360dialog or Twilio)
12. 🔲 Submit message templates for Meta approval
13. 🔲 Build chatbot backend
14. 🔲 Integrate Claude API (NLP + Vision)
15. 🔲 Onboard 10 curated artisans
16. 🔲 Test complete flow (team as buyers)
17. 🔲 Soft launch (20-30 invited buyers)
18. 🔲 Open registration via landing page

---

## Deferred (Post-Launch)

- 9PSB Wallet-as-a-Service integration (replace direct bank transfers)
- Kwik delivery API integration (replace in-house delivery)
- Voice note processing (Whisper API → Claude NLP)
- NDPC formal DPCO audit and certification
- Admin dashboard (disputes, artisan vetting, analytics)
- Marketing message templates (re-engagement campaigns)
