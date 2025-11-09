## 📌 [Issue #909](https://github.com/ikawrakow/ik_llama.cpp/issues/909) - Bug: performance regression for GLM?

| **Author** | `magikRUKKOLA` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-11-06 |
| **Updated** | 2025-11-07 |

---

## 📄 Description

### What happened?

I have a bench.log from Oct 30th with a 8k batches test on my test rig of 3945wx.  Here are the results:

```
main: n_kv_max = 73728, n_batch = 8192, n_ubatch = 8192, flash_attn = 1, n_gpu_layers = 99, n_threads = 12, n_threads_batch = 12

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  8192 |   2048 |      0 |   22.750 |   360.09 |  371.612 |     5.51 |
|  8192 |   2048 |   8192 |   27.226 |   300.89 |  398.527 |     5.14 |
|  8192 |   2048 |  16384 |   32.145 |   254.85 |  424.979 |     4.82 |
```

I pulled the latest master today and I saw some worse results.  Initially I realized I am getting CPU throttled because one of two of my CPU AIO didn't work but it turned out that even w/o throttling the situation remains the same.

```
llama_new_context_with_model: n_ctx         = 73728
llama_new_context_with_model: n_batch       = 8192
llama_new_context_with_model: n_ubatch      = 8192
llama_new_context_with_model: flash_attn    = 1
llama_new_context_with_model: mla_attn      = 0
llama_new_context_with_model: attn_max_b    = 512
llama_new_context_with_model: fused_moe     = 1
llama_new_context_with_model: grouped er    = 0
llama_new_context_with_model: fused_up_gate = 1
llama_new_context_with_model: fused_mmad    = 1
llama_new_context_with_model: rope_cache    = 0
llama_new_context_with_model: ser           = -1, 0
llama_new_context_with_model: freq_base     = 1000000.0
llama_new_context_with_model: freq_scale    = 1
llama_kv_cache_init:      CUDA0 KV buffer size =  9922.52 MiB
llama_kv_cache_init:      CUDA1 KV buffer size = 10363.52 MiB
llama_new_context_with_model: KV self size  = 20286.00 MiB, K (q8_0): 7038.00 MiB, V (f16): 13248.00 MiB
llama_new_context_with_model:  CUDA_Host  output buffer size =     0.58 MiB
llama_new_context_with_model:      CUDA0 compute buffer size =  4796.91 MiB
llama_new_context_with_model:      CUDA1 compute buffer size =  4896.00 MiB
llama_new_context_with_model:  CUDA_Host compute buffer size =  1312.09 MiB
llama_new_context_with_model: graph nodes  = 3854
llama_new_context_with_model: graph splits = 228
XXXXXXXXXXXXXXXXXXXXX Setting only active experts offload

main: n_kv_max = 73728, n_batch = 8192, n_ubatch = 8192, flash_attn = 1, n_gpu_layers = 99, n_threads = 12, n_threads_batch = 12

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  8192 |   2048 |      0 |   26.007 |   314.99 |  397.382 |     5.15 |
|  8192 |   2048 |   8192 |   30.775 |   266.19 |  422.269 |     4.85 |
|  8192 |   2048 |  16384 |   35.460 |   231.02 |  439.076 |     4.66 |
```

### Name and Version

/opt/ik_llama.cpp/ik_llama.cpp/build/bin/llama-server  --version
version: 3958 (575e2c282)
built with cc (Debian 15.2.0-4) 15.2.0 for x86_64-linux-gnu

### What operating system are you seeing the problem on?

Linux

### Relevant log output

