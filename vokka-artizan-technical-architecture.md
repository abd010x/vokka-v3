# Artizan — Chatbot Technical Architecture
## System Design Document v1.1 — with Claude AI Intelligence Layer

---

## 1. TECH STACK

### Core
- **Runtime:** Node.js 20+ (recommended) or Python 3.11+ (FastAPI)
- **Framework:** Express.js (Node) or FastAPI (Python)
- **Database:** PostgreSQL 15+ (relational data: users, orders, quotes, reviews)
- **Cache/State:** Redis 7+ (conversation state machine, session data, rate limiting)
- **Job Queue:** BullMQ (Node) or Celery (Python) — async tasks: matching, notifications, payment triggers, delivery booking
- **Image Storage:** Cloudinary (design images, portfolio photos, measurement photos)
- **Hosting:** Railway or Render (MVP) → AWS/GCP (scale)

### External APIs
- **WhatsApp:** 360dialog or Twilio (BSP for WhatsApp Business API)
- **Banking:** 9PSB Wallet-as-a-Service (wallets, escrow, disbursements)
- **Delivery:** Kwik API (delivery pricing, booking, tracking)
- **AI Intelligence:** Claude API (claude-sonnet-4-6) — NLP intent parsing, natural language understanding, design image recognition, body measurement estimation (beta)

---

## 2. AI INTELLIGENCE LAYER (Claude API)

Claude sits between the incoming message and the state machine. It transforms messy human input into structured data the bot can act on. Two roles:

### 2A. NLP — Natural Language Understanding

**Problem solved:** Without NLP, the bot asks 4 sequential questions (fabric? budget? timeline? occasion?) and expects numbered replies. With Claude, a buyer can type a single natural message like "I want ankara for my sister's wedding next month, budget around 20k" and Claude extracts all four fields at once.

**How it works:**

```javascript
// claude-nlp.js — Intent parser
const Anthropic = require('@anthropic-ai/sdk');
const client = new Anthropic();

async function parseIntent(userMessage, currentState, conversationHistory) {
  const response = await client.messages.create({
    model: 'claude-sonnet-4-6',
    max_tokens: 500,
    system: `You are the AI brain of Artizan, a WhatsApp-based custom tailoring service in Lagos, Nigeria.

Your job: parse the user's WhatsApp message and extract structured data.
You understand English, Nigerian Pidgin, Yoruba-English mix, and informal text speak.
The user's current conversation state is: ${currentState}

RULES:
- Return ONLY valid JSON, no explanation
- Extract every field you can identify from the message
- If a field is ambiguous, set it to null
- Interpret Nigerian currency shorthand: "20k" = 20000, "50k" = 50000
- Interpret informal dates: "next month" = relative date, "2 weeks" = 14 days
- Understand garment types: agbada, kaftan, senator, buba, iro, aso-ebi, etc.
- Understand fabric types: ankara, lace, aso-oke, adire, cotton, silk, etc.
- Recognize commands: "help", "menu", "wallet", "balance", "orders", "status"
- Recognize direct artisan requests: "I want [name] for this", "same tailor", "send to [name]"`,

    messages: [
      ...conversationHistory.slice(-4),  // last 4 messages for context
      {
        role: 'user',
        content: userMessage
      }
    ],
    tools: [{
      name: 'parse_message',
      description: 'Parse a WhatsApp message into structured intent data',
      input_schema: {
        type: 'object',
        properties: {
          intent: {
            type: 'string',
            enum: [
              'new_order',          // wants to sew something
              'order_details',      // providing fabric/budget/timeline/occasion
              'select_option',      // picking from numbered options (1, 2, 3)
              'confirm',            // yes, ok, sure, proceed
              'decline',            // no, cancel, nah
              'measurement_choice', // choosing measurement method
              'measurement_input',  // providing actual measurements
              'quote_select',       // selecting an artisan from quotes
              'status_update',      // brand sending production keyword
              'review',             // star rating or review text
              'dispute',            // raising a concern
              'command',            // help, menu, wallet, orders
              'direct_reorder',     // wants a specific artisan again
              'greeting',           // hi, hello, hey
              'unclear'             // can't determine intent
            ]
          },
          command: {
            type: 'string',
            enum: ['help', 'menu', 'wallet', 'balance', 'orders', 'status'],
            description: 'If intent is command, which command'
          },
          order_fields: {
            type: 'object',
            properties: {
              fabric: { type: 'string' },
              budget_min: { type: 'integer' },
              budget_max: { type: 'integer' },
              timeline: { type: 'string' },
              occasion: { type: 'string' },
              garment_type: { type: 'string' }
            }
          },
          selected_option: {
            type: 'integer',
            description: 'If user selected a numbered option (1-7)'
          },
          star_rating: {
            type: 'integer',
            description: 'If user gave a star rating (1-5)'
          },
          review_text: {
            type: 'string',
            description: 'Review comment text'
          },
          artisan_name: {
            type: 'string',
            description: 'If user named a specific artisan for direct reorder'
          },
          production_status: {
            type: 'string',
            enum: ['cutting', 'sewing', 'finishing', 'ready'],
            description: 'If brand sent a status update keyword'
          },
          raw_measurements: {
            type: 'object',
            description: 'If user typed measurement values'
          },
          confidence: {
            type: 'number',
            description: 'Confidence in parsing (0-1)'
          }
        },
        required: ['intent', 'confidence']
      }
    }],
    tool_choice: { type: 'tool', name: 'parse_message' }
  });

  // Extract the tool use result
  const toolUse = response.content.find(c => c.type === 'tool_use');
  return toolUse.input;
}
```

