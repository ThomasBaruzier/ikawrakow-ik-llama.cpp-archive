## 📌 [Issue #731](https://github.com/ikawrakow/ik_llama.cpp/issues/731) - Bug: (Speculative decoding) llama-server hangs but doesn't crash when draft's KV allocation fails (draft KV OOM)

| **Author** | `alint77` |
| :--- | :--- |
| **State** | ✅ **Open** |
| **Created** | 2025-08-26 |
| **Updated** | 2025-08-26 |

---

## 📄 Description

### What happened?

llama-server just won't produce anything and hangs when the draft model's KV cache CUDA memory allocation fails (OOM) whereas in mainline llama.cpp it crashes properly. 

on rtx 5090 32GB: 

./llama-server --api-key 1 -a qwen3 -m ~/Qwen3-Coder-30B-A3B-Instruct-IQ4_NL.gguf -ngl 99 -c 50000 -md Qwen3-0.6B-Q8_0.gguf -ngld 99  -fa -fmoe

this will lead to hang when sending a request, but -c 30000 fits and has no issues. 

### Name and Version

./llama-server --version
version: 3860 (0cc32ff0)
built with cc (Ubuntu 13.3.0-6ubuntu2~24.04) 13.3.0 for x86_64-linux-gnu

### What operating system are you seeing the problem on?

Linux

### Relevant log output

```shell

```