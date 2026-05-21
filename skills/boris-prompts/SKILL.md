---
name: boris-prompts
description: Write a high-quality prompt for any LLM or AI assistant — Claude, Claude Code, ChatGPT, Gemini, Cursor, Windsurf, Copilot, or any coding / chat agent. Use this skill whenever the user asks to write, improve, refine, shorten, or rewrite a prompt; asks "how should I phrase this for [model]" or "what's a good prompt for [task]"; describes a task they want an AI to do but hasn't yet formulated it as a prompt; or pastes an existing prompt and asks for revision. Based on Boris's (Anthropic, Claude Code creator) prompt methodology — short and accurate prompts, plan-before-code, feedback loops, persistent context in files. The universal principles (short, plan-first, feedback-loop, no-padding) apply to any LLM; the Claude-Code-specific anchors (CLAUDE.md, @file, slash commands) only apply when the target is Claude Code. If the user's intent is unclear (target model, deliverable, scope, or whether the AI has a way to self-verify is missing), ask 1–3 targeted clarifying questions via AskUserQuestion before writing the prompt.
---

# Boris-Style Prompt Writer

Write high-quality prompts for any LLM or AI assistant, using the methodology Boris (Anthropic, the creator of Claude Code) shared in his "Pro Tips & Tricks" talk.

The skill exists because most users write prompts that are too long, too prescriptive, and missing the things that actually matter — a feedback loop, a planning step, the right anchor. Boris's approach inverts this: keep the prompt short, let the model explore, give it a way to check its own work, and sink reusable context into files rather than re-pasting it. The principles were developed for Claude Code, but they generalize to any modern LLM (ChatGPT, Gemini, Cursor, etc.).

## When to use

Trigger this skill when **any** of the following is true:

- The user explicitly asks to write or improve a prompt for any LLM or AI assistant.
- The user describes a task in natural language and the next reasonable step is to phrase it as a prompt.
- The user pastes a long prompt and asks why it isn't working, or asks to shorten or refine it.
- The user asks "how should I phrase this" or "what's a good way to ask [model] to...".

Do not trigger for: writing marketing copy, writing user-facing UI text, or writing prompts for image-generation models. This skill is specifically for **instruction prompts** addressed to an LLM or AI agent.

## Target model — universal vs. Claude-Code-specific advice

Before writing, identify what the user is prompting:

- **Claude Code**: full toolkit applies — `CLAUDE.md`, `@file` references, slash commands, the `commit, push, PR` shorthand, plan mode by natural language.
- **Claude (claude.ai, API)**: most principles apply, but no `@file` or `CLAUDE.md`. Use system prompts / Projects for persistent context.
- **ChatGPT / Gemini / other chat LLMs**: the universal principles apply (short, plan-first, feedback-loop, no padding). Replace `CLAUDE.md` with the model's equivalent (Custom Instructions, system prompt, Projects, Gems, etc.). The "explore the codebase" pattern only works if the model has tools to do so.
- **Cursor / Windsurf / Copilot Chat**: editor-integrated; `@file` works (different syntax), per-project rules files work, plan-first works.
- **Agent frameworks (autonomous tools)**: emphasize the feedback loop heavily — these run without supervision.

If the target isn't obvious from context and it materially changes the prompt, ask.

## Core principles (in priority order)

These come straight from Boris's talk. Apply them in this order when writing or evaluating a prompt.

### 1. Short beats long

A prompt that fits in two sentences usually works better than one that fills the screen. The reason isn't aesthetic — it's that a long prompt eats the context window, repeats things Claude already knows, and over-constrains the solution space. If you find yourself writing more than ~5 lines, ask whether half of it belongs in `CLAUDE.md` instead.

### 2. "Make a plan first" is the single highest-ROI addition

The most reliable upgrade you can make to almost any non-trivial prompt is appending some form of:

> "Before you write code, make a plan and run it by me for approval."

This works because the model is genuinely good at planning when asked to, and most bad results come from skipping the planning step. You do not need plan mode, a special flag, or a system prompt — the plain-English instruction is enough.

### 3. Don't spec every detail — let Claude explore

If the task involves an existing codebase, the prompt should usually point Claude at the right starting place ("look at how `X` is implemented", "check git history for why this function has 15 args") rather than describing the full solution. Claude is good at finding patterns and similar code; it's wasteful to re-describe what the codebase already contains.

### 4. A feedback loop beats detailed instructions

When the deliverable is verifiable (UI matches a mock, tests pass, lint passes, screenshot is similar), the prompt should give Claude **a way to verify** rather than telling it how to do every step. The pattern:

