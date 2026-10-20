---
description: Check what changed — fast checks, a code reviewer, then an independent tester and adversary against the approved spec
argument-hint: "[part names to limit to]"
---

Check the current change. The doer never grades their own work: you (the implementer) run the steps and fix bugs, but the review, tests and attacks come from fresh agents that never see your reasoning. The reviewer reads the code; the tester and adversary never do.

Arguments: $ARGUMENTS

## 1. Config

Read `qa-kit.json` at the repo root (schema in the qa-kit plugin's README). If it is missing, stop and say the project needs one.

## 2. What changed

Run `qa-kit parts` (on PATH from this plugin; also at `${CLAUDE_PLUGIN_ROOT}/bin/qa-kit`). It prints the affected parts' entries, and warns on stderr about tested folders no part covers: pass those warnings on. If arguments name parts, keep only those. No changed files at all → say so and stop. Changed files but no parts (only config, CI or docs) → skip to the reviewer in step 6, then report.

## 3. Fast checks

Run `qa-kit fast` and wait for it to finish before step 6, so its results never mix with the tester's new files. Report failures; they must be fixed before going on, unless they come from the machine (a missing tool) rather than the change.

## 4. Small or big?

Big if any changed file matches a `sensitive` glob, or the change alters behaviour (logic, data, flows, APIs). Small only if it is just copy, colours, spacing or assets.

Small → run only the reviewer (step 6), then report.

## 5. Spec

The spec lives at `<specs>/<branch>.md` (`/` in the branch name becomes `-`).

If it is missing or its front matter lacks `status: approved`:

1. Draft it from what the owner asked for (this conversation, the PR description, the issue, or the criteria in the PR's qa-kit report), **not from the code**:
   - what the change is for, in two or three sentences;
   - numbered acceptance criteria `K1`, `K2`, … — each one observable and testable;
   - an **Interface** section: the functions, endpoints, screens or element ids the tests may use.
2. Show it to the owner in plain words and wait for approval. Apply their edits.
3. Save it with front matter `status: approved` and commit it on its own.

Never change an approved spec without the owner.

## 6. Reviewer, tester and adversary

Launch the `qa-kit:reviewer` agent on every change, with the base ref (`git merge-base origin/<base> HEAD`) and the spec path if there is one.

For a big change, launch the `qa-kit:tester` and `qa-kit:adversary` agents too, all three **in parallel**: in one message, each with `run_in_background: false`, and wait for every result before the report. Give the tester and adversary only:

- the spec path;
- the affected parts' entries from `qa-kit.json`;
- for the adversary, the findings folder `<findings>/<branch>/`.

Don't pass the diff, file names you changed, or how you built it.

## 7. Report

Tell the owner, simply:

- fast checks: pass/fail;
- each criterion: pass/fail;
- the reviewer's and the adversary's findings by severity, each with its mark (🔴 blocker, 🟠 major, 🟡 minor), and what held up;
- for each real bug, a **fix prompt** in a code block that someone can paste into a fresh Claude session as is: the criterion ID, what should happen, what happens instead, the command that reproduces it (the failing `qa-spec` test or repro file), and these rules: fix the code, never edit `qa-spec` tests, the repro or the spec, then run `/kontrol` again. Describe the behaviour, not a guessed code location.

Real bugs → fix the **code**, then run `/kontrol` again. Never edit the tester's `qa-spec` tests or the adversary's repros to make them pass; if you think one is wrong, say why and let the owner decide.

When the fast checks, every criterion and the reviewer pass with no blocker, and no part names limited the run, run `qa-kit pass`. Opening the PR stays blocked until it has run on the code as it is; on push it marks the commit `qa-kit/local` on GitHub. Then the last step is the owner's: they try it and say "oldu".

## In CI

When the `CI` environment variable is set, GitHub runs only the reviewer, the way a review bot would: no fast checks, no spec, no tester or adversary, which run on the author's computer before the PR. Skip steps 3–5 and launch only the `qa-kit:reviewer` agent with the base ref. No changed files passes. It posts its own blockers and majors as PR comments; don't post them again. Never fix code, and never end your turn before writing `.qa/verdict` (`fail` on any blocker, else `pass`) and `.qa/report.md`, in the language of the PR description, short enough to read in ten seconds:

```
✅ Passed — <one plain sentence: what was reviewed>
```
or
```
❌ Needs a fix — <one plain sentence>

1. 🔴 <the problem in plain words, one line> (`file:line`)
2. 🟠 <…>
```

List only blockers and majors, one line each, blockers first. Minors and what held up go inside one `<details><summary>Details</summary>` block below.
