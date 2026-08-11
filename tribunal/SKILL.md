---
name: tribunal
description: |
  Puts work on trial before you ship it. A second AI model (OpenAI Codex, running locally) acts as prosecutor and attacks the work; Claude defends every charge one by one with evidence; a fresh, blind judge rules on each dispute — FIX, DISMISS, or HUMAN DECISION; Claude then repairs only what was ordered. Works on code and on non-code alike: offers, landing pages, plans, documents, spreadsheets. Use whenever the user wants a second opinion, a critique, a review, a sanity check, a red-team pass, or asks "is this actually good", "what did I miss", "poke holes in this", "put this on trial", "tear this apart", "before I send this", or invokes /tribunal. Reach for it especially right after Claude has built or written something substantial, because the model that built a thing is the worst reviewer of it.
---

# Tribunal

A courtroom for work you are about to ship.

The reason this exists: a model that just built something is a bad reviewer of it. It carries the same assumptions it wrote the thing with, so its "critique" tends to be a polite list of nits. Asking a *different* model — with a different training and different blind spots — to attack the work surfaces things self-review structurally cannot.

But a hostile review alone is also not a verdict. Attackers overreach. They flag deliberate choices as bugs, invent problems the code already handles elsewhere, and pad findings to look thorough. Acting on an unchallenged accusation means "fixing" working things and breaking them.

So: prosecution, defense, and a blind judge. Five phases, starting with the one everybody forgets to name.

```
0. AUTHOR       One model produced the work — code, a script, an architecture,
                an offer. Name it. Everything below follows from this.
1. PROSECUTOR   A model that is NOT the author attacks the work. Numbered charges.
2. DEFENSE      The author answers every charge: admit, or deny with evidence.
3. JUDGE        A fresh run of a non-author model sees charge/defense pairs with
                authorship stripped. Rules per charge: FIX / DISMISS / HUMAN DECISION.
4. EXECUTION    The author repairs only what was ordered. Everything else is
                reported as dismissed, with the reason.
```

## Casting

One rule holds the whole thing up: **the author never prosecutes and never judges its own work.** Everything else is negotiable.

The default casting, because Claude is usually the one that just built the thing:

| Role | Who | Why |
|---|---|---|
| Author | Claude | it wrote the work in this session |
| Prosecutor | Codex | different model, different blind spots |
| Defense | Claude | you defend what you made — that is the point |
| Judge | Codex, fresh run | knows nothing about the build conversation |

Announce the casting in one line before starting, and let the user change it. Two common swaps:

- **Codex wrote the work** (the user built it through the Codex CLI or the Codex plugin): then Codex is the author and defends, Claude prosecutes, and a Claude subagent in a fresh context judges.
- **A human wrote the work** — an offer, a plan, a document the user wrote themselves. Both models are free, so use Codex as prosecutor and Claude as judge, or the reverse. Claude then defends as counsel rather than as author: it argues from what is in the files, and where a charge turns on intent it cannot verify, it says so plainly. Those charges tend to land on HUMAN DECISION, which is correct — only the author knows what they meant.

If the user asks for a casting where the author also prosecutes or judges, say why that defeats the mechanism and offer the nearest working alternative. It is the one constraint worth defending: without it this is just a model reviewing itself with extra steps.

## Before you start

**Check the tool exists.** This skill needs the Codex CLI on the machine:

```bash
command -v codex && codex --version
```

Missing? Tell the user plainly:

```bash
npm install -g @openai/codex
codex login
```

An OpenAI account is required (a ChatGPT Plus/Pro plan or an API key). If they'd rather not install it, offer the degraded fallback described at the bottom — but say clearly that it is weaker, because the whole value here is that the critic is a *different* model.

**Decide what is on trial.** In order of preference:

1. The user named files or a folder — use that.
2. Work Claude produced in this conversation — use those paths.
3. A git repo with uncommitted work — `git status --short` and `git diff`, and say which files you picked.

**Establish the author**, since the casting depends on it. If Claude built it in this conversation, that is settled. Otherwise ask — one short question, because guessing wrong puts the author in the prosecutor's chair and quietly turns the trial into self-review.

State both in one line before proceeding, so a wrong assumption costs a sentence instead of five minutes:

> Na ławie: `src/auth.ts`, `src/session.ts` (340 linii). Autor: Claude. Prokurator i sędzia: Codex.

**Set up a working directory** so the artifacts survive and can be re-read:

```bash
mkdir -p .tribunal
```

Use `.tribunal/` in the project root. Mention it to the user; suggest gitignoring it if the project is a repo.

## Phase 1 — Prosecutor

Write the prompt to a file rather than passing it as a shell argument. Prompts contain quotes, backticks and newlines that shell quoting mangles, and a mangled prompt produces a confidently wrong review.

Build `.tribunal/1-prosecutor-prompt.md` from `references/prosecutor.md`, filling in the context, the paths, and the user's focus if they gave one. Then:

```bash
codex exec -s read-only --skip-git-repo-check -C "$(pwd)" \
  -o .tribunal/2-charges.md - < .tribunal/1-prosecutor-prompt.md
```

`-s read-only` matters: the prosecutor reads and reasons, it never edits. `-o` writes the final message to a clean file with none of the progress chatter.

This takes minutes, not seconds, on real work. Run it with a generous timeout (600000 ms) or in the background — do not let a 2-minute default kill a review that was almost done.

