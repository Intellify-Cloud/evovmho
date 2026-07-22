# Kilo Jekyll Refactor Plan

## Goal
Fix critical Jekyll site defects and modernize the codebase with zero visual redesign. Preserve current appearance and behavior throughout.

## Guiding Principles
- Build must pass after each step.
- Refactor SCSS incrementally while preserving the current visual appearance.
- Establish design tokens before broad changes.
- Extract repeated HTML/Liquid into reusable includes and data-driven templates.
- Simplify selectors, responsive rules, calculator iframe styles, and repeated utility patterns.
- Review accessibility: headings, landmarks, alt text, buttons, forms, focus states, contrast.
- Check performance: unused CSS, duplicate generated CSS, image loading, font loading, DOM weight.
- Finish with a concise report of what changed, what duplication was removed, and any remaining risks.

---

## Phase 1: Critical Fixes (P0)
Build must pass after each step.

### 1.1 Fix Liquid syntax error in nav
- **File:** `_includes/navheader.html:24`
- **Change:** Replace `else if` with `elsif`
- **Risk:** Low. Homepage navigation will stop throwing Liquid errors.

### 1.2 Wire homepage includes
- **File:** `_layouts/home.html`
- **Change:** Add missing includes in order: `calculators2.html`, `textblock.html`, `textblock-lite.html`, `clients.html`, `team.html`, `about.html`, `portfolio_grid.html`, `referral.html`, `download.html`
- **Risk:** Medium. Requires spot-check that sections render in correct visual order and spacing remains acceptable.

### 1.3 Add `site.url` to config
- **File:** `_config.yml`
- **Change:** Add `url: https://vmhomeloans.co.za` (or current production URL)
- **Risk:** Low. Fixes broken `site.url` references in legal pages.

### 1.4 Create `/terms` page
- **File:** `terms.md` (new, root)
- **Front matter:** `layout: page`, `title: Terms of Use`, `background: grey`
- **Content:** Basic terms placeholder or link to legal page
- **Risk:** Low. Fixes 404 from footer links.

### 1.5 Move `jeyll-buid.md` out of publishable root
- **Option A:** Move to `_drafts/jeyll-buid.md`
- **Option B:** Add to `exclude` in `_config.yml`
- **Recommend:** Add to exclude list to avoid moving files unnecessarily.
- **Change:** `_config.yml` → add `jeyll-buid.md` to `exclude` array
- **Risk:** Low.

---

## Phase 2: Layout Consolidation (P1)

### 2.1 Consolidate `page.html` and `post.html`
- **Current:** Both include `nav.html`, both handle `background: grey`, both wrap in container.
- **Approach:** Keep `page.html` as the canonical inner-page layout. Make `post.html` extend `page.html` with only the post-specific `<h1>` header.
- **Change:** `post.html` front matter → `layout: page`, remove duplicate nav/background/container logic, add `{{ page.title }}` wrapper before `{{ content }}`.
- **Risk:** Medium. Check that draft posts with `layout: post` still render correctly.

### 2.2 Consolidate navigation includes
- **Current:** `nav.html` (inner pages) and `navheader.html` (homepage) duplicate structure with slight variants.
- **Approach:** Replace both with single `nav.html` that accepts a front matter variable `nav_variant: "light" | "dark"`, or detect via `page.layout == "home"`.
- **Change:** Merge `navheader.html` logic into `nav.html`, update `home.html` and `page.html`/`post.html` to use same include.
- **Fix Liquid:** Change `else if` → `elsif` (already covered in 1.1, verify in merged file).
- **Risk:** Medium. Requires testing nav links on both homepage and inner pages.

### 2.3 Fix `textblock.html` duplicated heading
- **File:** `_includes/textblock.html`
- **Change:** Remove the first duplicated heading block (lines 5-17), keep the second correct block.
- **Risk:** Low.

---

## Phase 3: Calculator DRY (P1)

