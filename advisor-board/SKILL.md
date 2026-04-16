---
name: advisor-board
description: |
  Convenes a virtual advisory board of 12 personas to debate a business or life decision across 3 rounds of parallel subagents. Each agent reads their persona profile, gives an unfiltered take, engages directly with other members, then lands a final recommendation. Output: one debate file with framed question, all rounds, and a synthesis that preserves real disagreement. Use whenever the user brings a decision, strategic question, dilemma, or situation they want stress-tested — especially when they say "convene the board", "ask the board", "board session", "stress-test this", or invoke "/advisor-board [question]". First run walks the user through a one-time setup (name, board roster, optional context files) and self-configures the skill.
model: sonnet
context: fork
disable-model-invocation: true
---

# Advisor Board

12 persona subagents debate a question across 3 rounds. Clarity over comfort.

## Step -1 — First-run check

If `references/user-config.md` does NOT exist → read and follow [references/onboarding.md](references/onboarding.md) before anything else. Onboarding captures the user's name, confirms the board roster, records optional context paths, and self-updates `{{USER_NAME}}` placeholders throughout the skill.

If `references/user-config.md` exists → read it to get the user's name, board order, and context paths. Proceed.

## Step 0 — Context Extraction

Do not assume anything is loaded. Read the question first, then grep the user's context files (from `user-config.md`) for keywords that match. Pull matching sections only — never full files.

Produce a **Context Brief** (max 300 words): values/red lines relevant to the decision, current-state items directly involved, strategy elements that apply, behavioral notes only if they explain a pattern in the question, prior attempts if any.

If the user has no context files configured, the Context Brief is built from the question + any conversation history.

## Step 1 — Frame the Question

Present this to {{USER_NAME}} and wait for confirmation. **This is the only pause point in the main flow.**

```
FRAMED QUESTION:

Situation: [plain-language statement of the decision]

Context that matters:
[Context Brief — max 300 words]

For the board: [crisp, specific version of the question]
What would help most: [stress-test / direction clarity / tradeoff map / action recommendation]
```

## Step 2 — Create Debate File

Path: `work/outputs/advisor-board-YYYY-MM-DD-[topic-slug].md`
(`topic-slug` = 2-4 word kebab-case summary, e.g. `launch-cohort-now`, `raise-prices`, `take-xyz-deal`)

Sections: `## Framed Question`, `## Round 1 — Initial Takes`, `## Round 2 — Rebuttals`, `## Round 3 — Final Positions & Recommendations`, `## Synthesis`.

## Steps 3-5 — Three rounds of debate

Read [references/agent-prompts.md](references/agent-prompts.md) for the three round prompt templates. For each round:

1. Dispatch all 12 board members (from `user-config.md` order) as subagents **in parallel** — single message, 12 Agent tool calls, model `sonnet`
2. Each agent writes to a temp file: `work/outputs/[debate-slug]/r[N]-[name-slug].md`
3. Wait for all 12 confirmations. If any agent returns an ERROR (e.g. persona not found), stop and report to {{USER_NAME}} before proceeding
4. Orchestrator assembles temp files into the debate file in canonical order (from `user-config.md`), then deletes the temp dir
5. Only after assembly is complete, proceed to the next round

Rounds are sequential across, parallel within — Round 2 starts only after all Round 1 temp files are assembled.

**No substitutions, no invented members.** If an agent cannot find their `## [NAME]` section in `references/board-members.md`, it must stop and return an ERROR. Never fabricate a persona.

## Step 6 — Synthesis

Read the complete debate file. Write the synthesis directly into the file:

```markdown
## Synthesis

### Where the board converges
[2-4 bullets — only list genuine alignment across 4+ members. Do not pad.]

### Unresolved tensions
[2-4 real disagreements that stayed disagreed after Round 3. Name specific members on each side. State the core disagreement plainly.]

### Clearest paths forward

**Path A: [name]** — what this path is, which members point toward it, what it requires accepting or giving up.

**Path B: [name]** — same format, only if genuinely distinct from Path A.

**Path C: [name]** — same format, only if applicable.

### The real question underneath this
[One sentence. The decision the debate revealed underneath the surface question.]
```

## Step 7 — Close

Report the debate file path. Offer to go deeper on any specific member's reasoning.

## Rules

- All subagents use model `sonnet`
- Board roster is fixed per `user-config.md` — no substitutions, no additions, no invented members
- Each agent writes its own temp file per round; orchestrator assembles after all confirmations — no concurrent writes to the debate file
- Orchestrator re-reads files fresh each session — never assume context is already loaded between rounds
- Context Brief is max 300 words, built from grep hits — never pass raw full files to agents
- Each agent greps for its own `## [NAME]` section in `board-members.md` and reads only that section (offset/limit), never the full file
- First person only — agents speak AS the persona, never "Goggins would say"
- No softening, no manufactured consensus — preserve real disagreement through to the synthesis
- Parallel within rounds, sequential across rounds