> "Do X. Then run [tests / take a screenshot / lint]. Iterate until [success criterion]."

This typically beats a multi-paragraph spec because the model can self-correct using ground truth.

### 5. Persistent context goes in files / settings, not prompts

If you find yourself pasting the same architecture notes, style rules, or common commands into every prompt, that's a signal — those belong in the target's persistent-context mechanism:

- **Claude Code**: `./CLAUDE.md` (project), `./CLAUDE.local.md` (personal), `~/.claude/CLAUDE.md` (global), nested `CLAUDE.md` (subdirectory).
- **Claude.ai**: Projects, system prompts.
- **ChatGPT**: Custom Instructions, Projects, system prompts.
- **Gemini**: Gems, system instructions.
- **Cursor / Windsurf**: `.cursorrules` / `.windsurfrules`, per-project rules.
- **API usage**: system prompt, fixed context block.

The prompt should be about *this specific task*, not background.

## Workflow

### Step 1 — Decide if you have enough information

Before writing the prompt, you need to be able to answer these four questions. If any is unclear, stop and ask the user.

| Question | Why it matters | What it changes |
|---|---|---|
| Which model / assistant is this prompt for? (Claude Code, Claude.ai, ChatGPT, Cursor, Gemini, etc.) | Determines which model-specific features (@file, CLAUDE.md, Projects, etc.) are available | Vocabulary and anchors |
| What is the deliverable? (Q&A / small edit / new feature / bug fix / refactor / debug / non-code task) | Determines which template to use | Template choice |
| Can the assistant verify its own output? (tests, screenshots, lint, build, manual check by user) | Determines whether to add an iteration loop | The "run X and iterate" line |
| Is this one-shot, or should the assistant plan first? | Most non-trivial tasks benefit from a plan; pure Q&A does not | The "make a plan first" line |
| Are there existing files, patterns, or persistent-context files (`CLAUDE.md`, `.cursorrules`, etc.) the assistant should anchor on? | Determines whether to add file references or "look at how X works" | Context anchors |

### Step 2 — If anything is unclear, ask 1–3 clarifying questions

Use the `AskUserQuestion` tool. **Cap at 3 questions.** Beyond that the user feels interrogated and the skill's benefit (saving them time) disappears.

Choose questions from this menu based on what's actually unclear — don't ask things you already know from context.

**If target model / assistant is unclear and would change the answer:**
- Question: "Which assistant is this prompt for?"
- Options: Claude Code (CLI) / Claude.ai (web/desktop) / ChatGPT / Cursor or Windsurf / Other (say which)

**If deliverable is unclear:**
- Question: "What do you want Claude to do?"
- Options: Answer a question about the code / Make a small edit / Build a new feature / Fix a bug / Refactor existing code

