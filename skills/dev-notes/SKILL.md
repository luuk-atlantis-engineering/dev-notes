---
name: dev-notes
description: "Keep per-ticket development notes outside the repository: the plan, the progress, and a decision log that records WHY every decision was made, what was rejected, who decided, and what it assumes. Use when starting or resuming work on a ticket (checks the ticket's notes and creates them if they're missing); the moment a decision is made (a user answer, a review response, a design choice, a default you picked yourself, a change of direction); after a commit, push or PR; at the end of a session; and whenever someone asks 'why did we…'. Triggers: 'we start working on <ticket>', 'start on', 'pick up', 'continue with', 'back to', 'let's work on', 'resume', 'note this', 'write it down', 'update the notes', 'tracker', 'decision log', 'record the decision', 'why did we', 'why is it like this'."
---

# Dev notes

Working notes per ticket, kept outside the repository: the plan, the progress, and above all the **decisions with
their reasons**.

The code and the commits show *what* was done. These notes keep *why*. A reviewer or a future teammate will
eventually ask "why did we build it this way?" When they do, the answer should be written down, not buried in an
old chat transcript that nobody can find. A reason reconstructed afterwards is weaker than one written at the time.
It's often a guess, and it misses the options nobody weighed.

## Where the notes live

- **Root folder.** Use the path that the project or user instructions (CLAUDE.md, memory) give for development
  notes. If none is given, ask once. Suggest `~/development_notes/`, and suggest adding one line to CLAUDE.md:
  `Development notes: <path>`.
- **One folder per ticket**, named by the ticket key (`PROJ-1234/`). Work without a ticket gets a short topic slug
  (`passkeys-spike/`). Work that spans tickets gets its own folder (`tools/`, `coordination/`).
- Create the folder when work on the ticket starts (see "Starting a ticket"), or the first time you write to it.
- **Never inside the repository's working tree, and never staged.** If a notes file turns up in the tree, move it to
  the notes folder.

## The files

| File | Holds | Update it |
|---|---|---|
| `DECISIONS.md` | The driver of the ticket, and one entry per decision | The moment a decision is made |
| `PLAN.md` | Goal, scope, approach, open questions | At the start, and when the direction changes |
| `PROGRESS.md` | Current state and next step, plus a dated log of the work, commits and PRs | After each step, commit, push or PR, and at the end of a session |

Add other files as needed: `REVIEW-TRACKER.md` (each review comment and what happened to it), `MANUAL-TESTING.md`,
`PR-<number>-BODY.md` drafts, query snapshots, scripts.

Start a new ticket folder from the three templates in `templates/`.

## Starting a ticket

When work starts or resumes on a ticket ("we start working on PROJ-1234", "pick up PROJ-1234", "continue with…"), do
this before anything else:

1. **Find the ticket's folder** under the notes root.
2. **If it exists, read it:** `PROGRESS.md` ("Now"), `PLAN.md` and `DECISIONS.md`. Tell the user where things stand
   in three lines: the state, the next step, and what's open or blocked. Then carry on from "Now".
3. **If it doesn't exist, create it now,** with the three files from `templates/`:
   - **Fill the driver and the goal from the ticket:** the summary and the description, if a tracker tool is
     available (Jira, GitHub issues, Linear). The driver says why the ticket exists, in the words of whoever wrote it.
     Leave out what the ticket doesn't say. Don't make it up.
   - **No tracker tool, or nothing useful in the ticket?** Ask the user one question: why does this ticket exist, and
     what's the goal?
   - **Set "Now"** in `PROGRESS.md` to "Started <date>", with the first step as the next step.
   - Tell the user the folder was created, in one line.
4. **Then start the work.**

## The decision log

**Record every decision when it's made, with its reason.** Don't wait until later, and don't record only the
outcome. Write the entry before you carry on with the work that follows from it.

### What counts as a decision

Anything where another reasonable choice existed:
- The user answers a question, picks an option, or corrects your approach.
- A reviewer asks for a change and you accept it, accept part of it, or push back.
- You choose between plausible approaches without asking. Record it as a **default** so it can be challenged later.
- The plan changes direction, or something is deferred or ruled out of scope.
- A check, a measurement or an experiment settles a question.
- A naming, structure, placement or testing choice, when there was a real alternative.

Mechanical steps with only one sensible way to do them aren't decisions.

### The entry

Number the entries per ticket (`D-1`, `D-2`, …) and append each new one at the end:

```markdown
### D-7 · Cache the price list per request, not per process · 2026-03-14

- **Decided by:** the user, after review comment from Sam
- **Decision:** the price list is loaded once per request and dropped at the end of it.
- **Why:** prices change during the day and a stale price on an invoice is a legal problem (the user: "we can
  never show yesterday's price"). One load per request keeps a page to a single query.
- **Trade-off:** every request still reads the price list once; fine at today's volume, not at 10x.
- **Rejected:**
  - A process-wide cache with a 5-minute expiry: it can serve a stale price for up to 5 minutes.
  - No cache: a page made up to 40 identical queries.
- **Assumes:** prices are read far more often than they change; one price list per tenant.
- **Source:** PR #412 review thread, 2026-03-14; the user in the session that day.
- **Status:** Active
```

The fields:

| Field | What goes in it |
|---|---|
| **Decided by** | `the user`, `<reviewer> in review`, `team agreement`, or `default (not discussed)` for your own choices |
| **Decision** | What was chosen, specific enough to check against the code |
| **Why** | The reason at the time. See the rules below |
| **Trade-off** | What this costs, risks or makes harder |
| **Rejected** | Each alternative, and why it lost |
| **Assumes** | What has to stay true. If it changes, the decision should be reopened |
| **Source** | Where it was decided: a conversation, a PR thread, a meeting, a ticket comment, with links where they exist |
| **Status** | `Active`, `Superseded by D-n (date)`, or `Premise weakened by D-n (date)` |

### Rules for the why

- **Give the real reason from the time:** the problem, the constraint, the evidence, or the stated reason. When the
  user gave a reason, quote them.
- **No restatements** like "we chose X because X is better", and no reasons made up afterwards.
- **If nobody gave a reason, write "No reason given."** Don't invent one. For a default you picked, say what tipped it.
- **Name the trade-off.** A decision with no cost listed usually means nobody looked.
- **List the alternatives.** If none was considered, write "No alternative was considered." That gap is worth
  knowing.
- **Write down the assumptions.** Those are what a later change silently breaks.
- **Record the driver once, at the top of `DECISIONS.md`:** why the ticket exists at all, in the words of whoever
  asked for it, and its hard constraints. Every decision is judged against it. If the driver is "less setup work",
  a decision justified by "more security" deserves a second look.

### Keeping the log true

- **Append only.** Never delete or rewrite an entry. A changed decision gets a new entry that supersedes the old one.
  Set the old entry's status to `Superseded by D-n (date)` and add one line saying why it changed.
- **Mark weakened premises.** When a later decision or a new fact undermines an earlier entry's premise, mark it on
  the earlier entry. For example: `Premise weakened by D-9 (date): the import job now writes bookings directly, so
  a check in the service no longer covers every write`.
- **Link answered questions.** When a question in `PLAN.md` gets an answer, mark it answered and link the entry.
- **Write in small appends when sessions run in parallel.** If several sessions work on the same ticket, make each
  change one small append or edit. Never rewrite the whole file, or another session's write gets lost.

## Plan and progress

- **`PLAN.md` stays current.** When the direction changes, add a dated banner at the top that links the decision.
  Keep the old plan below it, marked as replaced, instead of deleting it.
- **`PROGRESS.md` starts with "Now":** the state, the next step and what's blocking. Update it so a fresh session can
  resume from that section alone. Below it is the log, newest first: the date, what changed, the commit and PR
  references, and the decisions each step carried out (`carries out D-4`).

## Answering "why did we…?"

1. Read the ticket's `DECISIONS.md` first, and cite the entry.
2. If the decision isn't there, reconstruct it from the sources (session transcripts, commits, PR threads, ticket
   comments). Add the entry marked `Reconstructed on <date> from: <sources>`. Say plainly where the reason wasn't
   recorded or an alternative wasn't weighed. Don't fill the gaps with plausible reasons.

## Keep the notes out of the product

The notes are private working files, and reviewers can't see them. Commit messages, code comments and PR
descriptions must stand on their own. State the reason itself. Never write "see D-4", a notes file path, or any
label that exists only in the notes. Ticket keys are fine.

## Checklist

- **Starting or resuming a ticket:** find the folder; read it, or create it from the templates and the ticket. Then
  read "Now" before you act.
- **The user answered or chose:** write the decision entry, then continue.
- **After a review round:** write one entry for each comment whose outcome was a real choice: accepted, accepted in
  part, or pushed back.
- **You picked a default:** write an entry with `Decided by: default (not discussed)`.
- **After a commit, push or PR:** add a progress line.
- **At the end of a session:** bring "Now" up to date.

See `references/examples.md` for good and bad entries, a superseded decision, a review push-back and a reconstructed
entry.
