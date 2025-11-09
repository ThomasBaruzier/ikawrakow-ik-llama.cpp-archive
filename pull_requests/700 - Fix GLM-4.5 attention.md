## 🔀 [Pull Request #700](https://github.com/ikawrakow/ik_llama.cpp/pull/700) - Fix GLM-4.5 attention

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/fix_glm4_attn` |
| **Target Branch** | `main` |
| **Created** | 2025-08-17 |
| **Updated** | 2025-08-17 |
| **Merged** | 2025-08-17 |

---

## 📄 Description

There have been reports of `llama.cpp` being faster than `ik_llama.cpp` for the GLM-4.5 MoE models, see e.g. [here](https://github.com/ikawrakow/ik_llama.cpp/pull/689#issuecomment-3191092917) or [here](https://huggingface.co/ubergarm/GLM-4.5-Air-GGUF/discussions/2#68980a066bc7e6a3fcef131c), with the `llama.cpp` advantage increasing with context length, which points to a problem with the self-attention implementation in `ik_llama.cpp` for this specific model.

After wasting a lot of time scrutinizing the various kernels involved, I finally located the problem in [this  function](https://github.com/ikawrakow/ik_llama.cpp/blob/4bf5c8184b177ee5bbd77fc7fde34f57613c1f6b/ggml/src/ggml-cuda/fattn.cu#L72), where we have:
```c++
    if (use_gqa_opt && gqa_ratio % 8 == 0) {
        ggml_cuda_flash_attn_ext_mma_f16_switch_hs<8>(ctx, dst);
        return;
    }   

    if (use_gqa_opt && gqa_ratio == 4) {
        ggml_cuda_flash_attn_ext_mma_f16_switch_hs<4>(ctx, dst);
        return;
    }   

    if (use_gqa_opt && gqa_ratio == 2) {
        ggml_cuda_flash_attn_ext_mma_f16_switch_hs<2>(ctx, dst);
        return;
    }   
