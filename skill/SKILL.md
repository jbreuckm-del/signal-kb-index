---
name: signal-kb
description: >
  Jim's podcast, video and newsletter knowledge base - ingesting transcripts into a structured
  claim index and querying it for cross-source synthesis. Use whenever Jim says "ingest new
  transcripts", "what's new in the KB", "who's converging on X", "where do they disagree",
  "which predictions came due", "score the predictions", "is this signal or noise", "what should
  change my investment thinking", or asks anything spanning multiple episodes, videos or
  newsletters. Also when he pastes a transcript or references the Transcripts folder. A practice
  lane covers sources teaching a method - "review the practices", "what should we adopt", "did
  that change help". A live board checks dated claims against current prices - "run the live
  board", "claim vs tape", "what's actionable", "anything worth a punt".
---

# Signal KB

A two-layer system. Raw transcripts are the archive and the citation source. The **claim
index** is what you actually reason over. Never try to answer a cross-source question by
reading raw transcripts — read the index.

**One intake, two lanes.** Every source enters the same way — same folder, same fetchers,
same `ingest new transcripts` command, same dedupe against `ingested.txt`. What differs is
only what gets emitted at extraction. The `lanes` column on `sources.csv` decides:

- `claims` — the default. A source making assertions about the world: claims, predictions.
- `practice` — a source teaching a *method*: how to build skills, structure workflows, use
  tools. Emits practice rows instead of, or in addition to, claims.

A source can be `claims;practice` and feed both. Never create a second intake process, a
second folder, or a second skill for the practice lane — the front half is identical and
duplicating it would double the manual work for no gain.

## Layout

Two homes since 2026-10-25. Neither depends on any one computer being on.

```
GitHub  jbreuckm-del/signal-kb-index   (private; the INDEX OF RECORD)
  sources.csv               one row per ingested item; `lanes` routes extraction
  claims.jsonl              one JSON object per extracted claim
  predictions.csv           falsifiable predictions with lifecycle status (ledger master)
  practices.jsonl           actionable methods, with adoption status and effect tests
  ingested.txt              filenames already processed (skip these)
  schema-notes.md           Mode 5 log
  LEDGER_README.md, scoring-*.csv, README.md

Google Drive  My Drive/AI/Transcripts/   (folder id 1YQz5XHK9nskO0-ykZ3AZudu-S8Crhdyc)
  <source files>.txt          raw transcripts and newsletters (the archive)
  _inbox/urls.txt             YouTube URLs queued for Get-Transcripts
  _inbox/feeds.txt            RSS/Atom feeds for Get-Written (contains credentials)
  _tools/                     Get-Transcripts.ps1, Get-Written.ps1 + .bat launchers
  _index/                     (folder id 1Oy7_cSQcpQ3P3fWieRZcdXh4DaMRygNl)
    LEDGER_README.md          how other readers (ChatGPT) locate the published ledger
    predictions_YYYY-MM-DD.csv  dated published copies of the ledger - read-only snapshots
    seen-items.txt            Get-Written dedupe state (capture-owned)
    everything else           FROZEN 2026-10-25 - pre-migration copies, never edit
```

**Reaching them.**
- Index: clone the repo into the working directory (attach it with `add_repo`, access `push`,
  if the session lacks it). Work on `main`; commit and push.
- Raw transcripts: read with the Google Drive connector. Use `download_file_content` for the
  exact bytes Mode 4 checks quotes against; `read_file_content` is a rendering and may alter
  whitespace or escape characters. The device bridge (`device_list_dir` etc.) also works when
  chmo-09 is linked, but nothing requires it.

**The repo is the index of record; the container is scratch.** Any pass that touches the
index - including a query-mode pass that only enriches a `resolution_note` - commits and pushes
before it reports. Never force-push or amend a pushed commit; a correction is a new commit.

**Publishing the ledger.** Whenever a pass changes `predictions.csv`, also create a new Drive
file `predictions_YYYY-MM-DD.csv` (today's date; add `_r2` etc. for a second publish the same
day) in the `_index` folder with `create_file`, `contentMimeType: text/csv` and
`disableConversionToGoogleType: true`, from the committed bytes. Never edit or trash an earlier
dated copy. ChatGPT's scanner reads the ledger only by the `LEDGER_README.md` rule, so this is
what keeps its side working unchanged.

**Read `schema-notes.md` (repo) before doing anything else in a fresh session.** It is the
continuity layer, and it is what makes a new chat thread as capable as a long-running one. It
carries every failure mode found so far — the four separate ANSI-decode mojibake incidents, the
normalizer ordering bug, the multi-speaker attribution limit, the plugin route that could not
work, the arg-substitution hazard in skill files — plus every correction made to a claim and
why. **A conversation is not the memory; these files are.** If a fresh session cannot pick up
the work after reading this file, that is a defect in the file, not an argument for keeping one
thread alive forever. Fix the file.

---

## Mode 1 — Ingest

Trigger: "ingest new transcripts", "process the folder", or a new file appears.

1. List the Drive Transcripts folder with `search_files` (`parentId` = the folder id above).
   Compare titles against `ingested.txt` in the repo. Process only new files.
2. Download the new files' exact bytes into the working directory. Read each **in full**. Do not skim, do not sample. **A rendering web
   fetch is never a source file** — see "Capture" below.
3. **Assign the lane** before extracting — see "Lane assignment" below. Write it to the
   `lanes` column on the `sources.csv` row.
