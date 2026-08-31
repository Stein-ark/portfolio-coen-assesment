# Technical Report

**Personal Portfolio — Jonathan E. Abogye**
COEN 554 Web Programming · Ahmadu Bello University, Zaria
Build date: 26 August 2026

---

## 0. Scope and constraints

The site is eight hand-written HTML5 pages, four hand-written stylesheets, three
self-hosted font files, one photograph and four JSON files. There is no build
step, no package manager, no framework and no runtime JavaScript.

| Constraint | How it is met | How to verify |
|---|---|---|
| No JavaScript | No `.js` files exist; no inline event handler attributes; no `javascript:` URLs | `grep -rE '\son(click\|change\|submit\|load)=' *.html` returns nothing |
| Only JSON-LD `<script>` | 4 blocks, all `type="application/ld+json"` | `grep -oh '<script[^>]*>' *.html \| sort \| uniq -c` |
| No CSS framework | All CSS in `assets/css/` is written for this site | No `@import`, no vendor filenames |
| No external requests | The only `url()` values in CSS are the three local fonts; all icons are inline SVG | `grep -oh 'url([^)]*)' assets/css/*.css` |

The four outbound `href`s (GitHub, LinkedIn, and two `mailto:`) are user-initiated
navigation, not resource loads. Disconnect the network and every page renders
identically.

---

## 1. Visual design rationale

### 1.1 The asymmetric grid

The homepage is a three-column CSS Grid at `40fr / 30fr / 30fr`, filling the
viewport height. The asymmetry is the point: a symmetric three-column layout
reads as a template, while a dominant left column establishes a clear reading
order — headline first, then the photograph as the visual anchor, then the
supporting project detail on the right.

Two grid decisions are load-bearing and were both found by measurement rather
than assumption:

**`minmax(0, 40fr)` rather than `40fr`.** A grid track's automatic minimum is
`auto`, meaning it will not shrink below its content's minimum size. The first
draft used bare `fr` units, and the oversized display headline silently pushed
the split from 40/30/30 to 57/22/21 — the layout looked plausible but was not
the specified geometry. `minmax(0, …)` caps the automatic minimum at zero and
holds the ratio at every viewport.

**The header is lifted out of the grid.** On the homepage the `<header>` is
absolutely positioned across the top of the composition with no background and
no rule, and the photograph's grid cell spans every row starting at `y = 0`.
That is what lets the picture bleed past the header rather than starting below
it. The side columns carry generous padding; the picture cell carries none. The
contrast between padded columns and an unpadded full-bleed cell is the whole
composition.

The photograph is deliberately shorter than the column (`height: 62vh`,
`align-self: start`), leaving the lower part of the centre column white through
to the bottom band. The height is carried by the `<figure>`, not the `<img>`:
shrinking the image inside a full-height figure leaves an empty box below the
picture, whereas sizing the figure lets `object-fit: cover` crop cleanly.

Interior pages do not repeat the hero. They open with a large display title over
a hairline rule, then use a two-column body — a narrow sticky left rail carrying
a specification table, and a wide right column carrying the prose. The rail is
`position: sticky`, which is a layout property, not a script.

### 1.2 The typographic split, and why Orbitron is confined to display sizes

Orbitron is a squared geometric display face. Its strengths — even stroke
weight, wide flat curves, engineered feel — are exactly what make it hostile to
running text: the letterforms are low-contrast and similar in width, so at
paragraph sizes word shapes stop being distinguishable and reading slows.

It is therefore restricted to: display headings, the page titles, navigation
labels, specification-table labels, tags, buttons and the logo mark. Every
paragraph, description and table value is set in the native system stack
(`-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial,
sans-serif`), which costs nothing to load and is optimised by the operating
system for exactly this job.

The display treatment is uppercase, `letter-spacing: -0.03em`,
`line-height: 0.85`. Negative tracking on a wide face closes the gaps its
geometry opens up; sub-1 leading makes stacked lines nearly touch. `ENGI` over
`NEER` is the clearest expression of it. Display sizes are fluid via `clamp()`,
with a separate ceiling inside the `≥1024px` block because at that point the
headline shares the viewport with two other columns and can no longer be sized
against the full width.

Body copy is capped at `62ch`. Beyond roughly 75 characters the eye loses the
line when it returns to the left margin.

