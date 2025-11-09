## 📌 [Issue #687](https://github.com/ikawrakow/ik_llama.cpp/issues/687) - Bug: Surprising CUDA OOM during PP

| **Author** | `usrlocalben` |
| :--- | :--- |
| **State** | ✅ **Open** |
| **Created** | 2025-08-12 |
| **Updated** | 2025-09-26 |
| **Labels** | `wontfix` |

---

## 📄 Description

### What happened?

Crash during PP due to CUDA OOM.

What is being allocated during PP phase? Presumably, all necessary buffers were allocated at startup. Despite tuning params and reaching a running state, the system may still OOM.

I took measurements (nvidia-smi) along the way and can conclude that _whatever this is_ is growing linearly, and is about 34KiB/token (model is Kimi K2). It is larger with DeepSeek, so it could be related to the attention-heads.

Assuming that it is not a leak or similar defect, then I'd imagine the situation could be improved.
a) llama-server could fail to start since there aren't resources available to support the configured context, or
b) when the task starts, isn't there already enough information to know the request can't be filled? 

Alternatively, if this is intended behavior, what is being allocated at startup w/ `-c` vs. what is happening during processing? The startup failure wrt. `-c N` and VRAM usage give the impression that all CUDA allocations happen at startup. A given configuration is assumed valid by the fact that it starts. How can one compute the real VRAM req. and avoid a crash?

full setup:
```
-np 1
--temp 0.6 --top-p 0.95
-mla 3 -fa -fmoe
-b 4096 -ub 4096
-ctk f16 -c 128000 -np 1
-op 26,0,27,0,29,0
-m intel/Q2_K_S/DeepSeek-R1-0528-hf-256x20B-Q2_K_S-00001-of-00005.gguf
```

### Name and Version

version: 3835 (d60c8f4d)
built with cc (Debian 12.2.0-14+deb12u1) 12.2.0 for x86_64-linux-gnu

### What operating system are you seeing the problem on?

Linux

### Relevant log output

```shell
INFO [            update_slots] kv cache rm [p0, end) | tid="140612628107264" timestamp=1754840413 id_slot=0 id_task=3 p0=0
INFO [            update_slots] kv cache rm [p0, end) | tid="140612628107264" timestamp=1754840452 id_slot=0 id_task=3 p0=4096
INFO [            update_slots] kv cache rm [p0, end) | tid="140612628107264" timestamp=1754840492 id_slot=0 id_task=3 p0=8192
INFO [            update_slots] kv cache rm [p0, end) | tid="140612628107264" timestamp=1754840534 id_slot=0 id_task=3 p0=12288
INFO [            update_slots] kv cache rm [p0, end) | tid="140612628107264" timestamp=1754840578 id_slot=0 id_task=3 p0=16384
INFO [            update_slots] kv cache rm [p0, end) | tid="140612628107264" timestamp=1754840625 id_slot=0 id_task=3 p0=20480
INFO [            update_slots] kv cache rm [p0, end) | tid="140612628107264" timestamp=1754840673 id_slot=0 id_task=3 p0=24576
INFO [            update_slots] kv cache rm [p0, end) | tid="140612628107264" timestamp=1754840724 id_slot=0 id_task=3 p0=28672
INFO [            update_slots] kv cache rm [p0, end) | tid="140612628107264" timestamp=1754840777 id_slot=0 id_task=3 p0=32768
INFO [            update_slots] kv cache rm [p0, end) | tid="140612628107264" timestamp=1754840832 id_slot=0 id_task=3 p0=36864
INFO [            update_slots] kv cache rm [p0, end) | tid="140612628107264" timestamp=1754840890 id_slot=0 id_task=3 p0=40960
INFO [            update_slots] kv cache rm [p0, end) | tid="140612628107264" timestamp=1754840950 id_slot=0 id_task=3 p0=45056
INFO [            update_slots] kv cache rm [p0, end) | tid="140612628107264" timestamp=1754841012 id_slot=0 id_task=3 p0=49152
INFO [            update_slots] kv cache rm [p0, end) | tid="140612628107264" timestamp=1754841077 id_slot=0 id_task=3 p0=53248
INFO [            update_slots] kv cache rm [p0, end) | tid="140612628107264" timestamp=1754841144 id_slot=0 id_task=3 p0=57344
CUDA error: out of memory
  current device: 0, in function alloc at /path/to/ik_llama.cpp/ggml/src/ggml-cuda.cu:384
  cuMemCreate(&handle, reserve_size, &prop, 0)
/path/to/ik_llama.cpp/ggml/src/ggml-cuda.cu:110: CUDA error
```

---

## 💬 Conversation

👤 **ikawrakow** commented on **2025-08-12** at **05:12:48**

When loading the model, a "worst case" graph is created and the memory required to compute the graph is calculated. Compute buffers are allocated based on that, and are supposed to not require additional allocations later. Occasionally it happens that we still get OOM at run time. Happens in mainline `llama.cpp` too (I just ran into it yesterday while benchmarking GPT-OSS). This would indicate that the "worst case" graph is not really worst case, but I don't understand why.

---

👤 **ubergarm** commented on **2025-08-12** at **16:59:58**

I've had this happen sometimes too specifically when increasing batch sizes. I don't recall it happening with the default batch sizes specified. As such I always try to leave a little extra VRAM and not cram it totally full to avoid potential OOM at the end of my llama-sweep-bench etc.

Also I assume you're compiling with `-DGGML_SCHED_MAX_COPIES=1`, probably not related here as this issue tends to OOM on startup allocating buffers.

---

👤 **saood06** commented on **2025-08-13** at **05:04:19**

I think I experience this bug and I don't think it is exclusive to CUDA because there is still some memory loading after warmup which happens when higher and higher context is used, but it stays loaded until I drop the cache (or cache is disturbed). 

This can be observed by running a clean (AKA no memory load happened) sweep bench to a certain context, and then increase the context and it will no longer be "clean" once testing hits the context not covered by the clean sweep bench.

---

👤 **Ph0rk0z** commented on **2025-08-14** at **12:16:42**

In sweep bench I experience memory growth as the context increases. Something like 4mb per iteration on each GPU. I have learned to account for it. Models may load with 32k ctx and fail at 27k ctx if they're right at the edge. Some models load, create buffers and then OOM right away upon trying inference. Tensors have to be dialed back.

There is a bigger initial jump at the first processing/gen and then the slow steady rise.