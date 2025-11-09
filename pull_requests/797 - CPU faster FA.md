## 🔀 [Pull Request #797](https://github.com/ikawrakow/ik_llama.cpp/pull/797) - CPU: faster FA 

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/better_fa_masking` |
| **Target Branch** | `main` |
| **Created** | 2025-09-24 |
| **Updated** | 2025-09-26 |
| **Merged** | 2025-09-26 |

---

## 📄 Description

This PR adds a more effective use of the KQ mask to skip portions of the computation in the CPU FA kernels that are fully masked out.

This results in noticeable  PP performance gains of up to ~10% for the tested models.

Here is an example of a sweep bench for `IQ2_XXS` quantized Qwen3-30B-A3B model using `Q8_0` K-cache and running on a Ryzen-7950C CPU:

### Main branch

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  1024 |    256 |      0 |    2.046 |   500.41 |    6.836 |    37.45 |
|  1024 |    256 |   1024 |    2.309 |   443.47 |    7.469 |    34.28 |
|  1024 |    256 |   2048 |    2.662 |   384.63 |    7.858 |    32.58 |
|  1024 |    256 |   3072 |    3.005 |   340.80 |    8.510 |    30.08 |
|  1024 |    256 |   4096 |    3.345 |   306.11 |    8.909 |    28.73 |
|  1024 |    256 |   5120 |    3.683 |   278.06 |    9.360 |    27.35 |
|  1024 |    256 |   6144 |    4.029 |   254.15 |    9.773 |    26.20 |
|  1024 |    256 |   7168 |    4.368 |   234.42 |   10.203 |    25.09 |

### PR

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  1024 |    256 |      0 |    1.884 |   543.48 |    6.843 |    37.41 |
|  1024 |    256 |   1024 |    2.140 |   478.51 |    7.424 |    34.48 |
|  1024 |    256 |   2048 |    2.457 |   416.82 |    7.753 |    33.02 |
|  1024 |    256 |   3072 |    2.777 |   368.81 |    8.257 |    31.01 |
|  1024 |    256 |   4096 |    3.182 |   321.78 |    8.787 |    29.13 |
|  1024 |    256 |   5120 |    3.453 |   296.56 |    9.330 |    27.44 |
|  1024 |    256 |   6144 |    3.743 |   273.61 |    9.809 |    26.10 |
|  1024 |    256 |   7168 |    4.065 |   251.91 |   10.228 |    25.03 |

---

## 💬 Conversation

👤 **ikawrakow** commented on **2025-09-24** at **15:33:43**

So, after 3 relatively minor optimizations (PR [#794](https://github.com/ikawrakow/ik_llama.cpp/issues/794), PR [#796](https://github.com/ikawrakow/ik_llama.cpp/issues/796), and this PR), we end up with a non-negligible PP and TG performance gain. Here two graphs comparing this PR with 8cd2d7ccd7339756c11c893b014046dd6be37bb1 (last version before [#794](https://github.com/ikawrakow/ik_llama.cpp/issues/794)) for `IQ2_XXS` quantized Qwen3-30B-A3B running on Ryzen-7950X with `Q8_0` K-cache

### PP

<img width="792" height="612" alt="u16_pp" src="https://github.com/user-attachments/assets/dd9de5bd-bf3a-4b6a-9525-3316b905ce80" />

### TG

<img width="792" height="612" alt="u16_tg" src="https://github.com/user-attachments/assets/0782679a-f247-46be-96f4-2c7712f86af9" />

I didn't add mainline's data points for the above graphs to not stretch the y-axis plot range too much. But just for fun, here the sweep-bench data points for this PR and today's mainline:

### ik_llama.cpp, this PR

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  1024 |    256 |      0 |    1.842 |   555.79 |    6.622 |    38.66 |
|  1024 |    256 |   1024 |    2.099 |   487.77 |    7.396 |    34.61 |
|  1024 |    256 |   2048 |    2.413 |   424.45 |    7.748 |    33.04 |
|  1024 |    256 |   3072 |    2.747 |   372.76 |    8.252 |    31.02 |
|  1024 |    256 |   4096 |    3.135 |   326.62 |    8.714 |    29.38 |
|  1024 |    256 |   5120 |    3.395 |   301.60 |    9.055 |    28.27 |
|  1024 |    256 |   6144 |    3.719 |   275.32 |    9.456 |    27.07 |
|  1024 |    256 |   7168 |    4.045 |   253.17 |    9.858 |    25.97 |

### llama.cpp (commit `e789095`)

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  1024 |    256 |      0 |   10.359 |    98.85 |    7.851 |    32.61 |
|  1024 |    256 |   1024 |   12.965 |    78.98 |    9.482 |    27.00 |
|  1024 |    256 |   2048 |   15.581 |    65.72 |   10.947 |    23.39 |
|  1024 |    256 |   3072 |   18.298 |    55.96 |   12.413 |    20.62 |
|  1024 |    256 |   4096 |   20.960 |    48.86 |   14.532 |    17.62 |
|  1024 |    256 |   5120 |   23.796 |    43.03 |   16.491 |    15.52 |
|  1024 |    256 |   6144 |   26.266 |    38.99 |   16.109 |    15.89 |
|  1024 |    256 |   7168 |   29.873 |    34.28 |   19.574 |    13.08 |

---

👤 **Ph0rk0z** commented on **2025-09-25** at **16:56:31**

Got 1/2 T/S+ on ernie, both t/g and PP. So far no negative side effects.

---

👤 **ikawrakow** commented on **2025-09-25** at **17:22:03**

> Got 1/2 T/S+ on ernie, both t/g and PP. So far no negative side effects.

1/2 T/S+ as in 50 t/s -> 50.5 t/s, or 50 t/s -> 75 t/s?

---

👤 **Ph0rk0z** commented on **2025-09-25** at **18:15:08**

Yea like 50 -> 50.5 t/s. Maybe it will be more noticeable on qwen. Still, I'll take it.

---

👤 **ubergarm** commented on **2025-09-25** at **18:51:53**

I ran two test cases, one on my local AMD 9950X gaming rig with a 14B dense model, and one on the remote AMD EPYC 9965 192-Core, using single socket on DeepSeek-V3.1-Terminus.

Seems like a very slight uplift for TG on the AMD 9950X. PP seemed about the same or uplift is within the noise so hard to say with a single sweep. ik is *much* faster than mainline in this CPU-only compiled situation. ik can saturate the theoretical max speeds based on measured memory bandwidth during TG which is pretty amazing. Put it on two graphs to make it easier to see.

<img width="2087" height="1082" alt="sweep-bench-pr797-qwen3-14b" src="https://github.com/user-attachments/assets/8e86764e-560d-4591-a023-1be4fb299d95" />

<img width="2087" height="1110" alt="sweep-bench-pr797-qwen3-14b-vs-mainline" src="https://github.com/user-attachments/assets/6a5c8a9c-7709-4945-a47d-c0f43067fa6b" />

<details>

<summary>👈 Details</summary>

```bash
./build/bin/llama-sweep-bench \
  --model "$model" \
  -fa \ # -fa 1 on mainline now
  -ctk q8_0 -ctv q8_0 \
  -c 8704 \
  --warmup-batch \
  --threads 16

