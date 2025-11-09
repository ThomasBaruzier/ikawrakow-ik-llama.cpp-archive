## 🗣️ [Discussion #777](https://github.com/ikawrakow/ik_llama.cpp/discussions/777) - Is there a way to debug and figure out the optimal placement of the OT (output tokens) between the CPU and GPU?

| **Author** | `DrStone1971` |
| :--- | :--- |
| **State** | ✅ **Open** |
| **Created** | 2025-09-11 |
| **Updated** | 2025-09-13 |

---

## 📄 Description

Hi,

I've become really interested in using ik_llama.cpp and I'm trying to understand how to best use a model in a mixed GPU and CPU mode. I have an Nvidia 5090 with 32 Gb of Vram, an Intel Ultra Core 9 285K CPU with 196 Gb of Ram. I've compiled the entire environment (Flash_attn etc, etc) to target CUDA 13 and Pytorch specifically compiled for CUDA 13. I'm working under Ubuntu to best leverage the Unified Memory management of GGML. I was using the “ubergarm/Qwen3-235B-A22B-Instruct-2507-IQ5_K” model (and thank you for their work) using the configuration they proposed:

lama-server \
  --model ./Qwen3-235B-IQ5K/IQ5_K/Qwen3-235B-A22B-Instruct-IQ5_K-00001-of-00004.gguf \
  --alias ubergarm/Qwen3-235B-A22B-Instruct-2507-IQ5_K \
  -fa -fmoe \
  -ctk q8_0 -ctv q8_0 \
  -c 32768 \
  -ngl 99 \
  -ot "blk\.[0-9]\.ffn.*=CUDA0" \
  -ot "blk.*\.ffn.*=CPU" \
  --threads 16 \
  -ub 4096 -b 4096 \
  --host 127.0.0.1 \
  --port 9010
Now, I've noticed that only 7 Gb of Ram is being used, while the VRAM is allocated with all 32 Gb, but only 1.5 Gb is actually being used by llama-server. I assume the rest of the RAM is being utilized by the UNIFIED MEMORY. What’s frustrating is that during text/code generation, only the CPU is working and the GPU isn’t being used at all.

I suspect the problem lies in how I’ve moved the various layers from GPU to CPU, and the system is therefore just using the video card's memory for processing on the CPU, killing the data bus.

How can I figure out how to move or modify the various OT (output tokens?) to better balance the system? Does a DEBUG mode exist that tells you which OT is working and thus move it to the GPU, leaving the others on the CPU?

Thanks for your support.
DrStone71

---

## 💬 Discussion

👤 **firecoperana** commented on **2025-09-11** at **22:45:13**

I don't think there is an option for that. For MOE models, you want to place all non expert layers and the context to your 5090. If your 5090 has additional vram left, you can prioritize expert layers to 5090, and put the rest to CPU. For example, you can do -ot "blk.(0|1|2).ffn_.*_exps=CUDA0" -ot exps=CPU. This means put the first 3 expert layers to gpu and the rest of expert layers to CPU. 
You don't want model to overflow to unified memory. Just put them on CPU directly. You can adjust expert layers placed on 5090 based on your needs.

> 👤 **DrStone1971** replied on **2025-09-13** at **13:43:12**
> 
> thanks !
> 
> only way is testing... ok
> 
> The problem is that I imagined the expert levels, like neural ganglia to be used in particular contexts, but they are actually computation units needed for token analysis.
> 
> Now i work for convert Qwen3-Next-80B-A3B-Thinking in GGUF format, but there is problem with Qwen3NextForCausalLM (isn't a simple new llm)
> 
> Best Regards !