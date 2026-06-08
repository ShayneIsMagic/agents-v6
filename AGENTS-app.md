# AGENTS Guidelines — App (Frontend + Backend)

This template may be a **monorepo** (`/frontend` and `/backend` in one repository) or **separate frontend and backend repositories**. In both cases, frontend and backend are separate builds with separate dev servers.

- **Frontend**: React, HTML, CSS, SCSS (`/frontend`)
- **Backend**: Python Flask, Postgres, SQLAlchemy, Marshmallow (`/backend`)

**How to use this template:** Start from the structure below, then remove or skip sections that do not apply. Do not invent alternate conventions when a section still applies.

Read **Shared Rules** first, then **API Integration**, then the section for the part of the codebase you are working on. Load any relevant Skill files before starting work.

---

## Shared Rules

These apply to all agents, regardless of stack.

### Plan First
- Always create a plan and share it with the user **before making any changes**, unless the user explicitly asked for a direct fix (e.g. "fix this typo", "remove that console.log").
- Wait for explicit approval before proceeding.
- If scope, design, or approach is ambiguous — ask. Redundant confirmation beats silent assumptions and rework.

### Dev Server
- Always use the dev server while iterating. **Never run a production build during an agent session.**
- Before starting a dev server, check whether one is already running on the expected port. If one exists, kill the existing process first, then start fresh.
- When in doubt, restart the dev server rather than debugging a stale state.

### Dependencies
- **Ask the user before installing any new library or package.**
- After any install, ensure the appropriate lockfile is updated (`package-lock.json` for Node, `Pipfile.lock` for Python).

### Existing Code First
- Check for existing components, utilities, controllers, and helpers before creating new ones.
- Use established patterns from the codebase — don't invent new conventions when one already exists.
- When creating new objects or components, review similar existing ones for conventions before starting.

### Security
- Use HTTPS for production and external links. No mixed content. Local dev servers on `localhost` may use HTTP.
- Sanitize and validate all user input server-side. Never trust client-side validation alone.
- Use `rel="noopener noreferrer"` on all `target="_blank"` links.
- In React, use component event handlers (`onClick`, etc.) — do not use HTML `onclick` attributes or `dangerouslySetInnerHTML` with inline handlers.
- Never hardcode secrets, API keys, or connection strings. Use environment config.

### Documentation
- Do not create unnecessary documentation.
- Prefer 1–3 core `.md` files (README, `AGENTS-app.md`, and one optional reference). No sprawl.
- Add to existing docs when new guidance is needed rather than creating new files.
- Do not duplicate rules across README, `AGENTS-app.md`, and other files — one source of truth.
- When in doubt: fewer docs, more focused content.
- Required exceptions: `API_DOCUMENTATION.md` and the Postman collection in `postman/` — update both when endpoints change.

### Git
- Branch from `main` using the convention `<initials>/<feature-name>` (e.g. `sm/task-filter`).
- Open a PR for review when working with multiple contributors; solo work may commit directly to `main` when the user requests.
- **Do not commit or push unless the user explicitly asks.**
- Keep branches short-lived — merge or close when the feature is done.

### General Anti-Patterns

| Avoid | Do instead |
|---|---|
| Running production builds in agent sessions | Use the dev server only |
| Installing packages silently | Ask the user first |
| Creating new files/components without checking | Search existing codebase first |
| Hardcoded secrets or credentials | Use environment config |
| Duplicating documentation | Extend existing docs |
| Proceeding without a plan | Share plan, wait for approval |
| Starting a dev server without checking the port | Check port, kill stale process, then start |

---

## API Integration (Frontend ↔ Backend)

Frontend and backend integrate only through a documented HTTP API.

### Source of truth
- **Backend** owns `API_DOCUMENTATION.md` (at `/backend/API_DOCUMENTATION.md` in a monorepo, or in the backend repository when split) — every endpoint, request/response shape, status code, and auth requirement lives here.
- **Backend** maintains the Postman collection in `postman/` — keep it in sync with `API_DOCUMENTATION.md`.
- **Frontend** reads `API_DOCUMENTATION.md` from the backend **before** implementing or changing any API call.

### Creating or changing an API
1. **Backend first:** Define or update the endpoint in `API_DOCUMENTATION.md` **before** implementation.
2. Implement controller + route using `BaseController` / `BaseRoute` patterns.
3. Register the route in `create_app.py`.
4. Add Alembic migration if the schema changed.
5. Test with Postman; update the Postman collection.
6. **Frontend second:** Implement the UI using `useAPICall` / `useTriggeredAPICall` and `src/lib/apiCall.js`, matching the documented contract exactly.
7. Configure the frontend base URL in `environmentConfig.js` — never hardcode backend URLs in components.
8. Run both dev servers and verify the feature in the browser.

