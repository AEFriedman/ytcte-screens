# ytcte-screens — Changelog

Created: 2026-09-04

SHIPPED v0.0.0 — project created

APPROVED v0.1.0 — 2026-09-04 03:25 — spec approved (portrait hallway display: Slides embed + clock strip), build starting

SHIPPED v0.1.0 — 2026-09-04 03:44 — first build: portrait page (index.html) with an 80/20 Slides-deck/clock-strip split, 12-hour clock + secular date + civil-day Hebrew date (America/New_York, explicit Intl timezone), self-recovery (deck load-timeout retry + fixed-interval backstop reload, page-level clock-heartbeat reload), Drive-API change detection built but shipped disabled (blind 10-minute reload is the active path). Built and reviewed overnight on branch overnight/2026-09-04-first-build; not merged to main, not deployed.
