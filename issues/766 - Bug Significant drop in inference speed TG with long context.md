## 📌 [Issue #766](https://github.com/ikawrakow/ik_llama.cpp/issues/766) - Bug: Significant drop in inference speed (TG) with long context

| **Author** | `Eanya-Tonic` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-09-08 |
| **Updated** | 2025-09-26 |

---

## 📄 Description

### What happened?

ik_llama.cpp is an excellent project—thank you very much for providing such a high-quality framework for local inference!

While using ik_llama.cpp to run inference with Qwen3-Coder-480B IQ4-NL, I observed a surprising performance degradation, especially when the context length becomes long. On my hardware setup, when the context is short (less than 10K tokens), the inference speed (TG) remains approximately 6.3 tokens/s. However, as the context length increases, the inference speed gradually declines, dropping to around 2.8 tokens/s when the context reaches 70K–90K tokens. This is quite shocking, and I’m curious whether this issue stems from suboptimal parameter choices or hardware configuration on my end—or whether such performance degradation is expected behavior under long-context conditions?

Below is my launch script:
```
numactl --interleave=all \
        ./build/bin/llama-server \
        --model /path_to_model/Qwen3-Coder-480B-A35B-Instruct-IQ4_NL-00001-of-00006.gguf \
        --alias unsloth/Qwen3-Coder-480B-A35B-IQ4_NL \
        --ctx-size 102400 \
        --temp 0.7 \
        --top-p 0.8 \
        --top-k 20 \
        --repeat-penalty 1.05 \
        -ctk q8_0 \
        -ctv q8_0 \
        -fa \
        -amb 512 \
        -b 2048 \
        -ub 512 \
        -fmoe \
        --n-gpu-layers 99 \
        --override-tensor exps=CPU \
        --threads 36 \
        --threads-batch 38 \
        --host 0.0.0.0 \
        --port 8081 \
        --no-mmap \
        -rtr \
        --mlock \
        --numa numactl \
        --prompt-cache "/patch_to_cache/openai_local_8080.promptcache" \
        --prompt-cache-all \
        --slot-save-path "/path_to_cache/llama_slots/openai_local_8080" \
        --keep -1
```

Below is my hardware configurations: 
```
cpu: 2x Intel Xeon Glod 6138(20 core for 1 cpu, 40 core in total), with hyper-threads turns off.
memory: 12x 32G ddr4 2400 mt/s memory (384GB memory in total)
gpu: Nvidia 2080ti 22GB(which is a modified gpu, but works fine)
```
---
Additionally, I observed a similar inference performance degradation on another machine. Originally, I used ktransformers to run inference with DeepSeek-R1-671B Q4-K-M, supporting a context length of approximately 26K tokens. When the context was short (under 10K tokens), the inference speed was around 5.4 tokens/s; at 24K context length, it dropped slightly to approximately 5.0 tokens/s. After switching to ik_llama.cpp, the speed remained at 5.4 tokens/s with short context—but at 24K context length, the inference speed dropped noticeably to around 4.4 tokens/s. Such a significant performance decline is quite confused to me.

my hardware configuration if helps:
```
cpu: 2x Intel Xeon Platinum 8124M (18 core for 1 cpu, 36 core in total), with hyper-threads turns off.
memory: 16x 64G ddr4 2400 mt/s memory (1024GB memory in total)
gpu: Nvidia 3070 16GB(which is an engineering sample gpu, but works fine)
```

### Name and Version

./build/bin/llama-cli --version
version: 3844 (a3a52300)
built with cc (Ubuntu 11.4.0-1ubuntu1~22.04) 11.4.0 for x86_64-linux-gnu

### What operating system are you seeing the problem on?

Linux

### Relevant log output

```shell

```

---

## 💬 Conversation

👤 **ikawrakow** commented on **2025-09-08** at **07:29:16**

Tokens per second decrease with increasing context is normal and expected. How quickly performance decreases depends on how computationally expensive attention is relative to the feed forward part of the network. I have no experience with the 480B Qwen coder model, so cannot comment on specific TG values.

In your case attention is computed on the GPU, and you are using quantized KV cache. This is significantly slower than fp16 cache. Try removing the -ctk q8_0 -ctv q8_0 arguments. You may need to reduce the maximum context length, but you will see how much you pay for quantized cache in terms of performance.

---

👤 **Eanya-Tonic** commented on **2025-09-08** at **08:42:59**

> Tokens per second decrease with increasing context is normal and expected. How quickly performance decreases depends on how computationally expensive attention is relative to the feed forward part of the network. I have no experience with the 480B Qwen coder model, so cannot comment on specific TG values.
> 
> In your case attention is computed on the GPU, and you are using quantized KV cache. This is significantly slower than fp16 cache. Try removing the -ctk q8_0 -ctv q8_0 arguments. You may need to reduce the maximum context length, but you will see how much you pay for quantized cache in terms of performance.

