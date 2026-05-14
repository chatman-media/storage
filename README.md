# @chatman-media/storage

PostgreSQL storage adapters for [@chatman-media/sales](https://github.com/chatman-media/sales) — Drizzle ORM implementations of all engine repository interfaces.

## Installation

```bash
npm install @chatman-media/storage
```

**Peer dependencies** (install alongside):

```bash
npm install drizzle-orm postgres @chatman-media/sales
```

## Quick Start

```ts
import postgres from "postgres";
import { drizzle } from "drizzle-orm/postgres-js";
import { createPgRepos, schema } from "@chatman-media/storage";

const client = postgres(process.env.DATABASE_URL!);
const db = drizzle(client, { schema });

const repos = createPgRepos(db);

// upsert a Telegram user
const { id: userId } = await repos.users.upsert({ telegramId: 123456789, username: "alice" });

// start a conversation
const { id: convId } = await repos.conversations.create({ userId, styleSlug: "consultative" });

// create a lead
const { id: leadId } = await repos.leads.create({ conversationId: convId });
```

## Repositories

| Class | Interface | Key methods |
|-------|-----------|-------------|
| `PgUsersRepo` | `IUsersRepo` | `upsert({ telegramId, username? }) → { id }` |
| `PgConversationsRepo` | `IConversationsRepo` | `create({ userId, styleSlug }) → { id }` |
| `PgLeadsRepo` | `ILeadsRepo` | `create({ conversationId }) → { id }` |
| `PgSkillsRepo` | `ISkillsRepo` | `skillsForStyle(styleId) → SkillRow[]` |
| `PgSkillOutcomesRepo` | `ISkillOutcomesRepo` | `record(opts)`, `aggregates(slugs) → SkillAggregate[]` |
| `PgStyleRatingsRepo` | `IStyleRatingsRepo` | `getRating(styleId) → number`, `setRating(styleId, rating)` |
| `PgSelfPlayMatchesRepo` | `ISelfPlayMatchesRepo` | `insert(match) → id`, `byId(id)`, `list(opts) → SelfPlayMatchSummary[]` |
| `PgPairwiseMatchesRepo` | `IPairwiseMatchesRepo` | `insert(opts) → id` |
| `PgShadowEvaluationsRepo` | `IShadowEvaluationsRepo` | `update(evalId, patch)` |

### `createPgRepos(db)`

Convenience factory that constructs all repos from a single Drizzle instance and returns them as a named bundle:

```ts
const { skills, outcomes, ratings, matches, pairwise, shadowEvals, users, conversations, leads } =
  createPgRepos(db);
```

## Database Schema

| Table | Description |
|-------|-------------|
| `users` | Telegram user accounts (`telegram_id` unique) |
| `conversations` | Conversations linked to a user and a style slug |
| `leads` | Sales leads within a conversation |
| `skill_outcomes` | Per-lead skill performance records (`won` / `lost` / `draw`) |
| `style_ratings` | ELO ratings per style (default 1500), upserted on change |
| `skills` | Skill definitions — slug, family, prompt fragment, applicable stages (JSONB) |
| `self_play_matches` | Single-style match records with full transcript (JSONB) |
| `pairwise_matches` | Head-to-head comparisons between two self-play matches |
| `shadow_evaluations` | Evaluation run state — status, decision, pair counts, error |

## Environment Variables

| Variable | Description |
|----------|-------------|
| `DATABASE_URL` | PostgreSQL connection string — used by drizzle-kit CLI commands |

## Commands

| Command | Description |
|---------|-------------|
| `npm run build` | Bundle `src/index.ts` → `dist/` with type declarations |
| `npm run typecheck` | Run TypeScript type check without emitting |
| `npm run check` | Lint source with Biome |
| `npm run format` | Auto-format source with Biome |
| `npm run db:generate` | Generate Drizzle migration files from schema changes |
| `npm run db:migrate` | Apply pending migrations to the database |
| `npm test` | Run tests with bun |

## Migrations

```bash
# After changing src/schema.ts, generate a new migration:
npm run db:generate

# Apply all pending migrations:
DATABASE_URL=postgres://... npm run db:migrate
```

Migration files are written to `./migrations/` and tracked in version control.

## License

MIT
