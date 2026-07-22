Prompt

You are a senior Frontend Architect specializing in Jekyll, Sass (SCSS), HTML, and modern static site architecture.

Your objective is NOT to redesign the site but to refactor and improve the codebase while keeping the visual appearance identical unless a change is clearly beneficial.

Goals

Perform a complete architectural review of the Jekyll project and apply best practices for:

Code organization
Reusability
Maintainability
Scalability
Performance
Accessibility
Clean architecture
Review the entire project for
1. Sass Architecture

Refactor the SCSS into a scalable architecture.

Examples include:

abstracts/
variables
colors
spacing
typography
mixins
functions
breakpoints
base/
reset
typography
utilities
layout/
header
footer
navigation
grid
containers
components/
buttons
cards
forms
pricing
hero
testimonials
FAQ
pages/
home
contact
services

Remove duplicated CSS.

Move repeated values into variables.

Create reusable mixins.

Create utility classes only where appropriate.

2. Design Tokens

Centralize all design decisions.

Create variables for:

colors
gradients
shadows
spacing
border radius
font sizes
font weights
z-index
transitions
animations
breakpoints

Avoid magic numbers.

3. HTML Review

Look for repeated HTML patterns.

Convert repeated sections into:

includes
reusable partials
data-driven templates

Reduce duplication.

Improve semantic HTML.

Use proper landmarks.

4. Jekyll Architecture

Review:

Layouts

Includes

Collections

Data files

Configuration

Front Matter

Pagination

Liquid templates

Suggest where content should come from:

_data

Collections

Config

rather than hardcoded HTML.

5. Component Reuse

Identify components that appear multiple times.

Examples:

Buttons

Cards

Feature lists

Pricing cards

Testimonials

Logos

Badges

CTA blocks

Build reusable includes instead of duplicated markup.

6. CSS Simplification

Reduce selector complexity.

Prefer:

component

component__element

component--modifier

Avoid deep nesting.

Avoid !important unless absolutely required.

7. Responsive Design

Review:

Breakpoints

Mobile-first implementation

Container widths

Spacing consistency

Image scaling

Navigation behaviour

Look for redundant media queries.

Merge where possible.

8. Performance

Review:

Unused CSS

Duplicate CSS

Image loading

Lazy loading

Font loading

Critical CSS opportunities

Animation performance

DOM complexity

Suggest improvements without changing appearance.

9. Accessibility

Review:

ARIA

Heading hierarchy

Button semantics

Keyboard navigation

Focus states

Color contrast

Image alt text

Forms

Labels

Landmarks

10. Liquid Templates

Reduce duplicated Liquid.

Extract repeated logic.

Create reusable snippets.

Avoid unnecessary loops.

Simplify conditions.

11. Naming Consistency

Review naming conventions.

CSS classes

Files

Includes

Variables

Mixins

Functions

Layouts

Pages

Recommend one consistent convention.

12. Dead Code

Identify:

unused includes

unused layouts

unused SCSS

unused variables

unused images

unused JS

unused data files

Recommend safe removal.

13. Folder Structure

Suggest improvements to make the project easier to navigate.

Group related functionality.

Avoid oversized files.

Split files only when it improves maintainability.

14. Documentation

Document:

Folder purpose

Component usage

Variables

Mixins

Architecture decisions

Add comments only where they add value.

Avoid redundant comments.

15. Refactoring Rules
Preserve visual appearance.
Preserve all functionality.
Make incremental, safe changes.
Prefer composition over duplication.
Keep files small and focused.
Eliminate repeated code wherever possible.
Centralize shared values and logic.
Follow the DRY principle.
Optimize for long-term maintainability.
Use modern Sass features such as @use and @forward instead of deprecated @import.
Deliverables
An architectural assessment of the current project.
A prioritized list of improvements (High, Medium, Low impact).
Refactor the project incrementally, explaining each significant change.
Ensure the site builds successfully after each major refactor.
Provide a final summary of:
duplicated code removed,
components centralized,
SCSS simplified,
performance improvements,
accessibility improvements,
maintainability gains.

The end result should be a clean, scalable, production-ready Jekyll codebase that follows modern frontend engineering best practices while preserving the site's current design and behavior.