### 1.3 The restricted palette

| Token | Value | Contrast on `#FFFFFF` | Role |
|---|---|---|---|
| `--ink` | `#0A0A0A` | 19.6:1 (AAA) | Body text, headings |
| `--secondary` | `#4A4A4A` | 8.9:1 (AAA) | Secondary body copy |
| `--muted` | `#6B6B6B` | 5.3:1 (AA) | Labels, captions |
| `--grey-mid` | `#8A8A8A` | 3.4:1 | **Non-text only** |
| `--rule` | `#E4E4E4` | — | Hairline rules |
| `--accent` | `#E63329` | 4.3:1 | Display type and marks only |
| `--accent-deep` | `#C8261D` | 5.6:1 (AA) | Small red text, links, focus rings |

Two deliberate departures from the original specification, both for contrast:

**`#8A8A8A` is not used for label text.** At 3.4:1 it fails WCAG 2.1 SC 1.4.3,
which requires 4.5:1 for text below 18.66px bold / 24px regular — and the labels
in question are 11px. It is retained only for non-text marks (the unselected
carousel dots, icon strokes, the middot separators), where SC 1.4.11 sets the
threshold at 3:1 and it passes. Label text uses `#6B6B6B` at 5.3:1.

**Red is split into two tokens.** `#E63329` measures 4.3:1 — enough for large
text (SC 1.4.3 permits 3:1 at ≥24px) and for non-text UI, but not for 11px link
text. So `#E63329` is used for the accent keyline, the active carousel dot, the
active nav underline and the accordion marker, while small red text and every
focus ring uses `#C8261D` at 5.6:1.

Red appears in exactly five places: the active nav marker, the active carousel
dot, the accordion open/close marker, the accent keyline that opens each
interior page, and link hover states. Nothing else.

### 1.4 Mobile-first reasoning

`main.css` is written for small screens with no media query wrapper;
`responsive.css` adds capability upward at 480 / 768 / 1024 / 1440. Nothing in
the base layer is undone by a later layer — the breakpoints only add.

This ordering is not stylistic. A desktop-first sheet forces every small-screen
rule to be an override, so the mobile experience is expressed as a list of
corrections and each new desktop rule risks leaking downward. Building upward
means the narrowest, most constrained case is the one that is correct by
default.

Concrete consequences:

- The hero source order is photo → headline → content, which is the required
  small-screen order. Desktop repositions with explicit grid placement, so no
  `order` property and no source-order/visual-order mismatch for screen readers.
- The rotated `SOFTWARE` word is `display: none` below 768px. A vertical word in
  a 320px column has no gutter to occupy.
- The checkbox menu and its label are `display: none` from 768px up, which also
  removes them from the tab order. Leaving a focusable control that does nothing
  is a real defect, so the decorative three-line mark in the hero's right column
  is a separate `aria-hidden` graphic.
- Specification tables stay two-column at every width. Both cells carry
  `min-width: 0` and `overflow-wrap: break-word`, without which a flex item
  refuses to shrink below its content and a long label beside a long value
  overflows the viewport. This was an actual defect at 320px on the education
  page, caught by measuring `documentElement.scrollWidth` against
  `window.innerWidth` on all eight pages at all three target widths.

Verified with no horizontal overflow at 320px, 768px and 1440px on all eight
pages.

### 1.5 Accessibility decisions

- **Landmarks.** Every page uses `<header>`, `<nav aria-label="Main">`,
  `<main id="main">` and `<footer>`, with `<article>` and `<section>` where the
  content is genuinely self-contained. Each `<section>` is named by
  `aria-labelledby` pointing at its own heading.
- **Skip link.** First focusable element on every page; visible on focus.
- **Focus visibility.** `:focus-visible` draws a 2px `#C8261D` ring with a 2px
  offset on every interactive element. The mobile menu's ring is drawn on the
  `<label>` via `.nav-toggle:focus-visible + .nav-burger`, because the input
  itself is visually hidden.
- **Current page.** Marked with `aria-current="page"` and, visually, with an
  accent underline **plus** a bold weight. Colour is never the only signal.
- **The display headline is not read as fragments.** `ENGI` and `NEER` are
  `aria-hidden`; a visually hidden sibling carries the real string, so assistive
  technology hears "Jonathan E. Abogye, software engineer" rather than two
  nonsense syllables.
