# Workflow Explanation — Burger Club ZAVIQUE Voice Ordering

> **How to read this document.** The demonstration video does **not** show an automation/workflow canvas for this project. What it *does* show, end to end, is the **request lifecycle**: a live voice order on the website, and the Google Sheets backend (Menu, Orders, Customers, Config, API log) with a complete audit trail of the API calls that order produced. This document therefore explains the workflow as the evidenced request lifecycle, stage by stage, and then documents each sheet as a processing stage (purpose, input, processing, output, connection to next).
>
> Anything the video does not show is marked **"Not confirmed from video."**

---

## Part 1 — The Request Lifecycle (trigger to final result)

### Step 0 — Conversation starts (trigger)

- **What happens:** The customer opens the **Burger Club AI** widget on the Burger Club ZAVIQUE website. The widget cycles through connection states visible on screen: **"Connecting" → "Talk to interrupt" → "Listening"**.
- **Input:** Customer voice (or typed text via the "Send a message…" field).
- **Output:** A live conversation session; the assistant greets: *"Welcome to Burger Club! I'm your ordering assistant. How can I help you today?"*
- **Voice platform:** Not confirmed from video.

### Step 1 — Menu lookup (`get_menu`)

- **Purpose:** Give the assistant the current, authoritative menu — items, bilingual names, prices, availability, options, and quantity caps — before it quotes anything.
- **API evidence:** Log rows show `get_menu` → **200 TRUE**, summaries: *"menu returned: 10 items"* (also *"menu returned: 5 items"* and *"menu returned: 2 items"* on other calls — consistent with category/filtered menu queries; the filtering parameter itself is Not confirmed from video).
- **Input:** Menu request (exact request parameters: Not confirmed from video).
- **Processing:** Available items are read from the `Menu` sheet (rows where `IsAvailable = TRUE`).
- **Output:** 10-item menu (in the full-menu call) with prices in TRY.
- **Connection:** The assistant uses this data to take and price the order in Step 2.

### Step 2 — Order capture (conversation)

- **Purpose:** Turn free speech into a structured order.
- **What the video shows:** The customer orders **two Zavique Classics with extra cheese and one classic fries**, and chooses **pickup**. The assistant applies the `extra_cheese` option defined in the Menu sheet's `OptionsJSON`, and respects per-item `MaxQty` caps (10 in the visible menu).
- **Input:** Customer's spoken items, quantities, options, order type (pickup/delivery).
- **Output:** A structured item list ready for pricing.
- **Connection:** Before the order can be created, the assistant needs verified customer details → Step 3.

### Step 3 — Customer detail collection & phone read-back

