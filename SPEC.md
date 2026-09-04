# SPEC — ytcte-screens

Version: v0.1.0
Date: 2026-09-04

## Problem

A portrait-mounted 58" hallway display, driven by a UniFi Display Cast Pro pointed at a Web URL, currently shows only a looping Google Slides deck. Staff want a clock visible on the same screen. Nothing today can add one without giving up the deck or requiring a person to touch the screen for every change.

## Behavior

### Layout

The page fills the portrait viewport with two vertically stacked regions: the Slides deck above, the clock strip below. The split between them is expressed as a single configuration value, set to 80% deck / 20% strip for this cycle.

Whether the Cast renders the page portrait-native or renders it landscape and rotates the result afterward is unverified as of this cycle — there is no one awake to check it against the real hardware tonight. The page assumes portrait-native rendering (roughly 1080 wide by 1920 tall). It is built so that if that assumption turns out to be wrong, correcting it is a single rotation applied to one top-level wrapper, not a re-layout of the page.

A small on-screen readout of the actual rendered viewport dimensions ships enabled by default for this cycle, specifically so the assumption above can be checked by looking at the real screen. It is controlled by a configuration value so it can be turned off once that check has happened.

### Slides deck region

The deck plays via Google's own published-embed player: autostarting, looping, advancing on its own timing. The embed player's own chrome (controls, borders) is not visible to the viewer.

The deck's own page setup is still sized for a different aspect ratio than the 80% region requires as of this cycle. Until it's resized, the deck will visibly letterbox inside its region. This is expected for this cycle, is not something the page's layout compensates for, and is not part of this build — see External dependencies below.

### Clock strip

The strip shows three things, all computed against the America/New_York timezone explicitly, independent of whatever timezone or clock the Cast's hardware itself is set to:

- The current time, 12-hour with AM/PM, updating live. This is the dominant visual element in the strip, sized to be readable by someone walking past in the hallway.
- The day of week and secular date.
- The Hebrew date, sourced from Hebcal's date-conversion API. It follows the civil (midnight) day, not halachic nightfall. If a lookup fails, the strip continues showing the last successfully retrieved Hebrew date rather than going blank.

### Keeping the deck current

Two independent mechanisms govern when the deck region reloads, controlled by a configuration flag:

- **Default state for this cycle (flag off, no key configured):** the deck region reloads on a fixed timer, defaulting to every 10 minutes, regardless of whether the deck actually changed. This is the path that runs on the wall tomorrow and is the one that has to work correctly with nothing else enabled.
- **When enabled with a valid key:** the page instead polls the deck file's modification time directly against the Drive API on that same timer cadence, and reloads the deck region only when it detects an actual change. The fixed timer above continues running underneath as a backstop even while this is active.
- If the modification-time poll starts failing (for example, because the deck's sharing changed and the key can no longer read it), the page falls back to the fixed-timer behavior automatically and shows a small, unobtrusive on-screen indicator that it is running in that degraded state. It does not fail silently.

Enabling this mode requires a Drive API key that does not exist yet as of this cycle — see External dependencies. Once it exists, the key lives in this same configuration block, committed into this public page like the rest of the block. It is restricted by HTTP referrer to this site's origin and scoped to the Drive API only, the same pattern as a public Maps JS key: the restriction, not secrecy, is what makes it safe to commit.

### Recovery

The page runs unattended for weeks and has to recover on its own from both known failure modes without anyone visiting the screen:

- **The deck region going blank or unresponsive.** Because the deck plays in a cross-origin frame, the page cannot inspect its contents to detect this directly. Recovery instead combines two things: a load-timeout on every reload attempt (if the deck region doesn't finish loading within a bounded time, it's retried), and the fixed-timer reload described above, which also serves as a backstop for a deck that rendered fine initially and went blank sometime later without a new navigation to catch.
- **The page itself hanging, including a frozen clock.** A stale clock on a wall is worse than a black screen, because nobody notices it's wrong. The page watches its own clock for signs it has stopped advancing and reloads the entire page if it has.

### Configuration

All values specific to this one screen — the deck's embed URL and file id, the region split, the change-detection flag and key, the fixed reload interval, and the viewport-debug flag — live in a single place in the page. This is the only concession made toward the eventual multi-screen version of this product: later work should be able to replace this one block rather than hunt values out of the page.

## External dependencies (not addressed by this build)

- The deck's page setup needs to be resized to match the 80/20 split before the layout looks correct on the real screen. Owned by the deck's owner, outside this build.
- Change detection ships built but disabled: no Google Cloud project or Drive API key has been created yet. Creating that key is a manual step outside this build, done in Google Cloud Console. Once created, it is entered into this page's configuration block and committed to this repository — see the key-location note above.
- The viewport-orientation assumption and the exact strip-height percentage both still need confirmation by looking at the actual mounted screen.

## Out of scope

Unchanged from the Phase 0 brief: more than one screen; any dashboard, layout editor, or push-to-screen mechanism; per-screen configuration or a config store; photo slideshows, announcements, tickers, or any content region beyond the deck and the clock strip; any backend, database, or Apps Script; a custom domain or Railway hosting; any other campus screen; editing the deck's content beyond the page-setup resize; anything touching the Cast's network configuration or UniFi Connect beyond eventually pointing it at this page's URL.