4. Emit rows per the schema below: claims and predictions for the `claims` lane, practice
   rows for the `practice` lane, both for a source tagged `claims;practice`.
5. **Grade novelty in separate subagents**, each given a date-sorted digest of the existing
   index plus its own batch, and required to cite a real prior `claim_id` for any non-`novel`
   grade. Then verify mechanically that every cited id exists AND predates the new claim.
   Grading inside the ingesting session is a regression; Mode 4 records it as one rather than
   letting it pass unmentioned.
6. **Run the Mode 8 source-independence pass** on every new record carrying a figure.
7. Append to `sources.csv`, `claims.jsonl`, `predictions.csv`, `practices.jsonl`; append
   filenames to `ingested.txt`. Commit and push everything touched to the repo, and publish
   a dated ledger copy if `predictions.csv` changed (see Layout).
8. **Run the Mode 4 manifest before reporting.** Do not eyeball it, and never report a run
   whose verdict is INCOMPLETE as though it were finished. Every ingest so far has produced at
   least one quote that failed the verbatim check.
9. Report: N sources, M claims, K predictions, P practices added, plus anything that
   contradicts or confirms an existing claim in the index — **with the Mode 4 manifest table
   pasted in**, not summarised.
10. Run the Mode 5 self-improvement pass.

### Capture — where source text may come from

Three routes produce a source file, and only these three:

- `_tools\Get-Transcripts.ps1` — YouTube, via yt-dlp auto-captions.
- `_tools\Get-Written.ps1` — RSS/Atom, prefers `content:encoded`.
- **Claude in Chrome `get_page_text`** — for a one-off URL that is in no feed. Added
  2026-08-27. Needs Jim's browser connected and a question answered, so it is **unavailable to
  a scheduled run**. Write the result to the Drive Transcripts folder with `create_file`
  (`text/plain`, `disableConversionToGoogleType: true`) with the standard header and a
  `SOURCE:` line naming the capture route, then ingest it normally.

**A rendering web fetch is NEVER a source file.** `WebFetch` summarises and rewrites. On
2026-08-27 it returned an openai.com article with the author's "we" and "our" silently
converted to "the organization" throughout — every quote drawn from it would have failed Mode
4, and had the file never been staged the gate would have reported them `unchecked` rather
than failed, which is exactly how a paraphrase enters the index wearing quotation marks. Use a
fetch to decide whether a source is worth ingesting; never as the text quotes are drawn from.

### Lane assignment

Ask what the source is *for*, not what it is about. A source is `practice` when its
substance is a method the workflow could adopt — how to structure a skill, package tooling,
run a process — rather than an assertion about the world that could later be scored.

The diagnostic: extract two or three claims the normal way and check them. **If they come
out `investment_relevance: none` and `falsifiable: false`, the claims schema is measuring
the wrong axis for that source.** That is the signal to add the `practice` lane, not to
lower the extraction bar until the rows look substantive.

Keep the claims too when they corroborate something in the claims lane — a practice source
can still be a witness. Tag `claims;practice` and cross-link with `claim_ids`.

### Extraction rules — these are what make the index worth having

- **Claims must be self-contained.** "He thinks the moat holds" is useless in a table.
  "NVIDIA's CUDA lock-in persists through 2028 because inference workloads inherit training
  toolchains" is a claim. A reader who has not seen the episode must understand it.
- **Attribute to the speaker who made it**, not the host. On an interview, the guest is
  usually `speaker`; when the host asserts something substantive, that is a separate claim
  with the host as speaker.
- **Never invent a claim the speaker did not make.** If the transcript is auto-captioned and
  garbled at a key point, mark `"garbled": true` rather than guessing at meaning.
- **The `quote` field must be verbatim and contiguous** from the transcript. It is the audit
  trail. If you cannot find a verbatim anchor, the claim is your inference, not theirs — drop
  it. **Never elide with `...`** — a stitched quote fails verification and misrepresents what
  was said. If the useful span is interrupted, quote only the contiguous part that survives on
  its own. This is the single most common extraction error; it has occurred on every ingest.
- **Novelty is judged against the existing index**, not against the world. Before assigning
  `novelty`, grep `claims.jsonl` for the same entities and domain. A claim is `recycled` if
  substantially the same claim already exists from that speaker; `consensus` if three or more
  distinct speakers already hold it; `developing` if it extends something in the index;
  `novel` only if it is genuinely absent.
- **Be stingy.** A 90-minute episode yields roughly 10–25 claims, not 60. Filler, pleasantries,
  narrative anecdote with no assertion, and restatements of the question are not claims.
- **A prediction needs a resolution test or it is not a prediction.** If you cannot write an
  observable that would settle it, record it with `"falsifiable": false` and
  `"status": "unresolvable"`. Do not soften vague statements into crisp ones to make the table
  look better — the vagueness is the finding.

### sources.csv

`source_id,title,speaker_primary,host,outlet,date,type,url,word_count,file,ingested_date,lanes`

`source_id`: lowercase slug, `speaker-YYYY-MM-DD-topic` (e.g. `huang-2026-08-19-nvidia-moat`).
`type`: `youtube` | `newsletter` | `earnings` | `research` | `other` — the *format*.
`lanes`: `claims` | `practice` | `claims;practice` — the *use*. Defaults to `claims`.

### claims.jsonl — one JSON object per line

