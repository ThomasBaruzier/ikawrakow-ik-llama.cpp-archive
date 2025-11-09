## 📌 [Issue #761](https://github.com/ikawrakow/ik_llama.cpp/issues/761) - Support RWKV models in ik_llama.cpp

| **Author** | `xiyuan-code` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-09-05 |
| **Updated** | 2025-09-26 |
| **Labels** | `wontfix` |

---

## 📄 Description

### What happened?

ik_llama.cpp currently supports only LLaMA/Transformer models. RWKV is a hybrid architecture (RNN-style time-mixing + Transformer-style channel-mixing) that is not compatible with the current LLaMA inference.

Request:

Add RWKV forward pass support.

Allow a model-type option (--model-type llama / --model-type rwkv).

Ensure RWKV weights can be loaded correctly.

Keep existing LLaMA functionality unchanged.

References:

RWKV GitHub: [https://github.com/BlinkDL/RWKV-LM](https://github.com/BlinkDL/RWKV-LM?utm_source=chatgpt.com)

Labels:
enhancement, feature request, RWKV support

### Name and Version

./llama-server -m ./RWKV-v6-World-v3-7B-Q4_K_M.gguf -ngl 16 -c 2048 -t 12  --host 127.0.0.1 --port 8030 --reasoning-budget 0

### What operating system are you seeing the problem on?

_No response_

### Relevant log output

```shell

```

---

## 💬 Conversation

👤 **ikawrakow** commented on **2025-09-05** at **13:16:09**

What would be the advantage of having RWKV support in `ik_llama.cpp` instead of just using `llama.cpp`?