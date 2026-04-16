# Onboarding — first-run setup

Runs automatically the first time `/advisor-board` is invoked and `references/user-config.md` does not exist. One-time operation. Takes ~2 minutes.

## Goal

Capture three things from the user, then rewrite the skill's placeholders and write a config file so future invocations are zero-ceremony.

## Step 1 — Ask three questions

Use a single `AskUserQuestion` call with all three questions at once. Language: match the user's language (Polish or English — default to what they've been using in the session).

1. **Name** — "How should the board address you? (Used in 'Recommendation for [name]' and the synthesis.)"
2. **Board roster** — "Use the default 12-member board (Hormozi, Goggins, Naval, Taleb, Taylor Swift, Bryan Johnson, Mark Manson, James Clear, Daniel Priestley, Nick Saraev, Leon Hendrix, Rian Doris) or supply your own 12 names?"
   - Default → proceed with the starter `board-members.md` as-is
   - Custom → ask for the 12 names (comma-separated) in a follow-up question
3. **Context files (optional)** — "Do you have files the board should grep for context on every debate (e.g. values, current projects, strategy notes)? Paste paths relative to the project root, one per line. Write 'skip' if none."

## Step 2 — Generate `references/user-config.md`

Write this file using the user's answers:

```markdown
# User Config

**Name:** [answer to Q1]

**Board members** (canonical order — used for assembly in the debate file):
1. [name]
2. [name]
... 12 total

**Context files** (grep these for the Context Brief; skip if empty):
- [path]
- [path]

**First-run date:** YYYY-MM-DD
```

## Step 3 — Generate `references/board-members.md`

**If default board chosen:** the starter `board-members.md` is already in place — do nothing here, but proceed to Step 4 to substitute `{{USER_NAME}}` inside it.

**If custom board chosen:** overwrite `board-members.md` with profiles for each of the 12 names. For each name, write a 3-5 sentence persona profile using this structure:

```markdown
## [NAME]

[3-5 sentence persona — voice/tone, 2-3 core frameworks they're known for, what they push back on, signature stance. Write it so another agent reading only this section can plausibly debate in character.]

**Worldview:** [one-line distillation]
**Frameworks:** [comma-separated]
**Pushes back on:** [what they don't tolerate]
```

Use {{USER_NAME}} wherever the persona would naturally address the user by name. Step 4 will substitute it.

Order the sections in the same canonical order as `user-config.md`.

## Step 4 — Substitute `{{USER_NAME}}` across the skill

Replace every occurrence of `{{USER_NAME}}` with the real name captured in Step 1, in all three files:

- `.claude/skills/advisor-board/SKILL.md`
- `.claude/skills/advisor-board/references/agent-prompts.md`
- `.claude/skills/advisor-board/references/board-members.md`

This is a one-time operation. After substitution, no `{{USER_NAME}}` placeholders remain, and onboarding will not run again (because `user-config.md` now exists).

## Step 5 — Confirm to the user

Report back: board roster (12 names), number of context files configured, and confirm the skill is ready. Ask whether to kick off the first debate now or wait for the next invocation.

If yes → proceed to Step 0 of the main SKILL.md flow.
If no → end onboarding.

## Notes

- If the user wants to change the board or context files later, they can manually edit `user-config.md` and `board-members.md`, or delete `user-config.md` to re-run onboarding
- Placeholder `{{USER_NAME}}` is used (not `[USER_NAME]`) so it's easy to grep for and substitute without collision with the markdown link syntax
