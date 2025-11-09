## 🔀 [Pull Request #911](https://github.com/ikawrakow/ik_llama.cpp/pull/911) - Fix iqk_mul_mat when number of rows is not multiple of repack rows

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/fix_iqk_for_strange_numrows` |
| **Target Branch** | `main` |
| **Created** | 2025-11-06 |
| **Updated** | 2025-11-07 |
| **Merged** | 2025-11-06 |

---

## 📄 Description

I was playing with Qwen3-0.6B and got an assert when running CPU only. The issue is that this model used the token embedding tensor as the output tensor, and the number of rows (which is the number of tokens in the vocabulary) is 151669. When performing the final matrix multiplication with this tensor we decide that it is useful to repack it on the fly, to `Q8_0_R8` in that case. But the number of rows is not a multiple of 8, so we hit an assert.

The PR fixes this by avoiding to go the repacked route if the number of rows is not a multiple of interleaved rows.