```json
{
  "claim_id": "huang-2026-08-19-04",
  "source_id": "huang-2026-08-19-nvidia-moat",
  "speaker": "Jensen Huang",
  "speaker_interest": "independent|interested|relayed|unknown",
  "date": "2026-08-19",
  "claim": "self-contained sentence stating the assertion",
  "type": "prediction|mechanism|observation|valuation|framework|anecdote",
  "domain": ["AI", "semis"],
  "entities": ["NVDA", "TSMC"],
  "novelty": "novel|developing|consensus|recycled",
  "novelty_note": "what it is novel or recycled relative to, naming claim_ids where relevant",
  "conviction": "high|medium|hedged",
  "evidence_offered": "data|historical-analogy|first-principles|insider-position|assertion",
  "falsifiable": true,
  "quote": "verbatim anchor, under 40 words",
  "investment_relevance": "direct|indirect|none",
  "garbled": false,
  "verified_against": "NVDA-FY27Q2",
  "verification_note": "what the filing says versus what the speaker said"
}
```

`verified_against` and `verification_note` are **optional** - present only on claims that have
actually been checked against a filing, absent otherwise. They hold a `filing_id` from the
companion `earnings-kb` skill. Read with `.get()`; never assume they exist.

**A verification NEVER rewrites the claim.** If a filing contradicts what a speaker said, that
is a finding about the speaker and it is exactly the calibration evidence this index exists to
accumulate. Leave the claim and the quote standing, and record the discrepancy in the note.
Separate three cases explicitly, because they mean different things:

- **speaker error** - they said a wrong number and the transcript captured it correctly.
  `garbled` stays `false`. This is signal about the speaker.
- **transcription damage** - the audio or captions mangled it. Set `garbled: true`. This is
  signal about the pipeline, not the speaker.
- **different figure than assumed** - they were describing something other than what you
  compared against. Usually your error. Fix your comparison, not their claim.

When a number reconciles under none of the three, say so and leave it unexplained. An honest
unresolved note is worth more than a confident wrong reconciliation.

`evidence_offered: assertion` is a flag, not a neutral value. A speaker whose claims are
mostly `assertion` with `conviction: high` is the profile of confident noise.

**`speaker_interest` — what it tracks and what it does not.** It answers one question: does the
speaker hold **insider or principal standing at an entity the claim directly concerns**, such
that they have material influence over the outcome or a duty to promote it?

- `interested` — a CEO forecasting their own company, an operator describing the economics of
  their own business, a manager whose fund is built on the asset they are calling. Jensen Huang
  on NVIDIA. Sam Altman on OpenAI. Pompliano on Bitcoin.
- `relayed` — the claim is not the speaker's own. Someone is reading or reporting another
  party's position. The `speaker` field carries the ORIGINATOR; `relayed` records that the
  wording reaching the index passed through someone else and may be compressed or sharpened.
- `independent` — neither.
- `unknown` — affiliation could not be established from the source. Use this rather than
  guessing; a wrong `independent` is worse than an honest `unknown`.

**The limit, stated so the field is not over-trusted: this does NOT track position-holding.**
A fund manager long NVDA who is not an NVIDIA insider is `independent` under this schema, and
that is a real bias the field does not capture. Position-holding is a separate axis and is
currently untracked. Never describe `independent` as "disinterested" or "unbiased" — it means
"not an insider", nothing more. `evidence_offered: insider-position` remains the finer-grained
flag at the level of the individual claim.

An `interested` claim is not worth less. It is often worth more — insiders know things. It is
worth **differently**, and must never be averaged together with independent claims into a
single conviction or hit-rate number.

### predictions.csv

`prediction_id,source_id,speaker,speaker_interest,date_made,prediction,resolution_test,horizon_date,horizon_basis,conviction,falsifiable,status,resolved_date,resolution_note`

- `speaker_interest`: same values and same meaning as in the claim schema, and it must MATCH
  the value that speaker carries in `claims.jsonl`. Mode 4 asserts this; the two files are not
  allowed to drift.

- `horizon_basis`: `stated` if the speaker gave a date, `inferred` if you assigned one from
  context. Never present an inferred horizon as if the speaker gave it.
- `status` on ingest is always `pending` or `unresolvable`. Only the scoring pass changes it.

### practices.jsonl — one JSON object per line

```json
{
  "practice_id": "prac-001",
  "source_id": "isenberg-2026-08-19-skills-agents",
  "speaker": "Remy",
  "date": "2026-08-19",
  "practice": "imperative sentence: what to actually do, specific enough to act on today",
  "rationale": "the mechanism - why it works, or what failure it prevents",
  "applies_to": ["signal-kb SKILL.md", "_tools\\Get-Written.ps1"],
  "claim_ids": ["remy-03"],
  "cost": "low|medium|high",
  "status": "proposed|adopted|rejected|superseded",
  "adopted_date": "2026-08-25",
  "effect_test": "observable that would show this actually helped",
  "effect_note": "what has happened since, dated",
  "quote": "verbatim anchor, under 40 words"
}
```

The practice lane carries the **same falsifiability discipline as the prediction lane**, and
`effect_test` is where it lives. A practice you adopted and cannot say improved anything is
the workflow equivalent of an unresolvable prediction — and should be reported as one, not
quietly left in `adopted`.

- **A practice is an imperative, not an observation.** "Skills are SOPs for AI" is a claim.
  "Write the skill as an SOP with fixed modes and explicit field schemas" is a practice.
