# Architecture decisions

Dated 11 Nov 2026. Each has a trigger that would reopen it.

| # | Decision | Why | Reopen when |
|---|----------|-----|-------------|
| 001 | **Local-first web app, no server.** React + TypeScript + Vite, IndexedDB, OCR in a Web Worker. | The research ranks "buyers will not upload private chats" as the biggest risk (H2) and "evidence leaking" as threat #1. Running everything in the browser removes both, costs nothing to host, and needs no accounts. This is the research's Architecture C promoted to the product. | A feature genuinely needs a server (shared cases, a licensed provider that needs a secret key). |
| 002 | **Departure from the research: no Next.js / FastAPI / PostgreSQL.** | Follows from 001. Tenant isolation and row-level security have nothing to protect when there is no shared store. The engine is pure TypeScript and could move behind an API unchanged. | Same as 001. |
| 003 | Deterministic, versioned rules. No ML, no LLM, no score. | No lawful labelled data exists for this wedge; a probability would be invented. | Lawful, representative, labelled data exists and a calibrated model beats the rules on a held-out set. |
| 004 | Tesseract.js (Apache-2.0) with English and Malay data, self-hosted under `/ocr/`. | No request leaves the origin. Other scripts are stored and shown but not analysed. | Benchmark favours another engine, or another analysis language is validated. |
| 005 | Translation off. NLLB excluded (CC-BY-NC). | Licence and meaning-change risk. | A commercially licensed model is verified and benchmarked. |
| 006 | Every outside service is a state, all off (`ADAPTERS` in `src/data/countries.ts`). VirusTotal and public urlscan never receive evidence. | Licence and privacy terms in the research. | Owner approval, per provider. |
| 007 | The owner runs official checks themselves and records the result (`USER_REPORTED`). | Semak Mule has a CAPTCHA and no public API; ScamShield has no third-party API. No scraping, no CAPTCHA bypass. | An official API is published. |
| 008 | Evidence graph is derived in memory on every change, not stored. | Tens of nodes per case; recomputing is instant and a stale finding cannot survive an edit. | A case grows past a few thousand spans. |
| 009 | 25 locales registered; only English is "reviewed". The other 24 are machine-drafted previews and labelled so. Analysis: English and Malay, both BETA. | Honest tiers on two axes (interface vs analysis). | A native reviewer signs off a locale; fixtures by native authors exist. |
| 010 | Redaction of third-party identifiers is ON by default in reports; the owner can reveal each one. | Defamation and privacy risk in a shareable report. | Legal review says otherwise. |
| 011 | Default retention 30 days, hard delete with a counts-only receipt. | Storage limitation. | Owner changes the default. |
| 012 | No Go worker, no graph database, no Kubernetes. | Nothing to justify them. | A measured bottleneck. |
