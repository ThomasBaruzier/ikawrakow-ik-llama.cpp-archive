## 🔀 [Pull Request #826](https://github.com/ikawrakow/ik_llama.cpp/pull/826) - Fix performance regression introduced in [#823](https://github.com/ikawrakow/ik_llama.cpp/issues/823)

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/fix_perf_regression` |
| **Target Branch** | `main` |
| **Created** | 2025-10-13 |
| **Updated** | 2025-10-13 |
| **Merged** | 2025-10-13 |

---

## 📄 Description

See [comment 1](https://github.com/ikawrakow/ik_llama.cpp/pull/823#issuecomment-3393901534), [comment 2](https://github.com/ikawrakow/ik_llama.cpp/pull/823#issuecomment-3394324831), [comment 3](https://github.com/ikawrakow/ik_llama.cpp/pull/823#issuecomment-3394882355)

The refactoring broke tensor overrides, and I did not catch it (tested GPU-only and CPU-only), but not hybrid GPU/CPU.

@usrlocalben @sousekd  Can you confirm that this fixes the performance regression for you? Thanks.

---

## 💬 Conversation

👤 **usrlocalben** commented on **2025-10-13** at **15:10:47**

@ikawrakow all good here:
```
generation eval time =   23259.97 ms /   403 runs   (   57.72 ms per token,    17.33 tokens per second)
```