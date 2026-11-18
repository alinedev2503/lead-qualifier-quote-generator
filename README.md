# Lead Qualifier / Instant Quote Generator

A mini app that can be linked or embedded on a business website. Visitors answer a
short, smart question flow; the app either scores them (hot / warm / cold) or
produces an instant, clearly explained estimate based on the business's own
pricing rules. Every submission is written to a Google Sheet owned by the client.

> Portfolio repository for **Upwork Project B — Lead qualifier / quote generator**.
> The code is being prepared now; the gallery (mobile + desktop prints, sample
> leads sheet and a 45–60 s video) will be added only after the demo is ready.

---

## What the service delivers

| Tier | Delivered |
|---|---|
| **Starter — Lead Qualifier Mini App** | 5–8 questions, AI hot/warm/cold score, leads saved to Google Sheets |
| **Standard — Instant Quote Generator** | Client services + price rules, AI-written instant quote, e-mail alert |
| **Advanced — Qualifier + Quote + Dashboard** | Both flows, simple lead dashboard, custom branding |

All tiers are responsive and deployed live. Source code and content upload are
included from Standard; design customization is exclusive to Advanced.

---

## How it works

```
visitor ──▶ embeddable widget / standalone link
                 │  5–8 questions (qualification) or service inputs (quote)
                 ▼
          rules engine  ── deterministic score or price (never invented by the AI)
                 ▼
          Google Gemini ── human-readable explanation of the result
                 ▼
          Google Sheets ── client-owned lead row (+ e-mail alert on Standard/Advanced)
                 ▼
          lead dashboard (Advanced only)
```

**Key rule:** the numbers come from the client's pricing rules. Gemini only
explains them. The same applies to scoring: weights are explicit and auditable.

---

## Planned stack

- React 19 + TypeScript + Vite 8
- Tailwind CSS 4 + Lucide icons
- Node.js + Express proxy for `@google/genai` (API key stays server-side)
- Google Sheets API via service account (JWT, no SDK) — with local JSONL fallback
- PWA install support and branded theme tokens

## Reused patterns

This repo deliberately reuses proven pieces from the existing portfolio instead
of starting from zero:

- `Agent-StudyDeck-Mini-Guia-de-Estudos-` — `BrandContext` (logo, colors, app
  name), PWA setup, server-side Gemini proxy and deployment docs.
- `faq-lead-chatbot` — Google Sheets sink with local fallback, embeddable
  Shadow-DOM widget and multi-client config layout.
- `Agent-EducaFinance-AI` — question → deterministic analysis → AI explanation
  flow (`BudgetDiagnostic503020.tsx` + `/api/diagnose`).

## Repository layout

```
docs/ARCHITECTURE.md            design decisions and data flow
examples/questions.example.json 5–8 question flow (Starter)
examples/pricing-rules.example.json explicit price rules (Standard)
.env.example                    required environment variables
```

## Quickstart

```bash
npm install
cp .env.example .env
npm run dev
```

Configure `SHEET_ID` and `GOOGLE_SERVICE_ACCOUNT` to persist leads, or leave
`LEAD_SINK=local` for a demo that writes to `data/leads.jsonl`.

See [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) for the full design.
