# 01 - Measure: latency baseline

Model `Qwen3.5 0.8B` � host `Windows-AMD64` � llama.cpp `b10488`
Settings: `threads=4` `ngl=99` `ctx=2048`
`max_tokens=64` � warm-up discarded
Completed requests: `Q4_K_M` 10/10 � `UD-Q2_K_XL` 10/10

| Quantization | Size (GB) | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode (tok/s) |
|:--|--:|--:|--:|--:|--:|--:|
| Q4_K_M | 0.50 | 52604 | 1031 / 1524 | 184.8 / 193.9 | 10610 / 13737 / 13737 | 5.4 |
| UD-Q2_K_XL | 0.39 | 3570 | 763 / 1002 | 182.5 / 205.0 | 12328 / 13859 / 13859 | 5.5 |

- **TTFT** = prefill. Short prompts keep it small; long-context RAG is where it explodes.
- **TPOT** = per-output-token decode cost, bounded by memory bandwidth. `decode tok/s = 1000 / TPOT_p50`.
- `UD-Q2_K_XL` and `Q4_K_M` decode within 2% of each other here, for 0.11 GB difference on disk.

## My observation

UD-Q2_K_XL saves 0.11 GB (about 22%) compared with Q4_K_M. Its TTFT P50 is lower (763 ms vs 1031 ms), but TPOT and decode speed are nearly identical. Its E2E P50 is higher because output lengths varied between prompts. I would choose Q2 when memory or disk is limited, but not for a clear speed improvement; answer quality should decide the final choice.
