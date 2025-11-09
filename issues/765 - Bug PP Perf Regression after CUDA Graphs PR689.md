## 📌 [Issue #765](https://github.com/ikawrakow/ik_llama.cpp/issues/765) - Bug: PP Perf Regression after CUDA Graphs PR[#689](https://github.com/ikawrakow/ik_llama.cpp/issues/689)

| **Author** | `usrlocalben` |
| :--- | :--- |
| **State** | ✅ **Open** |
| **Created** | 2025-09-07 |
| **Updated** | 2025-09-11 |

---

## 📄 Description

### What happened?

PP Perf decrease of ~25% following the CUDA Graphs PR [#689](https://github.com/ikawrakow/ik_llama.cpp/issues/689) .

I've been suspicious for a while by _seat-of-the-pants_ impression, but didn't take time to measure until now. 

Hardware is 2S EPYC 9115 w/RTX 8000 (Turing)

Tested with CUDA 12.6 and 13.0

aside: I don't observe any problems using CUDA 13.0

aside2: It seems unfortunate that there is so much refactoring, the gpt-oss addition, and whitespace changes mixed in with the graphs PR, it's at least difficult for an outsider to read and get a sense of which concept could be the cause.

```
-b 4096 -ub 4096
-mla 2 -fa -fmoe
-ngl 999 -ot exps=CPU
-m Kimi-K2-0905-DQ4_K.gguf
```

quant is my own, but the same config as [anikifoss' K2](https://huggingface.co/anikifoss/Kimi-K2-Instruct-DQ4_K).

`-DGGML_CUDA_USE_GRAPHS=OFF` in the build invocation was only added in the CUDA Graphs commit, normally I build without this line regardless of ON/OFF value.

Commit prior to CUDA Graphs (e082df4)
|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  4096 |   1024 |      0 |   44.583 |    91.87 |   93.665 |    10.93 |
|  4096 |   1024 |   4096 |   46.379 |    88.32 |   98.202 |    10.43 |
|  4096 |   1024 |   8192 |   48.673 |    84.15 |  104.752 |     9.78 |
|  4096 |   1024 |  12288 |   51.006 |    80.30 |  111.377 |     9.19 |
|  4096 |   1024 |  16384 |   53.209 |    76.98 |  119.035 |     8.60 |
|  4096 |   1024 |  20480 |   55.804 |    73.40 |  130.798 |     7.83 |
|  4096 |   1024 |  24576 |   57.336 |    71.44 |  137.132 |     7.47 |
|  4096 |   1024 |  28672 |   59.623 |    68.70 |  144.862 |     7.07 |


CUDA_GRAPHS (fc06bc9) (GGML_CUDA_USE_GRAPHS=default)
|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  4096 |   1024 |      0 |   61.110 |    67.03 |   93.308 |    10.97 |
|  4096 |   1024 |   4096 |   63.197 |    64.81 |   96.553 |    10.61 |
|  4096 |   1024 |   8192 |   65.332 |    62.69 |  101.798 |    10.06 |
|  4096 |   1024 |  12288 |   67.504 |    60.68 |  107.475 |     9.53 |
|  4096 |   1024 |  16384 |   69.694 |    58.77 |  113.288 |     9.04 |
|  4096 |   1024 |  20480 |   71.882 |    56.98 |  121.111 |     8.46 |
|  4096 |   1024 |  24576 |   74.105 |    55.27 |  128.320 |     7.98 |
|  4096 |   1024 |  28672 |   76.336 |    53.66 |  136.589 |     7.50 |


CUDA_GRAPHS (fc06bc9) (GGML_CUDA_USE_GRAPHS=off)
|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  4096 |   1024 |      0 |   61.094 |    67.04 |   93.771 |    10.92 |
|  4096 |   1024 |   4096 |   63.175 |    64.84 |   96.601 |    10.60 |
|  4096 |   1024 |   8192 |   65.287 |    62.74 |  101.879 |    10.05 |
|  4096 |   1024 |  12288 |   67.474 |    60.71 |  107.304 |     9.54 |
|  4096 |   1024 |  16384 |   69.637 |    58.82 |  113.538 |     9.02 |
|  4096 |   1024 |  20480 |   71.847 |    57.01 |  121.361 |     8.44 |
|  4096 |   1024 |  24576 |   74.121 |    55.26 |  128.583 |     7.96 |
|  4096 |   1024 |  28672 |   76.310 |    53.68 |  135.303 |     7.57 |


HEAD (c519d41)
|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  4096 |   1024 |      0 |   62.670 |    65.36 |   93.086 |    11.00 |
|  4096 |   1024 |   4096 |   64.768 |    63.24 |   96.333 |    10.63 |
|  4096 |   1024 |   8192 |   66.890 |    61.23 |  101.669 |    10.07 |
|  4096 |   1024 |  12288 |   69.054 |    59.32 |  107.363 |     9.54 |
|  4096 |   1024 |  16384 |   71.233 |    57.50 |  113.207 |     9.05 |
|  4096 |   1024 |  20480 |   73.414 |    55.79 |  121.171 |     8.45 |
|  4096 |   1024 |  24576 |   75.642 |    54.15 |  128.810 |     7.95 |
|  4096 |   1024 |  28672 |   77.859 |    52.61 |  136.124 |     7.52 |
|  4096 |   1024 |  32768 |   80.073 |    51.15 |  145.548 |     7.04 |


```
cmake -B build \
        -DCMAKE_BUILD_TYPE=Release \
        -DGGML_NATIVE=ON \
        -DGGML_CCACHE=OFF \
        -DGGML_CUDA=ON \
        -DBUILD_SHARED_LIBS=OFF \
        -DGGML_SCHED_MAX_COPIES=1 \
        -DGGML_VULKAN=OFF \
        -DGGML_RPC=OFF \
        -DGGML_BLAS=OFF \
        -DGGML_CUDA_F16=ON \
        -DGGML_CUDA_USE_GRAPHS=OFF \ 
        -DGGML_CUDA_IQK_FORCE_BF16=1

cmake --build build --config Release -j16
```



### Name and Version

version: 3881 (c519d417)
built with cc (Debian 12.2.0-14+deb12u1) 12.2.0 for x86_64-linux-gnu

### What operating system are you seeing the problem on?

Linux

### Relevant log output

```shell

```

---

## 💬 Conversation

👤 **ikawrakow** commented on **2025-09-07** at **14:49:47**

I'm on vacation and will not be able to look into this for 2 weeks. But the CUDA graphs PR should not influence PP in any way, for MoE models it only enables graphs for TG, and it does not change how PP is done. If anything, the later PRs about PP optimisations for small batch sizes have a much higher chance of impacting Kimi PP performance.

---

👤 **usrlocalben** commented on **2025-09-07** at **15:10:55**

I was checking my work and noticed something that may narrow it down: Offload Policy.

The change observed in good/bad commit is the same as observed in modifying the offload policy.


modified policy:
-op 26,0,27,0,29,0  # slow pcie
XXXXXXXXXXXXXXXXXXXXX Setting offload policy for op MULTI_ADD to OFF
XXXXXXXXXXXXXXXXXXXXX Setting offload policy for op MUL_MAT to OFF
XXXXXXXXXXXXXXXXXXXXX Setting offload policy for op OUT_PROD to OFF

Good = e082df4 (commit prior to PR [#689](https://github.com/ikawrakow/ik_llama.cpp/issues/689))
Bad = fc06bc9 (PR [#689](https://github.com/ikawrakow/ik_llama.cpp/issues/689))

### Good Commit, Modified Offload Policy
|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  4096 |   1024 |      0 |   44.273 |    92.52 |   93.525 |    10.95 |
|  4096 |   1024 |   4096 |   46.003 |    89.04 |   97.695 |    10.48 |


### Good Commit, Default Offload Policy
|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  4096 |   1024 |      0 |   61.211 |    66.92 |   93.331 |    10.97 |
|  4096 |   1024 |   4096 |   63.343 |    64.66 |   97.517 |    10.50 |


### Bad Commit, Modified Offload Policy
|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  4096 |   1024 |      0 |   61.076 |    67.06 |   93.309 |    10.97 |
|  4096 |   1024 |   4096 |   63.165 |    64.85 |   96.312 |    10.63 |


### Bad Commit, Default Offload Policy
|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  4096 |   1024 |      0 |   61.103 |    67.03 |   93.172 |    10.99 |
|  4096 |   1024 |   4096 |   63.193 |    64.82 |   97.238 |    10.53 |

---

👤 **Ph0rk0z** commented on **2025-09-07** at **18:01:37**

>-DGGML_CUDA_IQK_FORCE_BF16=1

This is theoretically a nono on turning.

---

👤 **ikawrakow** commented on **2025-09-08** at **07:43:34**

Oh, this is related to offloading or not offloading experts to the GPU. Looking at the output, the enum values for the operations have changed. What you want is to not offload MUL_MAT_ID and MOE_UP_GATE. Try -op 28,0,30,0

---

👤 **sousekd** commented on **2025-09-10** at **17:21:01**

> Oh, this is related to offloading or not offloading experts to the GPU. Looking at the output, the enum values for the operations have changed. What you want is to not offload MUL_MAT_ID and MOE_UP_GATE. Try -op 28,0,30,0

Does this depend on the model? This is `ubergarm/DeepSeek-R1-0528-IQ4_KS_R4` tested using EPYC 9355 + RTX 6000 on the current main branch (d323871b) with:

```
./llama-sweep-bench \
    --model "$MODEL_PATH" \
    --no-mmap \
    -mla 3 -fa -fmoe \
    -amb 512 -b 8192 -ub 8192 \
    -ctk f16 \
    -c 262144 \
    -ngl 999 \
    -ot "blk\.[3-6]\.ffn_.*=CUDA0" \
    -ot exps=CPU \
    --parallel 2 \
    --threads 16 \
    --threads-batch 24 \
    --warmup-batch
```

## -op 27,0,29,0

```
XXXXXXXXXXXXXXXXXXXXX Setting offload policy for op MUL_MAT to OFF
XXXXXXXXXXXXXXXXXXXXX Setting offload policy for op OUT_PROD to OFF

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  8192 |   2048 |      0 |   14.798 |   553.57 |  119.408 |    17.15 |
|  8192 |   2048 |   8192 |   17.690 |   463.08 |  123.289 |    16.61 |
```

## -op 28,0,30,0

```
XXXXXXXXXXXXXXXXXXXXX Setting offload policy for op MUL_MAT_ID to OFF
XXXXXXXXXXXXXXXXXXXXX Setting offload policy for op FUSED_UP_GATE to OFF

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  8192 |   2048 |      0 |   33.092 |   247.55 |  121.040 |    16.92 |
|  8192 |   2048 |   8192 |   36.153 |   226.59 |  123.261 |    16.62 |
```

## -op 28,0,31,0

```
XXXXXXXXXXXXXXXXXXXXX Setting offload policy for op MUL_MAT_ID to OFF
XXXXXXXXXXXXXXXXXXXXX Setting offload policy for op MOE_FUSED_UP_GATE to OFF

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  8192 |   2048 |      0 |   57.912 |   141.46 |  119.270 |    17.17 |
|  8192 |   2048 |   8192 |   60.734 |   134.88 |  122.775 |    16.68 |
```

## without -op

```
|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  8192 |   2048 |      0 |   14.895 |   549.97 |  120.594 |    16.98 |
|  8192 |   2048 |   8192 |   17.838 |   459.24 |  124.075 |    16.51 |
```

Sorry I don't have Kimi K2 available at the moment.
And I did not manage to find MOE_UP_GATE :).

---

👤 **ikawrakow** commented on **2025-09-11** at **12:20:57**

> And I did not manage to find MOE_UP_GATE :). 

You did: -op 28,0,31,0. But you have a very fast GPU and PCIE, so not offloading is slower in your case.

---

👤 **sousekd** commented on **2025-09-11** at **12:54:05**

> You did: -op 28,0,31,0. But you have a very fast GPU and PCIE, so not offloading is slower in your case.

Ah, okay. I know that `-op 27,0,29,0` helped me on larger batch sizes in the past... on Windows.