- **Purpose:** Attach a reachable identity to the order and prevent wrong-number errors.
- **What the video shows:** The assistant asks: *"Great choice. Could I please have your name and phone number to complete your order?"* The customer gives a name and a +92 phone number (masked in this documentation as ***3456, exactly as the system's own log masks it). The assistant then **reads the number back digit-by-digit** — *"plus nine two three one four five four two three four five six, is that correct?"* — and proceeds only after the customer confirms.
- **Input:** Name, phone number.
- **Processing:** Verbal digit-by-digit verification loop.
- **Output:** A confirmed name + phone pair.
- **Connection:** Order creation → Step 4.

### Step 4 — Pending order creation (`create_order`)

- **Purpose:** Persist the order **before** it is final, so the customer can hear it read back and approve exactly what will be prepared.
- **API evidence:** `create_order` → **201 TRUE**, summary: *"pending created BC-261016-34KD 715.00 TRY phone \*\*\*3456"*.
- **Input:** Customer name, confirmed phone, order type (pickup), item list with options, and an idempotency key (the `Orders` sheet carries an `IdempotencyKey` column — this protects against duplicate orders from retried calls).
- **Processing (evidenced by Config + log):**
  - **Store-open guard:** the Config master switch and opening hours (Europe/Istanbul) are enforced. The log shows three `create_order` calls rejected with **409 FALSE**, error code **`STORE_CLOSED`** (displayed truncated as `STORE_CLOSE` in the sheet cell), summary *"The restaurant is not accepting orders right now."*
  - **Pricing:** Subtotal from live Menu prices; delivery fee 0 for pickup; total 715.00 TRY for the demo order.
  - The order row is written to the `Orders` sheet with status **pending**.
- **Output:** Order ID **BC-261016-34KD** in pending state.
- **Connection:** The assistant reads the order back → Step 5.

### Step 5 — Order read-back & verbal confirmation gate

- **Purpose:** Human-in-the-loop approval — the last chance to catch errors before the kitchen is committed.
- **What the video shows:** *"Your order is for two Zavique Classics with extra cheese and one classic fries, totaling seven hundred and fifteen Turkish Lira. Shall I confirm this order?"*
- **Supporting config (evidenced):** The Config sheet defines *"minutes a pending (unapproved) order stays confirmable"* — pending orders expire, so an unconfirmed order does not linger indefinitely. It also caps *"maximum simultaneous pending orders per phone number"*.
- **Input:** Customer's yes/no.
- **Output:** On "yes" → Step 6. (Behaviour on "no" — edit/repeat flow — is Not confirmed from video.)

### Step 6 — Order confirmation (`confirm_order`)

- **Purpose:** Finalise the pending order.
- **API evidence:** `confirm_order` → **200 TRUE**, summary: *"confirmed BC-261016-34KD phone \*\*\*3456"*.
- **Input:** Order ID BC-261016-34KD.
- **Processing:** The `Orders` row transitions from pending to confirmed; the `Customers` sheet is updated (see Part 2) — the video shows the demo customer's record carrying order count, first seen, last order time, and last order ID **BC-261016-34KD**.
- **Output:** A confirmed order.
- **Connection:** Completion announcement → Step 7.

### Step 7 — Completion announcement

- **What the video shows:** *"Your order is confirmed! The order ID is B C two six one zero one six dash three four K D. It will be ready for pickup in approximately fifteen minutes. Is there anything else I can help you with today?"*
- **Supporting config (evidenced):** The 15 minutes matches the Config *"pickup estimate in minutes"* key (a separate *"delivery estimate in minutes"* key exists for delivery orders).
- **Output:** Customer leaves with an order ID and a pickup ETA.

### Step 8 — Audit logging (continuous)

- **Purpose:** Every API call — successful or not — is written to the **API log sheet** with `Timestamp, Endpoint, RequestID, OrderID, StatusCode, Success, ErrorCode, Summary`.
- **Evidence:** The demo's full trail is visible in the log: `get_menu` 200s, the `create_order` 201 pending creation, the `confirm_order` 200 — plus earlier 409 `STORE_CLOSED` rejections proving the guard fires and is recorded.
- **Privacy:** Phone numbers are masked in log summaries (`***3456`).

### Lifecycle summary

```
Voice/text order → get_menu → order capture → name + phone (digit read-back)
   → create_order [guard: store open? → 409 STORE_CLOSED if not]
   → PENDING order in Orders sheet → read-back + "Shall I confirm?"
   → confirm_order → CONFIRMED + Customers sheet updated
   → order ID + pickup ETA announced → every call written to API log
```

---

## Part 2 — The Google Sheets Data Layer (stage by stage)

Workbook: **"Burger Club Orders (ZAVIQUE)"**. The video shows five sheets. Each is documented below as a processing stage.

### Stage A — Menu sheet (product catalogue)

- **Purpose:** Single source of truth for what can be sold, at what price, in both languages.
- **Columns (as shown):** `Category`, `NameTR`, `NameEN`, `DescriptionTR`, `DescriptionEN`, `Price`, `Currency`, `IsAvailable`, `PriceIsExample`, `MaxQty`, `OptionsJSON`, `SortOrder`.
- **Content shown (10 items):**

  | Category | Name (EN) | Price (TRY) |
  |---|---|---|
  | burger | Zavique Classic | 285 |
  | burger | Zavique Double | 390 |
  | burger | Truffle Burger | 345 |
  | burger | Black Cheddar | 330 |
  | burger | Spicy Jalapeno | 315 |
  | chicken | Crispy Chicken | 255 |
  | chicken | Grilled Chicken | 265 |
  | chicken | Hot Honey Chicken | 285 |
  | fries | Classic Fries | 95 |
  | fries | Cheddar Truffle | 135 |

- **Processing notes:** `OptionsJSON` holds per-item modifiers (e.g. `extra_cheese`; a `large` size option appears on fries). `MaxQty` = 10 on all visible rows. All visible rows have `IsAvailable = TRUE` and **`PriceIsExample = TRUE`** — the prices are explicitly example data, and descriptions read "Sample description" / "Örnek açıklama".
- **Input:** Maintained by the restaurant in the sheet.
- **Output:** Served to the assistant via `get_menu`.
- **Connection:** Feeds Step 1 (menu lookup) and Step 4 (pricing).

### Stage B — Config sheet (business rules)

- **Purpose:** Operational settings the API enforces, editable without code.
- **Keys evidenced in the video (paraphrased from the sheet's own descriptions):**
  - Master store switch — when FALSE, new orders are rejected (the `STORE_CLOSED` guard).
  - Opening time / closing time — `HH:MM`, timezone **Europe/Istanbul**; equal open/close = always open; ranges may wrap past midnight.
  - Currency code shown to customers — **REQUIRED**.
  - Delivery fee (in currency units) — **REQUIRED**.
  - Minimum subtotal for delivery orders — **REQUIRED**.
  - Minutes a pending (unapproved) order stays confirmable — **REQUIRED**.
  - `MaxQty` caps for menu items.
  - Maximum simultaneous pending orders per phone number.
  - Pickup estimate in minutes; delivery estimate in minutes.
- **Input:** Maintained by the restaurant.
- **Output:** Guard and pricing decisions in `create_order`; ETAs in the completion announcement.
- **Connection:** Consulted at Step 4 (creation) and Step 7 (ETA).

### Stage C — Orders sheet (order ledger)

- **Purpose:** The system of record for every order.
- **Columns (as shown):** `OrderID`, `Timestamp`, `IdempotencyKey`, `Status`, `CustomerName`, `Phone`, `OrderType`, `DeliveryAddress`, `ItemsJSON`, `Subtotal`, `DeliveryFee`, `Total`, `Currency`, `Notes`.
- **Demo row (as shown):** OrderID **BC-261016-34KD**, timestamped 2026-10-16, **pickup**, placed via **voice_agent** (per the Notes/source field shown), total **715 TRY**, delivery fee **0**.
- **Input:** Written by `create_order` (pending), updated by `confirm_order`.
- **Output:** Confirmed orders for the kitchen / staff; the order history behind the Customers sheet.
- **Connection:** Written at Steps 4 and 6.

### Stage D — Customers sheet (customer registry)

- **Purpose:** Recognise repeat customers and their history.
- **Content shown:** A record for the demo customer with **order count**, **first seen**, **last order time**, and **last order ID (BC-261016-34KD)**. (Full column list partially visible — Not confirmed from video beyond these fields.)
- **Input:** Updated when an order is confirmed.
- **Output:** Customer history keyed by phone number.
- **Connection:** Updated at Step 6.

### Stage E — API log sheet (audit trail)

- **Purpose:** Observability — prove what the API did, call by call.
- **Columns (as shown):** `Timestamp`, `Endpoint`, `RequestID`, `OrderID`, `StatusCode`, `Success`, `ErrorCode`, `Summary`.
- **Rows evidenced:**
  - `get_menu` → 200 TRUE — *"menu returned: 10 items"* (also 5-item and 2-item variants)
  - `create_order` → 409 FALSE — error `STORE_CLOSED`, *"The restaurant is not accepting orders right now."* (×3)
  - `create_order` → 201 TRUE — *"pending created BC-261016-34KD 715.00 TRY phone \*\*\*3456"*
  - `confirm_order` → 200 TRUE — *"confirmed BC-261016-34KD phone \*\*\*3456"*
- **Input:** One row per API call.
- **Output:** Support/debugging trail; phone numbers masked.
- **Connection:** Written at every step (Step 8).

---

## What is NOT shown (and therefore not claimed)

- The code/implementation behind `get_menu`, `create_order`, and `confirm_order` — **Not confirmed from video**.
- The API host or base URL — **Not confirmed from video**.
- The voice platform powering the widget — **Not confirmed from video**.
- Any automation canvas (e.g. n8n) — **not shown in the video**; no orchestration tool is claimed for this project.
- Payment processing — no payment step appears in the video; the demo order is a pickup order with no payment shown.
