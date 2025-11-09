## 🔀 [Pull Request #868](https://github.com/ikawrakow/ik_llama.cpp/pull/868) - Even more fused ops 

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/fuse_biased_qkv` |
| **Target Branch** | `main` |
| **Created** | 2025-10-26 |
| **Updated** | 2025-10-28 |
| **Merged** | 2025-10-27 |

---

## 📄 Description

This PR adds fusing of GEMV and subsequent additions on CUDA. This is mostly relevant for the Q, K, V computations when there is a bias. There is also a faster version of copying K and V into the KV cache (when the cache is `fp16`).

For a model such as GPT-OSS-20B running fully offloaded to the GPU, the combined effect of this PR and the preceding fusion PRs is a 10+% boost in TG performance.

Here is what we currently have for GPT-OSS-20B-MXFP4 on RTX-4080 with u-batch size of 2048

### This PR

 |    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  2048 |    512 |      0 |    0.189 | 10845.27 |    2.578 |   198.59 |
|  2048 |    512 |   2048 |    0.184 | 11131.77 |    2.647 |   193.40 |
|  2048 |    512 |   4096 |    0.192 | 10675.01 |    2.681 |   190.94 |
|  2048 |    512 |   6144 |    0.200 | 10226.91 |    2.727 |   187.73 |
|  2048 |    512 |   8192 |    0.210 |  9752.80 |    2.771 |   184.76 |
|  2048 |    512 |  10240 |    0.217 |  9422.68 |    2.823 |   181.37 |
|  2048 |    512 |  12288 |    0.226 |  9044.70 |    2.867 |   178.60 |
|  2048 |    512 |  14336 |    0.235 |  8715.78 |    2.907 |   176.13 |
|  2048 |    512 |  16384 |    0.245 |  8343.25 |    2.941 |   174.06 |
|  2048 |    512 |  18432 |    0.252 |  8118.83 |    2.988 |   171.37 |
|  2048 |    512 |  20480 |    0.262 |  7817.24 |    3.035 |   168.71 |
|  2048 |    512 |  22528 |    0.271 |  7568.79 |    3.081 |   166.17 |
|  2048 |    512 |  24576 |    0.279 |  7337.43 |    3.130 |   163.57 |
|  2048 |    512 |  26624 |    0.287 |  7138.97 |    3.175 |   161.25 |
|  2048 |    512 |  28672 |    0.295 |  6931.68 |    3.208 |   159.58 |
|  2048 |    512 |  30720 |    0.304 |  6736.47 |    3.250 |   157.56 |

### llama.cpp at 73a48c9790d320476b3e5ef75bda09f2f8269e6e

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  2048 |    512 |      0 |    0.315 |  6508.18 |    2.664 |   192.17 |
|  2048 |    512 |   2048 |    0.278 |  7362.40 |    2.836 |   180.51 |
|  2048 |    512 |   4096 |    0.295 |  6934.17 |    2.912 |   175.84 |
|  2048 |    512 |   6144 |    0.313 |  6547.90 |    2.984 |   171.59 |
|  2048 |    512 |   8192 |    0.330 |  6201.12 |    3.050 |   167.88 |
|  2048 |    512 |  10240 |    0.347 |  5899.50 |    3.112 |   164.54 |
|  2048 |    512 |  12288 |    0.364 |  5630.92 |    3.188 |   160.61 |
|  2048 |    512 |  14336 |    0.382 |  5361.12 |    3.233 |   158.36 |
|  2048 |    512 |  16384 |    0.399 |  5133.17 |    3.287 |   155.75 |
|  2048 |    512 |  18432 |    0.416 |  4917.59 |    3.345 |   153.08 |
|  2048 |    512 |  20480 |    0.428 |  4790.15 |    3.401 |   150.52 |
|  2048 |    512 |  22528 |    0.452 |  4532.86 |    3.458 |   148.06 |
|  2048 |    512 |  24576 |    0.470 |  4357.04 |    3.526 |   145.22 |
|  2048 |    512 |  26624 |    0.486 |  4213.83 |    3.567 |   143.53 |
|  2048 |    512 |  28672 |    0.495 |  4134.42 |    3.628 |   141.13 |
|  2048 |    512 |  30720 |    0.515 |  3979.36 |    3.685 |   138.94 |

---

## 💬 Conversation

👤 **ikawrakow** commented on **2025-10-26** at **17:11:40**

Here one more `sweep-bench` for Ling-mini-2.0. That one is interesting because a) it uses grouped expert routing and b) inference is so fast that the time spent preparing the inputs and building the graph starts being not negligible. In that regard mainline `llama.cpp` has the advantage of being able to reuse the previous graph for TG. Still, we get this

### This PR

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  2048 |    512 |      0 |    0.144 | 14218.87 |    1.168 |   438.41 |
|  2048 |    512 |   2048 |    0.117 | 17495.90 |    1.229 |   416.50 |
|  2048 |    512 |   4096 |    0.123 | 16628.10 |    1.308 |   391.41 |
|  2048 |    512 |   6144 |    0.130 | 15730.13 |    1.385 |   369.63 |
|  2048 |    512 |   8192 |    0.136 | 15064.36 |    1.445 |   354.21 |
|  2048 |    512 |  10240 |    0.143 | 14344.25 |    1.533 |   333.96 |
|  2048 |    512 |  12288 |    0.157 | 13078.99 |    1.600 |   319.98 |
|  2048 |    512 |  14336 |    0.156 | 13113.08 |    1.728 |   296.22 |
|  2048 |    512 |  16384 |    0.175 | 11701.45 |    1.800 |   284.51 |
|  2048 |    512 |  18432 |    0.172 | 11915.71 |    1.829 |   279.93 |
|  2048 |    512 |  20480 |    0.177 | 11543.95 |    1.896 |   270.06 |
|  2048 |    512 |  22528 |    0.185 | 11042.28 |    2.025 |   252.90 |
|  2048 |    512 |  24576 |    0.191 | 10708.38 |    2.065 |   248.00 |
|  2048 |    512 |  26624 |    0.200 | 10256.82 |    2.126 |   240.85 |
|  2048 |    512 |  28672 |    0.204 | 10034.25 |    2.168 |   236.14 |
|  2048 |    512 |  30720 |    0.210 |  9743.24 |    2.253 |   227.29 |

### llama.cpp at 73a48c9790d320476b3e5ef75bda09f2f8269e6e

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  2048 |    512 |      0 |    0.169 | 12104.81 |    1.258 |   406.91 |
|  2048 |    512 |   2048 |    0.144 | 14218.37 |    1.358 |   376.98 |
|  2048 |    512 |   4096 |    0.159 | 12873.22 |    1.441 |   355.29 |
|  2048 |    512 |   6144 |    0.175 | 11714.24 |    1.524 |   335.87 |
|  2048 |    512 |   8192 |    0.190 | 10760.03 |    1.605 |   319.08 |
|  2048 |    512 |  10240 |    0.207 |  9911.96 |    1.686 |   303.69 |
|  2048 |    512 |  12288 |    0.223 |  9204.21 |    1.772 |   288.92 |
|  2048 |    512 |  14336 |    0.239 |  8563.48 |    1.858 |   275.52 |
|  2048 |    512 |  16384 |    0.255 |  8030.81 |    1.937 |   264.29 |
|  2048 |    512 |  18432 |    0.272 |  7540.89 |    2.020 |   253.44 |
|  2048 |    512 |  20480 |    0.286 |  7157.64 |    2.102 |   243.60 |
|  2048 |    512 |  22528 |    0.295 |  6932.36 |    2.184 |   234.45 |
|  2048 |    512 |  24576 |    0.319 |  6410.44 |    2.267 |   225.89 |
|  2048 |    512 |  26624 |    0.336 |  6091.23 |    2.354 |   217.53 |
|  2048 |    512 |  28672 |    0.353 |  5795.40 |    2.443 |   209.56 |
|  2048 |    512 |  30720 |    0.358 |  5713.73 |    2.532 |   202.20 |

---

👤 **Nexesenex** commented on **2025-10-27** at **01:14:13**

If I execute that :

`llama-server -m LLaMa-3.3-70B-Q5_K_M.gguf -ngl 150 -b 128 -mg 0 -ts 31,31,19 --no-mmap -c 65536 -ctk q6_0 -ctv q5_0 --host 127.0.0.1 --port 8080`

And I prompt 8,000 tokens.

I get that : 

```
INFO [            update_slots] kv cache rm [p0, end) | tid="4120" timestamp=1761527135 id_slot=0 id_task=1276 p0=6657
Oops: bias CUDA0#inp_embd#0 is 8192 x 4 x 1 x 1
ik_llama.cpp.fks\ggml\src\ggml-cuda\mmvq.cu:195: GGML_ASSERT(ggml_nrows(bias) == 1) failed
```

Only this PR and the new WebUI PR merged on my branch.

---

👤 **ikawrakow** commented on **2025-10-27** at **05:11:32**

@Nexesenex Thanks for the report. Does the last commit fix the issue?

---

👤 **ikawrakow** commented on **2025-10-27** at **05:31:12**

@CISC What a funny coincidence! You fully [independently discovering](https://github.com/ggml-org/llama.cpp/pull/16789/files#diff-0ad629544d728ec8bf0e035d0250c5e382c669db9429750db9296dab14c68794R115) the [exact same](https://github.com/ikawrakow/ik_llama.cpp/pull/868/commits/21a37bfc6c510ed470bf500ddbcf8e9d6eac3329) copy optimization as I did here about 2 hours after I published this PR.

---

👤 **CISC** commented on **2025-10-27** at **07:41:54**

> @CISC What a funny coincidence! You fully [independently discovering](https://github.com/ggml-org/llama.cpp/pull/16789/files#diff-0ad629544d728ec8bf0e035d0250c5e382c669db9429750db9296dab14c68794R115) the [exact same](https://github.com/ikawrakow/ik_llama.cpp/pull/868/commits/21a37bfc6c510ed470bf500ddbcf8e9d6eac3329) copy optimization as I did here about 2 hours after I published this PR.

It was not a coincidence, I did indeed notice what you did.

---

👤 **CISC** started a conversation on `ggml/src/ggml-cuda/cpy.cu` on **2025-10-27** at **08:40:30**

```suggestion
        ggml_cpy_flt_cuda<float, float> (src0_ddc, src1_ddc, ne, ne00, ne01, ne02, nb00, nb01, nb02, nb03, ne10, ne11, ne12, nb10, nb11, nb12, nb13, main_stream, dest_ptrs_d, graph_cpynode_index);
