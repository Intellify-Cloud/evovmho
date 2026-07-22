# Codex Jekyll Refactor Plan

## Scope

This plan follows the audit prompt in `jeyll-buid.md` while preserving the current site design and behavior. The refactor should be incremental, with a build check after each major phase.

No visual redesign is intended. Any visual change should be deliberate, small, and called out.

## Current Baseline

The project is a Jekyll site with:

- Layouts in `_layouts`
- Includes in `_includes`
- Site content in `_data/sitetext.yml`
- Navigation data in `_data/navigation.yml`
- A `portfolio` collection in `_portfolio`
- SCSS and JavaScript entrypoints in `_assets`
- Compiled output in `assets/bundle.css` and `assets/bundle.js`

Important findings from the initial audits:

- `page.html` and `post.html` duplicate page shell behavior.
- `nav.html` and `navheader.html` duplicate navigation with slight variants.
- `head.html` has suspicious canonical URL and inline JavaScript issues.
- Most homepage content is already data-driven through `_data/sitetext.yml`.
- Calculator pages repeat the same iframe page structure.
- SCSS uses deprecated `@import`.
- Build uses obsolete `node-sass`.
- Webpack uses both `style-loader` and `MiniCssExtractPlugin.loader` for SCSS.
- Generated `dist` CSS exists inside `_assets`.
- Calculator iframe heights are duplicated in `site.scss` and `base/_page.scss`.
- Some SCSS files appear unused or legacy.

## Kilo Plan Cross-Check

Status: Reviewed

Source reviewed:

- `.kilo/plans/1784752568981-kilo-jekyll-refactor-plan.md`

Result:

- The core Codex refactor work was not overwritten in the key files checked: `_includes/nav.html`, `_includes/head.html`, `_assets/site.scss`, `webpack.config.js`, calculator front matter, and `_includes/calculator-iframe.html`.
- Some Kilo-style changes were applied on top of the Codex work. These should be reviewed individually rather than accepted blindly.

Kilo items already covered by the Codex implementation:

1. Add `site.url` to `_config.yml`.
2. Exclude `jeyll-buid.md` from published output.
3. Consolidate `nav.html` and `navheader.html`.
4. Remove the old Liquid `else if` issue by delegating `navheader.html` to `nav.html`.
5. Fix canonical URL generation.
6. Replace malformed baseurl JavaScript.
7. Add page description fallback in `head.html`.
8. Replace `node-sass` with Dart Sass.
9. Remove `style-loader` from the extracted SCSS pipeline.
10. Add missing `mobile-only` breakpoint support.
11. Move global styles out of `site.scss`.
12. Move calculator iframe styles into a component partial.
13. Move WhatsApp floating button styles into a component partial.
14. Centralize calculator iframe markup with `_includes/calculator-iframe.html`.
15. Delete unreferenced `_assets/**/dist` generated CSS.

Useful Kilo items to add to the Codex roadmap:

1. Create or keep a `/terms` page and make sure footer links point to a real route.
2. Keep footer links data-driven through `_data/footer.yml`, but fix homepage-only anchor links on inner pages.
3. Decide whether the extra homepage includes should remain wired in `home.html`.
4. Hide or defer `portfolio_grid.html` until portfolio content is real and asset references are verified.
5. Fix `contact.md` mailto formatting.
6. Consider an `enquire.md` page for the contact form instead of mixing contact details and form behavior.
7. Decide whether `partnership.md` should use `_includes/partnership.html` as the source of truth.
8. Add `<main>` landmarks to inner page layouts.
9. Replace placeholder `aria-label="..."` values in calculator cards.
10. Add Google Fonts preconnect links.
11. Add explicit build scripts for Jekyll and combined builds.
12. Produce a final refactor report with completed, skipped, and deferred work.

Kilo items that conflict with the Codex plan or need correction:

1. `home.html` was expanded to include `portfolio_grid.html`, but the Kilo plan also recommends hiding portfolio until real content exists. Decide one path before continuing.
2. `post.html` now has `layout: page` while still wrapping content in its own `container post` with `id="pagecontainer"`. That can produce nested duplicate page containers and should be corrected before post layout work continues.
3. `_includes/textblock.html` was simplified, but the current markup should be checked for balanced wrapper divs.
4. `_data/footer.yml` currently uses homepage-only anchors such as `#services`; these are still broken from inner pages unless changed to `/#services`.
5. The Kilo validation command uses `bundle exec jekyll build --trace --baseurl ''`; in this PowerShell environment that empty argument fails. Prefer `bundle exec jekyll build --trace` because `_config.yml` already sets `baseurl: ""`.
6. The Kilo plan says `_assets/base/_page.scss` is already imported by `site.scss`; that was false during the audit and should not be used as an implementation assumption.
7. The Kilo plan requires "no Sass warnings", but Bootstrap 4 with Dart Sass still emits deprecation warnings. The realistic target is no Sass errors, with deprecation warnings documented.

## Step 1: Jekyll Architecture Audit

Status: Completed

Purpose: understand the current Jekyll structure before planning refactors.

Files and folders reviewed:

- `_layouts/default.html`
- `_layouts/home.html`
- `_layouts/page.html`
- `_layouts/post.html`
- `_includes`
- `_data/navigation.yml`
- `_data/sitetext.yml`
- `_portfolio`
- `_config.yml`
- Main Markdown pages such as `index.md`, `contact.md`, and calculator pages

Findings:

- The site uses a clear Jekyll structure with layouts, includes, data files, and a small `portfolio` collection.
- `default.html` provides the outer document shell and footer.
- `home.html` hardcodes the homepage section order by including `navheader`, `hero`, `services`, and `calculators`.
- `page.html` and `post.html` duplicate navigation, background handling, and content container logic.
- Most secondary pages use `layout: page`.
- `index.md` is minimal and uses `layout: home`.
- Homepage content is mostly data-driven through `_data/sitetext.yml`.
- Navigation is data-driven through `_data/navigation.yml`.
- Footer links and contact details are still hardcoded in `footer.html`.
- Calculator pages repeat the same iframe page pattern.
- `nav.html` and `navheader.html` are two versions of the same core navigation pattern.
- `navheader.html` appears to use `else if`; Liquid should use `elsif`.
- `head.html` contains a likely incorrect canonical URL expression with `site.baseurl` prepended twice.
- `head.html` contains malformed-looking inline JavaScript for `baseurl`.
- `_portfolio` is configured as a collection, but public output/permalink behavior should be clarified before relying on it.

Recommended actions from Step 1:

1. Consolidate `nav.html` and `navheader.html` into one configurable include.
2. Extract shared page/post shell behavior.
3. Replace inline background scripts with layout/body classes.
4. Fix canonical URL generation.
5. Remove or repair malformed inline JavaScript in `head.html`.
6. Make footer links data-driven.
7. Convert calculator pages to a data-driven include or layout.
8. Clarify whether the portfolio collection should output public pages.

Verification for Step 1 follow-up changes:

- Build Jekyll after layout/include edits.
- Check homepage nav anchors.
- Check secondary page nav links back to homepage sections.
- Check generated page titles and canonical URLs.

## Step 2: SCSS Architecture And Build Audit

Status: Completed

Purpose: understand styling structure, build flow, duplication, and migration risks.

Files and folders reviewed:

- `_assets/site.scss`
- `_assets/base/_variables.scss`
- `_assets/base/_mixins.scss`
- `_assets/base/_page.scss`
- `_assets/components`
- `_assets/layout`
- `_assets/**/dist`
- `_assets/bundle.js`
- `_assets/site.js`
- `webpack.config.js`
- `package.json`

Findings:

- `site.scss` is the main styling entrypoint.
- `site.scss` imports Bootstrap functions, variables, mixins, and full Bootstrap using deprecated Sass `@import`.
- Bootstrap override values are mixed directly into `site.scss`.
- Global element styles, section-wide text alignment rules, calculator iframe styles, WhatsApp floating button styles, and layout imports all live in `site.scss`.
- `node-sass` is present in `package.json` and should be replaced with Dart Sass.
- Webpack SCSS config uses both `style-loader` and `MiniCssExtractPlugin.loader`; for extracted production CSS, this is redundant and potentially wrong.
- Generated `dist` CSS files exist inside `_assets/base`, `_assets/components`, and `_assets/layout`.
- Calculator iframe heights are duplicated in `_assets/site.scss` and `_assets/base/_page.scss`.
- `_assets/base/_page.scss` appears not to be imported by `site.scss`.
- `_assets/layout/_contact_0.scss` appears legacy or unused.
- `_assets/components/client-scroll.scss` appears unused and contains an invalid-looking selector: `#col-md-3 col-sm-6`.
- The `respond-to()` mixin defines `mobile`, `tablet`, and `desktop`, but project SCSS also uses `mobile-only`.
- Many hardcoded colors, sizes, shadows, and breakpoints remain outside tokens.
- Some comments contain encoding artifacts such as `â€“`.
- Some SCSS uses global Bootstrap-like selectors such as `.card`, `.card-body`, and `.card-title`, which can create broad side effects.
- Naming conventions are mixed across section, component, Bootstrap, ID, and BEM-like class names.

