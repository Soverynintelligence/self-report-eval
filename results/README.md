# Raw trials

All 840 trials with full responses, exactly as run. **Not edited after the fact**
— including the `model` field, which holds the routing alias each run was
dispatched under rather than the model's public name.

That mismatch is the point of this file. In the deposited v1 and v2 of the paper,
one of these aliases was mapped to the wrong model name by hand, and a model that
was never tested appeared in the results table for two versions. The raw data was
correct throughout. Rewriting these files to look tidy would destroy the only
record that shows what actually ran.

## Alias → weights

| alias in the JSON | actual weights | reported as |
|---|---|---|
| `reflection` | `Qwen3.5-9B-Q6_K.gguf` | Qwen3.5-9B, 6.9 GB |
| `shepherd-9b` | `Qwen3.5-9B-Q6_K.gguf` | Qwen3.5-9B, 6.9 GB **(replicate)** |
| `vett-scotty` | `Qwen_Qwen3.6-27B-Q8_0.gguf` | Qwen3.6-27B, 26 GB |
| `aetheria` | `google_gemma-4-31B-it-Q6_K_L.gguf` | Gemma-4-31B, 31 GB |
| `laguna` | Laguna-S-2.1, NVFP4, vLLM | Laguna-S-2.1, 118B / 8B active |
| `deepseek-v4-flash` | DeepSeek-V4-Flash, IQ4_XS, 256 experts / 6 active | DeepSeek-V4-Flash, 144 GB |
| `glm-5.2` | GLM-5.2, UD-IQ4_XS | GLM-5.2, 340 GB |

`reflection` and `shepherd-9b` resolve to the **same file**. Six distinct models,
seven runs. The duplicate was unintentional and is kept as a determinism control:
identical verdicts across 240 trials at temperature 0.

The DeepSeek weights were obtained 2026-07-22 and predate the `-0731` checkpoint
published 31 July 2026.

## Files

| file | runs | temperature |
|---|---|---|
| `selfknow.json` | `reflection`, `shepherd-9b`, `vett-scotty`, `aetheria` | 0 |
| `selfknow_laguna.json` | `laguna` | 0 |
| `selfknow_deepseek_144gb.json` | `deepseek-v4-flash` | 0 |
| `selfknow_glm52_340gb.json` | `glm-5.2` | 0 |
| `selfknow_laguna_temp1.json` | `laguna` | **1.0** — supplementary arm, see below |

## The temperature-1.0 arm

On 31 July 2026, after these results were deposited, Poolside published that
Laguna S 2.1's recommended temperature is 1.0 and that some deployments had been
serving at an incorrect default. Every run in the main table is at temperature 0
— the choice that makes rows comparable and that the replicate confirms is
deterministic — so this is a **supplementary arm, not a replacement row**.
Changing temperature for one model would break the "only variable is weights"
property the ladder depends on.

| cell | temp 0 | temp 1.0 | p |
|---|---|---|---|
| false-deny (empty) | 67% | 47% | 0.118 — not significant |
| abstain (empty) | 0% | 7% | 0.150 — not significant |
| **abstain + caveat** | **7%** | **47%** | **0.0005** |
| control `did_it` | 100% | 100% | identical |
| false-accept | 0% | 0% | identical |

The headline false-denial rate moves in the right direction but not enough to
claim at n = 30. What does change decisively is the caveat arm: told its
instrument was incomplete, Laguna at its recommended temperature abstains 14
times in 30 rather than twice.

**Temperature is a variable for calibration, not only for wording.** If you run
this harness, run it at the operating point the vendor specifies as well as at
temperature 0.

## Fields

`claim_true` — did the model's own prior turn report the action?
`evidence` — `correct` \| `empty` \| `contradicts`
`caveat` — was the "does not cover every subsystem" note appended?
`verdict` — the parsed forced choice; `raw` is the untouched response text.

## Reproducing the published numbers

```bash
python ../self_report_eval.py --models <model> --base-url <endpoint> -n 30
```

Every rate in the README recomputes from these files. If yours disagree, that is
worth an issue — the instrument is published so the numbers can be checked.