**What this enables:**

Without NLP (rigid):
```
Bot: What fabric?
User: 1 (ankara)
Bot: Budget range?
User: 2 (₦15k-₦30k)
Bot: When do you need it?
User: 2 weeks
Bot: Occasion?
User: 1 (wedding)
```
→ 8 messages, feels like a form

With NLP (natural):
```
User: I want ankara for my sister's wedding, around 20k, need it in 2 weeks
Bot: Got it! Ankara dress for a wedding, budget ₦20,000, needed in 2 weeks.
     Let me match you with artisans...
```
→ 2 messages, feels like a conversation

**Hybrid approach — NLP + state machine:**

Claude doesn't replace the state machine. It feeds INTO it. If Claude extracts all 4 order fields from one message, the state jumps from IDLE straight to MEASURE_CHOICE (or AWAITING_QUOTES if measurements exist). If Claude only extracts 2 fields, the state advances to the first missing field and asks for it. The state machine remains the source of truth for flow control.

```javascript
// After Claude parses, the state handler checks completeness:
async function handleOrderDetails(phone, parsed, stateData) {
  const fields = { ...stateData, ...parsed.order_fields };

  const missing = [];
  if (!fields.fabric) missing.push('fabric');
  if (!fields.budget_min) missing.push('budget');
  if (!fields.timeline) missing.push('timeline');

  if (missing.length === 0) {
    // All fields collected — skip ahead
    return {
      nextState: hasMeasurements(phone) ? 'AWAITING_QUOTES' : 'MEASURE_CHOICE',
      data: fields,
      reply: formatOrderSummary(fields),
      jobs: hasMeasurements(phone) ? [{ type: 'match_artisans', payload: fields }] : []
    };
  }

  // Ask for the first missing field only
  return {
    nextState: `ORDER_${missing[0].toUpperCase()}`,
    data: fields,
    reply: PROMPTS[missing[0]]
  };
}
```

### 2B. Image Recognition — Design Analysis

**Problem solved:** When a buyer sends a photo of what they want sewn, Claude Vision identifies the garment type, fabric patterns, and style details automatically — no need to ask "what garment is this?"

```javascript
// claude-vision.js — Design image analyzer
async function analyzeDesignImage(imageUrl) {
  const response = await client.messages.create({
    model: 'claude-sonnet-4-6',
    max_tokens: 300,
    system: `You are an expert Nigerian fashion analyst for Artizan, a custom tailoring service.
Analyze the clothing design image and extract structured details.
You are deeply familiar with Nigerian and West African fashion:
agbada, kaftan, senator suit, buba and sokoto, iro and buba, aso-oke,
ankara styles, lace gowns, English suits, casual wear, etc.

Return ONLY JSON. No explanation.`,

    messages: [{
      role: 'user',
      content: [
        {
          type: 'image',
          source: { type: 'url', url: imageUrl }
        },
        {
          type: 'text',
          text: 'Analyze this clothing design image.'
        }
      ]
    }],
    tools: [{
      name: 'analyze_design',
      description: 'Extract garment details from a design photo',
      input_schema: {
        type: 'object',
        properties: {
          garment_type: {
            type: 'string',
            enum: ['agbada', 'kaftan', 'senator_suit', 'buba_sokoto',
                   'iro_buba', 'gown', 'dress', 'blouse_skirt',
                   'english_suit', 'casual', 'aso_ebi', 'other'],
            description: 'Primary garment category'
          },
          garment_description: {
            type: 'string',
            description: 'Brief natural description for artisans, max 30 words'
          },
          detected_fabric: {
            type: 'string',
            enum: ['ankara', 'lace', 'aso_oke', 'adire', 'cotton',
                   'silk', 'chiffon', 'brocade', 'guinea', 'unknown'],
            description: 'Fabric if identifiable from the image'
          },
          style_details: {
            type: 'array',
            items: { type: 'string' },
            description: 'Notable details: embroidery, beading, cap style, collar type, etc.'
          },
          complexity: {
            type: 'string',
            enum: ['simple', 'moderate', 'complex', 'highly_complex'],
            description: 'Estimated production complexity'
          },
          gender_target: {
            type: 'string',
            enum: ['male', 'female', 'unisex']
          },
          confidence: {
            type: 'number',
            description: 'Confidence in analysis (0-1)'
          }
        },
        required: ['garment_type', 'garment_description', 'complexity', 'confidence']
      }
    }],
    tool_choice: { type: 'tool', name: 'analyze_design' }
  });

  const toolUse = response.content.find(c => c.type === 'tool_use');
  return toolUse.input;
}
```

