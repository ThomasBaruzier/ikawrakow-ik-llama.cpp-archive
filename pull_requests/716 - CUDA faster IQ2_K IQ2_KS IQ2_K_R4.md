## 🔀 [Pull Request #716](https://github.com/ikawrakow/ik_llama.cpp/pull/716) - CUDA: faster IQ2_K, IQ2_KS, IQ2_K_R4

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/cuda_iq2k_use_bperm1` |
| **Target Branch** | `main` |
| **Created** | 2025-08-21 |
| **Updated** | 2025-08-22 |
| **Merged** | 2025-08-22 |

---

## 📄 Description

This PR is a follow up of [#713](https://github.com/ikawrakow/ik_llama.cpp/issues/713), [#714](https://github.com/ikawrakow/ik_llama.cpp/issues/714), and applies a similar trick to 2-bit quants that need a table lookup (`IQ2_K, IQ2_KS, IQ2_K_R4`).

| model              |          test |    t/s (main)    |    t/s (PR)      |  Speedup  |
| ------------------ | ------------: | ---------------: | ---------------: | --------: |
| llama 8B IQ2_KS    |         pp512 |  8673.51 ± 56.38 |  9289.24 ± 64.59 |  1.071    |
| llama 8B IQ2_K     |         pp512 |  7230.06 ± 37.36 |  7569.58 ± 64.24 |  1.047    |
| llama 8B IQ2_K_R4  |         pp512 |  7414.71 ± 47.02 |  7611.86 ± 41.09 |  1.027    |
| llama 8B IQ2_KS    |         tg128 |    178.04 ± 0.16 |    190.74 ± 0.25 |  1.071    |
| llama 8B IQ2_K     |         tg128 |    183.20 ± 0.24 |    188.78 ± 0.11 |  1.030    |
| llama 8B IQ2_K_R4  |         tg128 |    172.98 ± 0.21 |    184.66 ± 0.08 |  1.068    |

`IQ2_KS` is now the new prompt processing speed champion (previous was `IQ2_KT`).