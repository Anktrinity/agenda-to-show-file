---
name: agenda-to-show-file
description: Turn an event agenda into the two show files every event needs - a planner-facing Run of Show and an AV-facing Cue-to-Cue sheet. Use when the user says "turn this agenda into a run of show", "build the ROS", "create the cue to cue", "make a show file from this agenda", "build my show docs", or provides an agenda (docx, PDF, spreadsheet, or pasted text) and asks for a run of show, cue sheet, Q2Q, or show file. One agenda in, two files out. Not for comparing or syncing existing show file versions.
---

# Agenda to Show File

One input: the event agenda, in any format (docx, PDF, xlsx, or pasted text).
Two outputs, always both, always Excel:

1. **Run of Show (ROS)** - planner-facing. Every scheduled item, every room, every day.
2. **Cue-to-Cue (Q2Q)** - AV-facing. Show-call granularity for the primary program room only.

This is a one-direction transformation. Never diff versions, never sync documents,
never merge multiple agendas. If the user provides more than one agenda version,
ask which one is current and use only that one.

## The Iron Rules (read before anything else)

1. **Never build structure. Fill structure.** Both output files start as copies of
   the blank templates in this skill's `assets/` folder. Never create headers,
   columns, or sheet layouts from memory.
2. **Never add, remove, rename, or reorder columns.** The templates are locked.
3. **Never invent facts.** Names, mic assignments, song titles, room names, preset
   contents - if the agenda does not state it, the cell gets `[CONFIRM]`.
   Invention is where inconsistency comes from. A `[CONFIRM]` is a feature: it
   becomes the planner's to-do list.
4. **A field that does not apply gets an en dash** (`–`), never blank, never guessed.
5. **The same agenda must always produce the same structure.** Same sheets, same
   segment count, same cue pattern, same numbering. If a judgment call is needed,
   use the deterministic rules below, never preference.

## Step 1 - Parse the agenda

Read the agenda with the right tool for its format (openpyxl for xlsx, pdftotext
or the pdf skill for PDF, docx tools for Word, plain reading for pasted text).

Normalize every scheduled item into this internal record, preserving agenda order:

- Day (e.g., "Thursday" or "Day 1")
- Start time, End time (convert everything to 24h HH:MM:SS; keep "TBD" as TBD)
- Title (verbatim from the agenda, typos included - do not editorialize)
- Speaker / owner / talent (only if the agenda names them)
- Location / room
- Room set, AV flags, attendance, notes (whatever columns the agenda offers)

**Room rule:** an item's room is the value in the agenda column labeled Location,
Room, Room location, or Venue - and only that column. Never take a room from a
Room Set, Setup, or Notes column even if it looks like a room name. If the
location cell is empty, the room is `[CONFIRM]`. Room name matching is
case-insensitive ("plaza" and "Plaza" are the same room).

**Day rule:** items with no day stated go at the end of the LAST day of the
event, after all dated items, with `[CONFIRM]` in their time cells and
"Day not stated in agenda [CONFIRM]" in Notes. Never attach them to Day 1 and
never drop them.

Do not drop items. Office holds, registration desks, breaks, receptions, off-site
items, and "guests on own" blocks all go in the ROS.

## Step 2 - Build the Run of Show

Copy `assets/ROS_Template_BLANK.xlsx`. Columns, in locked order:

| Item # | Start | End | Duration | Title | Speaker/Sponsor/Vendor | Location | Setup | Screen 1 | Lighting | Audio | Notes |

Deterministic rules:

- One sheet per event day, named `Day 1`, `Day 2`, ... in chronological order.
  Row 1 of each sheet: `DAY N  ·  {Day of week, Date}` (use `[CONFIRM]` for the
  date if the agenda gives only weekdays).
- Rows sorted by start time. Ties sort by room name A-Z. Items with TBD times go
  last in their day, in agenda order.
- Item # is sequential from 1 within each day. No gaps, no reuse.
- Duration formatted `H:MM`; for all-day or open-ended holds use the agenda's own
  language (e.g., `~9 hrs`, `24 Hour Hold`).
- Speaker/Sponsor/Vendor: the value of the agenda's speaker/host/owner column,
  verbatim. If the agenda has no such column or the cell is empty: `–`. Never
  extract names from notes or description columns into this cell.