**How it changes the order flow:**

Without image recognition:
```
User: [sends photo]
Bot: What type of garment is this?
User: Agbada
Bot: What fabric?
...
```

With image recognition:
```
User: [sends photo]
Bot: Beautiful! I see an agbada with embroidered collar detail
     in what looks like guinea brocade. Complexity: moderate.

     A few quick questions:
     - Fabric: Guinea brocade, or something different?
     - Budget range?
     - When do you need it by?
```

The bot pre-fills what it can detect, confirms with the buyer, and only asks for what it genuinely can't infer from the image. Faster, smarter, fewer messages.

### 2C. AI Body Scan (Beta)

Uses Claude Vision to estimate body measurements from a full-body photo.

```javascript
async function estimateBodyMeasurements(imageUrl, gender) {
  const response = await client.messages.create({
    model: 'claude-sonnet-4-6',
    max_tokens: 400,
    system: `You are a body measurement estimation tool for a tailoring service.
Given a full-body photo, estimate standard tailoring measurements in inches.
Gender: ${gender}

IMPORTANT CAVEATS:
- These are ROUGH ESTIMATES only. Flag this clearly.
- Accuracy depends on clothing tightness, camera angle, and distance.
- Always recommend video call or agent visit for precision.
- Return measurements in inches with a ± range.`,

    messages: [{
      role: 'user',
      content: [
        { type: 'image', source: { type: 'url', url: imageUrl } },
        { type: 'text', text: 'Estimate body measurements from this full-body photo.' }
      ]
    }],
    tools: [{
      name: 'estimate_measurements',
      description: 'Estimate body measurements from a photo',
      input_schema: {
        type: 'object',
        properties: {
          chest:        { type: 'object', properties: { value: {type:'number'}, margin: {type:'number'} }},
          waist:        { type: 'object', properties: { value: {type:'number'}, margin: {type:'number'} }},
          hips:         { type: 'object', properties: { value: {type:'number'}, margin: {type:'number'} }},
          shoulder:     { type: 'object', properties: { value: {type:'number'}, margin: {type:'number'} }},
          arm_length:   { type: 'object', properties: { value: {type:'number'}, margin: {type:'number'} }},
          dress_length: { type: 'object', properties: { value: {type:'number'}, margin: {type:'number'} }},
          overall_confidence: { type: 'string', enum: ['low','medium','high'] },
          warnings: { type: 'array', items: { type: 'string' } }
        }
      }
    }],
    tool_choice: { type: 'tool', name: 'estimate_measurements' }
  });

  return response.content.find(c => c.type === 'tool_use').input;
}
```

### 2D. Claude API Cost Estimation

At launch scale (10 artisans, ~50 orders/week):

| Call type | Per order | Tokens/call | Cost/call | Weekly cost |
|-----------|-----------|-------------|-----------|-------------|
| NLP intent parsing | ~6 calls | ~800 tokens | ~$0.003 | $0.90 |
| Image analysis | 1 call | ~1500 tokens | ~$0.006 | $0.30 |
| Body scan (beta) | 0.3 calls | ~2000 tokens | ~$0.008 | $0.12 |
| **Total** | | | | **~$1.32/week** |

At 500 orders/week: ~$13/week. Claude API costs are negligible compared to WhatsApp message fees and payment processing.

### 2E. Fallback Strategy

If Claude API is down or returns low confidence:
1. **NLP fallback:** Revert to rigid numbered-option prompts (the original state machine behavior)
2. **Image fallback:** Ask the buyer to describe the garment type manually
3. **Body scan fallback:** Offer video call or agent visit instead

