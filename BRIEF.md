# qa-kit — brief

Owner's plan, agreed in conversation on 2026-10-20. Nothing is built yet.

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

| Project | Parts | Notes |
|---|---|---|
| PlayMate (`../play-mate`) | Flutter app, Cloud Functions, HTML5 games | Games already follow the process; the app has no tests yet |
| Director | web, app, dashboard | Possibly `../directorai`, confirm |
| Shiny AI | app, dashboard | |

The other developers work in React.

## Open questions for the owner

1. Stack of each part: are the Director and Shiny AI apps React Native, Flutter or something else? Web and dashboards React?
2. Where do the repos live: one GitHub organisation? Is Director one repo or several?
3. Which projects already have tests?
4. Once a team works in a repo, PlayMate's "push straight to main" rule becomes "open a PR". The owner has not approved that yet.

## Working with the owner

Talk in Turkish and keep it very simple: no jargon, no dense tables. Analogies work well
(cook / taster / saboteur; mirror at home vs security at the door).
