# Design Arena

Design Arena is a full-stack competitive design platform where designers enter themed battles, submit work, receive private AI jury feedback, vote on other work, build public profiles, climb the leaderboard, and collaborate in team rooms.

## What ships in this build

- Premium dark editorial/tournament UI with responsive desktop and mobile layouts.
- Public landing page, battle discovery, battle detail/gallery, submission flow, leaderboard, designer profiles, and collaboration rooms.
- Manus OAuth session auth through the WebDev runtime, with protected submission, vote, comment, team-create, and team-join procedures.
- Drizzle/MySQL schema for users, profiles, battles, submissions, votes, comments, badges, user badges, teams, and team members.
- Demo seed data that is inserted on first public battle/leaderboard read so the environment is populated without a separate seed command.
- Secure server-side AI review through the preconfigured `invokeLLM` helper. The browser never receives an AI credential.
- AI duplicate/spam moderation on every submission, with a stored safety verdict and review trail.
- AI battle agent that monitors a battle on demand, detects likely duplicate pairs, summarizes feedback per submission, recommends next actions, and persists a final report for the public gallery.
- GitHub project submissions with public repository verification, latest default-branch commit capture, language/license/stars/forks metadata, and a verified build card in the battle review panel.
- NIP-07 Nostr-native login/linking with signed challenge verification, portable reputation events, and optional signed battle-result publishing.
- Admin-controlled signed Nostr reputation events for portable builder achievements.
- Lightning mainnet reward ledger with invoice submission and an admin-authorized LNbits-compatible payout adapter; no payout happens automatically without an explicit operator action.
- Admin automation console for queued submission reviews, AI battle-agent runs, generated result summaries, and signed Nostr result publishing.
- Server-side image upload validation (PNG/JPG/WebP, max 5 MB) with WebDev storage and `/manus-storage/...` URLs.
- Database-level one-vote-per-submission enforcement with a unique index, plus front-end disabled state.
- Near-realtime vote/comment/gallery refresh through short polling. The current WebDev runtime does not expose Supabase Realtime channels, so this is intentionally implemented as a safe polling fallback.
- Vitest coverage for auth logout and protected Design Arena server contracts.

## Architecture

```text
React + Vite + Tailwind 4
        |
        | tRPC hooks (typed client/server contract)
        v
Express + tRPC server
        |
        +-- Drizzle ORM -> managed MySQL/TiDB database
        +-- Manus OAuth -> session cookie auth
        +-- Manus storage -> server-side image upload / signed delivery
        +-- Built-in LLM -> server-side AI jury review
        +-- AI battle agent -> duplicate findings, feedback summaries, final reports
        +-- GitHub API -> verified public builder project metadata
        +-- Nostr NIP-07 -> signed identity challenges and relay publishing
        +-- Lightning API -> guarded mainnet invoice payment
```

The product brief asked for Supabase PostgreSQL/Auth/Storage/Realtime. This session's managed WebDev scaffold provides the same application boundaries through its built-in managed MySQL/TiDB, Manus OAuth, managed storage, and server-side LLM integration. The UI and server are structured so a Supabase adapter can be substituted later, but this deployed build does not claim to be connected to a Supabase project.

## Environment variables

In the managed WebDev environment, these are injected automatically. For a non-managed deployment, create equivalent server-side environment entries from this table rather than committing secret values:

| Variable | Purpose | Secret? |
| --- | --- | --- |
| `DATABASE_URL` | Managed MySQL/TiDB connection | Yes |
| `JWT_SECRET` | Session cookie signing | Yes |
| `VITE_APP_ID` | Manus OAuth application ID | No |
| `OAUTH_SERVER_URL` | Manus OAuth backend | No |
| `VITE_OAUTH_PORTAL_URL` | Frontend login portal | No |
| `BUILT_IN_FORGE_API_URL` | Server-side storage/LLM gateway | No |
| `BUILT_IN_FORGE_API_KEY` | Server-side storage/LLM gateway token | Yes |
| `OWNER_OPEN_ID` | Owner identity used for admin promotion | No |
| `OWNER_NAME` | Owner display name | No |
| `NOSTR_AUTOMATION_NSEC` | Dedicated operator key for signed result events, nsec1 format | Yes |
| `NOSTR_RELAY_URLS` | Comma-separated Nostr relay URLs; defaults to two public relays | No |
| `LIGHTNING_API_URL` | LNbits-compatible mainnet API base URL | No |
| `LIGHTNING_API_KEY` | Mainnet payout wallet API key | Yes |

