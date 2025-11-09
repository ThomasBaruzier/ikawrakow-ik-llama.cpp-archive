## 🔀 [Pull Request #921](https://github.com/ikawrakow/ik_llama.cpp/pull/921) - CUDA: fuse copies to K and V cache

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/fuse_kvcache_copy` |
| **Target Branch** | `main` |
| **Created** | 2025-11-08 |
| **Updated** | 2025-11-08 |
| **Merged** | 2025-11-08 |

---

## 📄 Description

This is a minor optimization, only noticeable for TG with small, fully offloaded models.
It is only applied when batch size is 1, and only if the K= and V-cache are both `f16`.

As with other fusions, only enabled via `-cuda fusion=1`.