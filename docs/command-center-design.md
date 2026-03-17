# Personal Command Center Design

## Summary

This repo will become a single-page application that acts as a personal command center. The first version should deliver two user-facing experiences:

1. A landing page with live-feeling market context and a centered search box.
2. A reports experience where daily OpenClaw-generated reports appear as cards and open into full detail pages.

The app should be built as a `Vite + React` SPA, hosted from a single VPS, and deployed from GitHub. OpenClaw will publish reports by committing structured content into this repository and pushing to GitHub over SSH with a dedicated bot key. GitHub is the source of truth; the VPS deploys from GitHub updates after validation and build.

## Goals

- Provide a fast personal dashboard homepage that feels useful immediately on open.
- Show near-real-time market movement for key tracked symbols such as `QQQ`, `SPY`, `gold`, and `BTC`.
- Give OpenClaw a reliable, structured way to publish daily reports with consistent site-owned presentation.
- Keep report browsing simple: newest reports visible as cards, with a clean drill-down into full details.
- Preserve future flexibility for changing the search target from Google to ChatGPT or another destination.

## Non-Goals

- Exchange-grade real-time market data for v1.
- A general-purpose CMS or database-backed editorial workflow.
- Bot-authored raw HTML pages with custom layout control.
- Private site access in v1. Publishing is private; viewing is public.

## Product Overview

### Landing page

The landing page should combine a market snapshot with a prominent search interaction.

Expected elements:

- A market movement panel showing `QQQ`, `SPY`, `gold`, `BTC`, and other future configurable symbols.
- Price, absolute change, percent change, and last-updated time for each symbol.
- A centered search bar styled like a command-center focal point, submitting to Google in v1.
- A configuration layer that allows the search destination to be swapped later without rewriting the UI component.

The landing page should degrade gracefully if market data is unavailable. Search must remain usable even if quote fetching fails.

### Reports tab

The second primary navigation tab should list daily reports as cards. Each card should surface enough information to decide whether to open it.

Card contents should include at least:

- Title
- Report date
- Short summary
- Optional tags or category hints
- Optional thumbnail or hero image

The list should default to newest-first ordering.

### Report detail page

Each report should have its own route and use the same site visual language as the rest of the app. OpenClaw supplies content, but the application owns presentation.

Reports should support:

- Headings and sections
- Paragraph text
- Bullet lists
- External links
- Images
- Structured sections for topics like market recap, news, politics, and visa bulletin tracking

## Technical Architecture

### Frontend stack

- Framework: `React`
- Build tool: `Vite`
- App type: client-side SPA
- Routing: SPA routing with routes for `/`, `/reports`, and `/reports/:slug`

This stack is a good match for a lightweight dashboard, straightforward deployment, and future UI iteration without introducing unnecessary backend complexity.

### Content model

Daily reports should live in-repo under a content directory such as:

```text
content/reports/
```

Recommended format:

- Frontmatter metadata plus Markdown or MDX body content

This format supports links and images while keeping rendering controlled by the app. It also keeps content diff-friendly in git and easy for OpenClaw to generate.

Required report metadata:

- `slug`
- `title`
- `date`
- `summary`

Optional metadata:

- `tags`
- `heroImage`
- `coverAlt`
- `publishedAt`

Content ingestion should happen at build time. The app should derive:

- A report index for the cards view
- Per-report route data for the detail page

The build should fail clearly if required metadata is missing or if two reports share the same slug.

### Market data

The landing page should use a delayed-quote source suitable for periodic refresh. The exact provider is intentionally left open for implementation, but the architecture should treat the provider as replaceable.

Expected characteristics:

- Refresh on a fixed interval
- Configurable symbol list
- Graceful error handling
- Optional last-known-value cache for better fallback behavior

The design should assume delayed near-real-time data is acceptable for v1.

## Publishing and Deployment

### Source of truth

GitHub is the trusted source of truth for both application code and report content.

OpenClaw publishes by:

1. Generating a new structured report file in the allowed content path.
2. Creating a git commit in this repository.
3. Pushing the commit to GitHub over SSH using a dedicated bot identity/key.

### Authentication boundary

The requirement that only OpenClaw can publish should be enforced at the GitHub access layer, not in the browser and not by exposing git SSH directly on the VPS.

This means:

- The bot SSH key should be scoped only to the GitHub identity/repository access needed for publishing.
- Normal site visitors do not get any publishing capability through the SPA.
- The VPS should only deploy code that comes from GitHub updates.

### VPS deployment flow

The single VPS hosts the built SPA and acts as the deployment target.

Recommended deployment shape:

1. GitHub receives a push from the OpenClaw bot or a trusted human maintainer.
2. A deployment trigger notifies the VPS, or the VPS pulls the latest GitHub commit on update.
3. The VPS validates changed files before publishing.
4. The VPS builds the SPA.
5. The VPS serves the newly built site only if validation and build succeed.

Validation should ensure that bot-authored publishing changes are limited to expected content and asset paths. If validation or build fails, deployment must stop without replacing the working site.

## OpenClaw Authoring Contract

The repo should eventually include instructions for OpenClaw in `AGENTS.md` or a skill file so the bot can publish consistently.

Those instructions should define:

- The report file location
- The required metadata schema
- The expected naming convention
- Allowed asset locations
- Allowed link and image conventions
- The required validation checks before push
- The expected git workflow for adding a new daily report

OpenClaw should not generate complete standalone HTML pages for reports in v1. It should generate structured content only, and the app should render that content inside a site-owned layout.

## UI Direction

The command center should feel intentional and polished rather than generic dashboard boilerplate.

UI principles for v1:

- Strong landing-page focal point around search and market context
- Consistent card and article styling across report views
- Clean visual hierarchy that makes daily scanning easy
- A design system foundation that can support future sections without rework

The market panel and report cards should be visually cohesive so the homepage and report surfaces feel like one product.

## Public Interfaces and Config

The first implementation should define a few small internal interfaces:

- Search provider config
  - label or provider name
  - submit URL
  - query parameter name
- Market symbol config
  - display label
  - provider symbol
  - asset type if needed for formatting
- Report metadata contract
  - slug
  - title
  - date
  - summary
  - optional tags/images

These do not need to be public APIs, but they should be explicit and typed in the codebase.

## Testing and Acceptance Criteria

The implementation should be considered successful when the following are true:

- The landing page renders a market panel and search bar successfully.
- Quote fetch failures do not break the page and show a reasonable fallback state.
- The reports list renders cards from structured content in newest-first order.
- Clicking a card opens the corresponding full report page.
- Links and images render correctly inside reports.
- Invalid report metadata causes a clear build-time failure.
- Duplicate report slugs are rejected.
- The deployment flow does not publish a broken build.
- Publishing rights are limited to the authorized GitHub bot identity or other explicitly allowed repo writers.

## Implementation Notes for the First Build

When implementation begins, the likely initial milestones are:

1. Scaffold the Vite + React app and base routes.
2. Build a simple design system shell for landing page, cards, and report layout.
3. Add a structured report content loader and local sample report data.
4. Add the reports listing and report detail routes.
5. Add the landing page market module with a pluggable data source.
6. Add deployment wiring from GitHub to the VPS.
7. Add OpenClaw authoring instructions and validation rules.

## Assumptions

- The site is public to view.
- Publishing is private and enforced through GitHub repository access.
- Google is the default search provider in v1.
- Search-provider switching is a planned future capability.
- Delayed-refresh market quotes are sufficient for v1.
- Report topic areas such as market movement, domestic and international news, politics, Trump news, and visa bulletin tracking are content conventions within reports, not separate app modules in v1.
