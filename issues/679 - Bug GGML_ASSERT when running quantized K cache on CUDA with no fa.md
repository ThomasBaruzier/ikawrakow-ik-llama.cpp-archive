## 📌 [Issue #679](https://github.com/ikawrakow/ik_llama.cpp/issues/679) - Bug: GGML_ASSERT when running quantized K cache on CUDA with no fa

| **Author** | `saood06` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-08-08 |
| **Updated** | 2025-08-08 |

---

## 📄 Description

### What happened?

I managed to reproduce the bug that was mentioned in [#645](https://github.com/ikawrakow/ik_llama.cpp/issues/645) without a draft model at all. With my 3090 on Windows

`.\llama-server.exe -m "tinyllama-1.1b-chat-v1.0.Q4_K_M.gguf" -t 6 -ngl 99 -ctk q8_0`

### Name and Version

version: 3766 (cac763fc)
built with MSVC 19.28.29335.0 for x64

and 

version: 3852 (a694d7d0) AKA [#645](https://github.com/ikawrakow/ik_llama.cpp/issues/645)
built with MSVC 19.28.29335.0 for x64

### What operating system are you seeing the problem on?

Windows

### Relevant log output

```shell
ik_llama.cpp\ggml\src\ggml-cuda\mmvq.cu:595: GGML_ASSERT(src0->ne[2] == src1->ne[2] && src0->ne[2] == dst->ne[2]) failed
```

---

## 💬 Conversation

👤 **ikawrakow** commented on **2025-08-08** at **07:53:55**

Yes, something is broken there. I can fix the assert, but I'm still getting NaNs after that.

---

👤 **saood06** commented on **2025-08-08** at **08:02:48**

It definitely worked at some point (https://github.com/ikawrakow/ik_llama.cpp/pull/105), so you could maybe try bisecting it and find what broke it and see if that helps

---

👤 **ikawrakow** commented on **2025-08-08** at **10:43:13**

[#374](https://github.com/ikawrakow/ik_llama.cpp/issues/374) breaks it. Bisecting takes a really looong time with each step triggering a full CUDA rebuild.