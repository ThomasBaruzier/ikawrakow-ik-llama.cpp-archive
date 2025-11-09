## 🔀 [Pull Request #734](https://github.com/ikawrakow/ik_llama.cpp/pull/734) - Heuristics for mmq_id -> original threshold

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/mmq_id_thresh` |
| **Target Branch** | `main` |
| **Created** | 2025-08-27 |
| **Updated** | 2025-08-27 |
| **Merged** | 2025-08-27 |

---

## 📄 Description

This is a follow up of [#728](https://github.com/ikawrakow/ik_llama.cpp/issues/728).

For large enough u-batches the original implementation of the fused `ffn_up_exps+ffn_gate_exps` op becomes faster than the `mmq_id` implementation added in [#728](https://github.com/ikawrakow/ik_llama.cpp/issues/728). In [#728](https://github.com/ikawrakow/ik_llama.cpp/issues/728) a fixed threshold of `u-batch = 2048` was used to transition to the original implementation. I have now investigated the speed of original vs `mmq_id` for 3 models with different number of total and active experts, and it looks like the best heuristics is to use `mmq_id` for `u-batch <= 32 * total_experts`. This PR makes this simple change.