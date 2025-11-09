## 🔀 [Pull Request #916](https://github.com/ikawrakow/ik_llama.cpp/pull/916) - Disable add + fused_rms_norm fusion

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/disable_add_fused_rms` |
| **Target Branch** | `main` |
| **Created** | 2025-11-07 |
| **Updated** | 2025-11-07 |
| **Merged** | 2025-11-07 |

---

## 📄 Description

This fused kernel computes two things at once that are both needed later
* `c = a + b`
* `d = fused_rms_norm(c)`

It turns out the back-end allocates `c` to go into `a`, and `d` to go into `b` (they are all the same shape and `a` and `b` are not needed after `c` has been computed). This somehow leads to a data race (I don't really understand why we get a race, but I observe it in practice as results changing from run to run).

Hence, this PR disables this fusion. This should hopefully solve the issues with GLM-4.5/6/Air and fusion.

---

## 💬 Conversation

👤 **ikawrakow** commented on **2025-11-07** at **17:10:54**

Btw, if anyone sees how the data ace occurs, I would appreciate an explanation. The kernel in question is https://github.com/ikawrakow/ik_llama.cpp/blob/12b9902395684fc204a4a45b43d45189aadfe5a1/ggml/src/ggml-cuda/norm.cu#L460

---

👤 **Ph0rk0z** commented on **2025-11-07** at **22:55:25**

I had fusion at 0 but it appears I got a speed drop anyway?