# GitHub Presentation Guide — Burger Club ZAVIQUE Voice Ordering

*How to publish this documentation pack as a clean, credible portfolio repository.*

## Repository name

```
burger-club-zavique-voice-ordering
```

## Repository description (one line — paste into GitHub's "About" field)

> Bilingual (TR/EN) AI voice ordering assistant for a burger restaurant — live Google Sheets menu, two-step pending→confirmed orders, and a full API audit log.

## Topics / tags

Add these under the repository's **About → Topics**:

```
ai-voice-assistant
voice-ordering
restaurant-automation
google-sheets-api
conversational-ai
bilingual-assistant
ai-agent
order-management
portfolio-project
```

*(Deliberately not tagged `n8n`: the demo video does not show or name an orchestration tool for this project — see PROJECT-LIMITATIONS.md. Add such tags only for projects whose videos/exports actually evidence them.)*

## Suggested commit messages

Initial upload (single commit):

```
docs: add full portfolio documentation pack for Burger Club ZAVIQUE voice ordering

- README with architecture, order flow, and demo evidence
- Workflow explanation (request lifecycle + Sheets data layer)
- Architecture doc with Mermaid flow
- Setup guide (verified facts vs suggested reconstruction)
- Limitations & disclosure
- Real demo-video frames in assets/
```

If committing file-by-file instead:

```
docs: add README for Burger Club ZAVIQUE voice ordering assistant
docs: document request lifecycle and Sheets data layer
docs: add architecture overview with Mermaid diagram
docs: add setup guide with verified vs reconstructed sections
docs: add limitations and disclosure
assets: add demo video frames (storefront, menu sheet, API log)
docs: add demo video link placeholder and GitHub presentation guide
```

## Upload instructions

**Option A — GitHub web UI (simplest)**

1. Go to https://github.com/new (signed in as **AiwithFahad**).
2. Repository name: `burger-club-zavique-voice-ordering`. Description: paste the one-liner above. Public. **Do not** initialise with a README (this pack already contains one).
3. Click **"uploading an existing file"**, then drag in the contents of this folder — keeping the structure: the six `.md` files at root, `assets/` with its three PNGs, and `demo/VIDEO-LINK.md`.
4. Commit with the initial-upload message above.
5. On the repo home page, click the ⚙️ next to **About**: paste the description again and add the topics listed above.
6. Replace the placeholder in `demo/VIDEO-LINK.md` with the real demo video URL (edit in place on GitHub), and paste the same link into the README's Demo Video section.
7. Optional: pin the repository on your GitHub profile (Profile → Customise pins) next to your other AI-agent projects.

**Option B — Git command line**

```bash
cd burger-club-zavique-voice-ordering
git init
git add .
git commit -m "docs: add full portfolio documentation pack for Burger Club ZAVIQUE voice ordering"
git branch -M main
git remote add origin https://github.com/AiwithFahad/burger-club-zavique-voice-ordering.git
git push -u origin main
```

(Create the empty repo on GitHub first, as in Option A steps 1–2, without uploading files.)

## Presentation advice

- **Lead with the demo.** Recruiters and clients watch video before they read. Put the demo link near the top of the README (it already is, via the thumbnail + Demo Video section) and add it to the repo's About → Website field as well.
- **Keep the honesty notes.** The "Not confirmed from video" disclosures are a credibility asset, not a weakness: they show you document systems the way you'd document them for a paying client — claims tied to evidence.
- **Don't add an `n8n` tag or claim** for this project unless you later attach the actual workflow export; the sheets and log are the evidence here, and they're strong on their own.
- **The numbers tell the story** — use them in your LinkedIn/portfolio blurb: *bilingual voice order completed end-to-end, 715 TRY total priced from a live spreadsheet menu, two-step pending→confirmed safety, closed-store orders rejected with 409 and logged.*
- **Replace example data before any live demo for a client:** the menu in the video carries `PriceIsExample = TRUE` sample prices — real prices and descriptions should be in place before showing this to a restaurant prospect.
- **Privacy:** the demo phone number stays masked (`***3456`) in every public artifact, matching the system's own log masking.
