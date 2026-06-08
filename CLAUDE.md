# CLAUDE.md

Guidance for AI assistants working in this repository.

## What this is

**Daily Ops** ("Icon Archives — presence over noise") is a single-page personal
daily-task / routine-tracking app. It is a single static HTML file
(`index.html`) with inline CSS and vanilla JS — no build step, no framework,
no package manager. The UI is in French.

```
.
├── README.md       # one-line project name
└── index.html      # the entire app: markup + <style> + <script>
```

There is no `package.json`, no bundler, no test suite, and no CI. Everything
lives in `index.html`.

## Running / previewing

Just open `index.html` in a browser, or serve the directory statically, e.g.:

```bash
python3 -m http.server 8000
```

Changes take effect on page reload — there's nothing to build or compile.

## Architecture (all inside `index.html`)

### Backend: Supabase
The app is a thin client over Supabase (Postgres + REST). The client is
created inline via the CDN bundle (`@supabase/supabase-js@2`) using a
hardcoded `SUPABASE_URL` and a **publishable** anon key (safe to expose
client-side; do not replace it with a service-role/secret key).

Tables used (all queried/mutated directly from the browser):
- `sous_objectifs` — today's tasks / sub-goals. Key columns: `titre`,
  `statut` (`a_faire` | `fait`), `date_prevue`, `recurrent`, `source`,
  `completed_at`, `objectif_id`.
- `objectifs_finaux` — long-term goals shown read-only (`statut`, `deadline`,
  `description`).
- `drafts_a_valider` — drafts (messages/emails/replies) awaiting human
  approval, with an inline accept/edit/reject loop (`statut`:
  `en_attente` | `valide` | `modifie` | `rejete`).
- `journal_quotidien` — per-day journal: `date`, `streak`, and a `data` JSON
  blob (`completedAt100`, `dayStartedAt`) used to sync streaks and the
  "day anchor" timestamp across devices.

### Key behaviors implemented in the `<script>` IIFE
- **Optimistic, write-through UI**: local arrays (`tasks`, `objectifs`,
  `drafts`) are mutated and re-rendered immediately, then synced to Supabase.
- **Carry-over (`carryOverOverdueTasks`)**: on load, unfinished tasks from
  past days get their `date_prevue` shifted to tomorrow (max `CAP_PER_DAY = 3`)
  or the day after, to avoid pile-ups.
- **Recurring tasks**: a recurring task is stored both as a dated row (today)
  and as a standing template (`date_prevue: null`, `recurrent: true`); the
  template gets re-seeded into a new dated row each day on load if today has
  no tasks yet.
- **Streaks (`checkStreak`)**: increments `streak` the moment today's
  completion hits 100%, decrements if it drops back below — persisted to
  `journal_quotidien`.
- **"Start my day" planning**: pressing the button stamps `openedAt`
  (`dayStartedAt` in Supabase) and generates a fixed timeline (`planBlocks`,
  offsets in minutes) anchored to that moment; synced so other devices show
  the same plan.
- **Verification modal**: checking off a task opens a short "honest check"
  modal (simulated scan delay) before marking it done — a soft friction step
  to discourage false completions; confirming triggers a confetti burst.
- **Drafts review loop**: `drafts_a_valider` rows render as editable cards
  with reject / approve actions; editing before approving sends `statut:
  'modifie'` instead of `'valide'`.

### Styling
All CSS is inline in `<style>`, using CSS custom properties defined on `:root`
(`--bg`, `--panel`, `--p1/--p2/--p3` gold/cream/silver accents, etc.) for a
dark "gold on near-black" theme. There's no CSS framework — keep new styles
consistent with the existing variable palette and the rounded-card / soft-glow
aesthetic.

## Conventions for changes

- **Keep it a single file.** This project intentionally has no build tooling;
  don't introduce bundlers, frameworks, or split the app into modules unless
  explicitly asked.
- **Match the existing style**: vanilla JS with small DOM-building helper
  functions (no templating libraries), French UI strings/comments, optimistic
  local state mirrored to Supabase.
- **Supabase schema changes**: this repo has no migrations folder — schema is
  managed externally in the Supabase project. If you need new columns/tables,
  call that out explicitly rather than assuming they exist.
- **Don't commit secrets beyond the existing publishable key.** The embedded
  `SUPABASE_KEY` is a public/publishable anon key by design; never swap it for
  a service-role key or add other credentials to client-side code.
- **No tests/linters exist.** Verify changes by opening `index.html` in a
  browser and exercising the flow manually (add/check/delete tasks, start-day
  button, draft approval, etc.).

## Git

Standard git workflow; commit messages in this repo are short, imperative
English summaries (e.g. "Add start button for daily planning and sync time",
"Integrate Supabase for task management").
