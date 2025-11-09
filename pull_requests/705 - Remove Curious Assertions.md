## 🔀 [Pull Request #705](https://github.com/ikawrakow/ik_llama.cpp/pull/705) - Remove Curious Assertions

| **Author** | `usrlocalben` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `remove_curious_assertions` |
| **Target Branch** | `main` |
| **Created** | 2025-08-19 |
| **Updated** | 2025-08-19 |
| **Merged** | 2025-08-19 |

---

## 📄 Description

This assertion can hit during prefill as MLA/KV tensors grow, e.g. Kimi K2 n_ctx >= 32768.

Closes [#704](https://github.com/ikawrakow/ik_llama.cpp/issues/704) 


- [x] I have read the [contributing guidelines](https://github.com/ggerganov/llama.cpp/blob/master/CONTRIBUTING.md)
- Self-reported review complexity:
  - [x] Low
  - [ ] Medium
  - [ ] High

---

## 💬 Conversation

👤 **ikawrakow** commented on **2025-08-19** at **11:41:04**

So, as part of the CUDA_GRAPH port I found it easier to copy mainline's `cpy.cu` and add the necessary stuff than to adapt the existing `cpy.cu` to the needs of CUDA graphs. When doing so I forgot about these asserts. But looking at the current code, there will be an integer overflow if `ggml_nelements(src) >=  INT_MAX`, so I'm surprised that it works with this fix. I think one needs to just replace `int n` with `int64_t n` everywhere.

But we can merge this and I can do it later.

---

👤 **ikawrakow** approved this pull request ✅ on **2025-08-19** at **11:41:23**