- **`applies_to` must name a real file or artifact.** If nothing in the workflow is named,
  the practice is too abstract to adopt and should be dropped.
- **`status: adopted` requires `adopted_date` and a real change somewhere.** Reading a good
  idea is not adopting it. Partial adoption is recorded as `adopted` with the gap stated
  plainly in `effect_note` — never as a clean success.
- **`rejected` is a valid, useful outcome.** Record why. A rejected practice stops the same
  suggestion being re-litigated on the next source that raises it.

---

## Mode 2 — Query

Load the repo's index files in full — it is small by design. Read raw transcripts only to pull a longer
quote once a claim has already been located through the index.

Answering standards:

- **Recurring ideas**: count by `domain` + `entities` across distinct speakers. Report the
  count and name the speakers. "Several people mentioned X" is not an answer.
- **Novel vs. recycled**: filter on `novelty`, and state what the novel claim is novel
  *relative to* — an unanchored novelty label is meaningless.
- **Convergence**: three or more distinct speakers holding a compatible claim. Say whether
  they converged independently or are visibly citing each other, and flag the latter — one
  idea echoing through four podcasts is not four data points. This distinction is the single
  most useful thing this system does. **Arrival timing is how you measure it.**

  For each cluster compute, from each distinct speaker's FIRST statement of the thesis:
  `age` (days since the earliest speaker stated it), `quiet` (days since anyone last restated
  it), `arrival span` (days between the first and last speaker's first statement), and the
  `arrival gaps` between each successive speaker.

  The gaps separate signal from echo. Report the sequence, not just the span - a cluster reading
  `+249d -> +63d -> +2d -> +0d -> +4d` is a completely different object from one reading
  `+227d -> +31d`, and an average hides it. Four shapes recur:

  - **Independent** - speakers months apart, different vantages, no visible citation. The
    strongest agreement this index can detect.
  - **Delayed crowd** - one voice alone for months, then a cluster inside a fortnight. Name who
    was early; they hold priority. **The arriving consensus is usually already in the price** -
    cross-check against Mode 7 before treating it as an idea.
  - **Event echo** - most arrivals within ~2 weeks of one trigger (a model release, an earnings
    print). Five commentators on one event is one data point, not five. Find the trigger, name it.
  - **Weaker than it looks** - two arrivals share a date because one cites the other. Check for
    this before counting; it has happened (Thompson's 2026-05-05 piece quotes the Huang
    interview, so power/energy has two origins, not three).

  Always check whether a dissent hides inside an apparent consensus: agreement that something is
  HAPPENING often conceals disagreement about whether it MATTERS. Baker holds the open-weight
  thesis and draws the opposite investment conclusion from the rest of that cluster.

  Do not harden these into thresholds. Report the numbers and the shape - the boundaries move
  with how often each speaker publishes, and a weekly writer and a quarterly one cannot be
  judged on the same spacing.

  **Freshness has a floor.** `quiet` only discriminates once themes have had time to go silent.
  In a corpus loaded back-catalogue-first, everything looks freshly restated - say so rather
  than reporting a live-versus-stale split the data cannot support. A cluster that goes quiet
  for a quarter is then worth a hard look: resolved, abandoned, or quietly failed.
- **Disagreement**: find claims with the same `entities` and opposing direction. Present as
  a table: speaker, position, mechanism, evidence type. Then state which side's argument
  survives contact with the other's mechanism, and say so plainly rather than balancing.
- **Predictions**: only from `predictions.csv`. Report the speaker's hit rate as
  `correct / resolved`, and separately report `unresolvable / total` — an unresolvable rate
  above ~40% means that speaker is not making testable claims and their hit rate is noise.
- **Should this change my investment thinking**: the bar is a claim that is (a) `novel` or
  `developing`, (b) `falsifiable: true`, (c) `investment_relevance: direct`, and (d) from a
  speaker with a non-trivial resolved track record. Say explicitly which of the four a
  candidate fails. Most fail (d), and saying so is the point.
- **Noise**: high `conviction` + `evidence_offered: assertion` + `falsifiable: false`, or a
  speaker with a high unresolvable rate. Name names. **Exception: check `lanes` first.** A
  `practice` source scores as noise on every claims-lane metric by construction — its value
  is in `practices.jsonl`, and calling it noise is a category error, not a finding.
- **Workflow questions** ("what should we adopt", "did that change help") are answered from
  `practices.jsonl`, never from `claims.jsonl`. See Mode 6.

Always cite `claim_id` and source file. Never assert something is in the corpus without a
`claim_id` behind it.

**Never print a bare id.** `vx-05` and `pred-035` are internal keys - they mean nothing to a
reader and force Jim to go look them up to follow his own analysis. **Every id that appears in
prose carries the speaker, the date, and the substance of the claim with it:**

> Visser, 6 May 2026 — the binding constraint on AI moved from model capability at the top of
> the stack to physical scarcity at the bottom: power, heat, land, permitting, substations,
> memory (`vx-05`)

The id goes last, in parentheses, as the traceability handle - not first, as the name of the
thing. A reader who does not care about the id should still be able to read the sentence and
understand the evidence. When four or more claims are cited in one answer, break them out into
a table with speaker, date, claim and id as columns rather than threading them through prose.
When a cited claim carries `speaker_interest: interested` or `relayed`, say so at the point of
citation - it changes how the reader should weight it.

---

## Mode 3 — Prediction scoring

Trigger: "score the predictions", "quarterly review", or a scheduled prompt.

1. Load `predictions.csv`; select `status: pending` with `horizon_date <= today`.
2. For each, **web-search the resolution test** before proposing an outcome. Never score from
   memory — that is exactly the recall bias this system exists to defeat.
3. Present a table: prediction, resolution test, what actually happened (with a source link),
   proposed status. Jim decides ambiguous ones; do not resolve them unilaterally.
4. Write back only the statuses he confirms.
5. Close with updated per-speaker scorecards: resolved, correct, wrong, partial,
   unresolvable-rate.
6. **Score `interested` and `independent` speakers in separate tables. Never blend them into
   one hit rate.** A CEO forecasting their own company has both better information and a duty
   to promote, and those pull in opposite directions — the resulting number means something
   different from an outside analyst's and is not comparable to it. Report each cohort's rate
   separately and say which is which. If an `interested` speaker's forecasts on their own
   entity are systematically high or low, that bias is itself the finding, and it is more
   useful than the hit rate.
7. `relayed` predictions carry a further caveat: a miss may be the relayer's compression rather
   than the originator's error. Before scoring one wrong, check the originating document.

---

## Standing cautions

- Auto-captions mangle proper nouns, tickers, and numbers. Any figure pulled from a `youtube`
  source is unverified until checked against a primary source — say so rather than repeating it
  as fact.
- Podcast guests are talking their book. `speaker_interest` marks insider or principal
  standing at the source level and `evidence_offered: insider-position` marks it at the
  individual-claim level; surface both whenever the speaker has a stake in the claim. Neither
  captures position-holding — a manager long the asset they are calling still reads as
  `independent`, and that gap must be stated when it matters rather than papered over.
- **Companion skill.** `earnings-kb` holds reported figures and company guidance for NVDA, MSFT,
  GOOGL, AMZN, META, AVGO, TSM, ORCL, MU and MRVL. When a claim here names one of those tickers AND
  carries a figure, that figure can be checked — see `verified_against` above. The contract is
  pointers only, both ways: neither skill copies the other's data or edits the other's records.
  A figure lives in exactly one file; a number appearing in both is a stale duplicate waiting to
  contradict itself.
- The index is a record of what people said, never a recommendation. Findings inform Jim's
  own decisions; do not translate them into trade suggestions.

---

## Mode 4 — Verification (mandatory after every ingest)

Never report an ingest as complete without running this. It is code, not judgment.

**The manifest is the point.** Every check reports `PASS`, `FAIL` or `NOT-RUN`, and **`NOT-RUN`
is a failure state, not a blank.** Adopted 2026-08-27 (prac-023) on the third occurrence of one
pattern: the check existed, the check was correct, and it never ran — the SKILL.md
description-length error, the `_index` write-back gap, and novelty grading done in-session
instead of by subagents. All three runs were reported clean. **A gate that silently does not
fire is worse than no gate, because it buys confidence it did not earn.**

Three checks cannot be verified by code — whether novelty grading was actually delegated,
whether the Mode 8 independence pass actually ran, and whether the index reached the repo. **They
are DECLARED as literals at the top of the script.** A declaration left at `None` stays
`NOT-RUN` and the run may not be reported complete. Never set one `True` without having done
the thing; that is the one way this check can be defeated, and it defeats the whole manifest.

```python
import json, csv, re, os
IDX, SRC = "<repo checkout>", "<dir of transcripts downloaded from Drive>"

# ---- DECLARATIONS. Set by the operator from what actually happened, never by inference. ----
NEW_SOURCE_IDS      = []    # source_ids this run added or re-extracted under a new lane
NOVELTY_BY_SUBAGENT = None  # True only if grading ran in separate subagents with a prior-digest
INDEPENDENCE_RUN    = None  # Mode 8 verdict: True = ran; False = legitimately N/A (no new
                            # record carries a figure or a second rendering could not exist)
COMMITTED_TO_REPO   = None  # True only after `git push` succeeded AND `git status` is clean AND
                            # `git rev-parse HEAD` equals `git rev-parse origin/main`
LEDGER_PUBLISHED    = None  # True = dated Drive copy created and its size matches the commit;
                            # False = predictions.csv unchanged this run (legitimately N/A)

src = {r['source_id']: r for r in csv.DictReader(open(f"{IDX}/sources.csv", encoding='utf-8'))}
cl  = [json.loads(l) for l in open(f"{IDX}/claims.jsonl", encoding='utf-8')]
pr  = list(csv.DictReader(open(f"{IDX}/predictions.csv", encoding='utf-8')))
pa  = [json.loads(l) for l in open(f"{IDX}/practices.jsonl", encoding='utf-8') if l.strip()]

M = {}
def check(name, fn):
    try:
        M[name] = ("PASS", fn() or "")
    except AssertionError as e:
        M[name] = ("FAIL", str(e))
    except Exception as e:
        M[name] = ("NOT-RUN", f"could not run: {type(e).__name__}: {e}")
def declared(name, val, yes, no, false_ok=False):
    # false_ok distinguishes "legitimately not applicable" from "skipped". A skip is a FAIL:
    # a regression that reports PASS is the exact thing this manifest exists to stop.
    M[name] = (("PASS", yes) if val is True else
               (("PASS" if false_ok else "FAIL"), no) if val is False else
               ("NOT-RUN", "not declared - set the literal at the top of this script"))

def structural():
    assert len({c['claim_id'] for c in cl}) == len(cl), "duplicate claim_id"
    assert len({p['prediction_id'] for p in pr}) == len(pr), "duplicate prediction_id"
    assert len({p['practice_id'] for p in pa}) == len(pa), "duplicate practice_id"
    assert all(c['source_id'] in src for c in cl), "orphan claim"
    assert all(p['source_id'] in src for p in pr), "orphan prediction"
    assert all(p['source_id'] in src for p in pa), "orphan practice"
    assert all(isinstance(c['falsifiable'], bool) for c in cl), "falsifiable must be bool"
    return f"{len(cl)} claims, {len(pr)} predictions, {len(pa)} practices"

def interest():
    # A speaker classified two ways across the two files silently corrupts Mode 3's cohort split.
    V = {'independent','interested','relayed','unknown'}
    assert all(c.get('speaker_interest') in V for c in cl), "bad/missing speaker_interest (claims)"
    assert all(p.get('speaker_interest') in V for p in pr), "bad/missing speaker_interest (predictions)"
    seen = {}
    for r in cl + pr:
        prev = seen.setdefault(r['speaker'], r['speaker_interest'])
        assert prev == r['speaker_interest'], f"speaker_interest drift for {r['speaker']}"
    return f"{len(seen)} speakers consistent across both files"

def lanes():
    ids = {c['claim_id'] for c in cl}
    for p in pa:
        assert 'practice' in src[p['source_id']]['lanes'], f"{p['practice_id']}: source not lane-tagged"
        assert all(cid in ids for cid in p['claim_ids']), f"{p['practice_id']}: dangling claim_id"
        assert p['status'] in ('proposed','adopted','rejected','superseded'), p['practice_id']
        if p['status'] == 'adopted':
            assert p['adopted_date'], f"{p['practice_id']}: adopted, no date"
    for sid, s in src.items():
        if s['lanes'] == 'skipped':
            assert not [c for c in cl if c['source_id'] == sid], f"{sid}: skipped source emitted claims"
    return f"{len(pa)} practice rows, lanes valid"

# Strip punctuation to a SPACE first, THEN collapse whitespace. The other order leaves a double
# space wherever removed punctuation sat between two spaces (an em-dash aside), breaking the
# substring test and reporting a FALSE failure. Found 2026-08-26 against a second rendering.
def norm(s): return re.sub(r'\s+',' ', re.sub(r'[^a-z0-9]',' ', str(s).lower())).strip()
BODIES, RECS = {}, [(c,'claim_id') for c in cl] + [(p,'practice_id') for p in pa]
def body(sid):
    f = src[sid]['file']
    if f not in BODIES:
        p_ = os.path.join(SRC, f)
        BODIES[f] = norm(open(p_, encoding='utf-8').read()) if os.path.exists(p_) else None
    return BODIES[f]

def quotes_new():
    # Records from THIS run. An unstaged new source is a FAILURE, not a note - that is the hole
    # a paraphrased web fetch walks through wearing quotation marks.
    tgt = [(r,k) for r,k in RECS if r['source_id'] in NEW_SOURCE_IDS]
    assert tgt or not NEW_SOURCE_IDS, "NEW_SOURCE_IDS names sources that produced no records"
    miss = sorted({r['source_id'] for r,_ in tgt if body(r['source_id']) is None})
    assert not miss, f"new source not staged, quotes unverifiable: {miss}"
    bad = [r[k] for r,k in tgt if norm(r['quote']) not in body(r['source_id'])]
    assert not bad, f"verbatim failures: {bad}"
    return f"{len(tgt)}/{len(tgt)} new-record quotes verbatim"

def quotes_corpus():
    done = [(r,k) for r,k in RECS if body(r['source_id']) is not None]
    bad  = [r[k] for r,k in done if norm(r['quote']) not in body(r['source_id'])]
    assert not bad, f"verbatim failures in previously-ingested records: {bad}"
    return f"{len(done)}/{len(RECS)} corpus records re-checked ({len(RECS)-len(done)} sources not staged this run)"

def novelty_citations():
    # Every non-novel grade must cite a real prior id that EXISTS and PREDATES the claim.
    byid = {c['claim_id']: c for c in cl}; pid = {p['prediction_id']: p for p in pr}
    # Prefix may carry MULTIPLE digits (t253-01, t256-14) and the number may run to three
    # digits or carry a disambiguating letter (t253-02b). The previous pattern allowed at most
    # one digit in the prefix, so 26 real claim_ids were invisible to it and any citation of a
    # t253/t256 claim FALSE-FAILED as "graded recycled with no cited prior id". Found 2026-08-28
    # by the gate flagging correct data; fixed here after checking all 826 ids in the index -
    # 26 invisible before, 0 after, no regressions.
    pat = re.compile(r'\b([a-z]{1,8}\d{0,4}-(?:\d{2,3}|[sn]\d{2})[a-z]?)\b|\b(pred-\d{3})\b')
    bad = []
    for c in cl:
        if c['source_id'] not in NEW_SOURCE_IDS: continue
        for m in pat.findall(c.get('novelty_note','')):
            cid = m[0] or m[1]
            d = (byid.get(cid) or {}).get('date') or (pid.get(cid) or {}).get('date_made')
            if d is None: bad.append(f"{c['claim_id']} -> {cid} MISSING")
            elif not d.replace('<=','') < c['date']: bad.append(f"{c['claim_id']} -> {cid} not prior ({d})")
        if c['novelty'] != 'novel' and not pat.search(c.get('novelty_note','')):
            bad.append(f"{c['claim_id']} graded {c['novelty']} with no cited prior id")
    assert not bad, "; ".join(bad)
    return "all cited prior ids exist and predate"

def queue_parity():
    n = sum(1 for l in open(f"{IDX}/ingested.txt", encoding='utf-8') if l.strip())
    # ingested.txt may legitimately EXCEED sources.csv by superseded duplicates; never fall short.
    assert n >= len(src), f"ingested.txt {n} < sources.csv {len(src)} - a source was never queued"
    return f"ingested.txt {n} >= sources.csv {len(src)} (excess = superseded duplicates)"

check("structural-integrity",  structural)
check("speaker-interest",      interest)
check("lane-integrity",        lanes)
check("quotes-this-run",       quotes_new)
check("quotes-corpus-resweep", quotes_corpus)
check("novelty-citations",     novelty_citations)
check("queue-parity",          queue_parity)
declared("novelty-by-subagent", NOVELTY_BY_SUBAGENT,
         "graded by subagents against a chronological prior-digest",
         "graded in-session - REGRESSION, name it in the report")
declared("source-independence", INDEPENDENCE_RUN,
         "Mode 8 ran", "N/A - no new record carries a checkable figure", false_ok=True)
declared("index-committed",     COMMITTED_TO_REPO,
         "pushed to the repo, working tree clean, HEAD == origin/main",
         "NOT pushed - the run is not finished")
declared("ledger-published",    LEDGER_PUBLISHED,
         "dated ledger copy published to Drive and size verified",
         "N/A - predictions.csv unchanged this run", false_ok=True)

w = max(len(k) for k in M)
for k,(v,note) in M.items(): print(f"{v:8s} {k:<{w}}  {note}")
fails   = [k for k,(v,_) in M.items() if v == "FAIL"]
notrun  = [k for k,(v,_) in M.items() if v == "NOT-RUN"]
print(f"\nVERDICT: {'COMPLETE' if not fails and not notrun else 'INCOMPLETE'}"
      f"  ({len(M)-len(fails)-len(notrun)}/{len(M)} pass"
      + (f", FAIL: {fails}" if fails else "") + (f", NOT-RUN: {notrun}" if notrun else "") + ")")
```

Fix every verbatim failure by shortening to the contiguous span that does verify, then re-run.
**Paste the manifest table itself into the ingest summary** — not a sentence saying the checks
passed. The table is the artifact; a prose assurance that everything is fine is exactly the
thing the manifest exists to replace.

---

## Mode 5 — Self-improvement

Run at the end of every ingest, query, or scoring session. The point is to make the schema and
these instructions better over time without churning them on every run.

Ask, in order:

1. **Did any step fail or need a workaround** that these instructions do not cover?
2. **Did Jim correct or reject** anything meaningful — a claim, a novelty call, a framing?
3. **Was a question asked that the index could not answer** because a field does not exist?
   This is the highest-value signal: it names a concrete schema gap.
4. **Did a source expose a format problem** — encoding, missing date, missing attribution?

Append anything found to `schema-notes.md` in the repo as a dated one-liner. Do not edit this skill
on a single observation.

**Only propose a change to this SKILL.md when the same observation appears three times** in
`schema-notes.md`, or when Jim corrects the same thing twice. Then state the pattern, the
proposed change, and what it costs — and let Jim decide. A skill that rewrites itself every run
is worse than one that never changes, because the extraction schema stops being comparable
across episodes, which destroys the novelty and convergence judgments that justify the system.

Never silently change the schema. A schema change means claims extracted before and after are
no longer directly comparable; say so when proposing one, and note whether a re-pass over the
existing corpus is needed.

---

## Mode 6 — Practice review

Trigger: "review the practices", "what should we adopt", "what did we adopt and did it work",
"how should we improve the workflow".

Answered from `practices.jsonl`. This is the practice lane's equivalent of Mode 3, and it runs
on the same cadence as the quarterly prediction scoring — the two passes belong together,
because both are asking "was the thing I believed a year ago right."

1. Load `practices.jsonl` in full.
2. **Report adopted practices against their `effect_test` first**, before discussing anything
   new. For each `status: adopted`, state what the test was and what has actually happened.
   An adopted practice whose `effect_test` has never been run is the finding — say so, and
   say what running it would take. Do not let a growing `adopted` count stand in for evidence
   that the workflow improved.
3. Then the `proposed` queue, ordered by `cost` ascending. For each, say what it would change
   in a named file and what the effect test would be. A proposal that cannot be given an
   effect test is not ready to adopt.
4. Recommend at most **two** adoptions per review. The constraint is deliberate: adopting
   several at once makes it impossible to attribute any subsequent improvement to a
   particular change, which destroys the effect tests of all of them.
5. Write back `status`, `adopted_date` and `effect_note` only for what Jim confirms. Commit.

Standing cautions for this lane:

- **A practice source is talking its book too.** Someone whose business is teaching AI
  workflows has a stake in the claim that AI workflows need heavy tooling. Mark
  `evidence_offered: insider-position` on the corresponding claim rows and weigh accordingly.
- **Adoption is not free.** Every adopted practice adds instructions that compete for
  attention on every future run. `cost` is a real field: a `low`-cost practice is one that
  adds a check, a `high`-cost one restructures how the skill is loaded. Prefer the former.
- **Do not adopt a practice that contradicts an effect test already passing.** If the current
  approach is measurably working, a new idea has to beat it, not merely sound better.

---

## Mode 7 - Live board (claim vs tape)

Trigger: "run the live board", "what's actionable", "claim vs tape", "any ideas worth a punt",
"what's working right now".

This is the short-horizon companion to Mode 3. Mode 3 scores predictions when their horizon
arrives; Mode 7 checks dated claims against live prices in between. It exists because a corpus
with no resolved predictions still contains information - namely which theses the tape is
currently agreeing with, and which claims are live at a level right now.

**What this mode is not.** It does not recommend, size, or time anything. It reports what was
claimed, what prices have done since, and what would falsify each thesis. Jim decides. Never
translate a row into a suggested trade, and never present interim tracking as resolution.

1. Load `predictions.csv` and `claims.jsonl`. Select rows that name a **specific instrument and
   a specific level, threshold or direction** - a price, a ratio, a percentage move, a breakout
   level. A directional claim with no instrument is not board material.
2. **Freshness gate.** Drop anything older than ~90 days unless price is still within a few
   percent of the level named. A stale level on a moved market is not an idea.
3. Pull live prices with the Twelve Data tools. Use `get_quote` for the level and 52-week range,
   and `get_time_series` when the claim names a **baseline date** - a resolution test that reads
   "X on date A versus date B" must be scored from the actual closes on those dates, never from
   a rolling-7-day proxy. Getting this wrong produces the opposite verdict; it has happened.
4. Assign one status per row:
   - `tracking` - price has moved toward the claim since it was made
   - `not-tracking` - price has moved against it, or gone nowhere when the claim required motion
   - `live` - price is at or within ~2% of the level the claim names, unresolved right now
   - `not-scoreable` - no instrument data available (see limits below)
5. **Mark what is already in the price.** A claim whose thesis is visible in a 20%+ drawdown has
   largely been discounted. Say so explicitly: being right from here requires the outcome to
   exceed what is priced. This is the difference between a correct call and an available idea,
   and it is the single most useful judgement in this mode.
6. Give every row a **disconfirmer** - the specific observable that would kill the thesis. A row
   without one does not go on the board.
7. Close with an interim scorecard by speaker, and state its window in days. **Label it a
   thermometer, not a track record.** Two or three weeks of tape says who is currently in sync;
   it says nothing about who forecasts well.
8. Write interim findings back into `resolution_note` on the relevant prediction rows, prefixed
   `INTERIM <date>`, and commit. Never change `status` - only Mode 3 does that, and only with
   Jim's confirmation.

**Data limits that shape this mode.** The connected market data covers stocks, ETFs, forex,
crypto and commodities. It does **not** cover bonds or yields, market indices directly, or
options. Use ETF proxies for indices (SPY, QQQ, IWM, SOXX). Rate-driven claims - which include
the sharpest disagreement in the index, Timmer's 4.5% danger zone against Eisman's 5% - cannot
currently be scored. Say so plainly rather than substituting a proxy that does not measure the
claim.

**Standing caution.** A claim can be correct and worthless at the same time, and the board's job
is to tell those apart. Prefer rows where the thesis is NOT yet in the price and the disconfirmer
is close by, because those are the only ones where being early is worth anything.

---

## Mode 8 — Source-independence pass

Adopted 2026-08-27 (`prac-014`, rewritten from Miller's observer-agent framing). Runs inside
Mode 1 step 6, before Mode 4, on every NEW record carrying a figure or drawn from a source that
could have more than one rendering. Declared in the Mode 4 manifest as `INDEPENDENCE_RUN`.

**One job: find a SECOND, independent rendering of the fact and try to break it.** Not
re-reading the same file more carefully — that is what the ingesting session already did, and
it is why the errors this pass exists to catch survived their first review.

**Run it as a separate subagent**, handed the figure and its source and told to look for
disagreement rather than to confirm. Not the session that extracted it: an agent given the same
staged files and asked "is this right" will agree, because it inherits the same anchor.

What counts as an independent rendering, in descending order of strength:

1. **The primary document behind a third-party account** — the filing behind the write-up, the
   published article behind the audio narration, the page behind a fetch.
2. **A different period's document from the same issuer, used as an anchor.** This is the one
   that works when two same-period sources agree with each other and are both wrong. The NVDA
   FY27Q2 sub-segment growth rates were only resolved by going back to the FY27Q1 commentary.
3. **An arithmetic identity the figure must satisfy** — segments summing to a total, a growth
   rate recomputed from levels, a share recomputed from a stated percentage.

Verdicts: `agrees` | `conflicts` | `no-second-source`. **`no-second-source` is legitimate and
common** — a podcast has no second rendering — and it means the figure stays marked
single-sourced, never that it is confirmed.

On `conflicts`, never silently pick a winner. Record which source governs and the arithmetic
that decides it, leave both figures visible, and say plainly when neither can be resolved.

**Retirement test, stated up front.** This pass runs only on figure-bearing new records and
does not touch the rest of the corpus. If it runs on three figure-bearing ingests and surfaces
nothing the primary pass had not already caught, it is overhead — retire it and record that in
`practices.jsonl` rather than letting it accumulate as unexamined ceremony.
