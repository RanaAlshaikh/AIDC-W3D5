## The numbers

- Locked model: Qwen/Qwen2.5-1.5B-Instruct-AWQ
- Target p95 end-to-end latency (your SLO today): 5.0 seconds
- Knee concurrency (highest concurrency whose p95 is still under target): 4
- Tokens per second at the knee: 271.6
- Max sustainable request rate at the target p95: 2.3 req/s

## The limiting family

- Compute-bound: token generation throughput scales smoothly while decode latency stays tight, pointing to tensor core throughput rather than memory bandwidth stalls at this concurrency.

## Why the knee, not the peak

- Peak throughput inflates capacity by batching past the service-level agreement, whereas the knee measures the highest sustainable concurrency delivered strictly within the latency SLO.
