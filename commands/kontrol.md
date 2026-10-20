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

1. Draft it from what the owner asked for (this conversation, the PR description, the issue), **not from the code**:
   - what the change is for, in two or three sentences;
   - numbered acceptance criteria `K1`, `K2`, … — each one observable and testable;
   - an **Interface** section: the functions, endpoints, screens or element ids the tests may use.
2. Show it to the owner in plain words and wait for approval. Apply their edits.
3. Save it with front matter `status: approved` and commit it on its own, so the PR's CI run finds it.

Never change an approved spec without the owner.

## 6. Reviewer, tester and adversary

Launch the `qa-kit:reviewer` agent on every change, with the base ref (`git merge-base origin/<base> HEAD`) and the spec path if there is one.

For a big change, launch the `qa-kit:tester` and `qa-kit:adversary` agents too, all three **in parallel**. Give the tester and adversary only:

- the spec path;
- the affected parts' entries from `qa-kit.json`;
- for the adversary, the findings folder `<findings>/<branch>/`.

Don't pass the diff, file names you changed, or how you built it.

## 7. Report

Tell the owner, simply:

- fast checks: pass/fail;
- each criterion: pass/fail;
- the reviewer's and the adversary's findings by severity, and what held up;
- for each real bug, a **fix prompt** in a code block that someone can paste into a fresh Claude session as is: the criterion ID, what should happen, what happens instead, the command that reproduces it (the failing `qa-spec` test or repro file), and these rules: fix the code, never edit `qa-spec` tests, the repro or the spec, then run `/kontrol` again. Describe the behaviour, not a guessed code location.

Real bugs → fix the **code**, then run `/kontrol` again. Never edit the tester's `qa-spec` tests or the adversary's repros to make them pass; if you think one is wrong, say why and let the owner decide.

When everything passes, the last step is the owner's: they try it and say "oldu".

## In CI

When the `CI` environment variable is set, nobody is there to approve. Skip step 3 (CI runs the fast checks in its own job). No changed files passes. A small change passes unless the reviewer finds a blocker. A big change without an approved spec fails with "spec needed". Never fix code. End by writing `.qa/verdict` containing `pass` or `fail` (fail on any failing criterion or any blocker from the reviewer or adversary), and put the report in `.qa/report.md`. Also write `.qa/comments.json`: an array with one `{"path", "line", "body"}` per reviewer finding whose line is in the diff (new side), where `body` is the severity and a one-line title in bold, what goes wrong, then the fix prompt inside `<details><summary>Prompt to fix with AI</summary>` … `</details>`. `qa-kit report` posts these as inline comments.
