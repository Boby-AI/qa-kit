# qa-kit

A Claude Code plugin: an independent **tester** and **adversary** check a change against its approved spec before it lands. The doer never grades their own work. Background in [`BRIEF.md`](BRIEF.md).

## Install

```bash
claude plugin marketplace add Boby-AI/qa-kit
claude plugin install qa-kit@boby
```

To try a local copy instead, use `claude plugin marketplace add /path/to/qa-kit`.

## Use

`/kontrol` in a project:

1. finds what changed since the base branch and which parts it touches;
2. runs those parts' fast checks;
3. stops there for small changes (copy, colours);
4. for bigger ones, drafts a spec with numbered criteria (`K1`, `K2`, …), which the owner approves;
5. runs the tester (writes `qa-spec` tests from the spec only) and the adversary (tries to break it) in parallel;
6. reports. The owner tries it and says "oldu".

`/tam-tarama` scans the whole project instead of a change: every part's fast checks, then the adversary on each part.

## Automatic checks

- **Before push:** add a pre-push hook that runs `npx --yes github:Boby-AI/qa-kit fast` (only the changed parts' fast commands). With lefthook:

  ```yaml
  pre-push:
    jobs:
      - name: qa-kit
        run: npx --yes github:Boby-AI/qa-kit fast
  ```

- **On PR, and the full-scan button:** copy [`templates/qa-kit.yml`](templates/qa-kit.yml) to `.github/workflows/`, fill in its setup steps, add the `ANTHROPIC_API_KEY` secret, and make the `kontrol` check required on `main`. "Run workflow" on the Actions tab starts a full scan.

## Project config: `qa-kit.json`

Put it at the repo root. It names only the project's own commands.

```json
{
  "base": "main",
  "specs": "docs/specs",
  "findings": ".qa/findings",
  "sensitive": ["packages/billing/**", "**/payments/**"],
  "parts": {
    "web": {
      "path": "apps/web",
      "kind": "web",
      "fast": ["pnpm --filter web check-types", "pnpm --filter web test"],
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
| `base` | Branch that changes are compared with. |
| `specs` | Folder for approved specs, one per branch. |
| `findings` | Folder for the adversary's repros and `findings.md`. |
| `sensitive` | Globs that always get the full check (money, rewards, payments, data). |
| `parts.<name>.path` | Folder of the part. Files outside every part count as shared and check all parts. |
| `kind` | `web`, `mobile`, `api`, `game` or `lib`; picks the adversary's attacks. |
| `fast` | Quick commands (format, lint, types, tests). |
| `test` | How the tester runs tests. |
| `run`, `url` | Optional: how to start the part and where it answers, for the adversary. |

A single-part repo has one entry with `"path": "."`.
