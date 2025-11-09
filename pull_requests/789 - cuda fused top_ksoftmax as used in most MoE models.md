## 🔀 [Pull Request #789](https://github.com/ikawrakow/ik_llama.cpp/pull/789) - cuda: fused top_k+softmax as used in most MoE models

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/cuda_topk_moe` |
| **Target Branch** | `main` |
| **Created** | 2025-09-23 |
| **Updated** | 2025-09-23 |
| **Merged** | 2025-09-23 |

---

## 📄 Description

Adapted from PR 16130 in mainline (but fixed the bug in that PR that leads to illegal memory access).

Has negligible impact on PP performance.

Results in a 3-6% TG performance gain when the model is fully offloaded to the GPU.