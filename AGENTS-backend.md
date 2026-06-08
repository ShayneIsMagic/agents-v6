# AGENTS Guidelines — Backend (Flask / Postgres)

Guidelines for the Flask backend repository. The frontend is a **separate repo** — integration happens only through a documented HTTP API.

**How to use this template:** Start from the structure below, then remove or skip sections that do not apply to the current project. Do not invent alternate conventions when a section still applies.

All paths below are relative to the **backend repo root**.

Read **Shared Rules** first, then **API Integration**, then the coding conventions. Load any relevant Skill files before starting work.

---

## Shared Rules

These apply to all backend agents.

### Plan First
- Always create a plan and share it with the user **before making any changes**, unless the user explicitly asked for a direct fix (e.g. "fix this typo", "correct this status code").
- Wait for explicit approval before proceeding.
- If scope, design, or approach is ambiguous — ask. Redundant confirmation beats silent assumptions and rework.

### Dev Server
- Always use the development server while iterating. **Never run a production deployment during an agent session.**
- Before starting a dev server, check whether one is already running on the expected port. If one exists, kill the existing process first, then start fresh.
- When in doubt, restart the dev server rather than debugging a stale state.

### Dependencies
- **Ask the user before installing any new library or package.**
- After any install, ensure `Pipfile.lock` is updated.

### Existing Code First
- Check for existing controllers, routes, models, and utilities before creating new ones.
- Use established patterns from the codebase — don't invent new conventions when one already exists.
- When creating new controllers or routes, review Task and Project implementations as reference before starting.

### Security
- Use HTTPS for all links and resources. No mixed content.
- Sanitize and validate all user input server-side. Never trust client-side validation alone.
- Never hardcode secrets, API keys, or connection strings. Use environment config via `config/config.py`.
- New endpoints must respect the role/permission system — test with unauthorized users.

### Documentation
- Do not create unnecessary documentation.
- Prefer 1–3 core `.md` files (README, `AGENTS-backend.md`, `API_DOCUMENTATION.md`). No sprawl.
- Add to existing docs when new guidance is needed rather than creating new files.
- Do not duplicate rules across README, `AGENTS-backend.md`, and other files — one source of truth.

### Git
- Branch from `main` using the convention `<initials>/<feature-name>` (e.g. `sm/task-filter`).
- Open a PR for review when working with multiple contributors; solo work may commit directly to `main` when the user requests.
- **Do not commit or push unless the user explicitly asks.**
- Keep branches short-lived — merge or close when the feature is done.

### General Anti-Patterns

| Avoid | Do instead |
|---|---|
| Installing packages silently | Ask the user first |
| Creating new files without checking | Search existing codebase first |
| Hardcoded secrets or connection strings | Use environment config |
| Duplicating documentation | Extend existing docs |
| Proceeding without a plan | Share plan, wait for approval |
| Starting a dev server without checking the port | Check port, kill stale process, then start |

---

## API Integration (Frontend ↔ Backend)

The backend repo owns the API contract. Document first, implement second, notify frontend.

### Source of truth
- **`API_DOCUMENTATION.md` in this repo** is the contract — every endpoint, request/response shape, status code, and auth requirement lives here.
- The Postman collection in `postman/` must stay in sync with `API_DOCUMENTATION.md`.
- The frontend repo reads your docs — if they are wrong or missing, frontend work breaks.

### Backend workflow for API changes
1. Define or update the endpoint in `API_DOCUMENTATION.md` before or alongside implementation.
2. Implement controller + route using `BaseController` / `BaseRoute` patterns.
3. Register the route in `create_app.py`.
4. Add Alembic migration if the schema changed.
5. Test with Postman; update the Postman collection.
6. Notify the user when breaking changes require frontend updates.

### Contract rules
- Response shapes in code must match what `API_DOCUMENTATION.md` documents.
- Use consistent error format across endpoints (status code + message body).
- CORS must allow the frontend dev-server origin during development.
- Coordinate breaking API changes with the user before merging — the frontend repo may need updates.

---

## Commands

```bash
pipenv shell                    # always activate the Python shell first

# Before starting the server:
# 1. Check whether a Python server is already running on the expected port
# 2. Kill the existing process if one is found
# 3. Then start the server

python app.py                   # start the development server with hot-reload
python app.py demo-data         # start and seed demo data
python app.py import            # import data from Taskwize v1 to v2
```

---

## Coding Conventions

### Controllers & Routes
- **New controllers** must inherit from `BaseController` — see `src/controllers/base_controller.py`.
- **New routes** must inherit from `BaseRoute` — see `src/routes/base_routes.py`.
- Before writing a new controller or route, review the existing Task and Project implementations as reference — they demonstrate the correct inheritance and patterns.
- Do not write standalone controller or route logic. Inheriting from the base classes avoids duplicating common functionality.

### Models
- All models use SQLAlchemy ORM. Never write raw SQL.
- Review existing models before adding new fields or relationships — conventions for naming, relationships, and nullable fields are established.

### Utilities
- Check all files in `src/util/` before writing any new helper function. The utility coverage is broad — duplication is likely if you skip this step.

### API documentation
- Every new or modified endpoint must be documented in `API_DOCUMENTATION.md` before the task is marked done.
- Update the Postman collection in `postman/` to match.

---

## Project Structure

