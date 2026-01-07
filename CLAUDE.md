# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project defaults

This repository follows my standard, opinionated architecture and stack choices. If something is not explicitly specified, assume these defaults and do not deviate unless instructed.

### Core principles
- Always use Supabase for:
  - Primary PostgreSQL database
  - Indexing and search
  - Object storage via Supabase Buckets (S3-compatible)
- Always use Upstash Redis for:
  - Caching
  - Rate limiting
  - Lightweight queues or ephemeral state
- Prefer REST APIs whenever possible
- Use WebSockets only when REST is insufficient (real-time, push, streaming)
- Always use Docker for development and production
- Backend framework is always NestJS
- Frontend framework depends on the product type:
  - Reactive web app: Vite
  - Static or content-first site: Astro
  - Mobile app: React Native
- AI and ML integrations:
  - LLM access via OpenRouter
  - Image generation via FAL
- Language: TypeScript with strict settings everywhere

Do not introduce alternative databases, cache providers, AI gateways, or frameworks unless explicitly requested.

## Tech stack summary

### Backend
- Framework: NestJS
- API style:
  - REST by default
  - WebSockets only for real-time features
- Database: Supabase PostgreSQL
- Storage: Supabase Buckets
- Cache: Upstash Redis
- Auth: Supabase Auth (unless explicitly overridden)
- Runtime: Node.js (current LTS)
- Language: TypeScript (strict)
- Containerization: Docker

### Frontend (Web)
- Package manager: `pnpm`
- Reactive applications:
  - Vite
  - React
- Static or content-focused sites:
  - Astro
- Styling:
  - TailwindCSS
  - Headless or primitive-based UI libraries preferred
- Communication:
  - REST for standard operations
  - WebSockets only where explicitly required

### Mobile
- Framework: React Native
- State management and data fetching should align with backend REST APIs
- No direct database access from mobile clients
- All secrets handled via backend or secure platform mechanisms

### AI and ML
- LLM routing and inference:
  - OpenRouter only
- Image generation:
  - FAL only
- AI logic must live behind backend APIs
- Do not call AI providers directly from frontend or mobile clients
- Always add:
  - Rate limiting
  - Input validation
  - Cost-awareness and logging

### Infrastructure
- Docker-first development
- Local environment should mirror production as closely as possible
- Configuration via environment variables only
- `.env.example` must exist and be up to date

## Commands

### Setup
- `pnpm install` - Install dependencies

### Development
- `pnpm dev` - Start local development environment
- Backend and frontend should be runnable independently if split

### Build
- `pnpm build` - Production build
- Docker images must be built via `Dockerfile`

### Testing
- `pnpm test` - Run tests
- Prefer:
  - Unit tests for services
  - Integration tests for API boundaries
  - Avoid UI snapshot tests unless justified

## Repository layout (recommended)

Backend (NestJS):
- `src/`
  - `modules/` - Feature modules
  - `controllers/` - REST controllers
  - `services/` - Business logic and integrations
  - `gateways/` - WebSocket gateways (only when required)
  - `lib/` - Shared utilities
  - `config/` - App and environment configuration
- `test/` - Backend tests

Frontend (Vite or Astro):
- `src/`
  - `pages/`
  - `components/`
  - `features/`
  - `lib/`
- `public/`

Mobile (React Native):
- `src/`
  - `screens/`
  - `components/`
  - `features/`
  - `lib/`

Infrastructure:
- `Dockerfile`
- `docker-compose.yml` (if needed)
- `.env.example`

## API and communication rules

### REST
- REST is the default and preferred API style
- Use resource-oriented endpoints
- Use correct HTTP status codes
- Avoid RPC-style endpoints unless explicitly justified
- Controllers must remain thin
- Business logic belongs in services

### WebSockets
- Use only for:
  - Real-time updates
  - Presence
  - Streaming state
- Implement via NestJS gateways
- Do not duplicate REST logic inside WebSocket handlers

## Database and data access

- Supabase is the single source of truth
- Database access only via backend services
- Never access Supabase directly from frontend or mobile apps
- Prefer explicit schemas and typed query results
- Handle migrations in a Supabase-compatible way

## Caching (Upstash Redis)

- Use Redis for:
  - Hot data caching
  - Rate limiting
  - Session-like ephemeral data
- Do not store primary or authoritative data in Redis
- Cache invalidation must be explicit and intentional

## Docker expectations

- Every project must include a Dockerfile
- No reliance on global system dependencies
- Multi-stage builds preferred
- Production images should be minimal and secure
- Dev and prod stages should be clearly separated if applicable

## TypeScript rules

- Strict mode enabled
- No `any`
- Explicit types at module and API boundaries
- Prefer discriminated unions for:
  - API responses
  - State machines
- Validate all external input

## Security and reliability defaults

- All external input validated
- Rate limiting on public endpoints
- Secrets never committed
- AI endpoints must be guarded against abuse
- Logging must avoid leaking sensitive data

## How Claude Code should work in this repo

When making changes:
1. Follow the stack rules strictly.
2. Do not introduce alternative tools or providers.
3. Keep changes scoped and incremental.
4. Match existing patterns before creating new ones.
5. Update types, tests, and Docker config together when required.

Before finishing:
- TypeScript must pass
- Tests must pass
- Docker build must succeed
- API boundaries must remain clean

## Common pitfalls to avoid

- Introducing non-Supabase databases or storage
- Using GraphQL unless explicitly requested
- Overusing WebSockets where REST is sufficient
- Calling AI providers directly from clients
- Bypassing NestJS service boundaries
- Environment-specific logic outside Docker
- Weak or implicit typing at API boundaries
