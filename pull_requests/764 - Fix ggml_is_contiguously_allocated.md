## 🔀 [Pull Request #764](https://github.com/ikawrakow/ik_llama.cpp/pull/764) - Fix ggml_is_contiguously_allocated

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/fix_contiguously_allocated` |
| **Target Branch** | `main` |
| **Created** | 2025-09-05 |
| **Updated** | 2025-09-05 |
| **Merged** | 2025-09-05 |

---

## 📄 Description

Otherwise hybrid inference with quants that have per row scales leads to an assert for MoE models.