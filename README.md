# sci4seng/data

Drop-zone + manifest for project data bundles. Raw data lives on
Drive — this repo tracks only the ingest contract + test list.

## Layout

```
manifest.yml             Drive URL + checksum per project bundle
dropzone/                drop zips or unpacked subdirs here
  README.md              ingest contract (what filenames the pipeline expects)
tests.md                 130 regression tests for the 3-repo split
```

## How updates flow

1. SME drops a refreshed bundle (e.g. `helix.zip`) on Drive.
2. Anyone clones this repo, runs `scripts/fetch.py` (TBD) against
   `manifest.yml` to pull the bundles into `dropzone/`.
3. From `sci4seng/core`, run `make refresh` — picks up `dropzone/`
   contents, melts lift CSVs, refreshes audit, regenerates the site.
4. `dropzone/` is gitignored; nothing huge ever lands in this repo.

## What lives here (tracked)

- `manifest.yml` — bundle URLs + SHA256 (~1 KB)
- `tests.md` — regression checklist for the split
- `dropzone/README.md` — usage contract
- `README.md` (this file)

## What does NOT live here

- Project git_repos, mbox, jira, derived artifacts → Drive
- Knit HTML outputs → `sci4seng/core/docs/lifts/` or Drive
- Anything > 100 KB tracked

## Sibling repos

- [sci4seng/core](https://github.com/sci4seng/core) — framework + site
- [sci4seng/lifts](https://github.com/sci4seng/lifts) — Rmd vignettes
