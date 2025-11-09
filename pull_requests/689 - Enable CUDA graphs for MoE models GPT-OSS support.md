## 🔀 [Pull Request #689](https://github.com/ikawrakow/ik_llama.cpp/pull/689) - Enable CUDA graphs for MoE models + GPT-OSS support

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/try_cuda_graphs` |
| **Target Branch** | `main` |
| **Created** | 2025-08-14 |
| **Updated** | 2025-08-17 |
| **Merged** | 2025-08-15 |

---

## 📄 Description

This PR enables CUDA graphs for MoE models (TG only, just like in mainline `llama.cpp`).

The implementation is largely based on mainline, but given the massive divergence between the two code bases and the fact that CUDA graphs in mainline are the result of many PR's, I couldn't cherry-pick, so it is copy/adjust.

Unlike earlier CUDA graph incarnations that I have tried, this time I'm observing a non-negligible TG performance gains on Linux. Given [recent reports](https://huggingface.co/ubergarm/GLM-4.5-Air-GGUF/discussions/2#689991f8dc5c06e6964439be) about mainline's TG performance being better than `ik_llama.cpp` on Windows, my guess is that on Windows the impact will be significantly higher.

I have tested with 3 different MoE models that I can fully offloaded on my RTX-4080 GPU (GPT-OSS-20B, Qwen3-30B-A3B, DeepSeek-Lite-16B). ~Worth noting that `mla = 0` and `mla = 2` will no longer work for DeepSeek models as some pieces are still missing when CUDA graphs are enabled. But given that `mla = 3` is the recommended option and the fact that use `mla = 0` or `mla = 2` was discouraged quite some time ago, this should be OK~. **Updated**: `-mla 1,3` is required for DeepSeek models only when not using `f16` KV cache.  

**Note**: this PR has been branched of the unmerged PR [#683](https://github.com/ikawrakow/ik_llama.cpp/issues/683), not the main branch, and hence it included GPT-OSS support. This is also the reason the change is so large (it includes the +7083/-4096 changes from [#683](https://github.com/ikawrakow/ik_llama.cpp/issues/683))

> [!IMPORTANT]
> For MoE models `-fmoe` is required, else graphs will be disabled.

> [!IMPORTANT]
> There is a report of reduced performance with CUDA graphs [here](https://github.com/ikawrakow/ik_llama.cpp/pull/689#issuecomment-3188662552) and [here](https://github.com/ikawrakow/ik_llama.cpp/pull/689#issuecomment-3191173019). If you observe lower performance after this PR has been merged, you can disable CUDA graphs by adding `-DGGML_CUDA_USE_GRAPHS=OFF` to the `cmake` build command.

@Thireus I would appreciate testing this PR with your Windows setup with GLM-4.5-Air (and other MoE models). 

Here are some performance comparisons on RTX-4080.

### DeepSeek-Lite-16B, Q4_0

<img width="792" height="612" alt="graph1" src="https://github.com/user-attachments/assets/17b51ae6-7f3c-4c79-9eb6-6c4347964e2f" />

### Qwen3-30B-A3B, Q2_K_S

<img width="792" height="612" alt="graph2" src="https://github.com/user-attachments/assets/1ec4a9c1-5648-4eb6-8705-2ec9ba04b05c" />

### GPT-OSS-20B, MXFP4

<img width="792" height="612" alt="graph3" src="https://github.com/user-attachments/assets/89f68b67-bf86-4c00-83fa-d81ded322ab5" />

---

## 💬 Conversation

👤 **Thireus** commented on **2025-08-14** at **12:34:14**

I'm starting to have doubts that @ikawrakow is even human or if he's AGI. This is really impressive work. I'll be testing the perfs, thank you so much!

---

👤 **ikawrakow** commented on **2025-08-14** at **12:44:54**

> I'm starting to have doubts that @ikawrakow is even human or if he's AGI. This is really impressive work. I'll be testing the perfs, thank you so much!

Haha. If that were true, it will be ikawrakowAGI becoming the multi-trillion dollar company, not OpenAI or Antropic or Google or ... So perhaps it is time for people to start investing into that :laughing:

---

👤 **Ph0rk0z** commented on **2025-08-14** at **13:46:14**

Is there effect for hybrid inference?

---

👤 **ikawrakow** commented on **2025-08-14** at **13:50:58**

> Is there effect for hybrid inference?

I haven't tested with that, but my guess is that it will depend on how much of the graph is done on the GPU. Also, if all MoE tensors are on the CPU, then I don't expect any effect because the MoE operations were the thing that disabled CUDA graphs in `ik_llama.cpp`, so whatever was run on the GPU should have already used CUDA graphs (without me putting my hand into the fire for that statement).

Isn't it easiest to simply try if you see an effect for your setup?

---

👤 **Ph0rk0z** commented on **2025-08-14** at **14:05:11**

Just asking since it takes a bit of time to load models off disk. I usually split MoE tensors between GPU and CPU. CPU T/G alone on my machine is not great with these large models.

---

👤 **espen96** commented on **2025-08-14** at **14:27:16**

Just a quick thest with the 20b model, 100% on my 3090 on windows. 

No hard numbers, but on a basic first prompt, I am seeing token generation speeds around 80-100 without CUDA graphs.
With CUDA graphs I am seeing speeds closer to 115-125 tps generation.

Mainline gets speeds around 117-130 tps generation speeds

Fair to say this appears to have made a significant change to the pure cuda performance.
```             
-ngl 999 -np 1--dry_multiplier 0.55 --dry_base 1.5 --temp 0.8 --min-p 0.05 --top-p 0.8 --top-k 40 --threads 1 --parallel 1 --threads 1 --main-gpu 0 -fa -fmoe --tensor-split 100,0 
 ```

---

👤 **usrlocalben** commented on **2025-08-14** at **14:28:44**

slow on turing w hybrid cpu/gpu.
my normal invocation is `-mla 2` but PR indicates it needs to be 3.
I've read conflicting info wrt. cuda graphs support on Turing. I'm not sure what to expect.

```
kimi q8_0, rtx 8000 (turing) exp=cpu

main, mla=2
prompt eval time     =  105367.08 ms / 10031 tokens (   10.50 ms per token,    95.20 tokens per second)
generation eval time =   38503.78 ms /   265 runs   (  145.30 ms per token,     6.88 tokens per second)

try_cuda_graphs, mla=3
prompt eval time     =  279420.25 ms / 10034 tokens (   27.85 ms per token,    35.91 tokens per second)
generation eval time =   69189.65 ms /   267 runs   (  259.14 ms per token,     3.86 tokens per second)
```

---

👤 **ikawrakow** commented on **2025-08-14** at **15:03:17**

@usrlocalben

Thanks for testing. I don't think I understand the results. CUDA graphs are disabled for PP, so the PR should not affect PP performance at all. For PP, the graph is created and evaluated in exactly the same way for `mla = 2` and `mla = 3`. The difference between `mla = 2` and `mla = 3` is in the way TG is done: when `mla = 2`, the K-cache is transposed (the entire K-cache!), and the FA implementation for K/V head sizes of 192/128 is used, which is available on Turing. For `mla = 3` one needs the FA implementation for K/V head sizes of 576/512, which initially required Ampere or higher, but was later relaxed to Turing or higher. As transposing a large K cache for every token is quite expensive, I would normally expect `mla = 3` to be much faster than `mla = 2` for TG. Unless of course the 576/512 FA kernel is somehow much slower on Turing.  

Oh, wait. It is hybrid, so the graph is split into many pieces, so some of the pieces will use CUDA graphs also for PP. That makes it much harder to predict. I need to do some testing for hybrid inference to understand better.

---

👤 **Ph0rk0z** commented on **2025-08-14** at **15:19:34**

With ampere, I loaded qwen-235b. Essentially performance is unchanged. I get the same 18t/s and it follows my benchmarks from a couple of days ago.

---

👤 **ikawrakow** commented on **2025-08-14** at **15:51:13**

OK, here is a graph for hybrid inference. DeepSeek-Lite, `Q4_0` (model has 27 layers), TG as a function of number of MoE layers left on the CPU (Ryzen-7950X) via `--n-cpu-moe` for the first 256 generated tokens (no previous context). So, the PR will not have impact for partially offloaded models when the number of layers left on the CPU is more than half. At least not on Linux (and it would be interesting to test such a scenario on Windows).

<img width="792" height="612" alt="v" src="https://github.com/user-attachments/assets/d45e79f0-2b0e-4c5a-9429-b19836cec32b" />

---

👤 **ikawrakow** commented on **2025-08-14** at **16:01:48**

@usrlocalben

I have updated the description. `mla = 2` can be used with `f16` cache. To enable `Q8_0` cache, I need to add the required `Q8_0 -> Q8_0` transpose op and graph capture, which shouldn't be too hard.

---

👤 **Thireus** commented on **2025-08-14** at **16:10:45**

llama.cpp (main) - Windows builds: [b6168](https://github.com/Thireus/llama.cpp/releases/tag/b6168)
```
CUDA_DEVICE_ORDER=PCI_BUS_ID CUDA_VISIBLE_DEVICES=1 ~/llama-b6168-bin-win-cuda-12.8-x64/llama-bench -m GLM-4.5-Air-THIREUS-BF16-SPECIAL_TENSOR-00001-of-00804.gguf -fa 1 \
  -ctk f16 \
  -ngl 99 \
  -b 4096 -ub 4096 \
  --threads 1 \
  --main-gpu 0 -p 8192 -n 512 --mmap 0
ggml_cuda_init: GGML_CUDA_FORCE_MMQ:    no
ggml_cuda_init: GGML_CUDA_FORCE_CUBLAS: no
ggml_cuda_init: found 1 CUDA devices:
  Device 0: NVIDIA RTX PRO 6000 Blackwell Workstation Edition, compute capability 12.0, VMM: yes
load_backend: loaded CUDA backend from C:\cygwin64\home\Thireus\llama-b6168-bin-win-cuda-12.8-x64\ggml-cuda.dll
load_backend: loaded RPC backend from C:\cygwin64\home\Thireus\llama-b6168-bin-win-cuda-12.8-x64\ggml-rpc.dll
load_backend: loaded CPU backend from C:\cygwin64\home\Thireus\llama-b6168-bin-win-cuda-12.8-x64\ggml-cpu-skylakex.dll
| model                          |       size |     params | backend    | ngl | threads | n_batch | n_ubatch | fa | mmap |            
test |                  t/s |
| ------------------------------ | ---------: | ---------: | ---------- | --: | ------: | ------: | -------: | -: | ---: | --------------: | -------------------: |
| glm4moe 106B.A12B IQ1_S - 1.5625 bpw |  47.83 GiB |   110.47 B | CUDA,RPC   |  99 |       1 |    4096 |     4096 |  1 |    0 |          pp8192 |       1997.36 ± 6.68 |
| glm4moe 106B.A12B IQ1_S - 1.5625 bpw |  47.83 GiB |   110.47 B | CUDA,RPC   |  99 |       1 |    4096 |     4096 |  1 |    0 |           tg512 |        104.83 ± 0.30 |

build: 092fd845 (6168)
```

ik_llama.cpp (main) - Windows builds: [main-b4074-62ef02e](https://github.com/Thireus/ik_llama.cpp/releases/tag/main-b4074-62ef02e)
```
CUDA_DEVICE_ORDER=PCI_BUS_ID CUDA_VISIBLE_DEVICES=1 ~/ik_llama-main-b4074-62ef02e-bin-win-cuda-12.8-x64-avx512/llama-bench -m GLM-4.5-Air-THIREUS-BF16-SPECIAL_TENSOR-00001-of-00804.gguf -fa 1 \
  -ctk f16 \
  -ngl 99 \
  -b 4096 -ub 4096 \
  --threads 1 \
  --main-gpu 0 -p 8192 -n 512 --mmap 0 -fmoe 1
ggml_cuda_init: GGML_CUDA_FORCE_MMQ:    no
ggml_cuda_init: GGML_CUDA_FORCE_CUBLAS: no
ggml_cuda_init: found 1 CUDA devices:
  Device 0: NVIDIA RTX PRO 6000 Blackwell Workstation Edition, compute capability 12.0, VMM: yes
| model                          |       size |     params | backend    | ngl | threads | n_batch | n_ubatch | fa | mmap | fmoe |          test |              t/s |
| ------------------------------ | ---------: | ---------: | ---------- | --: | ------: | ------: | -------: | -: | ---: | ---: | ------------: | ---------------: |
| glm4moe 106B.A12B IQ1_S - 1.5625 bpw |  46.20 GiB |   106.85 B | CUDA       |  99 |       1 |    4096 |     4096 |  1 |    0 |    1 |        pp8192 | 2803.68 ± 127.85 |
| glm4moe 106B.A12B IQ1_S - 1.5625 bpw |  46.20 GiB |   106.85 B | CUDA       |  99 |       1 |    4096 |     4096 |  1 |    0 |    1 |         tg512 |     44.92 ± 0.52 |

build: 62ef02e (1)
```

ik_llama.cpp (1faa0868e767e21b9956a3ba42d378d4a2b08838) - Windows builds: [ik-try_cuda_graphs-b4105-c71d91e](https://github.com/Thireus/ik_llama.cpp/releases/tag/ik-try_cuda_graphs-b4105-c71d91e)
```
CUDA_DEVICE_ORDER=PCI_BUS_ID CUDA_VISIBLE_DEVICES=1 ~/ik_llama-ik-try_cuda_graphs-b4105-c71d91e-bin-win-cuda-12.8-x64-avx512/llama-bench -m GLM-4.5-Air-THIREUS-BF16-SPECIAL_TENSOR-00001-of-00804.gguf -fa 1   -ctk f16   -ngl 99   -b 4096 -ub 4096   --threads 1   --main-gpu 0 -p 8192 -n 512 --mmap 0 -fmoe 1
ggml_cuda_init: GGML_CUDA_FORCE_MMQ:    no
ggml_cuda_init: GGML_CUDA_FORCE_CUBLAS: no
ggml_cuda_init: found 1 CUDA devices:
  Device 0: NVIDIA RTX PRO 6000 Blackwell Workstation Edition, compute capability 12.0, VMM: yes
| model                          |       size |     params | backend    | ngl | threads | n_batch | n_ubatch | fa | mmap | fmoe |          test |              t/s |
| ------------------------------ | ---------: | ---------: | ---------- | --: | ------: | ------: | -------: | -: | ---: | ---: | ------------: | ---------------: |
| glm4moe 106B.A12B IQ1_S - 1.5625 bpw |  46.20 GiB |   106.85 B | CUDA       |  99 |       1 |    4096 |     4096 |  1 |    0 |    1 |        pp8192 | 2523.48 ± 891.80 |
| glm4moe 106B.A12B IQ1_S - 1.5625 bpw |  46.20 GiB |   106.85 B | CUDA       |  99 |       1 |    4096 |     4096 |  1 |    0 |    1 |         tg512 |     91.38 ± 0.06 |

build: c71d91e (1)
```

ik_llama.cpp (8a83e1f0830cd6b4448fd225892d870dd904ba7e) - Windows builds: [ik-try_cuda_graphs-b4107-7693263](https://github.com/Thireus/ik_llama.cpp/releases/tag/ik-try_cuda_graphs-b4107-7693263)
```
CUDA_DEVICE_ORDER=PCI_BUS_ID CUDA_VISIBLE_DEVICES=0 ~/ik_llama-ik-try_cuda_graphs-b4107-7693263-bin-win-cuda-12.8-x64-avx512/llama-bench -m GLM-4.5-Air-THIREUS-BF16-SPECIAL_TENSOR-00001-of-00804.gguf -fa 1   -ctk f16   -ngl 99   -b 4096 -ub 4096   --threads 1   --main-gpu 0 -p 8192 -n 512 --mmap 0 -fmoe 1
ggml_cuda_init: GGML_CUDA_FORCE_MMQ:    no
ggml_cuda_init: GGML_CUDA_FORCE_CUBLAS: no
ggml_cuda_init: found 1 CUDA devices:
  Device 0: NVIDIA RTX PRO 6000 Blackwell Workstation Edition, compute capability 12.0, VMM: yes
| model                          |       size |     params | backend    | ngl | threads | n_batch | n_ubatch | fa | mmap | fmoe |          test |              t/s |
| ------------------------------ | ---------: | ---------: | ---------- | --: | ------: | ------: | -------: | -: | ---: | ---: | ------------: | ---------------: |
| glm4moe 106B.A12B IQ1_S - 1.5625 bpw |  46.20 GiB |   106.85 B | CUDA       |  99 |       1 |    4096 |     4096 |  1 |    0 |    1 |        pp8192 |  2907.18 ± 87.08 |
| glm4moe 106B.A12B IQ1_S - 1.5625 bpw |  46.20 GiB |   106.85 B | CUDA       |  99 |       1 |    4096 |     4096 |  1 |    0 |    1 |         tg512 |     92.08 ± 0.05 |

build: 7693263 (1)
```

Much better! Still not at the level of llama.cpp but getting close! Need to compare with llama-sweep-bench as well.

GLM-4.5-Air recipe used:

<details>

```
## Model head & embeddings
output_norm\.weight=f32
token_embd\.weight=q5_K
output\.weight=q8_0

## Multi-headed attention parameters
blk\.([0-9]|[1-3][0-9]|4[0-6])\.attn_k\.bias=f32
blk\.([0-9]|[1-3][0-9]|4[0-6])\.attn_output\.weight=iq4_xs
blk\.([0-9]|[1-3][0-9]|4[0-6])\.attn_k\.weight=iq4_xs
blk\.([0-9]|[1-3][0-9]|4[0-6])\.attn_q\.weight=iq4_xs
blk\.([0-9]|[1-3][0-9]|4[0-6])\.attn_q\.bias=f32
blk\.([0-9]|[1-3][0-9]|4[0-6])\.attn_v\.bias=f32
blk\.([0-9]|[1-3][0-9]|4[0-6])\.attn_v\.weight=iq4_xs
blk\.([0-9]|[1-3][0-9]|4[0-6])\.attn_norm\.weight=f32

## Core FFN weights
blk\.0\.ffn_gate\.weight=iq3_xxs
blk\.([1-9]|[1-3][0-9]|4[0-6])\.ffn_gate_inp\.weight=f32
blk\.0\.ffn_down\.weight=iq3_xxs
blk\.0\.ffn_up\.weight=iq3_xxs

## Other tensors
blk\.([0-9]|[1-3][0-9]|4[0-6])\.post_attention_norm\.weight=f32
blk\.46\.nextn\.shared_head_head\.weight=iq4_xs
blk\.46\.nextn\.embed_tokens\.weight=iq4_xs
blk\.46\.nextn\.shared_head_norm\.weight=f32
blk\.([1-9]|[1-3][0-9]|4[0-6])\.exp_probs_b\.bias=f32
blk\.46\.nextn\.enorm\.weight=f32
blk\.46\.nextn\.hnorm\.weight=f32
blk\.46\.nextn\.eh_proj\.weight=iq4_xs

