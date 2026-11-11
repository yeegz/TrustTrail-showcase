# Threat model

Scope: the browser app as shipped (static files + IndexedDB). There is no backend. Framework used for thinking: OWASP ASVS 5.0.0; this is not a certification claim.

**Assets:** screenshots and text containing a third party's messages and payment details; derived findings; exported reports.

**Trust boundary:** one browser profile on one device. Anything in evidence (OCR text, pasted text, file bytes) is hostile data.

| Threat | Control | Evidence |
|--------|---------|----------|
| Evidence leaving the device | No network code in the app; fonts, OCR worker, WASM and language data are same-origin; CSP `default-src 'self'; connect-src 'self'`; `Referrer-Policy: no-referrer`. | `tests/adversarial` static grep for `fetch(`, `XMLHttpRequest`, `WebSocket`, `sendBeacon`; network tab during OCR shows only `/ocr/*`. |
| Prompt injection in evidence | There is no model. Rules are regular expressions; text cannot change control flow. | Injection fixtures must not change the outcome. |
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
