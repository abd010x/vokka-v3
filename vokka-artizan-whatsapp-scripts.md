# Artizan — WhatsApp Bot Conversation Scripts
## Complete Flow Documentation v1.0

---

## 1. ENTRY (From Website)

**Trigger:** User clicks "Start on WhatsApp" → opens WhatsApp with pre-filled message

**Pre-filled message:** `Hi Artizan!`

---

### ARTIZAN:
> Hey there! 👋 Welcome to **Artizan** — custom fashion, entirely on WhatsApp.
>
> How would you like to get started?
>
> 1️⃣ **Personal account** — I want to sew custom clothes
> 2️⃣ **Business account** — I'm a tailor or fashion brand
>
> Reply 1 or 2.

---

## 2. BUYER ONBOARDING (Personal Account)

**User replies:** `1`

### ARTIZAN:
> Great choice! Let's set you up in 60 seconds.
>
> What's your full name?

**User replies:** `Amaka Obi`

### ARTIZAN:
> Nice to meet you, Amaka! 🙌
>
> What's your gender?
>
> 1️⃣ Female
> 2️⃣ Male
> 3️⃣ Prefer not to say

**User replies:** `1`

### ARTIZAN:
> And where in Lagos are you based?
>
> Type your area (e.g. Lekki, Yaba, Ikeja, Surulere, VI)

**User replies:** `Lekki Phase 1`

### ARTIZAN:
> You're all set, Amaka! ✅
>
> Here's how Artizan works:
> 📸 Send me a photo of any outfit you'd like sewn
> 🎯 I'll match you with skilled artisans in 15 minutes
> 💳 Pay securely — your money is held in escrow
> 🚚 We deliver the finished piece to your door
>
> Whenever you're ready, just send a photo of what you want made! 👗

---

## 3. BRAND ONBOARDING (Business Account)

**User replies:** `2`

### ARTIZAN:
> Welcome, fellow creative! 🧵 Let's get your brand on Artizan.
>
> What's your brand or business name?

**User replies:** `Stitches by Ada`

### ARTIZAN:
> Love the name! Where is your workshop located?
>
> Type your area in Lagos (e.g. Yaba, Surulere, Ikeja)

**User replies:** `Yaba`

### ARTIZAN:
> What types of garments do you specialize in? Select all that apply:
>
> 1️⃣ Agbada
> 2️⃣ Kaftan
> 3️⃣ Senator suit
> 4️⃣ Gown / dress
> 5️⃣ Bridal / aso-ebi
> 6️⃣ Casual wear
> 7️⃣ Ankara styles
> 8️⃣ Other
>
> Reply with the numbers (e.g. "1, 4, 7")

**User replies:** `1, 4, 7`

### ARTIZAN:
> Got it — Agbada, Gowns, and Ankara. Strong combination! 💪
>
> What's your typical price range per garment?
>
> 1️⃣ ₦5,000 — ₦15,000
> 2️⃣ ₦15,000 — ₦30,000
> 3️⃣ ₦30,000 — ₦60,000
> 4️⃣ ₦60,000+

**User replies:** `2`

### ARTIZAN:
> Now the most important part — your portfolio. 📷
>
> Please send 3–5 photos of your best past work. These are what buyers will see when they get your quote, so make them count!

**User sends:** *3-5 photos*

### ARTIZAN:
> Beautiful work! Last question — how many orders can you handle per week?
>
> 1️⃣ 1–3 orders
> 2️⃣ 4–7 orders
> 3️⃣ 8–15 orders
> 4️⃣ 15+ orders

**User replies:** `2`

### ARTIZAN:
> Your profile is submitted! ✅
>
> Our team will review and activate your account within 24 hours. Once you're live, you'll receive order requests that match your skills and price range — right here on WhatsApp.
>
> **Important to know:**
> 💰 You receive **40% of your quoted price upfront** when a buyer selects you — so you can start production immediately.
> ⭐ Your remaining **60% is released after the buyer completes their review** of the finished piece.
> 📊 Your rating builds over time and affects how often you're matched with new buyers.
>
> We'll message you as soon as you're activated! 🎉

---

## 4. FIRST ORDER — DESIGN SUBMISSION

**Buyer sends:** *Photo of an outfit*

