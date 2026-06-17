# agenda-to-show-file

A Claude skill for event professionals. One input, the event agenda, in any format (Word, PDF, spreadsheet, or pasted text). Two outputs, always both, always Excel:

1. **Run of Show (ROS)** - planner-facing. Every scheduled item, every room, every day.
2. **Cue-to-Cue (Q2Q)** - AV-facing. Show-call granularity for the primary program room only.

The skill fills structure. It never invents structure, and it never invents a fact. Anything the agenda does not state comes out flagged as `[CONFIRM]`, so the file doubles as your production chase list instead of a source of false certainty.

## What is in this package

```
SKILL.md                         the skill: triggers, iron rules, step-by-step logic
assets/ROS_Template_BLANK.xlsx   the locked Run of Show template (blank; the skill fills a copy)
assets/Q2Q_Template_BLANK.xlsx   the locked Cue-to-Cue template (blank; the skill fills a copy)
```

The columns are locked, the naming is locked, and the standby language is locked. The skill copies these blank templates and fills them, so the same agenda always produces the same structure.

## Install it

You have two ways:

1. **From a Claude chat:** open the `agenda-to-show-file.skill` file and click **Save skill**.
2. **From settings:** Settings, then Capabilities, then Skills, then upload `agenda-to-show-file.skill`. Confirm it appears in your skills list.

## Use it

Put one agenda file in a folder, start a session connected to that folder, and say it the way you would say it to a colleague:

> Turn this agenda into a run of show and a cue to cue.

The skill recognizes the request, reads the agenda, builds both files, and runs four verification checks before handing them back.

## Make it yours (fork)

This repo is meant to be forked. The skill is written in event-production language on purpose:

- **Triggers** are your cue words. Edit them in the `description` field of `SKILL.md`.
- **Locked templates** are your show file. Swap `assets/` for your own blank templates.
- **The no-inventing rule** is your cue lock. Keep it. It is what makes the output trustworthy.

Start from the painful document you rebuild every event. Decide what should always be pulled forward, what should never be assumed, what should be flagged, and what template the output must follow. That is your skill.

The document changes. The skill pattern does not.

## Credit

Built by Anca Platon Trifan for the Claude for Events community. Demonstrated with the fully fictional Summit Horizon 2026 conference, built from scrubbed documents.
