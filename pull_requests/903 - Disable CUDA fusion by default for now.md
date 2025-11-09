## 🔀 [Pull Request #903](https://github.com/ikawrakow/ik_llama.cpp/pull/903) - Disable CUDA fusion by default for now

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/disable_fusion_by_default` |
| **Target Branch** | `main` |
| **Created** | 2025-11-05 |
| **Updated** | 2025-11-05 |
| **Merged** | 2025-11-05 |

---

## 📄 Description

Issues have been reported, so disabling by default for now.

But if it works for you, enable via
```
cmake -DGGML_CUDA_FUSION=1 $other_cmake_args
```