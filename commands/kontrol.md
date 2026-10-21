---
description: Check what changed — fast checks and a code reviewer; with "derin", also an independent tester and adversary against an approved spec
argument-hint: "[derin] [part names to limit to]"
---

Check the current change. The doer never grades their own work: you (the implementer) run the steps and fix bugs, but the review, tests and attacks come from fresh agents that never see your reasoning. The reviewer reads the code; the tester and adversary never do.

By default this is light: fast checks and the reviewer. The tester and adversary are slow and costly, so they run only when the arguments include `derin` ("deep"). Suggest `derin` in the report when the change touches a `sensitive` path, but never run it unasked.

Arguments: $ARGUMENTS

## 1. Config

Read `qa-kit.json` at the repo root (schema in the qa-kit plugin's README). If it is missing, stop and say the project needs one. If its `qaKit` is older than this plugin's version (`${CLAUDE_PLUGIN_ROOT}/package.json`), say so in one line: the push hook and GitHub still run the old release until `qaKit` is raised in a PR.

## 2. What changed

Run `qa-kit parts --worktree` (on PATH from this plugin; also at `${CLAUDE_PLUGIN_ROOT}/bin/qa-kit`). It prints the affected parts' entries, and warns on stderr about tested folders no part covers: pass those warnings on. If arguments name parts (anything other than `derin`), keep only those. No changed files at all → say so and stop. Changed files but no parts (only config, CI or docs) → skip to the reviewer in step 6, then report.

## 3. Fast checks

Run `qa-kit fast --worktree` (with `derin`, then `qa-kit slow --worktree` too) and wait for it to finish before step 6, so its results never mix with the tester's new files. Report failures; they must be fixed before going on, unless they come from the machine (a missing tool) rather than the change.

## 4. Light or deep?

Without `derin` → run only the reviewer (step 6), then report.

## 5. Spec

The spec lives at `<specs>/<branch>.md` (`/` in the branch name becomes `-`).

If it is missing or its front matter lacks `status: approved`:

1. Draft it from what the owner asked for (this conversation, the PR description, the issue, or the criteria in the PR's qa-kit report), **not from the code**:
   - what the change is for, in two or three sentences;
   - numbered acceptance criteria `K1`, `K2`, … — each one observable and testable;
   - an **Interface** section: the functions, endpoints, screens or element ids the tests may use.
2. Save it, then ask for approval in **three to five short lines**: what the change does and the few things that matter most to test, grouped by topic in plain words, with no criterion IDs and no list of every criterion. Nobody reads a long list, and ten items look like ten problems. Point to the file for the details, wait for approval and apply their edits. For example:

   > Kupon ekleme test edilecek: kim ekleyebilir, boş kod reddediliyor mu, aynı kupon iki kez kullanılabiliyor mu. Ayrıntılar `docs/specs/<branch>.md` içinde. Onaylıyor musun?
3. Set its front matter to `status: approved` and commit it on its own.

Never change an approved spec without the owner.

## 6. Reviewer, tester and adversary

Launch the `qa-kit:reviewer` agent on every change, with the base ref (`git merge-base origin/<base> HEAD`) and the spec path if there is one.

With `derin`, launch the `qa-kit:tester` and `qa-kit:adversary` agents too, all three **in parallel**: in one message, each with `run_in_background: false`, and wait for every result before the report. Give the tester and adversary only:

- the spec path;
- the affected parts' entries from `qa-kit.json`;
- for the adversary, the findings folder `<findings>/<branch>/`.

Don't pass the diff, file names you changed, or how you built it.

## 7. Report

Tell the owner, simply:

- fast checks: pass/fail;
- criteria: how many passed in one line, and only the failing ones by name;
- the reviewer's and the adversary's findings by severity, each with its mark (🔴 blocker, 🟠 major, 🟡 minor), and what held up;
- for each real bug, a **fix prompt** in a code block that someone can paste into a fresh Claude session as is: the criterion ID, what should happen, what happens instead, the command that reproduces it (the failing `qa-spec` test or repro file), and these rules: fix the code, never edit `qa-spec` tests, the repro or the spec, then run `/kontrol` again. Describe the behaviour, not a guessed code location.

Real bugs → fix the **code**, then run `/kontrol` again. Never edit the tester's `qa-spec` tests or the adversary's repros to make them pass; if you think one is wrong, say why and let the owner decide.

When everything passes, the last step is the owner's: they try it and say "oldu".

## In CI

When the `CI` environment variable is set, GitHub runs only the reviewer, the way a review bot would: no fast checks, no spec, no tester or adversary, which run only when someone asks for `/qa-kit:kontrol derin` on their computer. Skip steps 3–5 and launch only the `qa-kit:reviewer` agent with the base ref. No changed files passes. It posts its own blockers and majors as PR comments; don't post them again. Never fix code, and never end your turn before writing `.qa/verdict` (`fail` on any blocker, else `pass`) and `.qa/report.md`, in the language of the PR description, short enough to read in ten seconds:

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
