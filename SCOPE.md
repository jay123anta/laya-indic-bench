# SCOPE — laya-indic-bench v0.1

Status: draft. Items marked **TODO** must be resolved before data work starts.
Everything else is decided and should not be reopened during v0.1.

Revised after reading `research/eval/laya_eval.py` and
`research/results/cpu_51_language_sweep.json` from the upstream Laya repo.

## 1. Question

Does Laya make correct, honestly-confident typed decisions in Hindi and
Assamese, and does its router send those languages to the right checkpoint?

Two languages, chosen as a deliberate contrast:

| | Hindi (`hi`, Devanagari) | Assamese (`as`, Bengali-Assamese script) |
|---|---|---|
| In Laya's 51-language sweep | yes | no — the sweep lists `bn` but no `as` |
| Published reference | English ckpt 0.10, multilingual ckpt 0.43 | none exists |
| Nearest reference | — | Bengali: English ckpt 0.08, multilingual 0.29 |

Hindi validates the harness against published numbers. Assamese is the new
evidence.

## 2. What is already known (so we don't claim it as a finding)

From `cpu_51_language_sweep.json`: 100 cases per language, 20-option MASSIVE
intent.

| | English ckpt | Multilingual ckpt |
|---|---|---|
| English | acc 0.82, conf 0.999, ECE 0.179 | acc 0.68, conf 0.872, ECE 0.209 |
| Hindi | acc 0.10, conf 0.941, ECE 0.850 | acc 0.43, conf 0.751, ECE 0.321 |
| Bengali | acc 0.08, conf 0.945, ECE 0.865 | acc 0.29, conf 0.694, ECE 0.409 |
| Macro (51 langs) | acc 0.227, ECE 0.733 | acc 0.366, ECE 0.387 |

