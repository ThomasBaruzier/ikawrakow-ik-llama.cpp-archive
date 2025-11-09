## 🔀 [Pull Request #864](https://github.com/ikawrakow/ik_llama.cpp/pull/864) - CUDA: fuse ffn_up*unary_op(ffn_gate) for MMVQ (V2)

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/reorg_mmvq_and_fuse_bias` |
| **Target Branch** | `main` |
| **Created** | 2025-10-25 |
| **Updated** | 2025-10-26 |
| **Merged** | 2025-10-26 |

---

## 📄 Description

This PR fuses ffn_up*unary_op(ffn_gate) into a single kernel for token generation (batch size = 1). We get in the range of 3% speedup for TG for MoE models, and ~1% for dense models.

There are two differences to [#861](https://github.com/ikawrakow/ik_llama.cpp/issues/861)
* This PR also handles the addition of a bias in the fused kernel (relevant for GPT-OSS models) 
* It splits the `mmvq.cu` and `iqk_mmvq.cu` files into separate template instances. This results in a much faster build as compilation is now parallelized

Note that fusing is already done on the CPU back-end, so no changes there.

---

## 💬 Conversation

👤 **Ph0rk0z** commented on **2025-10-26** at **14:42:26**

No more crash. This one works now.