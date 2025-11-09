## 📌 [Issue #755](https://github.com/ikawrakow/ik_llama.cpp/issues/755) - Bug: attention max batch option doesn't work for GLM 4.5

| **Author** | `ghost` |
| :--- | :--- |
| **State** | ✅ **Open** |
| **Created** | 2025-09-03 |
| **Updated** | 2025-09-04 |

---

## 📄 Description

### What happened?

I'm not sure if it's a param exclusive to deepseek. So I disabled FA and got 7 GB compute buffers requested

```
llama_new_context_with_model: n_ctx      = 20000
llama_new_context_with_model: n_batch    = 1024
llama_new_context_with_model: n_ubatch   = 1024
llama_new_context_with_model: flash_attn = 0
llama_new_context_with_model: mla_attn   = 0
llama_new_context_with_model: attn_max_b = 512
llama_new_context_with_model: fused_moe  = 1
llama_new_context_with_model: fused_up_gate = 1
llama_new_context_with_model: ser        = -1, 0
llama_new_context_with_model: freq_base  = 1000000.0
llama_new_context_with_model: freq_scale = 1
llama_kv_cache_init:      CUDA0 KV buffer size =  1093.75 MiB
llama_kv_cache_init:      CUDA1 KV buffer size =   781.25 MiB
llama_kv_cache_init:      CUDA2 KV buffer size =   703.12 MiB
llama_kv_cache_init:      CUDA3 KV buffer size =   781.25 MiB
llama_kv_cache_init:      CUDA4 KV buffer size =   781.25 MiB
llama_kv_cache_init:      CUDA5 KV buffer size =   703.12 MiB
llama_kv_cache_init:      CUDA6 KV buffer size =   781.25 MiB
llama_kv_cache_init:      CUDA7 KV buffer size =   781.25 MiB
llama_kv_cache_init:      CUDA8 KV buffer size =   312.50 MiB
llama_kv_cache_init:      CUDA9 KV buffer size =   312.50 MiB
llama_kv_cache_init:     CUDA10 KV buffer size =   234.38 MiB
llama_new_context_with_model: KV self size  = 7265.62 MiB, K (f16): 3632.81 MiB, V (f16): 3632.81 MiB
llama_new_context_with_model:  CUDA_Host  output buffer size =     1.16 MiB
llama_new_context_with_model: pipeline parallelism enabled (n_copies=1)
ggml_backend_cuda_buffer_type_alloc_buffer: allocating 7734.13 MiB on device 0: cudaMalloc failed: out of memory
ggml_gallocr_reserve_n: failed to allocate CUDA0 buffer of size 8109821952
llama_new_context_with_model: failed to allocate compute buffers
```

### Name and Version

3433c7b56d11227c604f81b67b83824097d7923d

### What operating system are you seeing the problem on?

Linux

### Relevant log output

```shell

```

---

## 💬 Conversation

👤 **ikawrakow** commented on **2025-09-03** at **16:05:33**

`-amb` works for DeepSeek and models that do not use GQA. It is possible to implement also for GQA models when not using FA, but I don't really see the point of it.

Why would you want to turn off FA for GLM-4.5?

---

👤 **ghost** commented on **2025-09-03** at **18:22:15**

@ikawrakow I got more than 2x tg speed increase when disabled FA for DeepSeek in my setup. I don't know why. I tried to reproduce it on llama.cpp, but I couldn't run it without -amb feature.
I got +30 t/s pp improvement running ik_llama instead of llama.cpp for GLM 4.5 (130->160) with the same layer split and override tensors things, and now I want to try to disable FA in hope it will increase tg too

---

👤 **ikawrakow** commented on **2025-09-03** at **18:59:51**

> I got more than 2x tg speed increase when disabled FA for DeepSeek

This is unexpected. What is your setup and what are your DeepSeek commands that result in a 2X difference in TG speed?

---

👤 **ghost** commented on **2025-09-04** at **20:10:13**

4x3090,2x2080Ti@22,2x3060,3070ti

`./llama-server -m "/<...>/DeepSeek-V3.1-GGUF/UD-Q2_K_XL/DeepSeek-V3.1-UD-Q2_K_XL-00001-of-00006.gguf" -ts 33,5,4,5,5,4,2,2,2 -sm layer -c 15000 -b 1024 -ub 1024 -ngl 62 -ncmoe 30 -t 7 -mla 3 -fmoe -amb 512 --no-mmap`

My usual setup also includes 2xP40, but unfortunately I can't run ik_llama with -fa using them. So I decided to exclude them and offload tensors on RAM (command above, but adding -fa). I got drastically worse results. Then, being curious, I disabled FA and it worked almost as good as with teslas in setup.

If ik_llama has profiler of some sort and you are curious why it happens, I can run it if you give me instructions how to launch it.