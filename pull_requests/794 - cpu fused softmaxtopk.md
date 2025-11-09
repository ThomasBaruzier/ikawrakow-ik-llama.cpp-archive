## 🔀 [Pull Request #794](https://github.com/ikawrakow/ik_llama.cpp/pull/794) - cpu: fused softmax+topk

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/cpu_topk_moe` |
| **Target Branch** | `main` |
| **Created** | 2025-09-24 |
| **Updated** | 2025-09-24 |
| **Merged** | 2025-09-24 |

---

## 📄 Description

CPU implementation of fused top-k + softmax. See also [#789](https://github.com/ikawrakow/ik_llama.cpp/issues/789), which did the same for the CUDA back-end.

Leads to 2-3% PP and TG improvement for most MoE models.