```
 
Hahaha. GLM-4.5 has a GQA ratio of 12, hence none of the conditions for GQA-related optimizations takes effect, so the slow path is taken, resulting in a poor performance for this, or any other model that does not have GQA of 2, 4, or 8.

This PR fixes it. I'm RAM (64 GB) and VRAM (16 GB) poor, so testing using Unsloth's `GLM-4.5-Air-UD-IQ2_XXS.gguf` with all experts left on the CPU (but even then, I can only go up to a context of 32k tokens). As attention is computed on the GPU where we have the problem at hand, this should still be representative for what VRAM-rich people can expect with full GPU offload after this PR. CPU is Ryzen-7950X, GPU is RTX-4080. OS is Linux.

### TG performance

<img width="792" height="612" alt="glm_tg" src="https://github.com/user-attachments/assets/6474dbdd-c8c5-4f1b-b770-2385bfd5c864" />

### PP performance

<img width="792" height="612" alt="glm_pp" src="https://github.com/user-attachments/assets/17c9cd3f-3f4b-49f8-8578-b4fe4ea25b95" />

<details>
<summary>ik_llama.cpp, main</summary>

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  4096 |   1024 |      0 |    3.914 |  1046.63 |   61.736 |    16.59 |
|  4096 |   1024 |   4096 |    4.272 |   958.91 |   67.199 |    15.24 |
|  4096 |   1024 |   8192 |    4.724 |   867.11 |   72.536 |    14.12 |
|  4096 |   1024 |  12288 |    5.174 |   791.59 |   77.991 |    13.13 |
|  4096 |   1024 |  16384 |    5.629 |   727.63 |   83.358 |    12.28 |
|  4096 |   1024 |  20480 |    6.107 |   670.75 |   93.370 |    10.97 |
|  4096 |   1024 |  24576 |    6.622 |   618.51 |  108.825 |     9.41 |
|  4096 |   1024 |  28672 |    7.067 |   579.62 |  122.549 |     8.36 |
</details>

<details>
<summary>ik_llama.cpp PR</summary>

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  4096 |   1024 |      0 |    3.770 |  1086.43 |   60.162 |    17.02 |
|  4096 |   1024 |   4096 |    4.102 |   998.45 |   62.571 |    16.37 |
|  4096 |   1024 |   8192 |    4.458 |   918.74 |   64.041 |    15.99 |
|  4096 |   1024 |  12288 |    4.853 |   844.04 |   66.053 |    15.50 |
|  4096 |   1024 |  16384 |    5.254 |   779.60 |   68.412 |    14.97 |
|  4096 |   1024 |  20480 |    5.684 |   720.62 |   76.075 |    13.46 |
|  4096 |   1024 |  24576 |    6.156 |   665.36 |   81.268 |    12.60 |
|  4096 |   1024 |  28672 |    6.592 |   621.37 |   85.233 |    12.01 |
</details>

<details>
<summary>llama.cpp, build: 6181 (de2192794)</summary>

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  4096 |   1024 |      0 |    4.350 |   941.71 |   65.528 |    15.63 |
|  4096 |   1024 |   4096 |    4.757 |   861.03 |   70.442 |    14.54 |
|  4096 |   1024 |   8192 |    5.103 |   802.67 |   69.448 |    14.74 |
|  4096 |   1024 |  12288 |    5.382 |   761.11 |   71.500 |    14.32 |
|  4096 |   1024 |  16384 |    5.935 |   690.15 |   73.830 |    13.87 |
|  4096 |   1024 |  20480 |    6.470 |   633.05 |   81.029 |    12.64 |
|  4096 |   1024 |  24576 |    6.626 |   618.18 |   86.152 |    11.89 |
|  4096 |   1024 |  28672 |    7.145 |   573.27 |   90.104 |    11.36 |
</details>

@Thireus It would be great if you could repeat you GLM-4.5-Air sweep bench with this PR. Thanks in advance!

---

## 💬 Conversation

👤 **Thireus** commented on **2025-08-17** at **07:08:37**

Thank you! On it!

---

👤 **Thireus** commented on **2025-08-17** at **11:24:12**

I can confirm this pull request fixes it! Thank you so much.

<img width="1779" height="1181" alt="GLM-4 5-Air_spp" src="https://github.com/user-attachments/assets/e819ac1b-81ef-4839-9799-afad6dceaba9" />
<img width="1758" height="1093" alt="GLM-4 5-Air_stg" src="https://github.com/user-attachments/assets/c52e9fc4-1af9-4ae7-8927-6962facd9bdd" />

Any idea what else could be causing the small speed degradation observed when compared to llama.cpp?

---

👤 **ikawrakow** commented on **2025-08-17** at **11:30:13**

Thank you!

> Any idea what else could be causing the small speed degradation observed when compared to llama.cpp?

That appears to be OS specific. It is faster for me and also for @ubergarm on Linux. Or maybe it is Blackwell? Not sure, hard to figure out with my computing resources.

---

👤 **Ph0rk0z** commented on **2025-08-17** at **13:10:05**

Improved hybrid inference too. As context builds TG no longer drops off so fast for big GLM.

---

👤 **ubergarm** commented on **2025-08-17** at **14:48:18**

Confirmed on GLM-4.5-Air using 2x RTX 6000 non-pro non-blackwell GPUs sm86 arch. This GQA Ratio 12 fast path fix improves TG *a lot* as context grows and is even faster than just the CUDA Graphs at lower context length. 

ik_llama.cpp is by far the fastest TG in this specific benchmark configuration.

<img width="3154" height="1141" alt="sweep-bench-GLM-4 5-Air-Q4_0-CUDA-graphs-plus-GQA-fix" src="https://github.com/user-attachments/assets/91c3e93e-6f76-414c-9618-1dca56e3b035" />

<details>

<summary>👈 Details</summary>

```bash
./build/bin/llama-sweep-bench \
    --model "$model"\
    -fa -fmoe \
    -c 20480 \
    -ngl 99 \
    --threads 1 \
    -ub 4096 -b 4096 \
    --warmup-batch
