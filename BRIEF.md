# qa-kit — brief

Owner's plan, agreed in conversation on 2026-10-20. Built so far: the plugin (tester, adversary, `/kontrol`). Git hook and PR gate come after the trial.

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
- **On PR** (GitHub Actions): the slow ones (adversary, screenshot comparison). This is the gate that can't be bypassed.
- **Full scan** of a whole project: **started on demand** (`/tam-tarama` or a GitHub button), not on a fixed
  nightly schedule. Runs in the background and reports when done.

## Projects that will use it

All repos are in the `Boby-AI` GitHub organisation. Test-file counts are from 2026-10-20.

| Project | Repo(s) | Parts | Tests today |
|---|---|---|---|
| Director | `directorai` (pnpm + Turbo monorepo) | `apps/web`, `apps/dashboard` (Next.js), `apps/native` (Expo / React Native), `apps/server` (Hono), `apps/executor`, `apps/agent`, shared `packages/*` | ~234 files, vitest per app; lefthook pre-commit (biome + custom checks) |
| Shiny AI | `xart` | Flutter app (root), `dashboard/` (Next.js), `supabase/` | ~20 Flutter tests, ~11 dashboard (vitest) |
| PlayMate | `play-mate` (old `play-mate-functions` and `play_mate_games` are merged in and abandoned) | Flutter app (root), `backend/` (Cloud Functions), `games/` (HTML5) | ~48, all in `games/`; none in the app or backend |

The other developers work in React.

## Decided

- Work lands through PRs, not straight pushes to main (approved 2026-10-20).
- Existing hooks (e.g. lefthook in `directorai`) are reused, not replaced by Husky.

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
