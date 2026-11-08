# Setup Guide — Burger Club ZAVIQUE Voice Ordering

> This guide has two clearly separated parts:
>
> - **Part A — Verified from video:** the components, schemas, settings, endpoints, and behaviours the demonstration video actually shows.
> - **Part B — Suggested reconstruction:** a practical path to rebuild an equivalent system. The original implementation (voice platform, API host, code) is **Not confirmed from video**, so Part B is guidance, not a claim about how the original was built.
>
> ⚠️ **No credentials, API keys, tokens, or secrets appear in this guide or anywhere in this repository.** Where a real deployment needs a credential, the guide says *what kind* is needed and *where it belongs* — never a value.

---

## Part A — Verified from Video

### A1. Components that must exist

| Component | What the video proves |
|---|---|
| Restaurant website | Burger Club ZAVIQUE site with pages Home, Menu, Order & Track, Zavique Club, and a TR language toggle |
| Voice widget | "Burger Club AI" — bilingual (TR/EN), voice states (Connecting / Talk to interrupt / Listening), plus typed message input |
| Ordering API | Operations `get_menu`, `create_order`, `confirm_order` with HTTP-style status codes (200/201/409), per-call request IDs, and error codes (e.g. `STORE_CLOSED`) |
| Google Sheets workbook | "Burger Club Orders (ZAVIQUE)" with five sheets (below) |

### A2. Workbook schema (exact, from the video)

**Menu** — `Category, NameTR, NameEN, DescriptionTR, DescriptionEN, Price, Currency, IsAvailable, PriceIsExample, MaxQty, OptionsJSON, SortOrder`
- One row per sellable item; bilingual names/descriptions; `OptionsJSON` carries modifiers (e.g. `extra_cheese`, size `large`); `MaxQty` caps per-item quantity.

**Orders** — `OrderID, Timestamp, IdempotencyKey, Status, CustomerName, Phone, OrderType, DeliveryAddress, ItemsJSON, Subtotal, DeliveryFee, Total, Currency, Notes`

**Customers** — keyed by customer (phone), tracking at least: order count, first seen, last order time, last order ID. *(Full column list partially visible in the video — Not confirmed from video beyond these fields.)*

**Config** — key/value settings (see A3).

**API log** — `Timestamp, Endpoint, RequestID, OrderID, StatusCode, Success, ErrorCode, Summary` (one row per API call; phone numbers masked in summaries, e.g. `***3456`).

### A3. Config keys (from the video)

| Setting | Notes from the sheet |
|---|---|
| Master store switch | When FALSE, new orders are rejected |
| Opening time / closing time | `HH:MM`, timezone Europe/Istanbul; equal values = always open; ranges may wrap past midnight |
| Currency code | Shown to customers — **REQUIRED** (TRY in the demo) |
| Delivery fee | In currency units — **REQUIRED** |
| Minimum subtotal for delivery | **REQUIRED** |
| Pending-order confirmable window | Minutes an unapproved order stays confirmable — **REQUIRED** |
| MaxQty caps | Per-item quantity ceiling |
| Max simultaneous pending orders per phone | Anti-abuse cap |
| Pickup estimate (minutes) | 15 quoted in the demo |
| Delivery estimate (minutes) | For delivery orders |

### A4. Endpoint contracts (from the API log)

| Endpoint | Success | Failure evidenced | Behaviour |
|---|---|---|---|
| `get_menu` | 200 — "menu returned: N items" | — | Returns available menu items |
| `create_order` | 201 — "pending created \<OrderID\> \<Total\> TRY phone \*\*\*\<last4\>" | 409, Success FALSE, ErrorCode `STORE_CLOSED` (cell displays truncated as `STORE_CLOSE`) — "The restaurant is not accepting orders right now." | Validates store-open state; writes a PENDING order; idempotency key stored |
| `confirm_order` | 200 — "confirmed \<OrderID\> phone \*\*\*\<last4\>" | — | Moves the pending order to confirmed; updates Customers |

### A5. Conversation behaviours (from the video)