```

## ik_llama.cpp@a3a52300 CUDA ubergarm/GLM-4.5-Air-Q4_0 -fmoe -ub 4096 -b 4096 (GQA Fix + CUDA Graphs)
|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  4096 |   1024 |      0 |    2.442 |  1676.99 |   20.339 |    50.35 |
|  4096 |   1024 |   4096 |    2.986 |  1371.74 |   21.896 |    46.77 |
|  4096 |   1024 |   8192 |    3.599 |  1138.09 |   23.563 |    43.46 |
|  4096 |   1024 |  12288 |    4.232 |   967.86 |   25.307 |    40.46 |
|  4096 |   1024 |  16384 |    4.810 |   851.58 |   26.923 |    38.03 |

## ik_llama.cpp@4bf5c8184 CUDA ubergarm/GLM-4.5-Air-Q4_0 -fmoe -ub 4096 -b 4096 (CUDA Graphs)
|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  4096 |   1024 |      0 |    2.556 |  1602.58 |   20.998 |    48.77 |
|  4096 |   1024 |   4096 |    3.232 |  1267.38 |   27.346 |    37.45 |
|  4096 |   1024 |   8192 |    3.927 |  1043.00 |   33.793 |    30.30 |
|  4096 |   1024 |  12288 |    4.579 |   894.53 |   40.446 |    25.32 |
|  4096 |   1024 |  16384 |    5.230 |   783.18 |   47.190 |    21.70 |

## ik_llama.cpp@d60c8f4d CUDA ubergarm/GLM-4.5-Air-Q4_0 -fmoe -ub 4096 -b 4096 (NO CUDA Graphs)
|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  4096 |   1024 |      0 |    2.563 |  1598.15 |   23.018 |    44.49 |
|  4096 |   1024 |   4096 |    3.226 |  1269.81 |   30.892 |    33.15 |
|  4096 |   1024 |   8192 |    3.946 |  1038.11 |   39.202 |    26.12 |
|  4096 |   1024 |  12288 |    4.581 |   894.11 |   47.717 |    21.46 |
|  4096 |   1024 |  16384 |    5.252 |   779.93 |   56.136 |    18.24 |

## ik_llama.cpp@d60c8f4d Vulkan ubergarm/GLM-4.5-Air-Q4_0 ggml_vulkan: 0 = NVIDIA RTX A6000 (NVIDIA) | uma: 0 | fp16: 1 | warp size: 32 | shared memory: 49152 | int dot: 1 | matrix cores: KHR_coopmat
|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|   512 |    128 |      0 |    0.861 |   594.40 |    3.052 |    41.94 |
|   512 |    128 |    512 |    0.904 |   566.46 |    3.314 |    38.62 |
|   512 |    128 |   1024 |    0.957 |   534.73 |    3.576 |    35.79 |
|   512 |    128 |   1536 |    1.014 |   504.83 |    3.836 |    33.37 |
|   512 |    128 |   2048 |    1.070 |   478.29 |    4.094 |    31.27 |
|   512 |    128 |   2560 |    1.135 |   451.12 |    4.365 |    29.32 |
|   512 |    128 |   3072 |    1.193 |   429.01 |    4.619 |    27.71 |
|   512 |    128 |   3584 |    1.257 |   407.43 |    4.880 |    26.23 |
|   512 |    128 |   4096 |    1.309 |   391.20 |    5.158 |    24.81 |
|   512 |    128 |   4608 |    1.370 |   373.80 |    5.422 |    23.61 |
|   512 |    128 |   5120 |    1.427 |   358.91 |    5.680 |    22.54 |
|   512 |    128 |   5632 |    1.484 |   344.99 |    5.917 |    21.63 |
|   512 |    128 |   6144 |    1.545 |   331.36 |    6.206 |    20.63 |
|   512 |    128 |   6656 |    1.600 |   319.97 |    6.470 |    19.78 |
|   512 |    128 |   7168 |    1.657 |   308.95 |    6.729 |    19.02 |
|   512 |    128 |   7680 |    1.714 |   298.67 |    6.992 |    18.31 |
|   512 |    128 |   8192 |    1.783 |   287.23 |    7.252 |    17.65 |
|   512 |    128 |   8704 |    1.833 |   279.28 |    7.511 |    17.04 |
|   512 |    128 |   9216 |    1.885 |   271.64 |    7.777 |    16.46 |
|   512 |    128 |   9728 |    1.945 |   263.21 |    8.037 |    15.93 |
|   512 |    128 |  10240 |    2.002 |   255.70 |    8.301 |    15.42 |
|   512 |    128 |  10752 |    2.055 |   249.15 |    8.570 |    14.94 |
|   512 |    128 |  11264 |    2.113 |   242.35 |    8.825 |    14.50 |
|   512 |    128 |  11776 |    2.167 |   236.27 |    9.090 |    14.08 |
|   512 |    128 |  12288 |    2.215 |   231.11 |    9.329 |    13.72 |
|   512 |    128 |  12800 |    2.275 |   225.05 |    9.607 |    13.32 |
|   512 |    128 |  13312 |    2.331 |   219.65 |    9.872 |    12.97 |
|   512 |    128 |  13824 |    2.387 |   214.53 |   10.129 |    12.64 |
|   512 |    128 |  14336 |    2.445 |   209.43 |   10.386 |    12.32 |
|   512 |    128 |  14848 |    2.495 |   205.22 |   10.654 |    12.01 |
|   512 |    128 |  15360 |    2.552 |   200.67 |   10.907 |    11.74 |
|   512 |    128 |  15872 |    2.605 |   196.56 |   11.172 |    11.46 |
|   512 |    128 |  16384 |    2.660 |   192.45 |   11.443 |    11.19 |

## llama.cpp@79c1160b0 CUDA ubergarm/GLM-4.5-Air-Q4_0 -ub 4096 -b 4096
|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  4096 |   1024 |      0 |    3.083 |  1328.69 |   21.168 |    48.37 |
|  4096 |   1024 |   4096 |    3.628 |  1128.86 |   26.655 |    38.42 |
|  4096 |   1024 |   8192 |    4.267 |   960.02 |   32.327 |    31.68 |
|  4096 |   1024 |  12288 |    4.885 |   838.46 |   38.094 |    26.88 |
|  4096 |   1024 |  16384 |    5.433 |   753.97 |   43.756 |    23.40 |

## llama.cpp@79c1160b0 Vulkan ubergarm/GLM-4.5-Air-Q4_0
|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|   512 |    128 |      0 |    0.849 |   603.27 |    3.084 |    41.51 |
|   512 |    128 |    512 |    0.889 |   576.03 |    3.347 |    38.24 |
|   512 |    128 |   1024 |    0.941 |   544.37 |    3.581 |    35.74 |
|   512 |    128 |   1536 |    0.996 |   513.91 |    3.855 |    33.20 |
|   512 |    128 |   2048 |    1.050 |   487.71 |    4.115 |    31.10 |
|   512 |    128 |   2560 |    1.116 |   458.97 |    4.377 |    29.24 |
|   512 |    128 |   3072 |    1.175 |   435.73 |    4.648 |    27.54 |
|   512 |    128 |   3584 |    1.235 |   414.69 |    4.905 |    26.10 |
|   512 |    128 |   4096 |    1.284 |   398.88 |    5.159 |    24.81 |
|   512 |    128 |   4608 |    1.348 |   379.76 |    5.429 |    23.58 |
|   512 |    128 |   5120 |    1.406 |   364.15 |    5.692 |    22.49 |
|   512 |    128 |   5632 |    1.464 |   349.72 |    5.965 |    21.46 |
|   512 |    128 |   6144 |    1.525 |   335.73 |    6.223 |    20.57 |
|   512 |    128 |   6656 |    1.586 |   322.81 |    6.496 |    19.70 |
|   512 |    128 |   7168 |    1.641 |   312.02 |    6.759 |    18.94 |
|   512 |    128 |   7680 |    1.699 |   301.31 |    7.020 |    18.23 |
|   512 |    128 |   8192 |    1.764 |   290.33 |    7.278 |    17.59 |
|   512 |    128 |   8704 |    1.822 |   281.01 |    7.540 |    16.98 |
|   512 |    128 |   9216 |    1.871 |   273.68 |    7.797 |    16.42 |
|   512 |    128 |   9728 |    1.934 |   264.77 |    8.064 |    15.87 |
|   512 |    128 |  10240 |    1.991 |   257.16 |    8.324 |    15.38 |
|   512 |    128 |  10752 |    2.038 |   251.19 |    8.591 |    14.90 |
|   512 |    128 |  11264 |    2.100 |   243.78 |    8.850 |    14.46 |
|   512 |    128 |  11776 |    2.157 |   237.40 |    9.123 |    14.03 |
|   512 |    128 |  12288 |    2.212 |   231.46 |    9.375 |    13.65 |
|   512 |    128 |  12800 |    2.263 |   226.28 |    9.653 |    13.26 |
|   512 |    128 |  13312 |    2.323 |   220.42 |    9.911 |    12.91 |
|   512 |    128 |  13824 |    2.379 |   215.22 |   10.168 |    12.59 |
|   512 |    128 |  14336 |    2.433 |   210.47 |   10.430 |    12.27 |
|   512 |    128 |  14848 |    2.482 |   206.32 |   10.691 |    11.97 |
|   512 |    128 |  15360 |    2.538 |   201.72 |   10.956 |    11.68 |
|   512 |    128 |  15872 |    2.592 |   197.52 |   11.215 |    11.41 |
|   512 |    128 |  16384 |    2.645 |   193.56 |   11.475 |    11.16 |


</details>

---

👤 **ubergarm** commented on **2025-08-17** at **15:52:33**

@Thireus 

> Any idea what else could be causing the small speed degradation observed when compared to llama.cpp?

I am not sure exactly the full command you are using for your llama-sweep-bench tests, but looking above at some of your llama-server commands, some observations:

* it is odd to me you're only specifying `-ctk f16`. it isn't hurting anything, just that is the default value. For a GQA model you would want to specify *both* `-ctk f16 -ctv f16` and for an MLA model (deepseek/kimi-k2) you would specify only the single `-ctk f16` for example. again, not an issue and no effect on speed here, just an observation.
* `--threads 36` for fully offloading GPU will likely give ~5% worse performance on ik due to overhead with those CPU threads when they are not even being used, so on ik I always use `--threads 1` when fully offloading to GPU for best speed.
* `--no-mmap` is not needed for full GPU offload as it shouldn't be malloc'ing any system RAM for weights anyway
* add `--warmup-batch` on ik's fork for llama-sweep-bench, not needed on my mainline port as it is hardcoded enabled.

As such, give this slightly massaged command a try (replacing with the latest tip of main executable)
```bash
CUDA_DEVICE_ORDER=PCI_BUS_ID CUDA_VISIBLE_DEVICES=1 \
    ~/ik_llama-ik-try_cuda_graphs-b4107-7693263-bin-win-cuda-12.8-x64-avx512/llama-sweep-bench \
    -m GLM-4.5-Air-THIREUS-BF16-SPECIAL_TENSOR-00001-of-00804.gguf \
    -fa -fmoe \
    -c 135168 \
    -ngl 99 \
    -b 4096 -ub 4096 \
    --threads 1 \
    --main-gpu 0 \
    --warmup-batch