```javascript
async function safeParseIntent(message, state, history) {
  try {
    const parsed = await parseIntent(message, state, history);
    if (parsed.confidence < 0.5) {
      return { intent: 'unclear', confidence: 0 };  // fallback to rigid prompts
    }
    return parsed;
  } catch (error) {
    logger.warn('Claude API failed, using rigid fallback', { error });
    return rigidParse(message, state);  // regex/keyword matching
  }
}
```

---

## 3. DATABASE SCHEMA

### users
```sql
CREATE TABLE users (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  phone         VARCHAR(15) UNIQUE NOT NULL,  -- WhatsApp number
  type          VARCHAR(10) NOT NULL,          -- 'buyer' | 'brand'
  name          VARCHAR(100),
  gender        VARCHAR(20),
  location      VARCHAR(100),                 -- area in Lagos
  wallet_id     VARCHAR(50),                  -- 9PSB wallet ID
  account_number VARCHAR(20),                 -- 9PSB virtual account
  status        VARCHAR(20) DEFAULT 'active', -- 'active' | 'suspended' | 'pending_review'
  rating_avg    DECIMAL(2,1) DEFAULT 0,
  rating_count  INTEGER DEFAULT 0,
  created_at    TIMESTAMPTZ DEFAULT NOW(),
  updated_at    TIMESTAMPTZ DEFAULT NOW()
);
```

### brand_profiles
```sql
CREATE TABLE brand_profiles (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id         UUID REFERENCES users(id),
  brand_name      VARCHAR(100) NOT NULL,
  specialties     TEXT[],                       -- array: ['agbada','gown','ankara']
  price_min       INTEGER,
  price_max       INTEGER,
  portfolio_urls  TEXT[],                       -- Cloudinary URLs
  weekly_capacity INTEGER DEFAULT 5,
  verified        BOOLEAN DEFAULT FALSE,
  verified_at     TIMESTAMPTZ,
  created_at      TIMESTAMPTZ DEFAULT NOW()
);
```

### measurements
```sql
CREATE TABLE measurements (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id     UUID REFERENCES users(id),
  method      VARCHAR(20),    -- 'ai_scan' | 'video_call' | 'agent_visit' | 'manual'
  chest       DECIMAL(4,1),
  waist       DECIMAL(4,1),
  hips        DECIMAL(4,1),
  shoulder    DECIMAL(4,1),
  arm_length  DECIMAL(4,1),
  dress_length DECIMAL(4,1),
  neck        DECIMAL(4,1),
  inseam      DECIMAL(4,1),
  raw_data    JSONB,          -- additional/custom measurements
  created_at  TIMESTAMPTZ DEFAULT NOW(),
  updated_at  TIMESTAMPTZ DEFAULT NOW()
);
```

### orders
```sql
CREATE TABLE orders (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  order_number    SERIAL,
  buyer_id        UUID REFERENCES users(id),
  brand_id        UUID REFERENCES users(id),      -- NULL until artisan selected
  status          VARCHAR(30) NOT NULL DEFAULT 'pending',
  -- Status values: pending | quoting | matched | paid | production |
  --                ready | dispatched | delivered | approved | reviewed | disputed | refunded

  -- Design details
  design_image_url  TEXT,
  garment_type      VARCHAR(50),
  fabric            VARCHAR(50),
  budget_min        INTEGER,
  budget_max        INTEGER,
  deadline          DATE,
  occasion          VARCHAR(50),

  -- Pricing
  artisan_price     INTEGER,          -- artisan's quoted price
  service_fee       INTEGER,          -- 15% of artisan_price
  delivery_fee      INTEGER,          -- from Kwik API
  total_price       INTEGER,          -- artisan + service + delivery

  -- Payment tracking
  advance_amount    INTEGER,          -- 40% of artisan_price
  advance_paid_at   TIMESTAMPTZ,
  final_amount      INTEGER,          -- 60% of artisan_price
  final_paid_at     TIMESTAMPTZ,

  -- Delivery
  delivery_id       VARCHAR(50),      -- Kwik booking reference
  delivery_status   VARCHAR(30),
  delivered_at      TIMESTAMPTZ,

  -- Approval
  approved_at       TIMESTAMPTZ,
  auto_approved     BOOLEAN DEFAULT FALSE,

  -- Timestamps
  created_at        TIMESTAMPTZ DEFAULT NOW(),
  updated_at        TIMESTAMPTZ DEFAULT NOW()
);
```

### quotes
```sql
CREATE TABLE quotes (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  order_id    UUID REFERENCES orders(id),
  brand_id    UUID REFERENCES users(id),
  price       INTEGER NOT NULL,
  days        INTEGER NOT NULL,
  status      VARCHAR(20) DEFAULT 'pending',  -- 'pending' | 'submitted' | 'selected' | 'expired'
  submitted_at TIMESTAMPTZ,
  expires_at  TIMESTAMPTZ,                    -- 15 min from request sent
  created_at  TIMESTAMPTZ DEFAULT NOW()
);
```

