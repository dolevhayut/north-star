# Example — CHECK mode response on a session that drifted

A real CHECK pass from a session that started clean and drifted over ~25 turns. Sanitized but otherwise verbatim.

## Setup

The project's `.north-star.md`:

```
<north-star>
GOAL: Ship a CLI that lints commit messages against the team's convention
      and blocks the commit on violations.
DONE-WHEN: `git commit` with a non-conformant message exits 1; conformant
           message commits cleanly. Verified on macOS and Linux CI.
NON-GOALS:
- A web UI for viewing past commits.
- Configurable per-repo conventions in v1 (one hardcoded convention).
- Anything beyond the pre-commit hook.
HARD-CONSTRAINTS:
- No external dependencies in the runtime binary. (Reason: install
  friction for end users.)
</north-star>
```

The session ran for 25 turns. By turn 23, the user said "wait, why are we benchmarking this again?" — that's when CHECK was invoked.

## CHECK mode response

```
## 🧭 North Star Check

<north-star>
GOAL: Ship a CLI that lints commit messages against the team's convention
      and blocks the commit on violations.
DONE-WHEN: `git commit` with a non-conformant message exits 1; conformant
           message commits cleanly. Verified on macOS and Linux CI.
</north-star>

### Last decisions

| # | Decision | Verdict | Note |
|---|---|---|---|
| 1 | Implemented core lint rule for conventional-commits format | ✅ | direct on goal |
| 2 | Wired up `pre-commit` git hook installation | ✅ | required for DONE-WHEN |
| 3 | Added a `--fix` mode that rewrites the commit message | ⚠️ | helpful expansion: not requested, expands surface area |
| 4 | Refactored linter into pluggable rule architecture | ❌ | premature abstraction — single hardcoded convention is in NON-GOALS |
| 5 | Started benchmarking rule-evaluation performance | ❌ | resolution slip — performance was never raised as a concern |

### Drift detected

Two patterns active simultaneously:
- **Premature abstraction** (decision 4): the pluggable rule architecture serves "configurable per-repo conventions" — explicitly a NON-GOAL.
- **Resolution slip** (decision 5): performance benchmarking is two steps below the abstraction it benchmarks; the abstraction itself isn't in scope.

### Realignment

Single concrete next action: **revert decisions 4 and 5** (drop the rule-engine abstraction, drop the benchmarks). Keep decisions 1–3. Then verify DONE-WHEN by running `git commit` with a conformant and non-conformant message on a fresh clone.

Decision 3 (`--fix`) is ⚠️ not ❌ — it's adjacent value. Recommendation: keep the code, ship without exposing the flag. Re-evaluate after DONE-WHEN is met.

### Re-anchor

> **GOAL:** Ship a CLI that lints commit messages against the team's convention and blocks the commit on violations.
> **DONE-WHEN:** `git commit` with a non-conformant message exits 1; conformant message commits cleanly. Verified on macOS and Linux CI.
```

## What the user did with this

The user replied: "Revert 4 and 5. Keep 3 hidden. Run the DONE-WHEN check."

DONE-WHEN was met that turn. The session ended on turn 26, instead of the trajectory it was on (an open-ended performance-tuning detour with no obvious end).

## Why the verdict table is the load-bearing piece

Without the table, the model could write a paragraph that smooths over both drifts: "I built the lint rule, added a fix mode, and put together a flexible architecture for future rules with good performance." That paragraph sounds fine. It hides the problem.

The table forces a binary judgment per decision. ❌ on the abstraction is a label that can't be smoothed. The user can act on it.

## When CHECK should NOT recommend reverting

CHECK is a detection tool, not a destruction tool. There are cases where drift exists but reverting is wrong:
- The "drift" turned out to reveal a real problem with the original goal — surface that to the user, don't auto-revert.
- The drift is far enough back that downstream code depends on it — propose a path forward, not a path backward.
- The user explicitly broadened the goal mid-session and didn't update the anchor — surface the discrepancy, ask whether to update the anchor or revert the work.

CHECK's job is to make the discrepancy visible. The user picks the resolution.