```
Then for mainline just remove `-fmoe` and `--warmup-batch`.


I'll post again in a bit as I'm doing longer context sweeps out to ~100k and using `-ctk q8_0 -ctv q8_0` does slow down TG at larger kv-cache depths on the dual old 6000 rig.

---

👤 **ubergarm** commented on **2025-08-17** at **17:20:23**

Okay, for this specific config/quant here is what I'm seeing:

* ik has faster PP than mainline which is fairly common with larger batch sizes
* ik and mainline have very similar TG performance for unquantized f16 kv-cache
* both ik and mainline TG slow down when using q8_0 kv-cache, but ik is doing better here

<img width="2081" height="1111" alt="sweep-bench-GLM-4 5-Air-Q4_0-CUDA-graphs-plus-GQA-fix-long-context" src="https://github.com/user-attachments/assets/cda4cfbe-5dad-471c-9c99-d7598ce7f64f" />

(in my previous graph in earlier comment, i probably had been running q8_0 despite not showing it in the table, so this chart is up to date most reflective now at this point in time)

<details>

<summary>👈 Details</summary>

```bash
./build/bin/llama-sweep-bench \
    --model "$model"\
    -fa -fmoe \
    -ctk q8_0 -ctv q8_0 \
    -c 102400 \
    -ngl 99 \
    --threads 1 \
    -ub 4096 -b 4096 \
    --warmup-batch
