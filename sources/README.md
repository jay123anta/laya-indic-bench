# Data sources and licences

## MASSIVE intent — English and Hindi rows

- Accessed as: `mteb/amazon_massive_intent`, split `test`, accessed 2026-09-27
- Original release: `AmazonScience/massive` (MASSIVE 1.1)
- Licence: **CC BY 4.0** (the original). The mteb re-upload is tagged
  Apache-2.0; where the two differ we follow the original, since a
  re-uploader cannot grant broader terms than the creator.
- Redistribution of derived data: **permitted with attribution**. Our
  Assamese translations and romanisations are derivative works and are
  released under CC BY 4.0 with the notice below.
- Changes made: selected rows 1–200 of the `en` and `hi` test splits;
  machine-translated the English rows into Assamese; transliterated the
  Hindi and Assamese rows into Latin script. Intent labels are unchanged.
- Attribution notice to carry in the data card:

  > Contains material from MASSIVE 1.1 (Amazon Science), used under
  > CC BY 4.0. Modified: subset selected, translated into Assamese,
  > transliterated into Latin script.

- Citation: FitzGerald et al., "MASSIVE: A 1M-Example Multilingual Natural
  Language Understanding Dataset with 51 Typologically-Diverse Languages",
  arXiv:2204.08582. <copy the exact BibTeX from the dataset card>

## Models

| Model | Id | Licence |
|---|---|---|
| Laya (English) | `convaiinnovations/laya` | Apache-2.0 |
| Laya (multilingual) | `convaiinnovations/laya`, subfolder `multilingual` | Apache-2.0 |
| Julia 1 | `SupersonicLabs/Julia-1` | Apache-2.0 (0.1B params, F32, decision-model / multilingual / routing) |

Laya's licence to be confirmed from its repo before release.

## IndicTrans2 — Assamese translation

- Model: `ai4bharat/indictrans2-en-indic-dist-200M` (gated; access granted 2026-09-27)
- Licence: <check the model page>
- Direction: eng_Latn -> asm_Beng, 5 beams
- Pipeline: IndicTransToolkit `IndicProcessor(inference=True)` for pre- and
  post-processing. Postprocessing is required: IndicTrans2 works internally
  in Devanagari and `postprocess_batch` transliterates into Assamese script.
- Environment: transformers pinned <5 (4.57.6), torch 2.10.0+cu128,
  `use_cache=False` (the model's code predates the transformers Cache API)
- Script check: 5,171 Assamese characters, 0 stray Devanagari, 6 danda
- **Machine-translated, not human-translated.** Verification is a separate
  step; see the data card.

## Transliteration

- Tool: TODO
- Licence: TODO
