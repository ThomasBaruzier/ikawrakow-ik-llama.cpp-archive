## 🔀 [Pull Request #760](https://github.com/ikawrakow/ik_llama.cpp/pull/760) - llama: Enable K-shift for quantized KV cache for cuda

| **Author** | `firecoperana` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `fcp/kshift` |
| **Target Branch** | `main` |
| **Created** | 2025-09-04 |
| **Updated** | 2025-10-26 |
| **Merged** | 2025-09-05 |

---

## 📄 Description

This fixed the crash when doing context shift for quantized kv cache. Before this PR, I can only use fp16 for kv cache . Tested with both q8_0 and q4_0 and don't see crash.

https://github.com/ggml-org/llama.cpp/pull/9571
It will fail on unsupported backends or quant types.


- [x] I have read the [contributing guidelines](https://github.com/ggerganov/llama.cpp/blob/master/CONTRIBUTING.md)
- Self-reported review complexity:
  - [X] Low
  - [ ] Medium
  - [ ] High

---

## 💬 Conversation

👤 **ikawrakow** approved this pull request ✅ on **2025-09-05** at **06:47:30**