## 📌 [Issue #756](https://github.com/ikawrakow/ik_llama.cpp/issues/756) - Bug: " Tool calls support from mainline" patch causes VRAM overflow for models that worked before

| **Author** | `Lissanro` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-09-04 |
| **Updated** | 2025-09-20 |

---

## 📄 Description

### What happened?

I no longer can load Kimi K2 after "Tool calls support from mainline" (0f9ecaec04cf40dd7524fdc6625f43c13b8038f8) patch:

```
ggml_backend_cuda_buffer_type_alloc_buffer: allocating 44063.76 MiB on device 0: cudaMalloc failed: out of memory
llama_model_load: error loading model: unable to allocate backend buffer
llama_load_model_from_file: failed to load model
llama_init_from_gpt_params: error: failed to load model '/mnt/neuro/models/Kimi-K2-Instruct/Kimi-K2-Instruct-IQ4_XS.gguf'
 ERR [              load_model] unable to load model | tid="131404752666624" timestamp=1756967078 model="/mnt/neuro/models/Kimi-K2-Instruct/Kimi-K2-Instruct-IQ4_XS.gguf"
free(): invalid pointer
```

I use this command:

```
numactl --cpunodebind=0 --interleave=all /home/lissanro/pkgs/ik_llama.cpp/build/bin/llama-server \
--model /mnt/neuro/models/Kimi-K2-Instruct/Kimi-K2-Instruct-IQ4_XS.gguf \
--ctx-size 131072 --n-gpu-layers 62 --tensor-split 15,25,30,30 -mla 3 -fa -ctk q8_0 -amb 512 -fmoe -b 4096 -ub 4096 \
-ot "blk\.3\.ffn_up_exps=CUDA0, blk\.3\.ffn_gate_exps=CUDA0, blk\.3\.ffn_down_exps=CUDA0" \
-ot "blk\.4\.ffn_up_exps=CUDA1, blk\.4\.ffn_gate_exps=CUDA1, blk\.4\.ffn_down_exps=CUDA1" \
-ot "blk\.5\.ffn_up_exps=CUDA2, blk\.5\.ffn_gate_exps=CUDA2, blk\.5\.ffn_down_exps=CUDA2" \
-ot "blk\.6\.ffn_up_exps=CUDA3, blk\.6\.ffn_gate_exps=CUDA3, blk\.6\.ffn_down_exps=CUDA3" \
-ot "ffn_down_exps=CPU, ffn_up_exps=CPU, gate_exps=CPU" \
--threads 64 --host 0.0.0.0 --port 5000 \
--slot-save-path /var/cache/ik_llama.cpp/k2
```

Reverting the patch 0f9ecaec04cf40dd7524fdc6625f43c13b8038f8 helps to fix this issue. Additionally to reverting it, I also had to add this in src/llama-vocab.cpp to prevent a minor compile error due to some missing logging defines:

```
LLAMA_ATTRIBUTE_FORMAT(2, 3)
void llama_log_internal        (ggml_log_level level, const char * format, ...);
void llama_log_callback_default(ggml_log_level level, const char * text, void * user_data);

#define LLAMA_LOG_INFO(...)  llama_log_internal(GGML_LOG_LEVEL_INFO , __VA_ARGS__)
#define LLAMA_LOG_DEBUG(...)  llama_log_internal(GGML_LOG_LEVEL_DEBUG , __VA_ARGS__)
#define LLAMA_LOG_WARN(...)  llama_log_internal(GGML_LOG_LEVEL_WARN , __VA_ARGS__)
#define LLAMA_LOG_ERROR(...) llama_log_internal(GGML_LOG_LEVEL_ERROR, __VA_ARGS__)
```

### Name and Version

latest git

### What operating system are you seeing the problem on?

Linux

### Relevant log output

```shell

```

---

## 💬 Conversation

👤 **ikawrakow** commented on **2025-09-04** at **07:07:16**

Can you give us the KV cache and compute buffer sizes in both cases? Thanks.

---

👤 **Lissanro** commented on **2025-09-04** at **07:25:35**

When working well (the patch is reversed):

