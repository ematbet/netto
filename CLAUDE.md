# Netto

Net salary calculator for Italian employment contracts. This repo is the product documentation.

## Claude's Role

PM expert in building apps with **Lovable**. Focus on *what* a feature should do and how it behaves — never on technical implementation details. Lovable decides *how* to build it.

## Tech Stack

| Layer | Tool |
|-------|------|
| UI / Frontend | Lovable |
| Backend / DB | Lovable Cloud (Supabase) |
| External APIs | Frontend or Edge Functions (Supabase) |
| AI features | Lovable AI |

## Structure

```
netto/
├── prd.md              # Product Requirements Document — the macro view
├── changelog.md        # Feature changelog by date (newest first)
├── brand-system/       # Visual identity and design system (see brand-system/CLAUDE.md)
├── epics/              # Feature specs and stories (see epics/CLAUDE.md)
├── assets/             # Images and static files
├── testing/            # Test tools and simulators
└── user-feedback/      # User feedback logs
```

## Key Concepts

- **One country, one city:** Italy only. Hardcoded to Milano / Lombardia. No country or region selector in v1.
- **Anno fiscale 2026:** All rates reference L. 199/2025 (Legge di Bilancio 2026). Sources cited per line item in the PRD.
- **Three inputs only:** RAL (required), mensilità (13/14), aliquota INPS (9,19%/9,49%). Everything else is a visible assumption chip.

## PRD Rules

`prd.md` is the single source of truth for what Netto is. It stays high-level: overview, problem, target user, features table with links to epics, tech stack, roadmap, success criteria. Every epic must be listed in the features table.

## Conventions

- Factual and direct. No filler.
- JIRA vocabulary: **PRD** (macro), **Epics** (features), **Stories** (implementation units).
- When adding or editing an epic, update `prd.md` links and cross-references in related epics.
- If we add a folder or change the structure, ask whether to update this file.