### Contract rules
- Documented request/response shapes in `API_DOCUMENTATION.md` must match what the backend returns (Marshmallow-serialized output).
- Match the error format used by existing endpoints — see `API_DOCUMENTATION.md` and existing Task/Project implementations.
- Auth for each endpoint is documented in `API_DOCUMENTATION.md`. Frontend auth calls use `src/services/authServices.js`.
- Frontend must handle documented error cases (401, 403, 404, validation errors) with toast notifications, not `alert()`.
- Configure CORS in `config/config.py` to allow the frontend dev-server origin during development.
- Coordinate breaking API changes with the user before merging — both sides may need updates.

---

## Frontend (React / SCSS)

All paths below are relative to **`/frontend`**.

### Commands

```bash
# Check package.json "scripts" — use the standard dev script for this project:
npm start          # common for Create React App
# or
npm run dev        # common for Vite and similar tooling

# Before running either command:
# 1. Check whether a dev server is already running on the expected port
# 2. Kill the existing process if one is found
# 3. Then start the dev server
```
**Do not run `npm run build` inside an agent session.**

### Coding Conventions

#### Components
- **Before creating a new component**, search `src/components/` thoroughly — especially:
  - `core/` — Avatar, Badge, Button, ColorPicker, ComboBox, DataTable, KanbanBoard, MarkdownEditor/Viewer, PaginationButtons, ThemeSwitcher
  - `input/` — Email, FormFieldPassword, FormFieldSelect, FormFieldTags, FormFieldText, QuillEditor, SearchInput, TextField
  - `layout/` — Form, FormRow, Label, Loading
  - `modals/` — ConfirmDelete, Modal
  - `navigation/` — CollapsibleSection, Header, NavigationLink, ProfileMenu, Sidebar
  - `ColorBox.jsx` and `DragDrop.jsx` — standalone utility components at `src/components/` root
- Check `src/context/` before adding new context providers — AppDataContext, AuthContext, ThemeContext already exist.
- Check `src/hooks/` before writing new hooks — useAPICall, useTriggeredAPICall, useAbortSignal, useClickOutside, useDataColumns, useDebounce, useDeepEffect, useDelayedLoading, useDynamicFilter, usePagination already exist.
- **Page components** live in `src/components/pages/` and use the `*Page` suffix (e.g. `ProjectListPage.jsx`, `ProjectDetailPage.jsx`). Shared forms use the `*Form` suffix. `Home.jsx` and `NoMatch.jsx` at the `pages/` root are established exceptions.

#### Styles
- **Before writing new styles**, check:
  - `src/styles/abstracts/` for variables, mixins, and theme definitions (light/dark)
  - `src/styles/components/` for existing component-level SCSS
  - `src/styles/layout/layouts.scss` for inline utility classes (used like Tailwind — check here before writing custom layout CSS)
- Use **CSS variables** for all colors, borders, and spacing. **Never hardcode hex values.**
- This project uses CSS variables for theming. Always verify you are using the correct variable — raw colors will break light/dark mode.
- Place new style files in the location that mirrors their component:
  - Component styles → `src/styles/components/`
  - Page styles → `src/styles/pages/`
  - Layout styles → `src/styles/layout/`
- Review `ProjectListPage.jsx` and `ProjectDetailPage.jsx` as reference before adding new page-level styles.
- `src/deleted-styles/` contains legacy SCSS kept for reference — do not import or reuse these files.

#### Patterns
- Use `src/hooks/fetch-hooks/useAPICall` and `useTriggeredAPICall` for API calls — don't roll custom fetch logic.
- Use `src/lib/apiCall.js` as the base API implementation.
- `src/services/api.js` and `src/services/authServices.js` hold established service helpers — extend these for shared API and auth logic; do not add parallel fetch implementations.
- Use `src/util/toastNotifications.jsx` for user feedback — don't use `alert()`.
- Use `src/helpers/checkAccess.js` for access control checks.

#### State management
- Application state lives in React context (`src/context/`) and component-level hooks — do not introduce `localStorage`, `sessionStorage`, Redux, or other global state stores without an explicit discussion with the user. `src/reducer/` is reserved for future use — do not add Redux.
- If persistent client-side state is genuinely needed for a feature, document where it lives, what key it uses, and how it is cleared — before implementing.

### Project Structure