## GPU-loaded ffn_*_shexp
# ffn_down_shexp (down-projection)
blk\..*\.ffn_down_shexp\.weight=iq3_xxs

# ffn_up_shexp (up-projection)
blk\..*\.ffn_up_shexp\.weight=iq3_xxs

# ffn_gate_shexp (gate-projection)
blk\..*\.ffn_gate_shexp\.weight=iq3_xxs

## CPU-loaded ffn_*_exps
# ffn_down_exps (down-extraction)
blk\..*\.ffn_down_exps\.weight=iq3_xxs

# ffn_up_exps (up-extraction)
blk\..*\.ffn_up_exps\.weight=iq3_xxs

# ffn_gate_exps (gate-extraction)
blk\..*\.ffn_gate_exps\.weight=iq3_xxs
```

</details>

---

👤 **usrlocalben** commented on **2025-08-14** at **16:36:42**

> @usrlocalben
> 
> I have updated the description. `mla = 2` can be used with `f16` cache. To enable `Q8_0` cache, I need to add the required `Q8_0 -> Q8_0` transpose op and graph capture, which shouldn't be too hard.

My results above were Q8_0 model, but cache is f16

---

👤 **ikawrakow** commented on **2025-08-14** at **16:40:00**

> My results above were Q8_0 model, but cache is f16

So, then, `mla = 2` should work in case you want to give it a try (given that `mla = 2` seems to work better for your setup). 

The last commit should make it possible to use this PR also with `mla = 2`  and `q8_0` KV cache.

---

👤 **ubergarm** commented on **2025-08-14** at **17:30:42**

I just ran some tests of this PR vs main vs mainline for a GLM-4.5-Air Q4_0 mix quant fully offloaded onto dual RTX A6000 48GB VRAM GPUs (non blackwell older sm86 arch cards). This is Linux Ubuntu 24.04.1 LTS with `Driver Version: 570.133.20 CUDA Version: 12.8 `.

This PR does give a slight boost in TG. Interestingly mainline doesn't seem to benefit from larger batches for PP here. ik gives about ~900 tok/sec PP in default batches, still beating mainline's ~700 tok/sec PP in default batches.

<img width="2087" height="1081" alt="sweep-bench-GLM-4 5-Air-Q4_0-CUDA-graphs" src="https://github.com/user-attachments/assets/59d9b86d-8929-4d96-af31-c27283f3a308" />

<details>

<summary>👈 Details</summary>

```bash
#!/usr/bin/env bash

model=/mnt/raid/hf/GLM-4.5-Air-GGUF/Q4_0/GLM-4.5-Air-Q4_0-00001-of-00002.gguf

./build/bin/llama-sweep-bench \
    --model "$model"\
    -fa -fmoe \
    -c 20480 \
    -ngl 99 \
    --threads 1 \
    -ub 4096 -b 4096 \
    --warmup-batch