### ARTIZAN:
> Love that! 👀 Let me get a few details so I can match you with the right artisans.
>
> What **fabric** are you thinking?
>
> 1️⃣ Ankara
> 2️⃣ Lace
> 3️⃣ Cotton
> 4️⃣ Aso-oke
> 5️⃣ Adire
> 6️⃣ Other (type it)

**User replies:** `1`

### ARTIZAN:
> What's your **budget range** for this piece?
>
> 1️⃣ ₦5,000 — ₦15,000
> 2️⃣ ₦15,000 — ₦30,000
> 3️⃣ ₦30,000 — ₦60,000
> 4️⃣ ₦60,000 — ₦100,000
> 5️⃣ ₦100,000+

**User replies:** `2`

### ARTIZAN:
> When do you need it by?
>
> Type a date or something like "2 weeks", "by June 20", "no rush"

**User replies:** `by June 20`

### ARTIZAN:
> What's the occasion? (optional — helps artisans understand the vibe)
>
> 1️⃣ Wedding / aso-ebi
> 2️⃣ Formal event
> 3️⃣ Casual
> 4️⃣ Work / professional
> 5️⃣ Skip

**User replies:** `1`

---

## 5. FIRST ORDER — MEASUREMENTS

### ARTIZAN:
> One more thing before I find your artisans — I need your body measurements. 📐
>
> This is a **one-time thing**. Once saved, every future order uses them automatically.
>
> How would you like to get measured?
>
> 1️⃣ 📸 **AI quick estimate** _(Beta)_ — Send a full-body photo
> 2️⃣ 📹 **WhatsApp video call** _(Fastest)_ — Have a friend and tape measure ready
> 3️⃣ 🏠 **Request an agent visit** _(Most accurate)_ — Free home visit
> 4️⃣ ✏️ **Manual input** — Enter your own measurements

### If user selects 1 (AI Quick Estimate):

**User replies:** `1`

### ARTIZAN:
> Great! Stand against a plain wall in fitted clothing and have someone take a full-body photo of you — front view.
>
> Send the photo when ready. 📸
>
> ⚠️ Note: AI estimates are approximate. For the most accurate fit, we recommend the video call or agent visit for your first order.

**User sends:** *Full-body photo*

### ARTIZAN:
> Processing your measurements... ⏳
>
> Here's what I got:
> - Chest: 38"
> - Waist: 32"
> - Hips: 40"
> - Shoulder: 17"
> - Arm length: 24"
> - Dress length: 42"
>
> Do these look right?
>
> 1️⃣ Yes, save these
> 2️⃣ No, let me try another method

### If user selects 2 (Video Call):

**User replies:** `2`

### ARTIZAN:
> I'll connect you with a trained Artizan agent on a WhatsApp video call.
>
> Please have ready:
> 📏 A measuring tape (or a string + ruler)
> 👤 A friend to help
> 👕 Wear fitted clothing
>
> When are you available?
>
> 1️⃣ Right now
> 2️⃣ In 1 hour
> 3️⃣ Tomorrow morning
> 4️⃣ Tomorrow afternoon

**User replies:** `1`

### ARTIZAN:
> An agent will call you on this number in 2 minutes. The call takes about 5–7 minutes. Stand by! 📞

*[After call]*

### ARTIZAN:
> Your measurements have been saved! ✅
>
> - Chest: 38"
> - Waist: 32"
> - Hips: 40"
> - Shoulder: 17"
> - Arm length: 24"
> - Dress length: 42"
> - Neck: 15"
>
> These are now attached to your Artizan profile forever. No more measuring! 🎉

### If user selects 3 (Agent Visit):

**User replies:** `3`

### ARTIZAN:
> A trained Artizan agent will visit you at home — completely free. 🏠
>
> What's your delivery address in Lekki Phase 1?

**User replies:** *Address*

### ARTIZAN:
> When would you like the visit?
>
> 1️⃣ Today
> 2️⃣ Tomorrow morning (9am–12pm)
> 3️⃣ Tomorrow afternoon (1pm–5pm)
> 4️⃣ Another day

**User replies:** `2`

### ARTIZAN:
> Booked! ✅ An agent will visit you tomorrow between 9am–12pm. Takes about 10 minutes.
>
> I'll notify you when they're on the way! We'll continue matching your order once measurements are saved.

### If user selects 4 (Manual Input):

**User replies:** `4`

### ARTIZAN:
> Alright! I'll ask for each measurement one at a time.
>
> All measurements in **inches**. If you're not sure about any, type "skip" and we can fill it in later.
>
> **Chest** (around the fullest part): ?