```

## ik_llama.cpp@a3a52300 CUDA ubergarm/GLM-4.5-Air-Q4_0
|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  4096 |   1024 |      0 |    2.486 |  1647.52 |   20.378 |    50.25 |
|  4096 |   1024 |   4096 |    3.038 |  1348.16 |   22.020 |    46.50 |
|  4096 |   1024 |   8192 |    3.662 |  1118.53 |   23.668 |    43.26 |
|  4096 |   1024 |  12288 |    4.275 |   958.14 |   25.410 |    40.30 |
|  4096 |   1024 |  16384 |    4.833 |   847.54 |   27.055 |    37.85 |
|  4096 |   1024 |  20480 |    5.393 |   759.53 |   28.536 |    35.88 |
|  4096 |   1024 |  24576 |    5.961 |   687.16 |   30.227 |    33.88 |
|  4096 |   1024 |  28672 |    5.828 |   702.86 |   31.975 |    32.03 |
|  4096 |   1024 |  32768 |    6.320 |   648.09 |   33.478 |    30.59 |
|  4096 |   1024 |  36864 |    6.831 |   599.60 |   35.015 |    29.24 |
|  4096 |   1024 |  40960 |    7.281 |   562.58 |   36.814 |    27.82 |
|  4096 |   1024 |  45056 |    7.758 |   527.96 |   38.411 |    26.66 |
|  4096 |   1024 |  49152 |    8.224 |   498.03 |   40.029 |    25.58 |
|  4096 |   1024 |  53248 |    8.726 |   469.42 |   41.639 |    24.59 |
|  4096 |   1024 |  57344 |    9.217 |   444.39 |   43.402 |    23.59 |
|  4096 |   1024 |  61440 |    9.726 |   421.13 |   44.935 |    22.79 |
|  4096 |   1024 |  65536 |   10.273 |   398.72 |   46.597 |    21.98 |
|  4096 |   1024 |  69632 |   10.731 |   381.70 |   48.222 |    21.24 |
|  4096 |   1024 |  73728 |   11.243 |   364.30 |   49.929 |    20.51 |
|  4096 |   1024 |  77824 |   11.726 |   349.32 |   51.489 |    19.89 |
|  4096 |   1024 |  81920 |   12.241 |   334.61 |   53.122 |    19.28 |
|  4096 |   1024 |  86016 |   12.636 |   324.15 |   54.776 |    18.69 |
|  4096 |   1024 |  90112 |   13.177 |   310.84 |   56.407 |    18.15 |
|  4096 |   1024 |  94208 |   13.651 |   300.05 |   58.048 |    17.64 |
|  4096 |   1024 |  98304 |   14.083 |   290.85 |   59.719 |    17.15 |

## ik_llama.cpp@a3a52300 CUDA ubergarm/GLM-4.5-Air-Q4_0 -ctk q8_0 -ctv q8_0
|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  4096 |   1024 |      0 |    2.497 |  1640.16 |   20.794 |    49.25 |
|  4096 |   1024 |   4096 |    3.047 |  1344.31 |   24.001 |    42.66 |
|  4096 |   1024 |   8192 |    3.670 |  1116.14 |   27.464 |    37.29 |
|  4096 |   1024 |  12288 |    4.287 |   955.37 |   31.006 |    33.03 |
|  4096 |   1024 |  16384 |    4.862 |   842.42 |   34.393 |    29.77 |
|  4096 |   1024 |  20480 |    5.433 |   753.94 |   37.675 |    27.18 |
|  4096 |   1024 |  24576 |    5.962 |   686.99 |   41.175 |    24.87 |
|  4096 |   1024 |  28672 |    6.312 |   648.89 |   44.665 |    22.93 |
|  4096 |   1024 |  32768 |    6.611 |   619.57 |   47.954 |    21.35 |
|  4096 |   1024 |  36864 |    7.399 |   553.62 |   51.301 |    19.96 |
|  4096 |   1024 |  40960 |    7.721 |   530.51 |   54.774 |    18.70 |
|  4096 |   1024 |  45056 |    8.235 |   497.40 |   58.187 |    17.60 |
|  4096 |   1024 |  49152 |    8.751 |   468.04 |   61.485 |    16.65 |
|  4096 |   1024 |  53248 |    9.280 |   441.40 |   64.954 |    15.77 |
|  4096 |   1024 |  57344 |    9.912 |   413.22 |   68.461 |    14.96 |
|  4096 |   1024 |  61440 |   10.354 |   395.58 |   71.778 |    14.27 |
|  4096 |   1024 |  65536 |   11.023 |   371.60 |   75.162 |    13.62 |
|  4096 |   1024 |  69632 |   11.436 |   358.18 |   78.641 |    13.02 |
|  4096 |   1024 |  73728 |   12.061 |   339.61 |   82.082 |    12.48 |
|  4096 |   1024 |  77824 |   12.581 |   325.58 |   85.468 |    11.98 |
|  4096 |   1024 |  81920 |   13.238 |   309.42 |   88.805 |    11.53 |
|  4096 |   1024 |  86016 |   13.532 |   302.70 |   92.380 |    11.08 |
|  4096 |   1024 |  90112 |   14.104 |   290.41 |   95.752 |    10.69 |
|  4096 |   1024 |  94208 |   14.442 |   283.63 |   99.241 |    10.32 |
|  4096 |   1024 |  98304 |   15.134 |   270.66 |  102.738 |     9.97 |

## llama.cpp@19f4dec CUDA ubergarm/GLM-4.5-Air-Q4_0
|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  4096 |   1024 |      0 |    3.036 |  1349.21 |   20.641 |    49.61 |
|  4096 |   1024 |   4096 |    3.576 |  1145.50 |   22.286 |    45.95 |
|  4096 |   1024 |   8192 |    4.138 |   989.85 |   23.930 |    42.79 |
|  4096 |   1024 |  12288 |    4.799 |   853.56 |   25.644 |    39.93 |
|  4096 |   1024 |  16384 |    5.377 |   761.76 |   27.243 |    37.59 |
|  4096 |   1024 |  20480 |    5.936 |   690.08 |   28.729 |    35.64 |
|  4096 |   1024 |  24576 |    6.568 |   623.62 |   30.457 |    33.62 |
|  4096 |   1024 |  28672 |    8.402 |   487.53 |   32.267 |    31.74 |
|  4096 |   1024 |  32768 |    8.787 |   466.16 |   33.621 |    30.46 |
|  4096 |   1024 |  36864 |    9.254 |   442.63 |   35.204 |    29.09 |
|  4096 |   1024 |  40960 |    9.727 |   421.08 |   36.917 |    27.74 |
|  4096 |   1024 |  45056 |   10.199 |   401.62 |   38.489 |    26.61 |
|  4096 |   1024 |  49152 |   10.655 |   384.42 |   40.071 |    25.55 |
|  4096 |   1024 |  53248 |   11.141 |   367.65 |   41.699 |    24.56 |
|  4096 |   1024 |  57344 |   11.608 |   352.86 |   43.462 |    23.56 |
|  4096 |   1024 |  61440 |   12.092 |   338.73 |   44.879 |    22.82 |
|  4096 |   1024 |  65536 |   12.674 |   323.17 |   46.555 |    22.00 |
|  4096 |   1024 |  69632 |   13.055 |   313.74 |   48.214 |    21.24 |
|  4096 |   1024 |  73728 |   13.538 |   302.55 |   49.780 |    20.57 |
|  4096 |   1024 |  77824 |   14.122 |   290.05 |   51.246 |    19.98 |
|  4096 |   1024 |  81920 |   14.513 |   282.23 |   52.963 |    19.33 |
|  4096 |   1024 |  86016 |   15.056 |   272.05 |   54.590 |    18.76 |
|  4096 |   1024 |  90112 |   15.451 |   265.09 |   56.048 |    18.27 |
|  4096 |   1024 |  94208 |   16.031 |   255.51 |   57.780 |    17.72 |
|  4096 |   1024 |  98304 |   16.410 |   249.61 |   59.421 |    17.23 |

## llama.cpp@19f4dec CUDA ubergarm/GLM-4.5-Air-Q4_0 -ctk q8_0 -ctv q8_0
|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  4096 |   1024 |      0 |    3.021 |  1355.73 |   21.133 |    48.46 |
|  4096 |   1024 |   4096 |    3.572 |  1146.75 |   26.617 |    38.47 |
|  4096 |   1024 |   8192 |    4.166 |   983.19 |   32.304 |    31.70 |
|  4096 |   1024 |  12288 |    4.817 |   850.25 |   38.066 |    26.90 |
|  4096 |   1024 |  16384 |    5.387 |   760.30 |   43.854 |    23.35 |
|  4096 |   1024 |  20480 |    6.027 |   679.63 |   49.559 |    20.66 |
|  4096 |   1024 |  24576 |    6.538 |   626.45 |   55.446 |    18.47 |
|  4096 |   1024 |  28672 |    7.433 |   551.05 |   61.323 |    16.70 |
|  4096 |   1024 |  32768 |    8.636 |   474.31 |   67.067 |    15.27 |
|  4096 |   1024 |  36864 |    8.510 |   481.30 |   72.836 |    14.06 |
|  4096 |   1024 |  40960 |    9.117 |   449.27 |   78.905 |    12.98 |
|  4096 |   1024 |  45056 |   10.182 |   402.27 |   84.452 |    12.13 |
|  4096 |   1024 |  49152 |   10.248 |   399.68 |   90.615 |    11.30 |
|  4096 |   1024 |  53248 |   10.896 |   375.92 |   96.755 |    10.58 |
|  4096 |   1024 |  57344 |   11.513 |   355.77 |  102.564 |     9.98 |
|  4096 |   1024 |  61440 |   11.813 |   346.75 |  108.832 |     9.41 |
|  4096 |   1024 |  65536 |   12.403 |   330.24 |  114.858 |     8.92 |
|  4096 |   1024 |  69632 |   13.278 |   308.47 |  121.222 |     8.45 |
|  4096 |   1024 |  73728 |   13.485 |   303.75 |  127.133 |     8.05 |
|  4096 |   1024 |  77824 |   13.984 |   292.91 |  133.428 |     7.67 |
|  4096 |   1024 |  81920 |   14.504 |   282.41 |  139.735 |     7.33 |
|  4096 |   1024 |  86016 |   15.127 |   270.77 |  145.894 |     7.02 |
|  4096 |   1024 |  90112 |   15.795 |   259.32 |  152.183 |     6.73 |
|  4096 |   1024 |  94208 |   16.297 |   251.34 |  158.356 |     6.47 |
|  4096 |   1024 |  98304 |   16.944 |   241.74 |  164.171 |     6.24 |

</details>

---

👤 **Thireus** commented on **2025-08-17** at **20:33:33**

@ubergarm - thank you for the tips and explanations.

You are maybe the 4th person this month to comment about the fact that the `--threads` parameters I use may not be optimum 😅. I'm starting to believe I'm imagining stuff. So, I have run yet another round of benchmarks based on your suggestion to debunk what appears to be a placebo for my config running Windows with a x299 CPU. See results below (and logs attached):

CPU: Intel® Core™ i9-7980XE with 18 cores and 36 threads

Commands:

```
for i in 36 1; do CUDA_DEVICE_ORDER=PCI_BUS_ID CUDA_VISIBLE_DEVICES=0 \
    ~/ik_llama-main-b4089-6908b72-bin-win-cuda-12.8-x64-avx512/llama-sweep-bench \
    -m GLM-4.5-Air-THIREUS-BF16-SPECIAL_TENSOR-00001-of-00804.gguf \
    -fa -fmoe \
    -c 135168 \
    -ngl 99 \
    -b 4096 -ub 4096 \
    --threads $i \
    --main-gpu 0 \
    --warmup-batch > GLM-4.5-Air_"4096"_"4096"_ik_llama-main-b4089-6908b72_threads_$i.txt 2>&1; done