## main@6bb76b14 default batches
|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|   512 |    128 |      0 |    1.843 |   277.80 |   12.438 |    10.29 |
|   512 |    128 |    512 |    1.896 |   270.11 |   12.503 |    10.24 |
|   512 |    128 |   1024 |    1.935 |   264.54 |   12.630 |    10.13 |
|   512 |    128 |   1536 |    1.989 |   257.36 |   12.745 |    10.04 |
|   512 |    128 |   2048 |    2.045 |   250.32 |   12.874 |     9.94 |
|   512 |    128 |   2560 |    2.082 |   245.91 |   12.975 |     9.87 |
|   512 |    128 |   3072 |    2.140 |   239.21 |   13.096 |     9.77 |
|   512 |    128 |   3584 |    2.188 |   234.03 |   13.244 |     9.66 |
|   512 |    128 |   4096 |    2.246 |   227.97 |   13.452 |     9.52 |
|   512 |    128 |   4608 |    2.284 |   224.20 |   13.612 |     9.40 |
|   512 |    128 |   5120 |    2.350 |   217.91 |   13.710 |     9.34 |
|   512 |    128 |   5632 |    2.398 |   213.55 |   13.849 |     9.24 |
|   512 |    128 |   6144 |    2.436 |   210.20 |   13.994 |     9.15 |
|   512 |    128 |   6656 |    2.478 |   206.58 |   14.021 |     9.13 |
|   512 |    128 |   7168 |    2.544 |   201.28 |   14.107 |     9.07 |
|   512 |    128 |   7680 |    2.575 |   198.81 |   14.081 |     9.09 |
|   512 |    128 |   8192 |    2.642 |   193.76 |   14.261 |     8.98 |

