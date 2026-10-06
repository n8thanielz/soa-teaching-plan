# ACCTG Teaching Plan

Planning tool for the University of Utah School of Accounting. It replaces the old "ACCTG Master Schedule" spreadsheet (Master Plan tab plus yearly Teaching Matrix tabs). Owner and only editor: Nate Zwart, Program Director. Department chair Ed Owens is kept in the loop through Nate.

The tool answers two questions for each academic year: how many sections of each course do we need, and who is teaching them.

## How it's built

- One self-contained file: `index.html`. Vanilla JS and CSS, no framework, no build step. Keep it that way unless there's a strong reason.
- External loads: Google Fonts (Libre Franklin) and SheetJS from cdnjs for Excel export. If SheetJS fails to load, export falls back to CSV.
- Runs by opening the file in Chrome or Edge, locally or from GitHub Pages.

## Data and privacy (important)

- `index.html` must never contain real data. `const SEED = /*SEED*/null;` stays null. With no data, the page shows a welcome screen asking the user to open a plan file.
- Real data lives in a JSON plan file (e.g. `ACCTG_teaching_plan.json`) that Nate keeps in a synced folder. Plan files are gitignored. Never commit one, never paste faculty names or contracts into code, tests, or docs.
- Saving uses the File System Access API (`showSaveFilePicker`, handle remembered in IndexedDB). Fallback is download/upload.
- localStorage keeps an autosave copy in the browser. It is a safety net, not the source of truth.

## Data model (plan file)

```
{
  version: 2,
  years: ["2025-26", "2026-27", ...],
  courses: [{ id, code, title, credits, category, active, assignable, variable?, note? }],
  faculty: [{ id, name, track, active, contracts: { "2026-27": 24 } }],
  needs: { courseId: { "FA26": 2, "SP27": 3 } },
  assignments: [{ id, term, courseId, facultyId, credits? }]
}
```

- Terms are `FA##`, `SP##`, `SU##`. IDs are stable; names and codes can change without breaking references.
- `assignment.credits` is an optional per-section override. Without it, credits come from the course.
- Bump `version` and add a step to `migrate()` for any change to the file shape. Old files must keep opening.

## Business rules

- Academic year is Fall, Spring, Summer. AY 2026-27 = FA26, SP27, SU27.
- Section counts are a hand-entered forecast based on last year's enrollment.
- One row per person, any number of assignments. No separate overload rows.
- Contracts are entered as written. Course releases are already subtracted (example: career-line standard is 24, tenure-track is 9, a career-line person with a 3-credit release is 21).
- Every assigned course counts toward the contract, summer included. Summer is not paid differently.
- Going over contract is allowed. It shows as overload (gold), not as an error.
- Problems (red badge on Checks): open sections, sections assigned beyond the plan, inactive course or person still assigned.
- Info only: overload, under contract, teaching with no contract entered, custom credit hours on non-variable courses.
- Super Sections (2100SS, 3100SS) are historical, 6 credits each, inactive. Don't plan new ones.
- ACCTG 6910 and MBAO 6910 are special topics with variable credit (usually 1.5 or 3). Credits are set per section.
- ACCTG 4999 gets a section count but no instructor (`assignable: false`).
- No funding-source tracking, no sign-offs, no comments.

## Writing and design conventions

- Plain, direct UI copy in sentence case. No corporate jargon.
- No em dashes anywhere: UI text, docs, comments.
- Colors carry meaning: red = open/problem, gold = overload or beyond plan, green = covered/at contract, slate = interactive/selected.

## Testing

No test suite yet. Before committing, open `index.html`, load a plan file, and click through all five tabs. Playwright works for screenshot checks. Use a fake plan file for any automated test; never commit real data.

## Roadmap

- Phase 1 (done): section plan, assignments with open-slot gaps, faculty load, course list, checks, file save/open, Excel export.
- Next: faculty-facing read-only view of each person's assignments for the year (per-person "Copy summary" exists as a stopgap).
- Later: connect to the Class Schedule Editor (days, times, rooms) so everything lives in one place, and pull instructor history for "who can teach this."
- Later: project 5000-level enrollments from past enrollment plus curriculum structure (prerequisites, sequences). Store actual enrollments per section when this starts.

## Open items to confirm with Nate

- Faculty `track` is blank for everyone after import.
- Name spellings chosen at import: "Eldredge, Tom" and "Pickett, Scott".