- **Icons** are `aria-hidden="true" focusable="false"`; their meaning is always
  carried by adjacent text.
- **Reduced motion.** `prefers-reduced-motion: reduce` stops the rotating badge
  and every transition.
- **Form.** Every control has a real `<label for>`; validation messaging is tied
  to the field and only revealed once the control has content, so an untouched
  form is never painted as an error.

---

## 2. JSON data structure definitions

### 2.1 Why the JSON exists when it cannot be fetched

This is the central tension of the brief and it deserves a direct answer rather
than a workaround.

Rendering `data/projects.json` into `projects.html` at run time requires
`fetch()` and DOM construction — both prohibited. So the JSON is **not** loaded
by the site. Nothing on any page reads it. Every page states this in an HTML
comment at the top of the file.

What the JSON is, instead, is the **canonical content schema**: the normative
definition of the entities the site is made of, their fields, types and
relationships. The HTML is a hand-rendered projection of it. In a build that
permitted scripting — or any server-side or build-time template — these files
would be the data source and the HTML would be generated from them; the markup
was written to make that substitution mechanical, with one repeating block per
entity and no content living only in the markup.

The honest consequence is that HTML and JSON can drift, because nothing enforces
their agreement. That is a real cost of the constraint, not something to paper
over. Two entities are already annotated for it: `image_url` is `null` on every
project because the design carries exactly one photograph, and the LinkedIn
handle and email address are marked as placeholders in both representations.

### 2.2 `profile.json`

Single root entity — the site owner. One object, because there is exactly one.

| Field | Type | Purpose |
|---|---|---|
| `id` | string | Stable slug key |
| `name` / `given_name` / `family_name` | string | Split for JSON-LD `givenName` / `familyName` |
| `title` | string | Professional title, used in `<title>` and JSON-LD `jobTitle` |
| `headline` | string | Short form for the hero |
| `short_bio` | string | ~2 sentences, for meta description and the CV profile |
| `long_bio` | string | Full paragraph for the About page |
| `status` | string | Current position in the degree |
| `availability` | string | Hiring availability, kept separate from `status` so it can change independently |
| `location` | object | `city`, `state`, `country`, `country_code` — split rather than a single string so it maps onto `PostalAddress` without parsing |
| `contact` | object | `email`, `email_note`, `preferred_channel` |
| `social[]` | array of object | `id`, `label`, `handle`, `url`, optional `note` |
| `knows_about[]` | array of string | Feeds JSON-LD `knowsAbout` |

`social` is an array rather than named keys (`github`, `linkedin`) so that
adding a network is data, not a schema change, and so it maps directly onto
`sameAs`.

### 2.3 `projects.json`

| Field | Type | Purpose |
|---|---|---|
| `id` | string | Slug; doubles as the HTML anchor (`projects.html#droneaid`) |
| `title` | string | Display name |
| `date` | string | Year or year range. Deliberately a string, not a date — `"2025 — 2026"` is not a point in time |
| `category` | enum | `web` \| `mobile` \| `engineering`; drives the filter |
| `featured` | boolean | Selects the three homepage carousel entries |
| `description` | string | Full prose |
| `summary` | string | One line, for cards and the carousel |
| `image_url` | string \| null | Null throughout; the slot a thumbnail would occupy |
| `stack[]` | array of string | Renders as tags; an array so it can be filtered on later |
| `role` | string | Contribution, kept distinct from `title` |
| `status` | string | `Complete` / `Delivered` / `Live` |
| `url` | string | `"#"` where no public link exists yet |

`categories[]` is declared at the top level rather than inferred from the
projects, so the filter can show a category with no members and the labels have
one definition.

### 2.4 `skills.json`

Two levels: `groups[]`, each with `skills[]`.

| Field | Type | Purpose |
|---|---|---|
| `groups[].id` | string | Slug |
| `groups[].label` | string | Heading |
| `groups[].competency_note` | string | Prose statement of actual working depth |
| `groups[].skills[].name` | string | Technology |
| `groups[].skills[].context` | string | What is actually done with it |

There is no numeric proficiency field, and that is a design decision, not an
omission. A percentage against a skill has no defined unit, no scale anchor and
no way to be verified — "React 85%" is unfalsifiable. A sentence saying what was
built with it is checkable against the projects page. `context` per skill does
the same job at finer grain.

