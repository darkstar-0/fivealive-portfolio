# FiveAlive — Technical Overview

This document covers the engineering decisions behind FiveAlive and the development process used to build and validate it. It's written for a technical audience — people who want to understand not just what was built, but how it was designed, tested, and refined through real-world use.

---

## The Problem

High jump is one of the more logistically complex field events to manage. Unlike running events, athletes don't all compete at the same time — they rotate through a jumping order, can pass on a height, can check out temporarily, can enter the competition at different starting heights, and can be eliminated at any point. Meets can have 20–30 athletes per gender competing simultaneously at a single pit, with one official responsible for tracking jump order, recording attempts, managing a waitlist, and maintaining the current bar height — all in real time under time pressure.

Most officials still do this on paper or improvise with spreadsheets. FiveAlive was built to replace that with a purpose-built queue manager.

---

## Architecture Decisions

### Vanilla JS, No Framework

FiveAlive is intentionally built without a framework, bundler, or build step. The entire app runs by opening `index.html` in a browser. This was a deliberate product decision: the target users are track meet officials, often working on a laptop or tablet at an outdoor venue with unreliable connectivity. No install, no npm, no server dependency — it just works.

The tradeoff is managing UI state manually. All state lives in a single global object per event and is rendered by explicit `render*()` calls rather than reactive bindings. This is verbose but completely transparent — there's no framework magic to debug under pressure at a meet.

### State Machine for Competition Logic

The core of the app is a queue state machine with two layers: a `rotation[]` (up to 5 active jumpers) and a `waitList[]` (athletes waiting to enter). Key transitions:

- **Pass** — athlete exits the rotation, their carried misses are preserved, and they re-enter via `buildRotation()` at the next height
- **Pass Until Height** — athlete is parked until a specific height is reached, then automatically re-enters
- **Check Out** — athlete is removed from the rotation immediately; a waitlist athlete fills the slot; the checked-out athlete does not auto-re-enter when the bar is raised
- **Check In** — athlete re-enters the rotation (if a slot is open) or the front of the waitlist, inserted in bib number order
- **Withdraw** — permanent; athlete is removed and shown as WD in the final results with their best mark

Getting these transitions right — especially the interaction between Pass, Check Out, and the waitlist fill logic — required several rounds of bug fixing. Early versions had cases where a checked-out athlete would silently re-enter the rotation on a bar raise, or where a waitlist athlete would be skipped during fill.

### PDF Parsing

Meet programs are distributed as PDFs, and officials need athlete rosters pre-loaded before the event. FiveAlive parses these PDFs using coordinate-based line reconstruction rather than raw text extraction. Raw text extraction from PDFs loses column structure — names, schools, and marks end up concatenated in unpredictable order depending on how the PDF was generated.

The coordinate approach groups text tokens by their vertical (y) position within a threshold, then sorts each line's tokens by horizontal (x) position to reconstruct columns. This handles both digitally-generated PDFs and scanned programs (via a server-side OCR function). Name format normalization (Last, First → First Last) and date/meet name extraction are applied as a post-processing pass.

### localStorage Persistence

All competition state is serialized to `localStorage` under a single key and restored on page load, with a 30-day expiry. The serialization separates *durable facts* (athlete records, attempts, heights) from *recomputable state* (derived UI values) to keep the saved payload clean and avoid stale derived data corrupting a restored session.

### Public Live Scoreboard

`live.html` is a read-only public view that syncs in real time via Supabase. When an official starts a competition, the app pushes state to a `live_state` table on every recorded result; `live.html` fetches the initial state and subscribes to Postgres change events for instant updates. Anyone with the 6-character meet code — coaches, athletes, spectators on their own devices — can open the URL and follow along without any login.

The page has two tabs serving different audiences:

- **Queue view** — the jump order with positional labels (UP / DECK / HOLE / HOLD), the current jumper's NOW card with attempt slots, and the waitlist. Designed for coaches and athletes who need to know who's up and who's next.
- **Scoreboard view** — a ranked results table showing every athlete's best height, attempt history across all heights jumped so far, total misses, and current place. Top-3 places are gold/silver/bronze highlighted. Athletes still in the competition show a pulsing indicator. The current bar height column is subtly highlighted. Designed for spectators who want to know who's winning and how athletes have performed.

One non-obvious architectural detail: the main app's `saveState()` function intentionally omits transient fields (`rotation`, `cur`, `waitList`) from localStorage — they're recomputed on load to avoid restoring a stale mid-jump state after a crash. `pushLiveState()` needed its own serialization path that reads directly from the live `EVENTS` object and layers those fields back in before pushing to Supabase, so the queue view has the data it needs.