Thanks! I understand now. I will make another try and remove the -ctk q8_0 -ctv q8_0 arguments.

---

👤 **usrlocalben** commented on **2025-09-15** at **14:13:06**

I recently upgraded from the same generation GPU (RTX 8000). I found the difference to be elucidating: 

<img width="1540" height="429" alt="Image" src="https://github.com/user-attachments/assets/ead5dc0d-bc78-4c06-9d74-6c14ea38b667" />

Measurements are K2 (Q8 w/MoE mixed Q4/Q6) on 2S EPYC 9115.  RTX 8000 (Turing) and RTX 6000 Pro (Blackwell), KV is FP16

If we use time-per-token instead of tok/sec we can visualize the quadratic complexity of attention. (t/sec hides this and I find it unfortunate that localllm et. al. have coalesced around this metric)

Blackwell is so much faster that it pushes the curve out roughly to the model context limits.

Also, Turing is slow enough that it mostly masks all the tweaking of CPU-side parameters. After revisiting all of them I managed to get another 10% or so perf (e.g. EPYC config, and NUMA distribute which is significant if done correctly) Certainly do your CPU-side measurements at very low context depth, n <= 1K.

---

👤 **Eanya-Tonic** commented on **2025-09-16** at **03:02:43**

> I recently upgraded from the same generation GPU (RTX 8000). I found the difference to be elucidating:
> 
> <img alt="Image" width="1540" height="429" src="https://private-user-images.githubusercontent.com/1012320/489567276-ead5dc0d-bc78-4c06-9d74-6c14ea38b667.png?jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NTc5ODc2NjYsIm5iZiI6MTc1Nzk4NzM2NiwicGF0aCI6Ii8xMDEyMzIwLzQ4OTU2NzI3Ni1lYWQ1ZGMwZC1iYzc4LTRjMDYtOWQ3NC02YzE0ZWEzOGI2NjcucG5nP1gtQW16LUFsZ29yaXRobT1BV1M0LUhNQUMtU0hBMjU2JlgtQW16LUNyZWRlbnRpYWw9QUtJQVZDT0RZTFNBNTNQUUs0WkElMkYyMDI1MDkxNiUyRnVzLWVhc3QtMSUyRnMzJTJGYXdzNF9yZXF1ZXN0JlgtQW16LURhdGU9MjAyNTA5MTZUMDE0OTI2WiZYLUFtei1FeHBpcmVzPTMwMCZYLUFtei1TaWduYXR1cmU9NmJmODkyYzY1MTgyMmQ0YTBkMzQ3OGFlOGUyOGExNTM2YjllMzkzMGY1OTE2MGMzOWI4ZDFmNmZiYzEyODlkMyZYLUFtei1TaWduZWRIZWFkZXJzPWhvc3QifQ.27EzOEglw0lrnZAdQ5NKoIBD5bOVkS09b2YL4TdDr6w">
> Measurements are K2 (Q8 w/MoE mixed Q4/Q6) on 2S EPYC 9115. RTX 8000 (Turing) and RTX 6000 Pro (Blackwell), KV is FP16
> 
> If we use time-per-token instead of tok/sec we can visualize the quadratic complexity of attention. (t/sec hides this and I find it unfortunate that localllm et. al. have coalesced around this metric)
> 
> Blackwell is so much faster that it pushes the curve out roughly to the model context limits.
> 
> Also, Turing is slow enough that it mostly masks all the tweaking of CPU-side parameters. After revisiting all of them I managed to get another 10% or so perf (e.g. EPYC config, and NUMA distribute which is significant if done correctly) Certainly do your CPU-side measurements at very low context depth, n <= 1K.

I’ve observed that my Turing architecture GPU (in this case, a 2080 Ti) doesn’t reach full load during runtime. Based on power consumption (~150W during computation / 300W max), I previously assumed the GPU wasn’t the bottleneck for inference and focused instead on CPU/memory bandwidth limitations. Regarding the performance gap between Turing and Blackwell, is the disparity primarily due to the older Turing architecture lacking newer features and precision support (as far as I understand, Turing GPUs only have second-generation Tensor Cores for FP16 acceleration and no support for FP8/BF16)? Or is it more due to lower memory bandwidth on Turing?
While not an AI expert, I’ve noticed the performance difference between Blackwell and Turing is enormous (up to 10x+), suggesting even without considering architectural difference, Turing is inherently slow.

Separately, my Intel Xeon 6138 CPU may also be problematic. Though it has decent paper specs, its real-world performance (including turbo frequencies) is poor. I plan to test with a slightly better CPU (e.g., Xeon 8124M, same socket) to see if any improvements.

