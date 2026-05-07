# North Star — כוכב הצפון

> A Claude Code skill that detects and corrects goal drift in long sessions.

Long Claude sessions drift. You start with a clear goal, iterate a few times, and twenty turns later the model is solving a slightly different problem than the one you came with. North Star pulls it back.

## What it does

Three modes:

- **ANCHOR** — at the start of a session, write a `<north-star>` block to `.north-star.md` that defines what "done" looks like.
- **CHECK** — mid-session, audit the last few decisions against the anchor and surface drift in a structured table.
- **RECOVERY** — when the user signals "we lost the plot", restate the anchor and propose a single corrective step.

It also catalogs ten specific drift patterns (helpful expansion, premature abstraction, recency hijack, …) so the skill can name what went wrong, not just flag that something did.

## Why it works

Goal drift in LLMs is structural — recency bias, attention dilution across long contexts, RLHF agreement pressure, lossy compaction, helpfulness-as-scope-creep. This skill doesn't prevent drift; it re-injects the goal into the recent context where attention is highest, and forces a structured ✅/⚠️/❌ verdict that fluent prose can't paper over.

Full mechanism: [`references/why-llms-drift.md`](references/why-llms-drift.md).

## Install

**Interactive (pick your agent):**
```bash
npx skills add dolevhayut/north-star
```

**Global — all agents, no prompts:**
```bash
npx skills add dolevhayut/north-star --yes --global
```

**Project-local — all agents, no prompts:**
```bash
npx skills add dolevhayut/north-star --yes
```

The CLI auto-detects which agents you use. The agents below are supported — use whichever matches your setup:

| Agent | Install path |
|---|---|
| Amp, Antigravity, Cline, Codex, Cursor, Gemini CLI, GitHub Copilot, Warp, and more | `.agents/skills` (universal, always included) |
| Claude Code | `.claude/skills` |
| Aider | `.aider-desk/skills` |
| Augment | `.augment/skills` |

Invoke with `/north-star` or by asking Claude to "run a north star check".

## Use

**Start of project:**

```
/north-star

What's the goal?
```

Claude walks you through writing `.north-star.md`. Takes about 60 seconds.

**Mid-project, when something feels off:**

```
/north-star
```

Claude reads `.north-star.md`, audits the last 3–5 decisions, returns a verdict table.

**Automatic re-injection** (recommended for sessions >20 turns):

Configure a `UserPromptSubmit` hook that prepends `.north-star.md` to every prompt — see [`references/automation.md`](references/automation.md).

## Repo layout

```
north-star/
├── SKILL.md                          # entry point — what the skill does and how
├── README.md                         # this file
├── references/
│   ├── why-llms-drift.md             # the mechanics: why drift is structural
│   ├── drift-taxonomy.md             # 10 drift patterns + recovery moves
│   ├── anchoring-techniques.md       # XML wrapping, file-based, hooks, sub-agents
│   └── automation.md                 # wiring this into UserPromptSubmit hooks
└── examples/
    ├── example-anchor.md             # a real, shipped .north-star.md
    └── example-check.md              # a real CHECK response on a drifted session
```

## License

MIT.

## Author

[Dolev Hayut](https://www.linkedin.com/in/dolevhayut) — Bulldog Media Solutions.

Built because I was tired of explaining the same project goal to Claude for the third time in one session.

## Contributing

Issues and PRs welcome. Especially:
- New drift patterns observed in the wild (with sanitized examples)
- Hook patterns for harnesses other than Claude Code
- Translations of the skill into other languages