### 2.5 `education.json`

| Field | Type | Purpose |
|---|---|---|
| `id` | string | Slug |
| `institution`, `institution_location` | string | Name and place |
| `qualification` | string | Award |
| `level` | string | `Undergraduate degree` / `Secondary` |
| `start_date`, `end_date` | string | Year strings |
| `expected_graduation` | string \| null | `YYYY-MM`, null once awarded |
| `status` | string | Human-readable state |
| `coursework[]` | array of string | Relevant modules |
| `highlights[]` | array of string | Notable outcomes |
| `is_placeholder` | boolean | **Marks incomplete records** |
| `placeholder_note` | string | What is missing and who supplies it |

`is_placeholder` is a schema-level flag rather than a comment because it is a
property of the record. Any future renderer can find every incomplete entry with
a filter instead of a text search.

---

## 3. HTTP/HTTPS and MIME types

### 3.1 The request/response cycle for one page load

Requesting `https://example.com/projects.html`:

1. **DNS** resolves `example.com` to an IP address.
2. **TCP** connection on port 443, then the **TLS handshake**: the server
   presents its certificate, the client validates it against a trusted CA,
   and both sides derive session keys. Everything after this point — headers,
   URLs, cookies, body — is encrypted. HTTP alone sends all of it in clear
   text, so any intermediary can read and alter it.
3. **Request:**
   ```
   GET /projects.html HTTP/1.1
   Host: example.com
   Accept: text/html,application/xhtml+xml,*/*;q=0.8
   Accept-Encoding: gzip, br
   ```
4. **Response:**
   ```
   HTTP/1.1 200 OK
   Content-Type: text/html; charset=utf-8
   Content-Length: 18342
   Cache-Control: public, max-age=0, must-revalidate
   ETag: "a1b2c3"
   ```
5. The browser reads `Content-Type`, selects the HTML parser, and builds the
   DOM incrementally as bytes arrive.
6. Encountering `<link rel="stylesheet">`, it issues **subresource requests**
   for the four stylesheets. CSS is render-blocking: the browser will not paint
   until it has them, because painting first would flash unstyled content.
7. `@font-face` rules are parsed but fonts are only fetched once a rule is
   matched by a rendered element. `font-display: swap` means text paints
   immediately in the fallback and re-renders when Orbitron arrives.
8. `<img src="assets/images/jonathan.jpg">` is fetched in parallel. Its `width`
   and `height` attributes let the browser reserve the correct box before the
   bytes land, avoiding layout shift.

Relevant status codes: **200** OK; **304** Not Modified, returned when the
browser revalidates with `If-None-Match` and the `ETag` matches — no body is
sent; **404** for the missing `assets/cv/jonathan-abogye-cv.pdf` until it is
added; **301/308** for permanent redirects, which is what a clean-URL rewrite
from `/projects` to `/projects.html` would use.

### 3.2 How static assets are served

Every request maps to a file on disk. There is no application layer: no
interpreter, no database, no session. The server resolves the path, reads the
file, sets `Content-Type` from the file extension, and streams the bytes. This
is why a static site is fast and why its attack surface is so small — there is
no code path an attacker can reach.

### 3.3 MIME types in this project

| Extension | MIME type | Consequence of the correct type |
|---|---|---|
| `.html` | `text/html; charset=utf-8` | Parsed as HTML; the charset makes `—` and `·` decode correctly |
| `.css` | `text/css` | Accepted as a stylesheet |
| `.json` | `application/json` | Served as data |
| `.svg` | `image/svg+xml` | Rendered as a vector, scaled by CSS |
| `.woff2` | `font/woff2` | Accepted by the font loader |
| `.jpg` | `image/jpeg` | Decoded as a raster image |

### 3.4 What breaks when `Content-Type` is wrong

The type header is authoritative. Browsers do not, in general, correct it from
the file extension or the bytes.

- **CSS served as `text/plain`.** In standards mode a stylesheet whose
  `Content-Type` is not `text/css` is refused outright. The page renders
  completely unstyled. This is the single most common static-hosting failure.
- **A font served as `application/octet-stream`.** Usually still loads, since
  the font loader sniffs the container — but strict configurations reject it and
  the display face silently falls back.
- **SVG served as `text/plain` or `image/jpeg`.** Refuses to decode; the `<img>`
  shows its broken-image state and the `alt` text.
