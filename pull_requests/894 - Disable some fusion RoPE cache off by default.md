## 🔀 [Pull Request #894](https://github.com/ikawrakow/ik_llama.cpp/pull/894) - Disable some fusion, RoPE cache off by default

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/disable_some_fusion` |
| **Target Branch** | `main` |
| **Created** | 2025-11-04 |
| **Updated** | 2025-11-04 |
| **Merged** | 2025-11-04 |

---

## 📄 Description

[#893](https://github.com/ikawrakow/ik_llama.cpp/issues/893) reports issues with RoPE cache. I suspect it is because of the associated fusion, but just in case,
* Make RoPE cache to be off by default. It can be enabled with `-rcache` or `--rope-cache`
* Disable the fused `fused_rms+fused_rms+rope+rope` op