# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

Static, no-build marketing/product website (Vietnamese, `lang="vi"`) for **Woodsoft's BAZIS** furniture-manufacturing software suite. It is a set of self-contained HTML files with all CSS and JS inlined per file — no bundler, no package manager, no test framework, no `package.json`. Pages are meant to work both hosted (e.g. GitHub Pages) and opened directly via `file://`.

There are no build/lint/test commands — this repo is plain HTML/CSS/JS. To preview, open an `.html` file in a browser directly, or serve the folder with any static file server.

## Site structure

- **`index.html`** — the home page: an interactive flow diagram ("Luồng dữ liệu sản xuất Woodsoft") showing how the BAZIS modules connect. Built from a `DATA` object (per-node title/description/feature list/extra HTML) and a `VISUALS` object (inline SVG illustrations) in its `<script>`. Nodes with `data-href` navigate to another page on click; nodes without it open a detail modal populated from `DATA`.
- **`Bazis_dunghinh.html`** — "BAZIS – Dựng hình" (3D design module), the most detailed marketing page. Has a showcase gallery (click a feature in the side list to swap the hero image) and a unified image lightbox, both driven by `data-title`/`data-desc`/`data-image`/`data-lightbox-*` attributes on DOM elements rather than a JS data object.
- **`du-toan.html`** — "BAZIS – Dự toán" (costing/estimation module). Has a feature grid with popup "example" panels (positioned relative to the clicked card), a YouTube embed that falls back to a plain link when loaded from `file://` (YouTube requires an HTTP referer), and an embedded-PDF viewer modal.
- **`cnc.html`** — "BAZIS – CNC". Built out in the same "hero + `estimate-detail` card stack" visual system as `du-toan.html` (see below). Its standout section is a `<details>/<summary>` accordion listing ~55 compatible CNC machine brands (scraped from bazis-center.ru), each expanding to show a logo + a machine-type/control-system/file-format table.
- **`nesting.html`** — "Woodsoft Nesting". Same `estimate-detail` visual system, content sourced from the separate Woodsoft Optimizers landing page (a different GitHub Pages site — see git history/commit messages for the URL). Resources (video, sample ZIP, a cutting-layout preview, a label-sheet preview, and 4 sheets of real Excel data) are consolidated into one card with a shared tab bar (`.xlsx-tab-btn` / `.xlsx-panel`, plain vanilla-JS `classList` toggling — no framework, reused for every tab regardless of content type). Two tabs use a generalized `.image-lightbox` (any `[data-full-img]` element opens its `data-full-img` src full-size on click — `openImageLightbox(src, alt)` in the page's `<script>`): the cutting-layout tab shows a cropped "highlight" (`images/nesting/vidu-preview.png`) that opens the full ~6000px `vidu-full.png` (all 52 real cut sheets); the label tab shows a `.label-grid` of 24 real label thumbnails (`images/nesting/labels-sheet8-thumb/`) that each open their own full-res version (`labels-sheet8-full/`) — these were rendered page-by-page from the sample ZIP's `label_MDF_17.5_cot_tho_NOTEXT.pdf` with PyMuPDF (`fitz`), filtered to the pages whose "SĐC:" (sheet number) field equals 8, matching sheet #8 in `vidu-full.png`. The "Chi tiết gia công" data tab uses `.xlsx-table-wrap.scroll-y` (`max-height` + sticky `<th>`) to let a wide/tall real table (19 columns × 20 rows) scroll in both directions inside a fixed-height box instead of growing the page.
- **`analyzer.html`, `packing.html`, `panelsaw.html`, `wood-viewer.html`** — still placeholder "coming soon" pages, copy-pasted from one identical template (only the `<title>`, hero heading/subtext, and feature list differ). Their content sections still say "Nội dung sẽ được cập nhật sớm..." awaiting real content. `cnc.html` and `nesting.html` used to be part of this group but have since been built out — don't assume the remaining four are representative of the finished pages' structure.
- **`images/`** — `logo.png`, `phukien.png`, referenced by `<img src="images/...">` in several pages. `images/cnc-logos/` holds ~54 CNC machine-brand logos (sourced from bazis-center.ru) used by the compatibility accordion on `cnc.html`. `images/nesting/` holds real assets pulled from other Woodsoft properties for `nesting.html`: a nesting-layout screenshot, 6 feature-illustration JPGs, ~6 partner/step logos, and a combined "connected CNC machines" logo-grid PNG — all extracted from base64 data URIs embedded in those source pages (see `optimizers_landing.html` / `woodsoft_landing.html` scraping approach in conversation history if re-scraping is ever needed). All of these are kept as separate small files rather than inlined as base64, to avoid repeating the bloat problem described below.
- **`downloads/`** — sample files linked from the pages. `BAZIS_BAN_VE_MAU.pdf` / `BAZIS_DU_TOAN_MAU.pdf` are also embedded as base64 directly inside `du-toan.html` (see below). `CongtrinhVPT_CNC.zip` (~6MB, real Woodsoft Nesting output from an actual project — 52 sheets/608 parts of NC/DXF/label-PDF/Excel/JSON/stats files) is *not* embedded — `nesting.html` just links to it directly (`<a href="downloads/CongtrinhVPT_CNC.zip" download>`), which is the preferred approach for anything bigger than a couple hundred KB.

## Repo-wide conventions (duplicated per file, not shared)

Every page independently repeats the same patterns rather than pulling from a shared stylesheet/script:
- Same blue-gradient CSS design system (`--primary:#0f6fb8`, `--primary-dark:#0b4f83`, etc.) redefined in each file's `<style>`.
- Font Awesome via the cdnjs CDN and Google Fonts "Inter".
- A fixed top-right "HOME" button linking back to `index.html`.
- An identical `.woodsoft-footer` block (copyright + Facebook/YouTube/phone/email social links).

**Because there is no shared CSS/JS include, any sitewide change (footer, color theme, header) must be hand-applied to every HTML file individually** — check all 9 pages, not just the one you're editing. The 6 placeholder pages in particular are meant to stay in sync with each other structurally (same template) until they get real content.

## Large embedded binary data — read carefully

Some files are huge because images/PDFs are embedded inline as base64 data URIs / JS string literals:
- **`Bazis_dunghinh.html` is ~27 MB** (only ~2000 lines, but several lines are individually megabytes long — embedded base64 images for the showcase/lightbox galleries).
- **`du-toan.html` is ~1.2 MB**, dominated by two giant `base64:` string literals (`PDF_FILES.drawing`, `PDF_FILES.estimate` in the bottom `<script>`) holding the two sample PDFs, plus `IMG_3D`/`IMG_TECH` base64 image constants.
- **`index.html` is ~450 KB**, with a couple of long lines holding a base64 logo/photo.

Do not try to read these files whole — a plain read will blow the context window or hit size limits. Prefer targeted `grep`/`rg` for the surrounding logic, or strip long lines first (e.g. `awk 'length($0) < 2000' file.html > clean.html`) before reading, and treat the base64 payloads themselves as opaque — never try to view or hand-edit their contents.

## Rules — always follow

- **Content is always in Vietnamese.** All visible text (headings, copy, labels, button text, alt text) must be written in Vietnamese, matching the existing tone of the site.
- **Never change the color theme on your own initiative.** The blue-gradient palette (`--primary:#0f6fb8`, `--primary-dark:#0b4f83`, etc.) is fixed. Only change colors/theme values if the user explicitly asks for it.
- **Footer/header changes must be applied to all 9 HTML pages.** Since there's no shared include, editing `.woodsoft-footer`, the top-bar/logo, or the "HOME" button in one file means replicating the exact same change across `index.html`, `Bazis_dunghinh.html`, `du-toan.html`, `analyzer.html`, `cnc.html`, `nesting.html`, `packing.html`, `panelsaw.html`, and `wood-viewer.html` — don't leave any page out of sync.
- **Never commit unless explicitly asked.** Making edits to files is fine; running `git commit` (or `git push`) is not, until the user asks for it in that turn.

## Editing patterns to know

- **`index.html`**: to add/edit a module node, update the corresponding entry in the `DATA` object (title/desc/list/extraHTML) and, if it should open a modal instead of navigating, omit `data-href` on its `.node`/`.chip` element and rely on `openModal()`. New illustrations go in the `VISUALS` object as inline SVG.
- **`du-toan.html`**: PDFs are decoded client-side in `pdfBlob()`/`pdfUrl()` via `atob()` + `Blob` and shown in an `<iframe>` modal (`pdfModal`) or downloaded via a generated `<a download>` link. The YouTube embed only renders as an `<iframe>` when `window.location.protocol !== 'file:'`.
- **`Bazis_dunghinh.html`**: the feature showcase and lightbox both read gallery items from DOM `data-*` attributes (see `getGalleryItems()`), not from a central JS array — new gallery entries should follow the same attribute pattern on the relevant `.showcase-item`/`figure` elements.
- **`cnc.html` / `nesting.html`**: content is plain hand-written HTML inside `.estimate-detail-card` blocks (hero, overview, feature list, workflow, benefits, value strip) — there's no JS data object to edit like `index.html`; add/edit a fact by editing the markup directly. Both pull real content from external sources (bazis-center.ru, a separate Woodsoft landing page) rather than invented copy — if extending them further, prefer sourcing real data the same way over fabricating specs.
