## 🔀 [Pull Request #880](https://github.com/ikawrakow/ik_llama.cpp/pull/880) - Biased mmvq: minor optimization

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/biased_mmvq` |
| **Target Branch** | `main` |
| **Created** | 2025-10-30 |
| **Updated** | 2025-10-31 |
| **Merged** | 2025-10-31 |

---

## 📄 Description

This PR derives from [PR 16847](https://github.com/ggml-org/llama.cpp/pull/16847) in mainline.

On my GPU (RTX-4080) it is a very minor improvement over the main branch (~0.5% better TG for GPT-OSS-20B-MXFP4, less for other models). But based on the discussion in the mainline PR, it may lead to larger performance gains for low memory bandwidth GPUs.

The PR also adds the `-mmvq | --merge-qkv` option (see [#878](https://github.com/ikawrakow/ik_llama.cpp/issues/878)) to `llama-bench`.