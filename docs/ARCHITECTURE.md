# Architecture — Lead Qualifier / Quote Generator

## Principles

1. **Deterministic numbers, explanatory AI.** Score weights and price rules are
   plain data, versioned in the client config. Gemini never invents a price, a
   discount or a score weight; it receives the computed result and writes a clear
   explanation.
2. **One codebase, many clients.** Each client is a folder under `clients/<slug>/`
   with a `config.json`, a question flow and a pricing-rules file. The deployed
   instance selects the client with `LEAD_CLIENT`.
3. **A lead is never lost.** Every sink falls back to local JSONL if Google
   Sheets is temporarily unavailable.
4. **Embed first.** The widget is a single `<script>` tag with Shadow DOM so it
   does not inherit CSS from the host site. A standalone landing link uses the
   same API.

## Data flow

```
GET  /api/config?client=<slug>   -> public config (branding + questions)
POST /api/qualify                -> { answers } -> { score, band, reasons }
POST /api/quote                  -> { answers } -> { lines, total, assumptions }
POST /api/leads                  -> { contact, result, sessionId } -> sink
GET  /api/leads                  -> dashboard (Advanced, token protected)
```

`POST /api/qualify` and `POST /api/quote` call the rules engine first, then call
Gemini with the computed result in the system prompt and a strict JSON response
schema. If Gemini fails, the endpoint still returns the deterministic result with
a neutral fallback explanation.

## Scoring (Starter)

Each answer carries a weight. The sum maps to bands:

```
>= 70  hot
40-69  warm
< 40   cold
```

Weights and band thresholds live in `clients/<slug>/config.json`; they are not
hard-coded in the prompt. The AI receives `band`, `score` and `reasons[]` and only
phrases them for the visitor.

## Quoting (Standard)

Price rules are explicit lines:

```json
{
  "currency": "USD",
  "items": [
    { "id": "base", "label": "Base setup", "amount": 300 },
    { "id": "page", "label": "Extra page", "amount": 75, "max": 10 }
  ],
  "multipliers": [
    { "when": "timeline == 'rush'", "factor": 1.25, "label": "Rush delivery" }
  ]
}
```

The server computes `lines[]`, `subtotal`, `multipliers[]` and `total`. Gemini
receives those values and returns a short, friendly explanation plus a list of
assumptions. The client can review the rules in plain text before launch.

## Storage

`LEAD_SINK` selects `local` (JSONL + CSV, demos), `sheets` (service account JWT,
no SDK) or `webhook`. The Google Sheets implementation is shared with the
`faq-lead-chatbot` pattern: sign an RS256 JWT with the service-account key,
exchange it for an access token, then append a row to `Leads!A:G`.

The lead inbox route requires `ADMIN_TOKEN` in production and returns 404 without
it. Leads are never exposed through the public widget API.

## Branding (Advanced)

`BrandContext` exposes theme tokens (primary color, logo, app name, tagline,
contact e-mail). The same tokens drive the widget, the standalone page and the
dashboard. This is the same approach used by `Agent-StudyDeck`.

## Testing before delivery

- realistic answers for hot / warm / cold, including edge values
- quote with zero optional items, with max items, and with a rush multiplier
- Sheets down -> local fallback -> recovery
- embed on a page with conflicting CSS (Shadow DOM isolation)
- mobile widths 320 / 375 / 768 / 1280
