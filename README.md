# Lab W3D5: vLLM Benchmark & Capacity Note

## 1. Predictions (by hand)

* **Target p95 Latency (SLO):** `5` seconds
* **Predicted Knee Concurrency:** `8`


## 2. Benchmark Results

 **Model:** `Qwen/Qwen2.5-1.5B-Instruct-AWQ`
 
 **GPU:** NVIDIA T4 (16 GB)
 
 **Serving Stack:** vLLM (`--dtype half --quantization awq --gpu-memory-utilization 0.85`)

| Concurrency | Throughput (tok/s) | TTFT p50 (s) | TTFT p95 (s) | Latency p95 (s) | Errors | Status |
|:-----------:|:------------------:|:------------:|:------------:|:---------------:|:------:|:------:|
| 1           | 72.75              | 0.085        | 0.323        | 2.386           | 0      | OK     |
| 2           | 169.74             | 0.060        | 0.092        | 1.702           | 0      | OK     |
| 4           | 281.15             | 0.059        | 0.116        | 1.795           | 0      | OK     |
| 8           | 431.70             | 0.142        | 0.433        | 2.366           | 0      | OK     |
| 16          | 701.61             | 0.243        | 0.247        | 2.363           | 0      | OK     |

### Knee Analysis
* **Measured Knee Concurrency:** `16` (Sweep-bounded)
* **Tokens/sec at Knee:** `701.61`
* **Latency p95 at Knee:** `2.363` s
* **Finding:** With an SLO of `5.0s`, latency stayed well under target through concurrency 16. Throughput climbed continuously from `72.8 tok/s` to `701.6 tok/s` without flattening, confirming the true hardware knee sits beyond concurrency 16.


## 3. Verification

`GREEN CHECK: PASS`

<img width="561" height="72" alt="Screenshot 2026-09-03 at 2 54 39 PM" src="https://github.com/user-attachments/assets/0d0c7f3c-e205-426c-8546-0c12c5ef8a4b" />

---

# Lab: Serving Cost & Scale-Out Economics

## 1. Predictions (by hand)

* **Formula:** `Cost per MTok ($) = (GPU_Hourly_Rate / (Tokens_Per_Sec * 3600)) * 1,000,000`
* **Scale Decision:** Adding a second GPU/replica at the knee concurrency is required to maintain the latency target. Pushing a single GPU further lowers paper cost per token, but violates the p95 SLO.


## 2. Key Results

* **GPU Hourly Rate:** `$0.35`
* **SLO Target (p95):** `<= 5.0s`
* **Operating Knee Concurrency:** `16` (sweep-bounded)
* **Knee Throughput:** `701.61 tok/s`
* **Knee Latency (p95):** `2.36s`
* **Cost at Knee:** `$0.1386 / M tokens`

### Scale-Out Plan

* **1x Knee (701.6 tok/s):** 1 replica | **$0.35/hr** | p95: 2.36s
* **1.5x Knee (1,052.4 tok/s):** 2 replicas | **$0.70/hr** | p95: 2.36s
* **2x Knee (1,403.2 tok/s):** 2 replicas | **$0.70/hr** | p95: 2.36s
* **3x Knee (2,104.8 tok/s):** 3 replicas | **$1.05/hr** | p95: 2.36s


## 3. Verification

`GREEN CHECK: PASS`

<img width="491" height="52" alt="Screenshot 2026-09-03 at 3 21 13 PM" src="https://github.com/user-attachments/assets/e7bdb6ea-1183-4ee8-be9f-1e6ed93ed05c" />

