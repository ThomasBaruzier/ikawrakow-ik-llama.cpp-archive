## 🗣️ [Discussion #758](https://github.com/ikawrakow/ik_llama.cpp/discussions/758) - GPT-OSS performance

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | ✅ **Open** |
| **Created** | 2025-09-04 |
| **Updated** | 2025-10-27 |

---

## 📄 Description

I see currently 2 of the 3 pinned items on the mainline `llama.cpp` [discussion page](https://github.com/ggml-org/llama.cpp/discussions) are devoted to GPT-OSS, so I thought it is useful to have one here too.

# Support

Support for the `GPT-OSS` models was added to `ik_llama.cpp` in PR [#689](https://github.com/ikawrakow/ik_llama.cpp/issues/689). Support for `MAXFP4`, the `fp4` variant used in these models was added in [#682](https://github.com/ikawrakow/ik_llama.cpp/issues/682). Improvements for the Harmony chat template and tool call support followed in [#677](https://github.com/ikawrakow/ik_llama.cpp/issues/677) and [#723](https://github.com/ikawrakow/ik_llama.cpp/issues/723).
Performance for these models was improved via SWA-specific optimizations in [#692](https://github.com/ikawrakow/ik_llama.cpp/issues/692), [#754](https://github.com/ikawrakow/ik_llama.cpp/issues/754) and [#757](https://github.com/ikawrakow/ik_llama.cpp/issues/757). Hybrid inference PP performance was increased in [#698](https://github.com/ikawrakow/ik_llama.cpp/issues/698) by offloading only the activated experts to the GPU (a feature that appears to be only useful for the GPT-OSS models). 

# Performance

There are two main advantages of `ik_llama.cpp` over `llama.cpp`: availability of higher quality quantization types and better performance. Hence, it makes sense to compare `ik_llama.cpp` and `llama.cpp` performance for GPT-OSS. I'll only provide results for the native `MXFP4` quantization type. All results reported here are for the 20B variant.

The GPT-OSS models use sliding window attention (SWA) for every second layer, with a fairly small windows size of 128 tokens. There is a significant difference between `ik_llama.cpp` and `llama.cpp` how SWA layers are handled. In mainline, by default only the tokens required to compute the last `u-batch` are kept in the cache. This has the distinct advantage of reducing KV-cache size by nearly a factor of 2. The downside of this is that the KV-cache is not reusable. To be able to reuse the KV-cache, one needs to add `--swa-full` to the `llama.cpp` command line. This keeps the full KV-cache also for SWA layers, and corresponds to what is done in `ik_llama.cpp`. Hence, when comparing performance, I'll provide `llama.cpp` results with and without `--swas-full`, the latter being a more fair comparison (due to the usability limitations one gets without `--swa-full`).

## CUDA performance, GPT-OSS-20B-MXFP4

Tests are run on an RTX-4080 GPU in a Ryzen-7950X rig. 

Command line is
```
./bin/llama-sweep-bench -m $model -c 30720 -ub 2048 -t 1 -ngl 100 [-fa -fmoe]
```
(the arguments in square brackets only apply to `ik_llama.cpp`). 

### PP performance

<img width="792" height="612" alt="u6_pp" src="https://github.com/user-attachments/assets/e19588f7-c458-48ed-9b3f-ab421e5bd789" />

Here `ik_llama.cpp` is ~40% faster at small `N_KV`, and 65% faster at 30k tokens compared to the corresponding calculation with `llama.cpp` using `--swa-full`.

### TG performance

<img width="792" height="612" alt="u6_tg" src="https://github.com/user-attachments/assets/2d6ae2dc-8981-4a57-b5d8-c1397fcd04d6" />

Here performance is similar at low context length, but `ik_llama.cpp` is ~15% faster at 30k tokens compared to `--swa-full`. `llama.cpp` manages to outperform `ik_llama.cpp` by a small margin (4.5% at 30k tokens) for large `N_KV` when not using `--swa-full`. I think this is simply due to the much smaller KQ mask for the SWA layers that has to be copied to the GPU for each token prediction. The `ik_llama.cpp` mask is 4 MiB at 32k tokens, which takes 0.27 ms for a 15 GiB/s PCI-E, which is ~3.5% of the time taken per token generation at 130 t/s. Without `--swa-full` the `llama.cpp` KQ SWA mask is ~15X smaller, so time to copy to the GPU is negligible.     

<details>
<summary>llama.cpp with --swa-full</summary>

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  2048 |    512 |      0 |    0.314 |  6530.99 |    2.751 |   186.09 |
|  2048 |    512 |   2048 |    0.300 |  6827.90 |    2.988 |   171.33 |
|  2048 |    512 |   4096 |    0.326 |  6275.32 |    3.203 |   159.87 |
|  2048 |    512 |   6144 |    0.353 |  5799.25 |    3.421 |   149.65 |
|  2048 |    512 |   8192 |    0.385 |  5322.84 |    3.316 |   154.43 |
|  2048 |    512 |  10240 |    0.415 |  4931.17 |    3.444 |   148.67 |
|  2048 |    512 |  12288 |    0.449 |  4560.65 |    3.599 |   142.27 |
|  2048 |    512 |  14336 |    0.473 |  4327.45 |    3.693 |   138.63 |
|  2048 |    512 |  16384 |    0.506 |  4044.18 |    3.798 |   134.81 |
|  2048 |    512 |  18432 |    0.534 |  3832.20 |    3.907 |   131.05 |
|  2048 |    512 |  20480 |    0.563 |  3634.44 |    4.018 |   127.44 |
|  2048 |    512 |  22528 |    0.587 |  3485.98 |    4.131 |   123.94 |
|  2048 |    512 |  24576 |    0.614 |  3334.87 |    4.261 |   120.15 |
|  2048 |    512 |  26624 |    0.652 |  3142.10 |    4.351 |   117.69 |
|  2048 |    512 |  28672 |    0.677 |  3023.10 |    4.463 |   114.71 |

</details>

<details>
<summary>llama.cpp without --swa-full</summary>

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  2048 |    512 |      0 |    0.312 |  6560.28 |    2.751 |   186.13 |
|  2048 |    512 |   2048 |    0.285 |  7181.66 |    2.984 |   171.59 |
|  2048 |    512 |   4096 |    0.303 |  6762.65 |    3.091 |   165.66 |
|  2048 |    512 |   6144 |    0.315 |  6504.91 |    3.204 |   159.80 |
|  2048 |    512 |   8192 |    0.335 |  6111.65 |    3.155 |   162.28 |
|  2048 |    512 |  10240 |    0.347 |  5910.30 |    3.210 |   159.48 |
|  2048 |    512 |  12288 |    0.365 |  5616.79 |    3.288 |   155.74 |
|  2048 |    512 |  14336 |    0.385 |  5316.97 |    3.330 |   153.73 |
|  2048 |    512 |  16384 |    0.402 |  5092.58 |    3.388 |   151.10 |
|  2048 |    512 |  18432 |    0.411 |  4977.99 |    3.449 |   148.44 |
|  2048 |    512 |  20480 |    0.434 |  4724.01 |    3.499 |   146.32 |
|  2048 |    512 |  22528 |    0.450 |  4553.29 |    3.559 |   143.86 |
|  2048 |    512 |  24576 |    0.462 |  4434.42 |    3.623 |   141.31 |
|  2048 |    512 |  26624 |    0.482 |  4252.86 |    3.664 |   139.76 |
|  2048 |    512 |  28672 |    0.498 |  4109.87 |    3.724 |   137.49 |

</details>

<details>
<summary>ik_llama.cpp</summary>

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  2048 |    512 |      0 |    0.205 | 10003.96 |    2.871 |   178.33 |
|  2048 |    512 |   2048 |    0.208 |  9826.36 |    2.964 |   172.75 |
|  2048 |    512 |   4096 |    0.230 |  8917.26 |    3.033 |   168.80 |
|  2048 |    512 |   6144 |    0.246 |  8330.35 |    3.122 |   164.01 |
|  2048 |    512 |   8192 |    0.263 |  7787.72 |    3.196 |   160.22 |
|  2048 |    512 |  10240 |    0.276 |  7409.15 |    3.274 |   156.40 |
|  2048 |    512 |  12288 |    0.292 |  7014.40 |    3.345 |   153.09 |
|  2048 |    512 |  14336 |    0.307 |  6661.16 |    3.412 |   150.08 |
|  2048 |    512 |  16384 |    0.322 |  6361.24 |    3.473 |   147.41 |
|  2048 |    512 |  18432 |    0.336 |  6102.96 |    3.543 |   144.53 |
|  2048 |    512 |  20480 |    0.351 |  5832.50 |    3.615 |   141.63 |
|  2048 |    512 |  22528 |    0.367 |  5586.18 |    3.685 |   138.95 |
|  2048 |    512 |  24576 |    0.381 |  5373.61 |    3.762 |   136.11 |
|  2048 |    512 |  26624 |    0.396 |  5174.20 |    3.833 |   133.59 |
|  2048 |    512 |  28672 |    0.410 |  4991.07 |    3.889 |   131.67 |

</details>

## CPU performance, GPT-OSS-20B-MXFP4

Tests are run on a Ryzen-7950X CPU. The command line used is
```
./bin/llama-sweep-bench -m $model -c 32768 -t 16 -ub 1024 -ctk q8_0 [-fa -fmoe]
```

### PP performance

<img width="792" height="612" alt="u7_pp" src="https://github.com/user-attachments/assets/2dcff5cf-b55e-4df8-8bde-1ac5d2214cfd" />

Not much comment is needed here. `ik_llama.cpp` is 3.6X faster at zero context, and ~8.8X faster at 25k tokens (I did not have the patience to wait for the `llama.cpp` results up two 32k tokens). There is no difference with and without `--swa-full` when running `llama.cpp` on the CPU 

### TG performance

<img width="792" height="612" alt="u7_tg" src="https://github.com/user-attachments/assets/ce6a5ffe-3d69-49cf-ac53-5ffd6bd0c1c9" />

Here performance is about the same at zero context length. At 25k tokens `ik_llama.cpp` is about 1.5X faster.

<details>
<summary>llama.cpp with --swa-full</summary>

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  1024 |    256 |      0 |    7.280 |   140.67 |   11.803 |    21.69 |
|  1024 |    256 |   1024 |    8.072 |   126.86 |   12.175 |    21.03 |
|  1024 |    256 |   2048 |    8.910 |   114.93 |   12.525 |    20.44 |
|  1024 |    256 |   3072 |    9.808 |   104.40 |   12.809 |    19.99 |
|  1024 |    256 |   4096 |   10.789 |    94.91 |   13.162 |    19.45 |
|  1024 |    256 |   5120 |   11.825 |    86.60 |   13.543 |    18.90 |
|  1024 |    256 |   6144 |   12.874 |    79.54 |   14.165 |    18.07 |
|  1024 |    256 |   7168 |   13.907 |    73.63 |   14.675 |    17.44 |
|  1024 |    256 |   8192 |   14.834 |    69.03 |   15.153 |    16.89 |
|  1024 |    256 |   9216 |   15.838 |    64.66 |   15.408 |    16.62 |
|  1024 |    256 |  10240 |   16.862 |    60.73 |   15.600 |    16.41 |
|  1024 |    256 |  11264 |   17.842 |    57.39 |   16.012 |    15.99 |
|  1024 |    256 |  12288 |   18.768 |    54.56 |   16.931 |    15.12 |
|  1024 |    256 |  13312 |   19.768 |    51.80 |   16.885 |    15.16 |
|  1024 |    256 |  14336 |   20.800 |    49.23 |   17.444 |    14.68 |
|  1024 |    256 |  15360 |   21.984 |    46.58 |   18.309 |    13.98 |
|  1024 |    256 |  16384 |   23.195 |    44.15 |   17.529 |    14.60 |
|  1024 |    256 |  17408 |   24.539 |    41.73 |   17.947 |    14.26 |
|  1024 |    256 |  18432 |   26.365 |    38.84 |   19.978 |    12.81 |
|  1024 |    256 |  19456 |   28.280 |    36.21 |   20.000 |    12.80 |
|  1024 |    256 |  20480 |   29.668 |    34.51 |   20.748 |    12.34 |
|  1024 |    256 |  21504 |   32.048 |    31.95 |   20.688 |    12.37 |
|  1024 |    256 |  22528 |   34.715 |    29.50 |   21.426 |    11.95 |
|  1024 |    256 |  23552 |   37.818 |    27.08 |   22.884 |    11.19 |
|  1024 |    256 |  24576 |   39.904 |    25.66 |   24.546 |    10.43 |

</details>

<details>
<summary>llama.cpp without --swa-full</summary>

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  1024 |    256 |      0 |    7.157 |   143.07 |   11.798 |    21.70 |
|  1024 |    256 |   1024 |    8.034 |   127.46 |   12.169 |    21.04 |
|  1024 |    256 |   2048 |    8.826 |   116.02 |   12.497 |    20.48 |
|  1024 |    256 |   3072 |    9.702 |   105.54 |   12.860 |    19.91 |
|  1024 |    256 |   4096 |   10.569 |    96.89 |   13.327 |    19.21 |
|  1024 |    256 |   5120 |   11.512 |    88.95 |   13.649 |    18.76 |
|  1024 |    256 |   6144 |   12.491 |    81.98 |   13.963 |    18.33 |
|  1024 |    256 |   7168 |   13.439 |    76.20 |   14.555 |    17.59 |
|  1024 |    256 |   8192 |   14.417 |    71.03 |   14.805 |    17.29 |
|  1024 |    256 |   9216 |   15.420 |    66.41 |   15.125 |    16.93 |
|  1024 |    256 |  10240 |   16.291 |    62.86 |   15.595 |    16.42 |
|  1024 |    256 |  11264 |   17.155 |    59.69 |   16.004 |    16.00 |
|  1024 |    256 |  12288 |   18.151 |    56.42 |   15.943 |    16.06 |
|  1024 |    256 |  13312 |   19.160 |    53.44 |   17.927 |    14.28 |
|  1024 |    256 |  14336 |   20.046 |    51.08 |   18.445 |    13.88 |
|  1024 |    256 |  15360 |   21.089 |    48.56 |   18.030 |    14.20 |
|  1024 |    256 |  16384 |   22.117 |    46.30 |   19.256 |    13.29 |
|  1024 |    256 |  17408 |   23.523 |    43.53 |   17.822 |    14.36 |
|  1024 |    256 |  18432 |   25.182 |    40.66 |   19.401 |    13.20 |
|  1024 |    256 |  19456 |   27.343 |    37.45 |   19.626 |    13.04 |
|  1024 |    256 |  20480 |   29.099 |    35.19 |   19.577 |    13.08 |
|  1024 |    256 |  21504 |   31.572 |    32.43 |   20.196 |    12.68 |
|  1024 |    256 |  22528 |   32.758 |    31.26 |   20.886 |    12.26 |
|  1024 |    256 |  23552 |   36.318 |    28.20 |   22.498 |    11.38 |
|  1024 |    256 |  24576 |   38.426 |    26.65 |   23.330 |    10.97 |

</details>

<details>
<summary>ik_llama.cpp</summary>

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  1024 |    256 |      0 |    1.993 |   513.76 |   11.842 |    21.62 |
|  1024 |    256 |   1024 |    2.053 |   498.86 |   12.038 |    21.27 |
|  1024 |    256 |   2048 |    2.133 |   480.17 |   12.245 |    20.91 |
|  1024 |    256 |   3072 |    2.232 |   458.81 |   12.430 |    20.59 |
|  1024 |    256 |   4096 |    2.314 |   442.56 |   12.599 |    20.32 |
|  1024 |    256 |   5120 |    2.404 |   425.97 |   12.746 |    20.08 |
|  1024 |    256 |   6144 |    2.497 |   410.12 |   12.900 |    19.84 |
|  1024 |    256 |   7168 |    2.587 |   395.89 |   13.040 |    19.63 |
|  1024 |    256 |   8192 |    2.682 |   381.83 |   13.180 |    19.42 |
|  1024 |    256 |   9216 |    2.774 |   369.18 |   13.319 |    19.22 |
|  1024 |    256 |  10240 |    2.894 |   353.79 |   13.594 |    18.83 |
|  1024 |    256 |  11264 |    3.172 |   322.85 |   13.663 |    18.74 |
|  1024 |    256 |  12288 |    3.271 |   313.05 |   13.779 |    18.58 |
|  1024 |    256 |  13312 |    3.251 |   314.96 |   13.777 |    18.58 |
|  1024 |    256 |  14336 |    3.624 |   282.56 |   14.228 |    17.99 |
|  1024 |    256 |  15360 |    3.511 |   291.66 |   14.155 |    18.08 |
|  1024 |    256 |  16384 |    3.504 |   292.25 |   14.518 |    17.63 |
|  1024 |    256 |  17408 |    3.793 |   269.98 |   14.338 |    17.85 |
|  1024 |    256 |  18432 |    3.972 |   257.79 |   14.488 |    17.67 |
|  1024 |    256 |  19456 |    4.127 |   248.12 |   14.776 |    17.33 |
|  1024 |    256 |  20480 |    3.798 |   269.60 |   15.039 |    17.02 |
|  1024 |    256 |  21504 |    4.409 |   232.24 |   14.810 |    17.29 |
|  1024 |    256 |  22528 |    4.194 |   244.14 |   15.571 |    16.44 |
|  1024 |    256 |  23552 |    4.313 |   237.42 |   15.386 |    16.64 |
|  1024 |    256 |  24576 |    4.526 |   226.24 |   15.575 |    16.44 |
|  1024 |    256 |  25600 |    4.482 |   228.45 |   15.735 |    16.27 |
|  1024 |    256 |  26624 |    4.482 |   228.49 |   16.177 |    15.82 |
|  1024 |    256 |  27648 |    4.537 |   225.69 |   16.216 |    15.79 |
|  1024 |    256 |  28672 |    4.756 |   215.33 |   16.305 |    15.70 |
|  1024 |    256 |  29696 |    4.735 |   216.27 |   16.765 |    15.27 |
|  1024 |    256 |  30720 |    5.095 |   200.97 |   16.782 |    15.25 |
|  1024 |    256 |  31744 |    5.113 |   200.28 |   16.992 |    15.07 |

</details>

---

## 💬 Discussion

👤 **AlekXL** commented on **2025-09-14** at **08:47:50**

So no GPT-OSS 120B support? At least I can't make it output coherent text - only gibberish. Otherwise provide to-the-point recipe: which quant and how to run it.

> 👤 **ikawrakow** replied on **2025-09-14** at **09:14:59**
> 
> I have tested with the MXFP4 model from ggml.org and it works fine

> 👤 **Djip007** replied on **2025-10-23** at **23:01:47**
> 
> On  AMD RYZEN AI MAX+ 395 (32) / 128Go :
> 
> | model                          |       size |     params | backend    | threads | n_ubatch | type_k | type_v | fa |          test |              t/s |
> | ------------------------------ | ---------: | ---------: | ---------- | ------: | -------: | -----: | -----: | -: | ------------: | ---------------: |
> | gpt-oss 120B MXFP4 - 4.25 bpw  |  59.02 GiB |   116.83 B | CPU        |      16 |     4096 |   bf16 |   bf16 |  0 |           pp1 |     28.52 ± 0.02 |
> | gpt-oss 120B MXFP4 - 4.25 bpw  |  59.02 GiB |   116.83 B | CPU        |      16 |     4096 |   bf16 |   bf16 |  0 |           pp2 |     41.77 ± 1.47 |
> | gpt-oss 120B MXFP4 - 4.25 bpw  |  59.02 GiB |   116.83 B | CPU        |      16 |     4096 |   bf16 |   bf16 |  0 |           pp4 |     63.09 ± 4.71 |
> | gpt-oss 120B MXFP4 - 4.25 bpw  |  59.02 GiB |   116.83 B | CPU        |      16 |     4096 |   bf16 |   bf16 |  0 |           pp8 |     92.49 ± 4.12 |
> | gpt-oss 120B MXFP4 - 4.25 bpw  |  59.02 GiB |   116.83 B | CPU        |      16 |     4096 |   bf16 |   bf16 |  0 |          pp16 |    116.18 ± 4.06 |
> | gpt-oss 120B MXFP4 - 4.25 bpw  |  59.02 GiB |   116.83 B | CPU        |      16 |     4096 |   bf16 |   bf16 |  0 |          pp32 |    149.21 ± 4.35 |
> | gpt-oss 120B MXFP4 - 4.25 bpw  |  59.02 GiB |   116.83 B | CPU        |      16 |     4096 |   bf16 |   bf16 |  0 |          pp64 |    216.87 ± 5.22 |
> | gpt-oss 120B MXFP4 - 4.25 bpw  |  59.02 GiB |   116.83 B | CPU        |      16 |     4096 |   bf16 |   bf16 |  0 |         pp128 |    278.34 ± 6.63 |
> | gpt-oss 120B MXFP4 - 4.25 bpw  |  59.02 GiB |   116.83 B | CPU        |      16 |     4096 |   bf16 |   bf16 |  0 |         pp256 |    320.98 ± 1.92 |
> | gpt-oss 120B MXFP4 - 4.25 bpw  |  59.02 GiB |   116.83 B | CPU        |      16 |     4096 |   bf16 |   bf16 |  0 |         pp512 |    332.53 ± 2.02 |
> | gpt-oss 120B MXFP4 - 4.25 bpw  |  59.02 GiB |   116.83 B | CPU        |      16 |     4096 |   bf16 |   bf16 |  0 |        pp1024 |    306.91 ± 1.25 |
> | gpt-oss 120B MXFP4 - 4.25 bpw  |  59.02 GiB |   116.83 B | CPU        |      16 |     4096 |   bf16 |   bf16 |  0 |        pp2048 |    228.94 ± 3.83 |
> | gpt-oss 120B MXFP4 - 4.25 bpw  |  59.02 GiB |   116.83 B | CPU        |      16 |     4096 |   bf16 |   bf16 |  0 |        pp4096 |    200.31 ± 2.29 |
> | gpt-oss 120B MXFP4 - 4.25 bpw  |  59.02 GiB |   116.83 B | CPU        |      16 |     4096 |   bf16 |   bf16 |  0 |          tg32 |     28.58 ± 0.03 |
> | gpt-oss 120B MXFP4 - 4.25 bpw  |  59.02 GiB |   116.83 B | CPU        |      16 |     4096 |   bf16 |   bf16 |  1 |           pp1 |     28.67 ± 0.01 |
> | gpt-oss 120B MXFP4 - 4.25 bpw  |  59.02 GiB |   116.83 B | CPU        |      16 |     4096 |   bf16 |   bf16 |  1 |           pp2 |     41.73 ± 1.88 |
> | gpt-oss 120B MXFP4 - 4.25 bpw  |  59.02 GiB |   116.83 B | CPU        |      16 |     4096 |   bf16 |   bf16 |  1 |           pp4 |     64.59 ± 1.74 |
> | gpt-oss 120B MXFP4 - 4.25 bpw  |  59.02 GiB |   116.83 B | CPU        |      16 |     4096 |   bf16 |   bf16 |  1 |           pp8 |     90.43 ± 6.76 |
> | gpt-oss 120B MXFP4 - 4.25 bpw  |  59.02 GiB |   116.83 B | CPU        |      16 |     4096 |   bf16 |   bf16 |  1 |          pp16 |    116.63 ± 4.61 |
> | gpt-oss 120B MXFP4 - 4.25 bpw  |  59.02 GiB |   116.83 B | CPU        |      16 |     4096 |   bf16 |   bf16 |  1 |          pp32 |    149.20 ± 3.57 |
> | gpt-oss 120B MXFP4 - 4.25 bpw  |  59.02 GiB |   116.83 B | CPU        |      16 |     4096 |   bf16 |   bf16 |  1 |          pp64 |    220.87 ± 3.26 |
> | gpt-oss 120B MXFP4 - 4.25 bpw  |  59.02 GiB |   116.83 B | CPU        |      16 |     4096 |   bf16 |   bf16 |  1 |         pp128 |   281.63 ± 10.09 |
> | gpt-oss 120B MXFP4 - 4.25 bpw  |  59.02 GiB |   116.83 B | CPU        |      16 |     4096 |   bf16 |   bf16 |  1 |         pp256 |    335.13 ± 3.83 |
> | gpt-oss 120B MXFP4 - 4.25 bpw  |  59.02 GiB |   116.83 B | CPU        |      16 |     4096 |   bf16 |   bf16 |  1 |         pp512 |    387.05 ± 4.75 |
> | gpt-oss 120B MXFP4 - 4.25 bpw  |  59.02 GiB |   116.83 B | CPU        |      16 |     4096 |   bf16 |   bf16 |  1 |        pp1024 |    419.31 ± 4.89 |
> | gpt-oss 120B MXFP4 - 4.25 bpw  |  59.02 GiB |   116.83 B | CPU        |      16 |     4096 |   bf16 |   bf16 |  1 |        pp2048 |    432.76 ± 2.65 |
> | gpt-oss 120B MXFP4 - 4.25 bpw  |  59.02 GiB |   116.83 B | CPU        |      16 |     4096 |   bf16 |   bf16 |  1 |        pp4096 |    415.24 ± 1.01 |
> | gpt-oss 120B MXFP4 - 4.25 bpw  |  59.02 GiB |   116.83 B | CPU        |      16 |     4096 |   bf16 |   bf16 |  1 |          tg32 |     28.73 ± 0.02 |
> | gpt-oss 120B MXFP4 - 4.25 bpw  |  59.02 GiB |   116.83 B | CPU        |      16 |     4096 |   q8_0 |   q8_0 |  1 |           pp1 |     28.55 ± 0.04 |
> | gpt-oss 120B MXFP4 - 4.25 bpw  |  59.02 GiB |   116.83 B | CPU        |      16 |     4096 |   q8_0 |   q8_0 |  1 |           pp2 |     42.46 ± 1.55 |
> | gpt-oss 120B MXFP4 - 4.25 bpw  |  59.02 GiB |   116.83 B | CPU        |      16 |     4096 |   q8_0 |   q8_0 |  1 |           pp4 |     62.05 ± 6.83 |
> | gpt-oss 120B MXFP4 - 4.25 bpw  |  59.02 GiB |   116.83 B | CPU        |      16 |     4096 |   q8_0 |   q8_0 |  1 |           pp8 |     92.30 ± 3.85 |
> | gpt-oss 120B MXFP4 - 4.25 bpw  |  59.02 GiB |   116.83 B | CPU        |      16 |     4096 |   q8_0 |   q8_0 |  1 |          pp16 |    115.83 ± 3.81 |
> | gpt-oss 120B MXFP4 - 4.25 bpw  |  59.02 GiB |   116.83 B | CPU        |      16 |     4096 |   q8_0 |   q8_0 |  1 |          pp32 |    149.40 ± 3.75 |
> | gpt-oss 120B MXFP4 - 4.25 bpw  |  59.02 GiB |   116.83 B | CPU        |      16 |     4096 |   q8_0 |   q8_0 |  1 |          pp64 |    218.15 ± 5.20 |
> | gpt-oss 120B MXFP4 - 4.25 bpw  |  59.02 GiB |   116.83 B | CPU        |      16 |     4096 |   q8_0 |   q8_0 |  1 |         pp128 |    288.04 ± 7.47 |
> | gpt-oss 120B MXFP4 - 4.25 bpw  |  59.02 GiB |   116.83 B | CPU        |      16 |     4096 |   q8_0 |   q8_0 |  1 |         pp256 |    348.56 ± 1.85 |
> | gpt-oss 120B MXFP4 - 4.25 bpw  |  59.02 GiB |   116.83 B | CPU        |      16 |     4096 |   q8_0 |   q8_0 |  1 |         pp512 |    390.93 ± 2.66 |
> | gpt-oss 120B MXFP4 - 4.25 bpw  |  59.02 GiB |   116.83 B | CPU        |      16 |     4096 |   q8_0 |   q8_0 |  1 |        pp1024 |    432.22 ± 3.77 |
> | gpt-oss 120B MXFP4 - 4.25 bpw  |  59.02 GiB |   116.83 B | CPU        |      16 |     4096 |   q8_0 |   q8_0 |  1 |        pp2048 |    449.12 ± 1.44 |
> | gpt-oss 120B MXFP4 - 4.25 bpw  |  59.02 GiB |   116.83 B | CPU        |      16 |     4096 |   q8_0 |   q8_0 |  1 |        pp4096 |    432.58 ± 0.94 |
> | gpt-oss 120B MXFP4 - 4.25 bpw  |  59.02 GiB |   116.83 B | CPU        |      16 |     4096 |   q8_0 |   q8_0 |  1 |          tg32 |     28.69 ± 0.04 |

> 👤 **Djip007** replied on **2025-10-23** at **23:19:50**
> 
> llama-sweep-bench -m openai_gpt-oss-120b/MXFP4.gguf -c 32768 -t 16 -ub 2048 -ctk q8_0 -fa -fmoe
> 
> |    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
> |-------|--------|--------|----------|----------|----------|----------|
> |  2048 |    512 |      0 |    4.310 |   475.15 |   17.674 |    28.97 |
> |  2048 |    512 |   2048 |    4.865 |   420.98 |   18.072 |    28.33 |
> |  2048 |    512 |   4096 |    4.990 |   410.44 |   18.465 |    27.73 |
> |  2048 |    512 |   6144 |    5.351 |   382.76 |   18.857 |    27.15 |
> |  2048 |    512 |   8192 |    5.671 |   361.14 |   19.318 |    26.50 |
> |  2048 |    512 |  10240 |    6.011 |   340.72 |   19.847 |    25.80 |
> |  2048 |    512 |  12288 |    6.344 |   322.84 |   20.337 |    25.18 |
> |  2048 |    512 |  14336 |    6.743 |   303.74 |   20.840 |    24.57 |
> |  2048 |    512 |  16384 |    7.139 |   286.87 |   21.433 |    23.89 |
> |  2048 |    512 |  18432 |    7.527 |   272.08 |   21.774 |    23.51 |
> |  2048 |    512 |  20480 |    7.966 |   257.10 |   22.232 |    23.03 |
> |  2048 |    512 |  22528 |    8.577 |   238.79 |   22.861 |    22.40 |
> |  2048 |    512 |  24576 |   10.029 |   204.21 |   23.583 |    21.71 |
> |  2048 |    512 |  26624 |   10.992 |   186.31 |   23.866 |    21.45 |
> |  2048 |    512 |  28672 |   11.294 |   181.33 |   24.363 |    21.02 |
> |  2048 |    512 |  30720 |   11.893 |   172.20 |   25.307 |    20.23 |

> 👤 **ikawrakow** replied on **2025-10-24** at **04:29:23**
> 
> Thanks for these results. This is CPU only, right?

> 👤 **edyapd** replied on **2025-10-25** at **13:38:57**
> 
> > Thanks for these results. This is CPU only, right?
> 
> If I understand correctly, this is a processor with an integrated graphics core. It uses up to 96 GB out of 128 GB of regular DDR5 memory as video memory. So, it's not accurate to say that this test is purely on the CPU. But still, the results are pretty good. Especially when compared to my P40, which shows only half the speed.

> 👤 **ikawrakow** replied on **2025-10-25** at **13:58:43**
> 
> I know what the AI MAX+395 is. But given that for GPT-OSS-120B I'm getting PP = ~330 t/s running CPU-only on a fairly old Ryzen-5975WX, and given that the Ryzen-5975WX (Zen3) tends to be slower than my Ryzen-7950X (Zen4, but I cannot test the 120B model on that one as it does not have enough RAM), and finally given that the AI MAX+395 is Zen5, I gestimated that these results must be CPU-only. The TG performance is also consistent with that hypothesis, given the AI MAX memory bandwidth. But I wanted to make sure, so that's why the question.
> 
> To use the integrated GPU, one must make it work with the Vulkan back-end, which may or may not work at this point (haven't tested on Vulkan lately).

> 👤 **Djip007** replied on **2025-10-26** at **21:24:46**
> 
> Yes it is on CPU only.
> 
> I don't know if I can build ik_llama.cpp with GPU support for this gfx1151 chip.
> 
> with llama.cpp on rocm 7.10a, GPU we have much higher result.
> 
> Is it possible to build ik_llama.cpp with rocm/hip?

> 👤 **ikawrakow** replied on **2025-10-27** at **13:07:53**
> 
> > Is it possible to build ik_llama.cpp with rocm/hip?
> 
> Most likely not. My guess is that Vulkan would be a better option. But I have added some new fused ops lately which are not implemented in the Vulkan back-end, so my guess is that it will not be optimal (too many ops running on the CPU, too many graph splits).
> 
> 475 t/s for PP is not a bad result for CPU-only.
> 
> The 29 t/s TG is somewhat disappointing. It basically means that the CPU does not get all of the available bandwidth.

> 👤 **Djip007** replied on **2025-10-27** at **19:35:25**
> 
> > The 29 t/s TG is somewhat disappointing. It basically means that the CPU does not get all of the available bandwidth.
> 
> Yes... it get only ~1/2 of the bandwidth with GPU we get 45/50 . But with a DDR@8000 it is faster than the "classique" 6400 we can have with "classic" zen5 (AMD Ryzen 9 9950).
> 
> So yes with ryzen 7940HS for example, we can only program mulmat op on the GPU and keep all other OP on CPU with good perfo.
> 
> For comparer we get on GPU with llama.cpp / rocm 7.10:
> 
> | model                          |       size |     params | backend    | ngl | n_ubatch | fa | mmap |            test |                  t/s |
> | ------------------------------ | ---------: | ---------: | ---------- | --: | -------: | -: | ---: | --------------: | -------------------: |
> | gpt-oss 120B MXFP4 MoE         |  59.02 GiB |   116.83 B | ROCm       | 999 |     2048 |  1 |    0 |             pp1 |         45.20 ± 0.32 |
> | gpt-oss 120B MXFP4 MoE         |  59.02 GiB |   116.83 B | ROCm       | 999 |     2048 |  1 |    0 |             pp1 |         45.40 ± 0.01 |
> | gpt-oss 120B MXFP4 MoE         |  59.02 GiB |   116.83 B | ROCm       | 999 |     2048 |  1 |    0 |             pp2 |         57.58 ± 0.95 |
> | gpt-oss 120B MXFP4 MoE         |  59.02 GiB |   116.83 B | ROCm       | 999 |     2048 |  1 |    0 |             pp3 |         74.03 ± 2.34 |
> | gpt-oss 120B MXFP4 MoE         |  59.02 GiB |   116.83 B | ROCm       | 999 |     2048 |  1 |    0 |             pp4 |         90.93 ± 2.95 |
> | gpt-oss 120B MXFP4 MoE         |  59.02 GiB |   116.83 B | ROCm       | 999 |     2048 |  1 |    0 |             pp8 |        142.31 ± 5.57 |
> | gpt-oss 120B MXFP4 MoE         |  59.02 GiB |   116.83 B | ROCm       | 999 |     2048 |  1 |    0 |            pp12 |       173.14 ± 12.88 |
> | gpt-oss 120B MXFP4 MoE         |  59.02 GiB |   116.83 B | ROCm       | 999 |     2048 |  1 |    0 |            pp16 |        205.43 ± 6.72 |
> | gpt-oss 120B MXFP4 MoE         |  59.02 GiB |   116.83 B | ROCm       | 999 |     2048 |  1 |    0 |            pp24 |       235.43 ± 11.38 |
> | gpt-oss 120B MXFP4 MoE         |  59.02 GiB |   116.83 B | ROCm       | 999 |     2048 |  1 |    0 |            pp32 |       234.24 ± 10.83 |
> | gpt-oss 120B MXFP4 MoE         |  59.02 GiB |   116.83 B | ROCm       | 999 |     2048 |  1 |    0 |            pp48 |       216.49 ± 10.21 |
> | gpt-oss 120B MXFP4 MoE         |  59.02 GiB |   116.83 B | ROCm       | 999 |     2048 |  1 |    0 |            pp64 |        311.52 ± 7.33 |
> | gpt-oss 120B MXFP4 MoE         |  59.02 GiB |   116.83 B | ROCm       | 999 |     2048 |  1 |    0 |            pp96 |       386.08 ± 10.33 |
> | gpt-oss 120B MXFP4 MoE         |  59.02 GiB |   116.83 B | ROCm       | 999 |     2048 |  1 |    0 |           pp128 |        446.85 ± 6.77 |
> | gpt-oss 120B MXFP4 MoE         |  59.02 GiB |   116.83 B | ROCm       | 999 |     2048 |  1 |    0 |           pp192 |        509.42 ± 8.09 |
> | gpt-oss 120B MXFP4 MoE         |  59.02 GiB |   116.83 B | ROCm       | 999 |     2048 |  1 |    0 |           pp256 |        594.22 ± 9.46 |
> | gpt-oss 120B MXFP4 MoE         |  59.02 GiB |   116.83 B | ROCm       | 999 |     2048 |  1 |    0 |           pp384 |        698.31 ± 3.26 |
> | gpt-oss 120B MXFP4 MoE         |  59.02 GiB |   116.83 B | ROCm       | 999 |     2048 |  1 |    0 |           pp512 |        763.53 ± 4.88 |
> | gpt-oss 120B MXFP4 MoE         |  59.02 GiB |   116.83 B | ROCm       | 999 |     2048 |  1 |    0 |           pp768 |        845.23 ± 6.57 |
> | gpt-oss 120B MXFP4 MoE         |  59.02 GiB |   116.83 B | ROCm       | 999 |     2048 |  1 |    0 |          pp1024 |        927.17 ± 1.20 |
> | gpt-oss 120B MXFP4 MoE         |  59.02 GiB |   116.83 B | ROCm       | 999 |     2048 |  1 |    0 |          pp1536 |        987.73 ± 1.96 |
> | gpt-oss 120B MXFP4 MoE         |  59.02 GiB |   116.83 B | ROCm       | 999 |     2048 |  1 |    0 |          pp2048 |       1017.17 ± 4.10 |
> | gpt-oss 120B MXFP4 MoE         |  59.02 GiB |   116.83 B | ROCm       | 999 |     2048 |  1 |    0 |          pp3072 |        939.48 ± 2.72 |
> | gpt-oss 120B MXFP4 MoE         |  59.02 GiB |   116.83 B | ROCm       | 999 |     2048 |  1 |    0 |          pp4096 |        953.72 ± 1.16 |
> | gpt-oss 120B MXFP4 MoE         |  59.02 GiB |   116.83 B | ROCm       | 999 |     2048 |  1 |    0 |            tg16 |         45.43 ± 0.01 |
> | gpt-oss 120B MXFP4 MoE         |  59.02 GiB |   116.83 B | ROCm       | 999 |     2048 |  1 |    0 |      pp512+tg64 |        264.68 ± 0.82 |

> 👤 **Djip007** replied on **2025-10-27** at **19:44:33**
> 
> > > Is it possible to build ik_llama.cpp with rocm/hip?
> > 
> > Most likely not.
> 
> I have a try ... but I need rocm7 so there is many element to patch. some are simple, other more complicated. I may try when I get more time, but I may need some help to make the right choices.