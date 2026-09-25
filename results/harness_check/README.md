# Harness verification

Run on Kaggle, CPU, laya 0.3.20, 100 cases per language,
mteb/amazon_massive_intent test split, seed 13, 20 options.

## Gate: reproduces upstream exactly

`check_en_unclamped.json` — English checkpoint, raw (unclamped) temperatures:

| | this run | upstream's stated value |
|---|---|---|
| accuracy | 0.8200 | 0.8200 |
| macro_f1 | 0.7876 | 0.7876 |
| ece | 0.1789 | 0.1789 |
| mean_confidence | 0.9989 | 0.9989 |

The harness is faithful.

## Baselines (clamped temperatures, i.e. as served)

| | English ckpt | Multilingual ckpt |
|---|---|---|
| en | 0.82, conf 0.958, ECE 0.138 | 0.71, conf 0.888, ECE 0.233 |
| hi | 0.10, conf 0.742, ECE 0.642 | 0.46, conf 0.818, ECE 0.369 |

H1 (replication) holds: upstream reports 0.10 and 0.43 for Hindi;
we get 0.10 and 0.46. At n=100 a 3-point gap is one or two items.

## Notes

- The multilingual checkpoint ships **no temperatures at all**
  (`"temperatures": {}`), so every bucket defaults to 1.0. The
  checkpoint this benchmark depends on is uncalibrated as shipped.
- The English checkpoint's `choice:11+` bucket is 0.1006 raw, clamped
  to 0.5. Every 20-option question here lands in that bucket.

## Files

- `check_en_unclamped.json` — the verification gate
- `base_english_ckpt.json` — English checkpoint, en + hi
- `base_multilingual_ckpt.json` — multilingual checkpoint, en + hi
