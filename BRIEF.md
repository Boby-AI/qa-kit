# qa-kit — brief

Owner's plan, agreed in conversation on 2026-10-20.

Status (2026-10-20): live in `directorai` (Boby-AI/directorai#362), `shinyai` (Boby-AI/shinyai#29) and `play-mate` (Boby-AI/play-mate#29): pre-push fast
checks; `/qa-kit:kontrol` locally on request (`derin` adds tester and adversary); GitHub reviews every ready PR; full scan
with `/qa-kit:tam-tarama`. A ruleset on each `main` requires the `kontrol` check (directorai's existing rules already require its tests; production is left out on purpose). Team guide:
[`KILAVUZ.md`](KILAVUZ.md).

## Goal

Every developer and designer, on every company project, gets their bigger changes checked by an
independent tester and an adversary before they land. It must not depend on people remembering.

## Origin

The process comes from PlayMate's game platform (`../play-mate/docs/GAME_PLATFORM.md` §0 and §6,
`../play-mate/.claude/agents/game-spec-tester.md`, `game-adversary.md`). There it already caught bugs that
the implementer's own tests missed, e.g. a double `match_ended` that would have paid out twice
(`../play-mate/docs/platform-log.md`).

Core idea: the doer never grades their own work.

1. The owner describes what they want; it becomes a spec with numbered acceptance criteria; the owner approves.
2. The implementer builds it.
3. The **tester** reads only the spec (never the code) and writes and runs the tests.
4. The **adversary** tries to break it: offline, double taps, back button, backgrounding, clock changes,
   small screens, long names, dark mode, large fonts.
5. The owner tries it and says "oldu".

Small changes (a colour, a text) skip this. Anything that touches money, rewards, payments or data goes through it.

## Shape

- **One Claude Code plugin**, installed by every project: tester agent, adversary agent, a `/kontrol` command.
- Each project, and each part of a monorepo, adds a small config that only names its own test commands.
- `/kontrol` checks **only what changed** (git diff). In a monorepo it picks the changed parts; a change to
  shared code checks every part that uses it.
- **Before push** (git hook, e.g. Husky for React): fast checks only (format, lint, tests), so nobody is tempted to skip.
- **Before the PR** (author's computer, optional): `/qa-kit:kontrol` for fast checks and the reviewer; `derin` adds tester and adversary.
- **On PR** (GitHub Actions): only the reviewer, like a review bot. No project setup on GitHub.
- **Full scan** of a whole project: **started on demand** with `/tam-tarama`, locally. Runs in the background and reports when done.

## Projects that will use it

All repos are in the `Boby-AI` GitHub organisation. Test-file counts are from 2026-10-20.

| Project | Repo(s) | Parts | Tests today |
|---|---|---|---|
| Director | `directorai` (pnpm + Turbo monorepo) | `apps/web`, `apps/dashboard` (Next.js), `apps/native` (Expo / React Native), `apps/server` (Hono), `apps/executor`, `apps/agent`, shared `packages/*` | ~234 files, vitest per app; lefthook pre-commit (biome + custom checks) |
| Shiny AI | `shinyai` (pnpm + Turbo monorepo; `xart` is the older Flutter app) | `apps/native` (Expo / React Native), `apps/dashboard` (Next.js), `supabase/` | ~44 files, vitest |
| PlayMate | `play-mate` (old `play-mate-functions` and `play_mate_games` are merged in and abandoned) | Flutter app (root), `backend/` (Cloud Functions), `games/` (HTML5) | ~48, all in `games/`; none in the app or backend |

The other developers work in React.

## Decided

- Work lands through PRs, not straight pushes to main (approved 2026-10-20).
- Existing hooks (e.g. lefthook in `directorai`) are reused, not replaced by Husky.
- v0.8 (2026-10-21), after play-mate pushes took 15+ minutes and developers deleted the hook: parts own several
  paths and their lock files, `shared` is explicit (no more "outside every part checks everything"), pushes count
  committed changes only, fast commands have a 90 s limit and long tests live in `slow`, and `qaKit` in
  qa-kit.json is the one version the hook and workflow run.
- `shinyai` removed qa-kit on 2026-10-21 (RB Mercan); its `qa-kit` ruleset still required the `kontrol` check.
- v0.7 (2026-10-21), after the Shiny developers removed the plugin: the mandatory local `/qa-kit:kontrol` before
  every PR took too long and used up their Fable quota. Now nothing is mandatory locally, the agents run on
  Opus, and the tester and adversary run only on request (`/qa-kit:kontrol derin`). GitHub's reviewer stays.
- A reviewer agent reads every change (Greptile-style inline comments with fix prompts); tester and adversary run on request.
- Tests and attacks run only locally, before the PR; GitHub only reviews the code (2026-10-20). Installing a project's
  toolchain, ffmpeg and the like on GitHub made every PR slow and fragile.
- Draft PRs wait for "Ready for review". Money is not the constraint; signal over noise is.

## Working with the owner

Talk in Turkish and keep it very simple: no jargon, no dense tables. Analogies work well
(cook / taster / saboteur; mirror at home vs security at the door).

## Trial 1 — directorai, 2026-10-20

A real fixed bug (90f13190: account deletion lost credits held by unstarted runs) was put back, with its
test removed, on a throwaway local worktree. The existing 6 tests still passed. A fresh session ran
`/kontrol` with an owner-approved spec:

- The tester, from the spec only, wrote 11 tests: 4 fail on the bug, all 11 pass on the real fix.
- The adversary found the same bug and its worst cases (kept credits 0, or negative).
- Fixed afterwards in the plugin: fast checks raced the tester; `qa-kit.json` counted as shared code and
  pulled in every part; the adversary skipped `findings.md`.
