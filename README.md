# gawk-data

The public dataset behind [gawk.dev](https://gawk.dev) — the raw, verifiable pipeline data archived durably. Gawk's live dashboard serves a rolling window (events expire within hours); this repo is the append-only history.

## Trust contract

Same contract as the dashboard: **every record traces to a public source.** Records are archived exactly as served by gawk's public API — no rewriting, no enrichment, no synthesis. Each line carries a capture envelope so provenance is auditable:

```json
{"v":1,"capturedAt":"<ISO8601>","endpoint":"<gawk.dev API path>","record":{...}}
```

- Archiving is **gated on gawk's own trust audit** (`/api/trust-audit`): if the live output audit reports breaches, the archive tick skips rather than persist suspect data.
- Records are deduplicated by their source event id; a record is written once and never mutated (append-only).
- Gaps are honest: if the archiver or the upstream API was down, the window is simply absent — nothing is backfilled synthetically.

## Layout

```
events/YYYY/MM/YYYY-MM-DD.ndjson     # live map events (GitHub/GitLab activity), deduped by id
snapshots/YYYY/MM/YYYY-MM-DD/
  models.json                        # OpenRouter top-weekly rankings as served
  sdk.json                           # SDK adoption series as served
  status.json                        # tool health (vendor-declared + probe) as served
  feed.json                          # curated feed cards as served
```

Events append hourly (the live window is ~4h, so hourly capture with drift tolerance loses nothing in normal operation). Snapshots are written once per UTC day (first tick of the day) and never overwritten.

## Provenance

Written by [`data-archive.yml`](https://github.com/Neelagiri65/aipulse/blob/main/.github/workflows/data-archive.yml) in the gawk repo. The archiver is fail-loud: a red workflow run means a capture problem, never silent invention.
