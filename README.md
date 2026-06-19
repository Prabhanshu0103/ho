# Damage Claim Adjudication System

An AI-powered full-stack web application that automates the adjudication of damage claims (car, laptop, package) using GPT-4o vision analysis. Adjusters submit a claim with image evidence and a conversation transcript; the system returns a structured verdict with confidence score and reasoning.

---

## Features

- **Claims Queue** — paginated, searchable table of all submitted claims with live status badges
- **Dashboard** — real-time stats: total claims, pending adjudication, supported vs. contradicted verdicts, breakdown by severity and object type
- **New Claim Form** — dynamic form accepting user ID, claim object type, transcript, and multiple evidence image URLs
- **Claim Detail** — full forensic review: transcript, image gallery, AI verdict panel with confidence score and line-by-line reasoning
- **GPT-4o Vision Adjudication** — one-click adjudication that sends all image URLs and the claim transcript to GPT-4o, returning a structured verdict (`supported` / `contradicted` / `inconclusive`), severity (`low` / `medium` / `high`), confidence score (0–100), and human-readable reasoning

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | React 19 + Vite 7, TypeScript, Tailwind CSS v4, shadcn/ui |
| Routing | Wouter |
| Data fetching | TanStack Query v5, Orval-generated React Query hooks |
| Forms | react-hook-form + Zod |
| API Contract | OpenAPI 3.1 spec → Orval codegen |
| Backend | Express 5, Node.js 24, TypeScript |
| Database | PostgreSQL + Drizzle ORM |
| AI | OpenAI GPT-4o (vision) |
| Monorepo | pnpm workspaces |

---

## Project Structure

```
artifacts-monorepo/
├── artifacts/
│   ├── api-server/          # Express 5 REST API
│   │   └── src/
│   │       ├── routes/
│   │       │   └── claims.ts        # CRUD + adjudicate + stats endpoints
│   │       └── lib/
│   │           └── adjudicate.ts    # GPT-4o vision adjudication logic
│   └── claims-ui/           # React + Vite frontend
│       └── src/
│           ├── pages/
│           │   ├── dashboard.tsx
│           │   ├── claims/
│           │   │   ├── index.tsx    # Claims queue
│           │   │   ├── new.tsx      # New claim form
│           │   │   └── detail.tsx   # Claim detail + adjudication
│           └── components/
│               ├── app-layout.tsx
│               └── status-badges.tsx
├── lib/
│   ├── api-spec/            # OpenAPI 3.1 YAML spec (source of truth)
│   ├── api-client-react/    # Orval-generated React Query hooks + Zod schemas
│   └── db/                  # Drizzle schema + migrations
└── pnpm-workspace.yaml
```

---

## API Endpoints

| Method | Path | Description |
|---|---|---|
| `GET` | `/api/claims` | List all claims |
| `POST` | `/api/claims` | Create a new claim |
| `GET` | `/api/claims/stats` | Dashboard statistics |
| `GET` | `/api/claims/:id` | Get claim by ID |
| `DELETE` | `/api/claims/:id` | Delete a claim |
| `POST` | `/api/claims/:id/adjudicate` | Run GPT-4o adjudication |
| `GET` | `/api/healthz` | Health check |

---

## Getting Started

### Prerequisites

- Node.js 24+
- pnpm 9+
- PostgreSQL database
- OpenAI API key (GPT-4o access required)

### Setup

```bash
# Install dependencies
pnpm install

# Set environment variables
DATABASE_URL=postgresql://user:password@localhost:5432/claims
OPENAI_API_KEY=sk-...
SESSION_SECRET=your-secret-here

# Push database schema
pnpm --filter @workspace/db run push

# Regenerate API hooks (if you change the OpenAPI spec)
pnpm --filter @workspace/api-spec run codegen
```

### Development

```bash
# Start the API server (port 5000 / $PORT)
pnpm --filter @workspace/api-server run dev

# Start the frontend (separate terminal)
pnpm --filter @workspace/claims-ui run dev
```

### Typecheck

```bash
pnpm run typecheck
```

---

## How Adjudication Works

1. User submits a claim with a transcript and one or more image URLs
2. Adjudicator clicks **Run Adjudication** on the claim detail page
3. The API sends all image URLs (as `image_url` content blocks) plus the transcript to `gpt-4o`
4. GPT-4o returns a structured JSON response:
   ```json
   {
     "verdict": "supported",
     "severity": "high",
     "confidence": 87,
     "reasoning": "The images clearly show ..."
   }
   ```
5. The claim record is updated in PostgreSQL and the verdict is displayed in the UI

---

## Database Schema

**`claims` table**

| Column | Type | Description |
|---|---|---|
| `id` | uuid | Primary key |
| `userId` | text | Claimant identifier |
| `claimObject` | enum | `car`, `laptop`, `package` |
| `userClaim` | text | Claim transcript |
| `imageUrls` | text[] | Array of evidence image URLs |
| `userHistory` | text | Optional prior history notes |
| `status` | enum | `pending`, `processing`, `adjudicated`, `error` |
| `verdict` | enum | `supported`, `contradicted`, `inconclusive` |
| `severity` | enum | `low`, `medium`, `high` |
| `confidence` | integer | 0–100 confidence score |
| `reasoning` | text | GPT-4o reasoning text |
| `createdAt` | timestamp | Creation time |
| `updatedAt` | timestamp | Last update time |

---

## License

MIT
