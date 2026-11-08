# Demo Video — Burger Club ZAVIQUE Voice Ordering

## Video link

📹 **Demo video:** _[PASTE DEMO VIDEO LINK HERE — e.g. YouTube / Loom / Google Drive share link]_

- **Duration:** ~40 seconds
- **Format:** Screen recording (1280×720) with the live voice conversation audible/visible in the website widget

## What the video shows

The video is a single end-to-end demonstration:

1. **The storefront** — the Burger Club ZAVIQUE website ("Premium Burgers. Gold Standard."), a luxury-branded burger site: 100% prime dry-aged beef, artisanal golden brioche buns, chef-curated sauces. Navigation: Home, Menu, Order & Track, Zavique Club, with a TR language toggle.
2. **A complete voice order** — the "Burger Club AI" widget (labelled "Bilingual AI Voice Assistant (TR / EN)") greets the customer, takes an order for **two Zavique Classics with extra cheese and one classic fries (pickup)**, collects the customer's name and phone, **reads the phone number back digit-by-digit**, creates the order, reads it back with the total (**715 Turkish Lira**), asks *"Shall I confirm this order?"*, and on confirmation announces order ID **BC-261016-34KD** with a **~15-minute** pickup estimate.
3. **The backend** — the Google Sheets workbook **"Burger Club Orders (ZAVIQUE)"**: the Menu sheet (10 bilingual items with example prices, options JSON, MaxQty), the Orders sheet (the demo order row), the Customers sheet (the demo customer's record), the Config sheet (store switch, hours, fees, limits, estimates), and the API log sheet.

## Key timestamps & screenshot sources

All screenshots in `../assets/` are real frames extracted from this video with ffmpeg — not mockups.

| Time (approx.) | What is on screen | Used as |
|---|---|---|
| 0:00–0:05 | Website hero + voice widget greeting ("Welcome to Burger Club! I'm your ordering assistant…") | `assets/project-demo-thumbnail.png` (frame at ~0:02) |
| ~0:05–0:15 | Order capture; assistant asks for name and phone number | — |
| ~0:15–0:22 | Customer gives demo details; assistant reads the phone back digit-by-digit for confirmation | — |
| ~0:24–0:32 | Order read-back (715 Turkish Lira), "Shall I confirm this order?", confirmation, order ID BC-261016-34KD, ~15-minute pickup estimate | — |
| ~0:35 | **Menu sheet** of the "Burger Club Orders (ZAVIQUE)" workbook — bilingual columns, prices, `PriceIsExample`, `OptionsJSON` | `assets/workflow-overview.png` — *the data-layer overview; the video shows no workflow canvas* |
| ~0:36–0:38 | Orders / Customers / Config sheets (demo order row BC-261016-34KD, total 715 TRY, pickup, via voice_agent; config keys incl. master switch, Europe/Istanbul hours, REQUIRED keys) | — |
| ~0:38–0:40 | **API log sheet** — `get_menu` 200 "menu returned: 10 items", `create_order` 201 "pending created BC-261016-34KD 715.00 TRY phone \*\*\*3456", `confirm_order` 200 "confirmed BC-261016-34KD", plus 409 `STORE_CLOSED` rejections | `assets/execution-result.png` (frame at ~0:38) |

## Notes

- The customer details in the video are demo data; the phone number is masked as ***3456 throughout this repository, exactly as the system's own API log masks it.
- The video does not show the backend implementation, the API host, or the voice platform — see `../PROJECT-LIMITATIONS.md`.