On the CPU side, NUMA optimizations are critical. I have used `numactl --interleave=all` to force memory interleaving across two NUMA nodes (dual-socket motherboard). Intel PCM shows UPI utilization near 99% (both links saturated), while local memory usage per node is 50% (expected due to interleaving). The measured memory bandwidth is ~70GB/s, far below the theoretical dual-socket peak (~150GB/s). I wonder if NUMA-aware optimizations could reduce UPI pressure by increasing local memory usage and leveraging node-local bandwidth more effectively?

I’ve seen potential approaches about numa optimizations in projects like:
[FastLLM](https://github.com/ztxz16/fastllm): Seems to use an "expert parallelism" strategy for NUMA and MoE models, allocating memory per expert and keeping computations within local nodes to minimize cross-NUMA traffic.
[LKTransformers](https://github.com/guqiong96/lktransformers): A variant of KTransformers that introduces a Backend_NUMA for NUMA-aware memory allocation instead of duplicating memory across sockets (as in the "double-socket replication" approach). I’m still researching the implementation details. As what I have seen so far, I think it also separate experts and bind them to node, which is very similar to FastLLM.

---

👤 **Eanya-Tonic** commented on **2025-09-16** at **07:43:03**

> On the CPU side, NUMA optimizations are critical. I have used `numactl --interleave=all` to force memory interleaving across two NUMA nodes (dual-socket motherboard). Intel PCM shows UPI utilization near 99% (both links saturated), while local memory usage per node is 50% (expected due to interleaving). The measured memory bandwidth is ~70GB/s, far below the theoretical dual-socket peak (~150GB/s). I wonder if NUMA-aware optimizations could reduce UPI pressure by increasing local memory usage and leveraging node-local bandwidth more effectively?
> 
> I’ve seen potential approaches about numa optimizations in projects like: 
> [FastLLM](https://github.com/ztxz16/fastllm): Seems to use an "expert parallelism" strategy for NUMA and MoE models, allocating memory per expert and keeping computations within local nodes to minimize cross-NUMA traffic. 
> [LKTransformers](https://github.com/guqiong96/lktransformers): A variant of KTransformers that introduces a Backend_NUMA for NUMA-aware memory allocation instead of duplicating memory across sockets (as in the "double-socket replication" approach). I’m still researching the implementation details. As what I have seen so far, I think it also separate experts and bind them to node, which is very similar to FastLLM.

Additional Update on Testing with LKTransformers and NUMA Optimization:
I’ve tested LKTransformers and ik_llama.cpp on an Intel Xeon 8259CL CPU paired with an Ampere GPU. Comparing configurations:

- With numactl --interleave=all and ik_llama.cpp:
  - Memory bandwidth measured via PCM: ~80GB/s (in my opinion, this is mainly affected by upi bandwidth)
  - Decoding speed: 5.85 tokens/s
  - Local memory usage per node: ~50%
- With LKTransformers (NUMA-aware memory allocation, no forced interleaving):
  - Memory bandwidth via PCM: ~100GB/s
  - Decoding speed improved to: 7.89 tokens/s
  - Local memory usage per node: ~98%
 
The NUMA-aware approach in LKTransformers avoids cross-socket memory duplication (unlike the "double-socket replication" method) and significantly reduces UPI contention, which aligns with the bandwidth and performance gains observed.

---

👤 **ikawrakow** commented on **2025-09-16** at **10:41:25**

@Eanya-Tonic 

There have been many NUMA related discussions here and elsewhere. I'm on vacation with just my phone, so tedious to search for specific threads and comments. But in short: ik_llama.cpp has no NUMA specific optimisations, so most of the time it is better to configure your CPU as a single NUMA node.

How does prompt processing (prefill) speed look like for LKTransformers relative to ik_llama? Compared to the command line arguments you used in your first post in this thread, you most likely want to remove -rtr, and want to use larger ubatch (given enough VRAM, -b 4096 -ub 4096 will give highest PP speed)

---

👤 **usrlocalben** commented on **2025-09-16** at **12:41:48**

@ikawrakow `--numa distribute` (w EPYC NPS1) is significantly faster on my system. The drop_caches startup protocol is important, and the load-time is quite long but it is effective, +10-15% TG throughput.

---

👤 **ikawrakow** commented on **2025-09-16** at **14:02:46**

> ikawrakow --numa distribute (w EPYC NPS1) is significantly faster on my system. 

That's why I wrote "most of the time". But maybe "often" or "could be" would have been more appropriate. Either way, it is always useful to try different NUMA configurations to see what works best.

---

👤 **saood06** commented on **2025-09-18** at **19:22:05**

Also @Eanya-Tonic if you want to see exactly how the performance degrades with context you can use `llama-sweep-bench`.

---

👤 **ikawrakow** commented on **2025-09-26** at **16:43:51**

Nothing to be done here.