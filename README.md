# Cronos402 MCP Data Service

Data persistence and analytics service for Cronos402 MCP servers.

## Setup

1. Copy `.env.example` to `.env`
2. Set `DATABASE_URL` and `INGESTION_SECRET`
3. Optionally set `MODERATION_SECRET` for admin endpoints
4. Run migrations and start the server

## API Endpoints

### Public

- `POST /ingest/event` - Ingest usage events
- `POST /index/run` - Trigger indexing
- `GET /events/summary?origin=` - Event summary
- `GET /servers?include=approved|all&sort=score|recent&limit=&offset=` - List servers
- `GET /server/:id` - Get server details

### Admin (requires `Authorization: Bearer ${MODERATION_SECRET}`)

- `POST /servers/:id/moderate` - Update moderation status
- `POST /score/recompute` - Recompute quality scores

## Moderation

- Status: `pending | approved | rejected | disabled | flagged`
- Quality score: 0-100, computed from success rate, latency p95, error rate

## Development

```bash
pnpm dev
```

## Database

```bash
pnpm db:generate  # Generate migrations
pnpm db:migrate   # Run migrations
```
