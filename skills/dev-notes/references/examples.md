# Decision entries: examples

## A weak entry, and the same decision done well

**Weak:**

```markdown
### D-3 · Use a queue for the emails · 2026-05-02
- **Decision:** emails go through a queue.
- **Why:** it's better.
```

That tells a later reader nothing. Better than what? Who decided? What did it cost? What was the other option, and
when should anyone revisit this?

**Good:**

```markdown
### D-3 · Send emails from a background queue, not inside the request · 2026-05-02

- **Decided by:** the user
- **Decision:** the request writes an outbox row in the same transaction; a background job sends it.
- **Why:** the mail server times out a few times a day, and inside the request that rolled back the whole
  sign-up (two support tickets last month). The user: "a slow mail server must never cost us a customer".
- **Trade-off:** emails can arrive up to a minute late; one more table and one more job to watch.
- **Rejected:**
  - Send in the request and retry 3 times: still blocks the request for up to 30 seconds.
  - Fire-and-forget task: an email is lost on a restart, and nothing records that it was lost.
- **Assumes:** a minute's delay is acceptable for every email we send. A one-time sign-in code would break this.
- **Source:** the session on 2026-05-02; support tickets #881 and #893.
- **Status:** Active
```

## A default you picked yourself

```markdown
### D-5 · Put the validation in the service, not the controller · 2026-05-03

- **Decided by:** default (not discussed)
- **Decision:** the "end date after start date" check lives in `BookingService`.
- **Why:** the other three rules for bookings are already there, and the import job calls the service directly,
  so a controller check would let imports skip it.
- **Trade-off:** the API returns the error from a little deeper in the stack; no visible change to callers.
- **Rejected:**
  - In the controller: the import job would bypass it.
- **Assumes:** every write to bookings goes through the service.
- **Source:** my own choice while implementing step 2.
- **Status:** Active
```

Recording defaults matters. They're the decisions nobody remembers making, and the first ones a reviewer
questions.

## Pushing back on a review comment

```markdown
### D-8 · Keep the type name in the filter labels (reviewer suggested dropping it) · 2026-05-09

- **Decided by:** the user, replying to Sam's review comment
- **Decision:** a type-specific field keeps its type's name in the label: "Power (Pumps)".
- **Why:** two types can each have a field called "Power". Without the type name, the filter list shows two
  identical "Power" entries, and the wrong one silently returns nothing, because values are stored per type.
- **Trade-off:** longer labels; a lookup of the type name per field.
- **Rejected:**
  - Sam's suggestion, a plain "Power (type-specific)": simpler, but leaves the duplicates indistinguishable.
- **Assumes:** field names aren't unique across types. If they become unique, revisit.
- **Source:** PR #455, review thread on `FieldLabels.cs`, 2026-05-05 to 2026-05-09.
- **Status:** Active. Waiting for Sam's answer.
```

## A decision that changed

The old entry stays. Only its status line changes:

```markdown
### D-2 · Combine a user's roles as a union · 2026-06-01
...
- **Status:** Superseded by D-11 (2026-06-20): a check on production data showed about 40 users would gain access.
```

The new entry says what changed and why:

```markdown
### D-11 · Combine roles the way the old system does (a part any role limits stays limited) · 2026-06-20

- **Decided by:** the user
- **Decision:** replaces D-2. A restriction set by any of a user's roles applies.
- **Why:** a read-only count on production data found about 40 users who would gain access under the
  union. The user: "no one's access may widen or narrow at the upgrade".
- **Trade-off:** users with several roles keep the old, stricter behaviour, including its known oddities.
- **Rejected:**
  - Keep the union and notify the customers: every customer would have to review their roles before upgrading.
- **Assumes:** customers expect the upgrade to change nothing about access.
- **Source:** `access-count.sql` and its results in this folder; the session on 2026-06-20.
- **Status:** Active
```

## A reconstructed entry

When someone asks "why?" and the log has nothing, rebuild the entry from the sources and be honest about the gaps:

```markdown
### D-4 · Keys rotate themselves every 90 days · 2026-02-10

- **Reconstructed on 2026-04-02 from:** the design session transcript (2026-02-10), PR #301's description.
- **Decided by:** the user and a design review, 2026-02-10
- **Decision:** signing keys expire after 90 days; a successor is created 14 days early.
- **Why:** it replaced the manual certificate renewal the old setup needed. **Whether to rotate at all was never
  discussed**, and no customer or security requirement was cited.
- **Trade-off:** an hourly background check and a runtime reload, the riskiest part of the PR.
- **Rejected:** "rotate immediately": it breaks the tokens other servers issued. **Not considered:** deriving the
  keys from the secret, which needs no rotation machinery.
- **Assumes:** rotation is needed. Nobody checked this assumption.
- **Source:** see "Reconstructed" above.
- **Status:** Reopened. A reviewer challenged it on 2026-04-01.
```

The bold gaps are the most useful part. They show where the decision was never actually weighed.