```
src/
├── assets/
│   ├── icons/               # regularIcons.js
│   └── images/              # logos, SVGs
├── components/
│   ├── auth/                # PrivateRoute, PublicRoute, SecurityWrapper, password mgmt
│   ├── core/                # Avatar, Badge, Button, ColorPicker, ComboBox, DataTable,
│   │                        # KanbanBoard, MarkdownEditor, MarkdownViewer,
│   │                        # PaginationButtons, ThemeSwitcher
│   ├── files/               # FileList
│   ├── forms/               # EditAvatar, LoginForm, PhotoCropper
│   ├── input/               # Email, FormFieldPassword, FormFieldSelect, FormFieldTags,
│   │                        # FormFieldText, QuillEditor, SearchInput, TextField
│   ├── layout/              # Form, FormRow, Label, Loading
│   ├── modals/              # ConfirmDelete, Modal
│   ├── navigation/          # CollapsibleSection, Header, NavigationLink,
│   │                        # ProfileMenu, Sidebar
│   ├── pages/
│   │   ├── auth/            # LoginPage, RegisterPage, VerifyAccountPage, etc.
│   │   ├── company/         # CompaniesListPage, CompanyPage
│   │   ├── contact/         # ContactsListPage
│   │   ├── organization/    # OrganizationForm, OrganizationPage, OrganizationSelect,
│   │   │                    # OrganizationsListPage
│   │   ├── project/         # ProjectDetailPage, ProjectForm, ProjectListPage
│   │   ├── settings/        # SettingsPage, TagsPage
│   │   ├── sprint/          # SprintDetailPage, SprintForm, SprintListPage
│   │   ├── task/            # TaskDetailPage, TaskListPage
│   │   ├── user/            # UserForm, UserListPage, UserPage, UserRoleSelect
│   │   └── Home.jsx, NoMatch.jsx, UniversalSearch.jsx
│   ├── routing/             # PrivateRoutes
│   ├── sprint/components/   # BurndownChart, SprintTable, TaskSelector
│   ├── tags/                # Tag, TagInputComponent, TagsInput
│   ├── task/                # TaskAssignments, TaskDetailDrawer, TaskKanban, TaskTable
│   ├── App.jsx
│   └── ColorBox.jsx, DragDrop.jsx  # standalone utility components
├── context/                 # AppDataContext, AuthContext, ThemeContext
├── deleted-styles/          # legacy SCSS — reference only, do not import
├── helpers/
│   ├── avatar/              # avatarArrays, calculateAvatarScale
│   ├── tags/                # addTagStyles, sortTags
│   ├── checkAccess.js
│   ├── colorPickerDefaults.js
│   └── fetchHelpers.js
├── hooks/
│   ├── fetch-hooks/         # useAPICall, useTriggeredAPICall
│   ├── useAbortSignal.js
│   ├── useClickOutside.js
│   ├── useDataColumns.jsx
│   ├── useDebounce.js
│   ├── useDeepEffect.js
│   ├── useDelayedLoading.js
│   ├── useDynamicFilter.js
│   └── usePagination.js
├── lib/
│   └── apiCall.js
├── reducer/                 # Redux reducers (future use)
├── services/
│   ├── api.js
│   └── authServices.js
├── styles/
│   ├── abstracts/           # variables, mixins, theme definitions (light/dark)
│   ├── auth/
│   ├── base/                # base styles and reset
│   ├── components/          # avatars, badges, buttons, color-picker, combo-box,
│   │                        # data-table, drag-drop, file-list, inputs, kanban-board,
│   │                        # loading, modals, pagination-buttons, tags,
│   │                        # task-assignments, theme-switcher, toast-notifications,
│   │                        # toggle-slider
│   │   └── input/           # quill-editor
│   ├── layout/              # header, sidebar, layouts
│   ├── pages/               # details-page, edit-avatar, photo-cropper, tags-page
│   └── app.scss
├── util/
│   ├── backgroundImageScale.js
│   ├── colorUtils.js
│   ├── filterObjects.js
│   ├── isDeepEqual.js
│   ├── stringUtils.js
│   ├── toastNotifications.jsx
│   └── toBase64.js
├── index.jsx                # application entry point
└── environmentConfig.js     # API base URL and environment configuration
```

### QA Checklist

**Code review — run before marking anything done:**
- [ ] No hardcoded colors — all values use CSS variables
- [ ] No duplicate component — searched all of `src/components/` before creating
- [ ] No duplicate hook — searched `src/hooks/` before creating
- [ ] No duplicate context — searched `src/context/` before creating
- [ ] Style file placed in the correct directory mirroring its component
- [ ] No imports from `deleted-styles/`
- [ ] Toast notifications used for user feedback (not `alert()`)
- [ ] `useAPICall` / `useTriggeredAPICall` used for all API calls — no custom fetch logic
- [ ] API calls match shapes documented in backend `API_DOCUMENTATION.md`