### reviews
```sql
CREATE TABLE reviews (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  order_id        UUID REFERENCES orders(id),
  reviewer_id     UUID REFERENCES users(id),    -- the buyer
  artisan_rating  INTEGER CHECK (artisan_rating BETWEEN 1 AND 5),
  artisan_comment TEXT,
  platform_rating INTEGER CHECK (platform_rating BETWEEN 1 AND 5),
  platform_comment TEXT,
  created_at      TIMESTAMPTZ DEFAULT NOW()
);
```

### disputes
```sql
CREATE TABLE disputes (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  order_id    UUID REFERENCES orders(id),
  type        VARCHAR(30),     -- 'fit' | 'quality' | 'wrong_design' | 'damaged'
  description TEXT,
  photo_urls  TEXT[],
  status      VARCHAR(20) DEFAULT 'open',  -- 'open' | 'investigating' | 'resolved_fix' | 'resolved_refund'
  resolution  TEXT,
  resolved_at TIMESTAMPTZ,
  created_at  TIMESTAMPTZ DEFAULT NOW()
);
```

### wallet_transactions
```sql
CREATE TABLE wallet_transactions (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id         UUID REFERENCES users(id),
  order_id        UUID REFERENCES orders(id),
  type            VARCHAR(20),  -- 'escrow_in' | 'advance_out' | 'final_out' | 'fee_out' | 'refund' | 'fund' | 'withdraw'
  amount          INTEGER NOT NULL,
  reference       VARCHAR(50),  -- 9PSB transaction ref
  status          VARCHAR(20),  -- 'pending' | 'completed' | 'failed'
  created_at      TIMESTAMPTZ DEFAULT NOW()
);
```

### conversation_history (for Claude NLP context)
```sql
CREATE TABLE conversation_history (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id     UUID REFERENCES users(id),
  phone       VARCHAR(15) NOT NULL,
  role        VARCHAR(10) NOT NULL,     -- 'user' | 'assistant'
  content     TEXT NOT NULL,
  message_type VARCHAR(20),             -- 'text' | 'image' | 'interactive'
  image_url   TEXT,                      -- if message was an image
  created_at  TIMESTAMPTZ DEFAULT NOW()
);
-- Index for fast lookup of recent messages per user
CREATE INDEX idx_conv_history_phone ON conversation_history(phone, created_at DESC);
-- Auto-cleanup: keep only last 20 messages per user (cron job or trigger)
```

---

## 4. CONVERSATION STATE MACHINE

State is stored in Redis with key `state:{phone_number}` and a JSON payload:

```json
{
  "state": "ORDER_FABRIC",
  "userId": "uuid",
  "data": {
    "designImageUrl": "https://...",
    "garmentType": "agbada"
  },
  "orderId": null,
  "lastMessageAt": "2026-06-09T10:03:00Z",
  "ttl": 86400
}
```

### Buyer states
```
NEW_USER
  → ONBOARD_ACCOUNT_TYPE
  → ONBOARD_NAME
  → ONBOARD_GENDER
  → ONBOARD_LOCATION
  → IDLE

IDLE (waiting for design image or command)
  → ORDER_DETAILS       (user sends image → Claude Vision analyzes → pre-fills fields)
  → ORDER_FABRIC        (only if NLP couldn't extract fabric from message)
  → ORDER_BUDGET        (only if NLP couldn't extract budget)
  → ORDER_TIMELINE      (only if NLP couldn't extract timeline)
  → ORDER_OCCASION      (only if NLP couldn't extract occasion)
  → MEASURE_CHOICE      (first order only, when all order fields collected)
  → MEASURE_INPUT       (if manual selected)
  → MEASURE_CONFIRM
  → AWAITING_QUOTES     (15 min timer starts — can skip straight here if all fields + measurements exist)
  → QUOTE_SELECT
  → PAYMENT_CONFIRM
  → IDLE                (order placed, tracking is async)

  NOTE: With NLP, a single message like "ankara agbada for wedding, 20k budget, 2 weeks"
  causes Claude to extract all 4 fields → state jumps IDLE → MEASURE_CHOICE (or AWAITING_QUOTES)
  skipping ORDER_FABRIC, ORDER_BUDGET, ORDER_TIMELINE, ORDER_OCCASION entirely.

DELIVERED
  → APPROVAL_PENDING    (24hr timer starts)
  → REVIEW_ARTISAN      (if approved)
  → REVIEW_PLATFORM
  → IDLE                (review complete → 60% released)

  → DISPUTE_TYPE        (if concern raised)
  → DISPUTE_PHOTO
  → DISPUTE_PENDING     (artizan team handles)
```

