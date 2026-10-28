# DistrictTiles

Public artifact repo holding the MBTiles map packages that DistrictView downloads over the air; every data file is
generated and published by GeoData. Part of Eddie's portfolio; the cross-project brain is
`OneDrive\Documents\My Projects\brain-appDevelopment` (registry, standards, digest `projects\DistrictTiles.md`).

## Stack
Data artifacts only: `.mbtiles` (vector tiles in SQLite) + JSON manifests + MIT `LICENSE`. No code, no build,
no version file: the publish commit ("Publish GeoData MBTiles <date>") is the version. No Git LFS; every file
must stay under GitHub's 100 MB limit.

## Commands
Nothing to build or test. Checks (run 2026-10-28):
```
git fsck                                                                    # integrity: clean
curl -I https://raw.githubusercontent.com/TheEduardoConde/DistrictTiles/main/<file>   # what clients see
```
Results 2026-10-28: `mbtiles_handoff.manifest.json` 200, `ffx_precincts.mbtiles` 200,
`districtview-map-manifest.json` 404.

## Architecture
- Flat root, one file per package: `nova_<geoid>_<locality>.mbtiles` (10 NoVA localities), `ffx_precincts.mbtiles`,
  `va_congressional_current.mbtiles`, `va_congressional_2026_proposed.mbtiles`.
- Manifests: `mbtiles_handoff.manifest.json` (SHA-256 + size per file; `path` values are GeoData's local Windows
  paths), `va_congressional_2026_proposed.manifest.json` (provenance of the provisional VPAP layer).
- Producer: GeoData `builders.publish_to_districttiles` (dashboard "Publish DistrictTiles"). It checks the origin
  URL, checks out `main`, pulls fast-forward only, replaces `*.mbtiles` / `*.manifest.json`, commits and pushes `main`.
- Consumer: DistrictView `MapUpdateManager` reads
  `https://raw.githubusercontent.com/TheEduardoConde/DistrictTiles/main` and fetches
  `districtview-map-manifest.json`, then downloads packages and verifies size + SHA-256.

## Project rules
- **Don't hand-edit generated files; regenerate with GeoData** (`uv run geodata build ...`, then publish from the
  GeoData dashboard). Manual edits to `*.mbtiles` or `*.manifest.json` are overwritten on the next publish.
- Leave the local copy on `main` and clean: GeoData's publish runs `git pull --ff-only` here and pushes `main`.
- **Must stay public**: DistrictView downloads from raw.githubusercontent.com. Visibility changes need Eddie.
- Known gap: `districtview-map-manifest.json` does not exist (404), so in-app map updates fail until GeoData
  generates it.
- Every publish adds tens of MB of binary history; avoid unnecessary republishes.
- Where things live: repo in OneDrive (`OneDrive\Documents\My Projects\DistrictTiles`); any build output or
  caches in `C:\My Projects\_build\DistrictTiles`. No secrets.
- Methodology not yet synced; master copy in the brain's `methodology\`.
