# Partner Solution Patterns – Catalog

This repository is the index of published Partner Solution Patterns (PSP).
It contains one file that matters: [`patterns.json`](patterns.json), a hand-maintained list of pattern repositories.

The catalog holds **references only**. All pattern data – name, description, type, tags, icon and documentation – lives in each pattern repository and is read from there by the PSP Flows app.

## How it works

```
patterns.json  ──►  list of repo URLs
                      │
                      ├─► <repo>/psp.json       name, description, type, tags
                      ├─► <repo>/psp_icon.svg   catalog icon
                      └─► <repo>/README.md      full documentation
```

The Flows app fetches `patterns.json`, then reads each repo's files via `raw.githubusercontent.com`. No GitHub API calls and no login are needed.

## `patterns.json`

```json
{
  "version": 1,
  "patterns": [
    { "repo": "https://github.com/EmbSandbox-Cognite/psp-minimal-knowledge-graph" }
  ]
}
```

| Field | Required | Description |
|---|---|---|
| `version` | yes | Format version of this file. Currently `1`. |
| `patterns[].repo` | yes | Full GitHub URL of a public pattern repository. |

The order of entries is the default display order in the app. Put the patterns you want partners to see first at the top.

The object form (`{ "repo": … }`) is intentional: new optional fields can be added later without breaking the app.

## Publishing a pattern

1. Create the pattern repo from [`psp-template`](https://github.com/EmbSandbox-Cognite/psp-template) and complete it according to its `.github/PSP_GUIDE.md`.
2. Check that the repo is **public** and that these files load without login:
   ```bash
   REPO=EmbSandbox-Cognite/psp-<name>
   curl -s https://raw.githubusercontent.com/$REPO/HEAD/psp.json | jq .
   curl -s -o /dev/null -w "%{http_code}\n" https://raw.githubusercontent.com/$REPO/HEAD/psp_icon.svg
   ```
3. Open a pull request here adding one line to `patterns.json`.
4. After merge, the pattern appears in the app within about 5 minutes (GitHub's raw file cache).

## Unpublishing and deprecating

- **Unpublish:** remove the entry from `patterns.json`.
- **Deprecate:** archive the pattern repository on GitHub and keep the entry. The app shows it as deprecated once automation adds archive status (see below); until then, state it at the top of the pattern's README and in its `psp.json` description.

## Conventions

All pattern repositories must follow these, because the app builds every URL from them:

| File | Location | Purpose |
|---|---|---|
| `psp.json` | repo root | Metadata: `name`, `description`, `type`, `tags` |
| `psp_icon.svg` | repo root | Icon, 256×256, see icon guidelines in `psp-template` |
| `README.md` | repo root | Documentation shown on GitHub and in the app |

Files are read from the repository's default branch (`HEAD`), so branch names don't matter.

Allowed `type` values: `notebook`, `toolkit-module`, `guide`, `flows-app`, `bundle`.

## Future automation (not implemented)

The catalog is maintained by hand on purpose: with fewer than about 20 patterns, a one-line PR is the simplest setup with nothing to keep running. Consider automating when one of these becomes true:

- Patterns get forgotten in `patterns.json`, or entries point at repos that no longer exist.
- You want sorting by popularity (stars) or last updated, or automatic deprecated status. This data is only available from the GitHub API, which is too rate-limited for partners to call directly from the app.
- The number of patterns grows well beyond 20.

The planned approach, when needed:

- A scheduled GitHub Action in this repository (hourly, plus manual "Run workflow"), using only the built-in `GITHUB_TOKEN` – no secrets.
- It lists public repos in the org with the topic `partner-solution-pattern` and regenerates `patterns.json`, adding `stars`, `updated` and `archived` per entry.
- It validates each pattern's `psp.json` and `psp_icon.svg` and reports problems in the run summary.
- Publishing then becomes "add the topic to the repo" instead of a PR here.
- The file format stays compatible: existing fields are unchanged, new fields are optional, so the app keeps working.
- Note: GitHub disables scheduled workflows in public repos after 60 days without activity. The workflow should commit at least once a day to stay enabled.

## License

Apache-2.0. Each pattern repository carries its own license file.