### 3.1 Create calculator layout
- **New file:** `_layouts/calculator.html`
- **Extends:** `page`
- **Content:** Single reusable iframe wrapper that reads `page.calculator_src`, `page.calculator_title`, `page.calculator_class` from front matter.
- **Fallback:** If no front matter, output nothing (graceful).

### 3.2 Migrate calculator pages to new layout
- **Files:** `bond-calculator.md`, `affordability-calculator.md`, `transfer-cost-calculator.md`, `deposit-savings-calculator.md`, `extra-repayment-calculator.md`, `amortisation-calculator.md`, `additional-payment-calculator.md`
- **Change:** Update front matter to `layout: calculator`, add `calculator_src: <current iframe src>`, `calculator_title: <current title>`, `calculator_class: bond-calc|transfer-calc|...`, remove inline iframe HTML.
- **Risk:** Low. Pure markup consolidation.

---

## Phase 4: Partnership Page Inline Content (P1)

### 4.1 Replace `partnership.md` inline content with include
- **File:** `partnership.md`
- **Current:** Entire page content is hardcoded inline, ignoring `partnership.html` include.
- **Change:** Strip inline HTML, replace with `{% include partnership.html %}`.
- **Follow-up:** Populate `sitetext.yml → partnership` and drive the include from data (see Phase 10).
- **Risk:** Low.

---

## Phase 5: Contact Page Consistency (P2)

### 5.1 Fix `contact.md` email link
- **File:** `contact.md:13-14`
- **Change:** Fix broken line-break in mailto link.

### 5.2 Create separate enquiry page for contact form
- **File:** `enquire.md` (new, root)
- **Front matter:** `layout: page`, `title: Enquire Now`, `background: grey`
- **Content:** `{% include contact.html %}`
- **Change:** Update CTA button in `textblock-lite.html` to point to `/enquire` instead of `contact`.
- **Risk:** Low. Keeps `contact.md` as lightweight info page.

---

## Phase 6: Footer and Navigation Links (P2)

### 6.1 Audit footer quick links
- **File:** `_includes/footer.html`
- **Current:** Links to `#services`, `#calculators`, `#team`, `#about`
- **Issue:** These anchor links only work on the homepage. On inner pages they resolve to `currentpage#section` which is broken.
- **Fix:** Change to absolute paths: `/`, `/contact`, etc., or keep as relative anchors and accept homepage-only behavior.

### 6.2 Make footer links data-driven
- **File:** `_includes/footer.html` + `_data/navigation.yml`
- **Change:** Move footer quick links and company links into `_data/navigation.yml` or a new `_data/footer.yml`.
- **Risk:** Low.

---

## Phase 7: Head and Meta Cleanup (P2)

### 7.1 Fix canonical URL
- **File:** `_includes/head.html:40`
- **Current:** `prepend: site.baseurl | prepend: site.baseurl` — double prepend bug.
- **Change:** Replace with `{{ page.url | replace:'index.html','' | prepend: site.baseurl | prepend: site.url }}` or simpler `{{ site.url }}{{ page.url | replace:'index.html','' }}`.

### 7.2 Fix or remove malformed baseurl JavaScript
- **File:** `_includes/head.html:52-54`
- **Current:** Broken JS split across lines.
- **Change:** Either fix to `var baseurl = "{{ site.baseurl | relative_url }}";` or remove entirely if unused.

### 7.3 Add page description fallback
- **File:** `_includes/head.html`
- **Change:** Use `{{ page.description | default: site.description | default: site.title }}` for meta description.

---

## Phase 8: Portfolio and Dead Code (P3)

### 8.1 Portfolio content cleanup
- **Files:** `_portfolio/project1.md`, `project2.md`, `example.md`
- **Action:** Replace placeholder/template content with real VM Homeloans portfolio items, or hide the portfolio section until content is ready.
- **Decision:** Hide the `portfolio_grid.html` include in `home.html` until real content is added. Prevents broken image references and placeholder content from rendering.