## PR797 ik/better_fa_masking@11621fe4 default batches
|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|   512 |    128 |      0 |    1.838 |   278.58 |   12.336 |    10.38 |
|   512 |    128 |    512 |    1.894 |   270.28 |   12.442 |    10.29 |
|   512 |    128 |   1024 |    1.939 |   264.08 |   12.531 |    10.21 |
|   512 |    128 |   1536 |    1.992 |   257.06 |   12.661 |    10.11 |
|   512 |    128 |   2048 |    2.036 |   251.42 |   12.765 |    10.03 |
|   512 |    128 |   2560 |    2.087 |   245.32 |   12.878 |     9.94 |
|   512 |    128 |   3072 |    2.145 |   238.74 |   13.029 |     9.82 |
|   512 |    128 |   3584 |    2.182 |   234.63 |   13.177 |     9.71 |
|   512 |    128 |   4096 |    2.236 |   228.96 |   13.318 |     9.61 |
|   512 |    128 |   4608 |    2.274 |   225.11 |   13.460 |     9.51 |
|   512 |    128 |   5120 |    2.321 |   220.55 |   13.650 |     9.38 |
|   512 |    128 |   5632 |    2.388 |   214.43 |   13.803 |     9.27 |
|   512 |    128 |   6144 |    2.432 |   210.56 |   13.933 |     9.19 |
|   512 |    128 |   6656 |    2.479 |   206.50 |   14.161 |     9.04 |
|   512 |    128 |   7168 |    2.515 |   203.56 |   14.221 |     9.00 |
|   512 |    128 |   7680 |    2.689 |   190.39 |   14.288 |     8.96 |
|   512 |    128 |   8192 |    2.627 |   194.89 |   14.525 |     8.81 |

## mainline llama.cpp master@835b2b91 + ug/port-sweep-bench
|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|   512 |    128 |      0 |    4.686 |   109.26 |   12.155 |    10.53 |
|   512 |    128 |    512 |    7.533 |    67.97 |   12.690 |    10.09 |
|   512 |    128 |   1024 |   10.493 |    48.79 |   13.601 |     9.41 |
|   512 |    128 |   1536 |   13.532 |    37.84 |   14.448 |     8.86 |
|   512 |    128 |   2048 |   16.227 |    31.55 |   15.422 |     8.30 |
|   512 |    128 |   2560 |   19.495 |    26.26 |   16.741 |     7.65 |
|   512 |    128 |   3072 |   22.003 |    23.27 |   17.411 |     7.35 |
|   512 |    128 |   3584 |   24.796 |    20.65 |   18.744 |     6.83 |
|   512 |    128 |   4096 |   27.440 |    18.66 |   19.325 |     6.62 |
|   512 |    128 |   4608 |   30.875 |    16.58 |   20.580 |     6.22 |
|   512 |    128 |   5120 |   33.334 |    15.36 |   21.387 |     5.98 |
|   512 |    128 |   5632 |   36.237 |    14.13 |   22.165 |     5.77 |
|   512 |    128 |   6144 |   38.980 |    13.13 |   23.980 |     5.34 |
|   512 |    128 |   6656 |   42.005 |    12.19 |   25.104 |     5.10 |
|   512 |    128 |   7168 |   44.842 |    11.42 |   26.050 |     4.91 |
|   512 |    128 |   7680 |   47.901 |    10.69 |   27.580 |     4.64 |
|   512 |    128 |   8192 |   50.316 |    10.18 |   27.895 |     4.59 |
```

</details>

Harder to say about this PR on the remote rig with a big model which tends to have more noisy sweeps in general, but here is the graph and data.

<img width="2087" height="1109" alt="sweep-bench-pr797" src="https://github.com/user-attachments/assets/0aa70038-c7b1-4e2f-8d1a-1b3dfa287701" />

<details>

<summary>👈 Details</summary>

```bash
SOCKET=1
numactl -N "$SOCKET" -m "$SOCKET" \
./build/bin/llama-sweep-bench \
    --model "$model"\
    -ctk q8_0 \
    --ctx-size 8704 \
    -fa -fmoe \
    -mla 3 \
    --threads 128 \
    --threads-batch 192 \
    --numa numactl \
    --no-mmap

