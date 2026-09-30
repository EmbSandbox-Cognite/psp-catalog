# Partner Solution Patterns – Catalog

This repository is the index of published Partner Solution Patterns (PSP).
It contains one file that matters: [`patterns.json`](patterns.json), a hand-maintained list of pattern repositories.

The catalog holds **references only**. For a repository built from the pattern template, name, description, type, tags, icon and documentation live in that repository and are read from there by the PSP Flows app. An external repository has no `psp.json`, so those fields can be set on its catalog entry instead.

## How it works

```
patterns.json  ──►  list of repo URLs
                      │
                      ├─► <repo>/psp.json       name, description, type, tags
                      ├─► <repo>/psp_icon.svg   catalog icon
                      └─► <repo>/README.md      full documentation
```

The Flows app fetches `patterns.json`, then reads each repo's files via `raw.githubusercontent.com`. No GitHub API calls and no login are needed.

An entry that already includes `name`, `description`, `type` or `tags` is an external repository. The app uses the values on the entry and does not require `psp.json` in that repo.

## `patterns.json`

```json
{
  "version": 1,
  "patterns": [
    {
      "repo": "https://github.com/EmbSandbox-Cognite/psp-minimal-knowledge-graph"
    },
    {
      "repo": "https://github.com/cognitedata/cognite-ai-tooling-marketplace",
      "name": "CDF best-practice AI plugins",
      "description": "Claude Code and Cursor plugin with skills for data modeling, DMS queries, Functions, Transformations, Workflows and naming checks.",
      "type": "ai-tooling",
      "tags": ["ai", "data-modeling", "best-practices"]
    }
  ]
}
```

| Field | Required | Description |
|---|---|---|
| `version` | yes | Format version of this file. Currently `1`. |
| `patterns[].repo` | yes | Full GitHub URL of a public repository. |
| `patterns[].name` | no | Display name. Set this for an external repository that has no `psp.json`. |
| `patterns[].description` | no | Short description. Same case as `name`. |
| `patterns[].type` | no | Kind of entry, for example `ai-tooling`. Same case as `name`. |
| `patterns[].tags` | no | Array of tag strings. Same case as `name`. |

A pattern repository only needs `repo`. The app reads the rest from that repo's `psp.json`.

An external repository is any public repo that is not built from the pattern template. Add `name`, `description`, `type` and `tags` on the entry so it can be listed without those files.

The order of entries is the default display order in the app. Put the patterns you want partners to see first at the top.

The object form (`{ "repo": … }`) is intentional: optional fields can be present on some entries and absent on others without breaking the app.

## Publishing a pattern

1. Create the pattern repo from [`psp-template`](https://github.com/EmbSandbox-Cognite/psp-template) and complete it according to its `.github/PSP_GUIDE.md`.
2. Check that the repo is **public** and that these files load without login:
   ```bash
   REPO=EmbSandbox-Cognite/psp-<name>
   curl -s https://raw.githubusercontent.com/$REPO/HEAD/psp.json | jq .
   curl -s -o /dev/null -w "%{http_code}\n" https://raw.githubusercontent.com/$REPO/HEAD/psp_icon.svg
   ```
3. Open a pull request here adding one entry to `patterns.json`. For an external repository, include `name`, `description`, `type` and `tags` on that entry.
4. After merge, the pattern appears in the app within about 5 minutes (GitHub's raw file cache).

## Unpublishing and deprecating

- **Unpublish:** remove the entry from `patterns.json`.
- **Deprecate:** archive the pattern repository on GitHub and keep the entry. The app shows it as deprecated once automation adds archive status (see below); until then, state it at the top of the pattern's README and in its `psp.json` description.

## Conventions

Pattern repositories built from the template must follow these, because the app builds every URL from them. External repositories are described by the optional fields on their `patterns.json` entry and do not need these files:

| File | Location | Purpose |
|---|---|---|
| `psp.json` | repo root | Metadata: `name`, `description`, `type`, `tags` |
| `psp_icon.svg` | repo root | Icon, 256×256, see icon guidelines in `psp-template` |
| `README.md` | repo root | Documentation shown on GitHub and in the app |

Files are read from the repository's default branch (`HEAD`), so branch names don't matter.

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