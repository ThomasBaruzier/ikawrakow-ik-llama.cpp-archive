## 📌 [Issue #725](https://github.com/ikawrakow/ik_llama.cpp/issues/725) - Bug: gpt-oss crash with context size >32k: GGML_ASSERT(nrc_x%8 == 0) failed

| **Author** | `bart2` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-08-25 |
| **Updated** | 2025-08-28 |

---

## 📄 Description

### What happened?

ik_llama.cpp arguments:
```
ik_llama.cpp/build/bin/llama-server --model /nvme/ggufs/gpt-oss-120b-Q8_0/gpt-oss-120b-Q8_0-00001-of-00002.gguf \
  -rtr \
  --ctx-size 128000 \
  -mla 3 \
  -fa \
  -b 4096 \
  -ub 4096 \
  -amb 512 \
  -fmoe \
  --temp 0.6 \
  --top_p 0.95 \
  --min_p 0.01 \
  --n-gpu-layers 999 \
  -ot exps=CPU \
  --parallel 1 \
  --threads 112 \
  --host 0.0.0.0 \
  --port 8080 \
  --threads-batch 112
```
Trying to use the model with `aider` for code refactor. Passing multiple source code files that cause the context size to go beyond 32k results in a following crash:
```
INFO [                    main] HTTP server listening | tid="140277929652224" timestamp=1756088976 n_threads_http="111" port="8080" hostname="0.0.0.0"
INFO [            update_slots] all slots are idle | tid="140277929652224" timestamp=1756088976
INFO [   launch_slot_with_task] slot is processing task | tid="140277929652224" timestamp=1756088991 id_slot=0 id_task=0
INFO [            update_slots] kv cache rm [p0, end) | tid="140277929652224" timestamp=1756088991 id_slot=0 id_task=0 p0=0
INFO [            update_slots] kv cache rm [p0, end) | tid="140277929652224" timestamp=1756089003 id_slot=0 id_task=0 p0=4096
INFO [            update_slots] kv cache rm [p0, end) | tid="140277929652224" timestamp=1756089014 id_slot=0 id_task=0 p0=8192
INFO [            update_slots] kv cache rm [p0, end) | tid="140277929652224" timestamp=1756089025 id_slot=0 id_task=0 p0=12288
INFO [            update_slots] kv cache rm [p0, end) | tid="140277929652224" timestamp=1756089037 id_slot=0 id_task=0 p0=16384
INFO [            update_slots] kv cache rm [p0, end) | tid="140277929652224" timestamp=1756089048 id_slot=0 id_task=0 p0=20480
INFO [            update_slots] kv cache rm [p0, end) | tid="140277929652224" timestamp=1756089060 id_slot=0 id_task=0 p0=24576
INFO [            update_slots] kv cache rm [p0, end) | tid="140277929652224" timestamp=1756089072 id_slot=0 id_task=0 p0=28672
INFO [            update_slots] kv cache rm [p0, end) | tid="140277929652224" timestamp=1756089084 id_slot=0 id_task=0 p0=32768
ik_llama.cpp/ggml/src/iqk/iqk_gemm_legacy_quants.cpp:1806: GGML_ASSERT(nrc_x%8 == 0) failed
ik_llama.cpp/ggml/src/iqk/iqk_gemm_legacy_quants.cpp:1806: GGML_ASSERT(nrc_x%8 == 0) failed
ik_llama.cpp/ggml/src/iqk/iqk_gemm_legacy_quants.cpp:1806: GGML_ASSERT(nrc_x%8 == 0) failed
```

### Name and Version

```
$ ik_llama.cpp/build/bin/llama-cli --version                                                                                                                                                                 (base)
version: 3858 (af13c9a2)
built with cc (Ubuntu 13.3.0-6ubuntu2~24.04) 13.3.0 for x86_64-linux-gnu
```

### What operating system are you seeing the problem on?

Linux

### Relevant log output

```shell

```

---

## 💬 Conversation

👤 **ikawrakow** commented on **2025-08-25** at **08:58:21**

So, there are similar reports elsewhere and I'm trying to understand the route cause.

But please do not use `-rtr` for a `Q8_0` model. This will repack the `Q8_0` tensors stored on the CPU (via `-ot`) to `Q8_0_R8`, so matrix multiplications with these will never get offloaded to the GPU(s) because the CUDA back-end does not support ` Q8_0_R8`. Which means that prompt processing of long prompts may be significantly slower than it could be.

---

👤 **ikawrakow** commented on **2025-08-27** at **15:28:16**

I think this should be solved on the latest main.

---

👤 **trilog-inc** commented on **2025-08-27** at **20:25:10**

Using it all day, can confirm the issue hasn't resurfaced

---

👤 **bart2** commented on **2025-08-28** at **15:48:41**

Can also confirm the crash is gone. Thanks for fixing it!