---

## Testing Process

### Manual and Exploratory Testing

After each significant feature or behavior change, the app was tested manually against realistic meet scenarios — not just happy-path flows. Exploratory testing uncovered a class of bugs that unit tests would have missed: edge cases that only appear when multiple state transitions happen in rapid succession, or when the queue is in an unusual configuration (e.g., all five rotation slots filled, then two athletes check out simultaneously, then one is withdrawn).

Representative bugs found through exploratory testing:

- **Checked-out athlete re-entering on bar raise** — `buildRotation()` was not filtering `checkedOut` athletes, so raising the bar would silently pull them back into the rotation
- **Waitlist skip on fill** — `refill()` was advancing the waitlist pointer incorrectly when the front of the waitlist had a `startH` higher than the current bar
- **Results table blank after event end** — `renderResultsTable()` was not being called from `showResults()`, producing a blank scoresheet with no error
- **Pass carrying incorrect miss count** — `skippedMisses` was being reset before the next rotation was built rather than after, causing athletes to carry the wrong number of misses into the next height
- **Re-check-in placement** — athletes checking back in were being appended to the end of the waitlist rather than inserted in bib number order

### Dry Runs With Real Users

Before the first live meet, the app was used in two full dry-run sessions with the actual officials who would be running it. These sessions simulated a complete meet from setup through scoresheet export, using real athlete rosters.

Feedback from the dry runs fell into two categories:

**Usability issues** — actions that were technically correct but too slow or error-prone under time pressure. For example, the original per-athlete action buttons (Out, Jump ASAP, Withdraw) were always visible on every row, creating a cluttered interface that led to accidental taps. This was redesigned as a hamburger menu (≡) per athlete row, reducing the chance of a fat-finger error during a fast-paced rotation.

**Missing behaviors** — things that officials expected to be there based on how they'd always run the event manually. Pass Until Height was one of these: officials routinely park an athlete at a specific height they've declared in advance, and the original Pass implementation didn't support this. It was identified in the dry run and added before the first live meet.

### Formalizing Test Criteria

The dry runs revealed that informal testing was not catching enough — bugs were being found at the pit rather than at the desk. After the first round of user feedback, we went back to the officials and worked through a more concrete set of test criteria, written from the perspective of what actually happens at a meet:

- What happens when the last athlete in the rotation passes?
- What happens when an athlete checks out and there's nobody on the waitlist?
- What happens when two athletes are eliminated on the same height?
- What should the scoresheet show for an athlete who withdrew after clearing one height?
- What is the correct jump order when athletes re-enter the rotation from the waitlist mid-height?

These became the basis for the Cypress end-to-end test suite, which was written after the behaviors were agreed on rather than before — ensuring the tests reflected actual meet rules, not assumptions made during initial development.

### Feature Requests and Iteration

After the first live meet, officials came back with a set of new feature requests grounded in real use:

- **Check-in redesign** — the original check-in modal used a simple list; officials wanted a card grid with search and sort so they could quickly find athletes by name or bib number during the pre-event rush
- **Athlete bib numbers on check-in cards** — bib numbers are how officials identify athletes at a meet; they weren't shown in the original UI
- **Sort controls on check-in** — sort by bib number (default) or alphabetically by name
- **Pass Until Height** — as described above; officials declared heights in advance and needed the app to track this automatically
- **Jump order after re-check-in** — officials expected re-checking-in an athlete to restore them to a sensible position relative to other athletes, not append them to the end of the queue

Each of these went through the same cycle: discuss the intended behavior with the user, implement it, test edge cases manually, get confirmation. Several introduced new bugs in adjacent behavior that were caught before going back to the field.

---

## What I'd Do Differently

- **Write test criteria first.** The Cypress suite came late — after the behaviors were already implemented and partially broken. Agreeing on concrete acceptance criteria before writing code would have caught several of the queue logic bugs earlier.
- **Separate state from rendering earlier.** The `render*()` functions grew large as features were added. A cleaner separation between state mutation and render scheduling would make the code easier to reason about.
- **Design the live sync data contract earlier.** The main app's `saveState()` intentionally omits transient competition fields from localStorage, which was the right call for crash recovery — but it meant `pushLiveState()` had to be built as a separate serialization path that re-reads from the live `EVENTS` object. If the live sync requirement had been on the table from the start, the state model would have been shaped differently from the beginning rather than retrofitted.
