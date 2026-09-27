# Design Arena builder-arena upgrade

- [x] Inspect and preserve existing battles, submissions, moderation, Nostr, Lightning, and automation flows.
- [x] Add persisted AI-agent runs with duplicate-pair findings, per-submission feedback, next actions, and final reports.
- [x] Add public AI-agent report rendering to battle detail and admin run/history controls.
- [x] Add GitHub public-repository verification, commit capture, project metadata, and submission/review UI.
- [x] Strengthen portable Nostr reputation with admin-signed reputation event publishing.
- [x] Keep Lightning rewards guarded behind invoice ownership, admin payout action, and managed secrets.
- [x] Apply additive database migration `drizzle/0003_dry_rocket_raccoon.sql`.
- [x] Add Vercel `server.ts` Express entrypoint, Vite-independent static serving, `vercel.json`, and `build:vercel`.
- [x] Run TypeScript checks, 11 Vitest tests, production builds, GitHub/API/AI smoke tests, and Vercel HTTP smoke testing.
- [x] Enable the Vercel connector and inspect the authenticated Vercel account.

## Vercel handoff

The local source is Vercel-ready. The connected Vercel account currently has no existing `design-arena` project and no linked Git repository exposed through the connector, so deployment still needs either a GitHub repository link or a direct local-source upload through the Vercel CLI/dashboard.

## Safety notes

- Nostr keys stay in the user's signer; the app never stores a user's private key.
- GitHub verification is public-repository-only and captures metadata without cloning or executing repository code.
- Mainnet rewards remain pending until a user supplies a valid Lightning invoice and an authorized payout action is performed.
- Automated publishing uses a server-side Nostr publisher key only when configured as a managed secret.
- AI battle monitoring is an explicit admin action; the persisted agent-run contract can support a future scheduler without changing the UI contract.
