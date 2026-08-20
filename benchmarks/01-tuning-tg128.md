# 01 - Tune: thread-count sweep

Model `Qwen3.5-0.8B-Q4_K_M.gguf` � host `Windows-AMD64` � llama.cpp `b10488`
CPU: **4 physical � 8 logical** cores � `ngl=99` � metric `tg128`

| threads (-t) | tg128 (tok/s) | vs best |
|:--|--:|--:|
| 1 | 40.3 | 100% |
| 2 | 31.8 | 79% |
| 4 | 13.0 | 32% |
| 8 | 9.8 | 24% |
| 16 | 16.1 | 40% |

**Best**: `-t 1` at 40.3 tok/s
**Slowest tested**: `-t 8` at 9.8 tok/s (4.12x spread)
**Against the physical-core default** (`-t 4`, 13.0 tok/s): 3.10x

Use this in your run:

```bash
LAB_N_THREADS=1 make bench
```

## My explanation

The best point is unexpectedly one CPU thread: 40.3 tok/s, versus 13.0 tok/s at the four-physical-core default. This machine uses GPU offload (`ngl=99`), so decode is not purely CPU-bound. Extra CPU threads add scheduling and synchronization overhead around GPU work rather than increasing useful parallel compute.

The curve falls sharply from one to eight threads, then partially recovers at sixteen. This non-monotonic behavior suggests contention and timing variability rather than a simple CPU scaling curve. I would use one thread for this workload and validate it under serving load.
