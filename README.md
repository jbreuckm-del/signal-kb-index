# signal-kb-index

Index of record for Jim Breuckman's signal-kb / Media Intelligence claim index. Migrated from Google Drive `AI/Transcripts/_index` on 2026-10-25.

## What lives where

| Thing | Location | Writer |
|---|---|---|
| `sources.csv`, `claims.jsonl`, `practices.jsonl`, `ingested.txt`, `schema-notes.md` | This repo | Claude |
| `predictions.csv` (the prediction ledger, master copy) | This repo | Claude only |
| Published ledger for other readers | Drive `AI/Transcripts/_index/predictions_YYYY-MM-DD.csv`, located by the rule in `LEDGER_README.md` | Claude publishes a dated copy after each ledger change |
| Raw transcripts and newsletters | Drive `AI/Transcripts/` | Capture scripts on chmo-09 |
| Scanner output (`DAILY_*`, `WEEKLY_*`, `XP-*` candidates), `seen-items.txt` | Drive | ChatGPT scanner and capture scripts |

## Rules

- **Every change is a commit.** The commit message names the mode (ingest, scoring, practice review, correction) and the source or prediction IDs touched.
- **Never rewrite history.** No force-push, no amending a pushed commit. A correction is a new commit.
- **Run the Mode 4 manifest before every push that touches records**, and paste its table into the commit message body.
- **Byte-exact files.** `.gitattributes` disables line-ending conversion; `sources.csv` mixes CRLF and LF from its Windows history and must not be normalized silently.
- **The Drive `_index` copies of these files are frozen as of 2026-10-25.** Do not edit them. `LEDGER_README.md` stays authoritative for how readers locate the published ledger.

## Migration record (2026-10-25)

Copied byte-exact from Drive. Sizes verified against Drive: `claims.jsonl` 739,707; `schema-notes.md` 88,280; `predictions.csv` 75,862; `practices.jsonl` 37,668; `sources.csv` 20,890; `ingested.txt` 6,050; `scoring-2026-10-24.csv` 3,987; `LEDGER_README.md` 2,230 bytes.

Baseline integrity at migration: 71 sources, 796 claims, 155 predictions, 24 practices. Structural, speaker-interest, lane and queue-parity checks all passed. Verbatim-quote checks were not run, because they need the raw transcripts staged; they run on the next ingest.
