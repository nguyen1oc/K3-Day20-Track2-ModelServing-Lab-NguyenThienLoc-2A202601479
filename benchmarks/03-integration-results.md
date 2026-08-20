# 03 - Integrate: RAG pipeline run

Host `Windows-AMD64` · llama.cpp `b10488` ·
retrieval backend: **keyword overlap** · 3 queries

| Query | Contexts retrieved | embed (ms) | retrieve (ms) | llm (ms) | total (ms) |
|:--|--:|--:|--:|--:|--:|
| Why is goodput more useful than raw throughp... | goodput, paged, radix | 0.0 | 0.0 | 5743.6 | 5743.7 |
| What problem does PagedAttention actually so... | paged, radix, disagg | 0.0 | 0.1 | 4687.4 | 4687.5 |
| When does splitting prefill and decode help?... | disagg, radix, batching | 0.0 | 0.1 | 6738.2 | 6738.4 |

Mean per stage (ms): embed **0.0** · retrieve **0.1** ·
llm **5723.1** · total **5723.2**
Dominant stage: **llm** (100% of total)

## Answers returned

**Why is goodput more useful than raw throughput?**

> Based on the provided context, **Goodput** is more useful than raw throughput because it focuses on **slo (Service Level Objectives) compliance** rather than purely maximizing throughput.

While raw throughput measures requests per second (RPS) that met specific targets, Goodput counts only requests that met the **TTFT** (Total Time to Failure) and **TPOT** (Total Time to Process) targets. This me

**What problem does PagedAttention actually solve?**

> PagedAttention solves the problem of **internal fragmentation** in GPU memory.

By storing the Key-Value (KV) cache in non-contiguous pages, the model avoids the wasted space that would occur if the KV cache were stored contiguously in contiguous memory blocks. This technique is particularly effective for large models where the KV cache is large and non-contiguous, allowing the GPU to utilize more

**When does splitting prefill and decode help?**

> Splitting prefill and decode helps when the **prefill operation is compute-bound** (requiring significant CPU/GPU power) and the **decode operation is memory-bandwidth-bound** (requiring significant memory throughput).

By distributing these tasks across separate pools, the system can utilize different hardware resources more efficiently:
*   **Prefill:** Can be offloaded to compute-bound resource


## Which N16-N19 pieces are real

N16 Cloud/IaC, N17 data pipeline, and N18 lakehouse are stubs in this demo. N19
retrieval is a keyword-overlap fallback, not a real embedding/vector service. N20 is
real: it calls the local llama-server. The LLM dominates latency (100%), as expected
for a small local RAG pipeline. To halve total latency, I would reduce LLM decode work
first by limiting output tokens or using a faster serving configuration.