Read `.tribunal/2-charges.md`. If it came back empty or the command failed, show the user the actual error instead of inventing charges.

**If the casting puts Claude in the prosecutor's chair** (because Codex authored the work), skip the shell entirely: dispatch a subagent with the same prompt, in a fresh context that has not seen the work being built. Same prompt, same output file, same rules — only the runner changes.

## Phase 2 — Defense

The author answers here, so if Codex wrote the work, the defense is a `codex exec` run against `.tribunal/2-charges.md`. In the default casting it is Claude, writing directly.

Answer **every single charge**. A charge you skip becomes a charge the judge never sees, which quietly turns the skill back into "one model's opinion".

For each one, write to `.tribunal/3-defense.md`:

```markdown
### Charge 3
**Position:** ADMIT | DENY | PARTIAL
**Argument:** ...
**Evidence:** path/to/file.ts:88-104 — what is actually there
```

Rules that make the defense worth having:

- Verify before you answer. Open the file and look. A defense written from memory of what you intended to build is worthless — the prosecutor read the real thing.
- Admit freely. There is no penalty here for being wrong, and a false denial that survives to the judge is how a real bug ships.
- Deny only with a location. "That's handled elsewhere" without a path is not a defense, and a good judge will treat it as a concession.
- PARTIAL is often the honest answer: the charge points at something real but overstates the impact or the fix.

## Phase 3 — Judge

The judge must not know who wrote what. If it learns that the accusations came from a rival model and the defense from the model on trial, it starts scoring reputations instead of arguments.

So when you build `.tribunal/4-judge-prompt.md` from `references/judge.md`:

- Label the sides only as CHARGE and RESPONSE.
- Strip every mention of Claude, Codex, GPT, OpenAI, Anthropic, "the model", "the assistant", "prosecutor", "defense".
- Do not carry over the prosecutor's confidence scores or severity labels. Let the judge weigh the argument, not the swagger it was delivered with.

```bash
codex exec -s read-only --skip-git-repo-check -C "$(pwd)" \
  -o .tribunal/5-verdict.md - < .tribunal/4-judge-prompt.md
```

Three possible rulings per charge:

| Ruling | Meaning |
|---|---|
| **FIX** | The charge holds and the defense did not refute it. Repair it. |
| **DISMISS** | The defense refuted it, or the charge was never material. |
| **HUMAN DECISION** | Real dispute, no decisive evidence either way — or the call is about product, risk appetite or budget, which is not a technical question. |

HUMAN DECISION is the default when the judge cannot decide from evidence in the files. This is deliberate. A judge that guesses in order to look decisive will order fixes to things that were fine, and the user loses trust in the whole mechanism. Being told "you need to decide this one, here's the tension" is more useful than a confident coin flip.

## Phase 4 — Execution

The author repairs its own work — whoever that is under the current casting.

Repair only what the judge marked FIX. Not the DISMISSED ones you privately still agree with, not the HUMAN DECISION ones you have an opinion about. The user consented to a mechanism; silently overruling it makes the verdict theatre.

Then present one table — this is the whole point of the exercise, the thing the user actually reads:

```markdown
| # | Charge (short) | Ruling | Status |
|---|---|---|---|
| 1 | Brak obsługi pustej odpowiedzi API | FIX | ✅ naprawione — `api.ts:47` |
| 2 | Hasło w logach | DISMISS | ⛔ obrona: maskowane w `logger.ts:22` |
| 3 | Retry może zdublować zamówienie | HUMAN DECISION | ⚠️ Twoja decyzja |
```

Below the table, expand only the HUMAN DECISION rows: what each side argued, and what the user is actually choosing between. Two or three sentences each. Those rows are the deliverable — they are the things a review of any kind would otherwise bury.

Close with the artifact paths so the user can read the raw material if they want to argue with it.

## Rounds

One trial is the default and is usually enough.

If the user asks for more rounds, a second round is **not** a re-run of the same trial. It re-prosecutes only the charges that were fixed, asking one question: did the repair actually work, or did it move the problem? Re-running the full trial mostly regenerates the same dismissed charges and burns minutes for nothing.

Stop when a round produces no new FIX rulings, and say so.

## Working on non-code

Everything above holds for an offer, a landing page, a client document, a spreadsheet, a plan. Only two things change:

- Point the prosecutor at what makes *that* artifact fail — a promise that cannot be kept, a price that ignores an obvious cost, a claim with no basis, a step the reader cannot actually perform, a missing case in the data.
- Tell both the prosecutor and the judge to write for a non-technical reader: plain sentences, no jargon, and every claim tied to a specific place in the document.

The four phases do not change. Neither does the reason the mechanism works.

## Language

Answer the user in the language they wrote to you in. The Codex prompts in `references/` instruct the prosecutor and judge to do the same — if the user writes Polish, the whole trial runs in Polish.

## If Codex is unavailable

Ask before falling back, because the fallback changes what the user is getting.

The fallback: run the prosecutor as a Claude subagent with the same prompt from `references/prosecutor.md`, in a fresh context so it has not seen the work being built. Then judge with a second, separate subagent.

Say plainly what is lost: same model, same training, same blind spots. A fresh context removes some of the self-review bias, but not the structural part. It is better than nothing and clearly worse than a real second model.
