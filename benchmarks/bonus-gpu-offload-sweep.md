# Bonus - GPU offload sweep

Host `Windows-AMD64` · backend(s) `nvidia_cuda, vulkan` ·
llama.cpp `b10488` · `threads=1` · metric `tg128`

| -ngl | tg128 (tok/s) | vs -ngl 0 | vs best |
|:--|--:|--:|--:|
| 0 | 15.5 | 1.00x | 55% |
| 8 | 22.9 | 1.48x | 82% |
| 16 | 27.9 | 1.80x | 100% |
| 24 | 24.3 | 1.57x | 87% |
| 32 | 22.7 | 1.47x | 81% |
| 99 | 17.1 | 1.11x | 61% |

Best: `-ngl 16` at 27.9 tok/s
-- 1.80x faster than CPU-only.

Where the curve flattens tells you the model ran out of layers to move. Where it
*peaks below* full offload tells you something did not fit and the accelerator
started paying to fetch weights it could not hold.

## My finding

Full offload was not best on this MX350. Performance peaked at `-ngl 16` with
27.9 tok/s, 1.80x faster than CPU-only, then fell to 17.1 tok/s at `-ngl 99`.
The 2 GB GPU has limited VRAM for model weights, the KV cache, and runtime overhead.
Moving too many layers therefore increases transfers or memory pressure across the
host-device link. A partial offload gives the best balance on this machine.
