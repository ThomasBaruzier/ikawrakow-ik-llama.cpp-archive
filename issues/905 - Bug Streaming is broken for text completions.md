## 📌 [Issue #905](https://github.com/ikawrakow/ik_llama.cpp/issues/905) - Bug: Streaming is broken for text completions.

| **Author** | `Ph0rk0z` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-11-05 |
| **Updated** | 2025-11-06 |

---

## 📄 Description

### What happened?

I pulled all the changes from last few days and reloaded R1. Client just sits long after I see timings in the console and CPU/GPU activity stop. No text ever appears.

Decided to test chat completions and streaming works there. Turned off streaming in text completions and text appears. Streaming back on and it waits forever again.

Haven't narrowed down the commit but you can likely see for yourself if you give it a go. In SillyTavern it's definitely busted.

### Name and Version

HEAD

### What operating system are you seeing the problem on?

Linux

### Relevant log output

```shell

```

---

## 💬 Conversation

👤 **firecoperana** commented on **2025-11-05** at **17:31:08**

Can you try https://github.com/ikawrakow/ik_llama.cpp/pull/906?

---

👤 **MrHills-2** commented on **2025-11-06** at **10:22:57**

This is fixed for me in 575e2c2