Do not put AI or storage credentials in `VITE_*` variables or browser code.

## Database setup / migration

The schema source lives in `drizzle/schema.ts`. The generated migrations are `drizzle/0001_remarkable_roulette.sql`, `drizzle/0002_tidy_thundra.sql`, and `drizzle/0003_dry_rocket_raccoon.sql`; all have been applied to the managed project database.

For a fresh local database:

```bash
pnpm drizzle-kit generate
pnpm drizzle-kit migrate
```

The app also seeds clearly synthetic demo content on the first public read. Demo records use `demo-*` open IDs and `@demo.designarena` emails.

## Run locally

```bash
pnpm install
pnpm dev
```

Other useful commands:

```bash
pnpm check       # TypeScript
pnpm test        # Vitest
pnpm build       # Vite client + bundled Express server
```

## Routes

- `/` — landing page
- `/battles` — active/upcoming/completed battle discovery
- `/battles/:id` — brief, gallery, AI scores, moderation evidence, GitHub build card, agent report, vote, comments, rules
- `/battles/:id/collaborate` — team rooms, shared submissions, discussion
- `/submit` — protected image submission, optional GitHub verification, and AI review flow
- `/leaderboard` — global rankings and badge cabinet
- `/profile/:username` — public portfolio profile
- `/profile/me` — authenticated current profile shell
- `/ops/automation` — guarded admin console for agent runs, result summaries, and Nostr publishing

## Deploy

### Vercel

This repository now includes a Vercel adapter: `server.ts` exports the Express application, `vercel.json` selects the Vercel build command, and `build:vercel` copies Vite output into the root `public/` directory for Vercel CDN serving.

Import the repository into Vercel with these settings:

| Setting | Value |
| --- | --- |
| Framework preset | Other |
| Install command | `pnpm install --frozen-lockfile` |
| Build command | `pnpm build:vercel` |
| Output directory | Leave blank |
| Root directory | `.` |

Vercel must receive the server-side environment variables listed above, including `DATABASE_URL`, `JWT_SECRET`, Manus OAuth/Forge values, and—if enabling live features—`NOSTR_AUTOMATION_NSEC`, `NOSTR_RELAY_URLS`, `LIGHTNING_API_URL`, and `LIGHTNING_API_KEY`.

Vercel runs the Express app as a serverless function. Do not depend on long-lived workers, local filesystem persistence, or in-process cron jobs.

### WebDev

Use the WebDev project publish flow after saving a checkpoint. The production build is already compatible with the managed Node runtime:

```bash
pnpm build
```

## Remaining limitations

- Authentication uses Manus OAuth supplied by this WebDev scaffold rather than Supabase email/password auth.
- Realtime uses polling for votes/comments/gallery refresh; native Supabase Realtime channels are not available in this runtime.
- Badge display is seeded as a product surface; automatic badge-award rules and rank recalculation are the next backend extension.
- The current profile editor and direct deletion/editing of submissions are not exposed in the UI yet.
- AI jury review is text/context-based and returns graceful fallback scores if the model gateway is unavailable; image vision can be added through the same server-side helper when a production image-review model is selected.
- Live Nostr publishing remains disabled until a dedicated `NOSTR_AUTOMATION_NSEC` is supplied as a managed secret.
- Live Lightning mainnet payouts remain disabled until an LNbits-compatible `LIGHTNING_API_URL` and `LIGHTNING_API_KEY` are supplied as managed secrets. The ledger and invoice workflow are available before configuration.
- GitHub verification currently targets public repositories through the public GitHub API; private repositories need an authenticated GitHub adapter before they can be supported.
- AI battle monitoring is an explicit admin action from the operator console. A recurring heartbeat can be added later without changing the persisted agent-run contract.