- Screen 1 / Lighting / Audio: if the agenda row has any value in an AV column,
  all three cells read `[CONFIRM]` (the planner specs them). If the AV column is
  empty or absent for that row, all three read `–`. No other outcomes.
- Notes column carries, in this order, joined with ` | ` (space pipe space):
  first the agenda's own notes verbatim, then every leftover column value that
  did not fit a locked column, each written `{Agenda column name}: {value}`, in
  the agenda's column order. Nothing gets thrown away, and no other separator or
  phrasing is allowed.

## Step 3 - Build the Cue-to-Cue

Copy `assets/Q2Q_Template_BLANK.xlsx`. Columns, in locked order:

| Cue # | Start | End | Duration | Video Preset | Center Screen | Outboard Screens | Downstage Timer | DSMs | Slide # | Video & PPT Notes | Audio | Mics | Lighting | Stage and Other Notes | Notes |

**Which room gets a Q2Q:** the primary program room only, chosen by this test and
nothing else: count, per room (using the Room rule above), the items whose Title
contains any of *plenary, general session, keynote, welcome, closing, capstone*.
The room with the highest count is the program room; break a tie by which room
appears first in the agenda. State the chosen room in the sheet name:
`Q2Q - DAY N - {ROOM}`. One sheet per day on which that room has at least one
timed item. All other rooms live in the ROS only.

**Header block:** exactly two placeholder fills - `{EVENT}` in the cheat-sheet
cell N1 becomes the event name, `{DATE}` in cell E2 becomes that sheet's day and
date (date `[CONFIRM]` if the agenda gives only weekdays). Crew lines stay blank
for the planner. Keep the preset cheat sheet exactly as the template defines it:
P1 walk-in / P2 intro / P3 session / P4 video, L1 intro / L2 presenter / L3 dark.
Nothing else in rows 1-5 changes. Freeze panes at A7 (and at A3 in the ROS).

**Segment header rows:** EVERY item whose room is the program room becomes a
full-row segment header: `{n} - {Segment Title}` numbered 1, 2, 3... in
chronological order - including meals, breaks, receptions, and evening
entertainment held in that room. No skipping, no merging.

**Pre-show block (locked, three rows, all unnumbered, before segment 1):**

1. Crew call row: Start `[CONFIRM]`, every other cell `–` except Stage and Other
   Notes = `TECH CREW CALL TIME`.
2. Walk-in cue: Start = 60 minutes before the first segment's start, End = first
   segment's start, Duration computed. P1 walk-in, Center Screen
   `Motion background`, Outboard Screens `Holding Slide`, Audio
   `Spotify Playlist`, Mics `–`, Lighting L1, Stage and Other Notes
   `DOORS / HOUSE MUSIC`.
3. VOG standby row: Start = first segment's start, End `–`, Duration `–`,
   P2 intro, Audio `VOG: [CONFIRM]`, Mics `–`, Lighting L1, Stage and Other
   Notes `STANDBY: VOG`. All other cells `–`.

**Cue numbering:** first cue after pre-show is 101. Every cue increments by 1,
continuously across all segments in the day. Pre-show rows carry no cue numbers.

**Which segments expand:** a segment gets the full three-cue block below only if
it is show-called content, decided by this test and nothing else: the agenda
names a speaker/host for it, OR its Title contains any of *session, plenary,
keynote, panel, presentation, welcome, closing, capstone, awards, forum,
showcase*. Everything else (meals, breaks, transitions, open work time, fairs,
receptions, karaoke, parties) is a single cue: full duration, P1 walk-in, Audio
`Spotify Playlist` (or as stated in the agenda), Lighting L1, Stage and Other
Notes `BREAK` for breaks/meals or the item title in caps for everything else.

**Standard expansion - every show-called segment becomes this cue block:**

1. **Standby cue** - Start = segment start minus 0:00:10, End = segment start,
   Duration 0:00:10. P1 walk-in, Lighting L1, bumper music `[CONFIRM]`, Stage
   Notes: `STANDBY: {mic} - {talent}` for each person entering. Mics assigned in
   order Lav 1, Lav 2... for presenters and HH 7, HH 8 for emcees/Q&A, but every
   assignment reads `[CONFIRM]` unless the agenda names the person.