## main@6bb76b14 default batches
|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|   512 |    128 |      0 |    2.732 |   187.44 |   12.877 |     9.94 |
|   512 |    128 |    512 |    4.244 |   120.64 |   13.223 |     9.68 |
|   512 |    128 |   1024 |    3.746 |   136.67 |   13.293 |     9.63 |
|   512 |    128 |   1536 |    5.496 |    93.16 |   13.412 |     9.54 |
|   512 |    128 |   2048 |    3.530 |   145.03 |   13.459 |     9.51 |
|   512 |    128 |   2560 |    4.366 |   117.26 |   13.475 |     9.50 |
|   512 |    128 |   3072 |    4.392 |   116.57 |   13.592 |     9.42 |
|   512 |    128 |   3584 |    4.645 |   110.23 |   13.648 |     9.38 |
|   512 |    128 |   4096 |    4.425 |   115.72 |   13.673 |     9.36 |
|   512 |    128 |   4608 |    4.479 |   114.31 |   13.815 |     9.27 |
|   512 |    128 |   5120 |    5.113 |   100.13 |   13.843 |     9.25 |
|   512 |    128 |   5632 |    4.028 |   127.10 |   13.916 |     9.20 |
|   512 |    128 |   6144 |    4.528 |   113.06 |   14.002 |     9.14 |
|   512 |    128 |   6656 |    4.613 |   110.99 |   14.002 |     9.14 |
|   512 |    128 |   7168 |    4.612 |   111.02 |   14.107 |     9.07 |
|   512 |    128 |   7680 |    5.161 |    99.20 |   14.195 |     9.02 |
|   512 |    128 |   8192 |    5.456 |    93.83 |   14.244 |     8.99 |

## main@6bb76b14 -ub 4096 -b 4096
|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  4096 |   1024 |      0 |   17.393 |   235.49 |  107.195 |     9.55 |
|  4096 |   1024 |   4096 |   20.092 |   203.86 |  111.865 |     9.15 |
|  4096 |   1024 |   8192 |   25.919 |   158.03 |  116.941 |     8.76 |

## PR797@11621fe4 default batches
|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|   512 |    128 |      0 |    2.428 |   210.85 |   12.963 |     9.87 |
|   512 |    128 |    512 |    3.837 |   133.45 |   13.238 |     9.67 |
|   512 |    128 |   1024 |    3.486 |   146.88 |   13.362 |     9.58 |
|   512 |    128 |   1536 |    3.980 |   128.64 |   13.513 |     9.47 |
|   512 |    128 |   2048 |    6.060 |    84.49 |   13.540 |     9.45 |
|   512 |    128 |   2560 |    4.290 |   119.34 |   13.554 |     9.44 |
|   512 |    128 |   3072 |    4.957 |   103.29 |   13.657 |     9.37 |
|   512 |    128 |   3584 |    4.125 |   124.11 |   13.749 |     9.31 |
|   512 |    128 |   4096 |    5.344 |    95.82 |   13.961 |     9.17 |
|   512 |    128 |   4608 |    4.452 |   115.01 |   13.850 |     9.24 |
|   512 |    128 |   5120 |    4.007 |   127.78 |   13.949 |     9.18 |
|   512 |    128 |   5632 |    5.972 |    85.73 |   14.063 |     9.10 |
|   512 |    128 |   6144 |    5.351 |    95.68 |   14.212 |     9.01 |
|   512 |    128 |   6656 |    4.964 |   103.15 |   14.276 |     8.97 |
|   512 |    128 |   7168 |    6.411 |    79.86 |   14.382 |     8.90 |
|   512 |    128 |   7680 |    5.231 |    97.87 |   14.351 |     8.92 |
|   512 |    128 |   8192 |    5.276 |    97.04 |   14.309 |     8.95 |