### Brand states
```
ONBOARD_BRAND_NAME
  → ONBOARD_BRAND_LOCATION
  → ONBOARD_BRAND_SPECIALTIES
  → ONBOARD_BRAND_PRICE_RANGE
  → ONBOARD_BRAND_PORTFOLIO
  → ONBOARD_BRAND_CAPACITY
  → PENDING_VERIFICATION
  → IDLE

IDLE
  → QUOTE_RESPOND       (received order request)
  → IDLE                (quote submitted or declined)

PRODUCTION
  → IDLE                (status updates are keyword-based, no state change needed)
```

### State transition handler (pseudocode)
```javascript
async function handleMessage(phone, message) {
  // 1. Load user + state
  const user = await db.users.findByPhone(phone);
  const state = await redis.get(`state:${phone}`);

  // 2. New user?
  if (!user) {
    await createUser(phone);
    await redis.set(`state:${phone}`, { state: 'ONBOARD_ACCOUNT_TYPE' });
    return sendMessage(phone, MESSAGES.WELCOME);
  }

  // 3. If message is an image, run Claude Vision first
  let enrichedMessage = message;
  if (message.type === 'image') {
    const analysis = await analyzeDesignImage(message.url);
    enrichedMessage = {
      type: 'design_image',
      url: message.url,
      analysis  // { garment_type, detected_fabric, complexity, ... }
    };
  }

  // 4. Run Claude NLP on text messages for intent + field extraction
  let parsed = null;
  if (typeof enrichedMessage === 'string' || enrichedMessage.type === 'design_image') {
    const history = await getConversationHistory(phone, 4);
    parsed = await safeParseIntent(
      typeof enrichedMessage === 'string' ? enrichedMessage : enrichedMessage.analysis.garment_description,
      state.state,
      history
    );
  }

  // 5. Check for global commands (Claude detects these too)
  if (parsed?.intent === 'command') {
    const commandHandler = COMMAND_HANDLERS[parsed.command];
    if (commandHandler) return commandHandler(phone);
  }

  // 6. Route by state, passing parsed NLP data + image analysis
  const handler = STATE_HANDLERS[state.state];
  if (!handler) return sendMessage(phone, MESSAGES.CONFUSED);

  const result = await handler(phone, enrichedMessage, state.data, parsed);

  // 7. Transition state
  await redis.set(`state:${phone}`, {
    state: result.nextState,
    data: { ...state.data, ...result.data },
    lastMessageAt: new Date().toISOString()
  });

  // 8. Store conversation turn for NLP context
  await storeConversationTurn(phone, enrichedMessage, result.reply);

  // 9. Send response
  await sendMessage(phone, result.reply);

  // 10. Trigger side effects (async jobs)
  if (result.jobs) {
    for (const job of result.jobs) {
      await queue.add(job.type, job.payload);
    }
  }
}
```

---

## 5. WEBHOOK ENDPOINT

```javascript
// POST /webhook — receives all WhatsApp messages
app.post('/webhook', async (req, res) => {
  // Acknowledge immediately (WhatsApp requires 200 within 5s)
  res.status(200).send('OK');

  const { from, type, text, image, timestamp } = parseWebhook(req.body);

  try {
    if (type === 'text') {
      // Text goes through Claude NLP for intent parsing
      await handleMessage(from, text.body);

    } else if (type === 'image') {
      // Upload to Cloudinary first, then Claude Vision analyzes
      const url = await uploadImage(image.id);
      await handleMessage(from, { type: 'image', url });
      // handleMessage runs analyzeDesignImage() internally
      // and pre-fills garment_type, fabric, complexity, etc.

    } else if (type === 'interactive') {
      // Button/list replies bypass NLP (already structured)
      await handleMessage(from, text.body || req.body.interactive.button_reply.id);
    }
  } catch (error) {
    logger.error('Webhook processing failed', { from, error });
    await sendMessage(from, MESSAGES.ERROR_GENERIC);
  }
});

// GET /webhook — Meta verification handshake
app.get('/webhook', (req, res) => {
  const mode = req.query['hub.mode'];
  const token = req.query['hub.verify_token'];
  const challenge = req.query['hub.challenge'];

  if (mode === 'subscribe' && token === process.env.VERIFY_TOKEN) {
    res.status(200).send(challenge);
  } else {
    res.sendStatus(403);
  }
});
```

---

## 6. ASYNC JOB QUEUE

Jobs that shouldn't block the webhook response:

