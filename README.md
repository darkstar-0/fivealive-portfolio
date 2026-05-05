# FiveAlive · High Jump Meet Manager
<img width="123" height="47" alt="image" src="https://github.com/user-attachments/assets/91352823-a179-4104-b7f1-61d948322630" />

FiveAlive is a browser-based tool for running high jump events at track meets. It manages jump order, attempts, bar height progression, and produces a printable scoresheet — all from a single HTML page with no install or backend required.

---

## Use Case

Running a high jump pit at a track meet involves constant judgment calls: who's up next, who passed, who just checked out to warm up, who needs to enter the rotation. Most meets still do this on paper or with ad-hoc spreadsheets. FiveAlive replaces that with a real-time queue manager built specifically for the rules and pace of a high jump event.

It handles:
- Girls and Boys events simultaneously, with a tab for each
- Up to 5 jumpers active in the rotation at once, with a managed waitlist
- Check-in, check-out, Pass, Pass Until Height, Jump ASAP, and Withdraw actions
- A live queue showing who is UP, ON DECK, IN THE HOLE, and on hold
- Auto-fill from the waitlist when a spot opens in the rotation
- A 2-minute per-jumper countdown timer
- A public-facing live scoreboard (`live.html`) for a second screen or audience display
- A final scoresheet with full attempt history, exported as PDF

---

## Features

### Setup
- **PDF import** — upload a meet program PDF and FiveAlive auto-extracts athlete rosters and starting heights for both events
- **AthleticLIVE import** — pull athlete data directly from an AthleticLIVE meet URL
- **Manual entry** — add athletes individually if needed
- **Bar heights** — configure starting height, increment, and number of pre-loaded heights in metric or imperial
<img width="820" height="801" alt="Import_Meet_Program" src="https://github.com/user-attachments/assets/239c5dc4-2a97-4b1d-882a-fa2aca9f6069" />

### Check-In
- Tap cards to check athletes in or out before the event starts
- Search by name or school; sort by bib number or alphabetically
- Add late entries on the fly
- See at a glance how many athletes have declared a starting height
<img width="893" height="1072" alt="Athlete_Check_In" src="https://github.com/user-attachments/assets/4ef702cc-59d2-42c2-9440-ac3b4d819233" />

### Competition
- Live jump queue with positional labels: UP / DECK / HOLE / HOLD / 5th / etc.
- Per-athlete hamburger menu for Out, Jump ASAP, Pass Until Height, and Withdraw
- Record O (make), X (miss), or P (pass) for each attempt
- Bar raise controls with configurable increment chips
- Checked-out jumpers tracked in a sidebar; check them back in to re-enter the rotation
- Undo last action via a toast notification
- Timer counts down 2 minutes per jumper
<img width="1221" height="853" alt="Active_Comp" src="https://github.com/user-attachments/assets/09eb5fe7-b275-404b-83fb-cf1c0ae6c5b2" />
<img width="1217" height="834" alt="Active_Comp_Scoreboard_filling" src="https://github.com/user-attachments/assets/f67cb794-5391-44f5-94d8-4f66a735efba" />


### Live Scoreboard (`live.html`)
A read-only public view that auto-refreshes every 3 seconds. Shows current height, NOW indicator, queue order, and attempt bubbles. Open it in a second browser tab on the same device and put it on a monitor or TV facing the athletes.

<img width="709" height="695" alt="Live_Queue" src="https://github.com/user-attachments/assets/d9b54baf-f684-4e26-b600-2a1499119b69" />
<img width="690" height="769" alt="Live_Scoreboard" src="https://github.com/user-attachments/assets/6da541da-92f7-4039-b1fb-7af3f82a0b93" />


### Results & Scoresheet
- Final standings with places, marks, and full attempt history
- Counts withdrawn athletes and gives them a real place in results
- Export to PDF via jsPDF
<img width="970" height="975" alt="Final_Results" src="https://github.com/user-attachments/assets/c765ffe9-42af-4b16-ba98-2986a38233e1" />


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

---

## Built By

Jack Faulkner — built for use at real track meets.
