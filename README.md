# dev-notes

A [Claude Code](https://claude.com/claude-code) skill that keeps development notes for each ticket: the plan, the
progress, and a **decision log that records why every decision was made**.

## Why

Code and commits record *what* was done. The reasons usually live only in a chat, a review thread or someone's head.
Months later a reviewer asks "why did we build it this way?", and the answer has to be dug out of old transcripts.
When it's finally found, it often shows that one alternative was never considered, or that a later change quietly
removed the original reason.

This skill makes Claude write the reason down at the moment of the decision:
- **what** was decided, and **who** decided it, including the defaults Claude picks on its own;
- **why**, the reason at the time, quoted where possible and never made up afterwards;
- the **trade-off** that was accepted;
- the **alternatives** that were rejected, and why, or an explicit note that none was considered;
- the **assumptions** that would reopen it if they changed.

## What you get

One folder per ticket, outside your repository:

```
development_notes/
  PROJ-1234/
    DECISIONS.md   the driver of the ticket, then one entry per decision (append-only)
    PLAN.md        goal, scope, approach, open questions
    PROGRESS.md    "Now" (state, next step, blockers) and a dated log
```

The skill also covers how to answer "why did we…?": cite the log, or reconstruct the decision from the sources and
mark the gaps. It also keeps the notes out of commits and PR descriptions, which have to stand on their own.

## Install

Copy the skill folder into your personal skills folder:

```sh
git clone https://github.com/luuk-atlantis-engineering/dev-notes.git
cp -r dev-notes/skills/dev-notes ~/.claude/skills/
```

To install it for one project only, copy it into that project's `.claude/skills/` folder instead.

## Configure

Tell Claude where your notes live, in your `CLAUDE.md`:

```
Development notes: ~/development_notes
```

If you don't, Claude asks once and suggests this default.

## Files

- [`skills/dev-notes/SKILL.md`](skills/dev-notes/SKILL.md): the rules Claude follows.
- [`skills/dev-notes/templates/`](skills/dev-notes/templates/): the starting files for a ticket folder.
- [`skills/dev-notes/references/examples.md`](skills/dev-notes/references/examples.md): good and bad entries, a
  default, a review push-back, a superseded decision and a reconstructed one.