- **JSON served as `text/html`.** Rendered as markup rather than offered as
  data, and — the reason it matters — any HTML inside the JSON becomes live
  markup in the response's origin. This is the classic MIME-confusion XSS
  vector, and why `X-Content-Type-Options: nosniff` exists: it tells the browser
  never to second-guess the declared type.
- **HTML served as `text/plain`.** The source is displayed as text.

This is precisely why the portrait placeholder used during development was named
`.svg` rather than being written as SVG under a `.jpg` extension. The extension
determines the header the host sends; a mismatched pair produces a file that
cannot be decoded by anything.

Vercel sets all of the above correctly from file extensions with no
configuration, which is why this project ships no `vercel.json`.

---

## 4. CMS Selection Justification

This portfolio is hand-written HTML, CSS and JSON with no content management
system. The alternative would be WordPress, the default answer for a personal
site. The choice turns on four dimensions.

**Performance and hosting cost.** The whole site is under 250 KB and every
request is a static file read, so time-to-first-byte is bounded by network
latency alone. A WordPress page load boots PHP, opens a MySQL connection, runs a
dozen or more queries and assembles HTML on every request — work usually undone
by a caching plugin whose purpose is to make WordPress behave like a static
site. Cost follows: this deploys free on Vercel's static CDN tier, while
WordPress needs a PHP host with a database, realistically $5-15 a month.

**Security surface.** A static site has no server-side code, no database and no
authentication. There is no login to brute-force, no SQL to inject and no upload
handler to abuse; compromise requires taking the hosting account or the Git
repository. WordPress core is well audited, but plugin vulnerabilities account
for the large majority of compromises, and every plugin is third-party code with
database access running on every request. Security becomes an ongoing patching
obligation rather than a property of the architecture.

**Version control and workflow.** Every byte here is a text file in Git:
diffable, reviewable, revertible, with the history as the audit log. WordPress
splits state between filesystem and database — content, settings and widget
configuration live in MySQL, which Git cannot meaningfully version. Staging then
requires database synchronisation tooling. For a developer's portfolio the
workflow is itself part of the demonstration.

**Maintenance burden.** With no dependencies there is no dependency rot, and
HTML and CSS are backward-compatible by specification, so this renders
identically in five years with nothing to patch. The cost is paid on the other
side: adding a project means hand-editing `projects.html` and `projects.json`
and keeping them consistent — tolerable at six projects, unpleasant at sixty.
WordPress inverts the trade: trivial editing, permanent update obligations
across core, themes and plugins.

**When migration would be justified.** Three conditions, any one of which
changes the answer:

1. **A non-technical editor.** The moment someone who does not use Git needs to
   publish, hand-edited HTML stops being viable. This is the decisive one.
2. **Updates more often than weekly.** At six projects revised a few times a
   year, hand-editing is cheaper than the maintenance a CMS demands; a blog
   publishing several times a week inverts that arithmetic.
3. **More than one or two contributors**, needing drafts, review and scheduled
   publishing — genuine editorial workflow.

For a single technical author maintaining eight largely static pages, none of these conditions holds,
and a CMS would add cost, attack surface and maintenance in exchange for editing
convenience that is not needed. Were only condition 2 met, the right first step
would be a static site generator such as Astro or Eleventy, rendering these same
JSON files at build time and keeping the static output — the migration path
`data/*.json` is already shaped for.

**Word count: 497.**

---

## 5. Known gaps

Items requiring action before submission:

1. `hello@jonathanabogye.dev` is a placeholder address, marked in every file.
2. The LinkedIn handle is unconfirmed, marked in every file.
3. The secondary-school entry is a placeholder in `education.html`, `cv.html`
   and `education.json` (`is_placeholder: true`).
4. All four entries on `interests.html` are placeholder copy, marked
   `PLACEHOLDER INTEREST` in the source and flagged visibly on the page.
5. Project links are `#` anchors pending public URLs.
6. The contact form posts via `mailto:`, which depends on the visitor having a
   configured mail client and offers no delivery confirmation. Production would
   post to a server-side endpoint — a serverless function accepting JSON,
   validating the payload, rate-limiting by IP, and handing off to a
   transactional mail API — with the HTML validation attributes retained as the
   first, non-authoritative check.