1. Greeting: *"Welcome to Burger Club! I'm your ordering assistant. How can I help you today?"*
2. Collect items/options and order type (pickup/delivery).
3. Ask for name + phone; **read the phone back digit-by-digit** and require confirmation.
4. Create the pending order, then **read the full order and total back** and ask *"Shall I confirm this order?"*
5. On confirmation, announce the **order ID** and the **pickup/delivery estimate**.
6. Language: bilingual Turkish/English throughout.

---

## Part B — Suggested Reconstruction

*Everything in Part B is a recommendation for rebuilding an equivalent system. It is not a description of the original implementation, which is Not confirmed from video.*

### B1. What you need

| Need | Notes |
|---|---|
| A website | Any stack; the demo site is a single-page restaurant site with an embedded chat/voice widget |
| A voice-assistant platform | Any bilingual (TR/EN) voice agent platform with a website widget and function/tool calling — **the original platform is Not confirmed from video; choose your own** |
| A small backend/API host | Anything that can expose three operations and read/write Google Sheets (e.g. a serverless function, a small web service, or an automation platform) — **the original host is Not confirmed from video** |
| A Google account | To own the Sheets workbook and authorise API access to it |
| Credentials (types only) | Google service-account or OAuth credentials for Sheets access; voice-platform API key — **store these in your host's secret manager / environment variables, never in this repo or in the spreadsheet** |

### B2. Build steps

1. **Create the workbook** "Burger Club Orders (ZAVIQUE)" with the five sheets and the exact columns from A2.
2. **Load the menu.** Add your items with bilingual names/descriptions, prices, `IsAvailable`, per-item `OptionsJSON` (e.g. `[{"id":"extra_cheese", ...}]`), `MaxQty`, and `SortOrder`. Keep `PriceIsExample = TRUE` while testing with sample prices; set it to FALSE with real prices for production.
3. **Fill in Config.** Set every REQUIRED key (currency, delivery fee, minimum delivery subtotal, pending window) plus hours, master switch, caps, and ETAs.
4. **Implement the three operations** against the workbook:
   - `get_menu` → return rows where `IsAvailable = TRUE`, ordered by `SortOrder`. Log the call.
   - `create_order` → check Config (master switch + hours in Europe/Istanbul) → if closed, log and return **409 `STORE_CLOSED`**; validate items against Menu and `MaxQty`; price from the Menu sheet (never trust client-sent prices); honour the `IdempotencyKey` (same key ⇒ return the existing order, don't duplicate); write the order as **PENDING**; log and return **201** with the new Order ID.
   - `confirm_order` → find the pending order by ID; reject if its confirmable window has expired; mark **CONFIRMED**; upsert the Customers row (order count, first/last seen, last order ID); log and return **200**.
   - Generate Order IDs in the demonstrated format style (e.g. `BC-YYMMDD-XXXX` — the exact generation rule of the original is Not confirmed from video).
   - Mask phone numbers in log summaries (last 4 digits only, as `***3456`).
5. **Connect the voice assistant.** Give the agent: the greeting, the bilingual menu via `get_menu`, tool definitions for `create_order` / `confirm_order`, and instructions enforcing the A5 behaviours (digit-by-digit phone read-back; full order read-back before confirming; never confirm without the customer's yes).
6. **Embed the widget** on the restaurant website.
7. **Test like the demo:** place the demo order (2 × Zavique Classic + extra cheese, 1 × Classic Fries, pickup) and verify: total **715 TRY**, a PENDING row appears, confirmation flips it to CONFIRMED, the Customers row updates, and the API log shows 200/201/200 rows. Then set the master switch to FALSE and verify a `create_order` attempt returns **409 `STORE_CLOSED`** and is logged.

### B3. Going-live checklist

- [ ] Real menu prices loaded; `PriceIsExample` set to FALSE
- [ ] Config hours/switch match actual trading hours
- [ ] Credentials stored as secrets on the host — none in code, repo, or sheets
- [ ] Closed-store, expired-pending, and duplicate-submit (idempotency) tests pass
- [ ] Staff know orders land in the Orders sheet and how to work from it