### 8.2 Remove dead includes (after confirming unused)
- Candidates: `articles.html` (blog disabled), `portfolio_grid.html` (if portfolio hidden), `textblock-lite.html` (if not needed).
- Do not remove until Phase 1 is complete and homepage is fully wired.

---

## Phase 9: Establish Design Tokens (P1)

### 9.1 Audit existing token-like values
- Inventory all current hardcoded values across SCSS and templates:
  - **Colors:** primary teals, font colors, button colors, background colors, WhatsApp green, error/success reds/greens
  - **Typography:** font families, font sizes, font weights, line heights
  - **Spacing:** section padding, card padding, gaps, margins
  - **Shadows:** box-shadow values used across cards, nav, buttons, modals
  - **Radius:** border-radius values for cards, buttons, inputs, modals
  - **Breakpoints:** 576px, 640px, 768px, 992px, 1024px, 1400px
  - **Z-index:** nav (1030), modal (1050-1052), WhatsApp float (1000)
  - **Transitions:** timing functions, durations used across hover/focus states

### 9.2 Normalize variables in `_assets/base/_variables.scss`
- **Goal:** Centralize all repeated values into variables without changing visual output.
- **Tasks:**
  1. Add missing color variables (WhatsApp green `#25D366`, hover `#20b858`, error `#d32f2f`, success `#2e7d32`, focus ring alpha)
  2. Add spacing variables for common hardcoded values (`3.5rem`, `4.5rem`, `960px`, `1200px`, `1240px`)
  3. Add z-index tokens (`z-nav: 1030`, `z-modal: 1050`, `z-whatsapp: 1000`)
  4. Add transition tokens (`ease-out: 0.3s ease`, `ease-spring: 0.35s cubic-bezier(0.34, 1.56, 0.64, 1)`)
  5. Keep existing variable names where changing names would create risk
- **Risk:** Low. Pure variable extraction with zero output change.

### 9.3 Add breakpoint tokens to `_mixins.scss`
- **Current `respond-to` mixin tokens:** `mobile`, `tablet`, `desktop`
- **Add missing tokens to match current raw media queries:**
  - `mobile-sm` → `max-width: 639.98px` (matches current `640px` usage)
  - `tablet-lg` → `min-width: 641px` and `max-width: 1024px` (matches current usage)
  - `wide` → `min-width: 1400px` (matches current extra-large screens)
- **Risk:** Low. Enables incremental migration in later phases.

---

## Phase 10: SCSS Architecture Cleanup (P1)

### 10.1 Remove duplicated Bootstrap overrides
- **Files:** `_assets/site.scss:13-34`, `_assets/base/_variables.scss:82-104`
- **Issue:** Bootstrap variable overrides are defined in both `_variables.scss` AND repeated inline in `site.scss` before the `@import "bootstrap/scss/bootstrap"` line.
- **Fix:** Keep overrides ONLY in `_variables.scss` (they must appear before Bootstrap import). Remove the duplicate block from `site.scss` lines 13-34.
- **Risk:** Low — CSS output should be identical.

### 10.2 De-duplicate calculator iframe height styles
- **Files:** `_assets/base/_page.scss`, `_assets/site.scss:145-173`
- **Issue:** `_page.scss` defines iframe base styles + desktop/mobile/tablet heights. `site.scss` repeats the exact same iframe base + height rules at the bottom of the file.
- **Fix:** Keep the styles in `_base/_page.scss` (already imported in `site.scss` at line 10). Remove the duplicate block from `site.scss` lines 145-173.
- **Risk:** Low — `_page.scss` is already imported, so styles will still be present.

### 10.3 Fix undefined mixin usage
- **Files:** `_assets/layout/_contact.scss:217`, `_assets/layout/_partnership.scss:381`
- **Issue:** Both use `@include respond-to(mobile-only)` but `_mixins.scss` only defines `mobile`, `tablet`, `desktop`. `mobile-only` is undefined and will throw a Sass error on build if that branch is reached.
- **Fix:** Replace `mobile-only` with `mobile` in both files.
- **Risk:** Low — behavior change is minimal (mobile breakpoint range may shift slightly).

