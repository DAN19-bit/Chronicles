# Chronicles — Proposed Architecture

**Status:** Proposal only; not an approved implementation decision.

## Candidate stack
- Next.js App Router with TypeScript for the web application.
- Supabase PostgreSQL for relational data.
- Supabase Auth for authentication.
- Supabase Storage for user-submitted media, with private-by-default upload handling where appropriate.
- GitHub Actions for continuous integration.

## Architectural principles
- Public pages should remain useful without exposing privileged credentials.
- Enforce authorization at the database/storage layer, not only in the UI.
- Never expose a Supabase service-role key in client-side code.
- Use Row Level Security policies and test them for each user role.
- Validate uploads, restrict file types and sizes, and use safe media delivery.
- Keep secrets in deployment environment settings, never in Git.
- Separate development, preview, and production environments.

## Candidate domain areas
- Accounts and profiles
- Testimonies and media
- Categories and tags
- Reactions and comments
- Witness accounts
- Reports and moderation actions
- Consent and visibility settings

## Decisions still required
- Final stack and hosting choice.
- Data model and retention/deletion policy.
- Anonymous publishing and visibility rules.
- Moderation workflow and administrator roles.
- Backup, recovery, and operational monitoring.

No database migration, production configuration, or deployment should be performed solely on the basis of this proposal.