for i in 36 1; do CUDA_DEVICE_ORDER=PCI_BUS_ID CUDA_VISIBLE_DEVICES=0 \
    ~/llama-b6222-bin-win-cuda-12.8-x64/llama-sweep-bench \
    -m GLM-4.5-Air-THIREUS-BF16-SPECIAL_TENSOR-00001-of-00804.gguf \
    -fa \
    -c 135168 \
    -ngl 99 \
    -b 4096 -ub 4096 \
    --threads $i \
    --main-gpu 0 > GLM-4.5-Air_"4096"_"4096"_llama-b6222_threads_$i.txt 2>&1; done
```

Binaries:
- ik_llama.cpp: https://github.com/Thireus/ik_llama.cpp/releases/tag/main-b4089-6908b72
- llama.cpp: https://github.com/Thireus/llama.cpp/releases/tag/b6222

<img width="1779" height="1181" alt="GLM-4 5-Air_spp" src="https://github.com/user-attachments/assets/a6b8b119-2887-4e9b-9811-4afc78e2d206" />
<img width="1820" height="1093" alt="GLM-4 5-Air_stg" src="https://github.com/user-attachments/assets/2090d5df-6660-4f58-b902-70142ae0a5a2" />

Recipe used:

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

I have also conducted additional tests when some layers are assigned to the CPU here: https://github.com/Thireus/GGUF-Tool-Suite/issues/20#issuecomment-3194241555

I believe the fact that you (and others) are observing a 5% loss when using too many threads may be due to how Windows manages threads differently than Linux. And I believe I'm confusing a lot of Linux folks, so I should probably use whatever number of threads is optimum on Linux despite running these benchmarks on Windows (when they have no effect on prefs of course)...

For DeepSeek-R1-0528 I have already benchmarked that **surprisingly** using --threads 36 (my max number of threads) performs noticeably better than --threads 18:

```
# --threads 18 + -ub 2028 -b 2048
CUDA_DEVICE_ORDER=PCI_BUS_ID \
CUDA_VISIBLE_DEVICES=0 \
~/ik_llama-main-b3746-f26fe36-bin-win-cuda-12.8-x64/llama-sweep-bench \
    -m DeepSeek-R1-0528-IQ1_S_R4-00001-of-00003.gguf \
    -c 8192 \
    -mla 3 -fa \
    -amb 512 \
    -fmoe \
    -ngl 99 \
    -ot exps=CPU \
    --warmup-batch \
    --threads 18 \
    -ub 2028 -b 2048
