# Anchoring techniques

Concrete techniques for keeping the north star high-attention across a long session. Ranked from cheapest to most invasive.

## Tier 1 — passive (no setup)

### XML-tagged block

Wrap the goal in `<north-star>...</north-star>`. Claude treats XML tags as elevated structural signals; they survive paraphrase better than free prose.

```
<north-star>
GOAL: Ship a daily digest email by Friday.
DONE-WHEN: production cron runs end-to-end and one real user receives a digest.
NON-GOALS: editor for digest content; per-user customization; analytics.
</north-star>
```

### End-of-response re-quote

In any response that risks drift, end by re-quoting the GOAL line. The last 200 tokens of context have outsized attention weight on the next generation.

### Explicit drift labeling

When you detect adjacent work, label it in your own message: "I'm exploring X — this is exploration, not the main task." The label is now in conversation context for future turns to reference.

## Tier 2 — file-based (one-time setup per project)

### `.north-star.md` at repo root

A small file at the project root containing only the `<north-star>` block. The skill reads this as its source of truth in CHECK mode.

```
.north-star.md
---
<north-star>
GOAL: …
USER: …
DONE-WHEN: …
NON-GOALS:
- …
- …
HARD-CONSTRAINTS:
- … (reason: …)
</north-star>
```

Survives compaction (files aren't summarized). Survives session boundaries. Easy to grep, easy to edit.

### Embed in `CLAUDE.md`

If the project already has a `CLAUDE.md`, add a `## North Star` section near the top. `CLAUDE.md` is auto-loaded into Claude Code system context every session.

```markdown
# Project conventions

## North Star
<north-star>
GOAL: …
DONE-WHEN: …
NON-GOALS: …
</north-star>

## Other conventions
…
```

## Tier 3 — active (hooks)

### `UserPromptSubmit` hook re-injection

Configure a hook that, on every user prompt, prepends the contents of `.north-star.md` as `additionalContext`. The goal is now at the *most recent* position in context for every turn — maximum attention weight.

See [`automation.md`](automation.md) for the exact hook configuration.

### Periodic CHECK auto-trigger

A hook that runs CHECK mode automatically every N turns (configurable). The user doesn't have to remember to invoke it; drift surfaces on its own.

See [`automation.md`](automation.md).

## Tier 4 — sub-agent isolation

### Pass north star explicitly to sub-agents

Sub-agents (`Task` tool, parallel agents) start with **no conversation history**. They see only the prompt you give them. So:

**Wrong:**
```
Task(prompt: "Investigate the auth flow and find the bug")
```

**Right:**
```
Task(prompt: """
<north-star>
GOAL: Ship a daily digest email by Friday.
DONE-WHEN: production cron runs end-to-end.
</north-star>

Within that goal: investigate why the cron's auth call is failing in staging.
Constraint: read-only investigation. Do not modify code.
""")
```

The sub-agent now has the same anchor as the parent session.

### Goal-narrowing for sub-agents

Sub-agent goals should be *narrower* than the parent goal — a slice of it. If the parent's GOAL is "ship the digest by Friday", the sub-agent's prompt north star is "diagnose the cron auth failure" with the parent goal quoted as upstream context.

## Tier 5 — discipline (the human side)

The model can be re-anchored. The human can also create drift. Some drift is operator-side, not model-side:

- **Pivoting silently.** If the goal changes mid-session, *update `.north-star.md`* — don't just keep going. The model is now anchored to the wrong goal and will fight you.
- **Chasing tangents.** If you spend 10 turns exploring an interesting side question, the model will absorb the tangent. Either label it as exploration in your message, or branch off into a separate session.
- **"Just one more thing".** Same as helpful expansion, but operator-driven. Each "just one more" extends the session past the point of useful focus. Ship the original ask, then start a new session for the next thing.

## Picking a tier

| Situation | Recommended tier |
|---|---|
| One-off short session | Tier 1 (passive) |
| Multi-day project | Tier 2 (`.north-star.md`) |
| Project where you constantly re-explain | Tier 3 (hooks) |
| Heavy multi-agent / parallel work | Tier 4 (sub-agent passing) |
| You keep pivoting | Tier 5 (operator discipline) |

The tiers compose. Most real projects end up at Tier 2 + Tier 4.
