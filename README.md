# TCG Vault — Collector Inventory & Price Tracking

## Vision
TCG Vault is a web app for collectors and shop owners to catalog and price-check trading cards across major games:
- Pokémon
- Yu-Gi-Oh!
- Magic: The Gathering

Users can add cards three ways:
1. **Scan with camera** (OCR + card recognition)
2. **Upload card photos**
3. **Manual entry** (name/set/number/condition/variant)

The goal is to make collection management simple while surfacing near-real-time market signals from sold listings and trusted marketplaces.

## Core Product Outcomes
- Know exactly what cards you own and where they are stored.
- Organize by **game → set → card** with quantities, condition, and language/printing details.
- View latest sold comps (e.g., last 3 sold eBay listings).
- Compare pricing across TCGPlayer and Card Kingdom.
- Support both individual collectors and small card shops.

## MVP Features

### 1) Account + Collection Management
- User authentication and profiles
- Create multiple collections (Personal, Store Inventory, Trade Binder)
- Card-level inventory fields:
  - Game, set, card number, card name
  - Variant (holo/reverse/1st edition/alt art)
  - Condition, language, grading info
  - Quantity and storage location (binder/box/shelf)

### 2) Card Intake (Scan/Photo/Manual)
- Mobile camera flow for scanning cards
- Image upload with asynchronous recognition pipeline
- Manual fallback form with autosuggest for card metadata

### 3) Price & Sold Listing Insights
- Show **last three sold eBay listings** with:
  - title, sold price, sold date, shipping, listing URL
- Show marketplace snapshots from:
  - TCGPlayer
  - Card Kingdom
- Normalize and cache prices by card + variant + condition mapping

### 4) Search, Filters, and Views
- Global search by game, set, card name, number
- Filters for condition, quantity, value bands, and duplicates
- Set-level completion view (owned vs total in set)

## Future Enhancements
- Trade matching between users
- Portfolio analytics (value trends over time)
- Alerts for price drops/spikes
- Bulk import/export (CSV, scanner integrations)
- POS-lite tools for shop inventory management

## Suggested Tech Stack
- **Frontend:** Next.js + TypeScript + Tailwind
- **Backend:** Node.js (NestJS or Express) + TypeScript
- **Database:** PostgreSQL
- **Queue/Jobs:** Redis + BullMQ
- **Storage:** S3-compatible object storage for images
- **Search:** PostgreSQL full-text (MVP), optional Elasticsearch later
- **Observability:** OpenTelemetry + structured logging

## Non-Functional Requirements
- Fast card search (<300ms for common queries)
- Price refresh jobs with source-aware rate limiting
- Idempotent card ingestion pipeline
- Clear audit trail for pricing snapshots
- Privacy-first handling of user-uploaded media

## Repository Contents
- `README.md` — product vision and MVP plan
- `ARCHITECTURE.md` — proposed system design, schema, APIs, and workflows
