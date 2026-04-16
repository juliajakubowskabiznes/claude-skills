# Onboarding — first-run setup

Runs automatically the first time `/advisor-board` is invoked and `references/user-config.md` does not exist. One-time operation. Takes ~2 minutes.

## Goal

Capture the user's name, build a personal board roster (the user picks who), record optional context paths, then rewrite `{{USER_NAME}}` placeholders and save a config file so future invocations are zero-ceremony.

## Step 1 — Ask the questions

Use `AskUserQuestion` calls. Language: match the user's language (Polish or English — default to whatever they've been using in this session).

### Q1 — Name
"How should the board address you? (Used in 'Recommendation for [name]' and the synthesis.)"

### Q2 — Board size
"How many advisors do you want on your board? Recommended: **4**. You can pick more, but each extra advisor adds ~15K tokens per debate. Cost guide on Sonnet:

| Advisors | Tokens / run | Approx cost |
|---|---|---|
| 4 | ~60K | ~$0.30 |
| 6 | ~90K | ~$0.45 |
| 8 | ~120K | ~$0.60 |
| 12 | ~175K | ~$1.00 (close to 200K Pro context limit) |

Type a number 4-12."

### Q3 — Who's on the board
"List the **N** people you want on your board. These can be anyone — entrepreneurs, thinkers, athletes, writers, fictional characters, mentors you admire — as long as they have a recognizable worldview, voice, and frameworks people associate with them.

Tip: pick advisors with **different worldviews**, not 4 versions of the same person. A board of 4 founders all saying 'ship faster' is useless. A board of 1 economist + 1 long-term thinker + 1 BS-caller + 1 values-questioner gives real friction.

Inspiration (if you want a starting point): Alex Hormozi, Naval Ravikant, David Goggins, Mark Manson, Nassim Taleb, James Clear, Daniel Priestley, Taylor Swift, Bryan Johnson, Nick Saraev, Leon Hendrix, Rian Doris — but feel free to pick anyone.

Give names comma-separated."

### Q4 — Context files (optional)
"Do you have files the board should grep for context on every debate (values, current projects, strategy notes)? Paste paths relative to the project root, one per line. Write 'skip' if none."

## Step 2 — Generate `references/user-config.md`

Write this file using the answers:

```markdown
# User Config

**Name:** [Q1 answer]

**Board members** (canonical order — used for assembly in the debate file and parallel dispatch):
1. [name]
2. [name]
... [N total, where N = answer to Q2]

**Context files** (grep these for the Context Brief; empty list if user skipped):
- [path]
- [path]

**First-run date:** YYYY-MM-DD
```

## Step 3 — Generate `references/board-members.md`

Build the file from scratch with one section per advisor in `user-config.md`. For each name, write a 3-5 sentence persona profile:

```markdown
## [NAME]

[3-5 sentences — voice/tone (sharp/warm/analytical/crude), 2-3 core frameworks they're known for, what they push back on, signature stance. Written so a subagent reading only this section can plausibly debate in character.]

**Worldview:** [one-line distillation]
**Frameworks:** [comma-separated]
**Pushes back on:** [what they don't tolerate]
```

Use {{USER_NAME}} wherever the persona would naturally address the user by name (Step 4 substitutes it).

Order sections to match the canonical order in `user-config.md`.

**For well-known names** (entrepreneurs, public thinkers, athletes), write the profile from your training knowledge of their public work. **For obscure names**, ask the user 1-2 quick questions about what they associate with that person before writing the section.

## Step 4 — Substitute `{{USER_NAME}}` across the skill

Replace every occurrence of `{{USER_NAME}}` with the real name from Q1, in:

- `.claude/skills/advisor-board/SKILL.md`
- `.claude/skills/advisor-board/references/agent-prompts.md`
- `.claude/skills/advisor-board/references/board-members.md`

One-time operation. After substitution, no `{{USER_NAME}}` placeholders remain, and onboarding will not run again (because `user-config.md` now exists).

## Step 5 — Confirm to the user

Report back: board roster (N names), number of context files configured, and confirm the skill is ready. Ask whether to kick off the first debate now or wait for the next invocation.

If yes → proceed to Step 0 of the main SKILL.md flow.
If no → end onboarding.

## Notes

- To change the board or context files later: edit `user-config.md` and `board-members.md` directly, or delete `user-config.md` to re-run onboarding from scratch
- Placeholder `{{USER_NAME}}` (not `[USER_NAME]`) is used because it grep-substitutes cleanly without colliding with markdown link syntax
- The shipped `board-members.md` contains 12 starter profiles as inspiration; onboarding overwrites it with the user's actual choices
