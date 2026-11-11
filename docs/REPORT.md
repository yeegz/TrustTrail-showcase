# TrustTrail build report

11 November 2026 · version 1.0.0 · live at **https://trusttrail-yeegz.web.app**

## What it is
A private tool for one moment: you are about to pay someone you met online for a secondhand item, and something feels off. You add the listing, the chat and the payment request. TrustTrail shows the exact words behind each warning sign, which details changed, and which checks nobody has done yet. It never calls a person a scammer and never calls a purchase safe.

## What was built
| Area | Delivered |
|------|-----------|
| Analysis engine | 14 versioned rules from the research + 1 for checks the user runs; contradiction finder (price, payee, account, storage, model, location); copied-review clustering; link spelling inspector; six-outcome ordered aggregation; evidence graph; timeline |
| Reading screenshots | On-device OCR (English + Malay), one row per chat message, word-level boxes, editable text with history |
| App | Home, 3-step new case, case workbench (Overview, Evidence, Findings in three views, Checks, Report), Settings, a step-by-step guide in every case |
| AI (optional) | "AI suggestions": says what is going on in plain words, then 2 to 4 next steps tailored to the deal, each with a ready-to-send message in the seller's language, and up to three extra lines worth a look. On-device where the browser has a built-in model; otherwise Google's free Gemini, only after an explicit yes each time, with numbers blocked out. The AI never decides the result |
| Privacy | No server, no account, no analytics. Images re-encoded on arrival. Third-party numbers hidden in reports by default. Cases delete themselves after 30 days; hard delete with a receipt |
| Languages | 25 interface languages (English reviewed, 24 machine-drafted previews), right-to-left support. Analysis in English and Malay |
| Platform | Installable, works offline, light and dark, phone to desktop |

There is **no backend**, on purpose: everything runs in the browser, so there is nothing server-side to operate, secure or pay for. The reasoning is in `DECISIONS.md`.

## Verified, and how
All of this was run on 11 Nov 2026 against the live site with real headless Chrome (`node scripts/verify.mjs https://trusttrail-yeegz.web.app`).

| Check | Result |
|-------|--------|
| Automated tests | 1,412 passing across 11 files (`npm test`) |
| Type check + production build | Clean (`npm run build`) |
| Fresh case with a real image through OCR | Result shown about 5 s after pressing "Check this deal" (includes first download of the OCR engine) |
| Requests to any other server during a full session (without pressing "Ask Google Gemini") | 0 |
| Free Gemini path on the live site, real Chrome (`node scripts/ai-check.mjs <url>`) | 0 requests before consent; 1 request after; phone and account numbers absent from what was sent; a summary and 3 tailored next steps returned, with messages written in the seller's language (Malay) |
| Console or page errors, 4 device profiles | 0 |
| Sideways scrolling at 390, 820, 1440 wide (and 320 by hand) | None |
| Offline worker registered | Yes |
| Automated accessibility audit (axe-core, WCAG 2.2 AA, 11 screens, light + dark) | 0 violations (118 found and fixed) |
| Prompt-injection text in evidence changing the result | 0 of 130 attempts |
| Disguised links flagged / real marketplace and bank hosts wrongly flagged | 17 of 17 / 0 of 15 |
| Findings that quote their source word-for-word | 537 of 537 |

Screenshots of every screen at each size are in `docs/screens/`.

## What the numbers do NOT mean
- The rule fixtures (840 cases) are **fictional and written by the same author as the rules**. On first run 84 of them disagreed with the engine; the engine was then fixed until they passed. 100% on that set is regression evidence, not real-world accuracy. Details in `BENCHMARK.md`.
- No native speaker has reviewed the Malay patterns or any of the 24 translations.
- No real users have tried it. Whether it helps people decide better than a checklist is still a hypothesis.
- Official check routes are verified for Malaysia and Singapore only.
- OCR accuracy on real phone screenshots has not been measured.
- No independent security review. Accessibility has passed an automated audit only; no screen-reader testing with real users yet.

## Known rough edges (from the test pass)
- A negotiated discount shows up as "the price changed" (labelled "Worth checking", with "a price you agreed together" as the innocent reading).
- Any unfamiliar link on the same line as a payment word counts as a payment link, which is a serious sign.
- Three identical default reviews ("Item received. Thanks.") count as copied reviews.
- A 10-digit bank account starting with 01 is read as a Malaysian mobile number.

## Cost
Nothing here can produce a bill: Firebase Hosting is on the free plan and the Google Cloud project has **no billing account attached** (checked with `gcloud billing projects describe trusttrail-yeegz`). The Gemini key is the free tier on that same project; if the allowance runs out, the button reports it and the rest keeps working.

## Where things are
- Live: https://trusttrail-yeegz.web.app (Firebase Hosting, free plan, project `trusttrail-yeegz`)
- Code: private `yeegz/TrustTrail` · Showcase: public `yeegz/TrustTrail-showcase`
- Next steps and resume instructions: `HANDOFF.md`
