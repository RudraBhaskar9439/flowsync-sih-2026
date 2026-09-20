# FLOWSYNC — SIH 2026

Functional browser prototype based on the supplied SIH deck. See outputs/FLOWSYNC-Technical-Blueprint.md for the implementation map, architecture, API proposals, limitations and demo script.

Run `python3 -m http.server 4173 --directory dist` and open http://localhost:4173.

Static ES modules, IndexedDB file vault, localStorage workflow state. No build step, API key or external package installation. Do not open index.html directly using file:// because browsers restrict ES module loading there.

All approval rules, durations and matching categories are illustrative. AI guidance uses keyword retrieval, government actions are simulated, and role preview does not implement authentication.
