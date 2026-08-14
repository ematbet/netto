Sync the Netto app codebase with this spec repo (PRD, epics, data model).

## Source repos

- **Code (Lovable app):** `/Users/matteobetti/Projects/netto-salary-calculator-web`
- **Spec (this repo):** `/Users/matteobetti/Projects/netto`

## Steps

1. **Pull latest code** from the Lovable repo (run `git pull` inside it).

1. **Analyze what changed** since the last sync by reading the git log of the Lovable repo. Filter out noise commits ("Changes", "Save plan in Lovable", "Updated plan file", "wip", "fix linter"). Group meaningful commits by date and feature area.

1. **Inspect the codebase** for built features:
   - `src/pages/`, `src/routes/`, or wherever routing is wired — which screens exist?
   - `src/components/` — feature-specific components (Fuel*, Dashboard*, Auth*, Vehicle*, etc.).
   - `supabase/migrations/` — actual DB schema in place.
   - `package.json` — any significant new dependencies (auth provider, mapbox, charting lib, etc.).
   - `.lovable/` — Lovable project metadata, sometimes has a feature inventory.

1. **Read the current spec state**: `prd.md`, `data-model/data-model-v1.md`, every `epic.md` under `epics/`, and `CLAUDE.md`.

1. **Compare code vs spec** — identify:
   - Features built in code but not yet reflected in any epic or in the PRD.
   - Epics still marked `**Status:** v1 — Draft` whose feature is now actually built.
   - Data model drift: do the Supabase migrations match `data-model/data-model-v1.md`? Note any tables/columns added, renamed, or missing.
   - Routing or auth flow drift from the Onboarding epic's *Routing* table.
   - Missing changelog entries.

1. **Update the changelog**: create `changelog.md` at the spec repo root if it doesn't exist. Prepend new entries at the top, organized by date (`YYYY-MM-DD`). Only add entries for changes not already there. Format: date header, feature area subheader (Dashboard / Onboarding / Fuel / Settings / Landing / Data model / Infra), terse bullet points — what changed and where (commit hash optional).

1. **Update epic statuses** — if a feature is now demonstrably built in code, change its `**Status:**` from `v1 — Draft` to `v1 — Built`. Be conservative: update only when there's clear evidence in `src/` and (where relevant) `supabase/migrations/`. Use `Live` only when the user explicitly says it has shipped.

1. **Update epic content sparingly** — if Lovable shipped a feature in a way that diverges from the epic spec, **do NOT silently rewrite the epic to match the code.** Flag the divergence in the final summary and let the user decide whether the epic or the code should change.

1. **Update `prd.md`** — if a feature moved from Draft to Built, reflect that in the v1.0 / Later tables. Update the *Last Updated* line. Keep the v1.0 vs Later split intact; nothing graduates between sections without an explicit user decision.

1. **Update `data-model/data-model-v1.md`** — if the Supabase migrations introduced new tables/columns or renamed existing ones, reflect those changes in the doc. If they *removed* something the doc still describes, flag it in the summary (do not auto-delete; it's safer to surface the conflict).

1. **Update `CLAUDE.md`** — only if the spec repo's structure changed (new top-level folders, renamed files).

1. **Show a summary** to the user containing:
   - Code commits inspected (date range, count).
   - Each file updated in the spec repo and why.
   - Any spec-vs-code divergences flagged for manual review.
   - Anything the data model says that the migrations contradict.

## Rules

- Do NOT create new epic folders unless an entirely new feature category was built that has no existing epic.
- Do NOT silently rewrite epic content to match what Lovable actually shipped — flag divergences instead.
- Do NOT touch `brand-system/` content.
- Do NOT auto-delete entries from `data-model-v1.md` — surface contradictions instead.
- Keep the same brevity and conventions already used in the docs: factual, direct, no filler, **no layout / no positions / no dimensions inside epics or stories** (Lovable owns those).
- If nothing meaningful changed since the last sync, say so and stop.