### 10.4 Clean up legacy styles inside `_textblock.scss`
- **File:** `_assets/layout/_textblock.scss`
- **Issue:** The file has two distinct blocks:
  - Lines 1-123: Legacy hardcoded CSS (uses `#ffffff`, `#6c757d`, `#01acc8`, no design tokens).
  - Lines 125-235: Newer SCSS using tokens and mixins.
- **Fix:** Remove the legacy block (lines 1-123). Keep only the tokenized block. The legacy block targets `.textblock-heading`, `.textblock-image`, `.textblock-title` which do not appear in current includes (`textblock.html` uses `.textblock-item`, `.textblock-icon`, etc.). Verify no markup references the legacy classes.
- **Risk:** Low — visual check of textblock section after removal.

### 10.5 Remove dead/unused SCSS partials
- **Candidates:**
  - `_assets/layout/_contact_0.scss` — Legacy contact styles. NOT imported in `site.scss`. Uses undefined `$gray-900`, `$white`, `@include serif-font` (also undefined).
  - `_assets/components/client-scroll.scss` — NOT imported in `site.scss`. Uses invalid ID selector `#col-md-3 col-sm-6` (space makes it an ID + tag selector, not a class selector).
  - `_assets/layout/_portfolio.scss` — NOT imported in `site.scss`. Uses undefined `$white`, `$gray-900`, `fade-out()` (deprecated Bootstrap function), `@include serif-font` (undefined).
- **Fix:** Delete all three files after confirming they are not referenced in any HTML/JS.
- **Risk:** Low — they are dead code and produce no compiled CSS.

### 10.6 Delete generated `dist/` folders from `_assets`
- **Locations:**
  - `_assets/base/dist/` — 3 files, 2 are 0 bytes
  - `_assets/components/dist/` — 1 file (153 B)
  - `_assets/layout/dist/` — 7 files including `dist/dist/_calculators.css` (double-nested folder!)
- **Issue:** Build artifacts committed to source. Not referenced by Jekyll, Webpack, or HTML.
- **Fix:** Delete all `dist/` directories under `_assets`. Run `npm run bundle` to confirm `assets/bundle.css` is regenerated cleanly.
- **Risk:** Low.

### 10.7 Normalize raw media queries to use tokens
- **Files:** `_assets/site.scss`, `_assets/base/_page.scss`, all `_assets/layout/*.scss`
- **Goal:** Replace hardcoded pixel breakpoints with the tokenized `respond-to()` mixin where safe.
- **Priority order:**
  1. `site.scss` calculator heights: Replace `640px`, `1024px`, `576px`, `992px` with `mobile-sm`, `tablet-lg`, `mobile`, `desktop` tokens.
  2. `_page.scss`: Replace with `mobile-sm`, `tablet-lg`, `mobile`.
  3. Layout files: Replace raw `@media` queries with `@include respond-to(...)`.
  4. Keep raw queries only where token granularity is insufficient.
- **Risk:** Medium — requires testing across all breakpoints.

### 10.8 Extract WhatsApp floating button into component
- **File:** `_assets/site.scss:176-250` ( floating WhatsApp styles)
- **Current:** WhatsApp float styles are inline in `site.scss` and also present in `_includes/footer.html` (inline `<style>` and JS).
- **Fix:** Move all WhatsApp styles to `_assets/components/_whatsapp.scss`. Import from `site.scss`. Remove inline `<style>` from `footer.html`.
- **Risk:** Low.