**User replies:** `38`

### ARTIZAN:
> **Waist** (around the narrowest part): ?

*[Continues through: Hips, Shoulder width, Arm length, Dress/garment length, Neck]*

### ARTIZAN:
> Your measurements are saved! ✅ Here's a summary:
>
> *[Shows all measurements]*
>
> Look right?
>
> 1️⃣ Yes, save these
> 2️⃣ I need to correct something

---

## 6. MATCHING + QUOTES

### ARTIZAN:
> Now matching your order to artisans... 🔍
>
> Garment: Ankara dress (wedding)
> Budget: ₦15,000 — ₦30,000
> Deadline: June 20
> Location: Lekki Phase 1
>
> You'll receive quotes within **15 minutes**. Sit tight! ⏱️

*[15 minutes later, or when all artisans respond]*

### ARTIZAN:
> Your quotes are in! Here are your matched artisans:
>
> **1️⃣ Stitches by Ada** ⭐ 4.8
> 📍 Yaba · ₦18,000 · 5 days
> 12 completed orders
>
> **2️⃣ TailorMade Lagos** ⭐ 4.6
> 📍 Surulere · ₦22,000 · 3 days
> 8 completed orders
>
> **3️⃣ Bespoke by Chidi** ⭐ 4.9
> 📍 Ikeja · ₦25,000 · 4 days
> 21 completed orders
>
> Reply with a number to select, or type "more info" + the number to see their portfolio.

**If user types:** `more info 1`

### ARTIZAN:
> **Stitches by Ada** ⭐ 4.8 (12 orders)
> 📍 Yaba, Lagos
> Specializes in: Agbada, Gowns, Ankara styles
>
> *[Sends 2-3 portfolio photos]*
>
> ₦18,000 · Ready in 5 days
>
> Reply **1** to select Ada, or check another artisan.

---

## 7. SELECTION + PAYMENT

**User replies:** `1`

### ARTIZAN:
> Great choice! Here's your order summary:
>
> 🧵 **Ankara dress (wedding)**
> 👩‍🎨 Artisan: Stitches by Ada
> 📅 Ready in: 5 days
>
> 💰 **Breakdown:**
> Artisan fee: ₦18,000
> Artizan service fee (15%): ₦2,700
> Delivery (Yaba → Lekki): ₦2,500
> ─────────────
> **Total: ₦23,200**
>
> 💳 Pay from your Artizan wallet or fund it now.
>
> 1️⃣ Pay now
> 2️⃣ Fund my wallet first
> 3️⃣ Cancel

**User replies:** `1`

### ARTIZAN:
> Processing payment... ✅
>
> **Order #1042 confirmed!** 🎉
>
> Here's what happens next:
> → ₦7,200 (40%) has been sent to Stitches by Ada to start production
> → ₦10,800 (60%) held in escrow until you review the finished piece
> → ₦2,500 delivery fee held until dispatch
> → ₦2,700 Artizan service fee collected
>
> You'll receive production updates right here. Your artisan has already been notified! 🧵

---

## 8. PRODUCTION TRACKING (Buyer Side)

*[When artisan updates status]*

### ARTIZAN → BUYER:
> 🔄 **Order #1042 update**
>
> ✅ Order confirmed
> ✅ Fabric sourced
> 🔄 **Cutting started**
> ⬜ Sewing
> ⬜ Quality check
> ⬜ Ready for delivery
>
> Your artisan is making progress! Estimated completion: June 18.

*[Next update]*

### ARTIZAN → BUYER:
> 🔄 **Order #1042 update**
>
> ✅ Order confirmed
> ✅ Fabric sourced
> ✅ Cutting done
> ✅ Sewing complete
> 🔄 **Quality check**
> ⬜ Ready for delivery
>
> Almost there! 🎉

---

## 9. PRODUCTION TRACKING (Brand Side)

### ARTIZAN → BRAND:
> You've been selected for Order #1042! ✅
>
> ₦7,200 production advance has been sent to your Artizan wallet. 💰
>
> Here's the full order:
> 📸 *[Design image attached]*
> 📐 *[Measurements attached]*
> 🧵 Garment: Ankara dress
> 🎀 Occasion: Wedding
> 📅 Deadline: June 20
>
> Update your progress by replying with these keywords:
> **"cutting"** → Cutting started
> **"sewing"** → Sewing in progress
> **"finishing"** → Quality check / finishing
> **"ready"** → Ready for pickup
>
> Start whenever you're set! 🧵

