# Schema notes

Mode 5 output. One dated line per observation. A change to SKILL.md is proposed only when the
same observation appears three times, or Jim corrects the same thing twice.

## Observations

- 2026-08-24 — Quote elision: wrote quotes stitched with `...` across non-contiguous spans; 7 of
  71 failed the verbatim check on the seed ingest. Fixed by shortening to contiguous spans.
- 2026-08-25 — Quote elision recurred: 2 of 21 failed on the Eisman 08-14 ingest.
- 2026-08-25 — Quote elision recurred: 2 of 43 failed on the 4-source batch.
  **THIRD OCCURRENCE — acted on.** SKILL.md quote rule now forbids `...` explicitly and Mode 4
  makes the verbatim check mandatory rather than optional.
- 2026-08-24 — Missing publish date: the Huang/Dwarkesh transcript carried no date; recorded as
  `2026-08-UNKNOWN` rather than inferred. Watch whether manually-saved sources routinely lack
  headers; if so the ingest step needs a prompt for date and attribution.
- 2026-08-25 — Source encoding: fetcher wrote the title through the ANSI codepage, producing
  mojibake in both filename and header. Fixed in the fetcher (UTF-8 read + smart-punctuation
  normalization), not in the schema.
- 2026-08-25 — Source encoding: captions carried undecoded HTML entities (`&amp;`, `&gt;&gt;`).
  Fixed in the fetcher.
- 2026-08-25 — Speaker identity: a guest was introduced only by first name ("Tibo"). Recorded as
  given rather than enriched with an inferred surname. If this recurs, `sources.csv` may need an
  `identity_confidence` field.
- 2026-08-25 — Same-event divergence: three sources described the NVIDIA $500B financing deal
  with opposite valences. The index handled it via `novelty_note` cross-references, but there is
  no first-class way to group claims about one *event*. If this recurs twice more, consider an
  `event_id` field. Cost: re-pass over the existing corpus to backfill.

## Open questions not yet acted on

- Compound claims (a speaker asserts a mechanism and a dated prediction in one breath) are
  currently split into a claim plus a prediction row with no link between them. A
  `related_claim_id` on predictions would close this. Not yet seen often enough to justify it.

## 2026-08-25 — back-catalogue batch (9 sources, Sep 2025 – Aug 2026)

- 2026-08-25 — **Chronological novelty.** Three sources predate most of the corpus. Judging
  novelty against the whole index rather than against what existed *at the time of utterance*
  inverts priority: a 2025 claim looks "recycled" because 2026 speakers repeated it. Handled
  this batch by judging chronologically and recording priority in `novelty_note`. **The schema
  has no field for "novel when said" vs "novel now."** If this recurs twice more, add
  `novelty_asof`. Cost: no re-pass needed — `novelty_note` already carries it in prose.
- 2026-08-25 — **Date bounding.** Thompson (2026-05-05) quotes the Huang/Dwarkesh interview as
  "recent", which bounds that source to `<=2026-05-05`, not the recorded `2026-08-UNKNOWN`.
  Recorded the bound in the `date` field as `<=2026-05-05`. **SECOND date-precision problem**
  (first: missing date entirely, 2026-08-24). One more and `date_confidence` is justified.
- 2026-08-25 — **Event grouping, second occurrence.** The NVIDIA $500B financing structure is
  now described by five claims across four sources and three dates (bj-11 Sep 2025, baker-08
  Jun/Aug 2026, nrb-01 Aug 11, eisman2-01 Aug 14, thompson-06 Aug 18). Still no first-class
  `event_id`. One more occurrence and it is worth the backfill.
- 2026-08-25 — **New extraction failure mode: cross-file quote attribution.** Two claims were
  written with quotes drawn from a *different* source file by the same author. Mode 4 caught
  both. This is distinct from elision and is a specific risk when ingesting several pieces by
  one writer in one pass. Watch for a third occurrence.
- 2026-08-25 — **Corrections applied to seven existing claims** after back-dated sources proved
  priority errors: thompson-06, thompson-05, thompson-02, baker-05, huang-04, visser-s06 and a
  note on eisman-07. Recorded as `CORRECTED` in `novelty_note` rather than silently overwritten.
- 2026-08-25 - **Index write-back gap.** The five INTERIM track-record notes generated during
  the signal report were written to the working copy but never committed to `_index` on Drive.
  Caught on the next ingest by byte-comparing device vs. working copy. **Rule going forward:
  any pass that touches `_index` commits before it reports, including query-mode passes that
  only enrich `resolution_note`.** The container is ephemeral; Drive is the index of record.
  First occurrence.
- 2026-08-25 - **Written-source fetcher added** (`_tools\Get-Written.ps1`). RSS/Atom, prefers
  `content:encoded`. Produces the same header shape as the YouTube fetcher, so ingest is
  unchanged. Motivation, measured across all 19 sources: median errors/10k words are 0.00 for
  pasted text, 1.32 for Stratechery audio narration, 1.39 for ordinary video - narration is NOT
  cleaner than video, contrary to my assumption. The corruption driver is unfamiliar proper
  nouns (one Stratechery piece hit 50.38/10k, almost entirely Kimi and Qwen), which matters
  because `entities` drives convergence counting.
- 2026-08-25 - **Practice lane added (Jim-directed schema change, not a three-occurrence
  trigger).** Diagnostic that prompted it: the Isenberg source produced 4 claims, ALL
  `investment_relevance: none` and `falsifiable: false`, three of them `framework`. By the
  index's own noise test that source reads as pure noise - yet Mode 5 and the plugin repo
  both came from it. The schema was measuring the wrong axis, not detecting low quality.
  Change: `lanes` column on sources.csv (`claims` | `practice` | `claims;practice`, default
  `claims`), new `practices.jsonl`, lane routing in Mode 1, Mode 6 practice review, Mode 4
  extended to validate practice rows. **Additive - no re-pass needed**, all 241 existing
  claims unchanged, `lanes=claims` backfilled.
  Kept, deliberately: the 4 Isenberg claims stay in claims.jsonl because `cm-09` (Thompson,
  harness stickiness, falsifiable, investment-direct) cites remy-01 and remy-02 as
  independent corroboration. A practice source can still be a witness. This is why `lanes`
  is a list rather than a single value.
- 2026-08-25 - **Claim density baseline, by source type** (19 sources, 241 claims): youtube
  median 1.27 claims/1k words (0.71-4.25), newsletter median 2.82 (2.38-5.62). Written
  sources run ~2.2x denser, which is structural - edited argument versus conversation - and
  means any per-speaker volume comparison must control for source type. Recorded as the
  effect test for prac-001.
