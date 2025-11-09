## 🔀 [Pull Request #852](https://github.com/ikawrakow/ik_llama.cpp/pull/852) - Fuse add + fused_rms_norm (CUDA)

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/fuse_add_fused_rms` |
| **Target Branch** | `main` |
| **Created** | 2025-10-21 |
| **Updated** | 2025-10-21 |
| **Merged** | 2025-10-21 |

---

## 📄 Description

Almost all LLMs add the result of a layer to the input. This becomes the input to the next layer, which is then subject to `RMS_NORM` plus multiplication with the attention norm tensor (handled as `FUSED_RMS_NORM` in `ik_llama.cpp`). Hence, it is useful to fuse `ADD` and `FUSED_RMS_NORM` to save one kernel launch (GPU) or thread synchronization point (CPU) per layer.

This PR adds this fusion op on CUDA.

We observe 1-2% TG performance gains (kernel launch overhead is negligible for PP, so no performance impact there).

Oh, I also re-formatted `llama-build-context.cpp` to have all calls to `llm_build_norm` on a single line. That way it is easier to grep for `RMS_NORM` operations to see their arguments.