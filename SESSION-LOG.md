# ytcte-screens — Session Log

## 2026-09-04 (morning, main, continued)

**Shipped v0.1.2**, fast path: `viewportDebugEnabled` flipped to `false`, and
nothing else — orientation is confirmed portrait-native on the real screen,
so the debug readout had served its purpose. Single-round SHIP, no blockers.

Backlog additions (not for this cycle, from the user directly): a nightly
full-page reload so shipped changes land on the Cast without a manual
reload; removing the now-confirmed-dead rotation hedge; revisiting strip
height and clock size after judging them from the hallway.

## 2026-09-04 (morning, main)

Merged `overnight/2026-09-04-first-build` into `main` and pushed (user-approved,
explicit — the real GitHub Pages deploy) with `.nojekyll` added first.

**Shipped v0.1.1**, fast path: the Hebrew date was rendering with nekudos and a
ב prefix on the month name (from Hebcal's `hebrew` field); switched to
`heDateParts` (day/month/year joined with spaces), added `dir="rtl"`. Code
review round 1 came back DO NOT SHIP — not over the diff, but because
`reference/Code.gs` and `reference/index.html` (a different project's code,
untracked, no `.gitignore`) were sitting in the repo carrying a Twilio SID,
staff phone numbers, and a live anonymous-access Apps Script URL, one
`git add -A` away from landing in this public repo's history. Added
`.gitignore` (`reference/*`, keeping `.gitkeep`), confirmed via
`git check-ignore` and a clean `git status`, resubmitted, SHIP on round 2.

**Pending, per code review's non-blocker notes:** the Hebrew date line still
has no visible degraded indicator on a failed Hebcal lookup (unlike the deck's
own change-detection path) — logged to `BACKLOG.md`.

## 2026-09-04 (overnight, branch overnight/2026-09-04-first-build)

**Shipped v0.1.0** — the project's first build: `index.html`, a static portrait
page for the hallway Cast Pro display. Slides deck (80%) above a clock strip
(20%, one config value), 12-hour clock + secular date + civil-day Hebrew date
(via Hebcal, all timezone-explicit to America/New_York), self-recovery
(deck load-timeout + fixed-interval backstop reload, page-level
clock-heartbeat reload), Drive-API change detection built but shipped
disabled — the active path tonight is a blind 10-minute reload.

Full path: spec (`v0.1.0`) went through 2 rounds (r1 REVISE SPEC on a
Drive-key-location contradiction, r2 APPROVED TO BUILD), code went through 1
round (SHIP, no blockers). Self-review before code review caught and fixed
several real bugs, most notably: the JSONP approach originally planned for
the Hebrew-date lookup does not work at all on real Chromium (cross-origin
read blocking silently refuses it) — confirmed by testing against the live
Hebcal endpoint and replaced with a plain `fetch()`, which Hebcal's CORS
headers support directly.

**Not done tonight, by explicit instruction:** nothing was merged to main,
nothing was pushed to main, no GitHub Pages deploy happened, no Google Cloud
project or Drive API key was created, no Drive sharing was touched. Work
stays on `overnight/2026-09-04-first-build` until reviewed.

**Pending, needs the actual screen:** viewport orientation (portrait-native
vs. landscape-then-rotated — the debug readout ships on for this), the exact
strip-height percentage (shipped at 20%), and the deck chrome-crop pixel
value (shipped at 60px). The deck ("BBYWelcome") also still needs its page
setup resized to 7.5in × 10.67in to match the 80/20 split — that's the
user's own task, not part of this build, and it will letterbox until done.

**Decisions logged this cycle** (see `verdicts/v0.1.0/` for the full review
history; `DECISIONS.md` is reset for the next cycle):
- Spec wording fix: the Drive API key's home is this repo's config block
  (restricted key, same pattern as a public Maps key), not "outside the
  repository" — matches what was already agreed in Phase 0.
- Enabling change detection later needs the deck file to stay/become
  publicly link-shared. Not a new decision — the user already checked this
  deck's sharing earlier in this session (`role: reader, type: anyone`) — but
  worth reconfirming against the actual file id before ever pasting in a
  real key.
- The 10-minute fixed reload was not revisited against deck-loop length,
  since the deck currently has one slide (5s loop) — moot for now, backlogged
  for when the deck grows.

**Full backlog additions** are in `BACKLOG.md` — notably the rotation hook's
`vh`-based sizing doesn't yet follow a rotated box correctly (inert at the
shipped 0deg default), and a handful of init-order/hardening items from code
review.

**Next session should:** check the actual screen (orientation, strip height,
crop pixels), resize the deck, decide whether/when to stand up the Drive API
key, and re-review this cycle's branch before it goes anywhere near main.
