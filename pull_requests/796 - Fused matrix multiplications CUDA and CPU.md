## 🔀 [Pull Request #796](https://github.com/ikawrakow/ik_llama.cpp/pull/796) - Fused matrix multiplications (CUDA and CPU)

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/fuse_qkv` |
| **Target Branch** | `main` |
| **Created** | 2025-09-24 |
| **Updated** | 2025-09-24 |
| **Merged** | 2025-09-24 |

---

## 📄 Description

This PR adds the ability to fuse quantized matrix multiplication that operate on the same activations $A$. A typical example where fusing can be done for most models are the $Q \cdot A$, $K \cdot A$ and $V \cdot A$ products. The implementation does not go as far as fusing these into a single kernel, but it does save the quantization of $A$ into the required type, along with the thread synchronization between such matrix multiplications.

The performance gain is surprisingly small, typically being $\le 1$%. Nevertheless, adding this micro-optimization.