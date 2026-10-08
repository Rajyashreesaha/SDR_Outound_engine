# Outreach Radar — SDR Outbound

A daily prospecting queue for an SDR selling to restaurant chains in India.
Each day it surfaces F&B brands that match the ICP (5+ outlets, any Indian city)
and haven't been contacted yet. One click logs a brand as processed, and it never
comes back.

**[Live demo](https://sdr-outbound-radar.vercel.app)** · [Architecture](ARCHITECTURE.md) ·
Next.js · Supabase · Google Places API · built with Claude Code

> This repo is a public overview. The source code is private.
> In the demo, brand names come from public listings and all contact details
> are fictional sample data.

![Brand detail: outlets, POS, decision-makers and a sample menu](docs/screenshots/brand-detail.png)

## What it does

- **Daily queue.** Up to 10 ICP-matched brands the SDR hasn't processed,
  newest discoveries first, with search and filters for city and POS.
- **Brand detail.** Outlets, cities, current POS, decision-makers with one-click
  WhatsApp (pre-filled opener) and LinkedIn, a sample menu with prices, and notes.
- **Mark as processed.** The app's only write action. It logs the brand to an
  audit table, removes it from the queue instantly, and it stays gone after a reload.
- **Processed log.** Every brand reviewed, with outcome and notes, exportable to CSV.
- **Last synced.** The header shows when brand data was last refreshed.

![The queue with a brand open](docs/screenshots/queue-with-detail.png)

## How it works

```
candidate generator  ──►  candidate names  ──►  brand discovery  ──►  Supabase  ──►  web app
 (Places Nearby Search)    (2+ city chains)     (Places Text Search,                  (queue,
                                                 5+ outlet ICP check)                  processed log)
```

1. **Generate candidates.** Samples restaurants across 20 Indian cities, rotating
   through food types and areas so every run sees new places. Names seen in 2+
   cities become candidates.
2. **Discover brands.** Verifies each candidate against the ICP (5+ outlets in
   India). Brands that are excluded, already processed or already tried are
   skipped before any API quota is spent, and runs resume after a quota limit.
3. **Work the queue.** A brand leaves the queue for good once it's processed.

## Stack

- **Frontend:** Next.js 16 (App Router), React 19, TypeScript, Tailwind CSS 4, Motion
- **Data:** Supabase (Postgres with Row Level Security)
- **Discovery:** Node.js scripts using the Google Places API (New)
- **Hosting:** Vercel

## Design notes

Brands and decision-makers are objects with relationships, and **"Mark as
processed" is an action that changes a brand's state and writes an audit
record**: the same shape (objects, actions, state, an audit trail) used in
ontology-style modeling, built as an ordinary relational app.

- **Objects:** brands, decision-makers and menu items, linked by foreign keys.
- **Action:** marking a brand processed is the app's only write; everything else reads.
- **State transition:** a brand leaves the queue for good once it has a processed record.
- **Audit trail:** each processed record stores who, when, the outcome and notes
  as its own row instead of a field changed on the brand.