Recommended actions from Step 2:

1. Replace `node-sass` with Dart Sass.
2. Confirm `sass-loader` compatibility with the current Webpack version.
3. Remove `style-loader` from the extracted SCSS loader chain.
4. Keep the first SCSS reorganization low-risk by moving files before changing behavior.
5. Add the missing `mobile-only` breakpoint support.
6. Centralize calculator iframe styles in one partial.
7. Move WhatsApp floating button styles into a component partial.
8. Move Bootstrap setup and global base styles out of `site.scss`.
9. Confirm and remove stale `_assets/**/dist` CSS only after checking references.
10. Confirm whether legacy-looking SCSS files are unused before deleting.

Verification for Step 2 follow-up changes:

- Run `npm run bundle`.
- Run `bundle exec jekyll build --trace`.
- Compare `assets/bundle.css` size before and after.
- Spot-check homepage, contact page, and calculator pages.

## Phase 3: Design Tokens

Priority: High

Goal: centralize design decisions without changing the visual output.

Tasks:

1. Inventory all current token-like values.
   - Colors
   - Font families
   - Font sizes
   - Font weights
   - Spacing
   - Container widths
   - Border radii
   - Shadows
   - Z-index values
   - Transitions
   - Animation timings
   - Breakpoints

2. Normalize existing variables in `_assets/base/_variables.scss`.
   - Keep existing variable names initially where changing names would create risk.
   - Add missing variables for repeated hardcoded values.
   - Avoid broad renaming until the structure is stable.

3. Add breakpoint tokens.
   - Define `mobile`, `mobile-only`, `tablet`, `desktop`, and wide desktop behavior.
   - Update `respond-to()` so every currently used breakpoint exists.

4. Move repeated hardcoded values into tokens.
   - Calculator iframe heights
   - WhatsApp colors and dimensions
   - Common container widths such as `720px`, `900px`, `1200px`
   - Common focus rings
   - Common alpha borders

Verification:

- Run the CSS build.
- Compare generated CSS size before and after.
- Spot-check homepage, contact page, and calculator pages.

## Phase 4: SCSS Architecture

Priority: High

Goal: make the SCSS easier to navigate before deeper cleanup.

Target structure:

```text
_assets/
  abstracts/
    _tokens.scss
    _breakpoints.scss
    _mixins.scss
    _functions.scss
    _index.scss
  base/
    _reset.scss
    _typography.scss
    _page.scss
    _utilities.scss
    _index.scss
  components/
    _buttons.scss
    _cards.scss
    _navbar.scss
    _whatsapp.scss
    _calculator-iframe.scss
    _index.scss
  layout/
    _footer.scss
    _hero.scss
    _services.scss
    _calculators.scss
    _about.scss
    _team.scss
    _clients.scss
    _testimonials.scss
    _referral.scss
    _textblock.scss
    _partnership.scss
    _download.scss
    _contact.scss
    _index.scss
  site.scss
```

Implementation notes:

- First reorganize with `@import` to reduce risk.
- Move to `@use` and `@forward` only after the build toolchain is updated to Dart Sass.
- Do not split files just to split them. Keep section files when they match an include.

Tasks:

1. Move Bootstrap override variables into a dedicated bootstrap setup partial.
2. Move global element styles out of `site.scss`.
3. Move calculator iframe styles out of `site.scss`.
4. Move floating WhatsApp styles into a component partial.
5. Remove or archive unused SCSS only after confirming no import or markup dependency.
6. Delete generated `_assets/**/dist` files only after confirming they are not referenced by Jekyll, Webpack, or templates.