**Accessibility — verify in the browser:**
- [ ] `alt` text present on all images
- [ ] Focus states visible on all interactive elements — tab through the UI
- [ ] `aria-label` present where needed (icon-only buttons, sections, toggles)
- [ ] Active/current state reflected in nav (`aria-current="page"` or `.active` class)

**Visual & theming — verify in the browser:**
- [ ] Light mode renders correctly — no raw colors bleeding through
- [ ] Dark mode renders correctly — switch theme and check all affected components
- [ ] No layout breakage at mobile viewport (375px) — resize the browser
- [ ] No layout breakage at tablet viewport (768px)
- [ ] Spot check in Safari — flexbox and CSS variable behavior differs from Chrome

**Runtime — verify in the browser:**
- [ ] No console errors or warnings in Chrome DevTools
- [ ] No console errors or warnings in Safari DevTools
- [ ] All API calls succeed and return expected data shapes
- [ ] Error states handled — test with network throttled or disconnected
- [ ] Toast notifications fire correctly for success and error states

### Anti-Patterns

| Avoid | Do instead |
|---|---|
| Hardcoded hex colors | CSS variables from `abstracts/` |
| New style file before checking existing | Check `abstracts/`, `components/`, `layouts.scss` first |
| New component without searching `src/components/` | Search all subdirectories first |
| New hook without checking `src/hooks/` | Review all existing hooks first |
| New context without checking `src/context/` | Review AppDataContext, AuthContext, ThemeContext first |
| HTML `onclick` attributes or inline handlers in JSX | Component event handlers (`onClick`, etc.) |
| `alert()` for user feedback | `toastNotifications.jsx` |
| Custom fetch logic | `useAPICall` / `useTriggeredAPICall` hooks |
| Guessing API response shapes | Read backend `API_DOCUMENTATION.md` first |
| Hardcoded backend URLs in components | `environmentConfig.js` |
| Styles scattered outside their component location | Follow directory conventions |
| Importing from `deleted-styles/` | Use only active styles in `src/styles/` |
| Skipping `ProjectListPage.jsx` / `ProjectDetailPage.jsx` as style reference | Always review before new page styles |

---

## Backend (Flask / Postgres)

All paths below are relative to **`/backend`**.

### Commands

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

### Coding Conventions

#### Controllers & Routes
- **New controllers** must inherit from `BaseController` — see `src/controllers/base_controller.py`.
- **New routes** must inherit from `BaseRoute` — see `src/routes/base_routes.py`.
- Before writing a new controller or route, review the existing Task and Project implementations as reference — they demonstrate the correct inheritance and patterns.
- Do not write standalone controller or route logic. Inheriting from the base classes avoids duplicating common functionality.

#### Models
- All models use SQLAlchemy ORM. Never write raw SQL.
- Review existing models before adding new fields or relationships — conventions for naming, relationships, and nullable fields are established.

#### Utilities
- Check all files in `src/util/` before writing any new helper function. The utility coverage is broad — duplication is likely if you skip this step.

#### API documentation
- Use Marshmallow schemas for request/response serialization.
- Every new or modified endpoint must be documented in `API_DOCUMENTATION.md` **before** the task is marked done. Document the serialized request/response shape.
- Update the Postman collection in `postman/` to match.

### Project Structure

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

### Database Migrations

- Use Alembic for **all** schema changes. Never modify the database directly.
- Create a migration file for every model change: `alembic revision --autogenerate -m "description"`
- Run and verify migrations locally before marking a task complete.
- Migration files live in `alembic/versions/`.

### QA Checklist

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
- [ ] Both dev servers running — feature verified in the browser against the frontend

### Anti-Patterns

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

Load the relevant skill file **before starting any task in these areas**. Do not rely on training memory for tool-specific syntax — skill files encode environment-specific constraints and known pitfalls.

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

Plan first (except direct fixes) · ask before installing · check existing code before creating · use the dev server only · check port and kill stale process before starting · branch `<initials>/<feature>` · **do not commit or push unless the user explicitly asks** · API docs are source of truth (`/backend/API_DOCUMENTATION.md`) · backend + docs before implementation, frontend second · verify in browser with both dev servers · **FE**: CSS variables always · `*Page` suffix · check `core/`, `input/`, `hooks/`, `context/` before creating · useAPICall / apiCall.js · extend `services/` not parallel fetch · authServices for auth · toastNotifications not alert() · light + dark mode + mobile tested · no console errors in Chrome + Safari · **BE**: pipenv shell first · BaseController + BaseRoute · Marshmallow serialization · SQLAlchemy not raw SQL · Alembic for schema · CORS in config/config.py · test with demo-data + Postman · check logs/ · role/permission verified · API docs + Postman updated before done · load skill files before document or file tasks.

*When in doubt — ask the user, check existing code, read the API docs, and restart the dev server rather than debugging a stale state.*
