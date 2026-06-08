# AGENTS Guidelines — Static Sites (HTML / CSS / JS)

This document covers plain static websites built with HTML, CSS, and JavaScript — no React, no Flask. Read it in full before starting any work on a static site.

**How to use this template:** Start from the structure and build order below, then remove or skip sections that do not apply to the current project (e.g. skip GTM/SEO depth if the site is internal-only). Do not invent alternate conventions when a section still applies.

**Default to asking.** Prefer more questions over fewer. If scope, design, content, analytics, forms, or hosting are partly specified or ambiguous — ask, even when questions overlap. Redundant confirmation beats silent assumptions and rework.

---

## Shared Rules

### Plan First
- Always create a plan and share it with the user **before making any changes**, unless the user explicitly asked for a direct fix (e.g. "fix this broken link", "update this copy").
- Wait for explicit approval before proceeding.

### Dev Server
- Always use a local development server while iterating. Never deploy directly from an agent session as the first test.
- Before starting a dev server, check whether one is already running on the expected port. If one exists, kill the existing process first, then start fresh.
- When in doubt, restart the dev server rather than debugging a stale state.

```bash
# Use the command specified in the project README — common options:
npx serve .                      # Node static server
python3 -m http.server 8080      # Python built-in server

# Before running either command:
# 1. Check whether a server is already running on the expected port
# 2. Kill the existing process if one is found
# 3. Then start the dev server
```

### Dependencies
- **Ask the user before installing any new library or package.**
- After any install, ensure the appropriate lockfile is updated (if the project uses one).

### Security
- Use HTTPS for all links and resources. No mixed content.
- Sanitize and validate all user input. Never trust client-side validation alone.
- Use `rel="noopener noreferrer"` on all `target="_blank"` links.
- Avoid inline JavaScript (`onclick`, `onload`, etc.) — use external scripts and event delegation.

### Documentation
- Do not create unnecessary documentation.
- Prefer 1–3 core `.md` files (README and `AGENTS-static.md`). No sprawl.
- Add to existing docs when new guidance is needed rather than creating new files.
- Update the README title, description, and "Running Locally" commands once they are known.

### Git
- Branch from `main` using the convention `<initials>/<feature-name>` (e.g. `sm/contact-form`).
- Open a PR when working with multiple contributors; solo work may commit directly to `main` when the user requests.
- **Do not commit or push unless the user explicitly asks.**
- Keep branches short-lived — merge or close when the feature is done.

### Client-side state and data privacy
- If a tool or page stores data in `localStorage` or `sessionStorage`, document it explicitly: what key is used, what is stored, and how it is cleared.
- Nothing should be sent to a server without the user's knowledge. If a server endpoint is added later, conduct a privacy review first.
- **Never trust client-side validation alone** if a server endpoint exists or is planned.

---

## Before Starting a Build

Confirm the following **before** writing any CSS variables, layout, or page structure:

- **Fonts** — names, weights, and any local files
- **Color palette** — hex values or design tokens
- **Logo and brand assets** — file formats and locations
- **Pages and sections** — full sitemap
- **Forms** — fields, endpoint (Formspree, Netlify Forms, etc.), captcha requirement
- **Analytics** — GTM container ID, or whether GTM is needed at all
- **Hosting** — any constraints on URL structure, redirects, or 404 handling
- **Accessibility and SEO priorities**

If any of the above are unavailable, use clearly marked placeholders and list what is still needed.

**If working in a Git repo:** create a feature branch (`<initials>/<website-name>`) and build on that branch. Do not commit or push unless the user explicitly asks.

---

## Suggested Build Order

