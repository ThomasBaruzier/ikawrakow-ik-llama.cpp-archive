## 🔀 [Pull Request #713](https://github.com/ikawrakow/ik_llama.cpp/pull/713) - CUDA: faster prompt processing for 4-bit quants

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/cuda_use_bperm` |
| **Target Branch** | `main` |
| **Created** | 2025-08-21 |
| **Updated** | 2025-08-21 |
| **Merged** | 2025-08-21 |

---

## 📄 Description

There is [this PR](https://github.com/ggml-org/llama.cpp/pull/15451) in mainline `llama.cpp`. It uses the `__byte_perm` Nvidia intrinsics to more efficiently assemble a 32-bit integer when each byte requires a lookup in a table of 16 values. These are the newly added `MXFP4`, along with `IQ4_NL, IQ4_XS, IQ4_KS, IQ4_KS_R4, IQ4_KSS`. I had noticed the `__byte_perm` instruction when searching for other SIMD instrinsics in the CUDA manual and had made a mental note to investigate the use of this instruction for handling lookup tables, but mainline PR 15451 already did it, so I could just take it from there.

As `IQ4_K` (and `IQ4_K_R4`) use blocks of 16, so the first 2 and second 2 bytes require different lookup tables, the trick is not directly applicable to these quantization types.

Here a quick performance comparison between the main branch and this PR on RTX-4080 

| model              |          test |   t/s (main)     |   t/s (PR)       |     Speedup |
| ------------------ | ------------: | ---------------: | ---------------: | ----------: |
| llama 8B IQ4_XS    |        pp1024 |  8206.24 ± 30.15 |  8887.15 ± 30.74 |  1.083      |   
| llama 8B IQ4_NL    |        pp1024 |  7692.93 ± 68.09 |  8597.81 ± 49.93 |  1.118      |   
| llama 8B IQ4_KS    |        pp1024 |  8191.36 ± 23.05 |   8618.85 ± 8.96 |  1.052      |   
| llama 8B IQ4_KS_R4 |        pp1024 |  8290.62 ± 21.63 |  8672.20 ± 10.09 |  1.046      |   
| llama 8B IQ4_KSS   |        pp1024 |  8139.23 ± 14.06 |  8685.01 ± 28.51 |  1.067      |   
| gpt-oss 20B MXFP4  |        pp2048 | 9618.83 ± 107.95 | 10355.26 ± 113.48|  1.077      |   
| llama 8B IQ4_XS    |         tg128 |    128.23 ± 0.08 |    129.05 ± 0.06 |  1.006      |
| llama 8B IQ4_NL    |         tg128 |    122.62 ± 0.05 |    123.68 ± 0.04 |  1.009      |
| llama 8B IQ4_KS    |         tg128 |    128.08 ± 0.04 |    128.68 ± 0.07 |  1.005      |
| llama 8B IQ4_KS_R4 |         tg128 |    124.45 ± 0.05 |    123.59 ± 0.03 |  0.993      |
| llama 8B IQ4_KSS   |         tg128 |    133.26 ± 0.08 |    134.02 ± 0.04 |  1.006      |
| gpt-oss 20B MXFP4  |         tg128 |    178.30 ± 0.11 |    180.72 ± 0.07 |  1.014      |

We see noticeable gains for PP. TG is severely memory bandwidth limited on the 4080, so much less impact there (but I wouldn't be surprised if the gain is somewhat larger on a 4090 or 5090).

As I had already done an optimized table lookup for the IQK quants, PP performance gains are somewhat lower than `MXFP4` and `IQ4_NL`.

---

## 💬 Conversation

👤 **Ph0rk0z** commented on **2025-08-21** at **18:47:12**

A hair under 20t/s up from 18 on qwen 235b.. wooo!