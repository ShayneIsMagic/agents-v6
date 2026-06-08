# AGENTS Guidelines — Frontend (React / SCSS)

Guidelines for the React frontend repository. The backend is a **separate repo** — integration happens only through a documented HTTP API.

**How to use this template:** Start from the structure below, then remove or skip sections that do not apply to the current project. Do not invent alternate conventions when a section still applies.

All paths below are relative to the **frontend repo root**.

Read **Shared Rules** first, then **API Integration**, then the coding conventions. Load any relevant Skill files before starting work.

---

## Shared Rules

These apply to all frontend agents.

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
- After any install, ensure `package-lock.json` is updated.
- Prefer JavaScript (`.jsx`) for all new components and utilities.

### Existing Code First
- Check for existing components, utilities, hooks, and helpers before creating new ones.
- Use established patterns from the codebase — don't invent new conventions when one already exists.
- When creating new components, review similar existing ones for conventions before starting.

### Security
- Use HTTPS for all links and resources. No mixed content.
- Never trust client-side validation alone — the backend validates all input.
- Use `rel="noopener noreferrer"` on all `target="_blank"` links.
- In React, use component event handlers (`onClick`, etc.) — do not use HTML `onclick` attributes or `dangerouslySetInnerHTML` with inline handlers.
- Never hardcode secrets, API keys, or backend URLs. Use `environmentConfig.js`.

### Documentation
- Do not create unnecessary documentation.
- Prefer 1–3 core `.md` files (README, `AGENTS-frontend.md`, and one optional reference). No sprawl.
- Add to existing docs when new guidance is needed rather than creating new files.
- Do not duplicate rules across README, `AGENTS-frontend.md`, and other files — one source of truth.

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
| Hardcoded secrets or backend URLs | Use `environmentConfig.js` |
| Duplicating documentation | Extend existing docs |
| Proceeding without a plan | Share plan, wait for approval |
| Starting a dev server without checking the port | Check port, kill stale process, then start |

---

## API Integration (Frontend ↔ Backend)

The backend repo owns the API contract. The frontend consumes it — never guesses shapes.

### Source of truth
- **Backend** owns `API_DOCUMENTATION.md` in the backend repo — every endpoint, request/response shape, status code, and auth requirement lives there.
- **You** must read `API_DOCUMENTATION.md` **before** implementing or changing any API call.
- If the docs are missing or unclear, ask the user — do not assume endpoint shapes.

### Frontend workflow for API changes
1. Read the updated `API_DOCUMENTATION.md` from the backend repo (or a shared link).
2. Confirm the endpoint exists and is deployed (or ask the user to coordinate backend work first).
3. Implement the UI using `useAPICall` / `useTriggeredAPICall` and `src/lib/apiCall.js`.
4. Match the documented request/response contract exactly.
5. Configure the base URL in `environmentConfig.js` — never hardcode backend URLs in components.
6. Handle documented error cases (401, 403, 404, validation errors) with toast notifications, not `alert()`.

### Contract rules
- Response shapes in code must match what `API_DOCUMENTATION.md` documents.
- Handle all documented error status codes with user-facing feedback.
- Coordinate with the user before consuming breaking API changes — backend may need to deploy first.
- If you need a new endpoint, ask the user to have backend implement and document it before you build the UI.

---

## Commands

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

---

## Coding Conventions

### Components
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

### Styles
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

### Patterns
- Use `src/hooks/fetch-hooks/useAPICall` and `useTriggeredAPICall` for API calls — don't roll custom fetch logic.
- Use `src/lib/apiCall.js` as the base API implementation.
- Use `src/util/toastNotifications.jsx` for user feedback — don't use `alert()`.
- Use `src/helpers/checkAccess.js` for access control checks.

### State management
- Application state lives in React context (`src/context/`) and component-level hooks — do not introduce `localStorage`, `sessionStorage`, or additional global state stores without an explicit discussion with the user.
- If persistent client-side state is genuinely needed for a feature, document where it lives, what key it uses, and how it is cleared — before implementing.

---

## Project Structure

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

---

## QA Checklist

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

---

## Anti-Patterns

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

Start from this template, trim what does not apply · read backend `API_DOCUMENTATION.md` before any API work · plan first (except direct fixes) · ask before installing · check existing code before creating · use the dev server only · check port and kill stale process before starting · branch `<initials>/<feature>` · **do not commit or push unless the user explicitly asks** · CSS variables always · `*Page` suffix for page components · check `core/`, `input/`, `hooks/`, `context/` before creating · state in React context not localStorage · useAPICall for fetch · toastNotifications not alert() · light + dark mode + mobile tested · no console errors in Chrome + Safari · load skill files before document or file tasks.

*When in doubt — ask the user, check existing code, read the API docs, and restart the dev server rather than debugging a stale state.*