### match_artisans
Triggered when buyer completes order details + measurements.
```javascript
{
  type: 'match_artisans',
  payload: {
    orderId: 'uuid',
    garmentType: 'agbada',
    buyerLocation: 'Lekki Phase 1',
    budgetMin: 15000,
    budgetMax: 30000
  }
}
// Logic:
// 1. Query brand_profiles WHERE specialties @> '{agbada}'
//    AND location near buyerLocation
//    AND price_min <= 30000 AND price_max >= 15000
//    AND verified = true AND status = 'active'
//    ORDER BY rating_avg DESC LIMIT 7
// 2. Send order request to each matched brand via WhatsApp
// 3. Create quote records with 15min expiry
// 4. Schedule 'collect_quotes' job for 15 mins later
```

### collect_quotes
Triggered 15 minutes after matching.
```javascript
{
  type: 'collect_quotes',
  payload: { orderId: 'uuid' }
}
// Logic:
// 1. Fetch all quotes WHERE order_id = orderId AND status = 'submitted'
// 2. If zero quotes → notify buyer, widen match or retry
// 3. If 1+ quotes → format and send to buyer as selection list
// 4. Set buyer state to QUOTE_SELECT
```

### process_payment
Triggered when buyer confirms payment.
```javascript
{
  type: 'process_payment',
  payload: { orderId: 'uuid', buyerId: 'uuid', brandId: 'uuid' }
}
// Logic:
// 1. Debit buyer wallet via 9PSB: total_price
// 2. Hold in escrow
// 3. Disburse 40% of artisan_price to brand wallet
// 4. Record wallet_transactions
// 5. Notify brand: "You've been selected! ₦X advance incoming"
// 6. Notify buyer: "Order confirmed! 40% sent to artisan"
// 7. Update order status → 'production'
```

### book_delivery
Triggered when brand marks order as "ready".
```javascript
{
  type: 'book_delivery',
  payload: { orderId: 'uuid' }
}
// Logic:
// 1. Get brand address + buyer address
// 2. Call Kwik API: book pickup
// 3. Store delivery_id on order
// 4. Notify brand: "Rider arriving in ~X mins"
// 5. Notify buyer: "Your order is on its way!"
```

### release_payment
Triggered when buyer completes review.
```javascript
{
  type: 'release_payment',
  payload: { orderId: 'uuid' }
}
// Logic:
// 1. Disburse remaining 60% to brand wallet via 9PSB
// 2. Collect service fee to Artizan revenue wallet
// 3. Release delivery fee to Kwik (or own rider fund)
// 4. Record transactions
// 5. Notify brand: "₦X released. Rating: ⭐⭐⭐⭐⭐"
// 6. Update brand rating_avg
// 7. Update order status → 'reviewed'
```

### auto_approve
Scheduled 24hrs after delivery confirmation.
```javascript
{
  type: 'auto_approve',
  payload: { orderId: 'uuid' },
  delay: 86400000  // 24 hours in ms
}
// Logic:
// 1. Check if order already approved/reviewed/disputed
// 2. If not → auto-approve, set auto_approved = true
// 3. Nudge buyer for review (still needed for payment release)
```

### review_nudge
Scheduled 24hrs and 48hrs after delivery if no review.
```javascript
// 24hr nudge:
// "Your artisan is waiting to be paid — a quick review releases their payment."
//
// 48hr nudge (final):
// "Last reminder — please leave a review so your artisan can be paid."
// After 48hrs with no review → auto-release payment + record as "no review"
```

---

## 7. WHATSAPP MESSAGE TEMPLATES

Templates that need Meta pre-approval (business-initiated messages):

### order_status_update (Utility)
```
Your order #{{1}} has been updated.
Status: {{2}}
Artisan: {{3}}

Track on WhatsApp anytime by typing "orders".
```

### delivery_notification (Utility)
```
🚚 Your order #{{1}} is on its way!
Estimated arrival: {{2}} minutes.
From: {{3}}
```

### review_reminder (Utility)
```
Hi {{1}}! Your order from {{2}} was delivered.
Your artisan is waiting to be paid — a quick review releases their payment.
Reply to leave a review now.
```

### quote_request (Utility — sent to brands)
```
New order request on Artizan!
Garment: {{1}}
Budget: ₦{{2}} - ₦{{3}}
Deadline: {{4}}
Reply with your price and days, or "pass" to decline.
You have 15 minutes to respond.
```

### payment_received (Utility)
```
💰 Payment received for Order #{{1}}.
₦{{2}} production advance sent to your wallet.
Full order details attached. Begin production!
```

---

## 8. API INTEGRATION SPECS

### 9PSB Wallet-as-a-Service