```
nvoc -R
Fetched thresholds for sensor ID 18: Nominal: 48.0°C, Warn: 70.0°C, Crit: 85.0°C
Fetched thresholds for sensor ID 23: Nominal: 48.0°C, Warn: 70.0°C, Crit: 85.0°C
Fetched thresholds for sensor ID 21: Nominal: 48.0°C, Warn: 70.0°C, Crit: 85.0°C
Detected 2 NVIDIA GPU(s)

NVOC System Report
==================
System: Linux xxx 6.16.9+deb14-amd64 x86_64
Driver Version: 580.82.09

System Temperatures:
  CPU: 68.0°C [OK] (Nominal: 48.0°C, Warn: 70.0°C, Crit: 85.0°C)
  RAM: 46.0°C [OK] (Nominal: 48.0°C, Warn: 70.0°C, Crit: 85.0°C)
  VR:  41.0°C [OK] (Nominal: 48.0°C, Warn: 70.0°C, Crit: 85.0°C)

GPU 0: NVIDIA GeForce RTX 3090
------------------------------------------------
  PCI Bus ID:        00000000:01:00.0
  VBIOS Version:     94.02.4B.00.0B
  Persistence Mode:  Disabled
  Core Temperature:  59°C
  Power Usage:       181W
  Current Power Limit: 400W
  Power Limits:      Default: 350W, Min: 100W, Max: 400W
  GPU Clock:         2010 MHz
  VRAM Clock:        10251 MHz
  GPU Utilization:   5%
  VRAM Utilization:  5%
  Memory Usage:      22.2 / 24.0 GB
  Applied Offsets:   GPU: 100 MHz, VRAM: 1500 MHz

GPU 1: NVIDIA GeForce RTX 3090
------------------------------------------------
  PCI Bus ID:        00000000:02:00.0
  VBIOS Version:     94.02.4B.00.0B
  Persistence Mode:  Disabled
  Core Temperature:  55°C
  Power Usage:       187W
  Current Power Limit: 400W
  Power Limits:      Default: 350W, Min: 100W, Max: 400W
  GPU Clock:         2040 MHz
  VRAM Clock:        10251 MHz
  GPU Utilization:   1%
  VRAM Utilization:  1%
  Memory Usage:      22.8 / 24.0 GB
  Applied Offsets:   GPU: 100 MHz, VRAM: 1500 MHz


Peer-to-Peer (P2P) Support Matrix:
=================================
GPU 0 -> GPU 1: Supported
GPU 1 -> GPU 0: Supported

Report generated at: Thu Nov  6 11:19:54 2025
==================```

---

## 💬 Conversation

👤 **ikawrakow** commented on **2025-11-06** at **13:07:31**

So, CUDA fusion has been disabled by default because of [#893](https://github.com/ikawrakow/ik_llama.cpp/issues/893), [#895](https://github.com/ikawrakow/ik_llama.cpp/issues/895).

Try rebuilding with
```
cmake -DGGML_CUDA_FUSION=1 $other_args
```

Does this fix the performance drop?

---

👤 **magikRUKKOLA** commented on **2025-11-06** at **19:15:52**

@ikawrakow 
> Does this fix the performance drop?

Unfortunately I will not be able to tell for the 3945wx.  Because I just upgraded the test rig for 3975wx.

What I can do is to check what is the difference for the new CPU.  I will drop the results below.

---

👤 **magikRUKKOLA** commented on **2025-11-06** at **19:40:55**

@ikawrakow 

Okay so the 3975wx w/o -DGGML_CUDA_FUSION=1:

```
 |    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
 |-------|--------|--------|----------|----------|----------|----------|
 |  8192 |   2048 |      0 |   25.710 |   318.63 |  257.397 |     7.96 |
 |  8192 |   2048 |   8192 |   30.548 |   268.17 |  283.907 |     7.21 |
 |  8192 |   2048 |  16384 |   35.045 |   233.75 |  307.970 |     6.65 |
```

with  -DGGML_CUDA_FUSION=1:
```
main: n_kv_max = 73728, n_batch = 8192, n_ubatch = 8192, flash_attn = 1, n_gpu_layers = 99, n_threads = 32, n_threads_batch = 32

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  8192 |   2048 |      0 |   25.755 |   318.08 |  237.353 |     8.63 |
|  8192 |   2048 |   8192 |   30.513 |   268.48 |  263.626 |     7.77 |
...
```

~~Well, the result it pretty clear -- it works better with CUDA_FUSION=1.  Did I miss some args?  Should I compile with CUDA_FUSION=1 from now on?~~

Ah... disabled by default.  Oke.

---

👤 **magikRUKKOLA** commented on **2025-11-06** at **19:58:36**

@ikawrakow 

Should I do the PPL scores in both cases to see if CUDA_FUSION=1 indeed causes the problem?  Or the llama-perplexity will not catch it?

[EDIT]:

I see no difference in PPL with or without the fustion.

Its 3.4532 (CUDA_FUSION=0) vs 3.4490 (CUDA_FUSION=1).

---

👤 **ikawrakow** commented on **2025-11-07** at **05:10:27**

@magikRUKKOLA 

Yes, fusion is supposed to not change the result. But people have reported crashes with CUDA illegal memory access, and also garbled output. I cannot reproduce with my setup, so not possible to debug. But if it works for you, then you just use it. If you don't want to add `-cuda fusion=1` to your command line each time, you can rebuild with `cmake -DGGML_CUDA_FUSION=1` and then it will be ON by default.