# AGENTS Guidelines — App (Monorepo)

Guidelines for full-stack application work when frontend and backend live in **one repository**, or as an index when using the split template files.

**How to use this template:** Start from the structure below, then remove or skip sections that do not apply. Do not invent alternate conventions when a section still applies.

---

## File Index

| File | Use when |
|---|---|
| `AGENTS-app.md` | Monorepo — read this index, then both split files below |
| `AGENTS-frontend.md` | Frontend-only repo, or frontend work in a monorepo |
| `AGENTS-backend.md` | Backend-only repo, or backend work in a monorepo |
| `AGENTS-static.md` | Static HTML/CSS/JS sites (no React, no Flask) |

**Naming convention:** `AGENTS-{scope}.md` — same pattern across all project types.

---

## Repository Layout

Frontend and backend are **separate builds** with separate dev servers. In most setups they are also **separate repositories**. They integrate only through a documented HTTP API.

| Setup | Frontend guidelines | Backend guidelines |
|---|---|---|
| Separate repos (recommended) | Copy `AGENTS-frontend.md` into the frontend repo | Copy `AGENTS-backend.md` into the backend repo |
| Monorepo | Read `AGENTS-frontend.md` | Read `AGENTS-backend.md` |

### Deploying to separate repos

1. Copy `AGENTS-frontend.md` into the frontend repo root.
2. Copy `AGENTS-backend.md` into the backend repo root.
3. Trim any sections that do not apply to that specific project.
4. Keep `API_DOCUMENTATION.md` in the backend repo as the API contract source of truth.

---

## Monorepo: What to Read

When both stacks live in one repo, read **both** split files in full — do not stop at this index. The split files contain the complete shared rules, coding conventions, QA checklists, and anti-patterns for each stack.

1. **[AGENTS-frontend.md](./AGENTS-frontend.md)** — React, SCSS, API consumption, frontend QA
2. **[AGENTS-backend.md](./AGENTS-backend.md)** — Flask, Postgres, API creation, backend QA

All frontend paths in `AGENTS-frontend.md` are relative to the frontend directory. All backend paths in `AGENTS-backend.md` are relative to the backend directory. Adjust if your monorepo uses different top-level folder names.

---

## API Integration Summary

Full details are in each split file. Core rules:

| Role | Responsibility |
|---|---|
| **Backend** | Owns `API_DOCUMENTATION.md` and Postman collection. Implements endpoints first. Documents every change. |
| **Frontend** | Reads `API_DOCUMENTATION.md` before any API work. Consumes documented contracts via `useAPICall` hooks. |

**Workflow for new features:** Backend API + docs first → frontend consumption second → coordinate breaking changes with the user.

---

## Shared Rules (Both Stacks)

These apply to all app agents. The split files (`AGENTS-frontend.md`, `AGENTS-backend.md`) contain the full version of each rule with stack-specific detail — read them, not just this summary.

### Plan First
- Always create a plan and share it with the user **before making any changes**, unless the user explicitly asked for a direct fix.
- Wait for explicit approval before proceeding.
- If scope, design, or approach is ambiguous — ask.

### Dev Server
- Always use the dev server while iterating. **Never run a production build or deployment during an agent session.**
- Before starting a dev server, check whether one is already running on the expected port. If one exists, kill the existing process first, then start fresh.
- When in doubt, restart the dev server rather than debugging a stale state.

### Dependencies
- **Ask the user before installing any new library or package.**
- After any install, update the appropriate lockfile (`package-lock.json` for Node, `Pipfile.lock` for Python).

### Existing Code First
- Search the codebase for existing components, controllers, hooks, utilities, and helpers before creating new ones.
- Follow established patterns — do not invent new conventions when one already exists.

### Security
- Use HTTPS. No mixed content. Never hardcode secrets — use environment config.
- Validate all input server-side. Never trust client-side validation alone.

### Documentation
- Prefer 1–3 core `.md` files. No sprawl. Extend existing docs rather than creating new files.

### Git
- Branch from `main` using `<initials>/<feature-name>` (e.g. `sm/task-filter`).
- Open a PR when working with multiple contributors; solo work may commit directly to `main` when the user requests.
- **Do not commit or push unless the user explicitly asks.**
- Keep branches short-lived — merge or close when the feature is done.

---

## Skills & References

Load the relevant skill file **before starting any task in these areas**.

Skill paths follow the [rapid-build-websites](https://github.com/Dev-Pipeline-145/rapid-build-websites) convention — relative to the repo root. If `skills/public/` is not present in this repo, skip skill loading or ask the user.

| Area | Skill file |
|---|---|
| Word document creation or editing | `skills/public/docx/SKILL.md` |
| PDF creation, editing, or reading | `skills/public/pdf/SKILL.md` |
| PowerPoint / slide decks | `skills/public/pptx/SKILL.md` |
| Spreadsheets / Excel files | `skills/public/xlsx/SKILL.md` |
| Frontend design tokens and UI quality | `skills/public/frontend-design/SKILL.md` |
| Reading uploaded files (routing by type) | `skills/public/file-reading/SKILL.md` |

> Reading the skill file is **required** before writing code or creating files in these areas — not optional.

---

## Quick Reference

Start from template, trim what does not apply · separate FE/BE builds · API docs are source of truth · backend + docs first, frontend second · plan first · ask before installing · check port and kill stale process before starting · branch `<initials>/<feature>` · **do not commit or push unless the user explicitly asks** · monorepo: read both `AGENTS-frontend.md` and `AGENTS-backend.md` · separate repos: copy the matching file · load skill files before document or file tasks.

*When in doubt — ask the user, check existing code, read the API docs, and restart the dev server rather than debugging a stale state.*
