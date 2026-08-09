# Beacon Case Study

## 1. Recommendation

Quote Beacon 9× H100 at \$3.60/GPU-hour, or \$32.40/hour total.

At Beacon's current 360B tokens/month, that is approximately \$0.0648 per
million tokens. My sizing estimate puts stock vLLM at ~12 H100s at Beacon's
2× peak. Because Tessera commits in 8-GPU nodes, its smallest configuration
that clears that requirement is 16 H100s at \$47.20/hour, or \$0.0944 per
million tokens.

That makes the proposed configuration approximately 31% cheaper per token.

## 2. The Case

Beacon averages 500M tokens/hour and peaks at approximately 1B tokens/hour
(277.8k tokens/s). The supplied traffic sample is 72.7% chat and 27.3%
analysis by token volume.

On one H100 running stock vLLM and Qwen3-8B, I measured three representative
workloads. Cold-prefix chat sustained roughly 15.5k logical tokens/s near the
SLA boundary. Analysis traffic sustained ~12.4k tokens/s at 0.7 RPS while
meeting the SLA. Chat with strong prefix reuse was dramatically cheaper:
a 12 RPS offered load produced ~46.6k logical tokens/s while remaining well
inside the SLA.

Using Beacon's reported 71% cache-hit rate as an explicit approximation between
my cold- and warm-chat measurements gives ~24.2k Beacon-shaped logical
tokens/s/GPU for stock vLLM. At the 2× peak that implies ~11.46 GPUs, rounded
to 12.

Applying the proposed stack's 1.45× tokens/GPU-hour claim gives a theoretical
requirement of 7.91 GPUs. I would not quote 8: it requires at least a 1.433×
uplift, leaving almost no room for the claim to degrade in production. I add
one GPU of margin and recommend 9.

I did not independently measure our production stack at 1.45×. I did,
however, observe that Beacon's workload has substantial performance headroom
from cache locality: warm-prefix chat handled several times the request rate
of cold-prefix chat while meeting the same latency SLA. That makes a 1.45×
uplift from prefix-aware routing and tuned batching technically credible,
though not proven by this benchmark.

## 3. Working

Environment:
- 1× NVIDIA H100 80GB HBM3 on RunPod
- Qwen/Qwen3-8B
- vLLM
- 32,768-token max context
- Prefix caching enabled
- Beacon SLA: p95 TTFT < 2,000 ms; p95 ITL < 60 ms

Representative results:

| Workload | Offered load | Achieved | Logical tok/s | p95 TTFT | p95 ITL | SLA |
|---|---:|---:|---:|---:|---:|---|
| Chat, cold prefix | 3.7 RPS | 3.43 RPS | 15,559 | 396 ms | 35.7 ms | Pass |
| Chat, cold prefix | 3.75 RPS | 3.47 RPS | 15,730 | 457 ms | 71.1 ms | Fail |
| Chat, warm prefix | 12 RPS | 10.28 RPS | 46,627 | 79 ms | 17.3 ms | Pass |
| Analysis | 0.7 RPS | 0.65 RPS | 12,433 | 1,367 ms | 21.3 ms | Pass |
| Analysis saturation | unlimited | 0.92 RPS | 17,591 | 32,896 ms | 265.7 ms | Fail |

Full traffic analysis, calculations, assumptions, and intermediate benchmark
results are in `analysis.ipynb`.

## 4. What I Am Least Sure About

The weakest assumption is how Beacon's reported 71% cache-hit rate maps to
multi-GPU effective throughput. Beacon itself is unsure how that dashboard
metric is calculated, and I approximated production chat capacity by
interpolating between measured cold- and warm-prefix single-GPU cases. That is
useful for sizing but is not equivalent to replaying Beacon's real request
sequence across a round-robin pool. The next measurement I would run is an
A/B replay of representative Beacon traffic across several stock-vLLM workers
behind round-robin versus the prefix-aware serving stack, measuring actual
cache hits, tokens/GPU-hour, and SLA compliance. That would directly validate
or reject the 1.45× assumption.
