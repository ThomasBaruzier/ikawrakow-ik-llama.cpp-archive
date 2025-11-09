## 📌 [Issue #694](https://github.com/ikawrakow/ik_llama.cpp/issues/694) - Bug: Generation breaks when it hits context length

| **Author** | `blueyred` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-08-15 |
| **Updated** | 2025-08-15 |

---

## 📄 Description

### What happened?

Set a small context size (512), start a generation when it gets to context shift the generation rapidly declines and goes into duplicating the last few words.

Issue doesn't happen (gemma-3-27b-it-qat-UD-Q4_K_XL.gguf) on mainline llama.cpp.

Entirely possible its something weird to do with my setup (motherboard is ASUS WS X299 PRO which needed some coaxing to see both GPUs)  


Tested on:
gemma-3-27b-it-qat-UD-Q4_K_XL.gguf (unsloth)
GLM-4.5-Air-IQ2_KL.gguf
GLM-4.5-Air-IQ1_KT.gguf
GLM-4.5-Air-IQ4_K-00001-of-00002.gguf (CPU offload)

Tested with and without tensor split (on Gemma) happens for both. Also tested with `--split-mode none`
Tested GLM-4.5-Air-IQ1_KT.gguf up to 32768 and was stable right until the generation hit the context length.

./build/bin/llama-server --model ~/data/Models/gemma-3-27b-it-qat-UD-Q4_K_XL.gguf --host 0.0.0.0 --port 5000 --ctx-size 512 -ngl 99 --tensor-split 2,1

```
INFO [            update_slots] slot context shift | tid="123609319292928" timestamp=1755269348 id_slot=0 id_task=0 n_keep=1 n_left=510 n_discard=255 n_ctx=512 n_past=511 n_system_tokens=0 n_cache_tokens=511
INFO [            update_slots] slot context shift | tid="123609319292928" timestamp=1755269355 id_slot=0 id_task=0 n_keep=1 n_left=510 n_discard=255 n_ctx=512 n_past=511 n_system_tokens=0 n_cache_tokens=511
```

NVIDIA-SMI 570.172.08             
Driver Version: 570.172.08     
CUDA Version: 12.8
NVIDIA RTX PRO 4500 + NVIDIA GeForce RTX 5060 Ti

### Name and Version

version: 3838 (fc06bc9d)
built with cc (Ubuntu 14.2.0-4ubuntu2~24.04) 14.2.0 for x86_64-linux-gnu

### What operating system are you seeing the problem on?

_No response_

### Relevant log output

```shell

```

---

## 💬 Conversation

👤 **blueyred** commented on **2025-08-15** at **17:26:15**

Just updated mainline (was possibly a week out of date) and I'm getting the same issue there now. Will investigate futher.

---

👤 **blueyred** commented on **2025-08-15** at **17:26:34**

Just updated mainline (was possibly a week out of date) and I'm getting the same issue there now. Will investigate futher.