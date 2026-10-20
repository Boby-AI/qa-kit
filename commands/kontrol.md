---
description: Check what changed — fast checks, then an independent tester and adversary against the approved spec
argument-hint: "[part names to limit to]"
---

Check the current change. The doer never grades their own work: you (the implementer) run the steps and fix bugs, but the tests and attacks come from two fresh agents that never see the code or your reasoning.

Arguments: $ARGUMENTS

## 1. Config

Read `qa-kit.json` at the repo root (schema in the qa-kit plugin's README). If it is missing, stop and say the project needs one.

## 2. What changed

Changed files = committed since `git merge-base origin/<base> HEAD`, plus staged, unstaged and untracked. Leave out files under `specs` and `findings`.

Map each file to the part whose `path` contains it. A file in no part is shared code: every part is affected. If arguments name parts, keep only those.

Nothing changed → say so and stop.

## 3. Fast checks

Run every `fast` command of each affected part. Report failures; they must be fixed before going on.

## 4. Small or big?

Big if any changed file matches a `sensitive` glob, or the change alters behaviour (logic, data, flows, APIs). Small only if it is just copy, colours, spacing or assets.

Small → report the fast checks and stop.

## 5. Spec

The spec lives at `<specs>/<branch>.md` (`/` in the branch name becomes `-`).

If it is missing or its front matter lacks `status: approved`:

1. Draft it from what the owner asked for (this conversation, the PR description, the issue), **not from the code**:
   - what the change is for, in two or three sentences;
   - numbered acceptance criteria `K1`, `K2`, … — each one observable and testable;
   - an **Interface** section: the functions, endpoints, screens or element ids the tests may use.
2. Show it to the owner in plain words and wait for approval. Apply their edits.
3. Save it with front matter `status: approved`.

Never change an approved spec without the owner.

## 6. Tester and adversary

Launch the `qa-kit:tester` and `qa-kit:adversary` agents **in parallel**. Give each only:

- the spec path;
- the affected parts' entries from `qa-kit.json`;
- for the adversary, the findings folder `<findings>/<branch>/`.

Don't pass the diff, file names you changed, or how you built it.

## 7. Report

Tell the owner, simply:

- fast checks: pass/fail;
- each criterion: pass/fail;
- the adversary's findings by severity, and what held up.

Real bugs → fix the **code**, then run `/kontrol` again. Never edit the tester's `qa-spec` tests or the adversary's repros to make them pass; if you think one is wrong, say why and let the owner decide.

When everything passes, the last step is the owner's: they try it and say "oldu".
