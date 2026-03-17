# Personal Command Center Progress

## Current Status

Project phase: planning completed, implementation not yet started.

What exists today:

- Approved design direction for the personal command center
- Initial design document in `docs/command-center-design.md`

What does not exist yet:

- Frontend app scaffold
- Routes, components, or styles
- Report content schema implementation
- Market data integration
- Deployment pipeline
- OpenClaw publishing instructions

## Decisions Made

- The app will be a `Vite + React` SPA.
- The site will have a landing page plus a reports section.
- The landing page will show market movement and a Google-style search bar.
- The reports section will list daily reports as cards and support full detail pages.
- Reports will be stored as structured content in the repo, not as bot-authored raw HTML pages.
- GitHub will be the source of truth for application code and report publishing.
- OpenClaw will publish by committing report content and pushing to GitHub over SSH with a dedicated bot key.
- A single VPS will deploy and host the site from GitHub updates.
- The site is public to view, but publishing is private.
- Delayed near-real-time market data is acceptable for v1.

## Completed

- Wrote the first design doc: `docs/command-center-design.md`

## In Progress

- Nothing currently in progress

## Next TODOs

1. Scaffold the `Vite + React` application.
2. Set up base SPA routing for `/`, `/reports`, and `/reports/:slug`.
3. Create an initial design system shell for layout, navigation, cards, and report pages.
4. Define the report content format and file layout under a content directory.
5. Add one or two sample reports to validate the content model.
6. Build the reports list page and report detail page.
7. Build the landing page search experience.
8. Add the market overview module with a pluggable quote data source.
9. Write OpenClaw publishing instructions in `AGENTS.md` or a dedicated skill/workflow doc.
10. Design the GitHub-to-VPS deployment flow and validation checks.

## Open Questions

- Which market data provider to use for v1
- Whether report content should be Markdown, MDX, or a JSON block format
- How the VPS deployment should be triggered: webhook, polling, or pull-on-demand
- What visual direction the command center should take in the first UI pass

## Context Recovery Notes

If work resumes later, start by reading:

1. `docs/command-center-design.md`
2. This file

The current repo intentionally has very little implementation state. The main source of truth right now is the design doc, and the next milestone is scaffolding the app.
