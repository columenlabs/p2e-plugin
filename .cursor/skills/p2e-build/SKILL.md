---
name: p2e-build
description: >-
  Build one P2E Wave end to end — work its member layers in order, verify every AC,
  and record evidence and status in P2E. Invoke as /p2e-build <release> <wave>;
  both arguments are required.
argument-hint: <release> <wave>
disable-model-invocation: true
icon: hammer
color: green
---

# p2e-build

Builds one **Wave** package. The build half of a two-session wave run; [`p2e-review`](../p2e-review/SKILL.md) is the other half. Read **`p2e-mode`** and its `references/p2e-model.md` first — every lifecycle rule and gate there applies here unchanged.

## Arguments (required)

`/p2e-build <release> <wave>` — e.g. `/p2e-build v0.15 W7` (`7` also accepted for the wave).

- Read them from the invocation (`$ARGUMENTS` on Claude Code).
- If either is missing, **stop** and ask for both. Never infer them from the branch, the latest release, or the last wave.

## Model

- Run the build session on **Sonnet**.
- Browser and UI checks run in **fast Sonnet subagents** (one per check or per story), so the main session keeps its context for code. Platforms without subagents run the checks inline.

## Steps

1. **Bind** — `.p2e/project.json` → `product_slug`. No binding → stop.
2. **Freeze the wave** — `waves.get` with `release` + `n`. Use its **ordered `members`**, `branch`, `gate` and `githubPrUrl`. Wave missing or not open → stop and report.
3. **Per member, in order** (skip members already `IN_REVIEW` or `DONE` unless they carry review notes, step 5):
   1. `stories op=get` — read the thick spec and every AC. Check `DEPENDS_ON` relations; an unmet dependency outside the wave → `story_log kind=BLOCKER`, move on.
   2. Move to `IN_PROGRESS` (preview, then write).
   3. **Coder** — implement on the wave `branch`.
   4. **Verifier** — for every AC: run `verificationCmd` or the check the tag shape needs (`ui` → screenshot/video from a browser subagent), upload proof (`story_assets` / `evidence`), then `criteria op=propose` with the verifier role (`PASS | FAIL | BLOCKED`).
   5. Any `FAIL` → fix and re-verify while still `IN_PROGRESS`. Coder and verifier cannot align → AC `BLOCKED`, escalate to the human.
   6. All ACs assessed, none `NOT_TESTED`, none `FAIL` → move to `IN_REVIEW`. Log a one-line `story_log kind=NOTE` with what shipped.
4. **Wave PR** — push the wave branch and open or update the PR recorded on the wave. Never merge it.
5. **Review notes** — re-running `/p2e-build` with the same arguments picks up notes the review left (reviewer `FAIL` assessments and review `story_log` notes). Fix, re-verify, and return the story to `IN_REVIEW`.
6. **Report** — one message: stories moved to `IN_REVIEW`, stories blocked and why, the PR link, and that the wave is ready for `/p2e-review <release> <wave>`.

## Hard rules

- Members come from `waves.get` only — never from `stories.list` wave filters.
- Never Mark DONE, never `criteria op=verdict` / `op=toggle`, never write reviewer assessments.
- Never merge. Merge happens only after review passes and a human approves.
- One story at a time, in wave order; don't start the next until the current one is `IN_REVIEW` or `BLOCKED`.
