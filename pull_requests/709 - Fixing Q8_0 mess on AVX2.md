## 🔀 [Pull Request #709](https://github.com/ikawrakow/ik_llama.cpp/pull/709) - Fixing Q8_0 mess on AVX2

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Source Branch** | `ik/fix_q80_avx2_mess` |
| **Target Branch** | `main` |
| **Created** | 2025-08-19 |
| **Updated** | 2025-08-20 |

---

## 📄 Description

On `AVX2` we now use `Q8_0_X4` for the activations of `Q8_0` tensors. 

I also noticed NaNs when using repacking for `Q6_K`, so disabling for now.

---

## 💬 Conversation

👤 **ikawrakow** commented on **2025-08-20** at **04:16:42**

This break other quants.