Verification:

- Run `npm run bundle`.
- Run Jekyll build.
- Confirm `assets/bundle.css` and `assets/bundle.js` still exist.

## Phase 5: Build Pipeline

Priority: High

Goal: modernize the styling build enough to support modern Sass safely.

Tasks:

1. Replace `node-sass` with `sass`.
2. Confirm `sass-loader` compatibility with the current Webpack version.
3. Remove `style-loader` from the SCSS extraction chain.
4. Keep `MiniCssExtractPlugin.loader` for production CSS output.
5. Add explicit build scripts if missing:
   - `npm run bundle`
   - Optional `npm run build:jekyll`
   - Optional combined build script

Verification:

- Run dependency install if needed.
- Run `npm run bundle`.
- Run Jekyll build with baseurl empty.
- Confirm no Sass deprecation or loader errors block the build.

## Phase 6: Layout And Include Reuse

Priority: High

Goal: reduce duplicated Liquid and make page assembly predictable.

Tasks:

1. Consolidate navigation.
   - Replace `nav.html` and `navheader.html` with one include that accepts a variant.
   - Support homepage anchors and internal page links.
   - Fix Liquid `else if` to `elsif`.
   - Ensure logo alt text exists.

2. Consolidate page shell behavior.
   - Extract common page layout logic from `page.html` and `post.html`.
   - Replace inline background script with body/layout classes.
   - Keep page/post markup differences explicit.

3. Clean `head.html`.
   - Fix canonical URL generation.
   - Fix or remove malformed `baseurl` JavaScript.
   - Consider page-specific description fallback.
   - Keep analytics behavior intact.

4. Make footer links data-driven.
   - Move quick links and company links to `_data/navigation.yml` or a footer data section.
   - Correct links that do not map to existing pages.
   - If `_data/footer.yml` is used, make homepage section links absolute from inner pages, for example `/#services`.

5. Preserve or create required legal routes.
   - Keep `/terms` available if the footer links to it.
   - Verify privacy/legal footer routes match actual pages.

Verification:

- Build Jekyll.
- Check homepage nav anchors.
- Check nav from secondary pages back to homepage sections.
- Check page titles and canonical links in generated HTML.

## Phase 7: Calculator Pages

Priority: Medium

Goal: remove repeated iframe page markup and centralize calculator metadata.

Tasks:

1. Create calculator data in `_data/calculators.yml` or extend `sitetext.yml`.
2. Add a reusable calculator include.
3. Add a calculator page layout or use a page include driven by front matter.
4. Centralize iframe class, src, title, and responsive height tokens.
5. Remove repeated `<br>` and wrapper markup from calculator pages.
6. Keep the existing include-based approach unless a layout adds real value; avoid churn between equivalent patterns.

Verification:

- Check each calculator page renders the correct iframe.
- Confirm responsive heights match current behavior.
- Confirm iframe titles are descriptive.

## Phase 8: Component Cleanup

Priority: Medium

Goal: align repeated components with includes and SCSS partials.

Targets:

- Buttons
- Cards
- Service cards
- Calculator cards
- Team cards
- Client logos
- CTA blocks
- Text blocks
- Partnership content blocks

Tasks:

1. Identify repeated component markup.
2. Extract includes only where repetition is real.
3. Align SCSS names with component names.
4. Reduce global `.card`, `.card-body`, and `.card-title` overrides where they risk affecting Bootstrap or unrelated sections.
5. Decide whether `partnership.md` should delegate to `_includes/partnership.html`.
6. Decide whether `textblock-lite.html` buttons should be driven by `_data/sitetext.yml`.

Verification:

- Spot-check affected sections on desktop and mobile.
- Confirm Bootstrap components still work.

## Phase 9: Responsive And Accessibility Review

Priority: Medium

Goal: improve quality without changing appearance.

Tasks:

1. Normalize breakpoints through the shared mixin.
2. Merge redundant media queries where safe.
3. Check heading order on homepage and secondary pages.
4. Check landmarks: header/nav/main/footer.
5. Confirm links and buttons have meaningful labels.
6. Replace placeholder `aria-label="..."`.
7. Confirm images have useful alt text.
8. Preserve visible focus states.
9. Respect reduced motion for pulsing or animated elements.
10. Add `<main>` landmarks to inner page layouts.
11. Check forms for `required`, `aria-required`, labels, and error-message relationships.
12. Verify color contrast for white text on `$primary`, muted text, and CTA buttons.