### 10.9 Update build toolchain to Dart Sass
- **Files:** `package.json`, `webpack.config.js`
- **Issue:** `package.json` uses `node-sass` (deprecated, native bindings). Webpack config uses `style-loader` + `MiniCssExtractPlugin.loader` both for SCSS, which is redundant.
- **Fix:**
  1. Replace `node-sass` with `sass` (Dart Sass) in `package.json` devDependencies.
  2. In `webpack.config.js`, remove `style-loader` from the SCSS rule. Keep only `MiniCssExtractPlugin.loader`, `css-loader`, `sass-loader`.
  3. Ensure `sass-loader` version is compatible (project uses webpack 4.x, so `sass-loader` ~v10 or v12 is needed).
- **Risk:** Medium — requires `npm install` and rebuild verification.

---

## Phase 11: HTML/Liquid Extraction (P2)

### 11.1 Drive partnership content from `sitetext.yml`
- **File:** `_includes/partnership.html`
- **Current:** Some sections are commented out; `partnership.md` reimplements content inline.
- **Fix:** Uncomment and complete the data-driven sections in `partnership.html` using `site.data.sitetext.partnership.*`. Update `partnership.md` to use `{% include partnership.html %}`.
- **Risk:** Medium. Requires validating all partnership content renders correctly.

### 11.2 Make document download data-driven
- **File:** `_includes/download.html`
- **Current:** Already data-driven via `sitetext.yml → document_download`. Verify this is complete and remove any hardcoded fallbacks.

### 11.3 Extract repeated card patterns
- **Candidates:**
  - Service cards (`_includes/services.html` + `_layout/_services.scss`)
  - Calculator cards (`_includes/calculators.html`, `calculators2.html` + `_layout/_calculators.scss`)
  - Team cards (`_includes/team.html` + `_layout/_team.scss`)
  - Client logo cards (`_includes/clients.html` + `_layout/_clients.scss`)
- **Decision:** Only extract if markup is repeated in 3+ places. Currently each is in one include + one layout partial, so extraction is optional. **Skip unless a pattern is found across multiple includes.**

### 11.4 Simplify `textblock.html` and `textblock-lite.html`
- **Current:** `textblock.html` duplicates heading logic (already fixed in 2.3). `textblock-lite.html` has hardcoded button hrefs.
- **Fix:** Make `textblock-lite.html` buttons data-driven via `sitetext.yml → textblock-lite.buttons`.

---

## Phase 12: Accessibility Review (P2)

### 12.1 Heading hierarchy
- **Audit:** Check homepage and all inner pages for proper `<h1>` → `<h2>` → `<h3>` nesting.
- **Findings to verify:**
  - `home.html` has no `<h1>` (relies on `hero.html` heading). Acceptable if `hero.html` renders an `<h1>`.
  - `page.html` has no page title heading (relies on content). Acceptable if each page provides its own `<h1>`.
  - `post.html` hardcodes `<h1>{{ page.title }}</h1>` which is correct.
- **Fix:** Ensure every page has exactly one `<h1>`.

### 12.2 Landmarks
- **Audit:** Confirm `header`, `nav`, `main`, `footer` landmarks exist on all layouts.
- **Current:** `nav` exists, `footer` exists, `header` exists in `hero.html` on homepage only. Inner pages (`page.html`, `post.html`) lack `<header>` and `<main>` landmarks.
- **Fix:** Add `<main>` wrapper in `page.html` and `post.html` around `{{ content }}`. Add `<header>` if needed on inner pages.

### 12.3 Alt text
- **Audit:** Check all `<img>` tags in includes for meaningful `alt` attributes.
- **Current:** Team images use `{{ person.name }}` — good. Client logos use `{{ client.title }}` — good. Portfolio modals use `{{ project.alt }}` — verify it's not empty.
- **Fix:** Add empty `alt=""` for decorative images only. Ensure all meaningful images have descriptive alt text.

### 12.4 Buttons and links
- **Audit:** Check for placeholder or empty link text.
- **Current:** Calculator cards use `aria-label="..."` placeholder text.
- **Fix:** Replace placeholder `aria-label="..."` with descriptive labels like `aria-label="Calculate your bond repayment"`.