```
app.py                          # main Flask entry point with CLI commands
create_app.py                   # Flask application factory
create_all.py                   # database table creation script
import_data.py                  # data import utility (v1 to v2)
Pipfile / Pipfile.lock          # Python dependency management
requirements.txt
alembic.ini                     # Alembic migration configuration
API_DOCUMENTATION.md            # API endpoint documentation — source of truth for FE/BE contract
DEMO_DATA_README.md             # demo data setup instructions
TASK_ASSIGNMENT_README.md       # task assignment system documentation
CONTRACT_PROJECT_RELATIONSHIP.md
postman/                        # Postman collection for API testing
config/
└── config.py                   # application configuration settings
alembic/
├── env.py                      # Alembic environment configuration
├── versions/                   # database migration files
└── script.py.mako              # migration script template
logs/                           # application and Alembic migration logs
src/
├── controllers/
│   ├── base_controller.py      # ← inherit from this for all new controllers
│   ├── auth_controller.py
│   ├── user_controller.py
│   ├── organization_controller.py
│   ├── project_controller.py
│   ├── task_controller.py
│   ├── sprint_controller.py
│   ├── company_controller.py
│   ├── contact_controller.py
│   ├── contract_controller.py
│   ├── invoice_controller.py
│   ├── time_entry_controller.py
│   ├── file_controller.py
│   ├── tag_controller.py
│   ├── search_controller.py
│   └── *_comment_controller.py
├── models/
│   ├── app_user.py
│   ├── organization.py
│   ├── project.py
│   ├── task.py
│   ├── sprint.py
│   ├── company.py
│   ├── contact.py
│   ├── contract.py
│   ├── invoice.py
│   ├── time_entry.py
│   ├── file.py
│   ├── tag.py
│   ├── role.py
│   ├── permission.py
│   ├── audit_log.py
│   └── *_comment.py
├── routes/
│   ├── base_routes.py          # ← inherit from this for all new routes
│   ├── auth_routes.py
│   ├── user_routes.py
│   ├── organization_routes.py
│   ├── project_routes.py
│   ├── task_routes.py
│   ├── sprint_routes.py
│   ├── company_routes.py
│   ├── contact_routes.py
│   ├── contract_routes.py
│   ├── invoice_routes.py
│   ├── time_entry_routes.py
│   ├── file_routes.py
│   ├── search_routes.py
│   └── *_comment_routes.py
├── lib/
│   ├── db.py                   # database connection and session management
│   ├── authenticate.py         # authentication middleware
│   ├── alembic.py              # migration utilities
│   ├── s3.py                   # AWS S3 file storage integration
│   ├── qbo.py                  # QuickBooks Online integration
│   ├── loaders.py              # data loading utilities
│   └── demo_data/              # demo data generation scripts
└── util/                       # shared utility modules — check ALL before adding new ones
uploads/
├── saved_files/                # permanent file storage
└── temp-downloads/             # temporary file storage
```

---

## Database Migrations

- Use Alembic for **all** schema changes. Never modify the database directly.
- Create a migration file for every model change: `alembic revision --autogenerate -m "description"`
- Run and verify migrations locally before marking a task complete.
- Migration files live in `alembic/versions/`.

---

## QA Checklist

**Architecture — verify before running:**
- [ ] New controller inherits `BaseController`
- [ ] New route inherits `BaseRoute`
- [ ] New routes registered in the application factory (`create_app.py`)
- [ ] No raw SQL — all queries use SQLAlchemy ORM
- [ ] Environment config used for all secrets and connection strings (nothing hardcoded)
- [ ] No new utility written without first checking all files in `src/util/`
- [ ] Existing model conventions followed (naming, relationships, nullability)
- [ ] Error handling consistent with existing controllers

**Database — verify before marking done:**
- [ ] Alembic migration created and reviewed for correctness
- [ ] Migration runs cleanly: `alembic upgrade head`
- [ ] Migration rolls back cleanly: `alembic downgrade -1` then `alembic upgrade head` again
- [ ] No direct database modifications outside of migrations

**Runtime — verify with the server running:**
- [ ] Run `python app.py demo-data` as a full smoke test — server starts, data seeds without errors
- [ ] Check `logs/` for any errors or warnings after running
- [ ] All new/modified endpoints tested in Postman using the collection in `postman/`
- [ ] Endpoints return the expected response shape and status codes
- [ ] Error cases tested — missing fields, invalid IDs, unauthorized access
- [ ] New endpoints respect the role/permission system — test with a user lacking the required role

**Documentation:**
- [ ] `API_DOCUMENTATION.md` updated if any endpoints were added or modified
- [ ] Postman collection updated to match

---

## Anti-Patterns

| Avoid | Do instead |
|---|---|
| Standalone controller logic | Inherit `BaseController` |
| Standalone route logic | Inherit `BaseRoute` |
| Raw SQL queries | SQLAlchemy ORM |
| Installing packages without asking | Ask user first, then update `Pipfile.lock` |
| New utility without checking `src/util/` | Review all existing utility files first |
| Hardcoded secrets or connection strings | Use environment config via `config/config.py` |
| Direct database schema changes | Alembic migration |
| New routes not registered in app factory | Register in `create_app.py` |
| Forgetting `pipenv shell` before running anything | Always activate the shell first |
| Adding endpoints without updating docs | Update `API_DOCUMENTATION.md` and Postman |
| Backend changes without frontend coordination | Document breaking changes; notify user |

---

## Skills & References

Load the relevant skill file **before starting any task in these areas**. Do not rely on training memory for tool-specific syntax.

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

Start from this template, trim what does not apply · `API_DOCUMENTATION.md` is source of truth · document before marking API work done · plan first (except direct fixes) · ask before installing · check existing code before creating · check port and kill stale process before starting · branch `<initials>/<feature>` · **do not commit or push unless the user explicitly asks** · pipenv shell first · BaseController + BaseRoute always · SQLAlchemy not raw SQL · Alembic for all schema changes · test with demo-data + Postman · check logs/ · role/permission verified · Postman updated · notify user on breaking changes · load skill files before document or file tasks.

*When in doubt — ask the user, check existing code, update the API docs, and restart the dev server rather than debugging a stale state.*
