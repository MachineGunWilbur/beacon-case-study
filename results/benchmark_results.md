# Benchmark Results

## Environment

- Provider: RunPod
- GPU: 1× NVIDIA H100 80GB HBM3
- Model: Qwen/Qwen3-8B
- Serving stack: vLLM
- Max context: 32,768 tokens
- Prefix caching: enabled
- SLA target: p95 TTFT < 2,000 ms, p95 ITL < 60 ms

## Chat — cold prefix

### Saturation
- Input/output: 4,334 / 203 tokens
- Requests: 100
- Request rate: unlimited
- Achieved throughput: 5.55 req/s
- Total logical throughput: 25,188 tok/s
- Median TTFT: 5,825 ms
- p99 TTFT: 15,842 ms
- Median ITL: 26.3 ms
- p99 ITL: 435 ms
- Result: FAIL

### 3.0 RPS
- Achieved throughput: 2.83 req/s
- Total logical throughput: 12,860 tok/s
- p95 TTFT: 348 ms
- p95 ITL: 18.7 ms
- Result: PASS

### 3.5 RPS
- Achieved throughput: 3.26 req/s
- Total logical throughput: 14,803 tok/s
- p95 TTFT: 411 ms
- p95 ITL: 46.7 ms
- Result: PASS

### 3.6 RPS
- Achieved throughput: 3.35 req/s
- Total logical throughput: 15,192 tok/s
- p95 TTFT: 405 ms
- p95 ITL: 43.1 ms
- Result: PASS

### 3.7 RPS
- Achieved throughput: 3.43 req/s
- Total logical throughput: 15,559 tok/s
- p95 TTFT: 396 ms
- p95 ITL: 35.7 ms
- Result: PASS

### 3.75 RPS
- Achieved throughput: 3.47 req/s
- Total logical throughput: 15,730 tok/s
- p95 TTFT: 457 ms
- p95 ITL: 71.1 ms
- Result: FAIL

### 4.0 RPS
- Achieved throughput: 3.67 req/s
- Total logical throughput: 16,648 tok/s
- p95 TTFT: 427 ms
- p95 ITL: 79.7 ms
- Result: FAIL

### 4.5 RPS
- Achieved throughput: 4.05 req/s
- Total logical throughput: 18,386 tok/s
- p95 TTFT: 528 ms
- p95 ITL: 85.5 ms
- Result: FAIL

## Chat — warm prefix

Representative shape:
- Shared prefix: 4,210 tokens
- Unique suffix: 124 tokens
- Total input: ~4,334 tokens
- Output: 203 tokens
- Prefixes: 40

### 80-request saturation
- Achieved throughput: 14.84 req/s
- Total logical throughput: 67,330 tok/s
- p95 TTFT: 1,063 ms
- p95 ITL: 26.7 ms
- Result: PASS

### 160-request saturation
- Achieved throughput: 12.85 req/s
- Total logical throughput: 58,290 tok/s
- p95 TTFT: 4,962 ms
- p95 ITL: 46.6 ms
- Result: FAIL on TTFT

### 10 RPS
- Achieved throughput: 8.90 req/s
- Total logical throughput: 40,375 tok/s
- p95 TTFT: 106 ms
- p95 ITL: 15.9 ms
- Result: PASS

### 12 RPS
- Achieved throughput: 10.28 req/s
- Total logical throughput: 46,627 tok/s
- p95 TTFT: 79 ms
- p95 ITL: 17.3 ms
- Result: PASS

## Analysis workload

Representative shape:
- Input: 18,712 tokens
- Output: 372 tokens
- Very little shared-prefix reuse

### Saturation
- Requests: 40
- Request rate: unlimited
- Achieved throughput: 0.92 req/s
- Total logical throughput: 17,591 tok/s
- p95 TTFT: 32,896 ms
- p95 ITL: 265.7 ms
- Result: FAIL

### 0.5 RPS
- Achieved throughput: 0.48 req/s
- Total logical throughput: 9,131 tok/s
- p95 TTFT: 1,317 ms
- p95 ITL: 14.0 ms
- Result: PASS

### 0.7 RPS
- Achieved throughput: 0.65 req/s
- Total logical throughput: 12,433 tok/s
- p95 TTFT: 1,367 ms
- p95 ITL: 21.3 ms
- Result: PASS

## Notes

- "Logical throughput" counts the full request token volume reported by the benchmark, including cached prefix tokens.
- It should not be interpreted as fresh prefill compute.
- Cold-prefix SLA boundary showed run-to-run variance; I treat ~3.5–3.7 offered RPS as the approximate safe region rather than claiming a precise cutoff.
- The warm-prefix results show that cache locality has a large effect on this workload.