```
llm_load_print_meta: model ftype      = IQ4_XS - 4.25 bpw
llm_load_print_meta: model params     = 1.027 T
llm_load_print_meta: model size       = 509.963 GiB (4.266 BPW) 
llm_load_print_meta: repeating layers = 508.485 GiB (4.263 BPW, 1024.571 B parameters)
...
llm_load_tensors: offloading 61 repeating layers to GPU
llm_load_tensors: offloading non-repeating layers to GPU
llm_load_tensors: offloaded 62/62 layers to GPU
llm_load_tensors:        CPU buffer size = 521227.74 MiB
llm_load_tensors:        CPU buffer size =   595.00 MiB
llm_load_tensors:      CUDA0 buffer size =  9791.76 MiB
llm_load_tensors:      CUDA1 buffer size = 10151.64 MiB
llm_load_tensors:      CUDA2 buffer size = 10573.95 MiB
llm_load_tensors:      CUDA3 buffer size = 11281.57 MiB
....................................................................................................
llama_new_context_with_model: n_ctx      = 131072
llama_new_context_with_model: n_batch    = 4096
llama_new_context_with_model: n_ubatch   = 4096
llama_new_context_with_model: flash_attn = 1
llama_new_context_with_model: mla_attn   = 3
llama_new_context_with_model: attn_max_b = 512
llama_new_context_with_model: fused_moe  = 1
llama_new_context_with_model: fused_up_gate = 1
llama_new_context_with_model: ser        = -1, 0
llama_new_context_with_model: freq_base  = 50000.0
llama_new_context_with_model: freq_scale = 0.03125
llama_kv_cache_init:      CUDA0 KV buffer size =   765.00 MiB
llama_kv_cache_init:      CUDA1 KV buffer size =  1147.51 MiB
llama_kv_cache_init:      CUDA2 KV buffer size =  1453.51 MiB
llama_kv_cache_init:      CUDA3 KV buffer size =  1300.51 MiB
llama_new_context_with_model: KV self size  = 4666.50 MiB, c^KV (q8_0): 4666.50 MiB, kv^T: not used
llama_new_context_with_model:  CUDA_Host  output buffer size =     1.25 MiB
llama_new_context_with_model: pipeline parallelism enabled (n_copies=1)
llama_new_context_with_model:      CUDA0 compute buffer size =  9032.02 MiB
llama_new_context_with_model:      CUDA1 compute buffer size =  7376.02 MiB
llama_new_context_with_model:      CUDA2 compute buffer size =  7376.02 MiB
llama_new_context_with_model:      CUDA3 compute buffer size =  7376.03 MiB
llama_new_context_with_model:  CUDA_Host compute buffer size =  2160.05 MiB
llama_new_context_with_model: graph nodes  = 13529
llama_new_context_with_model: graph splits = 174
```

When I try with the "Tool calls support from mainline" patch applied, the model size is the same, but it does not show further information and just crashes instead. In case I missed some important information, here is the full log with the crash: https://pastebin.com/4J6vy8Tq

When it works normally (with the patch reversed), I have 21845MiB on CUDA0 device, so "44063.76 MiB on device 0" when the patch is applied suggests something is wrong, since it is over the double the normal size.

If you would like me to run a specific command to test something or obtain more debugging information, please let me know.

---

👤 **ikawrakow** commented on **2025-09-04** at **08:02:24**

The `pastebin` thing is absolutely horrible, opening maximized browser windows with advertisements on each interaction. You can simply redirect the server output to a file, zip it, and drop it into the comment.

But yes, the output of the crashing invocation is not useful. What would be useful is to start the server with a smaller context (say, 32k) and smaller u-batch (say, 2048), so that it hopefully does not crash due to OOM. Once with the version that crashes, and once with the version that does not. Then post the two logs here.

---

👤 **Lissanro** commented on **2025-09-04** at **08:29:58**

Sorry, I wasn't aware pastbin has any ads (I guess I never saw them due to using adblocker). Thanks for letting me know. Will make sure to just attach text files in the future!

As of running with smaller batch size, I tried to go down to `-b 512 -ub 512` and `--ctx-size 1024 `and still get the same error: `allocating 44063.76 MiB on device 0: cudaMalloc failed: out of memory`. I also tried remove `-fa` option, but did not help either (based on the command I have shown in the first message).