1. Create the file structure (`css/`, `js/`, `assets/`, `components/`) — include `css/variables.css` (design tokens) and `css/common.css` (resets, base styles, utilities)
2. Create shared `main.css` (homepage styles) and `main.js`; reference `variables.css`, `common.css`, `main.css`, and `main.js` from all pages
3. Build shared components (header, footer, nav) as partials or copy consistently across pages
4. Add pages (index, about, contact, etc.) using the Common Patterns below — add page-specific CSS in `css/pages/` as needed
5. Add GTM placeholder if analytics is needed; leave `dataLayer` init and container ID for the user to fill
6. Add `404.html` with a "Page not found" message and a link to home
7. Update README with the site name, description, and the exact commands to run it locally

---

## When Fixing an Existing Site

**Audit order — work through these in sequence:**
1. Security and accessibility first — these are never deferred
2. Structure — shared CSS/JS, extract repeated header/footer into partials
3. Tracking consolidation — remove duplicate GTM/gtag, fix `dataLayer` init order
4. Dead code — remove orphaned DOM queries and listeners when markup changes; null checks alone are not a fix

**Specific fixes to make:**
- Migrate all inline `onclick` handlers to `data-*` attributes with event delegation
- Move global tracking functions out of inline `<script>` blocks into an external file or IIFE
- Remove duplicated CSS blocks — shared styles belong in `variables.css`, `common.css`, and component/page files, not inline or copy-pasted across pages
- Fix or remove any DOM queries referencing elements that no longer exist

**Completion criteria — the fix is done when:**
- [ ] All items in the QA Checklist below pass
- [ ] No inline `onclick` handlers remain in the markup
- [ ] No duplicate GTM or gtag scripts on any page
- [ ] No duplicated CSS across pages — shared styles in `variables.css`, `common.css`, and component/page files only
- [ ] No orphaned JS referencing removed elements
- [ ] Security audit passes: HTTPS links, `rel="noopener noreferrer"` on all `target="_blank"`, no mixed content

---

## HTML Best Practices

- Use semantic markup: `<header>`, `<main>`, `<footer>`, `<nav>`, `<section>`, `<article>`. Avoid generic `<div>` when a semantic element fits.
- Always include `<!DOCTYPE html>`, `<html lang="en">`, `<meta charset="utf-8">`, and `<meta name="viewport" content="width=device-width, initial-scale=1">`.
- Add a skip-to-content link at the very top (`href="#main"`) and `id="main"` on `<main>` for screen readers.
- Include `aria-label` on landmark sections, `alt` on all images, `aria-expanded` on toggles.
- Dropdowns and menus must support keyboard navigation: Enter/Space to toggle, Escape to close, arrow keys where applicable.
- Ensure visible focus states on all interactive elements. Aim for WCAG AA contrast (4.5:1 for text, 3:1 for large text).
- Use absolute URLs for `og:image`, `twitter:image`, and `canonical` links so crawlers resolve them correctly.
- Include a favicon (`favicon.ico` or `<link rel="icon">`) and `apple-touch-icon`.
- Associate every input with a `<label>` via `for`/`id` or by wrapping.
- Use `rel="noopener noreferrer"` on all `target="_blank"` links.
- Use consistent URL formats. Avoid `action="#"` on forms that submit via JavaScript unless intentional — ensure the submit handler calls `preventDefault()`.

---

## CSS Best Practices

- Move shared styles into external stylesheets and load via `<link rel="stylesheet">`. No duplication across pages.
- Use `:root` CSS variables in `variables.css` for all colors, spacing, and typography.
- Use a consistent set of breakpoints across all pages (e.g. 768px, 1024px). Avoid arbitrary variation.
- Avoid `!important`. Use proper specificity and structure instead.
- Keep CSS readable — minification is a build step, not a development practice.
- Use consistent class naming conventions site-wide (e.g. `.btn` not `.button` on one page and `.btn` on another).

---

## JavaScript Best Practices

