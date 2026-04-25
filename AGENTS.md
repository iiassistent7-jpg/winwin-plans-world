# AGENTS.md

## 1. Repository role

This repository is **winwin-plans-world** — a single-page tariff/payment surface served at `world.winwinnetwork.pro`.

It hosts one file: `index.html`, which is currently **byte-identical** to `winwin-plans/index.html`. Both repos serve the same page, just from different domains:
- `plans.winwinnetwork.pro` → `winwin-plans` (Israeli market)
- `world.winwinnetwork.pro` → `winwin-plans-world` (international)

The intent of the split is to allow regional differences in pricing, currency, payment methods, or copy. Today they are identical because that work has not started.

The page:
- Displays Win-Win tariffs (START / STANDARD / PRO) in 4 languages (ru / he / en / ar)
- Opens inside the Telegram WebApp (`telegram-web-app.js`)
- Routes the user to **PayPal** or **Bit** for actual payment
- Notifies the bot of payment intent via `tg.sendData({ action: "pay_intent", plan, price, method })`

### Important context

- **There is no automatic payment integration.** Activation is manual within ~1 hour of payment.
- This file is currently a twin of `winwin-plans/index.html`. **Most changes here must be mirrored there**, until the regional split actually diverges.
- A third related surface exists: `winwin-invite/pro_payment.html`. Three sources of truth for tariffs is technical debt.

This repo contains **no business logic**. The bot is the system of record.

---

## 2. Product mission & philosophy

Win-Win Network turns chaotic word-of-mouth recommendations into a transparent, trackable, and fair system.

The product makes four core promises to users:
- A recommendation is never lost
- A deal has a visible owner and status
- A reward is trackable, payable, and closable
- Trust grows when people behave well, and decays when they don't

### Trust and the Social Connector role

**Trust Index** is a central product concept, not decoration. Both businesses and agents accumulate trust separately. A future role — the **Social Connector** — is a trusted navigator built on top of agent trust.

The selection principle: people don't recommend weak services because they won't risk their own reputation.

In this repo's context: a clear and trustworthy payment page is itself a trust signal. International users especially — they're paying in a currency they care about, in a country with their own payment habits, and the page must respect that.

---

## 3. Channel strategy

**Telegram and WhatsApp are equal, first-class channels.** Both are in production.

This page is currently **TG-WebApp-centric** — it loads `telegram-web-app.js` and uses `tg.sendData` to notify the bot. WA users today reach payment instructions via WA messages from the bot.

When future work on this page is approved, it must not break the TG WebApp behavior, and must not assume the user is in TG either — the page may also be opened in a regular browser.

---

## 4. Product principles (in priority order)

1. **Simplicity** — every extra step on a payment page kills conversion.
2. **Understanding** — the user must immediately know: what plan, what price, how to pay, what happens next.
3. **Stability** — fail-safe beats elegant. A broken payment page costs revenue.
4. **Truth in statuses** — accurately reflect that **activation is manual** ("activates within 1 hour"). Never imply instant activation that isn't real. **"Only the correct truth."**
5. **Motivation and support** — be warm, not transactional.

**"Wow" is not a goal.** A beautiful UI is a consequence of simplicity and understanding.

---

## 5. Repo-specific map (winwin-plans-world)

### Tech stack
- Single `index.html`, vanilla HTML / CSS / JavaScript
- No build step, no framework, no NPM
- Loads `telegram-web-app.js` from `telegram.org`
- All four languages required: ru / he / en / ar; RTL via `body.rtl`

### Critical contracts

- `tg.sendData(JSON.stringify({ action: "pay_intent", plan, price, method }))` — payload consumed by the bot. Do not change `action`, `plan`, `price`, or `method` field names without coordinating with `amlatsa-bot`.
- `PP_URL = "https://paypal.me/istudio3377"` — payment recipient. Do not change without owner approval.
- Bit deep link target — same.
- Tariff names — `START`, `STANDARD`, `PRO` — must match the backend's `subscription_plan` values (`start`, `standard`, `pro`).

### Twin file management

This repo and `winwin-plans` serve byte-identical files today. Until the regional split is intentionally implemented:

1. Any change here must be mirrored in `winwin-plans`
2. Any change in `winwin-plans` must be mirrored here
3. The mirror is manual — there is no automation

When the regional split is finally implemented (currency, prices, payment methods specific to non-Israeli markets), this rule changes. Until then, drift between the two files is a bug.

### Consolidation note (technical debt)

`winwin-plans`, this repo, and `winwin-invite/pro_payment.html` form three sources of truth for tariffs. Consolidation is a planned, separate effort. **Do not "fix consolidation" as a side effect of any other task.**

### Known fragile areas

- Loaded inside Telegram WebApp on mobile — testing requires the Telegram client, not just a browser
- `tg.sendData` only fires when opened from inside the bot
- Currency, language, and RTL all interact in one file — easy to break one when fixing another

---

## 6. Working rules

1. Understand first — open the page, trace the payment flow end-to-end.
2. Explain what you see and what you'd change.
3. **Always ask: "does this change need to be mirrored in `winwin-plans`?"** Default answer is yes.
4. Identify other risks — does it affect what the bot expects from `pay_intent`?
5. Propose the smallest safe change.
6. Wait for explicit approval before modifying code.
7. Preserve existing behavior unless the task explicitly changes it.
8. Do not propose new payment integrations without explicit owner decision.

### Codex prompt discipline (used by this team)

- One file per prompt
- 2–3 find/replace blocks maximum per prompt
- `Find` strings must be unique (add a line of context if needed)
- `Find` must be copied from the file byte-for-byte
- After Codex returns the file:
  - `grep` confirms the new content is present
  - Verify no broken HTML / unclosed tags
  - Verify all four languages and RTL still render correctly
  - **Verify the matching change has been queued for `winwin-plans`** (or explicitly decided otherwise)

---

## 7. Constraints

Do not:
- Introduce a build step, bundler, framework, or NPM dependency
- Add an automatic payment processor without explicit owner approval
- Change `pay_intent` field names without coordinated bot change
- Change the PayPal account or Bit recipient without owner approval
- Drop the "manual activation within 1 hour" wording — it is the truth
- Let this file silently drift from `winwin-plans/index.html` without an intentional regional reason
- Ship strings in fewer than four languages

---

## 8. Output style

Prefer:
- Plain language, concrete recommendations
- Naming specific lines in `index.html` when discussing code
- Always asking: "should this change be mirrored in `winwin-plans`?"
- Calling out bot impact (does `pay_intent` payload change?) for every UI proposal

Avoid:
- "Modernize the payment page" — manual-activation is intentional
- Generic web-best-practice lectures
- Empty praise or reflexive apology
- Suggesting "consult a specialist" — you are the specialist for this project
