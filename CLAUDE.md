## Project Configuration

- **Language**: TypeScript
- **Package Manager**: pnpm
- **Add-ons**: prettier, eslint, tailwindcss

---

# CLAUDE.md

Single-user web app for **recording and managing car-rental data** with financial reporting (admin dashboard, record-keeping — _not_ a reservation system).

> **Status: greenfield.** No code is scaffolded yet — the only artifact is the PRD. The PRD is the authority for all design decisions until the codebase exists.

## Source of truth

[`docs/PRD-Aplikasi-Manajemen-Sewa-Mobil.md`](docs/PRD-Aplikasi-Manajemen-Sewa-Mobil.md) is the single source of truth (v1.0). **Consult it before any non-trivial work.** Its enum values and business rules are _binding_ — use them exactly as written.

## Conventions

- **Language:** prose and UI labels in **Bahasa Indonesia**; all technical identifiers (tables, columns, enums, types) in **English**.
- **Money is always INTEGER rupiah** — never use float (BR-11).

## Tech stack (planned — PRD §4)

- **Framework:** SvelteKit (TypeScript), full-stack — `load` + form actions for CRUD.
- **DB / ORM:** Turso (libSQL/SQLite) + Drizzle ORM (code-first, drizzle-kit migrations).
- **Auth:** Better Auth — email+password, single admin, **no public registration** (admin seeded).
- **UI:** Tailwind CSS + shadcn-svelte; charts via LayerChart.
- **Forms/validation:** sveltekit-superforms + Zod (one schema shared client & server).
- **Export:** SheetJS (xlsx) + pdfmake (no headless browser — serverless-friendly).
- Keep SvelteKit standard / platform-agnostic so hosting stays portable.

## Commands

None yet — project is not scaffolded. **Once scaffolded** (standard SvelteKit), expect:

```bash
npm install
npm run dev        # dev server
npm run build      # production build
npx drizzle-kit generate / migrate   # DB schema migrations
```

Update this section with the real commands as soon as `package.json` exists.

## Architecture principles (PRD §4)

- Business logic (transaction-code generation, price snapshots, derived status) lives **server-side** (server `load`/actions), not in client components.
- Validate on **both sides** with the same Zod schema — client for UX, **server is the authority**.

## Domain model (PRD §6 for full columns)

`car` → has many `rental` → has many `payment`; plus standalone `expense`. Auth tables (`user`, `session`, …) are managed by Better Auth, separate from domain tables.

## Critical guardrails

**Out of scope — never add without explicit instruction (PRD §2.2):**
booking / double-booking detection, multi-user or roles, late fees, notifications, deposit tracking (guarantee is only an optional URL), driver-vs-self distinction, linking expenses to cars/rentals, any date-based calculation (pickup/return dates are display-only).

**Binding business rules:**

- **Snapshots** locked at rental creation: `price_per_day`, `price_tier`, `total_cost`, `car_code_snapshot`, `car_name_snapshot`. Editing a car never changes existing rentals (BR-4).
- `rental.code` and `rental.car_id` are **immutable** after creation (BR-2, BR-3).
- **Transaction code** = `{CAR_CODE}-{NNN}` from per-car counter `car.transaction_seq`, incremented **atomically** in the creation transaction, zero-padded ≥3 digits, **never reused** (BR-2).
- `payment_status` and `outstanding` are **derived, never stored** — computed from non-deleted `payment.amount` vs `total_cost` (BR-7, §9).
- **Soft delete** via `deleted_at` on all domain tables; default queries exclude deleted rows (BR-9).
- `total_cost = price_per_day × rental_days`; `price_per_day` defaults from the car's tier but is **override-able** (BR-5, BR-6).

**Enums are binding** — use exact values from PRD §6.6: `price_tier`, `payment_scheme`, `payment_type`, `rental_status` (and derived `payment_status`).

## Workflow & IMPORTANTS

**Communication**

- **Always ask, instruct, and write plans in Bahasa Indonesia** (even when prompted in English). UI = Indonesia; code/comments = English.
- **If the prompt is ambiguous, ask before writing any code.**

**Planning & tasks**

- Enter plan mode for any task with 3+ steps or architectural decisions; include your recommended choice. Write the plan to `tasks/todo.md` as checkable items, check in before implementing, mark items done as you go, summarize at the end. If something goes sideways mid-task: STOP, re-plan, continue.

**Quality bar**

- Never mark a task complete without proving it works (dev server, the relevant page/endpoint, DB state in Prisma Studio). Ask "Would a senior dev approve this PR?"
- For non-trivial changes, pause: "Is there a simpler way?" — implement the clean solution, don't over-engineer simple changes.
- Bug reports: just fix it (point at the log/error, resolve). Prioritize thorough checks for performance and security.

**Self-improvement**

- After any correction from me: add the pattern to `tasks/lessons.md` with a rule preventing recurrence; review it next relevant session.

**Hard rules**

- **Never push to my GitHub repo.**
- **Use caveman-commit skill to always provide a commit message of latest changes.**
- **Never implement anything outside the current task scope.**
- **Use Context7 mcp** to fetch up-to-date library docs before implementing (Better Auth, Prisma, TanStack Query, shadcn/ui, Resend, etc.).
- **After a schema change, remind me to run generate && db push.**
- **New env vars** go in both `.env.local` (placeholder) and `.env.example`.
- **If an implementation differs from the PRD or makes a new decision, update the PRD** to reflect it.
- **Maintain the Session History below** as lean durable memory (see its maintenance rules). Read it before executing commands.
- Currency IDR (`Intl.NumberFormat('id-ID', { style:'currency', currency:'IDR' })`), date DD/MM/YYYY, timezone WIB (UTC+7) hardcoded. Landing page + app default to **light mode**.
- **`init` after every task to save memory.** After completing any task, provide: (1) a short summary paragraph of what was built + suggested next steps, and (2) follow-up run/check/verify instructions specific to what was built.
- Use the appropriate skills needed to solve a problem
