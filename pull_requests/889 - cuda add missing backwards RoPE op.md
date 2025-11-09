## 🔀 [Pull Request #889](https://github.com/ikawrakow/ik_llama.cpp/pull/889) - cuda: add missing backwards RoPE op

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/cuda_rope_back` |
| **Target Branch** | `main` |
| **Created** | 2025-11-03 |
| **Updated** | 2025-11-03 |
| **Merged** | 2025-11-03 |

---

## 📄 Description

While working on the RoPE cache PR ([#887](https://github.com/ikawrakow/ik_llama.cpp/issues/887)), I noticed that the backwards RoPE (needed for k-shift) is missing on CUDA. Someone was mentioning that k-shift was unbearably slow. Well, this explains it.

This PR adds the backwards RoPE to the CUDA back-end. It was just a matter of adding a call to the appropriate function and answering with "yes" the question if `ROPE_BACK` is supported.