llama_model_loader: - type q4_0:  137 tensors
llama_model_loader: - type q8_0:  333 tensors
llama_model_loader: - type q4_K:    1 tensors
llama_model_loader: - type q6_K:    1 tensors
```

## ik PR
|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  4096 |   1024 |      0 |    2.553 |  1604.50 |   21.019 |    48.72 |
|  4096 |   1024 |   4096 |    3.219 |  1272.39 |   27.313 |    37.49 |
|  4096 |   1024 |   8192 |    3.922 |  1044.26 |   33.758 |    30.33 |
|  4096 |   1024 |  12288 |    4.575 |   895.33 |   40.482 |    25.30 |
|  4096 |   1024 |  16384 |    5.226 |   783.72 |   47.059 |    21.76 |

## ik main
|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  4096 |   1024 |      0 |    2.611 |  1568.72 |   22.661 |    45.19 |
|  4096 |   1024 |   4096 |    3.273 |  1251.34 |   29.072 |    35.22 |
|  4096 |   1024 |   8192 |    3.949 |  1037.19 |   35.518 |    28.83 |
|  4096 |   1024 |  12288 |    4.589 |   892.60 |   42.313 |    24.20 |
|  4096 |   1024 |  16384 |    5.222 |   784.41 |   48.983 |    20.91 |

## mainline lcpp master@7a0de960 + ug/port-sweep-bench
|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  4096 |   1024 |      0 |    5.804 |   705.68 |   24.413 |    41.94 |
|  4096 |   1024 |   4096 |    7.252 |   564.78 |   32.104 |    31.90 |
|  4096 |   1024 |   8192 |    8.777 |   466.69 |   39.827 |    25.71 |
|  4096 |   1024 |  12288 |   10.179 |   402.40 |   47.508 |    21.55 |
|  4096 |   1024 |  16384 |   11.586 |   353.54 |   55.091 |    18.59 |

</details>

If I can get access to a rig with newer CUDA drivers >=12.9 I'd love to try the nv_coopmat2 Vulkan backend, which could give more TG...

---

👤 **Thireus** commented on **2025-08-14** at **17:48:10**

I'm facing this issue when running `llama-server`:

> CUDA error: operation not permitted when stream is capturing

```
$ CUDA_DEVICE_ORDER=PCI_BUS_ID CUDA_VISIBLE_DEVICES=0 ~/ik_llama-ik-try_cuda_graphs-b4107-7693263-bin-win-cuda-12.8-x64-avx512/llama-server -m GLM-4.5-Air-THIREUS-BF16-SPECIAL_TENSOR-00001-of-00804.gguf -fa   -ctk f16   -c 131072   -ngl 99   -b 4096 -ub 4096   --no-mmap   --threads 36   --main-gpu 0
INFO [                    main] build info | tid="27976" timestamp=1755193591 build=1 commit="7693263"
INFO [                    main] system info | tid="27976" timestamp=1755193591 n_threads=36 n_threads_batch=-1 total_threads=36 system_info="AVX = 1 | AVX_VNNI = 0 | AVX2 = 1 | AVX512 = 1 | AVX512_VBMI = 0 | AVX512_VNNI = 0 | AVX512_BF16 = 0 | FMA = 1 | NEON = 0 | SVE = 0 | ARM_FMA = 0 | F16C = 1 | FP16_VA = 0 | WASM_SIMD = 0 | BLAS = 1 | SSE3 = 1 | SSSE3 = 1 | VSX = 0 | MATMUL_INT8 = 0 | LLAMAFILE = 1 | "
llama_model_loader: max stdio successfully set to 2048
llama_model_loader: additional 803 GGUFs metadata loaded.
llama_model_loader: loaded meta data with 44 key-value pairs and 803 tensors from GLM-4.5-Air-THIREUS-BF16-SPECIAL_TENSOR-00001-of-00804.gguf (version GGUF V3 (latest))
llama_model_loader: Dumping metadata keys/values. Note: KV overrides do not apply in this output.
llama_model_loader: - kv   0:                       general.architecture str              = glm4moe
llama_model_loader: - kv   1:                               general.type str              = model
llama_model_loader: - kv   2:                               general.name str              = GLM 4.5 Air
llama_model_loader: - kv   3:                         general.size_label str              = 128x9.4B
llama_model_loader: - kv   4:                            general.license str              = mit
llama_model_loader: - kv   5:                               general.tags arr[str,1]       = ["text-generation"]
llama_model_loader: - kv   6:                          general.languages arr[str,2]       = ["en", "zh"]
llama_model_loader: - kv   7:                        glm4moe.block_count u32              = 47
llama_model_loader: - kv   8:                     glm4moe.context_length u32              = 131072
llama_model_loader: - kv   9:                   glm4moe.embedding_length u32              = 4096
llama_model_loader: - kv  10:                glm4moe.feed_forward_length u32              = 10944
llama_model_loader: - kv  11:               glm4moe.attention.head_count u32              = 96
llama_model_loader: - kv  12:            glm4moe.attention.head_count_kv u32              = 8
llama_model_loader: - kv  13:                     glm4moe.rope.freq_base f32              = 1000000.000000
llama_model_loader: - kv  14:   glm4moe.attention.layer_norm_rms_epsilon f32              = 0.000010
llama_model_loader: - kv  15:                  glm4moe.expert_used_count u32              = 8
llama_model_loader: - kv  16:               glm4moe.attention.key_length u32              = 128
llama_model_loader: - kv  17:             glm4moe.attention.value_length u32              = 128
llama_model_loader: - kv  18:                          general.file_type u32              = 24
llama_model_loader: - kv  19:               glm4moe.rope.dimension_count u32              = 64
llama_model_loader: - kv  20:                       glm4moe.expert_count u32              = 128
llama_model_loader: - kv  21:         glm4moe.expert_feed_forward_length u32              = 1408
llama_model_loader: - kv  22:                glm4moe.expert_shared_count u32              = 1
llama_model_loader: - kv  23:          glm4moe.leading_dense_block_count u32              = 1
llama_model_loader: - kv  24:                 glm4moe.expert_gating_func u32              = 2
llama_model_loader: - kv  25:               glm4moe.expert_weights_scale f32              = 1.000000
llama_model_loader: - kv  26:                glm4moe.expert_weights_norm bool             = true
llama_model_loader: - kv  27:               glm4moe.nextn_predict_layers u32              = 1
llama_model_loader: - kv  28:                       tokenizer.ggml.model str              = gpt2
llama_model_loader: - kv  29:                         tokenizer.ggml.pre str              = glm4
llama_model_loader: - kv  30:                      tokenizer.ggml.tokens arr[str,151552]  = ["!", "\"", "#", "$", "%", "&", "'", ...
llama_model_loader: - kv  31:                  tokenizer.ggml.token_type arr[i32,151552]  = [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, ...  
llama_model_loader: - kv  32:                      tokenizer.ggml.merges arr[str,318088]  = ["─á ─á", "─á ─á─á─á", "─á─á ─á─á", "...
llama_model_loader: - kv  33:                tokenizer.ggml.eos_token_id u32              = 151329
llama_model_loader: - kv  34:            tokenizer.ggml.padding_token_id u32              = 151329
llama_model_loader: - kv  35:                tokenizer.ggml.bos_token_id u32              = 151331
llama_model_loader: - kv  36:                tokenizer.ggml.eot_token_id u32              = 151336
llama_model_loader: - kv  37:            tokenizer.ggml.unknown_token_id u32              = 151329
llama_model_loader: - kv  38:                tokenizer.ggml.eom_token_id u32              = 151338
llama_model_loader: - kv  39:                    tokenizer.chat_template str              = [gMASK]<sop>\n{%- if tools -%}\n<|syste...
llama_model_loader: - kv  40:               general.quantization_version u32              = 2
llama_model_loader: - kv  41:                                   split.no u16              = 0
llama_model_loader: - kv  42:                                split.count u16              = 804
llama_model_loader: - kv  43:                        split.tensors.count i32              = 803
llama_model_loader: - type  f32:  331 tensors
llama_model_loader: - type q8_0:  105 tensors
llama_model_loader: - type q5_K:   56 tensors
llama_model_loader: - type q6_K:   43 tensors
llama_model_loader: - type iq4_nl:   92 tensors
llama_model_loader: - type iq4_xs:  176 tensors
init_tokenizer: initializing tokenizer for type 2
load: control token: 151334 '<eop>' is not marked as EOG
load: control token: 151331 '[gMASK]' is not marked as EOG
load: control token: 151362 '<|end_of_box|>' is not marked as EOG
load: control token: 151343 '<|begin_of_audio|>' is not marked as EOG
load: control token: 151335 '<|system|>' is not marked as EOG
load: control token: 151345 '<|begin_of_transcription|>' is not marked as EOG
load: control token: 151344 '<|end_of_audio|>' is not marked as EOG
load: control token: 151336 '<|user|>' is not marked as EOG
load: control token: 151332 '[sMASK]' is not marked as EOG
load: control token: 151346 '<|end_of_transcription|>' is not marked as EOG
load: control token: 151330 '[MASK]' is not marked as EOG
load: control token: 151333 '<sop>' is not marked as EOG
load: control token: 151337 '<|assistant|>' is not marked as EOG
load: control token: 151338 '<|observation|>' is not marked as EOG
load: control token: 151339 '<|begin_of_image|>' is not marked as EOG
load: control token: 151340 '<|end_of_image|>' is not marked as EOG
load: control token: 151341 '<|begin_of_video|>' is not marked as EOG
load: control token: 151342 '<|end_of_video|>' is not marked as EOG
load: control token: 151347 '<|code_prefix|>' is not marked as EOG
load: control token: 151348 '<|code_middle|>' is not marked as EOG
load: control token: 151349 '<|code_suffix|>' is not marked as EOG
load: control token: 151360 '/nothink' is not marked as EOG
load: control token: 151361 '<|begin_of_box|>' is not marked as EOG
load: control token: 151363 '<|image|>' is not marked as EOG
load: control token: 151364 '<|video|>' is not marked as EOG
load: special_eot_id is not in special_eog_ids - the tokenizer config may be incorrect
load: special_eom_id is not in special_eog_ids - the tokenizer config may be incorrect
load: printing all EOG tokens:
load:   - 151329 ('<|endoftext|>')
load:   - 151336 ('<|user|>')
load:   - 151338 ('<|observation|>')
load: special tokens cache size = 36
load: token to piece cache size = 0.9713 MB
llm_load_print_meta: format           = GGUF V3 (latest)
llm_load_print_meta: arch             = glm4moe
llm_load_print_meta: n_ctx_train      = 131072
llm_load_print_meta: n_embd           = 4096
llm_load_print_meta: n_layer          = 47
llm_load_print_meta: n_head           = 96
llm_load_print_meta: n_head_kv        = 8
llm_load_print_meta: n_rot            = 64
llm_load_print_meta: n_swa            = 0
llm_load_print_meta: n_swa_pattern    = 1
llm_load_print_meta: n_embd_head_k    = 128
llm_load_print_meta: n_embd_head_v    = 128
llm_load_print_meta: n_gqa            = 12
llm_load_print_meta: n_embd_k_gqa     = 1024
llm_load_print_meta: n_embd_v_gqa     = 1024
llm_load_print_meta: f_norm_eps       = 0.0e+00
llm_load_print_meta: f_norm_rms_eps   = 1.0e-05
llm_load_print_meta: f_clamp_kqv      = 0.0e+00
llm_load_print_meta: f_max_alibi_bias = 0.0e+00
llm_load_print_meta: f_logit_scale    = 0.0e+00
llm_load_print_meta: n_ff             = 10944
llm_load_print_meta: n_expert         = 128
llm_load_print_meta: n_expert_used    = 8
llm_load_print_meta: causal attn      = 1
llm_load_print_meta: pooling type     = 0
llm_load_print_meta: rope type        = 2
llm_load_print_meta: rope scaling     = linear
llm_load_print_meta: freq_base_train  = 1000000.0
llm_load_print_meta: freq_scale_train = 1
llm_load_print_meta: n_ctx_orig_yarn  = 131072
llm_load_print_meta: rope_finetuned   = unknown
llm_load_print_meta: ssm_d_conv       = 0
llm_load_print_meta: ssm_d_inner      = 0
llm_load_print_meta: ssm_d_state      = 0
llm_load_print_meta: ssm_dt_rank      = 0
llm_load_print_meta: model type       = 106B.A12B
llm_load_print_meta: model ftype      = IQ1_S - 1.5625 bpw
llm_load_print_meta: model params     = 110.469 B
llm_load_print_meta: model size       = 65.117 GiB (5.063 BPW)
llm_load_print_meta: repeating layers = 64.105 GiB (5.041 BPW, 109.227 B parameters)
llm_load_print_meta: general.name     = GLM 4.5 Air
print_info: vocab type       = BPE
print_info: n_vocab          = 151552
print_info: n_merges         = 318088
print_info: BOS token        = 151331 '[gMASK]'
print_info: EOS token        = 151329 '<|endoftext|>'
print_info: EOT token        = 151336 '<|user|>'
print_info: EOM token        = 151338 '<|observation|>'
print_info: UNK token        = 151329 '<|endoftext|>'
print_info: PAD token        = 151329 '<|endoftext|>'
print_info: LF token         = 198 '─è'
print_info: FIM PRE token    = 151347 '<|code_prefix|>'
print_info: FIM SUF token    = 151349 '<|code_suffix|>'
print_info: FIM MID token    = 151348 '<|code_middle|>'
print_info: EOG token        = 151329 '<|endoftext|>'
print_info: EOG token        = 151336 '<|user|>'
print_info: EOG token        = 151338 '<|observation|>'
print_info: max token length = 1024
ggml_cuda_init: GGML_CUDA_FORCE_MMQ:    no
ggml_cuda_init: GGML_CUDA_FORCE_CUBLAS: no
ggml_cuda_init: found 1 CUDA devices:
  Device 0: NVIDIA RTX PRO 6000 Blackwell Workstation Edition, compute capability 12.0, VMM: yes
llm_load_tensors: ggml ctx size =    0.66 MiB
model has unused tensor blk.46.attn_norm.weight (size = 16384 bytes) -- ignoring
model has unused tensor blk.46.attn_q.weight (size = 26738688 bytes) -- ignoring
model has unused tensor blk.46.attn_k.weight (size = 2228224 bytes) -- ignoring
model has unused tensor blk.46.attn_v.weight (size = 2228224 bytes) -- ignoring
model has unused tensor blk.46.attn_q.bias (size = 49152 bytes) -- ignoring
model has unused tensor blk.46.attn_k.bias (size = 4096 bytes) -- ignoring
model has unused tensor blk.46.attn_v.bias (size = 4096 bytes) -- ignoring
model has unused tensor blk.46.attn_output.weight (size = 53477376 bytes) -- ignoring
model has unused tensor blk.46.post_attention_norm.weight (size = 16384 bytes) -- ignoring
model has unused tensor blk.46.ffn_gate_inp.weight (size = 2097152 bytes) -- ignoring
model has unused tensor blk.46.exp_probs_b.bias (size = 512 bytes) -- ignoring
model has unused tensor blk.46.ffn_gate_exps.weight (size = 605552640 bytes) -- ignoring
model has unused tensor blk.46.ffn_down_exps.weight (size = 415236096 bytes) -- ignoring
model has unused tensor blk.46.ffn_up_exps.weight (size = 605552640 bytes) -- ignoring
model has unused tensor blk.46.ffn_gate_shexp.weight (size = 4730880 bytes) -- ignoring
model has unused tensor blk.46.ffn_down_shexp.weight (size = 3244032 bytes) -- ignoring
model has unused tensor blk.46.ffn_up_shexp.weight (size = 4730880 bytes) -- ignoring
model has unused tensor blk.46.nextn.eh_proj.weight (size = 17825792 bytes) -- ignoring
model has unused tensor blk.46.nextn.embed_tokens.weight (size = 329777152 bytes) -- ignoring
model has unused tensor blk.46.nextn.enorm.weight (size = 16384 bytes) -- ignoring
model has unused tensor blk.46.nextn.hnorm.weight (size = 16384 bytes) -- ignoring
model has unused tensor blk.46.nextn.shared_head_head.weight (size = 329777152 bytes) -- ignoring
model has unused tensor blk.46.nextn.shared_head_norm.weight (size = 16384 bytes) -- ignoring
llm_load_tensors: offloading 47 repeating layers to GPU
llm_load_tensors: offloading non-repeating layers to GPU
llm_load_tensors: offloaded 48/48 layers to GPU
llm_load_tensors:  CUDA_Host buffer size =   407.00 MiB
llm_load_tensors:      CUDA0 buffer size = 63980.86 MiB
....................................................................................................
llama_new_context_with_model: n_ctx      = 131072
llama_new_context_with_model: n_batch    = 4096
llama_new_context_with_model: n_ubatch   = 4096
llama_new_context_with_model: flash_attn = 1
llama_new_context_with_model: mla_attn   = 0
llama_new_context_with_model: attn_max_b = 0
llama_new_context_with_model: fused_moe  = 0
llama_new_context_with_model: ser        = -1, 0
llama_new_context_with_model: freq_base  = 1000000.0
llama_new_context_with_model: freq_scale = 1
llama_kv_cache_init:      CUDA0 KV buffer size = 24064.00 MiB
llama_new_context_with_model: KV self size  = 24064.00 MiB, K (f16): 12032.00 MiB, V (f16): 12032.00 MiB
llama_new_context_with_model:  CUDA_Host  output buffer size =     1.16 MiB
llama_new_context_with_model:      CUDA0 compute buffer size =  3584.03 MiB
llama_new_context_with_model:  CUDA_Host compute buffer size =  2112.05 MiB
llama_new_context_with_model: graph nodes  = 2239
llama_new_context_with_model: graph splits = 2
CUDA error: operation not permitted when stream is capturing
  current device: 0, in function prepare_row_mappigs at D:\a\ik_llama.cpp\ik_llama.cpp\ggml\src\ggml-cuda.cu:2281
  cudaStreamSynchronize(stream)
D:\a\ik_llama.cpp\ik_llama.cpp\ggml\src\ggml-cuda.cu:114: CUDA error
```

Similar issue documented here: https://github.com/pytorch/pytorch/issues/87794

---

👤 **ikawrakow** commented on **2025-08-14** at **18:10:39**

@Thireus You need `-fmoe`.

That being said, I though that I had fixed it to work without `-fmoe` by disabling graphs. Apparently not. Which is good (we noticed the error, so you will rerun with the correct arguments), but also bad (clearly I'm not an AGI, or if I'm one, then clearly not good enough, so not quite there yet to be the next trillion dollar thing).

---

👤 **Thireus** commented on **2025-08-14** at **18:13:26**

@ikawrakow - haha, yes much better with  `-fmoe` thank you!

---

👤 **ikawrakow** commented on **2025-08-14** at **18:14:14**

@ubergarm Thanks for the benchmarks! Interesting that in your case the gain is quite a bit smaller compared to what I observe. Not really sure why. Model? GPU?, Driver? (although you are on 570, so newer driver than the 565 I have installed on the system where I tested)

---

👤 **ubergarm** commented on **2025-08-14** at **18:15:30**

Okay, seeing good uplift here with this PR running [ubergarm/Qwen3-30B-A3B-Thinking-2507-GGUF "pure" Q4_0](https://huggingface.co/ubergarm/Qwen3-30B-A3B-Thinking-2507-GGUF/blob/main/Qwen3-30B-A3B-Thinking-2507-Q4_0.gguf) full GPU offload on my 3090TI FE 24GB VRAM GPU with newer CUDA drivers on Arch Linux. (this is a rare mainline compatible quant I released mainly for testing vulkan)

In fact, now CUDA backend is faster TG than vulkan here on ik's fork again for this one.

<img width="2267" height="1110" alt="sweep-bench-ik_llama cpp-PR689" src="https://github.com/user-attachments/assets/1ea08be7-cef3-40a9-97ca-809bdfd9dd52" />

<details>

<summary>👈 Details</summary>

```bash
./build/bin/llama-sweep-bench \
    --model "$model" \
    -fa \
    -c 20480 \
    -ngl 99 \
    --threads 1 \
     -ub 4096 -b 4096 \
    --warmup-batch
```

## ik PR CUDA -ub 4096 -b 4096 -fa -fmoe
|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  4096 |   1024 |      0 |    0.825 |  4967.56 |    6.030 |   169.81 |
|  4096 |   1024 |   4096 |    0.978 |  4188.51 |    6.645 |   154.11 |
|  4096 |   1024 |   8192 |    1.137 |  3603.97 |    7.389 |   138.59 |
|  4096 |   1024 |  12288 |    1.307 |  3133.06 |    8.175 |   125.26 |
|  4096 |   1024 |  16384 |    1.475 |  2776.30 |    8.593 |   119.17 |

## ik main CUDA -ub 4096 -b 4096 -fa -fmoe
|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  4096 |   1024 |      0 |    0.832 |  4925.56 |    7.295 |   140.37 |
|  4096 |   1024 |   4096 |    0.974 |  4207.28 |    7.928 |   129.16 |
|  4096 |   1024 |   8192 |    1.126 |  3637.96 |    8.637 |   118.56 |
|  4096 |   1024 |  12288 |    1.287 |  3183.04 |    9.443 |   108.44 |
|  4096 |   1024 |  16384 |    1.448 |  2828.99 |    9.888 |   103.56 |

## ik main Vulkan NV_coopmat2 -ub 4096 -b 4096 -fa
|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  4096 |   1024 |      0 |    2.197 |  1864.44 |    6.689 |   153.09 |
|  4096 |   1024 |   4096 |    2.361 |  1734.73 |    7.290 |   140.47 |
|  4096 |   1024 |   8192 |    2.550 |  1606.27 |    7.841 |   130.60 |
|  4096 |   1024 |  12288 |    2.763 |  1482.42 |    8.385 |   122.12 |
|  4096 |   1024 |  16384 |    2.984 |  1372.43 |    8.899 |   115.06 |

## mainline lcpp CUDA master@7a0de960 + ug/port-sweep-bench -ub 4096 -b 4096 -fa
|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  4096 |   1024 |      0 |    1.184 |  3458.00 |    6.796 |   150.68 |
|  4096 |   1024 |   4096 |    1.342 |  3052.62 |    7.336 |   139.59 |
|  4096 |   1024 |   8192 |    1.487 |  2754.18 |    8.013 |   127.79 |
|  4096 |   1024 |  12288 |    1.705 |  2401.73 |    8.794 |   116.44 |
|  4096 |   1024 |  16384 |    1.857 |  2206.05 |    9.215 |   111.13 |

## mainline lcpp Vulkan NV_coopmat2 master@7a0de960 + ug/port-sweep-bench -ub 4096 -b 4096 -fa
|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  4096 |   1024 |      0 |    2.219 |  1845.65 |    6.939 |   147.57 |
|  4096 |   1024 |   4096 |    2.411 |  1698.73 |    7.527 |   136.05 |
|  4096 |   1024 |   8192 |    2.625 |  1560.57 |    8.107 |   126.30 |
|  4096 |   1024 |  12288 |    2.842 |  1441.47 |    8.645 |   118.45 |
|  4096 |   1024 |  16384 |    3.068 |  1334.86 |    9.206 |   111.23 |

</details>

---

👤 **ikawrakow** commented on **2025-08-14** at **18:27:08**

Btw, there is [this issue](https://github.com/ggml-org/llama.cpp/issues/15297) in mainline, which is quite interesting. Somebody has noticed that `llama-bench` always uses the exact same tokens. This leads to exactly the same set of experts being activated for MoE models in each run. Which means that `llama-bench` results are potentially completely useless for MoE models. It may also mean that `ik_llama.cpp` may be in a huge disadvantage compared to mainline because in `ik_llama.cpp` there is a warm-up where **all experts get used**, which discards the potential advantage of having the fixed set of used experts stay in the cache(s). This is not done in mainline, where the warm-up run is with the **same tokens** as used later in the measurement runs. 

So, I think, one definitely needs `sweep-bench` results instead of `llama-bench`.

---

👤 **Thireus** commented on **2025-08-14** at **19:07:32**

Some stats at large context length. It looks like CUDA graphs are bringing a +9% TG speed increase for ik_llama.cpp. The gap with llama.cpp is quite noticeable though.

llama.cpp (main) - Windows builds: [b6168](https://github.com/Thireus/llama.cpp/releases/tag/b6168)

<details>

```
$ CUDA_DEVICE_ORDER=PCI_BUS_ID CUDA_VISIBLE_DEVICES=0 ~/llama-b6168-bin-win-cuda-12.8-x64/llama-server -m GLM-4.5-Air-THIREUS-BF16-SPECIAL_TENSOR-00001-of-00804.gguf \
 -fa \
  -ctk f16 \
  -c 131072 \
  -ngl 99 \
  -b 4096 -ub 4096 \
  --no-mmap \
  --threads 36 \
  --main-gpu 0
ggml_cuda_init: GGML_CUDA_FORCE_MMQ:    no
ggml_cuda_init: GGML_CUDA_FORCE_CUBLAS: no
ggml_cuda_init: found 1 CUDA devices:
  Device 0: NVIDIA RTX PRO 6000 Blackwell Workstation Edition, compute capability 12.0, VMM: yes
load_backend: loaded CUDA backend from C:\cygwin64\home\Thireus\llama-b6168-bin-win-cuda-12.8-x64\ggml-cuda.dll
load_backend: loaded RPC backend from C:\cygwin64\home\Thireus\llama-b6168-bin-win-cuda-12.8-x64\ggml-rpc.dll
load_backend: loaded CPU backend from C:\cygwin64\home\Thireus\llama-b6168-bin-win-cuda-12.8-x64\ggml-cpu-skylakex.dll
build: 6168 (092fd845) with clang version 19.1.5 for x86_64-pc-windows-msvc
system info: n_threads = 36, n_threads_batch = 36, total_threads = 36

system_info: n_threads = 36 (n_threads_batch = 36) / 36 | CUDA : ARCHS = 500,610,700,750,800,860,890,1200 | USE_GRAPHS = 1 | PEER_MAX_BATCH_SIZE = 128 | CPU : SSE3 = 1 | SSSE3 = 1 | AVX = 1 | AVX2 = 1 | F16C = 1 | FMA = 1 | BMI2 = 1 | AVX512 = 1 | LLAMAFILE = 1 | OPENMP = 1 | REPACK = 1 |

main: binding port with default address family
main: HTTP server is listening, hostname: 127.0.0.1, port: 8080, http threads: 35
main: loading model
srv    load_model: loading model 'GLM-4.5-Air-THIREUS-BF16-SPECIAL_TENSOR-00001-of-00804.gguf'
llama_model_load_from_file_impl: using device CUDA0 (NVIDIA RTX PRO 6000 Blackwell Workstation Edition) - 95288 MiB free
llama_model_loader: max stdio successfully set to 2048
llama_model_loader: additional 803 GGUFs metadata loaded.
llama_model_loader: loaded meta data with 44 key-value pairs and 803 tensors from GLM-4.5-Air-THIREUS-BF16-SPECIAL_TENSOR-00001-of-00804.gguf (version GGUF V3 (latest))
llama_model_loader: Dumping metadata keys/values. Note: KV overrides do not apply in this output.
llama_model_loader: - kv   0:                       general.architecture str              = glm4moe
llama_model_loader: - kv   1:                               general.type str              = model
llama_model_loader: - kv   2:                               general.name str              = GLM 4.5 Air
llama_model_loader: - kv   3:                         general.size_label str              = 128x9.4B
llama_model_loader: - kv   4:                            general.license str              = mit
llama_model_loader: - kv   5:                               general.tags arr[str,1]       = ["text-generation"]
llama_model_loader: - kv   6:                          general.languages arr[str,2]       = ["en", "zh"]
llama_model_loader: - kv   7:                        glm4moe.block_count u32              = 47
llama_model_loader: - kv   8:                     glm4moe.context_length u32              = 131072
llama_model_loader: - kv   9:                   glm4moe.embedding_length u32              = 4096
llama_model_loader: - kv  10:                glm4moe.feed_forward_length u32              = 10944
llama_model_loader: - kv  11:               glm4moe.attention.head_count u32              = 96
llama_model_loader: - kv  12:            glm4moe.attention.head_count_kv u32              = 8
llama_model_loader: - kv  13:                     glm4moe.rope.freq_base f32              = 1000000.000000
llama_model_loader: - kv  14:   glm4moe.attention.layer_norm_rms_epsilon f32              = 0.000010
llama_model_loader: - kv  15:                  glm4moe.expert_used_count u32              = 8
llama_model_loader: - kv  16:               glm4moe.attention.key_length u32              = 128
llama_model_loader: - kv  17:             glm4moe.attention.value_length u32              = 128
llama_model_loader: - kv  18:                          general.file_type u32              = 24
llama_model_loader: - kv  19:               glm4moe.rope.dimension_count u32              = 64
llama_model_loader: - kv  20:                       glm4moe.expert_count u32              = 128
llama_model_loader: - kv  21:         glm4moe.expert_feed_forward_length u32              = 1408
llama_model_loader: - kv  22:                glm4moe.expert_shared_count u32              = 1
llama_model_loader: - kv  23:          glm4moe.leading_dense_block_count u32              = 1
llama_model_loader: - kv  24:                 glm4moe.expert_gating_func u32              = 2
llama_model_loader: - kv  25:               glm4moe.expert_weights_scale f32              = 1.000000
llama_model_loader: - kv  26:                glm4moe.expert_weights_norm bool             = true
llama_model_loader: - kv  27:               glm4moe.nextn_predict_layers u32              = 1
llama_model_loader: - kv  28:                       tokenizer.ggml.model str              = gpt2
llama_model_loader: - kv  29:                         tokenizer.ggml.pre str              = glm4
llama_model_loader: - kv  30:                      tokenizer.ggml.tokens arr[str,151552]  = ["!", "\"", "#", "$", "%", "&", "'", ...
llama_model_loader: - kv  31:                  tokenizer.ggml.token_type arr[i32,151552]  = [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, ...  
llama_model_loader: - kv  32:                      tokenizer.ggml.merges arr[str,318088]  = ["─á ─á", "─á ─á─á─á", "─á─á ─á─á", "...
llama_model_loader: - kv  33:                tokenizer.ggml.eos_token_id u32              = 151329
llama_model_loader: - kv  34:            tokenizer.ggml.padding_token_id u32              = 151329
llama_model_loader: - kv  35:                tokenizer.ggml.bos_token_id u32              = 151331
llama_model_loader: - kv  36:                tokenizer.ggml.eot_token_id u32              = 151336
llama_model_loader: - kv  37:            tokenizer.ggml.unknown_token_id u32              = 151329
llama_model_loader: - kv  38:                tokenizer.ggml.eom_token_id u32              = 151338
llama_model_loader: - kv  39:                    tokenizer.chat_template str              = [gMASK]<sop>\n{%- if tools -%}\n<|syste...
llama_model_loader: - kv  40:               general.quantization_version u32              = 2
llama_model_loader: - kv  41:                                   split.no u16              = 0
llama_model_loader: - kv  42:                                split.count u16              = 804
llama_model_loader: - kv  43:                        split.tensors.count i32              = 803
llama_model_loader: - type  f32:  331 tensors
llama_model_loader: - type q8_0:  105 tensors
llama_model_loader: - type q5_K:   56 tensors
llama_model_loader: - type q6_K:   43 tensors
llama_model_loader: - type iq4_nl:   92 tensors
llama_model_loader: - type iq4_xs:  176 tensors
print_info: file format = GGUF V3 (latest)
print_info: file type   = IQ1_S - 1.5625 bpw
print_info: file size   = 65.12 GiB (5.06 BPW)
load: special_eot_id is not in special_eog_ids - the tokenizer config may be incorrect
load: special_eom_id is not in special_eog_ids - the tokenizer config may be incorrect
load: printing all EOG tokens:
load:   - 151329 ('<|endoftext|>')
load:   - 151336 ('<|user|>')
load:   - 151338 ('<|observation|>')
load: special tokens cache size = 36
load: token to piece cache size = 0.9713 MB
print_info: arch             = glm4moe
print_info: vocab_only       = 0
print_info: n_ctx_train      = 131072
print_info: n_embd           = 4096
print_info: n_layer          = 47
print_info: n_head           = 96
print_info: n_head_kv        = 8
print_info: n_rot            = 64
print_info: n_swa            = 0
print_info: is_swa_any       = 0
print_info: n_embd_head_k    = 128
print_info: n_embd_head_v    = 128
print_info: n_gqa            = 12
print_info: n_embd_k_gqa     = 1024
print_info: n_embd_v_gqa     = 1024
print_info: f_norm_eps       = 0.0e+00
print_info: f_norm_rms_eps   = 1.0e-05
print_info: f_clamp_kqv      = 0.0e+00
print_info: f_max_alibi_bias = 0.0e+00
print_info: f_logit_scale    = 0.0e+00
print_info: f_attn_scale     = 0.0e+00
print_info: n_ff             = 10944
print_info: n_expert         = 128
print_info: n_expert_used    = 8
print_info: causal attn      = 1
print_info: pooling type     = 0
print_info: rope type        = 2
print_info: rope scaling     = linear
print_info: freq_base_train  = 1000000.0
print_info: freq_scale_train = 1
print_info: n_ctx_orig_yarn  = 131072
print_info: rope_finetuned   = unknown
print_info: model type       = 106B.A12B
print_info: model params     = 110.47 B
print_info: general.name     = GLM 4.5 Air
print_info: vocab type       = BPE
print_info: n_vocab          = 151552
print_info: n_merges         = 318088
print_info: BOS token        = 151331 '[gMASK]'
print_info: EOS token        = 151329 '<|endoftext|>'
print_info: EOT token        = 151336 '<|user|>'
print_info: EOM token        = 151338 '<|observation|>'
print_info: UNK token        = 151329 '<|endoftext|>'
print_info: PAD token        = 151329 '<|endoftext|>'
print_info: LF token         = 198 '─è'
print_info: FIM PRE token    = 151347 '<|code_prefix|>'
print_info: FIM SUF token    = 151349 '<|code_suffix|>'
print_info: FIM MID token    = 151348 '<|code_middle|>'
print_info: EOG token        = 151329 '<|endoftext|>'
print_info: EOG token        = 151336 '<|user|>'
print_info: EOG token        = 151338 '<|observation|>'
print_info: max token length = 1024
load_tensors: loading model tensors, this can take a while... (mmap = false)
model has unused tensor blk.46.attn_norm.weight (size = 16384 bytes) -- ignoring
model has unused tensor blk.46.attn_q.weight (size = 26738688 bytes) -- ignoring
model has unused tensor blk.46.attn_k.weight (size = 2228224 bytes) -- ignoring
model has unused tensor blk.46.attn_v.weight (size = 2228224 bytes) -- ignoring
model has unused tensor blk.46.attn_q.bias (size = 49152 bytes) -- ignoring
model has unused tensor blk.46.attn_k.bias (size = 4096 bytes) -- ignoring
model has unused tensor blk.46.attn_v.bias (size = 4096 bytes) -- ignoring
model has unused tensor blk.46.attn_output.weight (size = 53477376 bytes) -- ignoring
model has unused tensor blk.46.post_attention_norm.weight (size = 16384 bytes) -- ignoring
model has unused tensor blk.46.ffn_gate_inp.weight (size = 2097152 bytes) -- ignoring
model has unused tensor blk.46.exp_probs_b.bias (size = 512 bytes) -- ignoring
model has unused tensor blk.46.ffn_gate_exps.weight (size = 605552640 bytes) -- ignoring
model has unused tensor blk.46.ffn_down_exps.weight (size = 415236096 bytes) -- ignoring
model has unused tensor blk.46.ffn_up_exps.weight (size = 605552640 bytes) -- ignoring
model has unused tensor blk.46.ffn_gate_shexp.weight (size = 4730880 bytes) -- ignoring
model has unused tensor blk.46.ffn_down_shexp.weight (size = 3244032 bytes) -- ignoring
model has unused tensor blk.46.ffn_up_shexp.weight (size = 4730880 bytes) -- ignoring
model has unused tensor blk.46.nextn.eh_proj.weight (size = 17825792 bytes) -- ignoring
model has unused tensor blk.46.nextn.embed_tokens.weight (size = 329777152 bytes) -- ignoring
model has unused tensor blk.46.nextn.enorm.weight (size = 16384 bytes) -- ignoring
model has unused tensor blk.46.nextn.hnorm.weight (size = 16384 bytes) -- ignoring
model has unused tensor blk.46.nextn.shared_head_head.weight (size = 329777152 bytes) -- ignoring
model has unused tensor blk.46.nextn.shared_head_norm.weight (size = 16384 bytes) -- ignoring
load_tensors: offloading 47 repeating layers to GPU
load_tensors: offloading output layer to GPU
load_tensors: offloaded 48/48 layers to GPU
load_tensors:        CUDA0 model buffer size = 63980.86 MiB
load_tensors:          CPU model buffer size =   407.00 MiB
....................................................................................................
llama_context: constructing llama_context
llama_context: n_seq_max     = 1
llama_context: n_ctx         = 131072
llama_context: n_ctx_per_seq = 131072
llama_context: n_batch       = 4096
llama_context: n_ubatch      = 4096
llama_context: causal_attn   = 1
llama_context: flash_attn    = 1
llama_context: kv_unified    = false
llama_context: freq_base     = 1000000.0
llama_context: freq_scale    = 1
llama_context:  CUDA_Host  output buffer size =     0.58 MiB
llama_kv_cache_unified:      CUDA0 KV buffer size = 23552.00 MiB
llama_kv_cache_unified: size = 23552.00 MiB (131072 cells,  46 layers,  1/1 seqs), K (f16): 11776.00 MiB, V (f16): 11776.00 MiB
llama_context:      CUDA0 compute buffer size =  3584.09 MiB
llama_context:  CUDA_Host compute buffer size =  2112.11 MiB
llama_context: graph nodes  = 3101
llama_context: graph splits = 2
common_init_from_params: added <|endoftext|> logit bias = -inf
common_init_from_params: added <|user|> logit bias = -inf
common_init_from_params: added <|observation|> logit bias = -inf
common_init_from_params: setting dry_penalty_last_n to ctx_size = 131072
common_init_from_params: warming up the model with an empty run - please wait ... (--no-warmup to disable)
srv          init: initializing slots, n_slots = 1
slot         init: id  0 | task -1 | new slot n_ctx_slot = 131072
main: model loaded
main: chat template, chat_template: [gMASK]<sop>
{%- if tools -%}
<|system|>
# Tools

You may call one or more functions to assist with the user query.

You are provided with function signatures within <tools></tools> XML tags:
<tools>
{% for tool in tools %}
{{ tool | tojson(ensure_ascii=False) }}
{% endfor %}
</tools>

For each function call, output the function name and arguments within the following XML format:
<tool_call>{function-name}
<arg_key>{arg-key-1}</arg_key>
<arg_value>{arg-value-1}</arg_value>
<arg_key>{arg-key-2}</arg_key>
<arg_value>{arg-value-2}</arg_value>
...
</tool_call>{%- endif -%}
{%- macro visible_text(content) -%}
    {%- if content is string -%}
        {{- content }}
    {%- elif content is iterable and content is not mapping -%}
        {%- for item in content -%}
            {%- if item is mapping and item.type == 'text' -%}
                {{- item.text }}
            {%- elif item is string -%}
                {{- item }}
            {%- endif -%}
        {%- endfor -%}
    {%- else -%}
        {{- content }}
    {%- endif -%}
{%- endmacro -%}
{%- set ns = namespace(last_user_index=-1) %}
{%- for m in messages %}
    {%- if m.role == 'user' %}
        {% set ns.last_user_index = loop.index0 -%}
    {%- endif %}
{%- endfor %}
{% for m in messages %}
{%- if m.role == 'user' -%}<|user|>
{% set content = visible_text(m.content) %}{{ content }}
{{- '/nothink' if (enable_thinking is defined and not enable_thinking and not content.endswith("/nothink")) else '' -}}
{%- elif m.role == 'assistant' -%}
<|assistant|>
{%- set reasoning_content = '' %}
{%- set content = visible_text(m.content) %}
{%- if m.reasoning_content is string %}
    {%- set reasoning_content = m.reasoning_content %}
{%- else %}
    {%- if '</think>' in content %}
        {%- set reasoning_content = content.split('</think>')[0].rstrip('\n').split('<think>')[-1].lstrip('\n') %}
        {%- set content = content.split('</think>')[-1].lstrip('\n') %}
    {%- endif %}
{%- endif %}
{%- if loop.index0 > ns.last_user_index and reasoning_content -%}
{{ '\n<think>' + reasoning_content.strip() +  '</think>'}}
{%- else -%}
{{ '\n<think></think>' }}
{%- endif -%}
{%- if content.strip() -%}
{{ '\n' + content.strip() }}
{%- endif -%}
{% if m.tool_calls %}
{% for tc in m.tool_calls %}
{%- if tc.function %}
    {%- set tc = tc.function %}
{%- endif %}
{{ '\n<tool_call>' + tc.name }}
{% set _args = tc.arguments %}
{% for k, v in _args.items() %}
<arg_key>{{ k }}</arg_key>
<arg_value>{{ v | tojson(ensure_ascii=False) if v is not string else v }}</arg_value>
{% endfor %}
</tool_call>{% endfor %}
{% endif %}
{%- elif m.role == 'tool' -%}
{%- if m.content is string -%}
{%- if loop.first or (messages[loop.index0 - 1].role != "tool") %}
    {{- '<|observation|>' }}
{%- endif %}
{{- '\n<tool_response>\n' }}
{{- m.content }}
{{- '\n</tool_response>' }}
{%- else -%}
<|observation|>{% for tr in m.content %}

<tool_response>
{{ tr.output if tr.output is defined else tr }}
</tool_response>{% endfor -%}
{% endif -%}
{%- elif m.role == 'system' -%}
<|system|>
{{ visible_text(m.content) }}
{%- endif -%}
{%- endfor -%}
{%- if add_generation_prompt -%}
    <|assistant|>{{- '\n<think></think>' if (enable_thinking is defined and not enable_thinking) else '' -}}
{%- endif -%}, example_format: '[gMASK]<sop><|system|>
You are a helpful assistant<|user|>
Hello<|assistant|>
Hi there<|user|>
How are you?<|assistant|>
'
main: server is listening on http://127.0.0.1:8080 - starting the main loop
srv  update_slots: all slots are idle
srv  log_server_r: request: GET / 127.0.0.1 200
srv  log_server_r: request: GET /props 127.0.0.1 200
srv  log_server_r: request: GET /favicon.ico 127.0.0.1 404
srv  params_from_: Chat format: Content-only
slot launch_slot_: id  0 | task 0 | processing task
slot update_slots: id  0 | task 0 | new prompt, n_ctx_slot = 131072, n_keep = 0, n_prompt_tokens = 104305
slot update_slots: id  0 | task 0 | kv cache rm [0, end)
slot update_slots: id  0 | task 0 | prompt processing progress, n_past = 4096, n_tokens = 4096, progress = 0.039269
slot update_slots: id  0 | task 0 | kv cache rm [4096, end)
slot update_slots: id  0 | task 0 | prompt processing progress, n_past = 8192, n_tokens = 4096, progress = 0.078539
slot update_slots: id  0 | task 0 | kv cache rm [8192, end)
slot update_slots: id  0 | task 0 | prompt processing progress, n_past = 12288, n_tokens = 4096, progress = 0.117808
slot update_slots: id  0 | task 0 | kv cache rm [12288, end)
slot update_slots: id  0 | task 0 | prompt processing progress, n_past = 16384, n_tokens = 4096, progress = 0.157078
slot update_slots: id  0 | task 0 | kv cache rm [16384, end)
slot update_slots: id  0 | task 0 | prompt processing progress, n_past = 20480, n_tokens = 4096, progress = 0.196347
slot update_slots: id  0 | task 0 | kv cache rm [20480, end)
slot update_slots: id  0 | task 0 | prompt processing progress, n_past = 24576, n_tokens = 4096, progress = 0.235617
slot update_slots: id  0 | task 0 | kv cache rm [24576, end)
slot update_slots: id  0 | task 0 | prompt processing progress, n_past = 28672, n_tokens = 4096, progress = 0.274886
slot update_slots: id  0 | task 0 | kv cache rm [28672, end)
slot update_slots: id  0 | task 0 | prompt processing progress, n_past = 32768, n_tokens = 4096, progress = 0.314156
slot update_slots: id  0 | task 0 | kv cache rm [32768, end)
slot update_slots: id  0 | task 0 | prompt processing progress, n_past = 36864, n_tokens = 4096, progress = 0.353425
slot update_slots: id  0 | task 0 | kv cache rm [36864, end)
slot update_slots: id  0 | task 0 | prompt processing progress, n_past = 40960, n_tokens = 4096, progress = 0.392695
slot update_slots: id  0 | task 0 | kv cache rm [40960, end)
slot update_slots: id  0 | task 0 | prompt processing progress, n_past = 45056, n_tokens = 4096, progress = 0.431964
slot update_slots: id  0 | task 0 | kv cache rm [45056, end)
slot update_slots: id  0 | task 0 | prompt processing progress, n_past = 49152, n_tokens = 4096, progress = 0.471233
slot update_slots: id  0 | task 0 | kv cache rm [49152, end)
slot update_slots: id  0 | task 0 | prompt processing progress, n_past = 53248, n_tokens = 4096, progress = 0.510503
slot update_slots: id  0 | task 0 | kv cache rm [53248, end)
slot update_slots: id  0 | task 0 | prompt processing progress, n_past = 57344, n_tokens = 4096, progress = 0.549772
slot update_slots: id  0 | task 0 | kv cache rm [57344, end)
slot update_slots: id  0 | task 0 | prompt processing progress, n_past = 61440, n_tokens = 4096, progress = 0.589042
slot update_slots: id  0 | task 0 | kv cache rm [61440, end)
slot update_slots: id  0 | task 0 | prompt processing progress, n_past = 65536, n_tokens = 4096, progress = 0.628311
slot update_slots: id  0 | task 0 | kv cache rm [65536, end)
slot update_slots: id  0 | task 0 | prompt processing progress, n_past = 69632, n_tokens = 4096, progress = 0.667581
slot update_slots: id  0 | task 0 | kv cache rm [69632, end)
slot update_slots: id  0 | task 0 | prompt processing progress, n_past = 73728, n_tokens = 4096, progress = 0.706850
slot update_slots: id  0 | task 0 | kv cache rm [73728, end)
slot update_slots: id  0 | task 0 | prompt processing progress, n_past = 77824, n_tokens = 4096, progress = 0.746120
slot update_slots: id  0 | task 0 | kv cache rm [77824, end)
slot update_slots: id  0 | task 0 | prompt processing progress, n_past = 81920, n_tokens = 4096, progress = 0.785389
slot update_slots: id  0 | task 0 | kv cache rm [81920, end)
slot update_slots: id  0 | task 0 | prompt processing progress, n_past = 86016, n_tokens = 4096, progress = 0.824658
slot update_slots: id  0 | task 0 | kv cache rm [86016, end)
slot update_slots: id  0 | task 0 | prompt processing progress, n_past = 90112, n_tokens = 4096, progress = 0.863928
slot update_slots: id  0 | task 0 | kv cache rm [90112, end)
slot update_slots: id  0 | task 0 | prompt processing progress, n_past = 94208, n_tokens = 4096, progress = 0.903197
slot update_slots: id  0 | task 0 | kv cache rm [94208, end)
slot update_slots: id  0 | task 0 | prompt processing progress, n_past = 98304, n_tokens = 4096, progress = 0.942467
slot update_slots: id  0 | task 0 | kv cache rm [98304, end)
slot update_slots: id  0 | task 0 | prompt processing progress, n_past = 102400, n_tokens = 4096, progress = 0.981736
slot update_slots: id  0 | task 0 | kv cache rm [102400, end)
slot update_slots: id  0 | task 0 | prompt processing progress, n_past = 104305, n_tokens = 1905, progress = 1.000000
slot update_slots: id  0 | task 0 | prompt done, n_past = 104305, n_tokens = 1905
slot      release: id  0 | task 0 | stop processing: n_past = 105118, truncated = 0
slot print_timing: id  0 | task 0 |
prompt eval time =  116916.81 ms / 104305 tokens (    1.12 ms per token,   892.13 tokens per second)
       eval time =   41511.79 ms /   814 tokens (   51.00 ms per token,    19.61 tokens per second)
      total time =  158428.61 ms / 105119 tokens
srv  update_slots: all slots are idle
srv  log_server_r: request: POST /v1/chat/completions 127.0.0.1 200
srv    operator(): operator(): cleaning up before exit...
```

</details>

# Large Prompt - Round 1:
```
Prompt
- Tokens: 104305
- Time: 116916.811 ms
- Speed: 892.1 t/s
Generation
- Tokens: 814
- Time: 41511.795 ms
- Speed: 19.6 t/s
```

# Large Prompt - Round 2:
```
Prompt
- Tokens: 104297
- Time: 121574.305 ms
- Speed: 857.9 t/s
Generation
- Tokens: 690
- Time: 35140.18 ms
- Speed: 19.6 t/s
```

ik_llama.cpp (main) - Windows builds: [main-b4074-62ef02e](https://github.com/Thireus/ik_llama.cpp/releases/tag/main-b4074-62ef02e)

<details>

```
$ CUDA_DEVICE_ORDER=PCI_BUS_ID CUDA_VISIBLE_DEVICES=0 ~/ik_llama-main-b4074-62ef02e-bin-win-cuda-12.8-x64-avx512/llama-server -m GLM-4.5-Air-THIREUS-BF16-SPECIAL_TENSOR-00001-of-00804.gguf -fa   -ctk f16   -c 131072   -ngl 99   -b 4096 -ub 4096   --no-mmap   --threads 36   --main-gpu 0 -fmoe
INFO [                    main] build info | tid="25220" timestamp=1755197623 build=1 commit="62ef02e"
INFO [                    main] system info | tid="25220" timestamp=1755197623 n_threads=36 n_threads_batch=-1 total_threads=36 system_info="AVX = 1 | AVX_VNNI = 0 | AVX2 = 1 | AVX512 = 1 | AVX512_VBMI = 0 | AVX512_VNNI = 0 | AVX512_BF16 = 0 | FMA = 1 | NEON = 0 | SVE = 0 | ARM_FMA = 0 | F16C = 1 | FP16_VA = 0 | WASM_SIMD = 0 | BLAS = 1 | SSE3 = 1 | SSSE3 = 1 | VSX = 0 | MATMUL_INT8 = 0 | LLAMAFILE = 1 | "
llama_model_loader: max stdio successfully set to 2048
llama_model_loader: additional 803 GGUFs metadata loaded.
llama_model_loader: loaded meta data with 44 key-value pairs and 803 tensors from GLM-4.5-Air-THIREUS-BF16-SPECIAL_TENSOR-00001-of-00804.gguf (version GGUF V3 (latest))
llama_model_loader: Dumping metadata keys/values. Note: KV overrides do not apply in this output.
llama_model_loader: - kv   0:                       general.architecture str              = glm4moe
llama_model_loader: - kv   1:                               general.type str              = model
llama_model_loader: - kv   2:                               general.name str              = GLM 4.5 Air
llama_model_loader: - kv   3:                         general.size_label str              = 128x9.4B
llama_model_loader: - kv   4:                            general.license str              = mit
llama_model_loader: - kv   5:                               general.tags arr[str,1]       = ["text-generation"]
llama_model_loader: - kv   6:                          general.languages arr[str,2]       = ["en", "zh"]
llama_model_loader: - kv   7:                        glm4moe.block_count u32              = 47
llama_model_loader: - kv   8:                     glm4moe.context_length u32              = 131072
llama_model_loader: - kv   9:                   glm4moe.embedding_length u32              = 4096
llama_model_loader: - kv  10:                glm4moe.feed_forward_length u32              = 10944
llama_model_loader: - kv  11:               glm4moe.attention.head_count u32              = 96
llama_model_loader: - kv  12:            glm4moe.attention.head_count_kv u32              = 8
llama_model_loader: - kv  13:                     glm4moe.rope.freq_base f32              = 1000000.000000
llama_model_loader: - kv  14:   glm4moe.attention.layer_norm_rms_epsilon f32              = 0.000010
llama_model_loader: - kv  15:                  glm4moe.expert_used_count u32              = 8
llama_model_loader: - kv  16:               glm4moe.attention.key_length u32              = 128
llama_model_loader: - kv  17:             glm4moe.attention.value_length u32              = 128
llama_model_loader: - kv  18:                          general.file_type u32              = 24
llama_model_loader: - kv  19:               glm4moe.rope.dimension_count u32              = 64
llama_model_loader: - kv  20:                       glm4moe.expert_count u32              = 128
llama_model_loader: - kv  21:         glm4moe.expert_feed_forward_length u32              = 1408
llama_model_loader: - kv  22:                glm4moe.expert_shared_count u32              = 1
llama_model_loader: - kv  23:          glm4moe.leading_dense_block_count u32              = 1
llama_model_loader: - kv  24:                 glm4moe.expert_gating_func u32              = 2
llama_model_loader: - kv  25:               glm4moe.expert_weights_scale f32              = 1.000000
llama_model_loader: - kv  26:                glm4moe.expert_weights_norm bool             = true
llama_model_loader: - kv  27:               glm4moe.nextn_predict_layers u32              = 1
llama_model_loader: - kv  28:                       tokenizer.ggml.model str              = gpt2
llama_model_loader: - kv  29:                         tokenizer.ggml.pre str              = glm4
llama_model_loader: - kv  30:                      tokenizer.ggml.tokens arr[str,151552]  = ["!", "\"", "#", "$", "%", "&", "'", ...
llama_model_loader: - kv  31:                  tokenizer.ggml.token_type arr[i32,151552]  = [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, ...
llama_model_loader: - kv  32:                      tokenizer.ggml.merges arr[str,318088]  = ["─á ─á", "─á ─á─á─á", "─á─á ─á─á", "...
llama_model_loader: - kv  33:                tokenizer.ggml.eos_token_id u32              = 151329
llama_model_loader: - kv  34:            tokenizer.ggml.padding_token_id u32              = 151329
llama_model_loader: - kv  35:                tokenizer.ggml.bos_token_id u32              = 151331
llama_model_loader: - kv  36:                tokenizer.ggml.eot_token_id u32              = 151336
llama_model_loader: - kv  37:            tokenizer.ggml.unknown_token_id u32              = 151329
llama_model_loader: - kv  38:                tokenizer.ggml.eom_token_id u32              = 151338
llama_model_loader: - kv  39:                    tokenizer.chat_template str              = [gMASK]<sop>\n{%- if tools -%}\n<|syste...
llama_model_loader: - kv  40:               general.quantization_version u32              = 2
llama_model_loader: - kv  41:                                   split.no u16              = 0
llama_model_loader: - kv  42:                                split.count u16              = 804
llama_model_loader: - kv  43:                        split.tensors.count i32              = 803
llama_model_loader: - type  f32:  331 tensors
llama_model_loader: - type q8_0:  105 tensors
llama_model_loader: - type q5_K:   56 tensors
llama_model_loader: - type q6_K:   43 tensors
llama_model_loader: - type iq4_nl:   92 tensors
llama_model_loader: - type iq4_xs:  176 tensors
llm_load_vocab: special tokens cache size = 36
llm_load_vocab: token to piece cache size = 0.9713 MB
llm_load_print_meta: format           = GGUF V3 (latest)
llm_load_print_meta: arch             = glm4moe
llm_load_print_meta: vocab type       = BPE
llm_load_print_meta: n_vocab          = 151552
llm_load_print_meta: n_merges         = 318088
llm_load_print_meta: vocab_only       = 0
llm_load_print_meta: n_ctx_train      = 131072
llm_load_print_meta: n_embd           = 4096
llm_load_print_meta: n_layer          = 47
llm_load_print_meta: n_head           = 96
llm_load_print_meta: n_head_kv        = 8
llm_load_print_meta: n_rot            = 64
llm_load_print_meta: n_swa            = 0
llm_load_print_meta: n_swa_pattern    = 1
llm_load_print_meta: n_embd_head_k    = 128
llm_load_print_meta: n_embd_head_v    = 128
llm_load_print_meta: n_gqa            = 12
llm_load_print_meta: n_embd_k_gqa     = 1024
llm_load_print_meta: n_embd_v_gqa     = 1024
llm_load_print_meta: f_norm_eps       = 0.0e+00
llm_load_print_meta: f_norm_rms_eps   = 1.0e-05
llm_load_print_meta: f_clamp_kqv      = 0.0e+00
llm_load_print_meta: f_max_alibi_bias = 0.0e+00
llm_load_print_meta: f_logit_scale    = 0.0e+00
llm_load_print_meta: n_ff             = 10944
llm_load_print_meta: n_expert         = 128
llm_load_print_meta: n_expert_used    = 8
llm_load_print_meta: causal attn      = 1
llm_load_print_meta: pooling type     = 0
llm_load_print_meta: rope type        = 2
llm_load_print_meta: rope scaling     = linear
llm_load_print_meta: freq_base_train  = 1000000.0
llm_load_print_meta: freq_scale_train = 1
llm_load_print_meta: n_ctx_orig_yarn  = 131072
llm_load_print_meta: rope_finetuned   = unknown
llm_load_print_meta: ssm_d_conv       = 0
llm_load_print_meta: ssm_d_inner      = 0
llm_load_print_meta: ssm_d_state      = 0
llm_load_print_meta: ssm_dt_rank      = 0
llm_load_print_meta: model type       = 106B.A12B
llm_load_print_meta: model ftype      = IQ1_S - 1.5625 bpw
llm_load_print_meta: model params     = 110.469 B
llm_load_print_meta: model size       = 65.117 GiB (5.063 BPW)
llm_load_print_meta: repeating layers = 64.105 GiB (5.041 BPW, 109.227 B parameters)
llm_load_print_meta: general.name     = GLM 4.5 Air
llm_load_print_meta: BOS token        = 151331 '[gMASK]'
llm_load_print_meta: EOS token        = 151329 '<|endoftext|>'
llm_load_print_meta: UNK token        = 151329 '<|endoftext|>'
llm_load_print_meta: PAD token        = 151329 '<|endoftext|>'
llm_load_print_meta: LF token         = 128 '├ä'
llm_load_print_meta: FIM PRE token    = 151347 '<|code_prefix|>'
llm_load_print_meta: FIM SUF token    = 151349 '<|code_suffix|>'
llm_load_print_meta: FIM MID token    = 151348 '<|code_middle|>'
llm_load_print_meta: EOT token        = 151336 '<|user|>'
llm_load_print_meta: max token length = 1024
ggml_cuda_init: GGML_CUDA_FORCE_MMQ:    no
ggml_cuda_init: GGML_CUDA_FORCE_CUBLAS: no
ggml_cuda_init: found 1 CUDA devices:
  Device 0: NVIDIA RTX PRO 6000 Blackwell Workstation Edition, compute capability 12.0, VMM: yes
llm_load_tensors: ggml ctx size =    0.66 MiB
model has unused tensor blk.46.attn_norm.weight (size = 16384 bytes) -- ignoring
model has unused tensor blk.46.attn_q.weight (size = 26738688 bytes) -- ignoring
model has unused tensor blk.46.attn_k.weight (size = 2228224 bytes) -- ignoring
model has unused tensor blk.46.attn_v.weight (size = 2228224 bytes) -- ignoring
model has unused tensor blk.46.attn_q.bias (size = 49152 bytes) -- ignoring
model has unused tensor blk.46.attn_k.bias (size = 4096 bytes) -- ignoring
model has unused tensor blk.46.attn_v.bias (size = 4096 bytes) -- ignoring
model has unused tensor blk.46.attn_output.weight (size = 53477376 bytes) -- ignoring
model has unused tensor blk.46.post_attention_norm.weight (size = 16384 bytes) -- ignoring
model has unused tensor blk.46.ffn_gate_inp.weight (size = 2097152 bytes) -- ignoring
model has unused tensor blk.46.exp_probs_b.bias (size = 512 bytes) -- ignoring
model has unused tensor blk.46.ffn_gate_exps.weight (size = 605552640 bytes) -- ignoring
model has unused tensor blk.46.ffn_down_exps.weight (size = 415236096 bytes) -- ignoring
model has unused tensor blk.46.ffn_up_exps.weight (size = 605552640 bytes) -- ignoring
model has unused tensor blk.46.ffn_gate_shexp.weight (size = 4730880 bytes) -- ignoring
model has unused tensor blk.46.ffn_down_shexp.weight (size = 3244032 bytes) -- ignoring
model has unused tensor blk.46.ffn_up_shexp.weight (size = 4730880 bytes) -- ignoring
model has unused tensor blk.46.nextn.eh_proj.weight (size = 17825792 bytes) -- ignoring
model has unused tensor blk.46.nextn.embed_tokens.weight (size = 329777152 bytes) -- ignoring
model has unused tensor blk.46.nextn.enorm.weight (size = 16384 bytes) -- ignoring
model has unused tensor blk.46.nextn.hnorm.weight (size = 16384 bytes) -- ignoring
model has unused tensor blk.46.nextn.shared_head_head.weight (size = 329777152 bytes) -- ignoring
model has unused tensor blk.46.nextn.shared_head_norm.weight (size = 16384 bytes) -- ignoring
llm_load_tensors: offloading 47 repeating layers to GPU
llm_load_tensors: offloading non-repeating layers to GPU
llm_load_tensors: offloaded 48/48 layers to GPU
llm_load_tensors:  CUDA_Host buffer size =   407.00 MiB
llm_load_tensors:      CUDA0 buffer size = 63980.86 MiB
....................................................................................................
llama_new_context_with_model: n_ctx      = 131072
llama_new_context_with_model: n_batch    = 4096
llama_new_context_with_model: n_ubatch   = 4096
llama_new_context_with_model: flash_attn = 1
llama_new_context_with_model: mla_attn   = 0
llama_new_context_with_model: attn_max_b = 0
llama_new_context_with_model: fused_moe  = 1
llama_new_context_with_model: ser        = -1, 0
llama_new_context_with_model: freq_base  = 1000000.0
llama_new_context_with_model: freq_scale = 1
llama_kv_cache_init:      CUDA0 KV buffer size = 24064.00 MiB
llama_new_context_with_model: KV self size  = 24064.00 MiB, K (f16): 12032.00 MiB, V (f16): 12032.00 MiB
llama_new_context_with_model:  CUDA_Host  output buffer size =     1.16 MiB
llama_new_context_with_model:      CUDA0 compute buffer size =  3584.03 MiB
llama_new_context_with_model:  CUDA_Host compute buffer size =  2112.05 MiB
llama_new_context_with_model: graph nodes  = 2149
llama_new_context_with_model: graph splits = 2
INFO [                    init] initializing slots | tid="25220" timestamp=1755197662 n_slots=1
INFO [                    init] new slot | tid="25220" timestamp=1755197662 id_slot=0 n_ctx_slot=131072
INFO [                    main] model loaded | tid="25220" timestamp=1755197662
INFO [                    main] chat template | tid="25220" timestamp=1755197662 chat_template="[gMASK]<sop>\n{%- if tools -%}\n<|system|>\n# Tools\n\nYou may call one or more functions to assist with the user query.\n\nYou are provided with function signatures within <tools></tools> XML tags:\n<tools>\n{% for tool in tools %}\n{{ tool | tojson(ensure_ascii=False) }}\n{% endfor %}\n</tools>\n\nFor each function call, output the function name and arguments within the following XML format:\n<tool_call>{function-name}\n<arg_key>{arg-key-1}</arg_key>\n<arg_value>{arg-value-1}</arg_value>\n<arg_key>{arg-key-2}</arg_key>\n<arg_value>{arg-value-2}</arg_value>\n...\n</tool_call>{%- endif -%}\n{%- macro visible_text(content) -%}\n    {%- if content is string -%}\n        {{- content }}\n    {%- elif content is iterable and content is not mapping -%}\n        {%- for item in content -%}\n            {%- if item is mapping and item.type == 'text' -%}\n                {{- item.text }}\n            {%- elif item is string -%}\n                {{- item }}\n            {%- endif -%}\n        {%- endfor -%}\n    {%- else -%}\n        {{- content }}\n    {%- endif -%}\n{%- endmacro -%}\n{%- set ns = namespace(last_user_index=-1) %}\n{%- for m in messages %}\n    {%- if m.role == 'user' %}\n        {% set ns.last_user_index = loop.index0 -%}\n    {%- endif %}\n{%- endfor %}\n{% for m in messages %}\n{%- if m.role == 'user' -%}<|user|>\n{% set content = visible_text(m.content) %}{{ content }}\n{{- '/nothink' if (enable_thinking is defined and not enable_thinking and not content.endswith(\"/nothink\")) else '' -}}\n{%- elif m.role == 'assistant' -%}\n<|assistant|>\n{%- set reasoning_content = '' %}\n{%- set content = visible_text(m.content) %}\n{%- if m.reasoning_content is string %}\n    {%- set reasoning_content = m.reasoning_content %}\n{%- else %}\n    {%- if '</think>' in content %}\n        {%- set reasoning_content = content.split('</think>')[0].rstrip('\\n').split('<think>')[-1].lstrip('\\n') %}\n        {%- set content = content.split('</think>')[-1].lstrip('\\n') %}\n    {%- endif %}\n{%- endif %}\n{%- if loop.index0 > ns.last_user_index and reasoning_content -%}\n{{ '\\n<think>' + reasoning_content.strip() +  '</think>'}}\n{%- else -%}\n{{ '\\n<think></think>' }}\n{%- endif -%}\n{%- if content.strip() -%}\n{{ '\\n' + content.strip() }}\n{%- endif -%}\n{% if m.tool_calls %}\n{% for tc in m.tool_calls %}\n{%- if tc.function %}\n    {%- set tc = tc.function %}\n{%- endif %}\n{{ '\\n<tool_call>' + tc.name }}\n{% set _args = tc.arguments %}\n{% for k, v in _args.items() %}\n<arg_key>{{ k }}</arg_key>\n<arg_value>{{ v | tojson(ensure_ascii=False) if v is not string else v }}</arg_value>\n{% endfor %}\n</tool_call>{% endfor %}\n{% endif %}\n{%- elif m.role == 'tool' -%}\n{%- if m.content is string -%}\n{%- if loop.first or (messages[loop.index0 - 1].role != \"tool\") %}\n    {{- '<|observation|>' }}\n{%- endif %}\n{{- '\\n<tool_response>\\n' }}\n{{- m.content }}\n{{- '\\n</tool_response>' }}\n{%- else -%}\n<|observation|>{% for tr in m.content %}\n\n<tool_response>\n{{ tr.output if tr.output is defined else tr }}\n</tool_response>{% endfor -%}\n{% endif -%}\n{%- elif m.role == 'system' -%}\n<|system|>\n{{ visible_text(m.content) }}\n{%- endif -%}\n{%- endfor -%}\n{%- if add_generation_prompt -%}\n    <|assistant|>{{- '\\n<think></think>' if (enable_thinking is defined and not enable_thinking) else '' -}}\n{%- endif -%}"
INFO [                    main] chat template | tid="25220" timestamp=1755197662 chat_example="[gMASK]<sop><|system|>\nYou are a helpful assistant<|user|>\nHello<|assistant|>\nHi there<|user|>\nHow are you?<|assistant|>" built_in=true
INFO [                    main] HTTP server listening | tid="25220" timestamp=1755197662 hostname="127.0.0.1" port="8080" n_threads_http="35"
INFO [            update_slots] all slots are idle | tid="25220" timestamp=1755197662
INFO [   launch_slot_with_task] slot is processing task | tid="25220" timestamp=1755197671 id_slot=0 id_task=0
INFO [            update_slots] kv cache rm [p0, end) | tid="25220" timestamp=1755197671 id_slot=0 id_task=0 p0=0
INFO [            update_slots] kv cache rm [p0, end) | tid="25220" timestamp=1755197673 id_slot=0 id_task=0 p0=4096
INFO [            update_slots] kv cache rm [p0, end) | tid="25220" timestamp=1755197675 id_slot=0 id_task=0 p0=8192
INFO [            update_slots] kv cache rm [p0, end) | tid="25220" timestamp=1755197677 id_slot=0 id_task=0 p0=12288
INFO [            update_slots] kv cache rm [p0, end) | tid="25220" timestamp=1755197679 id_slot=0 id_task=0 p0=16384
INFO [            update_slots] kv cache rm [p0, end) | tid="25220" timestamp=1755197681 id_slot=0 id_task=0 p0=20480
INFO [            update_slots] kv cache rm [p0, end) | tid="25220" timestamp=1755197684 id_slot=0 id_task=0 p0=24576
INFO [            update_slots] kv cache rm [p0, end) | tid="25220" timestamp=1755197687 id_slot=0 id_task=0 p0=28672
INFO [            update_slots] kv cache rm [p0, end) | tid="25220" timestamp=1755197690 id_slot=0 id_task=0 p0=32768
INFO [            update_slots] kv cache rm [p0, end) | tid="25220" timestamp=1755197694 id_slot=0 id_task=0 p0=36864
INFO [            update_slots] kv cache rm [p0, end) | tid="25220" timestamp=1755197698 id_slot=0 id_task=0 p0=40960
INFO [            update_slots] kv cache rm [p0, end) | tid="25220" timestamp=1755197702 id_slot=0 id_task=0 p0=45056
INFO [            update_slots] kv cache rm [p0, end) | tid="25220" timestamp=1755197707 id_slot=0 id_task=0 p0=49152
INFO [            update_slots] kv cache rm [p0, end) | tid="25220" timestamp=1755197712 id_slot=0 id_task=0 p0=53248
INFO [            update_slots] kv cache rm [p0, end) | tid="25220" timestamp=1755197718 id_slot=0 id_task=0 p0=57344
INFO [            update_slots] kv cache rm [p0, end) | tid="25220" timestamp=1755197723 id_slot=0 id_task=0 p0=61440
INFO [            update_slots] kv cache rm [p0, end) | tid="25220" timestamp=1755197730 id_slot=0 id_task=0 p0=65536
INFO [            update_slots] kv cache rm [p0, end) | tid="25220" timestamp=1755197736 id_slot=0 id_task=0 p0=69632
INFO [            update_slots] kv cache rm [p0, end) | tid="25220" timestamp=1755197743 id_slot=0 id_task=0 p0=73728
INFO [            update_slots] kv cache rm [p0, end) | tid="25220" timestamp=1755197750 id_slot=0 id_task=0 p0=77824
INFO [            update_slots] kv cache rm [p0, end) | tid="25220" timestamp=1755197757 id_slot=0 id_task=0 p0=81920
INFO [            update_slots] kv cache rm [p0, end) | tid="25220" timestamp=1755197765 id_slot=0 id_task=0 p0=86016
INFO [            update_slots] kv cache rm [p0, end) | tid="25220" timestamp=1755197773 id_slot=0 id_task=0 p0=90112
INFO [            update_slots] kv cache rm [p0, end) | tid="25220" timestamp=1755197782 id_slot=0 id_task=0 p0=94208
INFO [            update_slots] kv cache rm [p0, end) | tid="25220" timestamp=1755197791 id_slot=0 id_task=0 p0=98304
INFO [            update_slots] kv cache rm [p0, end) | tid="25220" timestamp=1755197800 id_slot=0 id_task=0 p0=102400
INFO [           print_timings] prompt eval time     =  134010.47 ms / 104304 tokens (    1.28 ms per token,   778.33 tokens per second) | tid="25220" timestamp=1755198235 id_slot=0 id_task=0 t_prompt_processing=134010.474 n_prompt_tokens_processed=104304 t_token=1.284806661297745 n_tokens_second=778.3272223930796
INFO [           print_timings] generation eval time =  430326.25 ms /  2456 runs   (  175.21 ms per token,     5.71 tokens per second) | tid="25220" timestamp=1755198235 id_slot=0 id_task=0 t_token_generation=430326.246 n_decoded=2456 t_token=175.21426954397393 n_tokens_second=5.707297713837328
INFO [           print_timings]           total time =  564336.72 ms | tid="25220" timestamp=1755198235 id_slot=0 id_task=0 t_prompt_processing=134010.474 t_token_generation=430326.246 t_total=564336.72
INFO [            update_slots] slot released | tid="25220" timestamp=1755198235 id_slot=0 id_task=0 n_ctx=131072 n_past=106759 n_system_tokens=0 n_cache_tokens=106759 truncated=false
INFO [            update_slots] all slots are idle | tid="25220" timestamp=1755198235
INFO [format_partial_response_oaicompat] DEBUG: Streaming finish_reason check | tid="12416" timestamp=1755198235 generated_text="\n<think>This is a straightforward question about experience points in DipiloBlop, and I need to use the provided book to find the answer. Let me analyze the relevant information:\n\n1. The book contains an experience table showing how much experience is needed to reach each level.\n\n2. I need to find out what level a player is at when they have half the experience needed to reach level 101 (max level).\n\n3. Looking at the experience table in the book:\n\n```\nLevel \tExperience needed to reach level \tExperience difference from last level\n1\t0\t0\n2\t266\t266\n3\t553\t287\n4\t888\t335\n5\t1,264\t376\n6\t1,697\t433\n7\t2,202\t505\n8\t2,761\t559\n9\t3,421\t660\n10\t4,162\t741\n11\t5,009\t847\n12\t5,987\t978\n13\t7,090\t1,103\n14\t8,339\t1,249\n15\t9,792\t1,453\n16\t11,423\t1,631\n17\t13,273\t1,850\n18\t15,397\t2,124\n19\t17,798\t2,401\n20\t20,537\t2,739\n21\t23,635\t3,098\n22\t27,183\t3,548\n23\t31,160\t3,977\n24\t35,720\t4,560\n25\t40,895\t5,175\n26\t46,750\t5,855\n27\t53,376\t6,626\n28\t60,943\t7,567\n29\t69,493\t8,550\n30\t79,195\t9,702\n31\t90,198\t11,003\n32\t102,691\t12,493\n33\t116,838\t14,147\n34\t132,892\t16,054\n35\t151,085\t18,193\n36\t171,731\t20,646\n37\t195,093\t23,362\n38\t221,634\t26,541\n39\t251,703\t30,069\n40\t285,768\t34,065\n41\t324,379\t38,611\n42\t368,189\t43,810\n43\t417,795\t49,606\n44\t474,049\t56,254\n45\t537,848\t63,799\n46\t610,073\t72,225\n47\t691,990\t81,917\n48\t784,837\t92,847\n49\t890,059\t105,222\n50\t1,009,301\t119,242\n51\t1,144,480\t135,179\n52\t1,297,644\t153,164\n53\t1,471,284\t173,640\n54\t1,667,999\t196,715\n55\t1,891,034\t223,035\n56\t2,143,788\t252,754\n57\t2,430,212\t286,424\n58\t2,754,802\t324,590\n59\t3,122,772\t367,970\n60\t3,539,721\t416,949\n61\t4,012,280\t472,559\n62\t4,547,846\t535,566\n63\t5,154,809\t606,963\n64\t5,842,739\t687,930\n65\t6,622,340\t779,601\n66\t7,505,874\t883,534\n67\t8,507,246\t1,001,372\n68\t9,642,117\t1,134,871\n69\t10,928,297\t1,286,180\n70\t12,385,868\t1,457,571\n71\t14,037,935\t1,652,067\n72\t15,910,153\t1,872,218\n73\t18,031,971\t2,121,818\n74\t20,436,698\t2,404,727\n75\t23,162,089\t2,725,391\n76\t26,250,771\t3,088,682\n77\t29,751,292\t3,500,521\n78\t33,718,506\t3,967,214\n79\t38,214,716\t4,496,210\n80\t43,310,285\t5,095,569\n81\t49,085,275\t5,774,990\n82\t55,630,259\t6,544,984\n83\t63,047,786\t7,417,527\n84\t71,454,370\t8,406,584\n85\t80,981,751\t9,527,381\n86\t91,779,442\t10,797,691\n87\t104,016,620\t12,237,178\n88\t117,885,480\t13,868,860\n89\t133,603,420\t15,717,940\n90\t151,417,175\t17,813,755\n91\t171,605,798\t20,188,623\n92\t194,486,167\t22,880,369\n93\t220,417,391\t25,931,224\n94\t249,805,802\t29,388,411\n95\t283,112,599\t33,306,797\n96\t320,860,372\t37,747,773\n97\t363,640,890\t42,780,518\n98\t412,125,351\t48,484,461\n99\t467,074,482\t54,949,131\n100\t507,038,931\t48,364,282\n101\t556,387,432\t49,348,501\n```\n\n4. The experience needed to reach level 101 is 556,387,432.\n\n5. Half of that experience is 556,387,432 ├╖ 2 = 278,193,716.\n\n6. Looking at the table, I need to find the level where a player first reaches or exceeds 278,193,716 experience points.\n\n7. Looking at the table:\n   - Level 75 requires 23,162,089 experience\n   - Level 76 requires 26,250,771 experience\n   - Level 77 requires 29,751,292 experience\n   - Level 78 requires 33,718,506 experience\n   - Level 79 requires 38,214,716 experience\n   - Level 80 requires 43,310,285 experience\n   - Level 81 requires 49,085,275 experience\n   - Level 82 requires 55,630,259 experience\n   - Level 83 requires 63,047,786 experience\n   - Level 84 requires 71,454,370 experience\n   - Level 85 requires 80,981,751 experience\n   - Level 86 requires 91,779,442 experience\n   - Level 87 requires 104,016,620 experience\n   - Level 88 requires 117,885,480 experience\n   - Level 89 requires 133,603,420 experience\n   - Level 90 requires 151,417,175 experience\n   - Level 91 requires 171,605,798 experience\n   - Level 92 requires 194,486,167 experience\n   - Level 93 requires 220,417,391 experience\n   - Level 94 requires 249,805,802 experience\n   - Level 95 requires 283,112,599 experience\n\n8. Half the experience needed for level 101 (278,193,716) is between the experience needed for level 94 (249,805,802) and level 95 (283,112,599).\n\n9. This means a player would be approximately level 94 when they have half the experience needed to reach level 101.\n\nTo be more precise, I can calculate the exact level by determining how far along between level 94 and 95 the player would be:\n- Experience needed for level 94: 249,805,802\n- Experience needed for level 95: 283,112,599\n- Difference: 283,112,599 - 249,805,802 = 33,306,797\n- How much more experience is needed to reach half the max level: 278,193,716 - 249,805,802 = 28,387,914\n- Progress between levels: 28,387,914 ├╖ 33,306,797 Γëê 0.852\n\nSo a player would be approximately level 94.852 when they have half the experience needed to reach level 101.\n\nSince we don't usually use decimal levels in games like DipiloBlop, we would say the player is approximately level 94 when they have about half the experience needed to reach the maximum level of 101.</think>Based on the experience table provided in the book, I can determine what level a player is at when they have half the experience needed to reach the maximum level of 101.\n\nThe experience needed to reach level 101 is 556,387,432. Half of this would be 278,193,716 experience points.\n\nLooking at the experience table:\n- Level 94 requires 249,805,802 experience\n- Level 95 requires 283,112,599 experience\n\nSince 278,193,716 (half of level 101) is between the experience required for levels 94 and 95, a player would be approximately level 94 when they reach half the experience needed to reach the maximum level.\n\nTo be more precise, a player would have about 85.2% of the way between levels 94 and 95 when they reach half the experience for level 101." model_name="gpt-3.5-turbo-0613" tool_calls_count=0
INFO [      log_server_request] request | tid="12416" timestamp=1755198235 remote_addr="127.0.0.1" remote_port=53332 status=200 method="POST" path="/v1/chat/completions" params={}
INFO [            update_slots] all slots are idle | tid="25220" timestamp=1755198235
INFO [            update_slots] all slots are idle | tid="25220" timestamp=1755198249
```

</details>

# Large Prompt - Round 1:
```
Prompt
- Tokens: 104304
- Time: 134010.474 ms
- Speed: 778.3 t/s
Generation
- Tokens: 2456
- Time: 430326.23 ms
- Speed: 5.7 t/s
```

# Large Prompt - Round 2:
```
Prompt
- Tokens: 104296
- Time: 140148.773 ms
- Speed: 744.2 t/s
Generation
- Tokens: 1758
- Time: 306114.491 ms
- Speed: 5.7 t/s
```

ik_llama.cpp (8a83e1f0830cd6b4448fd225892d870dd904ba7e) - Windows builds: [ik-try_cuda_graphs-b4107-7693263](https://github.com/Thireus/ik_llama.cpp/releases/tag/ik-try_cuda_graphs-b4107-7693263)

<details>

```
$ CUDA_DEVICE_ORDER=PCI_BUS_ID CUDA_VISIBLE_DEVICES=1 ~/ik_llama-ik-try_cuda_graphs-b4107-7693263-bin-win-cuda-12.8-x64-avx512/llama-server -m GLM-4.5-Air-THIREUS-BF16-SPECIAL_TENSOR-00001-of-00804.gguf -fa   -ctk f16   -c 131072   -ngl 99   -b 4096 -ub 4096   --no-mmap   --threads 36   --main-gpu 0 --port 8081 -fmoe
INFO [                    main] build info | tid="23312" timestamp=1755197037 build=1 commit="7693263"
INFO [                    main] system info | tid="23312" timestamp=1755197037 n_threads=36 n_threads_batch=-1 total_threads=36 system_info="AVX = 1 | AVX_VNNI = 0 | AVX2 = 1 | AVX512 = 1 | AVX512_VBMI = 0 | AVX512_VNNI = 0 | AVX512_BF16 = 0 | FMA = 1 | NEON = 0 | SVE = 0 | ARM_FMA = 0 | F16C = 1 | FP16_VA = 0 | WASM_SIMD = 0 | BLAS = 1 | SSE3 = 1 | SSSE3 = 1 | VSX = 0 | MATMUL_INT8 = 0 | LLAMAFILE = 1 | "
llama_model_loader: max stdio successfully set to 2048
llama_model_loader: additional 803 GGUFs metadata loaded.
llama_model_loader: loaded meta data with 44 key-value pairs and 803 tensors from GLM-4.5-Air-THIREUS-BF16-SPECIAL_TENSOR-00001-of-00804.gguf (version GGUF V3 (latest))
llama_model_loader: Dumping metadata keys/values. Note: KV overrides do not apply in this output.
llama_model_loader: - kv   0:                       general.architecture str              = glm4moe
llama_model_loader: - kv   1:                               general.type str              = model
llama_model_loader: - kv   2:                               general.name str              = GLM 4.5 Air
llama_model_loader: - kv   3:                         general.size_label str              = 128x9.4B
llama_model_loader: - kv   4:                            general.license str              = mit
llama_model_loader: - kv   5:                               general.tags arr[str,1]       = ["text-generation"]
llama_model_loader: - kv   6:                          general.languages arr[str,2]       = ["en", "zh"]
llama_model_loader: - kv   7:                        glm4moe.block_count u32              = 47
llama_model_loader: - kv   8:                     glm4moe.context_length u32              = 131072
llama_model_loader: - kv   9:                   glm4moe.embedding_length u32              = 4096
llama_model_loader: - kv  10:                glm4moe.feed_forward_length u32              = 10944
llama_model_loader: - kv  11:               glm4moe.attention.head_count u32              = 96
llama_model_loader: - kv  12:            glm4moe.attention.head_count_kv u32              = 8
llama_model_loader: - kv  13:                     glm4moe.rope.freq_base f32              = 1000000.000000
llama_model_loader: - kv  14:   glm4moe.attention.layer_norm_rms_epsilon f32              = 0.000010
llama_model_loader: - kv  15:                  glm4moe.expert_used_count u32              = 8
llama_model_loader: - kv  16:               glm4moe.attention.key_length u32              = 128
llama_model_loader: - kv  17:             glm4moe.attention.value_length u32              = 128
llama_model_loader: - kv  18:                          general.file_type u32              = 24
llama_model_loader: - kv  19:               glm4moe.rope.dimension_count u32              = 64
llama_model_loader: - kv  20:                       glm4moe.expert_count u32              = 128
llama_model_loader: - kv  21:         glm4moe.expert_feed_forward_length u32              = 1408
llama_model_loader: - kv  22:                glm4moe.expert_shared_count u32              = 1
llama_model_loader: - kv  23:          glm4moe.leading_dense_block_count u32              = 1
llama_model_loader: - kv  24:                 glm4moe.expert_gating_func u32              = 2
llama_model_loader: - kv  25:               glm4moe.expert_weights_scale f32              = 1.000000
llama_model_loader: - kv  26:                glm4moe.expert_weights_norm bool             = true
llama_model_loader: - kv  27:               glm4moe.nextn_predict_layers u32              = 1
llama_model_loader: - kv  28:                       tokenizer.ggml.model str              = gpt2
llama_model_loader: - kv  29:                         tokenizer.ggml.pre str              = glm4
llama_model_loader: - kv  30:                      tokenizer.ggml.tokens arr[str,151552]  = ["!", "\"", "#", "$", "%", "&", "'", ...
llama_model_loader: - kv  31:                  tokenizer.ggml.token_type arr[i32,151552]  = [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, ...  
llama_model_loader: - kv  32:                      tokenizer.ggml.merges arr[str,318088]  = ["─á ─á", "─á ─á─á─á", "─á─á ─á─á", "...
llama_model_loader: - kv  33:                tokenizer.ggml.eos_token_id u32              = 151329
llama_model_loader: - kv  34:            tokenizer.ggml.padding_token_id u32              = 151329
llama_model_loader: - kv  35:                tokenizer.ggml.bos_token_id u32              = 151331
llama_model_loader: - kv  36:                tokenizer.ggml.eot_token_id u32              = 151336
llama_model_loader: - kv  37:            tokenizer.ggml.unknown_token_id u32              = 151329
llama_model_loader: - kv  38:                tokenizer.ggml.eom_token_id u32              = 151338
llama_model_loader: - kv  39:                    tokenizer.chat_template str              = [gMASK]<sop>\n{%- if tools -%}\n<|syste...
llama_model_loader: - kv  40:               general.quantization_version u32              = 2
llama_model_loader: - kv  41:                                   split.no u16              = 0
llama_model_loader: - kv  42:                                split.count u16              = 804
llama_model_loader: - kv  43:                        split.tensors.count i32              = 803
llama_model_loader: - type  f32:  331 tensors
llama_model_loader: - type q8_0:  105 tensors
llama_model_loader: - type q5_K:   56 tensors
llama_model_loader: - type q6_K:   43 tensors
llama_model_loader: - type iq4_nl:   92 tensors
llama_model_loader: - type iq4_xs:  176 tensors
init_tokenizer: initializing tokenizer for type 2
load: control token: 151334 '<eop>' is not marked as EOG
load: control token: 151331 '[gMASK]' is not marked as EOG
load: control token: 151362 '<|end_of_box|>' is not marked as EOG
load: control token: 151343 '<|begin_of_audio|>' is not marked as EOG
load: control token: 151335 '<|system|>' is not marked as EOG
load: control token: 151345 '<|begin_of_transcription|>' is not marked as EOG
load: control token: 151344 '<|end_of_audio|>' is not marked as EOG
load: control token: 151336 '<|user|>' is not marked as EOG
load: control token: 151332 '[sMASK]' is not marked as EOG
load: control token: 151346 '<|end_of_transcription|>' is not marked as EOG
load: control token: 151330 '[MASK]' is not marked as EOG
load: control token: 151333 '<sop>' is not marked as EOG
load: control token: 151337 '<|assistant|>' is not marked as EOG
load: control token: 151338 '<|observation|>' is not marked as EOG
load: control token: 151339 '<|begin_of_image|>' is not marked as EOG
load: control token: 151340 '<|end_of_image|>' is not marked as EOG
load: control token: 151341 '<|begin_of_video|>' is not marked as EOG
load: control token: 151342 '<|end_of_video|>' is not marked as EOG
load: control token: 151347 '<|code_prefix|>' is not marked as EOG
load: control token: 151348 '<|code_middle|>' is not marked as EOG
load: control token: 151349 '<|code_suffix|>' is not marked as EOG
load: control token: 151360 '/nothink' is not marked as EOG
load: control token: 151361 '<|begin_of_box|>' is not marked as EOG
load: control token: 151363 '<|image|>' is not marked as EOG
load: control token: 151364 '<|video|>' is not marked as EOG
load: special_eot_id is not in special_eog_ids - the tokenizer config may be incorrect
load: special_eom_id is not in special_eog_ids - the tokenizer config may be incorrect
load: printing all EOG tokens:
load:   - 151329 ('<|endoftext|>')
load:   - 151336 ('<|user|>')
load:   - 151338 ('<|observation|>')
load: special tokens cache size = 36
load: token to piece cache size = 0.9713 MB
llm_load_print_meta: format           = GGUF V3 (latest)
llm_load_print_meta: arch             = glm4moe
llm_load_print_meta: n_ctx_train      = 131072
llm_load_print_meta: n_embd           = 4096
llm_load_print_meta: n_layer          = 47
llm_load_print_meta: n_head           = 96
llm_load_print_meta: n_head_kv        = 8
llm_load_print_meta: n_rot            = 64
llm_load_print_meta: n_swa            = 0
llm_load_print_meta: n_swa_pattern    = 1
llm_load_print_meta: n_embd_head_k    = 128
llm_load_print_meta: n_embd_head_v    = 128
llm_load_print_meta: n_gqa            = 12
llm_load_print_meta: n_embd_k_gqa     = 1024
llm_load_print_meta: n_embd_v_gqa     = 1024
llm_load_print_meta: f_norm_eps       = 0.0e+00
llm_load_print_meta: f_norm_rms_eps   = 1.0e-05
llm_load_print_meta: f_clamp_kqv      = 0.0e+00
llm_load_print_meta: f_max_alibi_bias = 0.0e+00
llm_load_print_meta: f_logit_scale    = 0.0e+00
llm_load_print_meta: n_ff             = 10944
llm_load_print_meta: n_expert         = 128
llm_load_print_meta: n_expert_used    = 8
llm_load_print_meta: causal attn      = 1
llm_load_print_meta: pooling type     = 0
llm_load_print_meta: rope type        = 2
llm_load_print_meta: rope scaling     = linear
llm_load_print_meta: freq_base_train  = 1000000.0
llm_load_print_meta: freq_scale_train = 1
llm_load_print_meta: n_ctx_orig_yarn  = 131072
llm_load_print_meta: rope_finetuned   = unknown
llm_load_print_meta: ssm_d_conv       = 0
llm_load_print_meta: ssm_d_inner      = 0
llm_load_print_meta: ssm_d_state      = 0
llm_load_print_meta: ssm_dt_rank      = 0
llm_load_print_meta: model type       = 106B.A12B
llm_load_print_meta: model ftype      = IQ1_S - 1.5625 bpw
llm_load_print_meta: model params     = 110.469 B
llm_load_print_meta: model size       = 65.117 GiB (5.063 BPW)
llm_load_print_meta: repeating layers = 64.105 GiB (5.041 BPW, 109.227 B parameters)
llm_load_print_meta: general.name     = GLM 4.5 Air
print_info: vocab type       = BPE
print_info: n_vocab          = 151552
print_info: n_merges         = 318088
print_info: BOS token        = 151331 '[gMASK]'
print_info: EOS token        = 151329 '<|endoftext|>'
print_info: EOT token        = 151336 '<|user|>'
print_info: EOM token        = 151338 '<|observation|>'
print_info: UNK token        = 151329 '<|endoftext|>'
print_info: PAD token        = 151329 '<|endoftext|>'
print_info: LF token         = 198 '─è'
print_info: FIM PRE token    = 151347 '<|code_prefix|>'
print_info: FIM SUF token    = 151349 '<|code_suffix|>'
print_info: FIM MID token    = 151348 '<|code_middle|>'
print_info: EOG token        = 151329 '<|endoftext|>'
print_info: EOG token        = 151336 '<|user|>'
print_info: EOG token        = 151338 '<|observation|>'
print_info: max token length = 1024
ggml_cuda_init: GGML_CUDA_FORCE_MMQ:    no
ggml_cuda_init: GGML_CUDA_FORCE_CUBLAS: no
ggml_cuda_init: found 1 CUDA devices:
  Device 0: NVIDIA RTX PRO 6000 Blackwell Workstation Edition, compute capability 12.0, VMM: yes
llm_load_tensors: ggml ctx size =    0.66 MiB
model has unused tensor blk.46.attn_norm.weight (size = 16384 bytes) -- ignoring
model has unused tensor blk.46.attn_q.weight (size = 26738688 bytes) -- ignoring
model has unused tensor blk.46.attn_k.weight (size = 2228224 bytes) -- ignoring
model has unused tensor blk.46.attn_v.weight (size = 2228224 bytes) -- ignoring
model has unused tensor blk.46.attn_q.bias (size = 49152 bytes) -- ignoring
model has unused tensor blk.46.attn_k.bias (size = 4096 bytes) -- ignoring
model has unused tensor blk.46.attn_v.bias (size = 4096 bytes) -- ignoring
model has unused tensor blk.46.attn_output.weight (size = 53477376 bytes) -- ignoring
model has unused tensor blk.46.post_attention_norm.weight (size = 16384 bytes) -- ignoring
model has unused tensor blk.46.ffn_gate_inp.weight (size = 2097152 bytes) -- ignoring
model has unused tensor blk.46.exp_probs_b.bias (size = 512 bytes) -- ignoring
model has unused tensor blk.46.ffn_gate_exps.weight (size = 605552640 bytes) -- ignoring
model has unused tensor blk.46.ffn_down_exps.weight (size = 415236096 bytes) -- ignoring
model has unused tensor blk.46.ffn_up_exps.weight (size = 605552640 bytes) -- ignoring
model has unused tensor blk.46.ffn_gate_shexp.weight (size = 4730880 bytes) -- ignoring
model has unused tensor blk.46.ffn_down_shexp.weight (size = 3244032 bytes) -- ignoring
model has unused tensor blk.46.ffn_up_shexp.weight (size = 4730880 bytes) -- ignoring
model has unused tensor blk.46.nextn.eh_proj.weight (size = 17825792 bytes) -- ignoring
model has unused tensor blk.46.nextn.embed_tokens.weight (size = 329777152 bytes) -- ignoring
model has unused tensor blk.46.nextn.enorm.weight (size = 16384 bytes) -- ignoring
model has unused tensor blk.46.nextn.hnorm.weight (size = 16384 bytes) -- ignoring
model has unused tensor blk.46.nextn.shared_head_head.weight (size = 329777152 bytes) -- ignoring
model has unused tensor blk.46.nextn.shared_head_norm.weight (size = 16384 bytes) -- ignoring
llm_load_tensors: offloading 47 repeating layers to GPU
llm_load_tensors: offloading non-repeating layers to GPU
llm_load_tensors: offloaded 48/48 layers to GPU
llm_load_tensors:  CUDA_Host buffer size =   407.00 MiB
llm_load_tensors:      CUDA0 buffer size = 63980.86 MiB
....................................................................................................
llama_new_context_with_model: n_ctx      = 131072
llama_new_context_with_model: n_batch    = 4096
llama_new_context_with_model: n_ubatch   = 4096
llama_new_context_with_model: flash_attn = 1
llama_new_context_with_model: mla_attn   = 0
llama_new_context_with_model: attn_max_b = 0
llama_new_context_with_model: fused_moe  = 1
llama_new_context_with_model: ser        = -1, 0
llama_new_context_with_model: freq_base  = 1000000.0
llama_new_context_with_model: freq_scale = 1
llama_kv_cache_init:      CUDA0 KV buffer size = 24064.00 MiB
llama_new_context_with_model: KV self size  = 24064.00 MiB, K (f16): 12032.00 MiB, V (f16): 12032.00 MiB
llama_new_context_with_model:  CUDA_Host  output buffer size =     1.16 MiB
llama_new_context_with_model:      CUDA0 compute buffer size =  3584.03 MiB
llama_new_context_with_model:  CUDA_Host compute buffer size =  2112.05 MiB
llama_new_context_with_model: graph nodes  = 2149
llama_new_context_with_model: graph splits = 2
INFO [                    init] initializing slots | tid="23312" timestamp=1755197087 n_slots=1
INFO [                    init] new slot | tid="23312" timestamp=1755197087 id_slot=0 n_ctx_slot=131072
INFO [                    main] model loaded | tid="23312" timestamp=1755197087
INFO [                    main] chat template | tid="23312" timestamp=1755197087 chat_template="[gMASK]<sop>\n{%- if tools -%}\n<|system|>\n# Tools\n\nYou may call one or more functions to assist with the user query.\n\nYou are provided with function signatures within <tools></tools> XML tags:\n<tools>\n{% for tool in tools %}\n{{ tool | tojson(ensure_ascii=False) }}\n{% endfor %}\n</tools>\n\nFor each function call, output the function name and arguments within the following XML format:\n<tool_call>{function-name}\n<arg_key>{arg-key-1}</arg_key>\n<arg_value>{arg-value-1}</arg_value>\n<arg_key>{arg-key-2}</arg_key>\n<arg_value>{arg-value-2}</arg_value>\n...\n</tool_call>{%- endif -%}\n{%- macro visible_text(content) -%}\n    {%- if content is string -%}\n        {{- content }}\n    {%- elif content is iterable and content is not mapping -%}\n        {%- for item in content -%}\n            {%- if item is mapping and item.type == 'text' -%}\n                {{- item.text }}\n            {%- elif item is string -%}\n                {{- item }}\n            {%- endif -%}\n        {%- endfor -%}\n    {%- else -%}\n        {{- content }}\n    {%- endif -%}\n{%- endmacro -%}\n{%- set ns = namespace(last_user_index=-1) %}\n{%- for m in messages %}\n    {%- if m.role == 'user' %}\n        {% set ns.last_user_index = loop.index0 -%}\n    {%- endif %}\n{%- endfor %}\n{% for m in messages %}\n{%- if m.role == 'user' -%}<|user|>\n{% set content = visible_text(m.content) %}{{ content }}\n{{- '/nothink' if (enable_thinking is defined and not enable_thinking and not content.endswith(\"/nothink\")) else '' -}}\n{%- elif m.role == 'assistant' -%}\n<|assistant|>\n{%- set reasoning_content = '' %}\n{%- set content = visible_text(m.content) %}\n{%- if m.reasoning_content is string %}\n    {%- set reasoning_content = m.reasoning_content %}\n{%- else %}\n    {%- if '</think>' in content %}\n        {%- set reasoning_content = content.split('</think>')[0].rstrip('\\n').split('<think>')[-1].lstrip('\\n') %}\n        {%- set content = content.split('</think>')[-1].lstrip('\\n') %}\n    {%- endif %}\n{%- endif %}\n{%- if loop.index0 > ns.last_user_index and reasoning_content -%}\n{{ '\\n<think>' + reasoning_content.strip() +  '</think>'}}\n{%- else -%}\n{{ '\\n<think></think>' }}\n{%- endif -%}\n{%- if content.strip() -%}\n{{ '\\n' + content.strip() }}\n{%- endif -%}\n{% if m.tool_calls %}\n{% for tc in m.tool_calls %}\n{%- if tc.function %}\n    {%- set tc = tc.function %}\n{%- endif %}\n{{ '\\n<tool_call>' + tc.name }}\n{% s.content is string -%}\n{%- if loop.first or (messages[loop.index0 - 1].role != \"tool\") %}\n    {{- '<|observation|>' }}\n{%- endif %}\n{{- '\\n<tool_response>\\n' }}\n{{- m.content }}\n{{- '\\n</tool_response>' }}\n{%- else -%}\n<|observation|>{% for tr in m.conten.content is string -%}\n{%- if loop.first or (messages[loop.index0 - 1].role != \"tool\") %}\n    {{- '<|observation|>' }}\n{%- endif %}\n{{- '\\n<tool_response>\\n' }}\n{{- m.content }}\n{{- '\\n</tool_response>' }}\n{%- else -%}\n<|observation|>{% for tr in m.content %}\n\n<tool_response>\n{{ tr.output if tr.output is defined else tr }}\n</tool_response>{% endfor -%}\n{% endif -%}\n{%- elif m.role == 'system' -%}\n<|system|>\n{{ visible_text(m.content) }}\n{%- endif -%}\n{%- endfor -%}\n{%- if add_generation_prompt -%}\n    <|assistant|>{{- '\\n<think></think>' if (enable_thinking is defined and not enable_thinking) else '' -}}\n{%- endif -%}"
INFO [                    main] chat template | tid="23312" timestamp=1755197087 chat_example="[gMASK]<sop><|system|>\nYou are a helpful assistant<|user|>\nHello<|assistant|>\nHi there<|user|>\nHow are you?<|assistant|>" built_in=true
INFO [                    main] HTTP server listening | tid="23312" timestamp=1755197087 hostname="127.0.0.1" port="8081" n_threads_http="35"
INFO [            update_slots] all slots are idle | tid="23312" timestamp=1755197087
INFO [   launch_slot_with_task] slot is processing task | tid="23312" timestamp=1755197120 id_slot=0 id_task=0
INFO [            update_slots] kv cache rm [p0, end) | tid="23312" timestamp=1755197120 id_slot=0 id_task=0 p0=0
INFO [            update_slots] kv cache rm [p0, end) | tid="23312" timestamp=1755197122 id_slot=0 id_task=0 p0=4096
INFO [            update_slots] kv cache rm [p0, end) | tid="23312" timestamp=1755197123 id_slot=0 id_task=0 p0=8192
INFO [            update_slots] kv cache rm [p0, end) | tid="23312" timestamp=1755197125 id_slot=0 id_task=0 p0=12288
INFO [            update_slots] kv cache rm [p0, end) | tid="23312" timestamp=1755197127 id_slot=0 id_task=0 p0=16384
INFO [            update_slots] kv cache rm [p0, end) | tid="23312" timestamp=1755197130 id_slot=0 id_task=0 p0=20480
INFO [            update_slots] kv cache rm [p0, end) | tid="23312" timestamp=1755197133 id_slot=0 id_task=0 p0=24576
INFO [            update_slots] kv cache rm [p0, end) | tid="23312" timestamp=1755197136 id_slot=0 id_task=0 p0=28672
INFO [            update_slots] kv cache rm [p0, end) | tid="23312" timestamp=1755197139 id_slot=0 id_task=0 p0=32768
INFO [            update_slots] kv cache rm [p0, end) | tid="23312" timestamp=1755197143 id_slot=0 id_task=0 p0=36864
INFO [            update_slots] kv cache rm [p0, end) | tid="23312" timestamp=1755197147 id_slot=0 id_task=0 p0=40960
INFO [            update_slots] kv cache rm [p0, end) | tid="23312" timestamp=1755197152 id_slot=0 id_task=0 p0=45056
INFO [            update_slots] kv cache rm [p0, end) | tid="23312" timestamp=1755197157 id_slot=0 id_task=0 p0=49152
INFO [            update_slots] kv cache rm [p0, end) | tid="23312" timestamp=1755197162 id_slot=0 id_task=0 p0=53248
INFO [            update_slots] kv cache rm [p0, end) | tid="23312" timestamp=1755197167 id_slot=0 id_task=0 p0=57344
INFO [            update_slots] kv cache rm [p0, end) | tid="23312" timestamp=1755197173 id_slot=0 id_task=0 p0=61440
INFO [            update_slots] kv cache rm [p0, end) | tid="23312" timestamp=1755197179 id_slot=0 id_task=0 p0=65536
INFO [            update_slots] kv cache rm [p0, end) | tid="23312" timestamp=1755197186 id_slot=0 id_task=0 p0=69632
INFO [            update_slots] kv cache rm [p0, end) | tid="23312" timestamp=1755197193 id_slot=0 id_task=0 p0=73728
INFO [            update_slots] kv cache rm [p0, end) | tid="23312" timestamp=1755197201 id_slot=0 id_task=0 p0=77824
INFO [            update_slots] kv cache rm [p0, end) | tid="23312" timestamp=1755197209 id_slot=0 id_task=0 p0=81920
INFO [            update_slots] kv cache rm [p0, end) | tid="23312" timestamp=1755197217 id_slot=0 id_task=0 p0=86016
INFO [            update_slots] kv cache rm [p0, end) | tid="23312" timestamp=1755197226 id_slot=0 id_task=0 p0=90112
INFO [            update_slots] kv cache rm [p0, end) | tid="23312" timestamp=1755197236 id_slot=0 id_task=0 p0=94208
INFO [            update_slots] kv cache rm [p0, end) | tid="23312" timestamp=1755197245 id_slot=0 id_task=0 p0=98304
INFO [            update_slots] kv cache rm [p0, end) | tid="23312" timestamp=1755197256 id_slot=0 id_task=0 p0=102400
INFO [           print_timings] prompt eval time     =  141097.11 ms / 104296 tokens (    1.35 ms per token,   739.18 tokens per second) | tid="23312" timestamp=1755197410 id_slot=0 id_task=0 t_prompt_processing=141097.114 n_prompt_tokens_processed=104296 t_token=1.3528525926210018 n_tokens_second=739.178832530905
INFO [           print_timings] generation eval time =  148612.29 ms /   921 runs   (  161.36 ms per token,     6.20 tokens per second) | tid="23312" timestamp=1755197410 id_slot=0 id_task=0 t_token_generation=148612.286 n_decoded=921 t_token=161.35970249728555 n_tokens_second=6.1973341827202635
INFO [           print_timings]           total time =  289709.40 ms | tid="23312" timestamp=1755197410 id_slot=0 id_task=0 t_prompt_processing=141097.114 t_token_generation=148612.286 t_total=289709.4
INFO [            update_slots] slot released | tid="23312" timestamp=1755197410 id_slot=0 id_task=0 n_ctx=131072 n_past=105216 n_system_tokens=0 n_cache_tokens=105216 truncated=false
INFO [            update_slots] all slots are idle | tid="23312" timestamp=1755197410
INFO [format_partial_response_oaicompat] DEBUG: Streaming finish_reason check | tid="28636" timestamp=1755197410 generated_text="\n<think>Hmm, the user is asking about what level you reach when you're halfway to the maximum experience in DipiloBlop. They've provided a massive amount of text from what appears to be a Wikibook about DipiloBlop, including skill guides and quest information.\n\nLooking through this material, I need to find the experience table that shows the progression of levels and their experience requirements. I recall seeing such a table earlier in the text. Let me scan through the Skills/Experience Table section.\n\nAh, here it is! The table shows experience needed for each level and the difference from the previous level. The maximum level appears to be 101 based on the table going up to level 101. The total experience needed to reach level 101 is 507,038,931 experience points.\n\nHalf of this would be approximately 253,519,465.5 experience points. Now I need to find which level requires at least this much experience.\n\nLooking at the table:\n- Level 100 requires 467,074,482 experience\n- Level 101 requires 507,038,931 experience\n\nHalf the total experience (253,519,465.5) falls between these two levels. Specifically, it's less than level 100's requirement but more than level 99's.\n\nLevel 99 requires 363,640,890 experience, which is less than half the total. Level 100 requires 467,074,482, which is more than half the total.\n\nSo when a player reaches exactly half the maximum experience (253,519,465.5), they would be somewhere between level 99 and level 100. Since experience points accumulate continuously, they would technically be at level 99 until they reach the 363,640,890 XP threshold for level 100.\n\nThe key point is that they haven't yet reached level 100, so they would still be at level 99 when they've accumulated half the total experience required for level 101.</think>Based on the **Skills/Experience Table** provided in the book, the maximum level in DipiloBlop is **101**, with a total experience requirement of **507,038,931** experience points to reach it.  \n\nHalf of this maximum experience is **253,519,465.5** experience points.  \n\nFrom the table:  \n- **Level 99** requires **363,640,890** experience (to reach level 99).  \n- **Level 100** requires **467,074,482** experience (to reach level 100).  \n- **Level 101** requires **507,038,931** experience (to reach level 101).  \n\n**253,519,465.5 XP** (half of max) falls **between the requirements for level 99 and level 100**:  \n- Since **363,640,890 > 253,519,465.5**, a player with half the total experience has **not yet reached level 99**.  \n- The closest lower level is **level 98**, which requires **312,125,351** experience.  \n\n**Conclusion**:  \nWhen you have **half the maximum experience for level 101 (253,519,465.5 XP)**, you are **between level 98 and level 99**. Specifically, you are **still at level 98**, as you have not yet accumulated enough XP to reach level 99.  \n\n### Key Details from the Table:\n| Level | XP to Reach Level | XP from Previous Level |\n|-------|-------------------|------------------------|\n| 98    | 312,125,351      | 48,484,461            |\n| **99**| **363,640,890**  | 51,515,539             |\n| 100   | 467,074,482      | 103,433,592            |\n| 101   | 507,038,931      | 40,000,000             |\n\n**Final Answer**: At **253,519,465.5 XP** (half of max XP for level 101), you are **at level 98**." model_name="gpt-3.5-turbo-0613" tool_calls_count=0
INFO [      log_server_request] request | tid="28636" timestamp=1755197410 remote_addr="127.0.0.1" remote_port=53255 status=200 method="POST" path="/v1/chat/completions" params={}
INFO [            update_slots] all slots are idle | tid="23312" timestamp=1755197410
INFO [            update_slots] all slots are idle | tid="23312" timestamp=1755197629
```

</details>

# Large Prompt - Round 1:
```
Prompt
- Tokens: 104304
- Time: 135300.045 ms
- Speed: 770.9 t/s
Generation
- Tokens: 624
- Time: 100281.348 ms
- Speed: 6.2 t/s
```

# Large Prompt - Round 2:
```
Prompt
- Tokens: 104296
- Time: 141097.114 ms
- Speed: 739.2 t/s
Generation
- Tokens: 921
- Time: 148612.274 ms
- Speed: 6.2 t/s
```

Recipe used:

[GLM-4.5-Air.ROOT-4.7789bpw-4.6437ppl.63GB-GGUF_5GB-GPU_58GB-CPU.6d32a73_ed85f05.recipe](https://github.com/Thireus/GGUF-Tool-Suite/blob/main/recipe_examples/llama.cpp_recipes/GLM-4.5-Air.ROOT-4.7789bpw-4.6437ppl.63GB-GGUF_5GB-GPU_58GB-CPU.6d32a73_ed85f05.recipe)

---

👤 **saood06** commented on **2025-08-14** at **22:51:53**

> It may also mean that ik_llama.cpp may be in a huge disadvantage compared to mainline because in ik_llama.cpp there is a warm-up where all experts get used, which discards the potential advantage of having the fixed set of used experts stay in the cache(s). This is not done in mainline, where the warm-up run is with the same tokens as used later in the measurement runs.

What? They did merge https://github.com/ggml-org/llama.cpp/pull/11571 in (a cleaner, revision from what I ported which was an earlier commit of it), and then updated it to be faster (https://github.com/ggml-org/llama.cpp/pull/14753), at least that was the state of it I was aware of from the PRs

---

👤 **ikawrakow** commented on **2025-08-15** at **06:05:53**

@Thireus Not sure I understand your results. Initially using CUDA graphs basically doubles TG performance in your setup. But then it becomes just 9% in the latest tests?

But looking at the graphs you had posted [here](https://github.com/ikawrakow/ik_llama.cpp/pull/668#issuecomment-3170522512), it seems `ik_llama.cpp` has a sharp drop in TG performance starting at around 45k tokens in the KV cache. It would be very useful to have `sweep-bench` results for `llama.cpp` and `ik_llama.cpp`.

---

👤 **Thireus** commented on **2025-08-15** at **06:09:44**

@ikawrakow - The last results are at 130K context size, processing a 104K prompt. I'll run sweep-bench, it will make things clearer hopefully.

---

👤 **Thireus** commented on **2025-08-15** at **09:32:22**

@ikawrakow - See results below:

<img width="1779" height="1181" alt="GLM-4 5-Air_spp" src="https://github.com/user-attachments/assets/0f330c98-8ec1-49f7-9fcf-c9116a1eae09" />
<img width="1758" height="1093" alt="GLM-4 5-Air_stg" src="https://github.com/user-attachments/assets/6df4ed63-ed78-4135-bef2-b4ff73772b8a" />

Around 35k context size this is where Cuda Graphs on ik_llama.cpp stop making a big difference. Also interesting to see that around 30k context size, llama.cpp PP speed becomes faster than ik_llama.cpp.

---

👤 **oovloveme** commented on **2025-08-15** at **10:12:53**

Seems when using hybrid inference tg was about 10% slower than last version on ubuntu.

cmake -B ./build -DGGML_CUDA=ON -DCMAKE_CUDA_ARCHITECTURES="120" -DGGML_SCHED_MAX_COPIES=1 -DGGML_BLAS=OFF
cmake --build ./build --config Release -j 56

/home/ee/ik_llama.cpp/ik_llama.cpp/build/bin/llama-server \
    --model /home/ee/models/DeepSeek-R1-0528-UD-Q4_K_XL/UD-Q4_K_XL/DeepSeek-R1-0528-UD-Q4_K_XL-00001-of-00008.gguf \
    --alias 0528 \
    -c 80000 -ctk q8_0 \
    -mla 3 -fa -amb 512 \
    --host 0.0.0.0 --port 8000 \
    -ts 1,1 --parallel 2  --threads 56 \
    -ngl 99 -ot ".ffn_.*_exps."=CPU \
    -fmoe

---

👤 **ikawrakow** commented on **2025-08-15** at **10:30:01**

> Seems when using hybrid inference tg was about 10% slower than last version on ubuntu.

If so, add `-DGGML_CUDA_USE_GRAPHS=OFF` to your `cmake` command. This should restore the previous behavior.

---

👤 **oovloveme** commented on **2025-08-15** at **10:52:17**

> > Seems when using hybrid inference tg was about 10% slower than last version on ubuntu.
> 
> If so, add `-DGGML_CUDA_USE_GRAPHS=OFF` to your `cmake` command. This should restore the previous behavior.

However my graphic card is blackwell(on cuda 13.0) not turing.
It's also slower without -fmoe( cuda graph should be disabled without fmoe? )

---

👤 **ikawrakow** commented on **2025-08-15** at **10:59:20**

> However my graphic card is blackwell(on cuda 13.0) not turing.

Then I need to update the description to not specifically mention Turing. Until now we had one user report performance degradation with graphs, now we have two.

---

👤 **ikawrakow** commented on **2025-08-15** at **11:04:43**

@oovloveme Btw, I would try using `-ot exps=CPU` instead of `-ot ".ffn_.*_exps."=CPU`. The difference is subtle, but may give a better TG performance.

---

👤 **Ph0rk0z** commented on **2025-08-15** at **11:25:41**

Wouldn't disabling cuda graphs just give lower performance if they had cuda graphs on before?

---

👤 **ikawrakow** commented on **2025-08-15** at **11:41:29**

> Wouldn't disabling cuda graphs just give lower performance if they had cuda graphs on before?

No, because CUDA graphs were disabled for MoE models before this PR, and for hybrid inference the impact is small to none as you found out yourself.

---

👤 **Thireus** commented on **2025-08-16** at **11:36:38**

@ikawrakow - You're right, something is up with the GLM-4.5 implementation. Here's Qwen3-235B-A22B-Thinking-2507 with more consistent results:
<img width="1782" height="1181" alt="Qwen3-235B-A22B-Thinking-2507_spp" src="https://github.com/user-attachments/assets/7a777e6c-924f-4a92-98e3-57aaae8f4f61" />
<img width="1683" height="1093" alt="Qwen3-235B-A22B-Thinking-2507_stg" src="https://github.com/user-attachments/assets/b0bf4006-cb40-4f54-9f48-e01bf8d5f95f" />

Recipe used:

<details>

```
## Quant mix recipe created using Thireus' GGUF Tool Suite - https://gguf.thireus.com/
# Model name: Qwen3-235B-A22B-Thinking-2507
# Link to the original model: https://huggingface.co/Qwen/Qwen3-235B-A22B-Thinking-2507

## Model head & embeddings — qbits: 32 6 
output_norm\.weight=f32
token_embd\.weight=q6_K
output\.weight=q6_K

## Multi-headed attention parameters — qbits: 32 5 
blk\.([0-9]|[1-8][0-9]|9[0-3])\.attn_q\.weight=q5_K
blk\.([0-9]|[1-8][0-9]|9[0-3])\.attn_k_norm\.weight=f32
blk\.([0-9]|[1-8][0-9]|9[0-3])\.attn_k\.weight=q5_K
blk\.([0-9]|[1-8][0-9]|9[0-3])\.attn_norm\.weight=f32
blk\.([0-9]|[1-8][0-9]|9[0-3])\.attn_q_norm\.weight=f32
blk\.([0-9]|[1-8][0-9]|9[0-3])\.attn_v\.weight=q5_K
blk\.([0-9]|[1-8][0-9]|9[0-3])\.attn_output\.weight=q5_K

## Core FFN weights — qbits: 32 
blk\.([0-9]|[1-8][0-9]|9[0-3])\.ffn_norm\.weight=f32
blk\.([0-9]|[1-8][0-9]|9[0-3])\.ffn_gate_inp\.weight=f32

## CPU-loaded ffn_*_exps
# ffn_down_exps (down-extraction) — qbits: 3 2 
blk\.(25|45|59)\.ffn_down_exps\.weight=q3_K
blk\.([0-9]|[6-8][0-9]|2[0-4]|4[0-4]|1[0-9]|3[0-9]|9[0-3]|2[6-9]|4[6-9]|5[0-8])\.ffn_down_exps\.weight=q2_K

# ffn_up_exps (up-extraction) — qbits: 3 1 
blk\.(8[2-9]|9[0-3])\.ffn_up_exps\.weight=q3_K
blk\.([0-9]|[1-7][0-9]|8[0-1])\.ffn_up_exps\.weight=iq1_m

# ffn_gate_exps (gate-extraction) — qbits: 3 1 
blk\.(8[2-9]|9[0-3])\.ffn_gate_exps\.weight=q3_K
blk\.([0-9]|[1-7][0-9]|8[0-1])\.ffn_gate_exps\.weight=iq1_m

## Summary of tensor sizes per class
# GPU Total: 5.429 GiB (95.1%) | 5.71 GiB max, if all were q8_0 | 5.43 GiB min, if all were q6_K
# CPU Total: 58.002 GiB (69.1%) | 83.95 GiB max, if all were q3_K | 54.21 GiB min, if all were iq1_m
# GPU+CPU Total: 63.431 GiB (82.1%)

## Summary of tensor counts and bpw per qtype
#
# GPU-loaded quants:
# QTYPE		Count	BPW	Assigned GiB	% Assigned	Max GiB (all)
# +f32       	471	32.0  	  0.19 GiB	-		-
# q8_0      	0  	8.5   	  0.00 GiB	0.0%		1.23
# q6_K      	2  	6     	  0.95 GiB	100.0%		0.95
# +q5_K      	376	5     	  4.29 GiB	-		-
#
# CPU-loaded quants:
# QTYPE		Count	BPW	Assigned GiB	% Assigned	Max GiB (all)
# +q3_K      	3  	3     	  0.97 GiB	-		-
# q3_K      	24 	3     	  7.73 GiB	12.8%		60.59
# +q2_K      	91 	2     	 22.39 GiB	-		-
# q2_K      	0  	2     	  0.00 GiB	0.0%		46.27
# iq1_m     	164	1.75  	 26.91 GiB	87.2%		30.84
#
# -Average BPW: 2.0800
#
# -Notes:
# - '+' means user-defined pre-assigned tensors, or tensor missing from csv data or f32 tensors
# - Recipe produced on the 2025-08-16 09:30:39 UTC+0000 using Thireus' GGUF tools (https://gguf.thireus.com/)
# - Script SHA-256: 6d32a7363bd5b80cbcfc8ef09fa50fdba9a37bd998d8ba91005aec6fffed15c3
# - Calibration dataset 'ppl_results.csv' SHA-256: 6251d5aeb5e3c42a328defefb4fa9930e5255c182f2f3dbe3d2b5bd0d6ed1000
# - tensors.bf16.map SHA-256: a27fd8086310d07b113e4c8f1673f8b3250cf2e2967454e4f496b196e916fa35
# - tensors.bf16.map model name: Qwen3-235B-A22B-Thinking-2507-THIREUS-BF16-SPECIAL_TENSOR-01132-of-01132
# - tensors.iq1_m.map SHA-256: 081288147c9ae8a20cb88593d524b1f4a5549b5fe47f6f9e913bb25a9f15f3c5
# - tensors.iq1_m.map model name: Qwen3-235B-A22B-Thinking-2507-THIREUS-IQ1_M-SPECIAL_TENSOR-01132-of-01132
# - tensors.q2_K.map SHA-256: e1de989c9899b2dd9acff81dddb0b9647843c5a082cef4d16fe27eee38630fb8
# - tensors.q2_K.map model name: Qwen3-235B-A22B-Thinking-2507-THIREUS-Q2_K-SPECIAL_TENSOR-01132-of-01132
# - tensors.q3_K.map SHA-256: efe0ba96d915e0130730ad543f8e1bdcb1c121b35133bbf51c80f8d4d43a6adb
# - tensors.q3_K.map model name: Qwen3-235B-A22B-Thinking-2507-THIREUS-Q3_K-SPECIAL_TENSOR-01132-of-01132
# - tensors.q8_0.map SHA-256: 255e77346241c4dacf8f1ad75bc4131ab0f86e23eb8d12a260d09b6e79bc6bab
# - tensors.q8_0.map model name: Qwen3-235B-A22B-Thinking-2507-THIREUS-Q8_0-SPECIAL_TENSOR-01132-of-01132
# - tensors.q6_K.map SHA-256: dc22be671003a8b6247ca184d07b3bf319b6cc63510086dad8fd5146b9783628
# - tensors.q6_K.map model name: Qwen3-235B-A22B-Thinking-2507-THIREUS-Q6_K-SPECIAL_TENSOR-01132-of-01132
# - tensors.iq1_m_r4.map SHA-256: 1a48c0a6ef6e492af570c3ca34e71048d785b0819ee3fe0ea87a976138f59715
# - tensors.iq1_m_r4.map model name: Qwen3-235B-A22B-Thinking-2507-THIREUS-IQ1_M_R4-SPECIAL_TENSOR-01132-of-01132
# - tensors.q5_K.map SHA-256: eb25fe42fab6ce6cd6568845ff6d2db513053e1df995a9c55bb22a4a3b2422bc
# - tensors.q5_K.map model name: Qwen3-235B-A22B-Thinking-2507-THIREUS-Q5_K-SPECIAL_TENSOR-01132-of-01132
# - GPG signatures: PASSED
# - Command used:
# ../../quant_assign.py ppl_results.csv --tolerance 0.01 --cpu-irq-k 1.5 --gpu-irq-k 1.5 --gpu-assign-qtype q5_K \
# --cpu-tensors-max-size 56 --gpu-tensors-max-size 95% --exponential-factor 8 --cpu-tensors \
# 'blk\.([0-9]|[1-8][0-9]|9[0-3])\.ffn_up_exps\.weight' 'blk\.([0-9]|[1-8][0-9]|9[0-3])\.ffn_gate_exps\.weight' \
# --gpu-tensors '.*' --cpu-quants iq1_m q2_K q3_K --gpu-quants q8_0 q6_K --cpu-assign-tensors \
# 'blk\.([0-9]|1[0-9]|2[0-4]|2[6-9]|3[0-9]|4[0-4]|4[6-9]|5[0-8]|[6-8][0-9]|9[0-3])\.ffn_down_exps\.weight=q2_K' \
# 'blk\.(25|45|59)\.ffn_down_exps\.weight=q3_K' --gpu-assign-tensors \
# 'blk\.([0-9]|[1-8][0-9]|9[0-3])\.attn_k_b\.weight=q8_0' --harmonize-tensors \
# 'blk\..*\.ffn_up_exps.*,blk\..*\.ffn_gate_exps.*' --harmonization-technique 3

## THE END!
```

</details>

---

👤 **Thireus** commented on **2025-08-17** at **11:58:46**

> @ikawrakow - See results below:
> <img alt="GLM-4 5-Air_spp" width="1779" height="1181" src="https://private-user-images.githubusercontent.com/721759/478373730-0f330c98-8ec1-49f7-9fcf-c9116a1eae09.png?jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NTU0MzIxOTksIm5iZiI6MTc1NTQzMTg5OSwicGF0aCI6Ii83MjE3NTkvNDc4MzczNzMwLTBmMzMwYzk4LThlYzEtNDlmNy05ZmNmLWM5MTE2YTFlYWUwOS5wbmc_WC1BbXotQWxnb3JpdGhtPUFXUzQtSE1BQy1TSEEyNTYmWC1BbXotQ3JlZGVudGlhbD1BS0lBVkNPRFlMU0E1M1BRSzRaQSUyRjIwMjUwODE3JTJGdXMtZWFzdC0xJTJGczMlMkZhd3M0X3JlcXVlc3QmWC1BbXotRGF0ZT0yMDI1MDgxN1QxMTU4MTlaJlgtQW16LUV4cGlyZXM9MzAwJlgtQW16LVNpZ25hdHVyZT1hNjA1ZGRmMTMxMWNiOTVjYjQ1MTliZDczZGY3YWE3MzdiN2M3YzNjNDUxNjBjNWQzNzFjNGJlM2FjYTllNzEyJlgtQW16LVNpZ25lZEhlYWRlcnM9aG9zdCJ9.3BIBgSEgRn40IDIdUzMtLDVkdf-4as_mZm0kFPXIfBM"> <img alt="GLM-4 5-Air_stg" width="1758" height="1093" src="https://private-user-images.githubusercontent.com/721759/478373734-6df4ed63-ed78-4135-bef2-b4ff73772b8a.png?jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NTU0MzIxOTksIm5iZiI6MTc1NTQzMTg5OSwicGF0aCI6Ii83MjE3NTkvNDc4MzczNzM0LTZkZjRlZDYzLWVkNzgtNDEzNS1iZWYyLWI0ZmY3Mzc3MmI4YS5wbmc_WC1BbXotQWxnb3JpdGhtPUFXUzQtSE1BQy1TSEEyNTYmWC1BbXotQ3JlZGVudGlhbD1BS0lBVkNPRFlMU0E1M1BRSzRaQSUyRjIwMjUwODE3JTJGdXMtZWFzdC0xJTJGczMlMkZhd3M0X3JlcXVlc3QmWC1BbXotRGF0ZT0yMDI1MDgxN1QxMTU4MTlaJlgtQW16LUV4cGlyZXM9MzAwJlgtQW16LVNpZ25hdHVyZT02M2UwZTg0YzQ2MjAxOGI3NWY3NzNlYjRiYWEwYTRhZWUwNzIwMDE2NWExY2M1MmVjMDg5ZjE3YThjNDkyOThkJlgtQW16LVNpZ25lZEhlYWRlcnM9aG9zdCJ9.QBcByMv_lvR8eF5cNFtrm8blm5vQTLtTcvLLmSVaHB4">
> 
> Around 35k context size this is where Cuda Graphs on ik_llama.cpp stop making a big difference. Also interesting to see that around 30k context size, llama.cpp PP speed becomes faster than ik_llama.cpp.

This has now been fixed. See https://github.com/ikawrakow/ik_llama.cpp/pull/700#issuecomment-3194321347

---

👤 **saood06** started a conversation on `src/llama-vocab.cpp` on **2025-08-17** at **13:22:14**

Is this needed?

It adds 816 lines to my model load of Deepseek (single line example: `load: control token: 128713 '<｜place▁holder▁no▁713｜>' is not marked as EOG`)

> 👤 **ikawrakow** replied on **2025-08-17** at **13:26:11**
> 
> Sorry, `LLAMA_LOG_DEBUG` is supposed to be active only in debug builds. Forgot to fix it. But it would be also good to understand if it is bad that these tokens are not marked as EOG and, if it is, understand why they are not marked as such.

> 👤 **saood06** replied on **2025-08-17** at **13:31:17**
> 
> To be honest I don't fully understand what it is printing. It says on one of the 816 lines:
> 
> ```
> load: control token:      1 '<｜end▁of▁sentence｜>' is not marked as EOG
> ```
> 
> but then later on load:
> 
> ```
> load: printing all EOG tokens:
> load:   - 1 ('<｜end▁of▁sentence｜>')
> ```
> 
> and I have used this model file extensively, it does treat that as an EOG.