## 🔀 [Pull Request #843](https://github.com/ikawrakow/ik_llama.cpp/pull/843) - Do not allocate KV cache for unused layers

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/no_KV_for_unused_layers` |
| **Target Branch** | `main` |
| **Created** | 2025-10-20 |
| **Updated** | 2025-10-20 |
| **Merged** | 2025-10-20 |

---

## 📄 Description

Some models have multiple token prediction layer(s) that are currently not used. The tensors for these layers are ignored, but KV cache is still allocated. This PR removes the unnecessary KV cache allocation. Somewhat surprisingly, this has a measurable effect on PP performance (~2% for GLM-4.5-AIR with all experts left on the CPU and batch/u-batch size of 4096). 

Another minor change is to not add a scale op to the experts weights when scaling is specified by the model parameters but the actual scale is 1 (e.g., GLM-4.5-AIR).