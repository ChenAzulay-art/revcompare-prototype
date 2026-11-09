# Build prompt — Revision comparison prototype

Paste everything below the line into Claude Code, from inside a folder that contains `rev_A.png`, `rev_B_aligned.png` and `changes.json`.

---

I'm a Product Manager candidate at Ramdor, a cloud platform for managing construction projects. I need a clickable prototype of ONE flow: reviewing what changed between two revisions of a drawing. It is a product prototype, not a real product — the detection results are mocked in `changes.json`.

**The problem it solves:** when a new drawing revision arrives, the Architect / the Owner's Design Coordinator / the Project Manager spend a lot of time comparing it to the previous revision by eye, and changes get missed. The prototype must show that finding the differences is **easy** (a ready, sorted list that jumps to each change on the drawing) and **reliable** (nothing is hidden, and a per-reviewer counter shows what you haven't checked yet).

**Before writing any code, reply with a short plan (max 8 bullets) and wait for my OK.**

## Technical constraints

- One file: `index.html` with inline CSS and vanilla JavaScript. No frameworks, no build step, no external dependencies except optionally one Google Font.
- It must work by double-clicking `index.html` (file://). Browsers block `fetch()` of local JSON on file://, so **copy the contents of `changes.json` into a `const DATA = {...}` at the top of the script**. Load images with relative `<img src="rev_A.png">`.
- Keep the code simple and readable, because I will edit it live in an interview. Clear sections with comments: `DATA`, `STATE`, `RENDER: documents`, `RENDER: compare`, `RENDER: summary`, `EVENTS`. Short functions. Aim for under ~600 lines.
- State lives in memory (a plain `state` object). No localStorage.
- All UI text in English.
- Initialize a git repo and commit after each working milestone, with clear messages.

## Data

`changes.json` contains the sheet info, 4 change types and 12 changes. Each change has `type`, `critical`, `confidence`, `title`, `location`, `before`, `after`, optional `note`, optional `sheetWarning`, and `box` = `[x0, y0, x1, y1]` as **fractions** of image width/height (or `null` for whole-sheet changes). Both images are already aligned to each other, so the same box works on both. Never hard-code the number 12 — every count is computed from the data.

## Global header (all screens)

- Left: "Ramdor · Documents" as plain text (do not recreate the Ramdor logo), and a breadcrumb: `Neve Rehovot – Building A / Architecture / Floor plans`.
- Right: a **"Reviewing as"** dropdown with three roles: `Project Manager` (default), `Design Coordinator`, `Architect`. Each role has its **own** set of reviewed changes. Switching role switches whose progress you see. There is **no** team view — one reviewer's checkmarks never count for another.

## Screen 1 — Documents list

A table styled like a document management system. Columns: Sheet, Title, Current rev, Uploaded by, Date, Changes.

Rows:
1. `A-100` Site Plan · Rev A · no previous revision → Changes cell: "—"
2. `A-101` Typical Floor Plan · Rev B · from `DATA.sheet` → Changes cell: a badge **"{n} changes · {k} critical"** plus "Not reviewed by you" (or "{x} of {n} reviewed by you", or "✓ Reviewed by you" when done — for the current role). Clicking the row or badge opens Screen 2.
3. `A-201` Sections · Rev B → Changes cell: a neutral badge **"No changes detected"**.

## Screen 2 — Compare view (the core)

**Top bar:** `A-101 Typical Floor Plan` · `Rev A (12 Oct 2026) → Rev B (8 Nov 2026)` · uploaded by. A "← Documents" back link.

**Sheet warning banner** (yellow), shown if any change has `sheetWarning: true`: show its `title` and `note`, plus a link "Show in list" that selects it.

**Layout:** change list on the left (~360px), drawing viewer filling the rest. On narrow screens, stack them.

**Change list:**
- Grouped by type in this order: Design, Data, Documentation, Minor. Each group header shows the label and count, and can collapse. **Minor is collapsed by default** with the text "Hidden by default — below significance threshold". Never remove Minor items, only collapse them.
- Each item shows: number, title, location (small grey text), `before → after` (before struck through in red, after in green/blue), a **Critical** red tag if `critical`, a confidence tag (High / Medium / Low; Medium and Low visibly softer/amber), and a **"Mark reviewed"** checkbox. Reviewed items look checked and slightly dimmed but stay in place.
- Each item has a small "Type ▾" control that lets the reviewer **reclassify** it into another type; the item then moves to that group (counts update).
- Clicking an item selects it (highlighted in the list and on the drawing).

**Drawing viewer:**
- View mode toggle: **Side by side** (Rev A | Rev B, each labelled) · **Overlay** (Rev B on top of Rev A with an opacity slider) · **Rev B only**. Default: Rev B only.
- Draw every change with a non-null `box` as a numbered outlined rectangle, positioned with percentages so it scales with the image. Colour by type (Design, Data, Documentation, Minor) with a small legend. Critical changes get a thicker red outline. Boxes of collapsed/hidden groups are not drawn.
- Selecting a change zooms the drawing to its box (CSS transform scale + translate, smooth transition) and pulses the box briefly. A "Fit to sheet" button resets the zoom. Selecting a change with `box: null` resets to fit and shows its banner/details.
- Clicking a box on the drawing selects the matching item in the list.

**Bottom bar (sticky):**
- Progress for the current role: **"{x} of {n} reviewed by you"** with a progress bar. The count includes Minor changes.
- **"Next unreviewed"** button: selects the next change the current role hasn't reviewed (expanding its group if collapsed).
- **"Finish review"** button: enabled only when all changes are reviewed by the current role; opens Screen 3. When disabled, its tooltip says how many are left.

## Screen 3 — Summary

- Heading: "You've reviewed {n} of {n} changes" with the role name and today's date.
- Section "Critical changes": the critical items with before → after and note.
- Section "All changes by type": compact list with counts, including any reclassifications the reviewer made (show "reclassified from X").
- Buttons: **"Export PDF"** (calls `window.print()`, with a print stylesheet that hides navigation and prints the summary cleanly) and **"Back to documents"** (the A-101 row now shows "✓ Reviewed by you" for this role).

## Visual style

Clean, professional B2B SaaS. White surfaces, light grey background, one blue accent (e.g. `#1f5fa8`), red only for critical/removed, amber for medium/low confidence. System font stack or Inter. 8px spacing grid, subtle borders, rounded corners 6–8px. Readable at 1366×768 laptop resolution, which is what I'll present on.

## Acceptance checklist (verify each before telling me you're done)

1. Double-clicking `index.html` works offline, with no console errors.
2. All counts come from `DATA`; deleting a change from `DATA` updates every number on every screen.
3. Clicking each of the 10 changes with a box zooms to the right place on the drawing.
4. Minor is collapsed by default, and "Next unreviewed" still reaches its items.
5. Reviewed progress is separate per role.
6. Reclassifying moves the item and updates counts and the summary.
7. Finish review stays disabled until everything is reviewed.
8. Print preview of the summary looks clean.

Do not add features that are not in this spec. If something is ambiguous, ask me instead of guessing.
