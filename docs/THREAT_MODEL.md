# Threat model

Scope: the browser app as shipped (static files + IndexedDB). There is no backend. Framework used for thinking: OWASP ASVS 5.0.0; this is not a certification claim.

**Assets:** screenshots and text containing a third party's messages and payment details; derived findings; exported reports.

**Trust boundary:** one browser profile on one device. Anything in evidence (OCR text, pasted text, file bytes) is hostile data.

| Threat | Control | Evidence |
|--------|---------|----------|
| Evidence leaving the device | No network code in the engine or UI except the one opt-in AI call (next row); fonts, OCR worker, WASM and language data are same-origin; CSP `default-src 'self'; connect-src 'self' generativelanguage.googleapis.com`; `Referrer-Policy: strict-origin`. | `tests/adversarial` static grep for `fetch(`, `XMLHttpRequest`, `WebSocket`, `sendBeacon`; network tab during OCR shows only `/ocr/*`. |
| Evidence sent to Google by the optional AI card | Off until the person presses "Ask Google Gemini" and confirms a dialog that states what is sent and that Google's free tier may use it to improve its products; asked every time; phone, account, email and ID numbers are blocked out before serialising; names and message text are NOT removed. On-device AI is offered instead wherever the browser has it. | `tests/ai.test.ts` (numbers absent from the request); real-Chrome run: 0 external requests before consent, 1 POST after, numbers absent from the body. |
| The AI inventing or judging | Reply must be JSON matching a schema; each note's quote must be a word-for-word substring of the line that was sent or the note is dropped; notes about lines the rules already cited are dropped; links are stripped; English verdict wording ("scam", "safe to buy", "legit") causes the whole answer to be withheld; the AI output is never an input to the engine and is left out of the report. | `tests/ai.test.ts`. Verdict filtering is English-only: other languages rely on the instruction and the "can be wrong" label. |
| Public Gemini key abused | Key restricted to one API and to this site's referrers; project has no billing account, so the ceiling is the free allowance. Referrers can be forged by non-browser clients. | A request without the site's referrer returns 403. |
| Prompt injection in evidence | The rule engine has no model. Rules are regular expressions; text cannot change control flow. | Injection fixtures must not change the outcome. |
| XSS through OCR or pasted text | React escaping only; no `dangerouslySetInnerHTML`; URLs are never rendered as links; strict CSP without `unsafe-eval` (only `wasm-unsafe-eval` for Tesseract). | XSS payload fixtures survive as inert text. |
| SSRF / link following | Evidence URLs are strings. The only outbound links are the official check sites from `src/data/countries.ts`, opened by the user with `rel="noopener noreferrer"`. | Fixtures with `127.0.0.1` and metadata addresses. |
| Malicious images, decompression bombs | Magic-byte sniffing (PNG/JPEG/WebP), 10 MB and 40 MP caps, max 10 images, redraw onto a canvas and re-encode as PNG (drops EXIF and any payload) before storage or OCR. | `src/ocr/image.ts`. |
| Look-alike domains | Punycode shown beside the written form, mixed-script detection, a small TR39-style skeleton against known brands. Partial table, documented as such. | Homoglyph fixtures. |
| False accusation through a shared report | Disclaimer on every report; outcomes describe the transaction; third-party identifiers blocked out by default; share warning. | Report view; JSON export. |
| Excessive retention | Cases expire (default 30 days) and are purged on next open; hard delete of rows and image blobs in one transaction; receipt stores counts only. | `src/store/db.ts`. |
| Someone else using the same browser profile | NOT mitigated. Cases are readable by anyone with access to the unlocked device. Stated here rather than hidden. | n/a |
| Supply chain | Lockfile; few runtime dependencies (react, react-router, idb, tesseract.js, two fontsource packages). | `package-lock.json`. |

**Known gaps:** no at-rest encryption; the confusables table is not the full Unicode set; `style-src 'unsafe-inline'` is allowed for React inline styles; no automated accessibility audit has been run; no independent security review.

**Questions for qualified legal review (not legal advice):** PDPA / GDPR position when all processing is on the user's device; defamation exposure of exported reports; whether describing Semak Mule results is permitted.
