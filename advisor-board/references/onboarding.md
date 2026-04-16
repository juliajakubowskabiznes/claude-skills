# Onboarding — first-run setup

Runs automatically the first time `/advisor-board` is invoked and `references/user-config.md` does not exist. One-time operation. Takes ~2 minutes.

## Goal

Capture the user's name, decide board size and roster, optional context paths — then rewrite `{{USER_NAME}}` placeholders and write a config file so future invocations are zero-ceremony.

## Step 1 — Ask the questions

Use `AskUserQuestion` calls. Language: match the user's language (Polish or English — default to what they've been using in the session).

### Q1 — Name
"How should the board address you? (Used in 'Recommendation for [name]' and the synthesis.)"

### Q2 — Board size
"How many advisors on the board? The default is **4** (Hormozi, Naval, Goggins, Manson — covers economics, leverage/long-game, execution, values). You can add more, but note the token cost: each extra advisor adds ~15K tokens per debate run. With 12 advisors one full run uses roughly 175K tokens, which is close to the 200K context window on Claude Pro — you may hit limits mid-debate."

Options:
- **4 (default)** — Hormozi, Naval, Goggins, Manson
- **6** — default 4 + Taleb, Clear
- **8** — default 6 + Priestley, Bryan Johnson
- **12 (full roster)** — all starter advisors, highest cost
- **Custom** — user provides own list (any count 4-12)

### Q3 — Context files (optional)
"Do you have files the board should grep for context on every debate (values, current projects, strategy notes)? Paste paths relative to the project root, one per line. Write 'skip' if none."

## Step 2 — Generate `references/user-config.md`

Write this file using the answers:

```markdown
# User Config

**Name:** [Q1 answer]

**Board members** (canonical order — used for assembly in the debate file and in the parallel dispatch):
1. [name]
2. [name]
... [N total, where N ∈ 4..12]

**Context files** (grep these for the Context Brief; empty list if user skipped):
- [path]
- [path]

**First-run date:** YYYY-MM-DD
```

## Step 3 — Generate or trim `references/board-members.md`

The shipped `board-members.md` contains 12 starter profiles. Based on the user's choice in Q2:

- **Default preset (4/6/8/12):** keep the `## [NAME]` sections that are in the user's roster, delete the others. This keeps the file lean.
- **Custom roster:** for any name NOT already in the starter file, write a 3-5 sentence persona profile using this structure:

```markdown
## [NAME]

[3-5 sentences — voice/tone, 2-3 core frameworks they're known for, what they push back on, signature stance. Written so a subagent reading only this section can plausibly debate in character.]

**Worldview:** [one-line distillation]
**Frameworks:** [comma-separated]
**Pushes back on:** [what they don't tolerate]
```

Order sections to match the canonical order in `user-config.md`.

## Step 4 — Substitute `{{USER_NAME}}` across the skill

Replace every occurrence of `{{USER_NAME}}` with the real name captured in Q1, in all three files:

- `.claude/skills/advisor-board/SKILL.md`
- `.claude/skills/advisor-board/references/agent-prompts.md`
- `.claude/skills/advisor-board/references/board-members.md`

One-time operation. After substitution, no `{{USER_NAME}}` placeholders remain, and onboarding will not run again (because `user-config.md` now exists).

## Step 5 — Confirm to the user

Report back: board roster (N names), number of context files configured, and confirm the skill is ready. Ask whether to kick off the first debate now or wait for the next invocation.

If yes → proceed to Step 0 of the main SKILL.md flow.
If no → end onboarding.

## Notes

- To change the board or context files later, edit `user-config.md` and `board-members.md` directly, or delete `user-config.md` to re-run onboarding
- Placeholder `{{USER_NAME}}` is used (not `[USER_NAME]`) so it grep-substitutes cleanly without collision with markdown link syntax
- Token cost per run, rough: 4 advisors ≈ $0.30, 6 ≈ $0.45, 8 ≈ $0.60, 12 ≈ $1.00 (Sonnet pricing)