- Move shared scripts to an external `.js` file. Non-trivial or reused script belongs in a file, not an inline block.
- Use `data-*` attributes with event delegation rather than inline `onclick` handlers.
- Register each interaction in one place — avoid duplicating the same handler logic across scripts or load hooks.
- Use IIFEs or modules to avoid polluting global scope. Prefer `const` and `let` over `var`.
- Query and manipulate the DOM only after `DOMContentLoaded`, or place scripts at the end of `<body>`.
- Guard all DOM lookups with null checks before calling methods. If the element no longer exists in the markup, remove the related code — don't rely only on null guards.
- Debounce or throttle scroll, resize, and input handlers to avoid blocking the main thread.
- Always initialize `dataLayer` before GTM loads: `window.dataLayer = window.dataLayer || [];`
- Never use `alert()` for form feedback — use inline messages or toast notifications.
- Validate required fields and any captcha before submitting a form. Handle fetch errors and show user feedback.

---

## SEO & Google Tag Manager

- Use a unique `<title>` and `<meta name="description">` on every page.
- One `<h1>` per page. Use `<h2>` for sections, `<h3>` for sub-sections. Never skip heading levels.
- Add JSON-LD structured data when the page clearly fits a schema type:
  - **Organization** + **WebSite** on the home page for brand/business sites
  - **LocalBusiness** (or a specific subtype) when there is a real address or local discovery matters
  - **Article**, **Product**, **Event**, **FAQPage** only when the page is genuinely that content type and the visible text matches the structured data
  - Skip JSON-LD on bare placeholders or when the client doesn't want SEO depth
  - Use [Google's Rich Results Test](https://search.google.com/test/rich-results) when adding anything beyond a simple Organization/WebSite block
- Keep site-wide JSON-LD in one shared include or partial — do not hand-paste the same JSON into every file.
- Inline JSON-LD in each page's `<head>` as `<script type="application/ld+json">` — do not rely on client-side JS injection alone.
- Use lowercase, hyphen-separated URLs (`/contact-us/`, `/about-our-team/`). Avoid query strings for pages meant to be indexed.
- Write descriptive, concise alt text for all images. Use `alt=""` for purely decorative images.
- Use descriptive anchor text on internal links — not "click here".
- Place the GTM snippet as early in `<head>` as practical, after charset and viewport meta tags.
- **Do not** load both GTM and a separate `gtag.js` on the same page — it causes double tracking.
- Push events with `dataLayer.push()`. Centralize tracking configuration in GTM rather than scattering inline scripts.
- Include the GTM noscript iframe in `<body>`.
- Consider `robots.txt` and `sitemap.xml` for crawlers on any site intended for indexing.

---

## Performance

- Use `loading="lazy"` for all below-the-fold images and iframes.
- Set explicit `width` and `height` attributes on images to prevent layout shift (CLS).
- Use `srcset` and `sizes` for responsive images when serving multiple resolutions. Prefer WebP or AVIF.
- Use `fetchpriority="high"` only on critical hero images.
- Preload key fonts with `<link rel="preload" as="font">`. Use `font-display: swap` or `optional`.
- Load scripts with `defer` or `async`. Minimize render-blocking resources.
- Use `<link rel="preconnect">` for critical third-party origins (fonts, analytics).
- For `<video>`, use `preload="none"` to defer loading when not immediately needed.

---

## Browser & Device Support

- Support the last 2 major versions of Chrome, Safari, Edge, and Firefox.
- Design and develop **mobile-first**. Use `min-width` media queries to enhance for larger screens.
- Provide fallbacks for modern features when supporting older browsers (e.g. `@supports` for progressive enhancement).

---

## File Structure

```
/
├── index.html
├── pages/
│   ├── about.html
│   └── contact.html
├── components/                # shared reusable layout components
│   ├── header.html
│   └── footer.html
├── css/
│   ├── variables.css          # design tokens — colors, spacing, typography
│   ├── common.css             # resets, base styles, utilities
│   ├── main.css               # homepage styles
│   ├── components/
│   │   ├── header.css
│   │   └── footer.css
│   └── pages/
│       ├── about.css
│       └── contact.css
├── js/
│   └── main.js
├── assets/
│   ├── icons/
│   │   ├── favicon.ico
│   │   └── apple-touch-icon.png
│   └── images/
│       ├── logo.svg
│       └── *.webp
├── 404.html
├── robots.txt
├── sitemap.xml
├── README.md
└── AGENTS-static.md
```

---

## Common UI Patterns

| Pattern | Usage |
|---|---|
| **Hero block** | Full-width headline + supporting text + CTA; above the fold |
| **Testimonial card** | Quote, attribution, optional photo; consistent card styling |
| **CTA button** | Primary (filled) and secondary (outline) variants; consistent sizing |
| **Accordion / FAQ** | Use `<details>`/`<summary>` or an ARIA pattern; must be keyboard accessible |
| **Contact form** | Name, email, phone, message; each input with a `<label>`; captcha when needed; inline success/error messages; endpoint left for user to configure |
| **Nav with current state** | Indicate active page with `aria-current="page"` or a consistent `.active` class |

---

## QA Checklist

**Code review — check before opening the browser:**
- [ ] `<!DOCTYPE html>` and `lang` attribute present on every page
- [ ] Skip-to-content link present and `id="main"` on `<main>`
- [ ] All form inputs have associated `<label>` elements
- [ ] Favicon and `apple-touch-icon` present
- [ ] No duplicate GTM / gtag scripts on any page
- [ ] `window.dataLayer = window.dataLayer || [];` initialized before GTM snippet
- [ ] GTM noscript iframe present in `<body>`
- [ ] Absolute URLs used for OG/Twitter images and canonical links
- [ ] JSON-LD present and correct where content fits a schema type
- [ ] All `target="_blank"` links have `rel="noopener noreferrer"`
- [ ] No inline `onclick` handlers — event delegation used throughout
- [ ] `robots.txt` and `sitemap.xml` present if site is to be indexed
- [ ] README updated with site name, description, and exact run commands
- [ ] No dead links — check all internal `href` values resolve to real pages

**Accessibility — verify in the browser:**
- [ ] Skip-to-content link works and is visible on focus
- [ ] Visible focus states on all interactive elements — tab through every page
- [ ] All dropdowns and menus operable by keyboard (Enter/Space/Escape/arrow keys)
- [ ] `alt` text on all images; `alt=""` on decorative images
- [ ] Heading hierarchy correct — one `<h1>`, no skipped levels
- [ ] WCAG AA contrast passes — check with browser DevTools or a contrast checker
- [ ] Active page indicated in navigation on every page

**Visual & layout — verify in the browser:**
- [ ] Renders correctly at mobile viewport (375px) — no overflow, no broken layout
- [ ] Renders correctly at tablet viewport (768px)
- [ ] Renders correctly at desktop viewport (1280px+)
- [ ] Header and footer consistent across all pages
- [ ] No layout shift on page load — images have explicit `width` and `height`

**Cross-browser — test in each before delivering:**
- [ ] Chrome — primary development browser, full check
- [ ] Safari — CSS variable and flexbox behavior, font rendering
- [ ] Firefox — layout and focus state verification
- [ ] Edge — spot check for any quirks

**Forms — if the site has a contact or submission form:**
- [ ] Form submits successfully end to end — test the actual endpoint
- [ ] Success message displays correctly after submission
- [ ] Error message displays correctly on failed or invalid submission
- [ ] Required field validation works before submission fires
- [ ] Captcha fires correctly if configured

**Tracking — if GTM is present:**
- [ ] Open GTM Preview mode and verify key triggers fire on correct interactions
- [ ] No double-firing of pageview or event tags
- [ ] `dataLayer.push()` calls appear correctly in GTM debug panel

**Performance — run before final delivery:**
- [ ] Run Lighthouse in Chrome DevTools (Performance, Accessibility, SEO tabs)
- [ ] Accessibility score 90+ — fix any flagged issues before delivering
- [ ] SEO score 90+ — fix any flagged issues before delivering
- [ ] All below-the-fold images use `loading="lazy"`
- [ ] Hero/above-fold images load without delay — no `loading="lazy"` on critical images

**Interactive tools — if the page has stateful JS tools:**
- [ ] `localStorage` save and restore works correctly in the same browser session
- [ ] Data persists correctly after page reload
- [ ] Export / import or JSON backup works if the tool supports it
- [ ] Reset clears all expected state and storage keys
- [ ] No console errors when switching tabs, panes, or tool states

**404 — verify the error page works:**
- [ ] Navigate to a non-existent URL — confirm `404.html` serves correctly in the hosting environment

---

## Anti-Patterns

### HTML

| Avoid | Do instead |
|---|---|
| Repeating header/footer markup in every HTML file | Shared includes, partials, or a build step |
| Relative URLs for OG/Twitter images and canonical | Absolute URLs |
| Relative URLs for preload links | Absolute or root-relative URLs |
| Inline layout/presentation styles (`display`, flex/grid, spacing) | CSS classes; inline only for values set at runtime |
| Skipping heading levels (h1 → h3) | Logical hierarchy: h1 → h2 → h3, no skips |
| Generic alt text ("image1.jpg", "photo") | Descriptive, concise alt text |
| Decorative images with descriptive alt | Use `alt=""` |
| Non-descriptive link text ("click here") | Anchor text describing the destination |
| Inputs without associated `<label>` | `for`/`id` pairing or wrapped label |
| `action="#"` with no JS fallback | Real endpoint, or documented that JS handles submission |
| Missing `cursor: pointer` on custom clickable elements | Add it for non-button elements acting as buttons |

### CSS

| Avoid | Do instead |
|---|---|
| Duplicating large CSS blocks across pages | Shared files: `variables.css`, `common.css`, component/page CSS |
| Mixing hardcoded colors with CSS variables | CSS variables consistently throughout |
| Conflicting media queries at different breakpoints | One consistent breakpoint system |
| Using `!important` | Proper specificity and structure |
| Undefined classes referenced in HTML | Define the class or remove its use |
| Inconsistent class names for the same pattern | Same names site-wide |

### JavaScript

| Avoid | Do instead |
|---|---|
| Inline `onclick` on many elements | `data-*` attributes with a single delegated listener |
| Global tracking functions on `window` | IIFE, module, or external file |
| `dataLayer.push()` without initializing | Initialize `window.dataLayer` before GTM loads |
| Calling analytics APIs before they load | Ensure the script loads before invoking it |
| DOM queries without null checks | Check for element existence; remove dead code when markup changes |
| Duplicating the same event logic in multiple places | One delegated listener or a single init module |
| Top-level tracking functions in inline `<script>` | External file, IIFE, or module |
| Copying the same script into every page | Single shared file included once |
| `alert()` for form feedback | Inline messages or toast notifications |
| Synchronous scripts in `<head>` | `defer` or `async`, or place at end of `<body>` |
| Starting dev server without checking the port | Check port, kill stale process, then start |

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

Start from this template, trim what does not apply · default to asking · plan first (except direct fixes) · confirm fonts/colors/assets before writing CSS · ask before installing · branch `<initials>/<feature>` · **do not commit or push unless the user explicitly asks** · check port and kill stale process before starting dev server · document localStorage keys and what they store · DOCTYPE + lang · semantic HTML · skip-link + `id="main"` · a11y (aria, alt, focus, labels, contrast) · favicon · `noopener` on `_blank` · HTTPS · external CSS/JS · delegation not onclick · remove dead DOM code · absolute social/canonical URLs · JSON-LD inline when content fits · mobile-first · `dataLayer` before GTM · no duplicate GTM/gtag · lazy images below fold · debounce heavy handlers · Lighthouse Accessibility + SEO ≥ 90 · test Chrome + Safari + Firefox + Edge · test 375px + 768px + 1280px · load skill files before document or file tasks.

*When in doubt — ask the user, check existing code, restart the dev server, and confirm brand assets before writing a single line of CSS.*
