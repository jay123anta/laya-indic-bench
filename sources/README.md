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

- Model id and version: TODO
- Licence: TODO

## Transliteration

- Tool: TODO
- Licence: TODO