---
main: n_kv_max = 8192, n_batch = 2048, n_ubatch = 2028, flash_attn = 1, n_gpu_layers = 99, n_threads = 18, n_threads_batch = 18

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  2028 |    507 |      0 |  109.917 |    18.45 |  156.104 |     3.25 |
|  2028 |    507 |   2028 |  109.416 |    18.53 |  156.624 |     3.24 |
---

# --threads 36 + -ub 2028 -b 2048
CUDA_DEVICE_ORDER=PCI_BUS_ID \
CUDA_VISIBLE_DEVICES=0 \
~/ik_llama-main-b3746-f26fe36-bin-win-cuda-12.8-x64/llama-sweep-bench \
    -m DeepSeek-R1-0528-IQ1_S_R4-00001-of-00003.gguf \
    -c 8192 \
    -mla 3 -fa \
    -amb 512 \
    -fmoe \
    -ngl 99 \
    -ot exps=CPU \
    --warmup-batch \
    --threads 36 \
    -ub 2028 -b 2048
---
main: n_kv_max = 8192, n_batch = 2048, n_ubatch = 2028, flash_attn = 1, n_gpu_layers = 99, n_threads = 36, n_threads_batch = 36

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  2028 |    507 |      0 |   94.077 |    21.56 |  109.357 |     4.64 |
|  2028 |    507 |   2028 |  101.274 |    20.02 |  131.667 |     3.85 |
---
```

My conclusion is that I should keep using `--threads 36` for practical reasons, as evidence suggests that no noticeable degradation and on the contrary speed improvements are observed in some cases.

Let me know your thoughts.

[GLM-4.5-Air_threads_1vs36.zip](https://github.com/user-attachments/files/21824432/GLM-4.5-Air_threads_1vs36.zip)