**If verification is unclear:**
- Question: "Can Claude check its own work after each attempt?"
- Options: Yes — there are tests / Yes — visual (I'll diff screenshots) / Yes — a CLI / build / lint will tell us / No — I'll review manually

**If scope is unclear:**
- Question: "Roughly how big is this task?"
- Options: A single line or two / A few files / A whole module or feature / I don't know — that's what I want Claude to figure out

**If planning preference is unclear (only ask if deliverable is small edit or larger):**
- Question: "Want Claude to propose a plan first, or just try it?"
- Options: Plan first, then approve / Just try it, I'll course-correct / Plan only — no code yet

**If codebase anchors are unclear:**
- Question: "Are there existing files or patterns Claude should look at first?"
- Options: Yes — I'll @-mention them / Look at git history / Look at similar features — Claude can find them / No, this is greenfield

### Step 3 — Pick a template and fill it in

Use the closest matching template from this set. These mirror the patterns Boris uses in his own daily work.

#### Template A — Codebase Q&A

```
[Question]. Look through git history if it helps.
```

Examples:
- `Why does parseConfig take 12 arguments? Look through git history.`
- `How is the RateLimiter class actually used? Show me real call sites, not just text matches.`
- `What did I ship this week? Use my git username from the log.`

#### Template B — Small edit (1 file, clear change)

```
[Task]. Make the change, then [verify].
```

Examples:
- `Rename the env var DB_URL to DATABASE_URL everywhere it's used. After changing files, run the test suite.`
- `Add a "Cancel" button next to "Save" in @components/UserForm.tsx. Match the existing button styles.`

#### Template C — Medium edit, plan-first

```
[Task]. Make a plan and run it by me before writing code.
```

Examples:
- `Replace the in-memory rate limiter with Redis. Plan first, then ask before changing files.`
- `The auth middleware is leaking session tokens in logs — fix it. Make a plan first.`

#### Template D — Build / UI work with feedback loop

```
[Task]. Use [feedback tool] to check after each change. Iterate until [success criterion].
```

Examples:
- `Implement the dark theme to match @design/dark-mock.png. Use puppeteer to screenshot the page, diff against the mock, iterate until visually close.`
- `Build the /pricing page from @design/pricing.fig export. Screenshot at desktop + mobile widths after each change. Iterate until both match.`
- `Make the failing tests in @tests/checkout.test.ts pass. Run the test suite after each change.`

#### Template E — Bigger feature, explore + plan

```
[Feature]. First explore the codebase to find similar patterns. Then make a plan and ask for approval. After implementing, run [verification].
```

Example:
- `Add an audit log for all admin actions. First look at how existing logging works in the codebase, then propose a plan, ask before writing code. Once implemented, run the test suite.`

#### Template F — Git / GitHub operation

```
commit, push, PR
```

Literally those words. Claude reads `git log` to match the project's commit format, creates a branch if needed, pushes, and opens a PR. Don't over-specify.

#### Template G — Bug fix (root cause, not symptom)

```
[Bug description, including how to reproduce]. Find the root cause, don't just patch the symptom. Make a plan first.
```

### Step 4 — Add the optional power-ups (when applicable)

These small additions punch above their weight:

| Add this | When |
|---|---|
| `@path/to/file.ts` | Whenever a specific file is the anchor for the task. |
| `Look through git history` | When the "why" of existing code matters. |
| `Don't change [X]` | When you want to mark a clear no-go zone (more effective than positively specifying what to change). |

### Step 5 — Output

Return two things to the user:

1. **The final prompt**, in a fenced code block so it's copy-pasteable.
2. **A one-line note** explaining why the prompt is structured this way (which 1–2 principles you applied).

If applicable, add a brief tip:
- If the prompt references the same architecture / style facts twice or more, suggest moving them to `CLAUDE.md`.
- If the user is about to do many similar prompts, suggest writing a `/slash-command` instead.

## Output examples

These show what the skill produces. Notice the prompts are short and the explanation is one sentence.

### Example 1 — User input was complete

User: "I want Claude Code to add JWT auth to my Express app. There are tests."

Skill output:

```
Add JWT auth to the Express app. Look at how existing middleware is structured first, then make a plan and run it by me. After implementing, run the test suite and iterate until it passes.
```

> Uses Template E (medium-size feature, explore + plan + feedback loop) because there's a test suite to verify against.

### Example 2 — User input was vague, skill asked

User: "help me prompt Claude to fix my CSS"

Skill asks:
- "What's broken in the CSS?" — options: Layout is off / Colors wrong / Responsive issues / I don't know — Claude should figure it out
- "Can Claude check the result visually?" — options: Yes, screenshot it / Yes, I'll eyeball it / I have visual regression tests

After user answers (e.g., "layout is off on mobile" + "yes screenshot it"):

```
The mobile layout on @pages/Home.tsx is broken. Screenshot it at 375px wide using puppeteer, find what's off, fix it, and re-screenshot. Iterate until the layout looks right.
```

> Uses Template D (feedback loop) because puppeteer gives Claude a way to self-verify.

### Example 3 — Existing prompt, asked to improve

User: *pastes a 30-line prompt full of architecture details and style rules*

Skill response: identify the parts that are reusable context (architecture, style, common commands) and suggest moving them to `CLAUDE.md`, leaving the prompt itself focused on the task.

```
Implement the payment retry logic described in @docs/retry-spec.md. Plan first, ask before writing code, then run the test suite.
```

> Original prompt was 30 lines; 25 of them were repeated architecture / style context that should live in `CLAUDE.md`. Kept only the task-specific part.

## Anti-patterns (don't do these)

- Polite filler ("Please could you help me with..."). It wastes context and the model doesn't care. Use a direct verb.
- Pasting the same architecture / style notes into every prompt. Put them in `CLAUDE.md` once.
- Long lists of step-by-step instructions for tasks that have a verification signal. Give it the verification signal instead and let it iterate.
- Asking for a 3000-line feature in one shot. Ask for a plan, review, then implement in stages.
- Generating a 200-line "perfect prompt" to save the user time. The whole point of the skill is *shorter* prompts. If your output is longer than the user's question, you're doing it wrong.

## When this skill should hand off

This skill writes the prompt — it doesn't run it. After producing the prompt:

- The user usually copies it into Claude Code themselves.
- If they ask "now run it", that becomes a separate task and this skill is done.