**Endpoints needed:**
```
POST /wallets/create          → Create wallet for new user
POST /wallets/fund             → Generate funding reference
POST /wallets/debit            → Debit buyer wallet (escrow in)
POST /wallets/credit           → Credit brand wallet (advance/final)
GET  /wallets/{id}/balance    → Check wallet balance
GET  /wallets/{id}/history    → Transaction history
POST /virtual-accounts/create → Generate virtual account number
```

**Escrow implementation:**
9PSB may not have native escrow. Workaround: use an Artizan master wallet as the escrow account. Buyer funds debit from buyer wallet → credit to master wallet. On release, master wallet → brand wallet. All automated via API.

### Kwik Delivery API

**Endpoints needed:**
```
POST /quotes        → Get delivery price (pickup + dropoff coords)
POST /bookings      → Book a delivery
GET  /bookings/{id} → Track delivery status
Webhook: delivery_status_update → Receive status callbacks
```

**Price quote request:**
```json
{
  "pickup": {
    "latitude": 6.5095,
    "longitude": 3.3711,
    "address": "12 Herbert Macaulay Way, Yaba"
  },
  "dropoff": {
    "latitude": 6.4489,
    "longitude": 3.4724,
    "address": "5 Admiralty Way, Lekki Phase 1"
  },
  "package_type": "clothing",
  "package_size": "small"
}
```

---

## 9. DEPLOYMENT ARCHITECTURE (MVP)

```
┌─────────────────────────────────────────┐
│               Railway / Render          │
│                                         │
│  ┌──────────────┐  ┌──────────────┐     │
│  │  Web server   │  │ Worker server │    │
│  │  (Express)    │  │ (BullMQ)     │    │
│  │  /webhook     │  │ Job processor │    │
│  └──────┬───────┘  └──────┬───────┘     │
│         │                  │            │
│  ┌──────┴──────────────────┴──────┐     │
│  │           Redis                 │    │
│  │   (state + queue + cache)       │    │
│  └─────────────────────────────────┘    │
│                                         │
│  ┌─────────────────────────────────┐    │
│  │         PostgreSQL               │   │
│  │   (Railway managed or Supabase)  │   │
│  └─────────────────────────────────┘    │
└─────────────────────────────────────────┘

External:
  → 360dialog / Twilio (WhatsApp BSP)
  → Claude API (NLP + Vision — intent parsing, image recognition)
  → 9PSB API (wallets)
  → Kwik API (delivery)
  → Cloudinary (images)
```

### Environment variables
```
DATABASE_URL=postgresql://...
REDIS_URL=redis://...
WHATSAPP_BSP=360dialog
BSP_API_KEY=xxx
BSP_WEBHOOK_SECRET=xxx
VERIFY_TOKEN=artizan-verify-xxx
PSB_API_KEY=xxx
PSB_API_SECRET=xxx
PSB_MASTER_WALLET_ID=xxx
KWIK_API_KEY=xxx
CLOUDINARY_URL=cloudinary://...
ANTHROPIC_API_KEY=sk-ant-xxx       # Claude API for NLP + Vision
ADMIN_SECRET=xxx
```

---

## 10. SECURITY CONSIDERATIONS

- **Webhook verification:** Validate every incoming webhook with BSP signature/secret
- **Rate limiting:** Max 30 messages/minute per phone number (Redis-based)
- **Data encryption:** Encrypt measurements and wallet data at rest (AES-256)
- **PII handling:** Never log full phone numbers or measurements in plain text
- **Wallet PINs:** Require PIN for withdrawals > ₦5,000 (stored as bcrypt hash)
- **Admin access:** 2FA required, audit log on all admin actions
- **Image handling:** Strip EXIF data from all uploaded photos before storage
- **NDPC compliance:** Consent recorded at onboarding, data deletion endpoint available

---

## 11. MONITORING + LAUNCH CHECKLIST

### Key metrics to track
- Message delivery rate (should be > 99%)
- Webhook response time (must stay < 5s)
- Quote response rate (% of artisans who respond in 15 min window)
- Order completion rate (% of orders that reach 'reviewed' status)
- Average time from order to delivery
- Dispute rate (target: < 5% of orders)
- Review completion rate (target: > 80%)

### Launch sequence
1. Deploy backend + database to Railway
2. Connect Redis instance
3. Register with BSP (360dialog), get API key
4. Create Meta Business Manager, submit for verification
5. Register phone number with BSP
6. Submit message templates to Meta for approval
7. Integrate 9PSB WaaS (create master wallet)
8. Integrate Kwik API (test with sandbox)
9. Onboard 10 curated artisans via WhatsApp
10. Test complete order flow (team members as buyers)
11. Soft launch with 20-30 invited buyers
12. Monitor, fix, iterate
13. Open registration via landing page
```
