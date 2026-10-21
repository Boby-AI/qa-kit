# qa-kit

A Claude Code plugin: a **reviewer** reads every change, and for bigger ones an independent **tester** and **adversary** check it against its approved spec before it lands. The doer never grades their own work. Team guide (Turkish): [`KILAVUZ.md`](KILAVUZ.md). Background in [`BRIEF.md`](BRIEF.md).

## Install

```bash
claude plugin marketplace add Boby-AI/qa-kit
claude plugin install qa-kit@boby
```

To try a local copy instead, use `claude plugin marketplace add /path/to/qa-kit`.

## Use

`/qa-kit:kontrol` in a project:

1. finds what changed since the base branch and which parts it touches;
2. runs those parts' fast checks;
3. has the reviewer read the diff, like a careful senior reviewer (every change, including config);
4. stops there unless asked for `derin` (deep);
5. with `derin`: drafts a spec with numbered criteria (`K1`, `K2`, …), which the owner approves, then runs the tester (writes `qa-spec` tests from the spec only) and the adversary (tries to break it) in parallel;
6. reports, with a paste-ready fix prompt per bug. The owner tries it and says "oldu".

Nothing is mandatory locally: the agents run on Opus, and the slow, costly tester and adversary only on request.

`/qa-kit:tam-tarama [parts]` scans the whole project instead of a change: every part's fast checks, then the adversary on each part.

## Automatic checks

- **Before push:** add a pre-push hook that runs the changed parts' fast commands, with the release `qaKit` names in qa-kit.json. With lefthook:

  ```yaml
  pre-push:
    jobs:
      - name: qa-kit
        run: npx --yes "github:Boby-AI/qa-kit#$(node -p "require('./qa-kit.json').qaKit")" fast
  ```

- **On PR:** copy [`templates/qa-kit.yml`](templates/qa-kit.yml) to `.github/workflows/`, add the `ANTHROPIC_API_KEY` secret, and make the `kontrol` check required on `main`. GitHub runs only the reviewer (no setup, a few minutes); tests and attacks run on the author's computer.

## Project config: `qa-kit.json`

Put it at the repo root. It names only the project's own commands.

```json
{
  "qaKit": "v0.8.0",
  "base": "main",
  "specs": "docs/specs",
  "findings": ".qa/findings",
  "sensitive": ["packages/billing/**", "**/payments/**"],
  "ignore": ["docs/**", ".github/**"],
  "shared": { "packages/**": "*", "pnpm-lock.yaml": "*" },
  "fastTimeout": 90,
  "parts": {
    "web": {
      "path": "apps/web",
      "kind": "web",
      "fast": ["pnpm --filter web check-types", "pnpm --filter web test"],
      "slow": ["pnpm --filter web test:e2e"],
      "test": "pnpm --filter web test",
      "run": "pnpm --filter web dev",
      "url": "http://localhost:3001"
    },
    "api": {
      "path": "apps/server",
      "kind": "api",
      "fast": ["pnpm --filter server test"],
      "test": "pnpm --filter server test"
    }
  }
}
```

| Field | Meaning |
|---|---|
| `qaKit` | The qa-kit release the push hook and GitHub run. Raise it to update both; `/qa-kit:kontrol` says when the plugin is newer. |
| `base` | Branch that changes are compared with. |
| `specs` | Folder for approved specs, one per branch. |
| `findings` | Folder for the adversary's repros and `findings.md`. |
| `sensitive` | Globs that always get the full check (money, rewards, payments, data). |
| `ignore` | Optional globs that never trigger checks, e.g. `docs/**`, `.github/**`. |
| `parts.<name>.path` / `paths` | The part's folder, or a list of folders and files (e.g. `["lib", "test", "pubspec.yaml"]`). A lock file goes with its manifest (`pubspec.lock` with `pubspec.yaml`). `qa-kit` warns about any folder with its own tests that no part covers. |
| `shared` | Files outside every part, as glob → part names or `"*"` for all. Anything else outside every part checks nothing and gets a warning. |
| `fastTimeout` | Seconds a fast command may take before it is stopped and reported (default 90). |
| `kind` | `web`, `mobile`, `api`, `game` or `lib`; picks the adversary's attacks. |
| `fast` | Quick commands that run before every push: format, lint, types, unit tests. |
| `slow` | Long commands (browser or end-to-end tests): only `/qa-kit:kontrol derin` and `/qa-kit:tam-tarama` run them. |
| `test` | How the tester runs tests. |
| `run`, `url` | Optional: how to start the part and where it answers, for the adversary. |

A single-part repo has one entry with `"path": "."`.
