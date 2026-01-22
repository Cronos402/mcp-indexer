![Cronos402 Logo](https://raw.githubusercontent.com/Cronos402/assets/main/Cronos402-logo-light.svg)

# Cronos402 MCP Indexer

Analytics and server directory service for the Cronos402 ecosystem.

Production URL: https://indexer.cronos402.dev

## Overview

The MCP Indexer provides data persistence, analytics, and server discovery for the Cronos402 ecosystem. It ingests usage events from MCP servers, computes quality scores, and maintains a public directory of available servers. Developers can discover services, monitor performance, and moderate server listings through this centralized index.

## Architecture

- **Framework**: Express.js with TypeScript
- **Database**: Drizzle ORM with PostgreSQL
- **Analytics**: Real-time event aggregation
- **Moderation**: Admin-controlled approval system
- **Quality Scoring**: Automated performance metrics

## Features

- Usage event ingestion from MCP servers
- Real-time analytics and metrics
- Public server directory
- Quality score computation
- Moderation and approval workflow
- Performance tracking (success rate, latency, errors)
- Search and filtering
- Admin moderation tools

## Setup

1. Copy `.env.example` to `.env`:

```env
DATABASE_URL=postgresql://user:password@localhost:5432/cronos402
INGESTION_SECRET=your-ingestion-secret
MODERATION_SECRET=your-moderation-secret
PORT=3010
```

2. Run migrations:

```bash
pnpm db:generate
pnpm db:migrate
```

3. Start the server:

```bash
pnpm install
pnpm dev
```

## API Endpoints

### Public Endpoints

#### POST /ingest/event
Ingest usage events from MCP servers.

Request:
```json
{
  "serverId": "server-123",
  "toolName": "get_weather",
  "success": true,
  "latencyMs": 145,
  "errorType": null,
  "timestamp": "2024-01-22T12:00:00Z"
}
```

Headers:
- `Authorization: Bearer ${INGESTION_SECRET}`

#### POST /index/run
Trigger manual indexing of servers.

#### GET /events/summary
Get event summary for a server.

Query parameters:
- `origin` (required): Server origin URL

Response:
```json
{
  "totalEvents": 1000,
  "successRate": 0.98,
  "avgLatencyMs": 150,
  "p95LatencyMs": 250,
  "errorRate": 0.02
}
```

#### GET /servers
List available MCP servers.

Query parameters:
- `include`: `approved` | `all` (default: approved)
- `sort`: `score` | `recent` (default: score)
- `limit`: Number of results (default: 20)
- `offset`: Pagination offset (default: 0)

Response:
```json
{
  "servers": [
    {
      "id": "server-123",
      "name": "Weather API",
      "description": "Get weather data for any city",
      "url": "https://api.example.com/mcp",
      "qualityScore": 95,
      "status": "approved",
      "createdAt": "2024-01-01T00:00:00Z"
    }
  ],
  "total": 50,
  "limit": 20,
  "offset": 0
}
```

#### GET /server/:id
Get detailed server information.

Response:
```json
{
  "id": "server-123",
  "name": "Weather API",
  "description": "Get weather data for any city",
  "url": "https://api.example.com/mcp",
  "qualityScore": 95,
  "status": "approved",
  "stats": {
    "totalCalls": 10000,
    "successRate": 0.98,
    "avgLatency": 150,
    "p95Latency": 250
  },
  "tools": [
    {
      "name": "get_weather",
      "description": "Get current weather",
      "callCount": 5000
    }
  ]
}
```

### Admin Endpoints

Require `Authorization: Bearer ${MODERATION_SECRET}`

#### POST /servers/:id/moderate
Update moderation status.

Request:
```json
{
  "status": "approved" | "rejected" | "flagged" | "disabled",
  "reason": "Optional moderation reason"
}
```

#### POST /score/recompute
Recompute quality scores for all servers.

## Moderation Statuses

- **pending**: Newly submitted, awaiting review
- **approved**: Verified and listed publicly
- **rejected**: Not suitable for directory
- **disabled**: Temporarily disabled by admin
- **flagged**: Requires attention

## Quality Score

Computed from 0-100 based on:

- **Success Rate** (40%): Percentage of successful calls
- **Latency P95** (30%): 95th percentile response time
- **Error Rate** (20%): Percentage of errors
- **Uptime** (10%): Server availability

Formula:
```typescript
const score =
  (successRate * 40) +
  (latencyScore * 30) +
  ((1 - errorRate) * 20) +
  (uptime * 10);
```

## Development

```bash
# Install dependencies
pnpm install

# Start development server
pnpm dev

# Build for production
pnpm build

# Start production server
pnpm start
```

## Database Operations

```bash
# Generate migrations
pnpm db:generate

# Run migrations
pnpm db:migrate

# Open Drizzle Studio
pnpm db:studio
```

## Database Schema

Core tables:

- `servers` - Registered MCP servers
- `usage_events` - Event logs from servers
- `server_stats` - Aggregated statistics
- `moderation_log` - Admin actions history
- `quality_scores` - Computed performance scores

## Deployment

### Production Build

```bash
pnpm build
NODE_ENV=production pnpm start
```

### Docker

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package.json pnpm-lock.yaml ./
RUN npm install -g pnpm && pnpm install --frozen-lockfile
COPY . .
RUN pnpm build
EXPOSE 3010
CMD ["pnpm", "start"]
```

### Environment Requirements

- PostgreSQL database
- HTTPS endpoint
- Secure ingestion secret
- Admin moderation secret

## Monitoring

- `GET /health` - Service health check
- `GET /metrics` - Prometheus metrics
- Database query monitoring
- Event ingestion rate tracking

## Integration

### Server Registration

Servers automatically register when sending first event:

```typescript
import axios from 'axios';

await axios.post('https://indexer.cronos402.dev/ingest/event', {
  serverId: 'my-server',
  toolName: 'my_tool',
  success: true,
  latencyMs: 100
}, {
  headers: {
    'Authorization': `Bearer ${INGESTION_SECRET}`
  }
});
```

### Client Discovery

Find available servers:

```typescript
const response = await fetch(
  'https://indexer.cronos402.dev/servers?include=approved&sort=score'
);
const { servers } = await response.json();
```

## Resources

- **Documentation**: [docs.cronos402.dev/indexer](https://docs.cronos402.dev)
- **Dashboard**: [cronos402.dev](https://cronos402.dev)
- **SDK**: [npmjs.com/package/cronos402](https://www.npmjs.com/package/cronos402)
- **GitHub**: [github.com/Cronos402/mcp-indexer](https://github.com/Cronos402/mcp-indexer)

## License

MIT
