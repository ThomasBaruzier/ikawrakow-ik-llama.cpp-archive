## 🔀 [Pull Request #892](https://github.com/ikawrakow/ik_llama.cpp/pull/892) - Merge Q and K into a single tensor

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/merge_only_qk` |
| **Target Branch** | `main` |
| **Created** | 2025-11-03 |
| **Updated** | 2025-11-05 |
| **Merged** | 2025-11-05 |

---

## 📄 Description

This PR is a follow up of [#878](https://github.com/ikawrakow/ik_llama.cpp/issues/878). When the user requests to merge Q, K, V into a single tensor via the `-mqkv` (or `--merge-qkv`) command line argument, but we find that V is of a different quantization type than Q, K, we now merge Q and K, and leave V in a separate tensor. This is actually quite a common situation for quantized models as V has a bigger impact on model quality than Q and K, so is often quantized with more bits. The ability to do so was requested by @Nexesenex ( https://github.com/ikawrakow/ik_llama.cpp/pull/878#issuecomment-3462860933), but I didn't go as far as also merging K and V when they are of the same type but Q is not.

Here is a quick `llama-sweep-bench` example with Qwen3-30B-A3B quantized with the default `IQ2_XXS` quantization scheme. In this case Q and K are `IQ2_XXS`, while V is `Q4_K`. GPU is RTX 4080

### This PR, Q and K merged

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  2048 |    512 |      0 |    0.273 |  7492.72 |    1.961 |   261.09 |
|  2048 |    512 |   2048 |    0.297 |  6901.43 |    2.101 |   243.64 |
|  2048 |    512 |   4096 |    0.326 |  6287.71 |    2.287 |   223.88 |
|  2048 |    512 |   6144 |    0.354 |  5779.04 |    2.479 |   206.54 |
|  2048 |    512 |   8192 |    0.384 |  5330.31 |    2.616 |   195.69 |
|  2048 |    512 |  10240 |    0.415 |  4938.68 |    2.811 |   182.14 |
|  2048 |    512 |  12288 |    0.446 |  4592.94 |    2.938 |   174.26 |
|  2048 |    512 |  14336 |    0.473 |  4326.46 |    3.120 |   164.09 |
|  2048 |    512 |  16384 |    0.504 |  4065.14 |    3.258 |   157.15 |
|  2048 |    512 |  18432 |    0.534 |  3836.81 |    3.402 |   150.48 |
|  2048 |    512 |  20480 |    0.563 |  3638.20 |    3.595 |   142.40 |
|  2048 |    512 |  22528 |    0.595 |  3442.81 |    3.734 |   137.12 |
|  2048 |    512 |  24576 |    0.624 |  3282.38 |    3.953 |   129.51 |
|  2048 |    512 |  26624 |    0.653 |  3138.20 |    4.071 |   125.78 |
|  2048 |    512 |  28672 |    0.684 |  2995.29 |    4.242 |   120.69 |
|  2048 |    512 |  30720 |    0.715 |  2866.01 |    4.444 |   115.22 |

### Main branch

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  2048 |    512 |      0 |    0.274 |  7468.21 |    2.041 |   250.83 |
|  2048 |    512 |   2048 |    0.297 |  6898.18 |    2.159 |   237.16 |
|  2048 |    512 |   4096 |    0.329 |  6217.01 |    2.360 |   216.92 |
|  2048 |    512 |   6144 |    0.356 |  5760.62 |    2.546 |   201.12 |
|  2048 |    512 |   8192 |    0.386 |  5311.62 |    2.701 |   189.59 |
|  2048 |    512 |  10240 |    0.417 |  4915.22 |    2.877 |   177.97 |
|  2048 |    512 |  12288 |    0.446 |  4595.34 |    3.002 |   170.53 |
|  2048 |    512 |  14336 |    0.477 |  4297.70 |    3.185 |   160.76 |
|  2048 |    512 |  16384 |    0.508 |  4033.74 |    3.351 |   152.80 |
|  2048 |    512 |  18432 |    0.546 |  3749.79 |    3.483 |   147.02 |
|  2048 |    512 |  20480 |    0.566 |  3618.20 |    3.660 |   139.87 |
|  2048 |    512 |  22528 |    0.609 |  3365.60 |    3.822 |   133.96 |
|  2048 |    512 |  24576 |    0.624 |  3284.31 |    4.013 |   127.58 |
|  2048 |    512 |  26624 |    0.656 |  3124.20 |    4.122 |   124.22 |
|  2048 |    512 |  28672 |    0.689 |  2972.54 |    4.340 |   117.96 |
|  2048 |    512 |  30720 |    0.718 |  2853.20 |    4.503 |   113.69 |

The PR does not look into op fusion (type and order of operations changes with Q and K merged, but a separate V tensor). Nevertheless, we see around 4% TG performance improvement for short context.

---

## 💬 Conversation

👤 **ikawrakow** commented on **2025-11-03** at **17:18:49**

I'm running into CUDA errors, so converting to draft for now.

---

👤 **ikawrakow** commented on **2025-11-04** at **08:43:16**

OK, the issues I was seeing were due to the fused `FUSED_RMS_NORM + FUSED_RMS_NORM + FAST_ROPE + FAST_ROPE`. This fusion is now disabled until I figure out what is wrong with it. Hence, this PR should be OK now to be tested.