**Brand replies:** `cutting`

### ARTIZAN → BRAND:
> Status updated! ✅ Your buyer has been notified.
>
> Next status: reply **"sewing"** when you move to the next stage.

**Brand replies:** `ready`

### ARTIZAN → BRAND:
> Marked as ready! ✅
>
> A dispatch rider will arrive at your workshop in Yaba within ~30 minutes to collect the order. Please have it packaged.
>
> You'll be notified when the buyer approves and your remaining ₦10,800 is released. 🎉

---

## 10. DELIVERY

### ARTIZAN → BRAND:
> A Kwik rider is on the way to your workshop. ETA: 20 minutes. 🏍️
>
> Please have Order #1042 packaged and ready for handoff.

*[After pickup]*

### ARTIZAN → BUYER:
> 🚚 **Your order is on its way!**
>
> Order #1042 has been picked up from your artisan and is being delivered to you.
>
> Estimated arrival: **45 minutes**
>
> You'll receive a notification when it arrives! 🎉

*[After delivery]*

### ARTIZAN → BUYER:
> 📦 **Delivered!**
>
> Your order from Stitches by Ada has arrived.
>
> Please try it on and let us know:
>
> 1️⃣ ✅ **Approve** — I'm happy with it!
> 2️⃣ ⚠️ **Raise a concern** — Something needs fixing
>
> ⏰ You have **24 hours** to respond. If we don't hear from you, the order is automatically approved and the artisan is paid.

---

## 11. APPROVAL + REVIEW

### If user approves:

**User replies:** `1`

### ARTIZAN:
> Wonderful! 🎉 We're so glad you love it.
>
> One last thing — your review releases your artisan's final payment. They're waiting on this to get paid, so it really matters.
>
> **Rate Stitches by Ada:**
> How many stars? (1–5)

**User replies:** `5`

### ARTIZAN:
> ⭐⭐⭐⭐⭐ Amazing!
>
> Want to add a comment? (or type "skip")

**User replies:** `Perfect fit, exactly what I wanted. Ada is incredibly talented!`

### ARTIZAN:
> Now rate **your Artizan experience:**
> How many stars? (1–5)

**User replies:** `4`

### ARTIZAN:
> Thanks for the honest feedback! 🙏
>
> Any suggestions for how we can improve? (or type "skip")

**User replies:** `skip`

### ARTIZAN:
> Review submitted! ✅
>
> ₦10,800 has been released to Stitches by Ada. 💰
>
> Thank you for using Artizan, Amaka! Whenever you're ready to sew something new, just send another photo. Your measurements are already saved — so it's even faster next time. 👗

---

## 12. REVIEW NUDGE (If buyer hasn't reviewed)

*[24 hours after delivery, if no response]*

### ARTIZAN → BUYER:
> Hi Amaka! 👋 Your order from Stitches by Ada was delivered yesterday.
>
> Your artisan is waiting to be paid — a quick review releases their final payment. It takes 30 seconds.
>
> 1️⃣ ✅ Approve and review now
> 2️⃣ ⚠️ Raise a concern
>
> ⏰ Auto-approved in 24 hours if we don't hear back.

*[If still no response after 48 hours]*

### ARTIZAN → BUYER:
> Last reminder, Amaka — your order has been auto-approved and Stitches by Ada has been paid. ✅
>
> If you have any concerns about the piece, you can still reach us here anytime.
>
> We'd still love your review to help future buyers! Reply with a star rating (1–5) whenever you get a chance. ⭐

---

## 13. DISPUTE FLOW

### If user raises concern:

**User replies:** `2` (Raise a concern)

### ARTIZAN:
> Sorry to hear that. Let's sort this out. 🛠️
>
> What's the issue?
>
> 1️⃣ **Fit issue** — Doesn't match my measurements
> 2️⃣ **Quality issue** — Poor stitching, fabric, or finishing
> 3️⃣ **Wrong design** — Doesn't match my reference image
> 4️⃣ **Damaged** — Arrived torn or stained

**User replies:** `1`

### ARTIZAN:
> Got it — fit issue. Please send a photo of the garment and point out what doesn't fit correctly.

**User sends:** *Photo + description*

