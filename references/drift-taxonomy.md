# Drift taxonomy — full catalog

A reference catalog of the drift patterns this skill recognizes. Each entry has: the signature, an example, why it happens, and the recovery move.

When CHECK mode flags drift, it should name the pattern from this list — naming the failure makes it correctable.

---

## 1. Helpful expansion

**Signature:** A response addresses the ask, then adds 1–3 things the user didn't request, framed as bonus value.

**Example:**
> User: "Add a logout button to the navbar."
> Response: "Done. I also added a session-timeout warning, refactored the navbar to use a context provider, and updated the test snapshots."

**Why it happens:** RLHF rewards helpfulness; "more" feels like "better". Claude doesn't see the cost of expansion — it sees the benefit.

**Recovery:** "Revert everything except the logout button. The other changes are out of scope." Then add an explicit NON-GOAL: "Don't refactor unrelated code in the same change."

---

## 2. Best-practice substitution

**Signature:** User asks for X. Response delivers Y because Y is "the right way" — even though Y is more work, more dependencies, or solves a problem the user didn't have.

**Example:**
> User: "Store the auth token in localStorage."
> Response: "I implemented a secure cookie-based session with HttpOnly flags and CSRF protection instead, since localStorage is vulnerable to XSS."

**Why it happens:** Training data over-represents "best practices" advice. The model defaults to the textbook answer over the asked-for answer.

**Recovery:** "I asked for X. If X has a real problem in this context, flag it as a question — don't silently substitute Y." Sometimes the best-practice is genuinely correct, but the substitution should be raised, not assumed.

---

## 3. Defensive over-engineering

**Signature:** Code accumulates try/catch, null checks, validation, or fallback paths for cases that can't happen given the actual call site.

**Example:** A function called once internally with a known-typed object grows runtime type validation, default-value fallbacks, and a try/catch wrapping a synchronous expression that can't throw.

**Why it happens:** Production code in the training set tends to be defensive. The model defaults to that style even when the call site doesn't warrant it.

**Recovery:** Add to the north star's HARD-CONSTRAINTS: "Don't add error handling, fallbacks, or validation for scenarios that can't happen. Trust internal call sites."

---

## 4. Recency hijack

**Signature:** The last 3–5 turns dominate the response, even though they were a side discussion. The model treats them as the spec.

**Example:**
> Turn 1: "Build a bug-tracker."
> Turns 2–8: discussion of how to render code blocks in comments.
> Turn 9 onward: model is now exclusively focused on rich-text rendering, has forgotten about the bug tracker proper.

**Why it happens:** Causal attention weights recent tokens highest. Without re-anchoring, recent topic = current topic.

**Recovery:** Pop back: "We're not building a markdown editor. We're building a bug tracker that happens to render markdown. Confirm the next ticket-list feature."

---

## 5. Demo drift

**Signature:** The model is building the example, the test fixture, or the README's hello-world section, and treating that as the product.

**Example:** Asked to ship a CLI tool. Two hours later there's a beautiful example folder, a polished README, and the actual `bin/` entry point doesn't work end-to-end.

**Why it happens:** Demos and tests are easier to "complete" than production code. The model gravitates toward visible completion.

**Recovery:** "DONE-WHEN is the binary in `bin/` runs end-to-end with real inputs. The demo folder is not the deliverable."

---

## 6. Tool-shaped response

**Signature:** The model does what's easy with the tools available, instead of what was asked. Often: it greps when it should think; it edits when it should ask.

**Example:**
> User: "Why is the cron job missing some runs?"
> Response: greps for "cron" across the repo, lists every cron registration, doesn't actually investigate the timing issue.

**Why it happens:** Tool use has high local value (something happened, output exists, looks like progress). Reasoning without tools doesn't produce visible output.

**Recovery:** "Stop searching. Write down your hypothesis for why runs are missing, then verify the hypothesis specifically."

---

## 7. Premature abstraction

**Signature:** The model generalizes a one-off into a framework before the one-off works. Adds config options, plugin points, or a "system" for what should be 20 lines.

**Example:** Asked to add a Slack notification on deploy. Delivered: a generic notification framework with pluggable backends, an event bus, and a configuration schema — for a single notification.

**Why it happens:** Abstraction looks like sophistication. The model is rewarded for "scalable design" thinking even when scale isn't a requirement.

**Recovery:** "First make the one case work concretely. If we get a second case, then we abstract."

---

## 8. Resolution slip

**Signature:** Decisions are being made at progressively lower levels of detail without confirming the higher levels. By the time you notice, the model is benchmarking compression algorithms for a feature that hasn't been approved.

**Example:**
> "Should we send notifications?" → "Yes" →
> "Push or email?" → "Email" →
> "What template engine?" → "MJML" →
> "How do we handle MJML compilation in the build?" →
> "Should we cache compiled MJML?" →
> "Memcached or Redis for the cache?" → 🛑 stop.

**Why it happens:** Each local decision is rational. The model doesn't track the depth.

**Recovery:** Pop the stack. "We're 5 levels deep on a feature we haven't shipped. Back to: do we send notifications, yes/no, this week or later?"

---

## 9. Goal substitution by clarification

**Signature:** A clarifying question morphs the goal. The user answers narrowly; the model adopts the narrow answer as the new full goal.

**Example:**
> Goal: "Improve onboarding conversion."
> Model: "Should I focus on the signup form?"
> User: "Yeah, start there."
> 10 turns later: every action is signup-form changes; broader funnel work has dropped off the radar.

**Why it happens:** "Yeah, start there" sounds like a direction; the model treats it as the new goal.

**Recovery:** "Signup form is one tactic toward the goal of improving onboarding conversion. Don't collapse the goal to the tactic."

---

## 10. Sympathetic pivot

**Signature:** User expresses mild frustration. Model interprets it as a pivot signal and changes direction, even though the user just wanted acknowledgment.

**Example:**
> User: "Ugh, this is harder than I thought."
> Model: "Want me to try a different approach?"
> User (didn't actually want that): "Sure, I guess."
> 30 minutes later: completely different solution, original work abandoned.

**Why it happens:** RLHF rewards responsiveness to emotional cues.

**Recovery:** Distinguish acknowledgment from instruction. "I hear you. Should I keep going on the current approach or actually pivot?"

---

## How CHECK mode uses this list

When CHECK mode produces its verdict table, the "Note" column should reference one of these patterns by name where applicable:

```
| 2 | Added retry logic with exponential backoff | ⚠️ | defensive over-engineering — not asked, no failure observed |
| 3 | Refactored client to use a generic transport interface | ❌ | premature abstraction — single transport in scope |
```

Naming the pattern is what makes it correctable. "This feels off" is a vibe; "This is premature abstraction" is a thing the model can stop doing.
