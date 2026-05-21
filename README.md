# boris-prompts

An agent skill that writes high-quality prompts for any LLM (Claude, Claude Code, ChatGPT, Gemini, Cursor, Windsurf, etc.) using **Boris's methodology** — the patterns Boris (Anthropic, Claude Code's creator) shared in his "Pro Tips & Tricks" talk.

If your request is unclear, the skill asks 1–3 targeted clarifying questions before writing. The output is short, copy-pasteable, and follows the five principles below.

## Original tweet
https://x.com/Etudecn/status/2057238154701426726

## Install

This skill is distributed via the [`skills`](https://github.com/vercel-labs/skills) CLI (`npx skills`). Works with Claude Code, Codex, Cursor, OpenCode, and 50+ other agents.

```bash
# GitHub shorthand
npx skills add LingyiChen-AI/boris-prompts --skill boris-prompts

# Or full URL
npx skills add https://github.com/LingyiChen-AI/boris-prompts --skill boris-prompts

# Or point at the skill folder directly
npx skills add https://github.com/LingyiChen-AI/boris-prompts/tree/main/skills/boris-prompts

# Install globally (available across all projects)
npx skills add LingyiChen-AI/boris-prompts --skill boris-prompts -g

# Install only for Claude Code
npx skills add LingyiChen-AI/boris-prompts --skill boris-prompts -a claude-code
```

After install, restart your agent (or start a new session) and the skill loads automatically.

## What it does

When you say things like:

- `write a prompt that makes Claude convert all our logs to JSON`
- `how should I prompt Cursor to add dark mode`
- `this ChatGPT prompt isn't working — keeps going off the rails`
- `give me a good way to ask Gemini to refactor this module`

…the skill triggers. If it has enough info, it writes the prompt directly. If not, it asks at most three questions (via `AskUserQuestion`) and then writes it.

## The five principles

The skill applies these in order:

1. **Short beats long.** A two-sentence prompt usually beats a screen-filling one. If you find yourself writing more than five lines, half of it probably belongs in a persistent-context file.
2. **"Make a plan first" is the single highest-ROI addition.** Appending *"Before you write code, make a plan and run it by me for approval"* upgrades almost any non-trivial prompt.
3. **Don't spec every detail.** Point the model at the right starting place ("look at how `X` is implemented") instead of re-describing the codebase.
4. **A feedback loop beats detailed instructions.** Give the model a way to verify (tests, screenshots, lint) and let it iterate, rather than spelling out every step.
5. **Persistent context lives in files / settings, not the prompt.** `CLAUDE.md`, `.cursorrules`, Custom Instructions, Projects, Gems — whichever the target supports.

## Example

**You:** "I want Claude to add JWT auth to my Express app. There are tests."

**Skill output:**

```
Add JWT auth to the Express app. Look at how existing middleware is structured first, then make a plan and run it by me. After implementing, run the test suite and iterate until it passes.
```

> Plan-first + feedback loop applied. Anchors Claude in existing patterns rather than re-describing them.

## Repo layout

This repo follows the convention the `skills` CLI expects: a top-level `skills/` directory containing one folder per skill, each with a `SKILL.md`.

```
boris-prompts/                  (this repo)
└── skills/
    └── boris-prompts/          (the skill)
        └── SKILL.md
```

To add more skills later, drop them in as siblings under `skills/`.

## Credits

Methodology from Boris's "Claude Code Pro Tips" talk (Anthropic). This repo just packages it as an agent skill.

Community: <https://linux.do>

## License

MIT
