# Project Limitations & Disclosure — Burger Club ZAVIQUE Voice Ordering

This document states plainly what this repository is, what it is based on, and what it cannot claim. It exists so that anyone reviewing this portfolio project — a client, employer, or fellow developer — can trust exactly what they are looking at.

## 1. Source of this documentation

- This documentation was produced **solely from a 40-second demonstration video** of the project, provided by the author.
- **No source code** was provided or reviewed.
- **No workflow/automation export (JSON or otherwise)** was provided or reviewed, and none is included in this repository. Any file purporting to be the original workflow export would be fabricated — there is intentionally no such file here.
- **No live deployment link** is available, and none is claimed. The website and backend shown are as they appear in the video only.

## 2. What the video does not establish

The following are explicitly **Not confirmed from video**:

| Item | Status |
|---|---|
| Backend / API implementation (language, framework, code) | Not confirmed from video |
| API host / base URL / deployment environment | Not confirmed from video |
| Voice platform powering the "Burger Club AI" widget | Not confirmed from video |
| Orchestration layer — including **whether n8n (or any automation tool) is involved at all** | **Not confirmed from video. The video shows no automation canvas, and this repository does not claim n8n for this project.** |
| LLM / speech models used by the assistant | Not confirmed from video |
| Exact request/response payloads of `get_menu`, `create_order`, `confirm_order` (beyond the log summaries and sheet columns visible) | Not confirmed from video |
| Order-ID generation rule (format `BC-261016-34KD` is observed; the rule behind it is not) | Not confirmed from video |
| Behaviour when a customer declines the read-back, edits a pending order, or orders for delivery | Not confirmed from video (delivery columns and config keys exist; no delivery order is demonstrated) |
| Payment handling | Not shown in the video at all — no payment step is claimed |
| Full column list of the Customers sheet | Partially visible only |

## 3. Data shown in the video is demonstration data

- **Menu prices are example prices.** The Menu sheet flags every visible row with `PriceIsExample = TRUE`, and item descriptions read "Sample description" / "Örnek açıklama". The 715 TRY demo total is arithmetically consistent with those example prices; it is not evidence of real-world pricing.
- **The demo customer is illustrative.** The customer name and phone number used in the video are demo details. In this documentation the phone number is masked as **\*\*\*3456** — the same masking the system's own API log uses.
- The order timestamp visible in the Orders sheet (2026-10-16) and the order ID are from the recorded demonstration run.

## 4. What *is* evidenced (and therefore documented as fact)

- The website, the bilingual voice widget, and a complete, successful voice order (greeting → order → phone read-back → pending creation → read-back → confirmation → order ID + 15-minute pickup estimate).
- The Google Sheets workbook structure: Menu, Orders, Customers, Config, and API log sheets, with the columns listed in the documentation.
- The API's observable contract: endpoints `get_menu`, `create_order`, `confirm_order`; status codes 200/201/409; the `STORE_CLOSED` rejection (displayed truncated as `STORE_CLOSE` in the sheet); idempotency key storage; pending → confirmed order states.
- The Config keys listed in the setup guide, including required keys, the Europe/Istanbul timezone, and the pending-order expiry and per-phone pending caps.

## 5. Reconstruction disclaimer

Where this repository explains *how to rebuild* an equivalent system (SETUP-GUIDE.md, Part B), that content is **suggested reconstruction** based on the evidenced contracts — it is not a description of the original implementation and must not be cited as one.

## 6. Assets

All images in `assets/` are **real frames extracted from the demonstration video** (timestamps noted in `demo/VIDEO-LINK.md`). No screenshot has been fabricated, mocked up, or edited to add features. Note that `assets/workflow-overview.png` shows the **Menu sheet (data layer)** — the video contains no workflow canvas to screenshot, and the file is named and captioned accordingly in the documentation.