At the moment it seems "Tool calls support from mainline" (https://github.com/ikawrakow/ik_llama.cpp/commit/0f9ecaec04cf40dd7524fdc6625f43c13b8038f8) broke loading of Kimi K2. If I try to load DeepSeek V3.1, I get similar error, so all DeepSeek-based models may be affected.

Please let me know if there is something else I could try to run to get some useful debug output.

---

👤 **ikawrakow** commented on **2025-09-04** at **08:47:29**

Are you building using `cmake` or some custom script? Have you tried removing the build folder and building from scratch?

---

👤 **Lissanro** commented on **2025-09-04** at **10:18:09**

This is how I build it, if there are some other build options I should be using, please let me know:

```
cd ~/pkgs/ik_llama.cpp && git pull && \
cd ~/pkgs && cmake ik_llama.cpp -B ik_llama.cpp/build \
-DGGML_CUDA_FA_ALL_QUANTS=ON -DBUILD_SHARED_LIBS=OFF \
-DGGML_CUDA=ON -DLLAMA_CURL=ON -DGGML_SCHED_MAX_COPIES=1 && \
cmake --build ik_llama.cpp/build --config Release -j --clean-first \
--target llama-quantize llama-cli llama-server llama-imatrix llama-perplexity && cd -
```

I tried to remove "build" folder and run make clean, then rebuild from scratch, but still got `ggml_backend_cuda_buffer_type_alloc_buffer: allocating 44063.76 MiB on device 0: cudaMalloc failed: out of memory`. Only reverting (https://github.com/ikawrakow/ik_llama.cpp/commit/0f9ecaec04cf40dd7524fdc6625f43c13b8038f8) helped. It is quite large patch unfortunately, so it is a bit difficult to pin point what part of it causes the issue.

---

👤 **ikawrakow** commented on **2025-09-04** at **10:51:59**

I simply don't see how PR [#723](https://github.com/ikawrakow/ik_llama.cpp/issues/723) could have affected model loading and compute graph inference. Also, I think at least  @magikRUKKOLA is successfully using DeepSeek-R1/V3.1 with the [#723](https://github.com/ikawrakow/ik_llama.cpp/issues/723) changes. 

To me it looks like the `--tensor-split` argument is not being honored for some reason, and all the tensors + KV caches go to `CUDA0`.

---

👤 **Lissanro** commented on **2025-09-04** at **11:10:54**

I see, in my case I have four GPUs, hence why tensor-split is necessary in my case. I checked multiple times that it is this specific patch that causes the problem, running git bisect confirmed it. After clean rebuild with unpatched ik_llama.cpp, the issue returns, as soon as I revert it and rebuild, it is solved.

I will see if I can pinpoint better what part of the patch causes this.

---

👤 **firecoperana** commented on **2025-09-04** at **12:18:32**

In your log file, I see Tensor blk.3.ffn_down_exps.weight buffer type overriden to CPU, but you have it overridden to CUDA0 in your command line. This does not look right to me. 
Can you add GGML_CUDA_ENABLE_UNIFIED_MEMORY=1 in your command line? This allows swapping to system RAM instead of crashing when the GPU VRAM is exhausted, so we can see how weights are actually distributed.

---

👤 **Lissanro** commented on **2025-09-15** at **23:19:53**

I really wanted to try to find a solution myself, but due to patch size that broke multi-GPU support, I had no luck so far. The code base keeps evolving so it is no longer possible to reverse https://github.com/ikawrakow/ik_llama.cpp/commit/0f9ecaec04cf40dd7524fdc6625f43c13b8038f8 and I cannot update ik_llama.cpp anymore because of it unfortunately.

About overrides, I tried only `-ot "ffn_down_exps=CPU, ffn_up_exps=CPU, gate_exps=CPU" ` and it crashed the same way. The other overrides I use to put specific layers to GPUs for more efficient VRAM utilization.

`GGML_CUDA_ENABLE_UNIFIED_MEMORY=1` - running with this option still leads to a crash:

```
llm_load_tensors: offloading 61 repeating layers to GPU
llm_load_tensors: offloading non-repeating layers to GPU
llm_load_tensors: offloaded 62/62 layers to GPU
llm_load_tensors:        CPU buffer size = 337618.67 MiB
llm_load_tensors:        CPU buffer size =   469.49 MiB
llm_load_tensors:      CUDA0 buffer size = 28820.63 MiB
llm_load_tensors:      CUDA1 buffer size = 59595.20 MiB
llm_load_tensors:      CUDA2 buffer size = 75487.25 MiB
llm_load_tensors:      CUDA3 buffer size = 68266.20 MiB
...............................................CUDA error: an illegal memory access was encountered
  current device: 1, in function ggml_backend_cuda_buffer_set_tensor at /home/lissanro/pkgs/ik_llama.cpp/ggml/src/ggml-cuda.cu:550
  cudaMemcpyAsync((char *)tensor->data + offset, data, size, cudaMemcpyHostToDevice, ((cudaStream_t)0x2))
/home/lissanro/pkgs/ik_llama.cpp/ggml/src/ggml-cuda.cu:116: CUDA error
```

This is a different error from "cudaMalloc failed: out of memory" but not sure if it means anything besides running out of VRAM.

So far, I narrowed down the issue to the following. First, go one patch before https://github.com/ikawrakow/ik_llama.cpp/commit/0f9ecaec04cf40dd7524fdc6625f43c13b8038f8:

git checkout b66cecca4525550fddf609bb875e58a4fad5f0ce

Then, I tried to minimize the problematic patch to the minimum that still compiles, but ended up with still very large partial patch (https://dragon.studio/2025/09/0f9ecae-partial.diff):

> patch -p1 < 0f9ecae-partial.diff 
patching file CMakeLists.txt
patching file common/CMakeLists.txt
patching file common/chat-parser.cpp
patching file common/chat-parser.h
patching file common/chat-template.hpp
patching file common/chat.cpp
patching file common/chat.h
patching file common/common.cpp
patching file common/common.h
patching file common/grammar-parser.cpp
patching file common/grammar-parser.h
patching file common/json-partial.cpp
patching file common/json-partial.h
patching file common/json-schema-to-grammar.cpp
patching file common/json-schema-to-grammar.h
patching file common/regex-partial.cpp
patching file common/regex-partial.h
patching file common/sampling.cpp
patching file common/sampling.h
patching file common/speculative.cpp
patching file examples/main/main.cpp
patching file examples/server/server.cpp
patching file examples/server/utils.hpp
patching file include/llama.h
patching file src/llama-grammar.cpp
patching file src/llama-grammar.h
patching file src/llama-impl.h
patching file src/llama-sampling.cpp
patching file src/llama-vocab.cpp
patching file src/llama-vocab.h

At this point, I am not really sure how to narrow it down further, so I would appreciate any ideas or help. If I reverse some of these files separately, it no longer builds, making it hard find the root cause. If I am doing something wrong and the way multi-GPU is supposed to be used changed, please let me know.

For reference, if I remove per-layer overrides for simplicity, my command looks like this:

```
~/pkgs/ik_llama.cpp/build/bin/llama-server \
--model /mnt/neuro/models/Kimi-K2-Instruct-IQ4_XS.gguf \
--ctx-size 131072 --n-gpu-layers 62 --tensor-split 15,25,30,30 -mla 3 -fa -ctk q8_0 -amb 512 -fmoe -b 4096 -ub 4096 \
-ot "ffn_down_exps=CPU, ffn_up_exps=CPU, gate_exps=CPU" \
--threads 64 --host 0.0.0.0 --port 5000
```

---

👤 **firecoperana** commented on **2025-09-16** at **02:04:42**

When you just use `-ot exps=CPU` or `--cpu-moe`, how much buffer does each GPU get? `-ot "ffn_down_exps=CPU, ffn_up_exps=CPU, gate_exps=CPU"` may not work as expected because you have space in it.

---

👤 **Lissanro** commented on **2025-09-16** at **20:22:41**

You are right. Removing spaces resolved the issue. So, it turns out that https://github.com/ikawrakow/ik_llama.cpp/commit/0f9ecaec04cf40dd7524fdc6625f43c13b8038f8 just removed support for spaces in `-ot` options... not sure if it intended, but now that I know about it, not really an issue for me.

Thank you very much, even though it may seem simple, it never occured to me that spaces that worked for months may stop working, so I was assuming it was something else and hence why could not find a solution for so long. Thanks again, really appreciate the help. I think I can finally close this issue as solved.

---

👤 **saood06** commented on **2025-09-20** at **01:22:45**

>You are right. Removing spaces resolved the issue. So, it turns out that https://github.com/ikawrakow/ik_llama.cpp/commit/0f9ecaec04cf40dd7524fdc6625f43c13b8038f8 just removed support for spaces in -ot options... not sure if it intended, but now that I know about it, not really an issue for me.

Similarly it also changed the default of the slots endpoint (from enabled by default to disabled by default) which may have been a good thing (provide some privacy when sharing an endpoint among people), but the `/slots/list` that I added makes it mostly pointless since it provides exactly what is in each slot memory wise.