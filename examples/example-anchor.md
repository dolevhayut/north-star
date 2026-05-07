# Example — a real `.north-star.md`

This is an actual `.north-star.md` from a shipped project (sanitized). It's here to show what "good" looks like — specific, narrow, with NON-GOALS that defuse the most likely drift.

```markdown
<north-star>
GOAL:
Ship a daily digest email that summarizes the user's open PRs,
in-progress Linear tickets, and unread Slack mentions, sent at 8am
in the user's local timezone.

USER:
Engineering managers at companies of 20–100 engineers who currently
get fragmented status from 3+ tools and start their morning re-
constructing context. Not individual contributors (different problem),
not exec leadership (different cadence).

DONE-WHEN:
A real EM (not us) receives the 8am digest in production for 5
consecutive workdays without manual intervention, and tells us in
their own words that it saved them time. Logged in `data/proof/`.

NON-GOALS:
- WYSIWYG editor for digest content. Templates are code-defined.
- Per-user customization beyond "include / exclude source X".
- Mobile push notifications. Email only for v1.
- Analytics dashboards. Whether the digest is useful is qualitative
  in v1; if it's useful, we instrument later.
- Slack/Teams delivery. Email is the constraint.

HARD-CONSTRAINTS:
- No new database. Use existing Postgres. (Reason: ops budget.)
- Cron driven by GitHub Actions, not a long-running worker.
  (Reason: simpler ops; we're a 2-person team.)
- All API calls cached for 1h. (Reason: rate limits across 4 sources;
  digest is daily so freshness <1h is irrelevant.)
- No PII in logs. (Reason: legal review pre-launch.)
</north-star>
```

## Why this anchor works

**GOAL is specific.** "Daily digest at 8am" not "improve developer productivity". A goal you could imagine writing a test for.

**USER excludes more than it includes.** Naming "not ICs, not execs" is more useful than naming the target — it pre-emptively rejects scope creep into adjacent personas.

**DONE-WHEN is observable.** "Real EM tells us it saved them time, logged in `data/proof/`" — there's a file you can point at. Compare to "users find it useful" (un-falsifiable).

**NON-GOALS list specific things.** Each one is a likely drift vector. "WYSIWYG editor" is a thing the model might propose; naming it as out-of-scope makes the propose-reject loop a single round instead of a 5-turn discussion.

**HARD-CONSTRAINTS each have a reason.** "No new database (reason: ops budget)" lets the model judge edge cases. If a reasonable-looking case for a new database comes up, the model now has the *reason* to weigh — not just a rule to obey.

## What it explicitly doesn't include

- Implementation strategy
- Tech stack details
- Architecture decisions
- Timeline

Those are *downstream* of the north star. They belong in the project, not the anchor. The anchor is what stays stable while implementation choices change.

## Anti-pattern: the over-broad anchor

Don't write this:
```
GOAL: build a great product that delights users.
```
This anchors nothing. Every drift is "consistent" with a goal this vague.

## Anti-pattern: the implementation anchor

Don't write this:
```
GOAL: a Next.js 16 app on Vercel using Postgres for the data layer
with TanStack Query on the frontend, Tailwind for styling, NextAuth
for auth, and Trigger.dev for background jobs.
```
This is a tech stack, not a goal. If Postgres turns out wrong for the use case, you can't change it — the anchor commits you. Keep the anchor at the "what changes for the user" level.

## Anti-pattern: the laundry list

Don't write this:
```
GOAL: a daily digest, plus a weekly report, plus a Slack integration,
plus mobile push, plus an admin dashboard, plus analytics, plus...
```
A goal with five components is five goals. Pick the one that, if shipped, would matter most. The rest is a roadmap, not an anchor.