- 2026-08-25 - **Plugin install rejected: SKILL.md frontmatter `description` capped at 1024
  characters.** v0.3.0 shipped at 1037 because I appended practice-lane triggers to the
  existing description without measuring it. **This is the same class of error as the three
  PowerShell encoding failures: reasoned-correct, not run-correct, shipped without the check
  that would have caught it.** Fix: `scripts/validate-plugin.py` in the plugin, enforcing the
  1024 cap (warning under 60 chars headroom), kebab-case names matching folders, semver, and
  that every ```python block in a SKILL.md compiles. Both failure paths verified by negative
  test - an over-length description and a deliberately broken python block each exit 1.
  Description trimmed to 916. Shipped as v0.3.1. Run it before every package.
- 2026-08-25 - **The .plugin distribution path does not support update.** v0.3.1 rejected with
  "A plugin named signal-kb is already installed"; no uninstall/update tool is exposed to the
  session (ListPlugins is read-only). Observed state: plugin skill stuck at v0.2.0 (0 mentions
  of `lanes`), standalone account skill saved at v0.3.1 (8 mentions). **Two signal-kb skills
  with divergent schemas now exist**, and this session's listing resolves to the v0.2.0 copy -
  which would ingest without the `lanes` column and without practices.jsonl, silently writing
  an index inconsistent with Drive. This is the first time the tooling has created a
  correctness risk rather than an inconvenience. Recorded against prac-002, cost raised
  medium -> high.
- 2026-08-25 - **Plugin removal QC.** Account side clean: ListPlugins returns empty, and the
  saved standalone skill is byte-identical to what was shipped (md5 851ab00c..., 20,348 bytes,
  description 916/1024). **But this session's container still holds the stale v0.2.0 plugin
  copy** at plugins/synced/signal-kb (0 mentions of `lanes`) and still lists it as
  `signal-kb:signal-kb`; the cache does not refresh mid-session. Rule: after any skill or
  plugin change, START A NEW SESSION before the next ingest, or the stale copy may run and
  write claims without `lanes` or practices. First occurrence.
- 2026-08-25 - **Stale entry found in ingested.txt during QC.** 20 lines against 19 sources;
  the extra was the old mojibake Eisman filename left behind when the file was renamed. Dead
  state, but it would have silently skipped that transcript had the mojibake name ever
  reappeared from a re-fetch. Removed. **`len(ingested.txt) == len(sources.csv)` is now part
  of the standing QC** - Mode 4 checks claims against sources but never checked the queue
  file against sources.
- 2026-08-25 - **Visser back-catalogue ingest: 19 sources, 164 claims, 25 predictions.** Three
  findings worth carrying forward.
  (1) **Chronological novelty inversion, THIRD occurrence - the rule now has to change.** All 29
  pre-existing Visser claims were dated 2026-08-23/24; the entire back-catalogue predates them.
  Eight existing claims were corrected from novel/developing to recycled, with priority credited
  to posts up to six months earlier (visser-s03 -> vr-01, visser-s09 -> vr-06, visser-s06 ->
  vh-05, visser-n02 -> va-01, visser-s07 -> vai-03, visser-s11 -> vai-18, visser-s04 -> vr-02,
  visser-s08 -> developing). **`novelty` judged against the finished index is unsound whenever
  sources arrive out of chronological order.** Proposed: add `novelty_asof` (the date the
  judgment was made against) OR require Mode 1 to re-run novelty over the affected speaker after
  any back-dated ingest. Cost: the second option needs no schema change but must be done every
  time. Jim to decide.
  (2) **Extraction quality: 0 verbatim failures on 164 claims from five parallel subagents.**
  First clean ingest. The prompt carried the no-ellipsis rule verbatim and told each agent to
  search the file for its own quote before emitting. Fold that instruction into Mode 1.
  (3) **Novelty inflation is real and now measurable.** The index reads 301 novel / 35 recycled
  / 1 consensus across 405 claims. Even after marking 30 Visser claims recycled, 129 of his 193
  read novel. One `consensus` claim in a corpus where five speakers discuss the same six
  companies is not credible. **The convergence detection this system exists for is not working.**
  Likely cause: novelty is assigned per-source at ingest, and no pass ever re-reads across
  speakers. Audit before the next signal report.
- 2026-08-25 - **Duplicate source caught at ingest.** The RSS fetcher re-pulled Silent IPO under
  its full title; 99.6% vocabulary overlap with the manually pasted copy. Not re-ingested. The
  source row now points at the RSS copy (proper title, date and URL) after all 12 existing quotes
  were re-verified against it; the paste filename stays in ingested.txt so it is never re-ingested.
  **`ingested.txt` can legitimately exceed `sources.csv` by superseded duplicates** - the QC rule
  added earlier today needs that exception.
- 2026-08-25 - **Get-Written.ps1 filename bug.** `Clean-Name` truncates at 120 chars, which cut
  the date off "The Human Infrastructure Layer..." leaving "- 2026-0.txt". The date survived in
  the file header so nothing was lost, but truncation should preserve the trailing date. Fix:
  truncate the title portion, then append the date.
- 2026-08-25 - **Mode 6 re-pass on the Isenberg source: 4 practices -> 10.** The original ingest
  ran it through the claims schema and got 4 items. Re-reading the same transcript through the
  practice lens found six more, including the single most useful item in it - the 150-skills /
  500-hours deletion with no backups - which the claims pass dropped because it is an anecdote
  with `investment_relevance: none`. **A source re-read under the correct lane is not a
  duplicate ingest.** Rule for Mode 1: when a source is retagged into a new lane, re-extract it
  under that lane rather than assuming the original pass covered it.
- 2026-08-25 - **First `rejected` practice recorded (prac-010, the skills catalogue UI).** The
  rejection is the point: it stops the same high-cost suggestion being re-litigated every time a
  source demos a skills dashboard. Confirms the `rejected` status is carrying weight rather than
  being decorative.
- 2026-08-25 - **prac-005 and prac-006 adopted; prac-005's effect test passed on day one.** Skills
  backup built as dated immutable snapshots in Google Drive (AI\Skills-Backup), written by
  scheduled task trig_01Gyv7PbVEKUwhwqcDV1HDFo on the 1st monthly. **Chose the Drive connector
  over the device bridge because device-bridge folder grants are per-session** - a scheduled run
  would need a permission prompt nobody is awake to answer, and would silently write nothing.
  Verified by round-trip, not assumption: staged the archive back out of Drive, unzipped it, and
  confirmed all 9 authored SKILL.md files byte-identical plus all 9 content hashes matching
  INVENTORY.csv. Residual risk logged in the practice: the Drive connector is interactively
  authenticated and may be absent headless.
- 2026-08-26 - **Timmer ingest: 5 sources, 61 claims, 13 predictions. New speaker, and the first
  one who materially changes what the index can answer.** Three findings.
  (1) **The LinkedIn RSS bridge works.** linkedinrss.cns.me returned all five Weekly Asset
  Allocation Reviews as full text, not summaries. Third-party free service - treat as a
  reliability dependency, not a permanent fixture.
  (2) **ENCODING BUG, FOURTH OCCURRENCE OF THE SAME ROOT CAUSE.** 198 characters of mojibake
  across the 5 files. `Invoke-WebRequest`'s `$resp.Content` decodes as latin-1/ANSI when a feed
  omits a charset in Content-Type - the identical failure mode as `Get-Content -Raw` in
  Get-Transcripts.ps1. **Three prior occurrences were all about READING local files; nobody
  checked the HTTP path.** Fixed by decoding `RawContentStream` explicitly as UTF-8 with a BOM
  trim. Files repaired in place and re-committed before extraction, so no quote carries
  corruption. **Rule: every byte entering the pipeline gets an explicit encoding, whether it
  comes from disk or from the wire.**
  (3) **Convergence detection finally has something to work with.** Timmer arrives at
  open-weight-models-pressure-frontier-economics independently, through the Silicon Data token
  expenditure index, without citing Thompson or Visser - a genuine third distinct speaker, so
  that cluster is now `consensus` rather than propagation. And he produces the index's first
  clean head-to-head disagreements, both dated and both falsifiable: Fed hikes (Timmer, pred-075,
  resolves 2026-09-30) against Visser's no-hikes (pred-052, visser-n09); and semiconductor cycle
  cresting (Timmer pred-081) against Visser's constrained-through-2028 (pred-058), six days apart.
- 2026-08-26 - **Both new speakers reverse themselves within weeks, unacknowledged.** Timmer:
  Bitcoin "floundering around $65k" with gold to lead (jtc-13, 08-10) to "plenty of room to run"
  at $80k resistance (jtp-10/11, 08-25) - 15 days. Visser: Bitcoin decoupling from software and
  benefiting from AI (va-01, 04-11) to agentic AI as Bitcoin's "extreme headwind" (vu-02, 07-06)
  - 12 weeks. Neither notes the change. **This is only visible because both back-catalogues were
  ingested chronologically; a single recent post from either would have read as conviction.**
  Candidate schema addition: a `supersedes` field linking a claim to the earlier one it reverses.
  First occurrence - do not act yet.
- 2026-08-26 - **Audio-derived transcripts corrupt QUOTES far worse than the error rate implies.**
  The Stratechery RSS feed delivered the authoritative text of "Nvidia's Risky Business", already
  in the index from an audio narration transcript. 93.4% vocabulary overlap, but **5 of 12 quotes
  did not match the text Thompson actually wrote**, and two errors were substantive: Google Cloud
  prior-quarter growth was 63%, not 62% (the wrong figure had propagated into the claim text), and
  "$500 million into US railway bonds" had lost the word "million". A 1.32-errors-per-10k-words
  rate sounds negligible; one wrong word inside a 30-word quote destroys the whole quote, so the
  per-quote failure rate was 42%. **Rule: when a text rendering of an audio-derived source
  arrives, switch the source file to it and re-verify every quote.** Done here - source repointed,
  5 quotes re-anchored, 1 claim figure corrected, all 12 now verify.
- 2026-08-26 - **BUG IN THE MODE 4 NORMALIZER ITSELF - false failures.** It collapsed whitespace
  BEFORE stripping punctuation, so punctuation sitting between two spaces (an em-dash aside like
  "the risk - unmarked, unlike equity - is") left a double space and broke the substring test.
  Two of the six apparent failures above were this bug, not real. Fixed to strip punctuation to a
  space first, then collapse. **Checked for retroactive damage: all 476 pre-existing records pass
  under both the old and new normalizer, and zero verdicts change** - the bug only bites when a
  quote is compared against a DIFFERENT rendering of the same text, which had never happened
  before today. **The verification gate is itself code that can be wrong; this is the first time
  it has been tested against an independent rendering rather than the file it was extracted from.**
- 2026-08-26 - **New lane value: `lanes=skipped`.** The Stratechery feed also delivers weekly
  "This Week in Stratechery" roundups (titles of the form "2026.NN: ..."), which are link lists
  with blurbs largely written by Andrew Sharp, not analysis. They carry no original assertions,
  so extracting from them would produce thin restatements of already-indexed material. Recorded
  with source rows and `lanes=skipped` so provenance is kept and the judgement is not
  re-litigated every week, but no claims are emitted. Mode 4 now asserts a skipped source
  produces zero claims. **Expect one of these every Friday.**
- 2026-08-27 - **MY ERROR, corrected: a claimed "unacknowledged reversal" by Timmer was not one.** The
  2026-08-26 note said both new speakers reversed themselves within weeks. Checking against actual price
  data: Timmer's 08-10 claim was CONDITIONAL - "gold will likely be the leader of the pack until Bitcoin
  finds a new catalyst of its own" - and Bitcoin closed 63,970 on 08-10 then 78,338 on 08-21, +22.5% in
  eleven days. His condition was met; his 08-25 position is that conditional resolving, not a
  contradiction. Withdrawn from jtp-10, jtp-11 and pred-083. The Visser April-to-July reversal (va-01
  versus vu-02) stands. **Root cause: I compared two claims by date without re-reading whether the
  earlier one contained a condition.** Rule: before recording a reversal, check whether the earlier
  claim was conditional and whether its condition resolved in between.
- 2026-08-27 - **Mode 7 added: live board, claim vs tape.** Checks dated claims naming a specific
  instrument and level against current prices, between Mode 3 horizon scorings. Rules that matter:
  a freshness gate (drop anything over ~90 days unless price is still near the level); baseline
  dates MUST be scored from actual closes via get_time_series, not a rolling-7-day proxy - using
  the proxy produced the OPPOSITE verdict on pred-018 before it was caught; every row carries a
  disconfirmer; and rows already reflected in the price are flagged as such, because a correct
  call and an available idea are different things. Never sizes, never recommends, never changes
  `status` - only Mode 3 does that.
  **Data limits found:** the connected market feed has no bonds, no yields, no indices directly,
  no options. That blanks out the rate-driven claims INCLUDING the sharpest disagreement in the
  index (Timmer's 4.5% danger zone versus Eisman's 5% demarcation). A Treasury yield source is
  now the single highest-value data addition.
  **First run, 2026-08-27:** Timmer 3 tracking / 0 against / 1 live; Visser 0 / 2 / 0. Two to
  three weeks of tape - a thermometer, not a track record, and labelled that way on the board.
  Notable: Timmer's stated fact that gold had a 7%+ week verifies exactly against independent
  price data (18 Aug 4,336 to 25 Aug 4,659 = +7.4%), which is itself a data-quality signal about
  the speaker. Notable the other way: pred-018 (Visser, Bitcoin seven-sigma) is FLAT at -0.1%
  against its own 24 Aug baseline with 58 days left and an analogue requiring roughly a double -
  he published at the top of a move that had already happened.
- 2026-08-27 - **Convergence freshness measure added to Mode 2.** Four numbers per cluster,
  computed from each distinct speaker's FIRST statement: age, quiet, arrival span, and the
  arrival GAPS between successive speakers. **The gap sequence is the measure that matters** -
  `+249d -> +63d -> +2d -> +0d -> +4d` and `+227d -> +31d` have similar spans and mean opposite
  things, and any average destroys the distinction. Four shapes named: Independent, Delayed
  crowd, Event echo, Weaker-than-it-looks. Deliberately NOT thresholded - the boundaries move
  with publication cadence, and a weekly writer cannot be judged on a quarterly writer's spacing.
  First run over 8 clusters:
    Independent    - Inference dwarfs training (Huang 2025-09-26 -> +227d Thompson -> +31d Baker,
                     three vantages, no cross-citation, 8/8 falsifiable). The only clean one.
    Delayed crowd  - AI capex debt-financed (Huang alone 249 days, Thompson, then FOUR speakers
                     in 6 days). Also Debasement (Visser alone 166 days, then 3 in 14 days).
    Event echo     - Open-weight pricing: Baker, Thompson, then 3 speakers in 8 days around the
                     Kimi K3 release. Not five confirmations.
    Weaker         - Power/energy: Huang and Thompson share 2026-05-05 because Thompson's piece
                     quotes the Huang interview. Two origins, not three.
  **Two findings that generalise.** (1) A delayed crowd identifies who was early AND warns the
  arriving consensus is probably priced - gold ran 4,077 to 4,621 between Visser's February
  position and the August arrivals. (2) A dissent can hide inside an apparent consensus: Baker
  holds the open-weight thesis and draws the OPPOSITE investment conclusion ("a token is a
  token"), which a speaker count alone reports as agreement.
  **Limit stated on the board:** `quiet` does not discriminate yet - every cluster was restated
  within 23 days because the corpus is weeks old and was loaded back-catalogue-first. Re-check
  after the October scoring pass.
- 2026-08-27 - **Ingest: 5 sources, 81 claims, 12 predictions, 12 practices.** Three Compound
  episodes (TCAF 253, TCAF 256, Five Reasons WAYT), Pompliano/Yampolskiy, and Isenberg #2
  (practice lane only). New credentialed voices: **Bill Dudley** (relayed - The Compound reading
  his Bloomberg op-ed, 10 claims, stated end-2027 horizon) and **Alex Kantrowitz** (19 claims,
  verified introduction). 81/81 quotes verify under normalized checking.
- 2026-08-27 - **New structural limit: multi-host audio without diarization cannot be attributed.**
  yt-dlp auto-captions carry no speaker labels. On TCAF 256 the panel contains co-host Michael
  Batnick AND guest Michael Santoli, and the transcript renders the latter "Michael Sani" -
  garbled AND colliding on first name. There is no reliable way to split them from text alone,
  so all 21 claims from that episode are attributed to `The Compound (TCAF 256 panel)` rather than
  guessed at. **Rule: when a multi-speaker audio source cannot be split, attribute to the show,
  not to a speaker.** A wrong attribution is worse than a coarse one - it corrupts the per-speaker
  calibration record that Mode 3 exists to build. Cost of the rule: those 21 claims can never
  enter a convergence cluster as an independent voice, and should not be counted as one.
  Where a guest IS explicitly introduced by full name (Kantrowitz on TCAF 253, Ali K. Miller on
  Isenberg #2, Dudley by op-ed citation), attribution stands.
- 2026-08-27 - **MY ERROR, corrected: my spot-check method was wrong, not the extraction agent.**
  I reported a quote as missing "at offset -1" using a raw `t.find()` against the transcript.
  Raw find fails on any punctuation or whitespace difference. Re-checked with the same normalizer
  Mode 4 uses, all 87 quotes verified. **Rule: never spot-check a quote with a method weaker than
  the one the pipeline uses.** An ad-hoc check that is stricter than the real gate manufactures
  failures and wastes a verification cycle on nothing.
- 2026-08-27 - **Novelty grading was moved to subagents with a chronological digest and a
  mechanical verifier.** Each agent got `prior-digest.txt` (all indexed claims, date-sorted) plus
  its batch, and was required to cite a real prior claim_id for any non-`novel` grade. A verifier
  then checked every cited id exists AND predates the new claim. This is worth keeping: it catches
  the two failure modes graders actually have - inventing a plausible id, and judging a July claim
  against an August one. 81/81 passed; the only two flags were false positives from string-comparing
  the `<=2026-05-05` date form.
- 2026-08-27 - **Practice lane: 12 entries from Ali K. Miller, all logged `proposed` or `rejected`,
  none adopted.** Three are worth revisiting and are marked as such in `effect_note`: a goals doc to
  give Mode 5 something to rank relevance against (prac-012, cheapest and best fit); requiring every
  scored resolution to state a CAUSE rather than a verdict (prac-019, would turn Mode 3 from a
  scorecard into a calibration instrument); and a genuinely separate verification subagent rather
  than the ingesting session checking its own work (prac-014 - this is the exact failure mode behind
  the description-length error and the repeated mojibake). Two were rejected outright: "do smart
  things" as a scheduled-task prompt (prac-011 - it works only because Miller's agent reads email,
  calendar, Stripe and GitHub; this environment has two Drive folders and no such surface), and
  installing a third-party GitHub skill unread (prac-022).
- 2026-08-27 - **prac-002 retired to `superseded`: the plugin route is dead.** Jim deleted
  `_tools\\signal-kb.plugin` and `_tools\\signal-kb-plugin\\`; `plugins/synced` is empty and no
  signal-kb registration survives outside `.trash`. The practice said "keep skills in a GitHub repo
  published as a plugin, so there is one source of truth with version history." Neither half
  delivered: the `.plugin` upload path is install-only with no update mechanism, so it produced TWO
  divergent signal-kb skills instead of one - the exact opposite of the stated goal - and the repo
  was never pushed, so the version history never existed. **Lesson for the practice lane: a practice
  can fail because the tooling it assumes does not exist in this environment, which is a different
  failure from the practice being wrong.** Remy's advice is sound where a plugin update path ships;
  it was never sound here, and `effect_test` is what surfaced that rather than a judgement call.
  Function on retirement split three ways: backup/rollback/integrity → prac-005 (dated Drive
  snapshots, SHA-256 manifest, effect test PASSED); single-source-of-truth → direct account sync,
  verified 2026-08-27 when the saved SKILL.md came back byte-identical (md5 4d2c2fb8...) to the
  delivered file; **multi-machine distribution → nothing. That is an open gap, recorded as accepted
  cost.** The skill syncs to the account; `_tools\\*.ps1`, `yt-dlp.exe` and the `.bat` launchers do
  not, and reach a second machine only by hand-copy from Drive.
- 2026-08-27 - **Skill save verified, not assumed.** `skills/synced/signal-kb/SKILL.md` is
  byte-identical to the delivered file (26,547 bytes, same md5), carries all 7 modes including
  Mode 7 and the Mode 2 freshness measure, and passes validate-skill.py at 803/1024 description
  chars. **Rule: after any skill save, diff the synced copy against the delivered file.** The
  registry listing showing the new description text is suggestive, not proof - a description can
  update while the body does not, and the plugin episode above is exactly what a silent version
  mismatch looks like from the outside.
- 2026-08-27 - **New field: `speaker_interest` on claims AND predictions, backfilled across all
  559 claims and 101 predictions.** Values `independent` / `interested` / `relayed` / `unknown`.
  It answers exactly one question: does the speaker hold INSIDER or PRINCIPAL standing at an
  entity the claim directly concerns? Result: 489 independent / 60 interested / 10 relayed on
  claims; 86 / 13 / 2 on predictions. `interested` speakers: Huang, Nadella, Altman, Tibo
  (OpenAI), Petersen (Flexport CEO on freight and trade), Pompliano (runs Bitcoin-focused asset
  management, claims are about Bitcoin). `relayed`: Dudley, whose op-ed reached the index through
  The Compound reading it aloud.
  **Why this was needed.** Adding NVIDIA earnings materials forced the question, but the problem
  was already present and invisible: Huang alone holds 31 claims and 8 predictions in this index,
  all of them forecasts about the company he runs, and Mode 3 was going to average his hit rate
  in with Eisman's and Thompson's and report one number. Those numbers mean different things. An
  insider has better information AND a duty to promote, pulling in opposite directions, and the
  blend is uninterpretable. Mode 3 now scores the cohorts in SEPARATE tables and is told that a
  systematic high or low bias in an insider's forecasts about their own entity is itself the
  finding, more useful than the hit rate.
  **The limit, written into SKILL.md so the field is not over-trusted: it does NOT track
  position-holding.** Baker, Gerstner, Tang and Fox are investors with real exposure to what they
  discuss and all classify as `independent`, because they are not insiders. That is a genuine
  bias the field does not capture. `independent` means "not an insider" - never "disinterested".
  Position-holding remains untracked and must be stated in prose when it matters.
  **Enforcement:** Mode 4 now asserts the field is present and valid in BOTH files and that no
  speaker carries different values across them. Cross-file drift would silently corrupt the
  cohort split, which is the one thing this field exists to make possible.
  `unknown` is a first-class value: use it rather than guessing an affiliation. Andrew Fox was
  the near-miss here - resolved to `independent` only after checking his introduction in the
  BG2 transcript ("Gavin Baker... he's brought his main guy, Andrew Fox"), which establishes him
  at Baker's fund and NOT as a SpaceX insider.

## 2026-08-27 - earnings-kb split out as a separate skill

- **New sibling skill `earnings-kb`.** Reported figures and company guidance for NVDA, MSFT,
  GOOGL, AMZN, META, AVGO, TSM, ORCL, MU. Storage `_earnings/`: `filings.csv`, `guidance.csv`,
  `deltas.jsonl`. **Kept separate from signal-kb deliberately** - a claim is interesting because
  nobody else said it, a filing line is interesting because it MOVED. Those are opposite tests
  and one schema cannot serve both. Cross-reference is pointers only, both directions, neither
  skill editing the other's records: `verified_against` + `verification_note` on a claim (both
  OPTIONAL, read with `.get()`), and `bears_on` + `direction` on a delta. Every id is asserted
  to resolve in both gates - a dangling pointer is worse than no pointer.
- **The skill's core principle: delta engine, not summarizer.** The level is priced; the change
  often is not. First run bore this out - the headline was $96.2B revenue, which every wire
  carried, while the actual signal was supply commitments going $119.0B -> $279.0B (+134.5%),
  primarily memory, which sits in the CFO commentary and no headline.
- **MY ERROR, caught by the delta engine on its first run, and it is the case FOR the skill.**
  Reading NVIDIA's year-over-year lines I told Jim the marginal buyer was shifting away from the
  hyperscalers, because ACIE grew +138% against hyperscale's +102%. The sequential increment says
  the opposite: of the $13.754B added to Data Center quarter over quarter, hyperscale took
  $10.831B (78.7%) and ACIE took $2.923B (21.3%). Hyperscale grew 28.6% sequentially against
  ACIE's 7.8%. The YoY gap is a base effect from ACIE's small prior-year figure. **Rule now in
  the skill: always report the share of the SEQUENTIAL INCREMENT by segment alongside - never
  instead of - year-over-year rates.** YoY is the number that gets quoted and it is the one that
  hides a mix shift. Root cause: I read growth rates instead of computing where the marginal
  dollar went.
- **New failure mode named: `quote_status`.** This skill has two fetch paths and they do NOT
  carry the same guarantee. A quote from a staged source file can pass the normalized substring
  gate; a quote from a web fetch that renders and summarises the page cannot. On the first run I
  assembled two "quotes" from summary prose - plausible sentences that were never returned as
  quoted strings. **Assembling a plausible sentence and calling it a quote manufactures
  evidence** and is the worst failure either skill can commit. Both were blanked and marked
  `removed-not-verbatim`; five more are marked `unverified-fetch` and may not be presented as
  verbatim. Zero rows in the first run are `verified`. The gate now asserts a quote is present
  if and only if the status claims one.
- **cmp-10 resolved, and NOT the way I said I would resolve it.** I had told Jim I would mark the
  "$29 of earnings" figure `garbled`. Reading the transcript at that point shows clean, coherent
  text with adjacent figures that parse ("EBIT 61 billion. That would be 102% year-over-year"),
  so it is the speaker's number, not caption damage - `garbled` stays `false`. Three findings
  kept apart in `verification_note`: (1) SPEAKER ERROR - he stated the guidance midpoint as $92B
  against NVIDIA's actual $91.0B, though his derived 97% YoY is internally consistent with his
  own wrong input; (2) OUTCOME - actual $96.2B, beating the guide by 5.7% and clearing the top of
  the stated band by 3.7%; (3) UNRECONCILED - "$29" matches no reported figure under any unit
  reading, and is not even consistent with his own adjacent $61B EBIT, which at a 16-18% tax rate
  implies roughly $50B of net income. **Left standing unexplained rather than guessed at or
  deleted.** The general rule is now written into both skills: a verification NEVER rewrites a
  claim. Separate speaker error, transcription damage, and comparing against the wrong figure -
  the third is usually your own mistake. Erasing a speaker's wrong number destroys exactly the
  calibration record the index exists to build.
- **Stated coverage limit.** The index's #2 and #4 most-discussed entities are OpenAI (69 claims)
  and Anthropic (53), and neither files. Much of the AI-capex argument resolves only indirectly
  through hyperscaler disclosures about them. The skill is required to say when a check is
  indirect and name the proxy, and never to present a hyperscaler's disclosure as the private
  company's own reporting.
- **Known gap, not solved:** call Q&A. Prepared remarks and CFO commentary are covered by EDGAR
  and IR materials. Third-party transcripts can be read for excerpts but not archived wholesale,
  so the highest-signal part of a call is not in the corpus. When a specific call matters Jim
  supplies a transcript from his broker and it is staged as a normal source.
- 2026-08-27 - **Working convention: build threads and query threads are separate, and neither
  is long-lived.** Decided after this session hit a context compaction mid-build and lost
  fidelity that had to be recovered by re-reading files.
  - **Build thread** - schema changes, new fields, skill edits, debugging, backfills. Iterative,
    lots of dead ends, reasoning about the machinery rather than the content.
  - **Query thread** - ingest a quarter, ingest transcripts, run the live board, score
    predictions, answer a question. Starts from the files, ends with a committed result,
    disposable.
  **The split is by TASK TYPE, never by subject.** Splitting earnings from podcasts into
  separate threads was considered and rejected: the two indexes are built to cross-reference,
  and the NVDA FY27Q2 analysis only worked because one pass read the filings AND the claim
  index together. A topic split would break the thing the pointer contract exists to enable.
  **Neither kind of thread should be kept alive as a permanent home.** A long thread recreates
  the dependency this file exists to remove - and once it compacts, it is actively WORSE than a
  fresh session that reads these notes. If a new thread cannot pick up the work after reading
  this file, the defect is in this file. Fix the file; do not keep a thread on life support.

## 2026-08-27 - MRVL FY27Q2: ingest BLOCKED, baselines loaded

- 2026-08-27 - **The ingest was requested before the filing existed.** Instruction said Marvell
  "reported today after the close"; the request arrived 09:59 ET and Marvell's own IR notice puts
  the call at 1:45 p.m. Pacific (16:45 ET), with the release ahead of it. EDGAR confirms the most
  recent MRVL 8-K carrying Items 2.02/9.01 is 2026-05-27 (Q1 FY27) - **no Q2 FY27 filing exists.**
  Nothing was ingested. **Rule for Mode 1 step 1: confirm the 8-K or IR release actually exists on
  EDGAR before extracting anything.** "Reported today" is a statement about the calendar, not about
  the wire, and a skill whose job is figures must never fill a gap the source has not yet filled.
  This is the failure the whole `quote_status` apparatus exists to prevent, arriving one layer
  earlier - at the level of the filing rather than the quote. First occurrence.
- 2026-08-27 - **A ticker can be listed as covered and carry zero rows.** MRVL is in SKILL.md's
  coverage list and had no baseline in `filings.csv` at all, so an FY27Q2 ingest would have had
  nothing to delta against and would have degraded into the summarizer the skill exists not to be.
  **Candidate QC for Mode 4: assert every ticker in the coverage list either has at least one
  `filings.csv` row or is explicitly listed as not-yet-baselined.** Loaded MRVL-FY26Q2 (YoY base)
  and MRVL-FY27Q1 (sequential base) plus the five FY27Q1-issued guidance rows as `pending`, so
  tonight's print can be scored the moment it lands. First occurrence.
- 2026-08-27 - **`guidance.csv` has no `quote_status` column and `deltas.jsonl` does.** All five
  MRVL guidance quotes came back through a rendering web fetch, which by this skill's own standard
  is `unverified-fetch` and may not be presented as verbatim - but the guidance schema has nowhere
  to record that, so the rows look as authoritative as a gate-checked quote. The three pre-existing
  NVDA guidance rows have the same undeclared provenance. **Candidate schema addition: carry
  `quote_status` on `guidance.csv` with the same four values and the same present-iff-claimed
  assertion in Mode 4.** First occurrence - not acted on. Interim: provenance is stated in prose
  here and in the run report.
- 2026-08-27 - **Segment redefinition on the first non-NVDA ticker.** Marvell collapsed five end
  markets to two (Data center; Communications and other) between FY26Q2 and FY27Q1. Recorded as
  `MRVL-FY27Q1-01`, `kind: disclosure`. A two-way recast of FY26Q2 is derivable and is stored in
  that row's notes marked DERIVED, never as a reported figure. **No FY27Q2-vs-FY26Q2 segment delta
  may be computed as though the series were continuous.**
- 2026-08-27 - **Two source-access limits found, both worth solving before tonight.**
  (1) `device_bash` could not mount the granted `Transcripts` folder in this session; work ran on
  staged copies and is committed back by `device_commit_files`. (2) EDGAR's `browse-edgar` 10-Q
  index and `data.sec.gov/submissions` both refused or failed to yield the FY27Q1 10-Q, so
  `supply_commitments` is EMPTY on both MRVL baselines rather than estimated. Purchase and supply
  obligations were the single highest-signal figure of the NVDA run; without an FY27Q1 baseline
  there is no delta to compute on it for MRVL. **Fix path: the 8-K Archives directory listing DOES
  resolve, so navigate to the 10-Q by accession rather than by form-type index.**
- 2026-08-27 - **Quote provenance across the whole earnings corpus is still 0 `verified`.** 5
  `unverified-fetch`, 2 `removed-not-verbatim`, 2 `none`. No earnings quote in this KB has ever
  passed the substring gate, because the fetch path renders and summarises rather than returning
  raw text. **The only route to `verified` is a saved source file.** Standing ask: save the release
  and CFO commentary text into the source folder at ingest time.

## 2026-08-27 - NVDA FY27Q2 earnings call transcript ingested into earnings-kb

- 2026-08-27 - **The Q&A gap is closed for one filing, and it was worth more than the release.** Jim
  saved the SA transcript of the Q2 FY27 call to the Transcripts root. 11 new deltas
  (`NVDA-FY27Q2-09` .. `-19`) and 8 new `guidance.csv` rows came out of it, **none of which existed
  in the press release or the CFO commentary**: the first full-year guide NVIDIA has ever issued
  (FY28 revenue +~70%, explicitly supply-constrained against ~100% demand), a gross-margin path two
  periods further out than required (74% Q3 -> 71-72% trough in Q4 -> 72-73% FY28) with memory
  pricing named as the cause, ~25% of FY28 revenue attributed to AI labs leaning on NVIDIA's balance
  sheet, the six named capital providers and the take-or-pay-plus-revenue-share structure, AWS at
  +2 million GPUs through FY29Q2, DSO out to 60 days, and revenue per gigawatt at 18 -> 25 -> 40 USD
  billions. **The skill's standing note that Q&A is "a known gap" understates it: prepared remarks on
  the call also carried material the written materials did not.** Rule: when a transcript is
  available, ingest it as part of the quarter, not as an optional extra.
- 2026-08-27 - **First `verified` quotes in earnings-kb, ever.** Provenance went 0 verified / 5
  unverified-fetch / 2 removed-not-verbatim to **13 verified / 3 unverified-fetch / 2
  removed-not-verbatim / 2 none**. All 40 candidate quotes passed the normalized substring gate on
  the first pass, and every one was searched in the file before being written. **The only thing that
  changed is that a raw source file existed.** This is the concrete argument for the standing ask:
  a saved source file is the difference between evidence and paraphrase.
- 2026-08-27 - **SOURCE CONFLICT, and the transcript lost. Recorded as `NVDA-FY27Q2-19`.** The call
  transcript states hyperscale grew 13% sequentially and ACIE 25% - the OPPOSITE ordering to the
  recorded figures, which give hyperscale +28.6% and ACIE +7.8%. **Rejected on arithmetic against a
  third source**: at 13% growth FY27Q1 hyperscale would have been 43.36B, or 57.6% of Data Center,
  and the FY27Q1 CFO commentary states hyperscale was "approximately 50%" (37.869/75.246 = 50.33%).
  The dollar LEVELS agree across both sources to rounding (49 vs 48.7; 40 vs 40.3); only the
  percentages are damaged. This is the audio-derived-transcript failure mode logged 2026-08-26,
  recurring in a new place - **figures inside prepared remarks, not quotes.** SECOND OCCURRENCE.
  **Rule applied: when an audio-derived transcript and an authoritative text disagree on a figure,
  the text governs - but the disagreement is RECORDED as a delta, never silently dropped.**
  Nothing in `filings.csv` was altered.
- 2026-08-27 - **The FY27Q2 mix conclusion survived a second challenge, and I nearly inverted it a
  second time.** The 2026-08-27 note above records inverting the hyperscale/ACIE read once already
  by using YoY rates instead of the sequential increment. The transcript's stated percentages would
  have inverted it AGAIN, back to the original wrong answer. What stopped it was going to the FY27Q1
  commentary for an independent anchor rather than trusting either Q2 source. **Rule: a mix
  conclusion is only as good as the PRIOR-quarter figure it is differenced against - verify that
  one against its own source before revising the conclusion.**
- 2026-08-27 - **A one-`direction` field forces splitting a delta when pointers disagree.** The FY28
  guide confirms the supply-constrained cluster (pred-032, pred-064, vai-11, vx-05) and contradicts
  the 40-month-cycle peak call (pred-081). Those cannot share a row, so they were written as
  `-09` (confirms) and `-10` (contradicts) over the same disclosure. This is the right outcome and
  the schema handles it, but it means **one disclosure can legitimately produce several deltas and
  `delta_id` count is not a measure of how much was found.** Noted so a future pass does not
  "de-duplicate" them.
- 2026-08-27 - **Deliberate refusal to credit a correct prediction for the wrong mechanism.**
  pred-034 (Thompson: NVIDIA's margins become hard to sustain) is about to look right - guidance
  now shows a 350bp peak-to-trough - but the cause management names is upstream MEMORY pricing, not
  TPU and Trainium commoditisation. Written as `complicates`, not `confirms`, in `-12`. **Mode 3
  should score outcome and mechanism separately or this distinction will be lost at horizon.**
  This is the same instinct as prac-019 (require a scored resolution to state a CAUSE), and is the
  second independent argument for it.
- 2026-08-27 - **`guidance.csv` still has no `quote_status`, and this ingest made the gap concrete.**
  The file now holds rows with genuinely different provenance side by side: 8 new NVDA rows whose
  quotes are gate-verified against the transcript, 3 older NVDA rows from a rendering web fetch, and
  5 MRVL rows from a rendering web fetch - and nothing in the schema distinguishes them. SECOND
  OCCURRENCE today. **One more and the column goes in**, with the same four values and the same
  present-iff-claimed assertion Mode 4 already applies to `deltas.jsonl`.
- 2026-08-27 - **Scope held: the transcript was NOT extracted into signal-kb.** It carries a great
  deal of Huang claim material (open vs closed models, agentic compute multiples of 15-100x, AGI
  framing, custom-silicon dismissal) and he already holds 31 claims and 8 predictions there as an
  `interested` speaker. That is a signal-kb ingest with its own novelty grading and `lanes` routing,
  and doing it silently inside an earnings pass would have written claims without either. Left as a
  separate decision for Jim. **The pointer contract worked as designed in the other direction: 29
  signal-kb ids were verified to exist before any `bears_on` was written, and all 29 resolved.**

## 2026-08-27 - OpenAI Hugging Face incident report ingested (signal-kb)

- 2026-08-27 - **NEW FAILURE MODE: the web fetcher silently REWRITES first-person source text.**
  Jim pasted an openai.com URL. `WebFetch` returned what looked like full body text, but it had
  converted OpenAI's "we" and "our" into "the organization" and "OpenAI" throughout - "We consider
  this incident a warning shot" came back as "The organization considers this incident a warning
  shot". **Every quote extracted from it would have failed Mode 4, and if the source file had never
  been staged the gate would have reported them as `unchecked` rather than failed** - which is how a
  paraphrase gets into the index wearing quotation marks. This is the same class as the
  `removed-not-verbatim` problem in earnings-kb, one step earlier: not an assembled sentence, a
  silently reworded one. **Rule: `WebFetch` output is never a source file. It may be used to decide
  whether a source is worth ingesting; it may never be the text quotes are drawn from.**
  First occurrence.
- 2026-08-27 - **Capture path that worked: Claude in Chrome `get_page_text`.** Returned the real
  first-person article text, which was written to the Transcripts root as a normal source file with
  the standard header and then ingested through the ordinary path. 15/15 quotes verified. This is a
  third capture route alongside `Get-Transcripts.ps1` and `Get-Written.ps1`, and it is the only one
  available for an arbitrary URL that is not in a feed. **Cost/limit: it needs Jim's browser
  connected and a question answered, so it is unavailable to a scheduled run** - the same constraint
  that sent prac-005 to the Drive connector rather than the device bridge. Worth adding to
  `_tools\` guidance as the manual fallback for one-off URLs.
- 2026-08-27 - **`device_bash` could not mount the granted Transcripts folder this session**
  (second time today). All work ran on staged copies with `device_commit_files` writing back.
  Not blocking, but it means the byte-comparison QC has to be done by size and re-listing rather
  than by diffing in place. If this recurs the mount should be treated as unreliable rather than
  as the default path.
- 2026-08-27 - **First `research`-type source and first corporate first-party document in the
  corpus.** Every prior source is `youtube` or `newsletter`, and every prior speaker is a person or
  a show. `OpenAI` enters as a speaker with `speaker_interest: interested` - it is the principal
  describing its own failure and its own remediation, which is the definition of the field. The
  `type` taxonomy already had `research` and `earnings` values that had never been used; `research`
  fits and no schema change was needed.
- 2026-08-27 - **Density 3.04 claims/1k words on 4,932 words**, against a newsletter median of 2.82
  and a YouTube median of 1.27. In range for an edited written source. Predictions were deliberately
  thin - 3 from a 5,000-word document, one of them recorded `unresolvable` with `falsifiable: false`
  because "developers and defenders will have to prepare for AI-enabled attackers" names no
  threshold, date or actor. **A corporate incident report is almost entirely observational; that is
  the finding, not a shortfall in extraction.** Both scoreable predictions carry
  `horizon_basis: inferred` because the source says "soon" and gives no date.
- 2026-08-27 - **Novelty came out 11 novel / 4 developing, and the novelty-inflation warning of
  2026-08-25 applies.** Mitigations actually run: every non-`novel` grade cites a real prior
  claim_id, and the mechanical verifier was run over all 14 citations - all exist and all predate
  2026-08-26. **But the grading was done in-session rather than by subagents with a chronological
  digest, which is the improvement adopted earlier today.** That is a process regression, and it is
  the argument for prac-014 (a genuinely separate verification subagent rather than the ingesting
  session checking its own work) appearing for the third time. The high novel count is defensible
  here - AI-safety incident forensics is a genuinely new domain for a corpus that is otherwise
  macro and semis - but it should be re-checked at the next convergence pass.
- 2026-08-27 - **A two-day-old claim was QUALIFIED by a primary document, and NOT rewritten.**
  tibo-03 (Tibo, OpenAI, 2026-08-24) says OpenAI paused frontier RL and "resumed only against a
  defined set of principles". OpenAI's own report on 2026-08-26 says some workloads are slowly
  resuming but **"Our largest planned frontier RL run remains on hold"**. Compatible if Tibo meant
  partial resumption; his phrasing does not read that way. Recorded in oai-13's `novelty_note` as
  two OpenAI-side sources two days apart leaving different impressions. **tibo-03 was left standing
  and unedited** - per the standing rule, a verification never rewrites a claim, and an
  interested-speaker impression that the primary document does not support is exactly the
  calibration evidence the index exists to hold.
- 2026-08-27 - **Schema gap found by a question the index cannot answer.** Every capex claim in the
  index (vai-11 Visser, bs-11 Gerstner, pred-032 Thompson) models compute demand as training plus
  serving. This source names a third category - safety and chain-of-thought monitoring as a
  permanent compute overhead on both training AND tool-enabled inference, mandated by policy at
  GPT-5.6 Sol capability and above. **There is no way to ask the index "which claims account for a
  monitoring tax on compute demand" because nothing tags it.** Recorded as oai-14 with
  `domain: [AI, capex]`. Not a schema change - `domain` already carries it - but flagged so a future
  convergence pass looks for a cluster here. First occurrence.

## 2026-08-27 - prac-014 rewritten and adopted; verification manifest adopted (prac-023)

- 2026-08-27 - **Jim's decision: rewrite prac-014 in place as a source-independence adversary and
  mark the observer framing superseded.** Miller's original - a general observer agent watching
  the workforce for friction and missing access - was rejected on two grounds. (1) **Wrong failure
  class.** Classifying every incident in this file: every mechanical gate that exists and RUNS has
  a perfect record - four quote-elision catches, cross-file quote attribution, the novelty-citation
  verifier, 81/81 and 15/15 clean. **No quote failure has ever escaped Mode 4.** The failures that
  did escape were figure conflicts between two renderings, and all three were caught by fetching a
  DIFFERENT source, never by a closer re-read. (2) **Structural.** An observer agent I have to
  remember to spawn has precisely the same failure mode as the check I forgot to run, which is the
  failure it was meant to fix. The coverage half of Miller's idea went to prac-023 instead, where
  code enforces it.
- 2026-08-27 - **prac-023 adopted: the verification manifest. NOT-RUN is a failure state.** The
  recurring failure in this system is COVERAGE, not detection - three occurrences of "the check
  existed, the check was correct, and it never ran": the SKILL.md description-length error, the
  `_index` write-back gap, and novelty grading done in-session. All three were reported as clean
  runs. Mode 4 (signal-kb, 10 checks) and Mode 3 (earnings-kb, 9 checks) now emit PASS/FAIL/NOT-RUN
  per check and a hard verdict, and the manifest table must be pasted into the report rather than
  summarised.
  **Validated by negative test before adoption, not by reasoning** - which is the correction to the
  "reasoned-correct, not run-correct" pattern logged on 2026-08-25. Three dry runs against the live
  index: today's actual run returns INCOMPLETE with `novelty-by-subagent` FAIL; nothing declared
  returns three NOT-RUN; a new source whose file was never staged returns FAIL on `quotes-this-run`,
  which is the hole a paraphrased web fetch walks through wearing quotation marks.
  **The one way to defeat it, stated in the practice: declaring True without having done the thing.**
  No code can stop that. What the manifest buys is converting a silent omission into a deliberate
  false statement.
- 2026-08-27 - **New earnings-kb check: `baseline-present`.** Reports which covered tickers hold no
  `filings.csv` row. Currently 2 of 10 - AMZN, AVGO, GOOGL, META, MSFT, MU, ORCL and TSM are named
  in coverage and carry nothing to difference against. Not a hard failure, but it is the state in
  which the next ingest quietly degrades into the summarizer this skill exists not to be.
- 2026-08-27 - **The OpenAI source was RE-EXTRACTED under the practice lane, not assumed covered.**
  Retagged `claims;practice` and re-read through the practice lens, per the rule set on 2026-08-25
  after the Isenberg re-pass took 4 practices to 10. Yielded prac-023 (manifest, adopted) and
  prac-024 (stop safely on a broken or non-existent source, proposed - held back only by the
  two-adoption cap in Mode 6). Both quotes gate-verified.
- 2026-08-27 - **SCHEMA STRAIN: rewriting a practice in place breaks the quote-to-practice
  correspondence.** prac-014's `quote` is still Miller's words about an observer agent; the
  `practice` field is now the adversary we actually adopted. **The quote is the audit trail for what
  the SOURCE said; the practice text is what WE adopted, and there is no field that separates them.**
  Recorded in `effect_note` in prose for now. Candidate addition if this recurs: `practice_as_stated`
  alongside `practice`, or a `derived_from` pointer for a practice that descends from a source
  rather than being stated by it. First occurrence - not acted on.
- 2026-08-27 - **earnings-kb violated its own no-bare-dollar-digit rule in four places, cleaned.**
  The rule was written on 2026-08-27 after the argument-substitution corruption, and the paragraph
  stating it notes that it must obey itself - but the four pre-existing instances elsewhere in the
  file were never fixed. **Exactly the coverage failure class the manifest exists to catch: the rule
  existed, was correct, and was never applied to the rest of the document.** Rewritten with the unit
  trailing. Both SKILL.md files now contain zero bare dollar-sign-digit sequences, and both
  frontmatter blocks are byte-identical to the synced originals so triggering behaviour is unchanged.
- 2026-08-27 - **The bridge to chmo-09 dropped mid-session and `_index` could not be committed.**
  The manifest correctly returns INCOMPLETE with `index-committed` FAIL - which is the first time a
  gate in this system has refused to call a run finished, on its first live use, for a real reason.
  Working copies are complete and delivered to Jim; the write-back is outstanding.

## 2026-08-28 - skills saved and verified; a standing rule falsified

- 2026-08-28 - **Both skills saved to the account and verified BY BODY AND BY BEHAVIOUR.** signal-kb
  43,942 bytes md5 `9145211f478be0b5fe33b434370a0892`; earnings-kb 30,021 bytes md5
  `a52248e1af4464d6532c8aa3459b3cef`; both byte-identical to the delivered files, both containing
  `NOT-RUN` six times against zero in the superseded versions. Account manifest lists 14 skills with
  **no duplicated name** - the two-divergent-copies failure of the plugin route did not recur.
  **Then the behavioural check, which is the one that matters:** the python was extracted from the
  ACCOUNT copies and run with every declaration left at `None`. signal-kb Mode 4 returned INCOMPLETE
  7/10 with three NOT-RUN; earnings-kb Mode 3 returned INCOMPLETE 7/9 with two NOT-RUN. The old
  gates would have printed "all gates pass" on identical input. **A body can match on disk and still
  not be the code that runs - verify the behaviour, not only the bytes.** This extends the
  2026-08-27 skill-save rule rather than replacing it.
- 2026-08-28 - **RULE FALSIFIED: "the cache does not refresh mid-session; START A NEW SESSION before
  the next ingest."** Written 2026-08-25 after the plugin episode. The synced skills directory
  refreshed TWICE inside a single long-running session today, and the save was read back from the
  same thread that delivered it - old versions at 08:05, new at 13:05, both observed live. The
  original observation was true of the stale PLUGIN copy at `plugins/synced`, which really did
  persist; it was over-generalised to account-synced skills, where it does not hold. **Cost of the
  error: a mandatory session restart before every ingest since 08-25, for nothing.** Superseded.
  What survives is the narrower and still-correct rule: after any skill save, diff the synced copy
  against the delivered file - now strengthened to include running its code.
  **General lesson, and the second time this exact shape has appeared this week** (the first was
  reading YoY growth rates instead of the sequential increment): an observation made under one set
  of conditions was written as a universal rule without testing whether the conditions were load-
  bearing. Before a rule goes into SKILL.md, state what would falsify it.
- 2026-08-28 - **My error, corrected in the moment: the skill files were delivered under the wrong
  filenames.** They were sent as `signal-kb-SKILL.md` and `earnings-kb-SKILL.md` so Jim could tell
  them apart in one message; the save affordance keys off the filename being exactly `SKILL.md`, so
  no save button appeared and Jim - who has always saved skills by button - had no way forward.
  Fixed by sending each as a file named `SKILL.md` in its own message, captioned with which skill it
  is. **Rule: deliver a skill as a file named exactly `SKILL.md`, one per message, and put the
  identifying name in the caption, never in the filename.** First occurrence.
- 2026-08-28 - **prac-023's effect test passed on first live use, and prac-014's remains OPEN.** The
  manifest surfaced a real finding immediately - `baseline-present` reports 2 of 10 covered tickers
  hold a `filings.csv` row (missing AMZN, AVGO, GOOGL, META, MSFT, MU, ORCL, TSM). MU is the most
  load-bearing of those given that NVIDIA has just named memory pricing as the cause of a 350bp
  gross-margin trough. The source-independence pass cannot be judged yet: it needs three
  figure-bearing ingests and none has run under the new gate. Do not record it as working.

## 2026-08-28 - IREN FY26Q4 ingested; first NeoCloud, and the schema strains

- 2026-08-28 - **IREN added to earnings-kb coverage on Jim's direct request. It carries ZERO claims
  in signal-kb** - not two, like MRVL, but none. The claim-count proxy did not trigger it and the
  direct request supersedes, per the rule written after MRVL. **The justification that actually
  holds is structural, not the request:** IREN is the FIRST NeoCloud in the KB, and NeoClouds are
  the entity class NVIDIA guided to lead FY27Q3 growth, the class the six-capital-provider financing
  platform exists to fund, and the class OpenAI and Anthropic buy from without either of them
  filing. It is a filer standing in a spot where the index previously had only third-party
  commentary. **`earnings-kb` SKILL.md still lists ten tickers and IREN is not among them** - the
  coverage list and the data have diverged and the file needs updating.
- 2026-08-28 - **SCHEMA LIMIT, stated rather than papered over: `filings.csv` is built for a product
  company and IREN is not one.** Its load-bearing figures are contracted ARR, megawatts of IT load,
  revenue per megawatt, CapEx and cost of capital - and NOT ONE has a column. What the schema does
  hold, it holds badly: `gaap_eps`, `nongaap_eps`, `gross_margin_gaap` and `gross_margin_nongaap`
  are all EMPTY for both IREN rows because the company does not report them in that form, so the row
  looks sparse when the filing was in fact dense. Everything of substance went into `deltas.jsonl`
  and `guidance.csv`, which are free-form enough to carry it. **This works, and it should be
  recognised as a workaround.** If a second infrastructure name is ever added, the honest fix is a
  `metrics_json` column on `filings.csv` for company-specific headline measures, not more columns.
  First occurrence - not acted on.
- 2026-08-28 - **BUG IN A CHECK I WROTE YESTERDAY: `baseline-present` counts tickers PRESENT, not
  tickers present AND covered.** It reported "3/10 covered tickers baselined" after this ingest, but
  one of those three is IREN, which is not in the COVERED list at all. The numerator is inflated by
  any uncovered ticker. The missing-list it prints is still correct. Fix: intersect `have` with
  `COVERED` before counting. **Noted with some satisfaction that the manifest surfaced a defect in
  the manifest** - which is the first thing prac-023 has caught that was not a process skip.
- 2026-08-28 - **New gate hazard found and closed BEFORE it could bite: the transcript carries a
  third-party auto-summary in its header.** The IREN file opens with an "Earnings calls summary /
  Powered by Yahoo Scout" block before the first Operator line. A quote drawn from it would have
  passed a naive substring gate and been attributed to management - the same class as the WebFetch
  paraphrase found on 2026-08-27, arriving through a different door. The gate for this ingest was
  built to split header from body and to RAISE on any quote found only in the header, and a negative
  control was run first: summary text was correctly refused. **Rule: before gating a saved
  transcript, check whether the file contains third-party summary or advertisement text, and gate
  against the speaker body only.** Note the NVDA and MRVL transcripts also contain "Advertisement"
  markers, which are inert, and an SA header block. First occurrence.
- 2026-08-28 - **The most consequential finding cuts against what the index currently holds.** IREN
  closed 2.8B USD of GPU financing against NON-investment-grade deployments at single-digit fixed
  rates, including 2.4B at 9% led by Blue Owl and PIMCO funding 90% of that GPU CapEx, with no
  investment-grade offtake and no NVIDIA credit support, and states it no longer needs one. The
  index's reading of NVIDIA's take-or-pay-and-backstop structure is that it is what makes NeoCloud
  GPU financing underwritable, and that it is an unrecognised price cut. If GPUs finance themselves
  on third-party credit at 9%, that structure is a competitive convenience rather than a structural
  necessity. Recorded as `complicates` on baker-08, nrb-01 and thompson-06 rather than `contradicts`,
  because IREN cites NVIDIA's own six-provider platform as a reason the market deepened - the two
  sources are not independent.
- 2026-08-28 - **A first ingest of a ticker produces OBSERVATIONS, not deltas, and the rows say so.**
  Thirteen of the fourteen IREN rows carry a real prior (FY26Q3 was loaded as a baseline in the same
  pass, so the sequential comparison is genuine); the fourteenth, `IREN-FY26Q4-13`, records an
  ABSENCE - no depreciation schedule, GPU useful life or residual-value assumption anywhere in the
  release or the call, and no analyst asked. It is labelled as an absence in `prior` and in `why`.
  **This matters because IREN is the purest available test of the chip-depreciation prediction** -
  an entire asset base of GPUs, 25-30B USD of guided CapEx, a stated two-year cash payback, and no
  stated useful life. That prediction cannot be checked against this filing, and recording why is
  worth more than leaving the row out.
- 2026-08-28 - **Mode 4 caught an apparent cash conflict that resolved by arithmetic.** The call
  states 7.6B USD of cash at 30 June of which 1.7B restricted; the release states 5,895.6M. 5895.6 +
  1700 = 7595.6. The release reports unrestricted, the call reports the total. Recorded as
  `IREN-FY26Q4-14` rather than silently reconciled, because the restricted portion is earmarked for
  Microsoft GPU CapEx and is therefore not available to the FY27 funding plan - a distinction the
  headline cash figure hides. Third consecutive ingest in which the independence pass changed
  something; its effect test now has two of the three runs it needs.

## 2026-08-28 - six-source ingest: 121 claims, and the manifest earns its keep twice

- 2026-08-28 - **Ingest: 6 sources, 121 claims, 32 predictions.** Ed Zitron (Diary of a CEO), Ed
  Zitron again (The Compound TCAF 257), The Compound hosts (same episode), Neil Movva (Invest Like
  The Best), Prof G China Decode (Alice Han / James King), Ed Yardeni, Raoul Pal with Andreas Steno.
  88,603 words, 1.37 claims/1k - at the YouTube median of 1.27. Extraction ran in three parallel
  subagents, novelty grading in three MORE subagents against a date-sorted prior digest, per Mode 1
  step 5. **121/121 quotes verbatim on the first pass.** Novelty came out 56 novel / 49 developing /
  11 consensus / 5 recycled - 54% non-novel against a historic ~11%, which is the first ingest where
  the novelty distribution looks like a real corpus rather than an inflated one.
- 2026-08-28 - **THE CATCH OF THE RUN: one man on two podcasts nearly entered the index as two
  independent voices.** Ed Zitron appeared on Diary of a CEO on 08-27 and on The Compound on 08-28.
  The first extraction pass attributed the entire Compound episode to "The Compound (TCAF 257 panel)"
  under the multi-speaker rule, which was correct given no speaker labels - but it meant roughly half
  those claims were Zitron restating himself 24 hours later, and the convergence machinery would have
  counted them as a second speaker agreeing. Caught because the two extractions returned IDENTICAL
  figures ($34.33bn/$24.1bn Microsoft, $20.9bn OpenAI loss, ~$22bn real demand, "run rate" undefined).
  Confirmed by grepping the transcript: "joining us today, Ed Zitron." A dedicated subagent then split
  the episode Z/H/U and deduped against the prior day. Result: 13 Zitron claims (5 graded `recycled`
  citing his own prior-day ids), 10 genuinely separate host claims. **Rule: when two sources dated
  within a few days return the same figures, check for the same speaker BEFORE grading novelty. The
  "weaker than it looks" convergence shape can hide inside what looks like two sources.**
- 2026-08-28 - **A speaker reversed himself in 24 hours and the index now records both.** Zitron's
  08-27 episode is built on "2027 is when it all breaks"; on 08-28 he says "the reason I'm no longer
  giving timelines cuz I was wrong" (`ezc-11`). He also softens Oracle from "will probably die"
  (`ez-13`) to "the only one of them I'm certain would get bailed out" (`ezc-12`), and CONCEDES a
  factual challenge outright when a host corrects him on cloud revenue accelerating (`ezc-13`). Per
  the 2026-08-27 rule, checked first whether the earlier claim was conditional - it was not. Both
  sides recorded, nothing erased; the prior-day prediction stays `pending` with its horizon so the
  calibration record survives the retraction.
- 2026-08-28 - **MY ERROR, caught and corrected mid-run: I hand-transcribed a 57-file directory
  listing to compute the ingest diff.** That is precisely the re-typing hazard the notes warn about,
  and it was wrong - I misread one filename date and, worse, the listing I typed was STALE, so I
  concluded three earnings transcripts were the only new files when the fetcher had in fact just
  added SIX podcast sources. Re-ran `device_list_dir` and diffed mechanically. **Rule: never compute
  a set difference from a hand-copied tool output. Re-issue the listing call.**
- 2026-08-28 - **BUG IN THE MODE 4 NOVELTY-CITATION REGEX - false failure, second gate defect found
  by the gate.** `pat = [a-z]{2,8}\d?-(?:\d{2}|[sn]\d{2})` allows at most ONE digit in the id prefix,
  so it cannot parse `t253-*` or `t256-*` - **25 existing claim_ids in the index are invisible to it.**
  `cmp2-07` was graded `recycled` citing t253-01 and t253-02, both real and both prior, and the gate
  reported "graded recycled with no cited prior id." Verified by running the manifest twice on
  identical data: INCOMPLETE 9/10 with the saved regex, COMPLETE 10/10 with `[a-z]{1,8}\d{0,4}`.
  **The data was right and the gate was wrong.** Not fixed in the skill - that needs Jim to save a
  new SKILL.md. Until then this check will false-fail on any citation of a t253/t256 claim.
  This is the second time a check has flagged a defect in itself (after `baseline-present` yesterday),
  which is worth noting as an argument FOR the manifest rather than against it.
- 2026-08-28 - **`speaker_interest` cannot vary per claim, and the extraction prompt asked it to.**
  I told a subagent to judge Raoul Pal per claim - `interested` on crypto, `independent` on macro -
  which produced 3 independent and 1 interested and tripped the drift assertion. The schema allows
  exactly ONE value per speaker across both files and Mode 4 enforces it. Set to `independent`
  uniformly: Pal runs a media and research business, not a fund holding the asset, so he does not
  meet the insider-or-principal test that made Pompliano `interested`. **His large public crypto
  exposure is a real bias the field explicitly does not capture** - stated in the report rather than
  encoded, per the schema's own limit. Do not write per-claim interest into a per-speaker field.
- 2026-08-28 - **Speaker identity resolved by an independent rendering, not guessed.** The Invest
  Like The Best guest was captioned only as "Neil" with his company mangled to "Sal research". The
  podcast feed identifies him as **Neil Movva**, ex-NVIDIA. Recorded with the surname because a
  second rendering supplied it - which is the Mode 8 pass doing identity work rather than figure
  work. The COMPANY name remains unresolved and is left out of `entities` rather than guessed.
  **Also note a title discrepancy worth remembering: the podcast feed titles this episode "Making AI
  10x Cheaper"; the YouTube title Jim's fetcher captured says "1000x Cheaper".** Two renderings of
  one episode, a factor of 100 apart, and the figure appears in neither as a clean company claim.
  Third occurrence of the first-name-only identity problem - `identity_confidence` on sources.csv is
  now justified if it recurs once more.
- 2026-08-28 - **Get-Transcripts filename truncation, SECOND AND THIRD OCCURRENCE.** Two of the six
  files lost their dates to the 120-character `Clean-Name` truncation: the Raoul Pal file ends
  "Raoul Pal The Journey .txt" with no date at all, and the Zitron file ends "- 2026-08-.txt" with a
  half-written date. The 2026-08-25 note proposed the fix - truncate the title, THEN append the date -
  and it was never applied. Dates were recovered from the file headers, so nothing was lost, but the
  filename is what `ingested.txt` dedupes on. **The fix is one line in `_tools\Get-Transcripts.ps1`
  and has now been deferred twice.**
- 2026-08-28 - **New coverage the index had none of: China's physical-AI buildout.** 15 of 16 Prof G
  claims graded novel - Chinese humanoid shipments at 97% of H1 2026 global, state-backed
  teleoperation centres buying robots to resell motion data, Unitree at a 50bn USD listing on 252m
  USD of revenue that is 75% research-and-education. There is no second speaker on ANY of it, so it
  cannot yet be weighed - it is a new lane, not a confirmed one.
- 2026-08-29 - **Ingest: 3 new files, 1 producing claims.** Warren Pies (Monetary Matters,
  2026-08-25) -> 20 claims, density 1.69/1k words against a youtube median of 1.27. Two files
  `lanes=skipped`: Ben Thompson 2026.35 is the Friday This Week in Stratechery roundup (562 words,
  pointers only - the 2026-08-26 note predicted "expect one of these every Friday" and this is the
  third), and **"AI Copium - Silicon Valley Knows Something We Don't"** is a commentary channel
  narrating clips of Gavin Baker's Invest Like The Best episode. Every substantive extraction from
  it maps 1:1 onto already-indexed claims (baker-01, -06, -09, -11, -12, -13, -14). **Second
  double-count of this exact shape in two ingests** - Zitron across two podcasts on 2026-08-27,
  Baker via a narrator here. Both were caught by figure-identity, not by the filename. **The
  pattern to watch is not a repeated speaker, it is a repeated FIGURE arriving under a new
  speaker's name.** A commentary channel is the most dangerous form because the name in the
  filename is genuinely new.
- 2026-08-29 - **`novelty-citations` regex confirmed broken on live data, SECOND OCCURRENCE, and
  the fix is still not in SKILL.md.** The shipped pattern `[a-z]{2,8}\d?-` cannot match a source
  prefix of one letter followed by three digits, so `t256-15` and `t256-08` are invisible to it.
  Run twice on identical data: with the shipped pattern, `FAIL - wp-16 graded corroborated with no
  cited prior id`, VERDICT INCOMPLETE 9/10. With `[a-z]{1,8}\d{0,4}-`, PASS, VERDICT COMPLETE 10/10.
  **The data was right both times; the gate was wrong.** wp-18 escaped only by accident - it happens
  to also cite `nrb-03`, which the old pattern does match. **The manifest run reported for this
  ingest used the PATCHED pattern, applied in the run script and NOT in the skill.** That is a
  divergence between the skill as saved and the skill as executed, and it is exactly the kind of
  drift the manifest exists to prevent. Third occurrence of a check flagging a defect in itself
  (after `baseline-present` and the first `novelty-citations` catch). Needs a SKILL.md save.
- 2026-08-29 - **Mode 8 independence pass, third consecutive figure-changing run: 3 conflicts of 8
  figures checked.** (1) **wp-05 Fed odds** - Pies's ~75% July peak does not appear in any public
  rendering (CME FedWatch reads 67% on 07-31, 44.4% on 08-07), and his "back to ~42%" describes a
  move independent sources date to 2026-08-27/28 on the Warsh news, two days AFTER the recording.
  Unresolved, both figures left standing; the directional claim that the market underpriced a
  September hike is if anything strengthened, since the public series moved his way two days later.
  (2) **wp-12 lab ARR** - the $115bn he presents as 3Fourteen's own estimate is independently
  published on 08-27 as the combined ARR of OpenAI and Anthropic ALONE, which inverts the direction
  of his $10bn "miss"; the $105bn "leak" has no second rendering at all. (3) **wp-16 index weights**
  - GICS Information Technology is 37.1% of the S&P 500, not "around half" (reaching 50% requires
  folding in GOOGL/AMZN/META from other sectors), and semiconductors were already 19.7% at end-June
  versus his stale ~18%. **The two errors run in OPPOSITE directions relative to his conclusion:**
  the tech figure flatters his concentration argument, the semi figure understates it. Recorded on
  the claims in `verified_against` / `verification_note`, the fields cmp-10 established.
  **The retirement test is now settled in favour of keeping the pass** - three consecutive
  figure-bearing ingests, three in which it changed the record, and this one is the first where it
  broke figures the primary pass had accepted without any transcript ambiguity to flag them.
- 2026-08-29 - **What the pass could NOT check, stated so it is not mistaken for verified.** wp-06
  (3Fourteen's 0.28% core PCE nowcast) and wp-09 (seven-cycle 20bp/12bp pass-through betas) are
  proprietary house numbers with no second rendering; wp-09's internal arithmetic is sound
  (20 - 12 = 8bp bear flattening, and the label is used correctly) but the coefficients stay
  single-sourced. wp-01 (SPX ~7,800) and wp-05's ~30% trough corroborate cleanly. **`no-second-source`
  is not a weaker `agrees`.**
- 2026-08-29 - **Pies vs the existing Fed disagreement, now four-way.** Timmer (jts-09, jts-10,
  jtf-06): hike, and a hike relieves the long end. Visser (visser-n09, vd-06): no hikes at all.
  Yardeni (ey-02, ey-03, 2026-08-28): neutral is 25-75bp higher, hike to bring long yields down.
  Pies (wp-05, wp-08, wp-09): the hike is a coin flip AND hikes do not lower the 10-year - he
  agrees with Timmer and Yardeni on the action while rejecting the transmission mechanism both
  use to justify it. **Yardeni is dated after Pies and so could not affect a novelty grade**, but
  wp-08 is the direct contradiction of ey-03 and the two should be scored together when the
  September meeting resolves. **This is now the densest disagreement in the index and the one
  worth a Mode 2 query when the FOMC decides.**

## 2026-09-02 - Ingest: 6 sources, 81 claims, 19 predictions

- 2026-09-02 - **`device_bash` still cannot mount this Transcripts folder.** Third+ recurrence
  (see 2026-08-27 entries). All file movement went through `device_stage_files`/`device_commit_files`
  against the container's own `Bash`/`Read`/`Grep`. Treating this as a stable characteristic of
  this session type rather than a transient fault - no further troubleshooting attempted.
- 2026-09-02 - **Missing-header source resolved by web research, not guessed.** "unknown -
  unknown.txt" had blank TITLE/SPEAKER/DATE (only a YouTube URL). Read the transcript's own
  opening lines (host self-identifies as "Raj Mahajan", guest as "Ken Griffin... founder and CEO
  of Citadel"), then `WebSearch` + `WebFetch` on Goldman Sachs' own episode page confirmed title,
  host, guest, and both a recording date (2026-06-02) and publish date (2026-07-09). Used publish
  date as the canonical `date` field; left the file un-renamed on Drive, recording resolved
  metadata only in `sources.csv`. Same resolution pattern as the 2026-08-28 "Neil"/Neil Movva
  case - an independent rendering does the identity work Mode 8 exists for.
- 2026-09-02 - **Four garbled recurring-panelist names resolved by web research.** A Moonshots-
  with-Diamandis panel auto-caption mangled "Alex Weer Gross" / "Dave Blondon" / "Sel"/"See"/
  "Salem" - `WebSearch` against a Spotify episode listing confirmed the three recurring panelists
  are Alexander Wissner-Gross, David Blundin, and Salim Ismail. Corrected silently in `speaker`
  (standard practice for known recurring show participants, per the 2026-08-28 Movva case);
  `garbled: true` reserved for claims where a specific FACTUAL term was mangled (e.g. OpenAI's
  chip name "Jalapeno" transcribed as "Halleluino"), not for panelist identity.
- 2026-09-02 - **`speaker_interest` global-per-speaker drift, THIRD AND FOURTH occurrence -
  pattern now confirmed, proposing a SKILL.md addition.** Same shape as the 2026-08-28 Raoul Pal
  case: David Blundin (VC investor) discloses SpaceX as "my largest financial bet" on one claim
  but gives general AI-industry commentary (CUDA moat, training-vs-inference economics) on four
  others; Salim Ismail gives independent commentary on one claim but relays a third-party
  Principal Financial Group survey on another. Both tripped the Mode 4 drift assertion when
  drafted with topic-varying values. **Fix applied, not novel:** Blundin set to `independent`
  uniformly (his SpaceX position is real and stated in the claim text, but per the schema's own
  limit a large position alone does not confer insider/principal standing - same reasoning as
  the Pal case). Ismail's relayed claim (si-02) was instead re-pointed: `speaker` changed from
  "Salim Ismail" to "Principal Financial Group" (the actual originator, per the `relayed`
  definition's own rule that the speaker field carries the originator), `speaker_interest` set to
  `independent`, avoiding the drift entirely rather than forcing a compromise value. **Proposed
  SKILL.md change** (third occurrence of this exact pattern - Pal 2026-08-28, Blundin and Ismail
  today - meets the two-strikes-plus bar in Mode 5): add one line under the `speaker_interest`
  rules stating explicitly that a speaker takes ONE value across every claim regardless of topic,
  chosen for their most defensible role across the batch, with any narrower/topic-specific stake
  left in the claim text - not decided per claim. Cost: one line in SKILL.md; saves a rework loop
  on every batch with a panel guest who both opines and relays. Held for Jim's decision per Mode 5
  (not applied to the skill by this session).
- 2026-09-02 - **`novelty-citations` regex false-positive, new failure mode (distinct from the
  2026-08-28 prefix-length bug, which remains fixed).** The current pattern correctly matches any
  id-shaped substring in `novelty_note` prose, but cannot tell a genuine citation from a mention
  made explicitly to explain why an id is NOT being cited - e.g. "baker2-01 restates this" (self-
  reference to the new claim's own id) or "Patel's claims are dated 2026-07-29, AFTER this claim,
  so they cannot serve as a valid prior citation" (a same-batch subagent's own chronology-caution
  language). Both read as citations to the gate and FAIL as "not prior" or "self not prior."
  Hit 11 times this run across two independent novelty-grading subagents drafting cautious, well-
  reasoned prose. **Not a subagent quality problem - both were doing exactly what was asked**
  (naming the closest near-miss and explaining why it doesn't qualify). Fixed by rewriting the 11
  `novelty_note` fields to describe near-misses without emitting bare id-shaped tokens (e.g. "a
  same-day claim from Jurrien Timmer" instead of "jtp-05"). **Candidate SKILL.md/script fix, not
  yet at the three-occurrence bar:** either instruct novelty-grading subagents to never print a
  non-cited id in bare `prefix-NN` form, or refine the regex/check to only flag ids inside the
  `cited_claim_ids`-equivalent role rather than any prose mention. First occurrence at this
  volume - noting the pattern rather than proposing a change yet.
- 2026-09-02 - **In-session novelty grading really is a regression - measured, not asserted.**
  Two general-purpose subagents graded all 81 new claims independently, each given the full
  715-claim chronological digest plus its own batch (novelty/novelty_note stripped from the
  input so grading was blind to my drafts). Mechanically verified after the fact: every cited
  prior `claim_id` exists and predates its citing claim, zero exceptions. **34 of 81 grades
  (42%) differed from my own draft grades** - mostly `developing`/`recycled` downgrades from a
  drafted `novel`, once the subagents found the real prior claim my own read of the corpus
  missed. This is the strongest evidence yet for the skill's standing rule that grading inside
  the ingesting session is disallowed; recording the number since prior notes on this rule were
  qualitative.
- 2026-09-02 - **Mode 8 source-independence pass: 5 conflicts of 61 figure-bearing claims
  checked** (two parallel subagents, WebSearch/WebFetch, by source group). All 5 left standing
  per policy (never silently resolved), recorded in `verified_against`/`verification_note` on
  the claim record itself (confirming that field pair is for general Mode-8 source-independence
  findings, not only earnings-kb filing cross-references - matches the wp-05/wp-12/cmp-10
  pattern from 2026-08-29, not a new use). Findings: (1) **wg-07** Gaybrick's ~150-country
  stablecoin/~60-country fiat figures vs. Stripe's own May-2025 101-country disclosure and a
  ~46-country independent fiat count. (2) **baker2-04** Baker's 9-10 month Nebius/CoreWeave
  payback vs. an independent Moomoo analysis putting it near 22 months. (3) **tl-03** Lee's
  "ETH up 54% since June 30" vs. independent daily price data showing ~61%. (4) **tl-09** Lee's
  ~$140-150T addressable tokenization market vs. Bitwise's own $257T combined stock+bond figure.
  (5) **kg-09** Griffin's "seven-year" solar carbon payback vs. Argonne/NREL literature citing
  1-4 years. No pattern across the five beyond "single-source spoken figures underperform a
  quick independent check" - worth another data point before treating that as a rule.
- 2026-09-02 - **Three earnings-call transcripts (IREN, MRVL, NVDA Q2 FY27) present in the
  Transcripts root are owned by the sibling `earnings-kb` skill, not signal-kb - confirmed
  against the 2026-08-27 task-type-split note.** Not ingested for claims. **Gap identified and
  closed:** `earnings-kb` tracks its own coverage in `_earnings/` and does NOT write to this
  skill's `_index/ingested.txt`, so these three files would have kept reappearing as "new" on
  every future `signal-kb` Mode 1 run, forcing a re-triage each time. Added `lanes: skipped`
  rows to `sources.csv` and filenames to `ingested.txt` for all three, matching the existing
  skipped-source convention (Thompson roundups, the AI Copium narrator video) - now a dedupe
  entry exists here even though the actual claims work stays entirely in earnings-kb. If this
  recurs (a new earnings file appears before earnings-kb has run on it), the same close-the-loop
  step should repeat rather than re-litigating scope each time.