### 12.5 Forms
- **Audit:** `_includes/contact.html` has labels for all inputs, required field indicators, error message containers.
- **Fix:** Add `required` attribute to required inputs. Add `aria-required="true"`. Ensure error messages are linked via `aria-describedby`.

### 12.6 Focus states
- **Audit:** `site.scss` has global focus styles (`a:focus, button:focus, input:focus...`). Verify these are not overridden by component styles.
- **Fix:** Ensure all interactive elements (buttons, links, inputs, nav toggler) have visible focus indicators. Remove any `outline: none` without a `box-shadow` replacement.

### 12.7 Color contrast
- **Audit:** Check text color vs background color contrast for:
  - `$font-color-2` (#334155) on `$bg-color-2` (#F8FAFC) — body text on light background
  - `$font-color-3` (#64748B) on white — muted text
  - White text on `$primary` (#38b6ff) — button text
  - `$font-color-1` (#0F172A) on white — headings
- **Target:** WCAG AA minimum (4.5:1 for normal text, 3:1 for large text).
- **Fix:** Adjust colors if any ratios fall below threshold.

---

## Phase 13: Performance Review (P3)

### 13.1 Unused CSS audit
- **Tool:** Use Chrome DevTools Coverage or `purgecss`-style scan.
- **Audit:**
  - Bootstrap full import brings in ~200KB of CSS. Components like `.modal`, `.carousel`, `.dropdown`, `.tooltip` are unused.
  - `.portfolio-modal` styles exist in `_portfolio.scss` (dead file) but modals are used in `modals.html`. Verify if modal styles are in `site.scss` or missing.
  - `.contact-us` class in `_contact_0.scss` (dead file) — not used.
- **Fix:** If Bootstrap is significantly oversized, consider switching to `bootstrap/scss/functions`, `variables`, `mixins`, `reboot`, `grid`, `buttons`, `navbar`, `cards`, `forms`, `utilities` cherry-pick. **Defer unless CSS size is critical.**

### 13.2 Remove duplicate generated CSS
- **Done in Phase 10.5** (delete `dist/` folders).

### 13.3 Image loading
- **Audit:**
  - Hero images (`hero.jpg`, `hero2.jpg`, `hero_o.jpg`) are ~87-134KB each. No `loading="lazy"` since they're above-the-fold.
  - Client logos have `loading="lazy"` — correct.
  - Portfolio images reference external GitHub URLs and broken local paths.
  - No WebP/AVIF modern formats.
- **Fix (optional):**
  - Compress hero images with `imagemin` or similar.
  - Add `fetchpriority="high"` to hero image if converted to `<img>`.
  - **Defer unless performance budgets are defined.**

### 13.4 Font loading
- **Audit:** Google Fonts `Comfortaa` loaded via `<link>` in `head.html`. `font-display` is not set.
- **Fix:** Add `&display=swap` to the Google Fonts URL (already present). Consider adding `rel="preconnect"` for `fonts.googleapis.com` and `fonts.gstatic.com`.
- **Risk:** Low.

### 13.5 DOM weight
- **Audit:** `footer.html` has many nested divs. Homepage will have 9+ sections after wiring.
- **Fix:** No action unless DOM nodes exceed reasonable thresholds. Current structure is typical for a marketing site.

### 13.6 Bundled JavaScript dependencies
- **Audit:** `bundle.js` imports jQuery, Popper, Bootstrap JS, Font Awesome CSS, site.js, site.scss. Total bundle size is unknown without measuring.
- **Fix:** Consider removing jQuery if Bootstrap 4 is replaced with Bootstrap 5 (no jQuery dependency). **Out of scope for this refactor.**

---

## Phase 14: Build Pipeline and Final Validation (P1)

### 14.1 Standardize build commands
- **File:** `package.json`
- **Current:** `npm run bundle` runs webpack. No script for Jekyll build.
- **Fix:** Add scripts:
  - `npm run build:css` — runs webpack only
  - `npm run build:jekyll` — runs `bundle exec jekyll build --trace --baseurl ''`
  - `npm run build` — runs both in sequence
- **Risk:** Low.

### 14.2 Validation after each phase
Run after every phase:
```powershell
npm run bundle
bundle exec jekyll build --trace --baseurl ''
```

Spot-check pages:
- `/` — all sections render in order
- `/contact` — details load correctly
- `/enquire` — form renders and submits
- `/partnership` — data-driven content appears
- `/bond-calculator`, `/affordability-calculator`, `/transfer-cost-calculator` — iframes render with correct src/title/class
- `/legal`, `/privacy-statement` — no broken `site.url` references
- `/terms` — exists and renders
- 404.html — works for unknown routes

**SCSS-specific validation:**
- Confirm `assets/bundle.css` size does not increase after deduplication (should shrink or stay flat).
- Confirm no Sass compilation warnings or errors.
- Confirm calculator iframe heights are correct on mobile/tablet/desktop.
- Confirm `.textblock-section`, `.contact-section`, `.partnership-section`, `.referral-section` render without missing styles.
- Confirm no `$undefined-variable` or `Undefined mixin` errors in build output.

**Accessibility validation:**
- Run Lighthouse or axe DevTools on homepage, contact page, calculator page.
- Check for heading hierarchy violations, missing alt text, low contrast, missing form labels.
- Keyboard navigation pass: tab through all interactive elements on each spot-check page.

**Performance validation:**
- Run Lighthouse performance audit on homepage.
- Target: Performance score > 80 (or baseline comparison).
- Check for unused CSS warnings.

---

## Phase 15: Final Report (P3)

### 15.1 Concise change summary
After all phases complete, produce a report covering:
1. **What changed:** List of files modified, added, or deleted.
2. **Duplication removed:** Specific examples of consolidated includes, deduplicated SCSS, deleted dead files.
3. **Accessibility improvements:** Heading fixes, alt text additions, contrast adjustments, focus state enhancements.
4. **Performance changes:** CSS bundle size before/after, image optimizations, font loading improvements.
5. **Remaining risks:** Any technical debt left, external dependencies (Ooba iframes), placeholder content, build toolchain version constraints.

### 15.2 Report format
- Markdown file saved to `.kilo/plans/` or project root as `REFRACTOR-REPORT.md`.
- Include a table of phases with status (completed / skipped / deferred).
- Include a "before vs after" file tree summary.

---

## Open Questions

### Q1: Partnership content source
Should `partnership.html` be populated from `sitetext.yml` (like other sections) or keep the inline structured HTML in `partnership.md` as the source of truth?

**Recommendation:** Populate `sitetext.yml → partnership` and use the include. This matches the existing site data pattern and makes the content editable without touching markdown.

### Q2: Contact page scope
Should the contact form (`_includes/contact.html`) be merged into `contact.md` as a second section, or kept as a separate page?

**Recommendation:** Keep separate. Create `enquire.md` (or similar) with `layout: page` and `{% include contact.html %}`, update the CTA button in `textblock-lite.html` to point there. Keeps `contact.md` as a lightweight info page.

### Q3: Portfolio visibility
Should the portfolio section be enabled on the homepage now, or hidden until real content exists?

**Recommendation:** Hide the `portfolio_grid.html` include in `home.html` until real content is added. Prevents broken image references and placeholder content from rendering.

### Q4: Bootstrap version upgrade
Should Bootstrap 4 be upgraded to Bootstrap 5 during this refactor?

**Recommendation:** Defer to a separate initiative. Current refactor should preserve Bootstrap 4 to avoid scope creep and visual regressions.

### Q5: Font Awesome optimization
Should the full Font Awesome package be replaced with a smaller subset or SVG icons?

**Recommendation:** Defer. Evaluate usage count first. If < 20 icons are used, consider switching to `@fortawesome/fontawesome-svg-core` + individual icon packages. If usage is heavy, keep full package.
