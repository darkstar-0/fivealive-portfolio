# FiveAlive · High Jump Meet Manager

| Light Mode                            | Dark Mode                            |
| ----------------------------------- | ----------------------------------- |
| <img width="119" height="55" alt="image" src="https://github.com/user-attachments/assets/2436d8d4-c99d-4f72-947e-9318daa5ff58" /> | <img width="119" height="58" alt="Screenshot 2026-05-05 at 3 22 20 PM" src="https://github.com/user-attachments/assets/b3ada936-e45e-4d6e-849f-f6cef3f4dc7c" />|


FiveAlive is a browser-based tool for running high jump events at track meets. It manages jump order, attempts, bar height progression, and produces a printable scoresheet — all from a single HTML page with no install or backend required.

---

## Use Case

Running a high jump pit at a track meet involves constant judgment calls: who's up next, who passed, who just checked out to warm up, who needs to enter the rotation. Most meets still do this on paper or with ad-hoc spreadsheets. FiveAlive replaces that with a real-time queue manager built specifically for the rules and pace of a high jump event.

It handles:
- Girls and Boys events, with a tab for each
- Up to 5 jumpers active in the rotation at once, with a managed waitlist
- Check-in, check-out, Pass, Pass Until Height, Jump ASAP, and Withdraw actions
- A live queue showing who is UP, ON DECK, IN THE HOLE, and on hold
- Auto-fill from the waitlist when a spot opens in the rotation
- A 2-minute per-jumper countdown timer
- A public-facing live scoreboard (`live.html`) — real-time Supabase sync, accessible on any device with the meet code
- A final scoresheet with full attempt history, exported as PDF

---

## Features

### Setup
- **PDF import** — upload a meet program PDF and FiveAlive auto-extracts athlete rosters and starting heights for both events
- **AthleticLIVE import** — pull athlete data directly from an AthleticLIVE meet URL
- **Manual entry** — add athletes individually if needed
- **Bar heights** — configure starting height, increment, and number of pre-loaded heights in metric or imperial
<img width="820" height="801" alt="Import_Meet_Program" src="https://github.com/user-attachments/assets/d8bc6fcc-3cbe-4594-85b3-6badc31a13c8" />

### Check-In
- Tap cards to check athletes in or out before the event starts
- Search by name or school; sort by bib number or alphabetically
- Add late entries on the fly
- See at a glance how many athletes have declared a starting height
<img width="893" height="1072" alt="Athlete_Check_In" src="https://github.com/user-attachments/assets/12c4f06c-4955-4978-8e25-109437731dc6" />

### Competition
- Live jump queue with positional labels: UP / DECK / HOLE / HOLD / 5th / etc.
- Per-athlete hamburger menu for Out, Jump ASAP, Pass Until Height, and Withdraw
- Record O (make), X (miss), or P (pass) for each attempt
- Bar raise controls with configurable increment chips
- Checked-out jumpers tracked in a sidebar; check them back in to re-enter the rotation
- Undo last action via a toast notification
- Timer counts down 2 minutes per jumper
<img width="1221" height="853" alt="Active_Comp" src="https://github.com/user-attachments/assets/83a7a8f1-a893-436d-b7c0-70505720d18b" />
<img width="1217" height="834" alt="Active_Comp_Scoreboard_filling" src="https://github.com/user-attachments/assets/ed07b265-2613-4fb9-a939-f68b58b5a6fd" />


### Live Scoreboard (`live.html`)
A read-only public view that syncs in real time via Supabase — anyone with the 6-character meet code can open it on their own device, no login required. Two tabs serve different audiences:
- **Queue** — current jumper NOW card, positional queue labels (UP / DECK / HOLE / HOLD), and the waitlist. For coaches and athletes tracking who's up next.
- **Scoreboard** — ranked results table with best heights, full attempt history across all heights jumped, and place standings updated live. For spectators following the competition.
<img width="709" height="695" alt="Live_Queue" src="https://github.com/user-attachments/assets/03a5ad8f-6b08-48db-bb16-0af6c1a05673" />
<img width="690" height="769" alt="Live_Scoreboard" src="https://github.com/user-attachments/assets/06269647-eae3-49ed-9107-217907279587" />

### Results & Scoresheet
- Final standings with places, marks, and full attempt history
- Counts withdrawn athletes and gives them a real place in results
- Export to PDF via jsPDF
<img width="970" height="975" alt="Final_Results" src="https://github.com/user-attachments/assets/bed68dc1-6319-451c-b5bb-e4a0e63abeb5" />

---

## How to Use

FiveAlive is a static web app — no server, no build step, no install.


### Hosted version
Works in any modern browser. All state is saved to `localStorage` under the key `fiveAlive_state` and survives page refreshes for 30 days.

### Workflow
1. **Setup** — import athlete data (PDF, AthleticLIVE, or manual), set bar heights, click **Proceed to Check-In**
2. **Check-In** — tap each athlete card as they arrive; set their starting height if not already imported
3. **Start Competition** — click **Start Competition** to open the competition panel
4. **Run the event** — record results, manage the queue, use the hamburger menu for individual athlete actions
5. **End event** — click **End** to finalize; view and export the scoresheet

---

## Tech Stack

| Layer | Choice |
|-------|--------|
| Language | Vanilla JavaScript (ES6+) |
| Styling | Custom CSS with CSS custom properties (dark theme default, light theme toggle) |
| PDF parsing | pdf.js |
| PDF export | jsPDF |
| Auth / persistence | Supabase (optional login for trial/sharing features) |
| Build | None — open `index.html` directly |

No frameworks, no bundler, no dependencies to install.

---

## Files

| File | Purpose |
|------|---------|
| `index.html` | All panels and modals |
| `script.js` | All app logic |
| `styles_v2.css` | All styles |
| `live.html` | Public-facing live scoreboard |
| `auth.js` | Supabase auth helpers |
| `pdf-data.js` | Demo/test athlete data |

---

## Testing

Cypress end-to-end tests cover the core competition flow.

---

## Known Limitations

- PDF import uses coordinate-based line reconstruction tuned for common meet program formats; unusual layouts may need manual cleanup after import.
- The official's session state is stored in `localStorage` on their device — if they switch devices mid-meet, they'd need to re-import the roster. The live scoreboard itself is unaffected, as it reads from Supabase.

---

## Built By

Jack Faulkner — built for use at real track meets.
