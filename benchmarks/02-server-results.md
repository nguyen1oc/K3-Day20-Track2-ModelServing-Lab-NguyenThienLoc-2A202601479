# 02 - Serve: load test + saturation reading

Host `Windows-AMD64` · llama.cpp `b10488` ·
`--parallel 4` · `ctx=2048` · `threads=1` ·
`ngl=99`

| Users | Reqs | RPS | P50 (ms) | P95 (ms) | P99 (ms) | Eff. concurrency | Failures |
|:--|--:|--:|--:|--:|--:|--:|--:|
| 10 | 46 | 0.81 | 9300 | 16000 | 20000 | 8.1 | 0.0% |
| 50 | 26 | 0.47 | 22000 | 54000 | 55000 | 11.6 | 0.0% |

*Effective concurrency = RPS x average latency (Little's Law) -- how many requests were
really in flight, regardless of how many users locust simulated. It counts queued requests
too, so the occupancy/slot ratio can legitimately exceed 1.0; it is occupancy, not
utilisation. For true slot utilisation use the server's own gauges (`make metrics`).*

## What these two runs say

| Going from 10 to 50 users | |
|:--|--:|
| Offered load | 5x |
| Throughput actually delivered | **0.58x** (12% of linear) |
| P95 latency | **3.38x** |
| Effective concurrency at 50 users | 11.6 vs `--parallel 4` slots (occupancy/slot ratio 2.91) |

**Saturated.** Throughput delivered only 0.58x for 5x the offered load, and effective concurrency (11.6) is at or above all 4 decode slots. Saturation sets in somewhere at or below 50 users; the load you added beyond that point became queue time rather than throughput.

Throughput moved 0.58x while P95 moved 3.38x. That gap is the goodput argument: past saturation you buy throughput by spending latency, and if your SLO is a P95 target then the requests you added are no longer being served within it. (This lab does not fix an SLO number for you -- pick one in your write-up and state how much goodput you keep at it.)

## My reading

The server is saturated at 50 users. RPS fell from 0.81 to 0.47 even though the
offered load increased 5x, while P95 rose from 16 s to 54 s. The peak busy-slot
reading of 3.92/4 confirms that all decode slots were occupied; the extra latency
was mainly queue time. I would tune `--parallel` first and re-measure, because it
directly controls concurrent decode capacity and can improve goodput before adding
more hardware.
