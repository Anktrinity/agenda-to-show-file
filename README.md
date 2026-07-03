# agenda-to-show-file

A Claude skill for event professionals. You give it one thing, your event agenda, in any format (Word, PDF, spreadsheet, or pasted text). It gives you back two files, every time:

1. **Run of Show (ROS)**, planner-facing. Every scheduled item, every room, every day.
2. **Cue-to-Cue (Q2Q)**, AV-facing. Show-call detail for your main program room.

The skill fills a fixed template. It never makes up structure, and it never makes up a fact. Anything your agenda does not state comes back flagged as `[CONFIRM]`, so the file doubles as your chase list instead of pretending to be finished.

---

## The one idea to understand first

There are two different files in play, and they travel differently. Mixing them up is the single most common point of confusion.

- **The skill** is the tool. It is `SKILL.md` plus two blank templates in the `assets/` folder. You install it once. It already contains the templates. You never hand the templates to Claude.
- **The agenda** is your input. It is a different file for every event. You bring it fresh each time.

So when you run this, you are only ever supplying your agenda. The templates are already inside the installed skill, and Claude copies them for you. If you ever think "I need to give Claude the template," stop, you do not. Installing the skill covered that.

---

## What is in this package

```
SKILL.md                        the skill logic: triggers, rules, step-by-step behavior
assets/ROS_Template_BLANK.xlsx  the blank Run of Show template (the skill fills a copy)
assets/Q2Q_Template_BLANK.xlsx  the blank Cue-to-Cue template (the skill fills a copy)
```

Everything above is bundled inside the `agenda-to-show-file.skill` file. When you install that one file, you get all three.

---

## Before you start

You need three things:

1. A Claude product that supports skills (for example the Claude desktop app).
2. The `agenda-to-show-file.skill` file (download it from this repo, see "Get the skill" below).
3. Your event agenda, in any format.

You do not need Excel, Python, or any template files of your own. The skill carries its own templates.

---

## Get the skill

**If you downloaded from GitHub:**

- Easiest: on the GitHub page, open the green **Code** button and choose **Download ZIP**, then unzip it. Inside you will find `agenda-to-show-file.skill`.
- Or clone the repo with `git clone` if you are comfortable with Git.

Either way, the file you actually install is `agenda-to-show-file.skill`.

---

## Step 1: Install the skill

Pick one of these two ways.

**Way A, from a Claude chat:**

1. Open (attach) the `agenda-to-show-file.skill` file in a chat.
2. Click **Save skill**.
3. Done. It is now installed.

**Way B, from settings:**

1. Open **Settings**.
2. Go to **Capabilities**, then **Skills**.
3. Upload `agenda-to-show-file.skill`.
4. Confirm it now appears in your skills list.

You only do this once. After that the skill is always available.

---

## Step 2: Point Claude at your agenda

1. Put your agenda file in a folder (in Cowork, connect that folder to the session; in a normal chat, just attach the agenda file).
2. Make sure only the current agenda is present. If you have several draft versions, keep the one you want, or Claude will ask which one is current.

That is all you provide. No templates.

---

## Step 3: Ask for the files

Say it the way you would say it to a colleague:

> Turn this agenda into a run of show and a cue to cue.

The skill recognizes the request, reads your agenda, and builds both files.

---

## Step 4: What you get back

Two Excel files, named for your event:

- `{Event Name}_Run_of_Show_v1.xlsx`
- `{Event Name}_Q2Q_v1.xlsx`

Before handing them over, the skill runs four checks: every agenda item appears in the ROS exactly once, the Q2Q segment count matches the program-room items, the cue numbers run continuously, and the headers match the locked templates.

**About the yellow `[CONFIRM]` cells:** those are not errors. They are the skill telling you the agenda did not state something (a speaker name, a room, a mic, a date). Treat the highlighted cells as your to-do list before the show.

---

## Troubleshooting

**"It says it cannot find the template" or the output looks empty.**
This almost always means the skill was installed without its `assets/` folder. Reinstall using the `agenda-to-show-file.skill` file (not loose files), since that bundle carries both templates inside it. To confirm the bundle is complete, you can rename a copy to `.zip`, open it, and check that `assets/ROS_Template_BLANK.xlsx` and `assets/Q2Q_Template_BLANK.xlsx` are both there.

**Claude asks which agenda to use.**
You have more than one agenda version in the folder. Tell it which one is current. It will use only that one.

**A duration or time shows `[CONFIRM]` and an item sorts to the bottom of its day.**
Your agenda has a time that ends before it starts (often a stray AM/PM typo). The skill flags it rather than guessing. Fix the time in the agenda and rerun.

---

## For maintainers (forking and packaging)

This section is for the person who owns the repo, not the everyday user. It exists because the whole skill breaks if the two template files do not ride along with it. Four things keep a fresh download complete.

**1. Make sure the templates are actually committed.**
The number one packaging failure is a `.gitignore` that quietly excludes the templates. If your ignore file lists `*.xlsx`, `assets/`, or Office patterns like `~$*`, the blank templates never reach GitHub. Check with:

```
git ls-files assets/
```

Both `ROS_Template_BLANK.xlsx` and `Q2Q_Template_BLANK.xlsx` must appear. If they do not, they are not in the repo, and every fresh clone will be broken.

**2. Avoid the Git LFS trap.**
If the `.xlsx` files are tracked with Git LFS, anyone who uses "Download ZIP" from the GitHub web page gets small pointer text files instead of real spreadsheets, which fails exactly like a missing template. For files this small, keep them as normal Git objects rather than LFS.

**3. Rebuild the `.skill` bundle after any template change.**
The `.skill` file is a zip snapshot of `SKILL.md` plus `assets/`. If you edit a template and forget to regenerate the bundle, you ship a stale one. Commit the source (`SKILL.md` and `assets/`) as the source of truth, and regenerate the bundle on release:

```
# from the repo root, produce agenda-to-show-file.skill
zip -r agenda-to-show-file.skill SKILL.md assets/
```

**4. Tell users which path is supported.**
The supported install is the `.skill` file (Step 1 above). If someone instead uses the raw source, the loader expects the `assets/` folder to sit right next to `SKILL.md`, which the skill's relative paths already assume.

**Fastest end-to-end proof:** on a second machine, delete your local copy, download the repo the way a stranger would, and run `git ls-files assets/`. If both templates list, a fresh user has everything.

---

## Make it yours

This repo is meant to be forked. The skill is written in event-production language on purpose.

- **Triggers** are your cue words. Edit them in the description field of `SKILL.md`.
- **Locked templates** are your show file. Swap `assets/` for your own blank templates, then rebuild the `.skill` bundle (see maintainer step 3).
- **The no-inventing rule** is your cue lock. Keep it. It is what makes the output trustworthy.

Start from the painful document you rebuild every event. Decide what should always be pulled forward, what should never be assumed, what should be flagged, and what template the output must follow. That is your skill. The document changes. The skill pattern does not.

---

## Credit

Built by [Anca Platon Trifan](https://www.linkedin.com/in/ancatrifan/) for the Claude for Events community. Demonstrated with the fully fictional Summit Horizon 2026 conference, built from scrubbed documents.
