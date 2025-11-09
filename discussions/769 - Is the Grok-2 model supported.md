## 🗣️ [Discussion #769](https://github.com/ikawrakow/ik_llama.cpp/discussions/769) - Is the Grok-2 model supported?

| **Author** | `edyapd` |
| :--- | :--- |
| **State** | ✅ **Answered** |
| **Created** | 2025-09-09 |
| **Updated** | 2025-09-19 |

---

## 📄 Description

If Grok-2 is supported, how can I run it?
Currently, I'm getting the following error on load:
llama_model_load: error loading model: error loading model vocabulary: unknown pre-tokenizer type: 'grok-2'

---

## 💬 Discussion

👤 **firecoperana** commented on **2025-09-17** at **22:14:04**

https://github.com/ikawrakow/ik_llama.cpp/pull/782

It's supported now. You can build from this PR or wait after it's merged to main.

> 👤 **edyapd** replied on **2025-09-19** at **05:57:39**
> 
> Thank you so much.
> Yes, Grok-2 is working now. I tried it with the `grok-2-UD-Q2_K_XL` model.
> Unfortunately, I'm only able to load 45/65 layers onto my Tesla P40. As a result, the inference is very slow, around 0.9 tokens/second.

> 👤 **firecoperana** replied on **2025-09-19** at **17:11:51**
> 
> Yes, it's a very large model since it has at least 100b active parameters.