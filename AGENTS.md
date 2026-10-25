# Repository guidance for Codex and Claude

Keep `AGENTS.md` and `CLAUDE.md` identical when updating either file. Reviewed
2026-10-25 against organizer commit `f8f2629`; recheck upstream rules before submission.

## Purpose and sources

This repository implements **MORE THAN A HACK 2027 Phase 1: Ball Possession**.
Build a reproducible algorithm that labels every tactical-video frame of FC Barcelona
vs Athletic Club, LaLiga 2024-25, matchday 2. Ranking uses `spellF1`, with `macroF1`
as the first tie-break. This is a Python analytics toolkit, with no web application.

Read [README.md](README.md), [CHALLENGE.md](CHALLENGE.md), [DATA.md](DATA.md),
[EVALUATION.md](EVALUATION.md), [SUBMISSION.md](SUBMISSION.md), [RULES.md](RULES.md),
[FAQ.md](FAQ.md), then [OVERVIEW.md](OVERVIEW.md). The metric implementation in
`evaluation/possession_metrics.py` defines scoring behavior. Enrolment terms govern
participation and restricted assets; [LICENSE](LICENSE) covers repository contents
and the public ground truth, not the gated match bundle. Preserve [CITATION.cff](CITATION.cff).

External references:

- [Competition and registration](https://landing.mobileworldcapital.com/more-than-a-hack-2027)
- [Registration terms v2](https://landing.mobileworldcapital.com/hubfs/More%20Than%20A%20Hack%202027%20-%20Terms%20and%20Conditions%20v2.pdf)
- [Organizer repository](https://github.com/metrica-sports/more-than-a-hack-2027)
- Notebook references: [kloppy Metrica](https://kloppy.pysport.org/user-guide/loading-data/metrica/)
  and [Sportscode](https://kloppy.pysport.org/user-guide/loading-data/sportscode/).
  These online docs may describe a newer version than the pinned starter dependency.

## Competition boundaries

- Produce labels algorithmically. Never hand-label or hand-correct this match,
  its tracking, transitions, or outputs, including for training, validation, or
  selecting parameter values. Do not encode observations as frame-specific rules.
- The public five-minute ground truth is **only a pipeline correctness check**.
  Never train, tune, select models, or select hyperparameters against it. Do not
  use private ground truth or human-produced information about this match from any source.
- External labelled data from other matches is permitted when organizers can access
  it and the technical document states its source, labelling process and amount.
  Use only data and models the team has rights to use and organizers can inspect.
- Never publish the gated videos, tracking, Smart Tagging, homography, or exports
  that contain or reconstruct them. Keep restricted assets under ignored `data/`
  or outside the repository. Do not force-add ignored files.
- LLM assistance is permitted, but **do not send restricted materials or any
  extracts to hosted/cloud LLMs**, including assistant context, without the written
  authorization required by terms section 8. Keep restricted analysis local.
- Terms section 8 requires stopping use and deleting provided materials and
  reconstructive derivatives when participation ends. Section 18 continues
  confidentiality for 24 months after competition completion. Coordinate any
  destructive cleanup with the user; do not perform it automatically.
- Participants retain project IP, subject to terms section 16, including Metrica's
  broad commercial-use license to challenge results. Do not assume code ownership
  removes those participation obligations.

### Source conflicts and unresolved details

- The landing page and terms describe Initial Challenge -> top 20 Innovation teams
  -> top 5 finalists and a jury pitch. Treat `OVERVIEW.md`'s broad "no jury/no
  presentation" wording as describing Phase 1, not as cancelling later phases.
- Terms section 5 requires 3-5 members in every phase, at least three with technical
  profiles, and individual registrations using the same team name. Its requirement
  is stricter than the landing page's wording about teams below three not advancing.
- `RULES.md` permits algorithmic pseudo-labels on this match, but an organizer's
  [issue #2 reply](https://github.com/metrica-sports/more-than-a-hack-2027/issues/2#issuecomment-6342808718)
  rejects training a detector using this match's automatic tracking/homography as
  labels. Obtain clarification before relying on training on the provided match.
  The [issue #10 reply](https://github.com/metrica-sports/more-than-a-hack-2027/issues/10#issuecomment-6366363848)
  also prohibits choosing parameters against hand labels of this match.
- Initial submission is due **November 17, 2026**; the cutoff time/timezone is
  unspecified. Registration closes November 10, 2026. Do not invent a midnight cutoff.
- Innovation runs November 26, 2026-January 14, 2027, using 10 additional matches.
  Its deliverables are due January 14 at 11:00 CET; finalist presentations are due
  February 22 at 11:00 CET. Both times are 17:00 in Bangkok. The final is March 2,
  2027 at Fira Montjuic, Barcelona. Use later organizer instructions for these phases.
- The issue #2 reply permits CUDA and says runtime is not a requirement. Exact
  hardware and numerical reproducibility questions remain open in
  [issue #11](https://github.com/metrica-sports/more-than-a-hack-2027/issues/11).
  Preserve the published requirement to regenerate the same submission file.

## Repository map

| Path | Responsibility |
| --- | --- |
| `games.py` | Game/source constants, asset paths, frame conversions and graded half boundaries |
| `starter/baseline.py` | Raw nearest-player baseline and centered smoothing; numpy plus standard library |
| `starter/load_atd.py` | kloppy tracking loader and raw/alignment checks |
| `starter/load_smart_tagging.py` | Standard-library SportsCode XML reader and time-base checks |
| `starter/load_homography.py` | Matrix loading, projection, inverse projection and alignment checks |
| `starter/utils.py` | Matplotlib pitch drawing, frame plots and animation |
| `starter/01_smart_tagging.ipynb` | Guided automatic-event exploration |
| `starter/02_tracking_atd.ipynb` | Guided tracking exploration |
| `starter/03_first_submission.ipynb` | Baseline generation and scoring walkthrough |
| `starter/make_dummy_submission.py` | Constant-label examples for checking the submission loop |
| `evaluation/possession_metrics.py` | Validation, frame metrics, spells, transitions and seeded bootstrap |
| `evaluation/run_evaluation.py` | Score/rank CSVs; write scoreboard and per-submission reports |
| `evaluation/validate_submission.py` | Format checks and label/possession diagnostics |
| `evaluation/gt/` | Public five-minute CSV and metadata JSON; never replace with private truth |
| `evaluation/results/` | Four tracked reference baselines; other result CSVs are ignored |
| `data/README.md` | Gated asset layout; no match assets ship in this clone |
| `requirements*.txt` | Minimal scorer and optional notebook dependencies |

## Submission and alignment contract

- Write UTF-8 CSV with exactly `frame,label`, **199,319 data rows**, integer frames
  **0..199318 in order**, and labels `H`, `A`, `D`, `X`. No dataframe index or blank
  cells. `H` is Barcelona, `A` Athletic, `D` dead ball, `X` abstention.
- A possession is a maximal live `H` or `A` run. Loose/contested balls retain the
  last controlling team until the opponent gains control; touches and deflections
  alone do not transfer possession. Dead-ball ownership is not requested.
- Tactical video is the reference at **25 fps**. Delivered ATD uses
  `video_frame = frame_id - 1`; homography is already zero-based; Smart Tagging
  uses `round(seconds * 25)`. Do not apply `games.py`'s organizer IdFrame offsets
  to the delivered ATD. Reuse its helpers for the counter they actually describe.
- Full-match graded blocks are inclusive **11185..90651** and **110808..190197**
  (158,857 frames). Outside them is ungraded. Public truth covers only
  **11185..18684** (7,500 frames, 17 possessions); `X` elsewhere is not a training label.
  The second graded block begins before kickoff. Playback anchors in `games.py`
  differ by one frame from Smart Tagging kickoff checks; do not silently unify them.
- Broadcast has **164,488 frames**, its own cuts/replays and no supplied alignment
  to tactical. Never index the submission by broadcast frames.
- ATD coordinates are normalized 0-1. Scale axes separately for distances;
  `games.py` uses 105.3 x 68.0 m. Track slots are not player identities. Handle NaNs,
  duplicated/fragmented tracks, identity switches and the missing ball (about 41%
  of graded frames). Check homography `valid` and `reprojection_error`.
- Smart Tagging is automatic evidence, not ground truth or possession intervals.
  Handle title-case columns, inconsistent team spelling (`Atletic Club`) and `N/A`.
- `spellF1` matches same-team possession intervals one-to-one by temporal IoU,
  averaging F1 at 0.3/0.4/0.5/0.6/0.7. Dead-ball segments are not possessions.
  `macroF1` averages frame F1 over H/A/D and penalizes graded abstentions.
- Temporal metrics forward-fill `X` within each contiguous graded run and back-fill
  leading gaps. Entirely abstained runs remain `X`. Never carry state across halftime.
- Baseline windows are **half-widths**: `--window 50` means +/-50 frames (101 total).
  The baseline votes only H/A, uses D when neither occurs, and breaks H/A ties toward H.
  Missing ball becoming D is a baseline heuristic, not the competition's possession rule.

## Setup and verification

Run commands from the repository root. Scoring supports Python 3.8+; use Python
3.9+ for optional notebooks, as advised upstream. Create an isolated environment:

```powershell
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\python.exe evaluation/run_evaluation.py --no-detail
.\.venv\Scripts\python.exe evaluation/validate_submission.py evaluation/results/baseline_smooth50.csv
# Optional notebook dependencies; preserves kloppy==3.18.0:
.\.venv\Scripts\python.exe -m pip install -r requirements-starter.txt
```

Launch notebooks with their kernel working directory set to `starter/` (for example,
`python -m notebook starter` from the root); their imports and relative paths assume
that directory. Notebook 03 demonstrates a public-score smoothing sweep; it does not
override the rule against using those scores for parameter or model selection.

Public scorer expectations (pipeline check, never a tuning target):

| Baseline | macroF1 | spellF1 |
| --- | --- | --- |
| `baseline_smooth125` | 0.5812 | 0.4242 |
| `baseline_smooth50` | 0.6207 | 0.3256 |
| `baseline_smooth25` | 0.6200 | 0.2868 |
| `baseline_raw` | 0.5972 | 0.0378 |

The published **private full-match** baseline floor is `smooth50` spellF1 **0.2433**;
do not confuse that with locally measured public scores. Private scoring cannot be
reproduced from this clone. `--bootstrap 1000` adds seeded confidence intervals;
the short public window does not establish a reliable ranking.

Only after authorized local asset access, unpack the documented tactical/broadcast
folders in `data/` or set `$env:MTAH_DATA_DIR = 'D:\path\to\bundle'` before starting
Python. This is a process environment setting; the repository does not load `.env`.
Run locally: `python starter/load_atd.py`, `python starter/load_smart_tagging.py`,
`python starter/load_homography.py`, and `python starter/baseline.py`. Do not expose
their restricted-data outputs to hosted assistants. Missing assets are expected in
a fresh clone, not an installation failure.

## Change and delivery discipline

- Read neighboring code and keep changes focused. Preserve Python 3.8 compatibility
  for core tooling and the lightweight evaluation dependencies. No test runner,
  linter configuration, package build, or CI workflow currently ships here.
- Add solution logic separately from the official evaluator. Do not modify metrics,
  ground truth, validation, or tracked baselines to improve reported performance.
- Verify relevant behavior with synthetic inputs, then run the public scorer and
  validator. For data-dependent work, state which checks actually ran locally.
  Review the diff before committing and preserve baseline/GT integrity.
- Generated reports and scoreboard are already ignored. Add ignore rules for any
  new temporary/cache/log outputs before creating them. Clear restricted notebook
  outputs before publication; never commit credentials or private machine paths.
- Final Phase 1 deliverables: `<team_name>.csv`, a public repository with pinned
  dependencies and an exact regeneration command, and a short PDF or Markdown
  technical document describing method, assets, training provenance, possession
  rules, limitations and reproduction. Keep the generated CSV out of public Git.
- The submission form is emailed to registered participants, requires a Google
  account for upload, and accepts one editable response per team; the last response
  before the deadline counts. There is no live leaderboard or submission feedback.
  Validate the exact final CSV, record seeds/versions and verify repeated runs
  reproduce the same file. Do not claim a score without naming its ground truth.
- Keep the organizer repository as `upstream` and the team's fork as `origin`.
  Commit/push only when the user authorizes them. Never force-push or discard work
  without explicit permission; if a merge is needed, use `git merge --no-ff`.
