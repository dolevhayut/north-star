# Why LLMs drift — the mechanics

This is the technical background for why goal drift in long Claude sessions is structural, not a bug. Understanding the mechanism is what lets the skill counter it deliberately.

## 1. Attention is a softmax over the whole context

Every token in the context competes for attention weight. The total weight sums to 1. Adding more tokens dilutes everything that was already there.

A goal stated in a 200-token system message has roughly 1.0 share of "goal-relevant" attention when context is small. By turn 40 in a session with 30,000 tokens of conversation, that same 200 tokens has on the order of 0.7% share — and most of the new tokens are talking about whatever you've been working on for the last 10 turns.

The original goal didn't go away. It just got out-shouted.

**Counter:** Re-quote the goal in the most recent turn. Recent tokens get the highest attention weight in autoregressive decoding because they're the closest neighbors to the next token being generated.

## 2. Recency bias in causal attention

Transformer LLMs use causal attention — each token can only attend to itself and earlier tokens. But the *position* of attention is not uniform. Empirically, models attend most strongly to:

- The very start of the context (anchoring tokens)
- The very end of the context (the immediate prompt)

The middle is the weakest. This is the "lost in the middle" effect documented by Liu et al. (2023). For a 50-turn session, turn 1 (where you stated the goal) and turn 50 (the current question) both get heavy attention. Turns 5–45 — including any goal restatements you did mid-session — get less.

**Counter:** Either keep the goal at the very start (system prompt / `CLAUDE.md` / pinned message) or re-inject it at the very end (re-quote in the latest assistant turn).

## 3. RLHF agreement pressure

Claude is fine-tuned with RLHF. Among the things RLHF rewards: agreeing with the user's framing, extending the user's line of thought, treating the most recent question as the topic.

This is mostly desirable. The side-effect: if the user spent 8 turns exploring a tangent, the model now treats the tangent as the actual project. The tangent isn't labeled "tangent" in the conversation — it's just "the recent topic", which RLHF says is what you should engage with.

**Counter:** Explicitly label tangents while you're on them. "This is exploration, not the main task" — written in your message — survives as context the model can attend to. Without the label, the model can't distinguish exploration from drift.

## 4. Compaction is lossy paraphrase

When Claude Code auto-compacts a long session, the summary it generates is a paraphrase. Paraphrases preserve gist but lose specificity. Compare:

- Original turn 1: "Build this in Bun, not Node. We had ESM compatibility issues with the Sentry SDK in Node and it cost us a day. Bun is non-negotiable for this codebase."
- After compaction: "User prefers Bun runtime."

The "we tried this and it broke" reason is gone. So is "non-negotiable". Two compactions later, the model proposes adding `node-fetch` and is surprised when the user pushes back.

**Counter:** Before compaction, write hard constraints to a file (`.north-star.md`, `CLAUDE.md`). Files survive compaction unchanged.

## 5. Helpfulness as scope creep

Claude is trained to be helpful. "Helpful" includes noticing adjacent issues and offering to fix them. From inside a single response, this looks like good citizenship: "I noticed your auth check has a race condition, want me to also fix that?"

From outside — across 20 turns — this is scope expansion. Three "while I'm here" fixes per session, over a 30-session project, is a different project.

**Counter:** A NON-GOALS list. Naming what's *not* in scope explicitly is more effective than just naming what *is* in scope, because the model has to actively check before adding.

## 6. Resolution drift

A goal at high abstraction ("ship a chat app") gets refined through decisions:
- "Use SSE" (mid-level)
- "Custom event format" (lower)
- "Buffer the last 50 events for resume" (lower)
- "Compress the buffer" (lower)
- "Benchmark Snappy vs LZ4" (lowest — and four steps from "ship a chat app")

Each step felt locally rational. Five steps deep, you're optimizing a benchmark that has nothing to do with shipping a chat app.

**Counter:** Periodically pop back to the top. CHECK mode in this skill forces a re-grounding at the goal level — does this branch still serve the trunk?

## How the skill exploits these mechanics

| Mechanic | What this skill does |
|---|---|
| Attention softmax | Re-quotes north star at end of every CHECK response |
| Recency bias | Wraps north star in `<north-star>` XML — Claude treats tagged blocks as elevated |
| RLHF agreement | Forces a structured ✅/⚠️/❌ judgment that can't be smoothed over with prose |
| Compaction loss | Stores north star in a file, not just conversation |
| Scope creep | NON-GOALS list as a defensive perimeter |
| Resolution drift | Forces popping back to GOAL on every CHECK |

The skill isn't fighting Claude's nature — it's redirecting it.