After the temperature clamp (#42, #208), Hindi on the English checkpoint still
sits at 0.10 accuracy with 0.742 mean confidence and ECE 0.642. Clamping
reduces the dishonesty; it does not remove it.

So "the English checkpoint fails silently on Devanagari" is upstream's own
result, not ours. Our contribution is Assamese, the romanised and translated
conditions, the additional metrics, and per-language calibration.

## 3. Pre-registered hypotheses

Written before any run, so results cannot be rationalised afterwards.

- **H1 (replication).** Hindi reproduces upstream's numbers on both
  checkpoints, within sampling noise. If not, our harness is wrong.
- **H2.** Assamese on the multilingual checkpoint lands near Bengali's 0.29
  rather than near Hindi's 0.43, because Assamese is absent from the reported
  language list while sharing Bengali's script.
- **H3.** The English checkpoint fails silently on Assamese: accuracy near the
  0.05 random baseline with mean confidence above 0.9, matching the Bengali
  pattern.
- **H4.** Romanised Hindi and romanised Assamese are routed to the English
  checkpoint and score worse than their native-script equivalents. Romanised
  Bangla was explicitly fixed upstream in 0.3.8; Hindi and Assamese are
  unstated.
- **H5.** Translate-to-English-first beats running the multilingual checkpoint
  on the original text. Upstream's English multilingual score (0.68) is well
  above every Indic score, so translation may dominate despite its losses.

Any hypothesis may be refuted. That is the point of writing them down.

## 4. Task

**One task in v0.1: MASSIVE intent, 20 options, Choice type.**

Deliberately identical to upstream's setup so results are comparable:

```
dataset       mteb/amazon_massive_intent, split test
sampling      first 100 rows per language, random.Random(13) fresh per language
options       gold label + rng.sample of 19 others, then shuffled — per case
instructions  "What is the user asking for in `utterance`?"
rendering     option keys: "_" -> " " and "." -> ": "
head_max_len  192
max_len       512
```

There is **no fixed list of 20 labels**. Options are drawn per case from that
language's full label set. Comparability comes from using the same script,
seed and parameters, not from copying a label list.

Random baseline: 0.05. Majority-class baseline: computed and reported.

**Noul and Score tasks are deferred to v0.2.** Adding them means inventing
label schemes with no upstream reference, which breaks comparability and
multiplies the verification burden. One task, done properly, first.

## 5. Conditions

All conditions derive from **the same 100 English utterances**, so only the
surface form varies. `label_text` is never altered, which means the harness
produces the same options, same order and same gold index for every
condition. This makes it a controlled experiment rather than seven separate
datasets.

| Id | Description | Source |
|---|---|---|
| `en` | English original | MASSIVE `en`, rows 1–100 |
| `hi_native` | Hindi, Devanagari | MASSIVE `hi` **if row-parallel with `en`** (TODO 1) |
| `as_native` | Assamese, Bengali-Assamese script | translated from `en` |
| `hi_roman` | Hindi transliterated to Latin | from `hi_native` |
| `as_roman` | Assamese transliterated to Latin | from `as_native` |
| `hi_translated` | Hindi translated back to English | from `hi_native` |
| `as_translated` | Assamese translated back to English | from `as_native` |

The `translated` conditions test the practical alternative every developer
considers: translate first, then use the English checkpoint.

## 6. Models under test

| Id | Checkpoint | Why |
|---|---|---|
| `laya-en` | `convaiinnovations/laya` | expected to fail on Indic scripts — measured deliberately |
| `laya-multi` | `convaiinnovations/laya`, `--subfolder multilingual` | the realistic choice |
| `router` | `laya.Router` defaults | tests the routing decision itself, not just the model |

Plus two non-model baselines: **random** (0.05) and **majority class**.

Every result records the exact `laya` version. Behaviour changed materially
across 0.3.6 → 0.3.10 within one week; a number without a version is not a
result.

Both temperature regimes are reported where relevant, via the harness's
`--unclamped` flag, since upstream's published figures predate the clamp.

## 7. Sizes and splits

| Split | Rows per condition | Used for |
|---|---|---|
| `main` | 100 | reported numbers — matches upstream exactly |
| `calib` | 100 (rows 101–200) | fitting calibration temperatures ONLY |

100 rows per condition, because upstream used 100 and comparability matters
more than sample size in v0.1. It also fits what one volunteer can verify in
about 25 minutes, which is what makes the request easy to say yes to.

Calibration rows are exported and verified in the same pass, but never used
for reported accuracy. A confidence interval accompanies every accuracy,
because n=100 is small: a difference under roughly 10 points is not
meaningful.

## 8. Data sources and licences

**TODO 2 — verify before relying on any of these.** Record for each: exact
name, version, languages covered, licence, and whether redistribution of
derived data is permitted.

| Need | Source | To verify |
|---|---|---|
| English and Hindi rows | `mteb/amazon_massive_intent` | licence; whether `hi` rows are row-parallel with `en` |
| Assamese rows | none exists | must be translated from English |
| Translation | IndicTrans2 | licence; record exact model version |
| Transliteration | TODO — pick one | licence; record exact version |

Rule: **a condition with no verified source and no verification plan does not
ship in v0.1.** Cut it rather than publish unverified data.

If a licence forbids redistributing source text, publish only the derived
items plus a script that regenerates them, and say which case applies.

## 9. Data creation and verification

For any text not taken directly from a licensed source:

1. Translate or transliterate, recording tool and version.
2. A native speaker reviews every item and marks it **fine** / **sounds
   unnatural** / **wrong meaning**.
3. Items marked *wrong meaning* are corrected or dropped, never kept.
4. The verification rate per condition goes in the data card. Nothing is
   described as "human-verified" unless a person actually read it.

**TODO 3:** name who verifies Assamese and who verifies Hindi, and how many
items each can review. This constraint — not compute — decides what v0.1
contains. If no Assamese verifier can be found, ship Hindi-only and add
Assamese in v0.2.

**TODO 4 — romanisation method.** Automatic transliteration is consistent and
reproducible but produces one canonical spelling; real users vary. v0.1 uses
automatic transliteration with a human naturalness check, and the data card
states plainly that this is canonical transliteration, not natural typing.
Natural typed variation is future work.

**Privacy:** MASSIVE utterances are synthetic voice-assistant commands, so no
personal data is involved. Any item that nonetheless contains a name or
identifier is replaced before publication.

## 10. Metrics

Per (model × condition):

- accuracy, macro-F1, with a confidence interval
- **mean confidence, and the gap (mean confidence − accuracy)** — the sign
  says whether the model is over- or under-confident, which ECE alone does not
- ECE (15 bins, matching upstream's implementation), Brier, NLL
- **confident-wrong rate**: share of items answered wrongly at confidence
  ≥ 0.9 — the silent-failure measure, and the headline for H3
- **order-flip rate**: re-run with option order reshuffled under a different
  seed, report how often the predicted label changes
- accuracy at 50% coverage (already in the harness)
- p50 / p95 latency
- random and majority baselines beside every number

For the `router` model, additionally record **which checkpoint it chose** per
item, since routing is itself a hypothesis (H4).

Calibration follows the settled recipe: fit on the `calib` split, and clear
any per-bucket temperature overrides inherited from the base checkpoint before
refitting. Evidence: `results/calibration_ablation.md`.

## 11. Harness

`harness/laya_eval.py` is vendored **unmodified** from upstream
(`research/eval/laya_eval.py`), because its docstring states that it
reproduces `en` at accuracy 0.8200 / macro-F1 0.7876 / ECE 0.1789 /
mean confidence 0.9989 exactly under raw temperatures. Reproducing those four
numbers is the harness verification gate: nothing else in this project is
trustworthy until it passes.

Extensions live in `harness/indic_eval.py`, which imports from the vendored
file and adds:

1. `--rows PATH` to load local JSONL rows instead of a dataset config, since
   `mteb/amazon_massive_intent` has no `as` config. Rows need only
   `{"text": ..., "label_text": ...}`.
2. Chunked scoring. The upstream `score_cases` collates every case into one
   forward pass, which is fine for 100 cases but will exhaust GPU memory at
   scale.
3. Confident-wrong rate, Brier, NLL, and the order-flip pass.

## 12. Deliverables for v0.1

1. Dataset (JSONL rows per condition) on Hugging Face, with a data card giving
   provenance and verification rate per condition.
2. `harness/` — the vendored harness plus extensions, runnable with one
   command.
3. `LEADERBOARD.md` — results per model per condition, with baselines.
4. `calibration/` — fitted temperatures for Hindi and Assamese, with ECE
   before and after.
5. Repo licence, and a per-source licence note in `sources/`.

## 13. Row and item format

Row files (`data/rows/<condition>.jsonl`) carry only what the harness needs:

```json
{"text": "...", "label_text": "alarm_set"}
```

Published evaluation items carry the full record:

```json
{"id": "as_native-0001",
 "lang": "as", "script": "beng", "condition": "as_native",
 "state": {"utterance": "..."},
 "questions": {"intent": {"type": "choice",
                          "instructions": "What is the user asking for in `utterance`?",
                          "criteria": {"alarm_set": "alarm: set", "...": "..."}}},
 "gold": {"intent": {"label": "alarm_set"}},
 "source": "translated from MASSIVE en row 1, IndicTrans2 v<x>",
 "verified": "fine",
 "verifier_id": "A1"}
```

`state` / `questions` / `gold` match what the upstream fine-tuning notebook
consumes, so the same files work unchanged if v0.3 fine-tunes.

## 14. Definition of done

- [ ] Harness reproduces `en` at 0.8200 / 0.7876 / 0.1789 / 0.9989 under
      `--unclamped`
- [ ] Hindi reproduces upstream's 0.10 and 0.43 within sampling noise (H1)
- [ ] Every source's licence recorded and permits our use
- [ ] Verification rate reported per condition, with no unverified item
      published as verified
- [ ] Every number carries its `laya` version and temperature regime
- [ ] One command reproduces every table in the repo

## 15. Explicitly not doing

Named so they do not creep in:

- Noul and Score tasks (v0.2)
- more languages (v0.2)
- fine-tuning an Indic checkpoint (v0.3)
- running Jev for comparison — no API access, and published figures are quoted
  as context only, never as a measured head-to-head
- a serving layer, cache, or client library — upstream and the community
  already ship these

## Open TODOs

1. Are `mteb/amazon_massive_intent` `hi` rows row-parallel with `en`? If not,
   Hindi is a separate sample and the controlled-experiment claim weakens.
2. Licences for the dataset, IndicTrans2, and the chosen transliteration tool.
3. Named verifiers for Assamese and Hindi, with realistic item counts.
4. Which transliteration tool and version.