```
This codepath will never be hit, it is caught by the first clause.

---

👤 **CISC** started a conversation on `ggml/src/ggml-cuda/cpy.cu` on **2025-10-27** at **08:46:57**

Not sure why this check is here, you definitely want to use `cudaMemcpyAsync`.

> 👤 **ikawrakow** replied on **2025-10-27** at **08:52:33**
> 
> Sure. The function needs cleaning up. But I see in mainline the copy indirection has been eliminated. If I try to do the same here, CUDA graphs get disabled, and I don't immediately see why this happens here but not there. Do you know why?

> 👤 **CISC** replied on **2025-10-27** at **08:58:56**
> 
> This?
> https://github.com/ikawrakow/ik_llama.cpp/blob/f76e98536f9377b29e2058cf83e0aa0450b631c8/ggml/src/ggml-cuda.cu#L3514-L3528

> 👤 **ikawrakow** replied on **2025-10-27** at **09:02:52**
> 
> I have tried disabling it (and all other places where `GGML_OP_CPY` is being handled differently), and yet CUDA_GRAPHS still get disabled.
> 
> I'm asking because if I could remove the copy indirection, then there is another minor optimization that can be done (which you then can copy to mainline :wink: )

> 👤 **CISC** replied on **2025-10-27** at **09:22:06**
> 
> Did you get these?
> https://github.com/ikawrakow/ik_llama.cpp/blob/f76e98536f9377b29e2058cf83e0aa0450b631c8/ggml/src/ggml-cuda.cu#L3558
> https://github.com/ikawrakow/ik_llama.cpp/blob/f76e98536f9377b29e2058cf83e0aa0450b631c8/ggml/src/ggml-cuda.cu#L3579

> 👤 **ikawrakow** replied on **2025-10-27** at **09:26:32**
> 
> Yes.

> 👤 **CISC** replied on **2025-10-27** at **09:31:19**
> 
> Maybe make a PR so I can see what changes you have made? :)

> 👤 **ikawrakow** replied on **2025-10-27** at **09:41:49**
> 
> [#871](https://github.com/ikawrakow/ik_llama.cpp/issues/871)

---

👤 **Nexesenex** commented on **2025-10-27** at **12:14:50**

> @Nexesenex Thanks for the report. Does the last commit fix the issue?

It apparently does, I merged the fix and compiled, then tried the same prompt again and 2 others (10, 12k tokens) with success, thank you @ikawrakow.

---

👤 **ikawrakow** commented on **2025-10-27** at **12:36:18**

@Nexesenex

Thanks!

Do you see any improvement with the dense 70B model you are testing with?

---

👤 **Nexesenex** commented on **2025-10-27** at **13:27:13**

> @Nexesenex
> 
> Thanks!
> 
> Do you see any improvement with the dense 70B model you are testing with?

For : `llama-server -m LLaMa-3.3-70B-Q5_K_M.gguf -ngl 150 -b 128 -mg 0 -ts 31,31,19 --no-mmap -c 65536 -ctk q6_0 -ctv q5_0 --host 127.0.0.1 --port 8080`

With main branch, for around 10k token prompt:
```