### ARTIZAN:
> Thanks, Amaka. I've flagged this to our team and notified your artisan.
>
> We're comparing your measurements on file with the delivered piece. You'll hear back within **2 hours** with a resolution.
>
> Your remaining payment (₦10,800) stays in escrow until this is resolved. 🔒

*[After review]*

### ARTIZAN → BUYER (Minor fix):
> We've reviewed your case with Stitches by Ada. The artisan has agreed to adjust the fit at no extra cost.
>
> Here's the plan:
> 🚚 A rider will pick up the garment from you
> 🧵 Ada will adjust it within 2 days
> 🚚 Re-delivered to you free of charge
>
> 1️⃣ Sounds good, arrange pickup
> 2️⃣ I'd rather have a refund

### ARTIZAN → BUYER (Major issue):
> We've reviewed your case. This falls below Artizan's quality standard.
>
> Here's your refund:
> ✅ Remaining artisan fee (60%): ₦10,800 → refunded
> ✅ Service fee: ₦2,700 → refunded
> ─────────
> **Total refund: ₦13,500** returned to your Artizan wallet
>
> ⚠️ The 40% production advance (₦7,200) and delivery fee (₦2,500) are non-refundable as those services were already rendered.
>
> The artisan has received a quality strike. We take this seriously.
>
> Sorry about this experience. We'll match you with a different artisan next time. 💛

---

## 14. REPEAT ORDER (Returning Buyer)

**Buyer sends:** *New design photo*

### ARTIZAN:
> Welcome back, Amaka! 👋 Love this design.
>
> Let me get the details:

*[Skips to fabric → budget → timeline → occasion questions]*
*[Skips measurements entirely — already saved]*

### ARTIZAN:
> Your measurements are already on file ✅
> Matching artisans now... 🔍
>
> You'll have quotes within 15 minutes!

---

## 15. DIRECT RE-ORDER (Specific Artisan Request)

**Buyer says:** `I want Stitches by Ada for this one`

### ARTIZAN:
> Sending this directly to Stitches by Ada! 📨
>
> She'll receive your design, measurements, and details. Her quote will come through shortly — no competitive matching for this one.

*[Ada responds]*

### ARTIZAN:
> Stitches by Ada quoted:
>
> ₦20,000 · 4 days
>
> 💰 Breakdown:
> Artisan fee: ₦20,000
> Service fee (15%): ₦3,000
> Delivery (Yaba → Lekki): ₦2,500
> ─────────
> **Total: ₦25,500**
>
> 1️⃣ Accept and pay
> 2️⃣ Go through regular matching instead
> 3️⃣ Cancel

---

## 16. WALLET OPERATIONS

### Checking balance:

**User says:** `balance` or `wallet`

### ARTIZAN:
> 💰 **Your Artizan Wallet**
>
> Balance: ₦5,300
>
> 1️⃣ Fund my wallet
> 2️⃣ Withdraw to bank
> 3️⃣ Transaction history

### Funding wallet:

**User replies:** `1`

### ARTIZAN:
> How much would you like to add?
>
> Transfer to this account:
> 🏦 Bank: [MFB Partner Name]
> 📝 Account: 80XXXXXXXX
> 👤 Name: Artizan/Amaka Obi
>
> Your wallet updates automatically once the transfer is received. ⚡

---

## 17. BRAND — PAYMENT RECEIVED NOTIFICATION

### After buyer review is completed:

### ARTIZAN → BRAND:
> 🎉 **Payment released!**
>
> The buyer approved Order #1042 and left a review.
>
> ₦10,800 (remaining 60%) has been added to your Artizan wallet.
>
> **Order summary:**
> Total earned: ₦18,000 (₦7,200 advance + ₦10,800 final)
> Rating received: ⭐⭐⭐⭐⭐
> Comment: "Perfect fit, exactly what I wanted. Ada is incredibly talented!"
>
> Your current rating: ⭐ 4.8 (13 orders)
> Wallet balance: ₦23,800
>
> Keep up the amazing work! 🧵

---

## 18. HELP / MENU

**User types:** `help` or `menu`

### ARTIZAN:
> Here's what I can help with:
>
> 📸 **Sew something new** — Send a design photo
> 💰 **Wallet** — Check balance, fund, or withdraw
> 📐 **Measurements** — View or update
> 📦 **My orders** — Track active orders
> 📞 **Support** — Talk to a human
>
> What would you like to do?

---

*End of conversation scripts. Update WhatsApp number in all CTAs before deployment.*
