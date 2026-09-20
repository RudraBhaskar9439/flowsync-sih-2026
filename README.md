# FLOWSYNC — SIH 2026

Functional browser prototype based on the supplied SIH deck. See docs/TECHNICAL-BLUEPRINT.md for the implementation map, architecture, API proposals, limitations and demo script.

Run `python3 -m http.server 4173 --directory dist` and open http://localhost:4173.

Static ES modules, IndexedDB file vault, localStorage workflow state. No build step, API key or external package installation. Do not open index.html directly using file:// because browsers restrict ES module loading there.

All approval rules, durations and matching categories are illustrative. AI guidance uses keyword retrieval, government actions are simulated, and role preview does not implement authentication.

## Verification

Run `node tests/engine-test.mjs` for workflow, dependency, scheduling and expiry checks.

## Vercel deployment

Import this repository in Vercel. The committed `vercel.json` selects the Other preset and serves `dist` with no installation or build step. The prototype has no server secrets or required environment variables.

Browser state is isolated by origin: the Vercel deployment starts with fresh demo state and does not migrate documents from the previous hosted site.
