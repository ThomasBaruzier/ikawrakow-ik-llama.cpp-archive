## 🔀 [Pull Request #887](https://github.com/ikawrakow/ik_llama.cpp/pull/887) - RoPE cache

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/rope_cache` |
| **Target Branch** | `main` |
| **Created** | 2025-11-02 |
| **Updated** | 2025-11-03 |
| **Merged** | 2025-11-03 |

---

## 📄 Description

When performing the RoPE operation one needs to compute the rotation angles for each token in the batch and each `Q` and `K` attention head. When there are no per layer frequency factors, this computation only depends on the token position and the index within the attention head, so is exactly the same (per token) for every layer and each head. Hence, one could simply compute the cosine and sine of the rotation angles once per graph, and then reuse the result for all layers.

This PR implements this idea for a subset of the supported models (Qwen3, Qwen3-MoE, Ling/Ring, GPT-OSS, GLM-4.5-MoE).

We observe small but noticeable performance gains for PP and TG.

~The PR needs a bit more work to add a command-line argument to enable/disable this feature as the implementation is only for the CUDA and CPU back-ends.  Also, for now the implementation is only for the `NEOX` and `NORM` RoPE variants, so vision related RoPE variants still need to get implemented.~

The command line option to disable RoPE cache is `-no-rcache` or `--no-rope-cache`.

For now the optimization is only implemented for `NEOX` and `NORM` RoPE types. Adding implementation for vision related RoPE variants is left for a future PR.

~Still, putting it out there for testing.~

It will also  be interesting to see how long it will take until this optimization is fully independently discovered in mainline `llama.cpp` /s