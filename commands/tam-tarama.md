---
description: Full scan of the whole project — every part's fast checks, then the adversary attacks each part
argument-hint: "[part names to limit to]"
---

Scan the whole project, not just a change. Run it in the background and report when done; it can take a while.

Arguments: $ARGUMENTS

1. Read `qa-kit.json` at the repo root. If it is missing, stop and say the project needs one. If arguments name parts, keep only those.
2. Run `qa-kit fast --all`, then `qa-kit slow --all` (on PATH from this plugin; also at `${CLAUDE_PLUGIN_ROOT}/bin/qa-kit`), and wait for both.
3. Launch a `qa-kit:adversary` agent per part, at most three at a time, each with `run_in_background: false` so you wait for it. Give each: there is no spec, so the truth is the part's own docs (`README`, `docs/`) plus the rules every product keeps (no crash, no data loss, money and credits change exactly once); the part's entry from `qa-kit.json`; and the findings folder `<findings>/full-<YYYY-MM-DD>/<part>/`.
4. Report simply: fast checks per part, findings per part by severity, and parts with no tests at all. For each real bug, add a paste-ready fix prompt as `/kontrol` describes.

Pass on any warnings `qa-kit` prints about tested folders no part covers. Never fix code during a scan.