INFO [            update_slots] kv cache rm [p0, end) | tid="38076" timestamp=1761571060 id_slot=0 id_task=0 p0=9984
INFO [           print_timings] prompt eval time     =   20545.33 ms /  9991 tokens (    2.06 ms per token,   486.29 tokens per second) | tid="38076" timestamp=1761571086 id_slot=0 id_task=0 t_prompt_processing=20545.327 n_prompt_tokens_processed=9991 t_token=2.0563834451005905 n_tokens_second=486.290629494483
INFO [           print_timings] generation eval time =   25421.75 ms /   256 runs   (   99.30 ms per token,    10.07 tokens per second) | tid="38076" timestamp=1761571086 id_slot=0 id_task=0 t_token_generation=25421.75 n_decoded=256 t_token=99.3037109375 n_tokens_second=10.07011712411616
INFO [           print_timings]           total time =   45967.08 ms | tid="38076" timestamp=1761571086 id_slot=0 id_task=0 t_prompt_processing=20545.327 t_token_generation=25421.75 t_total=45967.077000000005
INFO [            update_slots] slot released | tid="38076" timestamp=1761571086 id_slot=0 id_task=0 n_ctx=65536 n_past=10246 n_system_tokens=0 n_cache_tokens=10246 truncated=false
```

With this PR, same prompt:

```
INFO [            update_slots] kv cache rm [p0, end) | tid="34088" timestamp=1761571261 id_slot=0 id_task=0 p0=9984
INFO [           print_timings] prompt eval time     =   20272.87 ms /  9991 tokens (    2.03 ms per token,   492.83 tokens per second) | tid="34088" timestamp=1761571287 id_slot=0 id_task=0 t_prompt_processing=20272.865 n_prompt_tokens_processed=9991 t_token=2.029112701431288 n_tokens_second=492.8262482880441
INFO [           print_timings] generation eval time =   25224.19 ms /   256 runs   (   98.53 ms per token,    10.15 tokens per second) | tid="34088" timestamp=1761571287 id_slot=0 id_task=0 t_token_generation=25224.185 n_decoded=256 t_token=98.53197265625 n_tokens_second=10.14898994754439
INFO [           print_timings]           total time =   45497.05 ms | tid="34088" timestamp=1761571287 id_slot=0 id_task=0 t_prompt_processing=20272.865 t_token_generation=25224.185 t_total=45497.05
INFO [            update_slots] slot released | tid="34088" timestamp=1761571287 id_slot=0 id_task=0 n_ctx=65536 n_past=10246 n_system_tokens=0 n_cache_tokens=10246 truncated=false
```

I made more shots, broadly there's +1-2.5% PP speed, and same for TG speed in favor of this PR. Definitely a keeper for L3.1/3.3 70B full Cuda offload on multi-GPU.

---

👤 **magikRUKKOLA** commented on **2025-10-28** at **23:15:53**

Whoa!

Yeah, this improves the things for sure.  Preliminary, I am getting the following for the Ling-1T with Ling-flash-2.0 with spec. decoding:

```
Statistics Summary:
==================
Total Data Points: 60

Acceptance Statistics:
  Mean:    62.2762
  Median:  62.6854
  Mode:    63.0000
  Std Dev: 2.1341
  Range:   [58.2631, 66.8065]

Speed Statistics:
  Mean:    7.09
  Median:  7.07
  Mode:    7.20
  Std Dev: 0.15
  Range:   [6.62, 7.47]
```

That is, 7.09 tps vs 7.04 tps (pr-863).  Its about +1% in decode.