2. **Session cue** - Start = segment start, End = segment end, Duration computed.
   P3 session, Downstage Timer set to the session duration, Mics listing who is
   live, Lighting L2.
3. **Exit cue** - Start = segment end, End = segment end plus 0:00:30, Duration
   0:00:30. P1 walk-in, Lighting L1, walk-off music `[CONFIRM]`, Stage Notes:
   `everyone exit SR`. Append ` | STANDBY: [CONFIRM]` only when the next segment
   is itself show-called; when the next segment is a single-cue item or the day
   ends, the cell is exactly `everyone exit SR`.

Breaks and meals: single cue, full duration, P1 walk-in, Audio `Spotify Playlist`,
Lighting L1, Stage Notes `BREAK`. Video playback items: single cue, P4, both
screens `VPB Full`. Stream/record starts and stops only if the agenda mentions
streaming or recording, as their own unnumbered rows: `START/END STREAMING`,
`ROLL/END RECORDS`.

**Cell-fill determinism (locked wording, no alternatives):**

- Every value copied from the agenda is stripped of leading and trailing
  whitespace, otherwise verbatim.
- No cell is ever left empty: a cell with nothing to say gets the en dash `–`.
- All times are zero-padded `HH:MM:SS` (`07:00:00`, never `7:00:00`).
- Cue # cells are literal integers, never formulas.
- Start/End are literal HH:MM:SS values. Duration is literal `H:MM:SS` computed
  End minus Start; if End precedes Start or either is TBD, Duration = `[CONFIRM]`.
- When the agenda names nobody for a standby, the Stage and Other Notes cell is
  exactly `STANDBY: [CONFIRM]` - no mic slot, no placeholder name.
- Standby cue Audio cell: `Bumper: [CONFIRM]`. Exit cue Audio cell:
  `Walk-off: [CONFIRM]`. Session cue Audio cell: `Speaker: {names}` if the
  speaker column names them, else `[CONFIRM]`.
- Mics: only people from the agenda's speaker/host column count as named. Named
  people get `Lav 1 - {name}`, `Lav 2 - {name}`... in the column's listed order.
  `HH 7` / `HH 8` appear only for people the speaker column labels MC, emcee, or
  host, and for Q&A standbys. Everyone else: `[CONFIRM]`.
- Single-cue segments: Mics `–`, DSMs `–`, Slide # `–`, Downstage Timer `–`.
- DSMs and Slide # are always `–` unless the agenda explicitly states slide or
  DSM content.
- Unnamed agenda columns carried into Notes are prefixed `Col {letter}:`
  (e.g., `Col E: {value}`).
- Center Screen: `Motion background` on walk-in/break cues, `–` on standby and
  exit cues, `[CONFIRM]` on session cues. Outboard Screens: `Holding Slide` on
  walk-in/break/standby/exit cues, `[CONFIRM]` on session cues.
- Video & PPT Notes: always `–` unless the agenda states deck or video content.
- The Q2Q Notes column (last column) is always `–`. Agenda notes live in the ROS
  Notes column only; never copy them into the Q2Q.

**Standby/go language is locked:** `STANDBY:` prefixes in Stage and Other Notes,
GO is implied by the cue row itself. Never write "cue", "take", or freeform call
language in cells.

## Step 4 - Verify before delivering (mandatory)

Re-open both finished files and check, programmatically, not by eye:

1. Every agenda item appears in the ROS exactly once: ROS data row count equals
   the agenda's item count.
2. For each Q2Q day, the segment header count equals the number of timed
   program-room items that day in the ROS. A missing segment is a hard failure.
3. Cue numbers per sheet are continuous integers starting at 101, no gaps,
   no formulas.
4. Header rows match the blank templates cell-for-cell.

If any check fails, fix the file and run all four checks again. Do not deliver
files that have not passed.

## Step 5 - Deliver

File names, exactly:

- `{Event Name}_Run_of_Show_v1.xlsx`
- `{Event Name}_Q2Q_v1.xlsx`

`{Event Name}` is the event's name from the agenda with spaces preserved, or
`[CONFIRM]` asked of the user if the agenda has no name. Save both to the user's
working folder and present both files. Close by listing the `[CONFIRM]` count in
each file so the planner knows what to chase.
