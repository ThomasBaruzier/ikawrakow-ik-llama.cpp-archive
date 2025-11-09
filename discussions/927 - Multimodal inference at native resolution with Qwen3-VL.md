## 🗣️ [Discussion #927](https://github.com/ikawrakow/ik_llama.cpp/discussions/927) - Multimodal inference at native resolution with Qwen3-VL

| **Author** | `vapetrov` |
| :--- | :--- |
| **State** | ✅ **Answered** |
| **Created** | 2025-11-09 |
| **Updated** | 2025-11-09 |

---

## 📄 Description

I'm running commit `e5fc02c` and using the model `unsloth/Qwen3-VL-8B-Instruct-Q6_K`.
One of Qwen3-VL's key features is the ability to process images at their native resolution.
With `ik_llama.cpp` I get exactly 768 tokens regardless of the input image size.
With `llama.cpp` I get between 3,000 - 4,000 tokens depending on image size.
With `transformers` I get roughly 1,000 tokens/ megapixel.

Is there any way to get ik_llama to use the native image size, or at least increase the number of tokens to something similar to the upstream llama.cpp?

---

## 💬 Discussion

👤 **ikawrakow** commented on **2025-11-09** at **06:35:39**

IIRC, the ability to handle different resolutions got added to `llama.cpp` only very recently. We haven't come around to port these changes to `ik_llama.cpp`, but it will eventually happen.