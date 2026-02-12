# TCG Vault Architecture

## 1. System Overview
TCG Vault is designed as a modular service-oriented application for card inventory, recognition, and market data enrichment.

### Primary Services
1. **Auth & User Service**
   - Registration/login
   - User profiles and roles (collector, shop_owner, admin)

2. **Catalog Service**
   - Canonical card metadata across Pokémon, Yu-Gi-Oh!, MTG
   - Set metadata and card identifiers

3. **Collection Service**
   - User inventory records
   - Storage locations and quantities
   - Collection-level views and aggregates

4. **Ingestion Service**
   - Image upload handling
   - OCR and card identification pipeline
   - Manual entry assist via autosuggest

5. **Market Data Service**
   - eBay sold-listing ingestion (last 3 sold comps)
   - TCGPlayer/Card Kingdom price snapshots
   - Price normalization and caching

6. **API Gateway / BFF**
   - Unified API for web and mobile clients
   - Auth token validation and request orchestration

## 2. High-Level Data Flow
1. User scans/uploads a card image.
2. Ingestion Service queues recognition job.
3. Recognized card is mapped to canonical catalog record.
4. Collection Service creates/updates inventory row.
5. Market Data Service fetches and stores latest price/sold comps.
6. Client receives merged card + inventory + pricing payload.

## 3. Proposed Data Model (PostgreSQL)

### users
- id (uuid, pk)
- email (unique)
- password_hash
- display_name
- role (collector/shop_owner/admin)
- created_at, updated_at

### tcg_games
- id (pk)
- slug (`pokemon`, `yugioh`, `mtg`)
- name

### sets
- id (uuid, pk)
- game_id (fk -> tcg_games)
- external_set_id
- name
- release_date
- total_cards

### cards
- id (uuid, pk)
- game_id (fk)
- set_id (fk)
- external_card_id
- card_number
- name
- rarity
- metadata_jsonb

### card_variants
- id (uuid, pk)
- card_id (fk)
- finish (normal/holo/reverse)
- edition (unlimited/1st edition/etc)
- language
- is_graded (bool)

### collections
- id (uuid, pk)
- user_id (fk)
- name
- type (personal/store/trade)

### inventory_items
- id (uuid, pk)
- collection_id (fk)
- card_variant_id (fk)
- condition
- quantity
- storage_location
- notes
- created_at, updated_at

### card_images
- id (uuid, pk)
- inventory_item_id (fk)
- image_url
- source (camera/upload)
- processing_status (queued/processed/failed)

### price_snapshots
- id (uuid, pk)
- card_variant_id (fk)
- source (ebay_sold/tcgplayer/cardkingdom)
- currency
- price_amount
- observed_at
- raw_payload_jsonb

## 4. API Design (MVP)

### Auth
- `POST /api/v1/auth/register`
- `POST /api/v1/auth/login`
- `GET /api/v1/auth/me`

### Catalog
- `GET /api/v1/games`
- `GET /api/v1/games/:gameId/sets`
- `GET /api/v1/sets/:setId/cards`
- `GET /api/v1/cards/search?q=`

### Collection
- `POST /api/v1/collections`
- `GET /api/v1/collections`
- `GET /api/v1/collections/:id/items`
- `POST /api/v1/collections/:id/items`
- `PATCH /api/v1/items/:itemId`

### Ingestion
- `POST /api/v1/ingestion/uploads` (returns presigned upload URL)
- `POST /api/v1/ingestion/scan` (start recognition)
- `GET /api/v1/ingestion/jobs/:jobId`

### Market Data
- `GET /api/v1/cards/:cardId/market`
  - Returns:
    - last 3 sold eBay listings
    - latest TCGPlayer price
    - latest Card Kingdom price
    - timestamp and staleness indicator

## 5. External Integrations

### eBay Sold Listings
- Fetch completed/sold listings per canonical card query.
- Store top three most relevant sold entries after normalization.
- Save listing URL, title, sold price, sold date, shipping where available.

### TCGPlayer + Card Kingdom
- Pull current listed prices via API/partner feed where permitted.
- If APIs are unavailable, maintain pluggable connector interfaces.
- Respect rate limits and terms of service.

## 6. Background Job Strategy
- Queue-based ingestion for image recognition and market refresh.
- Scheduled jobs:
  - hot cards refresh every 1-3 hours
  - long-tail cards refresh every 24 hours
- Retry policy with dead-letter queue for failed provider calls.

## 7. Security & Compliance
- JWT-based auth with rotating refresh tokens.
- Encrypt sensitive data at rest and in transit.
- Signed URLs for private image uploads/access.
- PII minimization and configurable image retention windows.

## 8. Scalability Considerations
- Read-heavy endpoints use caching layer (Redis).
- Partition price snapshots by date/source for query performance.
- Event-driven processing for ingestion/market updates.
- Horizontal scaling for worker pools during release spikes.

## 9. MVP Delivery Plan

### Phase 1
- Auth, catalog browsing, manual card entry, base inventory

### Phase 2
- Image upload + async recognition pipeline

### Phase 3
- eBay sold listings (last 3) + TCGPlayer/Card Kingdom pricing

### Phase 4
- Set completion dashboards and portfolio valuation summaries
