## 🔀 [Pull Request #714](https://github.com/ikawrakow/ik_llama.cpp/pull/714) - CUDA: faster IQ3_K, IQ3_KS, IQ3_K_R4

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/cuda_iq3k_use_bperm1` |
| **Target Branch** | `main` |
| **Created** | 2025-08-21 |
| **Updated** | 2025-08-21 |
| **Merged** | 2025-08-21 |

---

## 📄 Description

This PR is a follow up of [#713](https://github.com/ikawrakow/ik_llama.cpp/issues/713), and applies a similar trick to 3-bit quants that need a table lookup (`IQ3_K, IQ3_KS, IQ3_K_R4`).

Here some performance comparisons to the main branch for LlaMA-3.1-8B-Instruct on RTX-4080

| model              |          test |     t/s (main)   |     t/s (PR)     |  Speedup  |
| ------------------ | ------------: | ---------------: | ---------------: | --------: |
| llama 8B IQ3_KS    |         pp512 |  8096.66 ± 36.79 |  8507.01 ± 44.55 |  1.051    |   
| llama 8B IQ3_K     |         pp512 |  6705.65 ± 29.94 |  7027.30 ± 36.92 |  1.048    |   
| llama 8B IQ3_K_R4 |         pp512 |  6503.14 ± 46.09 |  7062.74 ± 38.44 |  1.086    |   
| llama 8B IQ3_KS    |         tg128 |    148.14 ± 0.32 |    154.84 ± 0.08 |  1.045    |   
| llama 8B IQ3_K     |         tg128 |    144.16 ± 0.15 |    148.57 ± 0.15 |  1.031    |   
| llama 8B IQ3_K_R4 |         tg128 |    138.70 ± 0.02 |    145.49 ± 0.10 |  1.049    |