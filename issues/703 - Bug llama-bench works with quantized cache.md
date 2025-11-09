## 📌 [Issue #703](https://github.com/ikawrakow/ik_llama.cpp/issues/703) - Bug: llama-bench works with quantized cache?

| **Author** | `Ph0rk0z` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-08-18 |
| **Updated** | 2025-08-18 |

---

## 📄 Description

### What happened?

Every time I try to use -ctk q8_0 and -ctv q8_0 llama bench says invalid argument for the cache type. Same commands work in llama-sweep-bench, server, etc.

Also found there's a thp command but it appears thp are always used on linux.

### Name and Version

head

### What operating system are you seeing the problem on?

Linux

### Relevant log output

```shell

```

---

## 💬 Conversation

👤 **ikawrakow** commented on **2025-08-18** at **13:29:51**

```
./bin/llama-bench -m ../models/Llama-3.2-3B-Instruct-BF16.gguf -t 1 -ngl 100 -fa 1 -ctk q8_0 -ctv q8_0 -p 512 -n 0
```
| model                          |       size |     params | backend    | ngl | threads | type_k | type_v | fa |          test |              t/s |
| ------------------------------ | ---------: | ---------: | ---------- | --: | ------: | -----: | -----: | -: | ------------: | ---------------: |
| llama ?B BF16                  |   6.72 GiB |     3.61 B | CUDA       | 100 |       1 |   q8_0 |   q8_0 |  1 |         pp512 | 12003.60 ± 204.00 |

My guess is you are forgetting something. Not using flash attention (required for quantized V-cache except for DeepSeek with MLA), or perhaps forgetting that `llama-bench` requires `-fa 1`, not just `-fa` as in `llama-sweep-bench` (same for `-fmoe`, it needs to be `-fmoe 1`, etc.)

---

👤 **Ph0rk0z** commented on **2025-08-18** at **22:27:26**

You're right, I was missing fa 1 and fmoe 1. It never complained about those but did so about rtr.