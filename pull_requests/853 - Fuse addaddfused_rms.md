## 🔀 [Pull Request #853](https://github.com/ikawrakow/ik_llama.cpp/pull/853) - Fuse add+add+fused_rms

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/fuse_add_add_fused_rms` |
| **Target Branch** | `main` |
| **Created** | 2025-10-21 |
| **Updated** | 2025-10-22 |
| **Merged** | 2025-10-22 |

---

## 📄 Description

As a follow up of [#852](https://github.com/ikawrakow/ik_llama.cpp/issues/852), some models have two adds before the fused `RMS_NORM` op, so we can easily fuse also the additional add op.

This PR does that on CUDA. We see 0.5-1% TG performance gains.