Verification:

- Keyboard navigation pass.
- Mobile viewport pass.
- Check generated HTML landmarks.

## Phase 10: Performance Review

Priority: Medium

Goal: reduce unnecessary CSS and asset overhead.

Tasks:

1. Remove stale generated CSS from `_assets` if unused.
2. Identify unused SCSS partials.
3. Check image references and loading behavior.
4. Add lazy loading where appropriate.
5. Review font loading.
6. Review bundled JavaScript dependencies.
7. Confirm Font Awesome usage still justifies the full package or choose a smaller path later.
8. Add Google Fonts preconnect links if the current font loading remains.
9. Decide whether portfolio should be hidden until real content and image references are ready.

Verification:

- Compare bundle sizes.
- Confirm no missing images or icons.
- Confirm no broken console errors in local preview.

## Phase 11: Naming Consistency

Priority: Low

Goal: make naming predictable without a risky rewrite.

Preferred convention:

- SCSS partials: `_component-name.scss`
- Includes: `component-name.html` only when introducing new files; avoid renaming existing includes unless necessary.
- CSS classes: component-oriented names with BEM-style modifiers where helpful.
- Tokens: clear purpose-based names, not one-off visual names.

Tasks:

1. Avoid new ID selectors for styling.
2. Prefer section/component class names over generic Bootstrap class overrides.
3. Avoid new `!important`.
4. Rename only where the old name is misleading and local impact is small.

Verification:

- Search for accidental broken class references.
- Build CSS and Jekyll.

## Phase 12: Dead Code Review

Priority: Low

Goal: remove only confirmed dead code.

Candidates to verify:

- `_assets/base/_page.scss`
- `_assets/layout/_contact_0.scss`
- `_assets/components/client-scroll.scss`
- `_assets/**/dist/*.css`
- `_assets/layout/_portfolio.scss` if portfolio is not rendered
- Unused includes not referenced by layouts/pages
- Empty or placeholder data sections

Tasks:

1. Confirm import references.
2. Confirm template references.
3. Confirm generated output references.
4. Remove in small commits or patches.

Verification:

- Build after each removal group.
- Search generated output if available.

## Phase 13: Documentation

Priority: Low

Goal: make future edits less fragile.

Tasks:

1. Add a short architecture note to `README.md` or a dedicated docs file.
2. Document SCSS folder purpose.
3. Document how to add a section.
4. Document how to add a calculator.
5. Document build commands.
6. Add comments only where they explain non-obvious decisions.
7. Add a final refactor report with completed, skipped, deferred, and remaining-risk sections.

## Recommended Implementation Order

1. Build baseline and record current output status.
2. Fix build pipeline enough to support Dart Sass.
3. Add missing breakpoint token support.
4. Move global styles out of `site.scss`.
5. Centralize calculator iframe styles.
6. Extract WhatsApp floating button styles.
7. Consolidate nav includes.
8. Clean `head.html`.
9. Consolidate page/post shell behavior.
10. Make calculator pages data-driven.
11. Remove confirmed dead SCSS and generated source-folder CSS.
12. Resolve Kilo cross-check conflicts that affect markup or routes.
13. Run accessibility and responsive pass.
14. Add final documentation.

## Build Checkpoints

Run after each high-impact phase:

```powershell
npm run bundle
bundle exec jekyll build --trace
```

Suggested pages to inspect after each phase:

- `/`
- `/contact`
- `/bond-calculator`
- `/affordability-calculator`
- `/transfer-cost-calculator`
- `/partnership`
- `/legal`

## Definition Of Done

The refactor is complete when:

- The site builds successfully.
- The current visual appearance is preserved.
- Sass tokens are centralized.
- SCSS source files are organized by purpose.
- Deprecated Sass architecture has a migration path or has been migrated.
- Repeated navigation/page/calculator patterns are centralized.
- Dead generated CSS in `_assets` is removed if confirmed unused.
- Accessibility issues found during the audit are resolved or documented.
- The final summary lists duplicated code removed, components centralized, SCSS simplified, performance improvements, accessibility improvements, and maintainability gains.
