# Prediction Ledger — Read First

**Canonical prediction ledger for Media Intelligence.** Writer: Claude. Every other process reads it; none writes it.

## Which File Is Current
- The current ledger is the **newest file in this folder named `predictions_YYYY-MM-DD.csv`** (latest date wins).
- If no dated file exists yet, the current ledger is `predictions.csv` (the 2026-10-24 version).
- Each weekly update is saved as a new dated full copy; earlier copies are never edited and serve as version history. `predictions.csv` is frozen.
- Always locate the ledger by this rule, not by a fixed Drive file ID.

## Why
Chat's Prediction Audit (Drive ID 1y7O4dxqqrtlprWrKJsrFCPJaPe_AyWOHhu1bLendi88) and Intelligence Ledger (1DtqktKTWxQ8gHY9sZMt4cs3gMA6ze3gkHj87YTpQJR0) return not-found by ID and title, and Jim confirmed they are not in Drive Trash. They are treated as lost.

## Known Gaps
- The Prediction Audit held about 62 records (P001–P062 per the Operations sheet). Their wording, tests and outcomes are not in this file and are not reconstructed.
- `external_ids` is reserved for Chat P IDs where a record here demonstrably matches one; it stays blank until a match is verified from surviving evidence.
- The Sept 19 Chat scoring (P048 confirmed, P054 failed, same Fed event) is not imported.

## Conventions
- `status`: pending, correct, wrong, unresolvable. Early resolution by the named event counts.
- `scored_by`: who applied the outcome. `resolution_note` leads with the dated scoring and keeps any prior note after `| PRIOR:`.
- New resolution tests must state rounded vs. unrounded figures for any statistical release.
- Forecast candidates from the daily scanner arrive as XP items in scanner files; Claude adjudicates weekly and assigns `pred-NNN` IDs. No other process allocates IDs.
- Original tests are frozen once written; corrections append to the note, never overwrite the test.

## History
- 2026-10-24: Became canonical. Applied 14 scorings (6 correct, 6 wrong, 2 still pending); detail in `scoring-2026-10-24.csv`. Normalized `falsifiable` to TRUE/FALSE. Added `external_ids` and `scored_by` columns.
- 2026-10-24: Switched to dated weekly copies so updates run in the cloud without any one computer.
