---
name: p2e-review
description: >-
  Review one built P2E Wave from its recorded evidence, without re-testing, and send
  notes back to the builder. Invoke as /p2e-review <release> <wave>; both arguments
  are required.
argument-hint: <release> <wave>
disable-model-invocation: true
icon: search
color: purple
---

# p2e-review

Reviews one **Wave** after [`p2e-build`](../p2e-build/SKILL.md) has moved its stories to `IN_REVIEW`. Read **`p2e-mode`** and its `references/p2e-model.md` first — the reviewer role and its blindness rules apply here unchanged.

## Arguments (required)

`/p2e-review <release> <wave>` — e.g. `/p2e-review v0.15 W7` (`7` also accepted for the wave).

- Read them from the invocation (`$ARGUMENTS` on Claude Code).
- If either is missing, **stop** and ask for both. Never infer them.

## Model

- Run the review session on **Opus**, in a separate session from the build.
- Start it once the build reports the wave ready for review.

## Steps

1. **Bind** — `.p2e/project.json` → `product_slug`. No binding → stop.
2. **Freeze the wave** — `waves.get` with `release` + `n`; use its ordered `members` and PR. Members not yet `IN_REVIEW` → list them as not ready and review the rest.
3. **Per member, in order:**
   1. `stories op=get` for the spec and ACs; `criteria op=list` with the **reviewer viewer role** only.
   2. Read the evidence (`story_assets`, `evidence`) and the PR diff for that story.
   3. Judge each AC from the AC text, the evidence and the diff. Ask: does this proof actually show the AC holds, and is it the shape the tag needs (`ui` needs visual proof)?
   4. `criteria op=propose` with the reviewer role: `PASS | FAIL`, a one-line `summary` and an `analysis` that says what is missing for any `FAIL`.
4. **Notes back to the builder** — for every `FAIL` or gap, write a `story_log kind=NOTE` on that story naming the AC and what the builder must fix or prove. These are what `/p2e-build` picks up on its next run.
5. **Report** — one message: wave verdict (`pass` only if every AC passed), per-story fail counts, cross-story seams spotted in the diff, and the notes sent. On `pass`, say the wave is ready for human approval and merge.

## Hard rules

- **No re-testing.** Don't run the app, tests or `verificationCmd`. Weak or missing evidence is a `FAIL` with a note asking the builder for better proof.
- **Blind to the verifier.** Don't read verifier assessments, `story_log` entries, or the inline log on `stories op=get`. If you see verifier output by accident, say so in the report.
- Reviewer proposes only: `PASS | FAIL`, no `BLOCKED`, never `op=verdict` / `op=toggle`, never status writes, never Mark DONE.
- Never merge. Merge happens only after review passes and a human approves.
