# System Architecture — THORNFIELD-MAINT-001

| Layer | Choice | Why |
|---|---|---|
| Frontend | Next.js + Ascendra UI | Common framework default (`FW-025`) — client stated no preference |
| Backend | NestJS | Common framework default — matches `ascendra-commons`'s scope |
| Database | PostgreSQL (Supabase) | Common framework default |
| ORM | Drizzle | Common framework default — matches `ascendra-commons`'s scope |
| State Management / Data Fetching | React Query | Standard pairing with the above |
| Cache | None this phase | No signal for one — small scale, no hot-path read pattern identified |
| Logging | Pino → Axiom | Common framework default (`FW-041`) |
| Error Tracking | Sentry | Common framework default (`FW-041`) |
| Containerisation | Docker | Standard |
| CI/CD | GitHub Actions | Standard |
| Hosting | Vercel (web) + Render (API) | Common, low-ops-overhead pairing for a project this size |

**Deviations from the common default:** none — every choice above is the framework's stated default, not a project-specific override.

**Actor Identity Shape:** **Multiple actor identity spaces** — confirmed at Step 4. Three distinct, non-overlapping actor kinds interact with this system: authenticated Staff (Property Manager, Operations Lead — one identity space, one table), unauthenticated Tenants (token-link only — a second, distinct identity space, never the same table as Staff), and unauthenticated Contractors (per-job token link — a third, distinct identity space). No foreign key can point at three different tables; every Master table's `created_by`/`updated_by`/`deleted_by` uses the `varchar` actor-label shape (`FW-037`), never a FK.