## PR797@11621fe4 -ub 4096 -b 4096
|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  4096 |   1024 |      0 |   17.033 |   240.47 |  106.565 |     9.61 |
|  4096 |   1024 |   4096 |   21.454 |   190.92 |  111.898 |     9.15 |
|  4096 |   1024 |   8192 |   26.107 |   156.89 |  117.965 |     8.68 |
```

</details>

---

👤 **ubergarm** commented on **2025-09-25** at **19:52:09**

Okay one more test, this one clearly shows this PR797 is better than main on my local gaming rig running GLM-4.5-Air IQ2_KL. I used `-ub 1024` mainly to match what ik was doing above.

<img width="2087" height="1082" alt="sweep-bench-pr797-air" src="https://github.com/user-attachments/assets/3a865702-98be-4fac-bb05-f9b18234a51e" />

<details>

<summary>👈 Details</summary>

```bash
model=/mnt/models/ubergarm/GLM-4.5-Air-GGUF/GLM-4.5-Air-IQ2_KL.gguf
./build/bin/llama-sweep-bench \
  --model "$model" \
  -fa -fmoe \
  -ctk q8_0 -ctv q8_0 \
  -c 9216 \
  -ub 1024 \
  --warmup-batch \
  --threads 16 \
  --no-mmap

## main@6bb76b14 -ub 1024
|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  1024 |    256 |      0 |    4.849 |   211.19 |   20.204 |    12.67 |
|  1024 |    256 |   1024 |    5.233 |   195.68 |   20.885 |    12.26 |
|  1024 |    256 |   2048 |    5.375 |   190.52 |   21.689 |    11.80 |
|  1024 |    256 |   3072 |    5.912 |   173.20 |   22.500 |    11.38 |
|  1024 |    256 |   4096 |    6.537 |   156.64 |   23.411 |    10.93 |
|  1024 |    256 |   5120 |    6.949 |   147.35 |   23.844 |    10.74 |
|  1024 |    256 |   6144 |    7.474 |   137.01 |   24.723 |    10.35 |
|  1024 |    256 |   7168 |    7.987 |   128.20 |   25.616 |     9.99 |
|  1024 |    256 |   8192 |    8.534 |   119.99 |   26.547 |     9.64 |

## PR797 ik/better_fa_masking@11621fe4 -ub 1024
|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  1024 |    256 |      0 |    4.112 |   249.01 |   20.093 |    12.74 |
|  1024 |    256 |   1024 |    4.648 |   220.33 |   20.897 |    12.25 |
|  1024 |    256 |   2048 |    5.134 |   199.46 |   21.565 |    11.87 |
|  1024 |    256 |   3072 |    5.722 |   178.97 |   22.305 |    11.48 |
|  1024 |    256 |   4096 |    6.401 |   159.99 |   22.901 |    11.18 |
|  1024 |    256 |   5120 |    6.729 |   152.18 |   23.678 |    10.81 |
|  1024 |    256 |   6144 |    7.605 |   134.64 |   24.487 |    10.45 |
|  1024 |    256 |   7168 |    7.735 |   132.38 |   25.294 |    10.12 |
|  1024 |    256 |   8192 |    8.206 |   124.79 |   26.301 |     9.73 |
```

</details>

---

👤 **magikRUKKOLA** commented on **2025-09-26** at **01:11:46**

ubergarm/DeepSeek-V3.1-Terminus/IQ5_K

```
llm_load_print_meta: model ftype      = IQ5_K - 5.5 bpw                                                                     
llm_load_print_meta: model params     = 671.026 B                                                                           
llm_load_print_meta: model size       = 464.062 GiB (5.941 BPW)   
```

master:
```
main: n_kv_max = 143360, n_batch = 8192, n_ubatch = 4096, flash_attn = 1, n_gpu_layers = 99, n_threads = 64, n_threads_batch = 64

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  4096 |   1024 |      0 |   32.522 |   125.95 |  138.529 |     7.39 |
|  4096 |   1024 |   4096 |   33.064 |   123.88 |  141.525 |     7.24 |
|  4096 |   1024 |   8192 |   33.588 |   121.95 |  146.144 |     7.01 |
```


pr-797:
```
|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  4096 |   1024 |      0 |   32.634 |   125.51 |  138.274 |     7.41 |
|  4096 |   1024 |   4096 |   33.103 |   123.74 |  141.111 |     7.26 |
|  4096 |   1024 |   8192 |   33.609 |   121.87 |  145.452 |     7.04 |
```

---

👤 **ikawrakow** commented on **2025-09-26** at **06:37:51**

@ubergarm @magikRUKKOLA Thanks for testing!

This PR only touches the self-attention portion of the calculation, so has more impact when attention represents a larger fraction of the computational cost, which seems to be the case for Qwen3-MoE.

The more interesting comparison on other platforms and for other models is against  8cd2d7c, sp one can see the combined effect of the 3 micro-optimizations (which is more noticeable than each one of the 3 separately).