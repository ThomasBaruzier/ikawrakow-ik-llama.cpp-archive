## 🔀 [Pull Request #928](https://github.com/ikawrakow/ik_llama.cpp/pull/928) - DeepSeek TG optimizations

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | ✅ **Open** |
| **Source Branch** | `ik/deepseek_opt` |
| **Target Branch** | `main` |
| **Created** | 2025-11-09 |

---

## 📄 Description

This PR adds some optimizations to the DeepSeek MLA attention compute graph:

* Fuse concat and copy into K cache (only enabled if `-cuda fusion=1`)
* Avoids a copy when batch size is 1 (as in TG)
 
Combined effect: about +2% in TG performance with full GPU offload (tested with DeepSeek-Lite on RTX-4080)