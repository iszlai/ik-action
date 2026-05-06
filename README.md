# inkode GitHub Action

Run [inkode](https://inkode.co) (`ik`) against your repository on every PR /
push and post a sticky comment summarising new findings, fixed issues, and the
overall score.

```yaml
- uses: iszlai/ik-action@v1
  with:
    token: ${{ secrets.IK_TOKEN }}
    fail-on: new-errors
```

## What you get

- **Inline annotations** on the PR diff for every finding (severity-mapped to
  `::error` / `::warning` / `::notice`).
- **A sticky PR comment** — single comment, updated on subsequent runs —
  listing new findings introduced by the PR and findings the PR fixed.
- **A status check** that passes/fails according to your `--fail-on` policy.
- **Trend data** on `https://ik.inkode.co/p/<your-project>` once you have at
  least two reports on `main`.

## Quickstart

1. **Get a push token**: email [hello@inkode.co](mailto:hello@inkode.co) with
   the project slug you want to use (e.g. your repo name). We'll send back a
   token within one business day.
2. Add it as a repo secret called `IK_TOKEN`:
   ```sh
   gh secret set IK_TOKEN --repo your-org/your-repo
   ```
3. Drop a `.ik.yaml` into your repo (run `ik init` locally if you don't have
   one yet).
4. Add `.github/workflows/ik.yml`:

```yaml
name: ik
on: [pull_request, push]
jobs:
  scan:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write   # required for the sticky PR comment
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0      # ik needs git history for hotspot/coupling
      - uses: iszlai/ik-action@v1
        with:
          token: ${{ secrets.IK_TOKEN }}
          fail-on: new-errors
```

That's it.

## Inputs

| Name        | Default                  | Description |
|-------------|--------------------------|-------------|
| `token`     | _(required)_             | Project push token (set as a repo secret). |
| `endpoint`  | `https://api.inkode.co`  | inkode server URL. Override for self-hosted instances. |
| `fail-on`   | `new-errors`             | Policy that decides exit code. See below. |
| `output`    | `github-annotations`     | `human` \| `json` \| `github-annotations` \| `sarif`. |
| `comment`   | `true`                   | Post (or update) a sticky PR comment. |
| `ik-version`| `latest`                 | ik release tag (e.g. `v0.5.0`). |
| `repo-path` | `.`                      | Path to the repository root. |

## Outputs

| Name         | Description                                  |
|--------------|----------------------------------------------|
| `score`      | Overall score 0–100.                         |
| `grade`      | Letter grade A–F.                            |
| `report-url` | Public URL of the uploaded report.           |

## `fail-on` policies

| Policy            | Fails when… |
|-------------------|-------------|
| `never`           | never. |
| `score:N`         | the overall score is below N. Example: `score:60`. |
| `new-errors`      | the PR introduces ≥ 1 new finding with severity `error`. **Recommended for PRs.** |
| `new-warnings`    | the PR introduces ≥ 1 new finding (any severity). |
| `score-drop:N`    | the overall score drops by ≥ N points vs the base branch. Example: `score-drop:5`. |

Diff-aware policies (`new-errors`, `new-warnings`, `score-drop:N`) compare
against the most recent report on the PR's base branch. If no baseline exists
yet (the project's first scan, or the base branch has never been scanned),
they degrade to `score:60`.

## Examples

See [`examples/`](examples/) for ready-to-copy workflows:

- [`basic.yml`](examples/basic.yml) — minimal setup.
- [`pr-only.yml`](examples/pr-only.yml) — only run on pull requests.
- [`matrix.yml`](examples/matrix.yml) — run across multiple Go versions.

## Permissions

The action posts PR comments via
[`marocchino/sticky-pull-request-comment`](https://github.com/marocchino/sticky-pull-request-comment).
That requires `pull-requests: write` on the workflow's `permissions:` block.

## What `ik` actually does

[inkode](https://inkode.co) runs static checks across categories — security
(secrets, dep-audit), testing (test-presence), maintainability (duplication,
coupling, complexity, magic-numbers, todo-density, dead-code, hotspots) — and
combines the results into a single 0–100 score. Findings the PR introduces or
fixes are diffed against the base branch's most recent scan.

## Questions, bug reports, or want a token?

Email **[hello@inkode.co](mailto:hello@inkode.co)** — fastest path to a push
token, a feature request, or a bug report. For action-specific issues you can
also [open an issue](https://github.com/iszlai/ik-action/issues).

## License

MIT
