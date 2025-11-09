## 🔀 [Pull Request #835](https://github.com/ikawrakow/ik_llama.cpp/pull/835) - Better argsort (CPU)

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/cpu_argsort` |
| **Target Branch** | `main` |
| **Created** | 2025-10-15 |
| **Updated** | 2025-10-16 |
| **Merged** | 2025-10-16 |

---

## 📄 Description

In the `ARGSORT` op implementation inherited from `llama.cpp` there is a [comment](https://github.com/ikawrakow/ik_llama.cpp/blob/9d364b88ba91450e00230453321f5762708ef54f/ggml/src/ggml.c#L19922) that C does not have a functional sort, so the implementation just uses bubble sort. This is still true in the current mainline `llama.cpp` implementation, even though the CPU implementation is in C++ where we do indeed have better sorting algorithms than bubble sort.

This PR moves the `ARGSORT` into a C++ file, and uses `std::sort / std::partial_sort`.

The `ARGSORT` op is modified to take an additional argument that specifies in how many of the sorted items we are interested. When selecting the top experts via `ggml_top_k`, this new parameter is set to the number of active experts, thus allowing to use `std::partial_sort`, which is quite a bit faster than full sorting when the number of active experts is small compared to the total number of experts.

For a model such as Ling-mini-2.0, which uses 8 active experts out of 256 total experts, but only has 1B active parameters (so inference is very fast), this results in a noticeable performance gain when running on the CPU.

Here `sweep-bench` results for `Q4_K_M` quantized Ling-mini-2.0 running on Ryzen-7950X

### Main branch

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  2048 |    512 |      0 |    1.592 |  1286.32 |    8.103 |    63.19 |
|  2048 |    512 |   2048 |    1.777 |  1152.20 |    9.224 |    55.50 |
|  2048 |    512 |   4096 |    2.026 |  1010.78 |    9.704 |    52.76 |
|  2048 |    512 |   6144 |    2.290 |   894.41 |   10.354 |    49.45 |
|  2048 |    512 |   8192 |    2.553 |   802.12 |   11.019 |    46.46 |
|  2048 |    512 |  10240 |    2.797 |   732.15 |   11.679 |    43.84 |
|  2048 |    512 |  12288 |    3.210 |   637.92 |   12.427 |    41.20 |
|  2048 |    512 |  14336 |    3.527 |   580.59 |   13.113 |    39.05 |

### PR

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  2048 |    512 |      0 |    1.488 |  1376.55 |    7.848 |    65.24 |
|  2048 |    512 |   2048 |    1.684 |  1216.24 |    8.777 |    58.34 |
|  2048 |    512 |   4096 |    1.942 |  1054.43 |    9.279 |    55.18 |
|  2048 |    512 |   6144 |    2.212 |   925.70 |    9.967 |    51.37 |
|  2048 |    512 |   8192 |    2.475 |   827.37 |   10.507 |    48.73 |
|  2048 |    512 |  10240 |    2.792 |   733.44 |   11.395 |    44.93 |
|  2048 |    512 |  12288 |    3.024 |   677.28 |   11.842 |    43.24 |
|  2048 |    512 |  14336 |    3.400 |   602.39 |   12.651 |    40.47 |

---

## 💬 Conversation

👤 **magikRUKKOLA** commented on **2025-10-16** at **00:47:23**

A bubble sort?!

[EDIT]:

Is there any profile information regarding in which routines the code is spending most of the time etc.?  Is it possible to dump the data which is sorted in order to determine the most optimal algo etc.?

---

👤 **magikRUKKOLA** commented on **2025-10-16** at **01:57:52**

DeepSeek-V3.1-Terminus-5.4498bpw

*edited

```
/opt/ik_llama.cpp/ik_llama.cpp/build/bin/llama-server --version
version: 3908 (41bdd8655)
built with cc (Debian 15.2.0-4) 15.2.0 for x86_64-linux-gnu

n_kv_max = 131072, n_batch = 8192, n_ubatch = 8192, flash_attn = 1, n_gpu_layers = 99, n_threads = 64, n_threads_batch = 64

    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
-------|--------|--------|----------|----------|----------|----------|
  8192 |   2048 |      0 |   35.329 |   231.88 |  267.697 |     7.65 |
  8192 |   2048 |   8192 |   39.120 |   209.41 |  280.846 |     7.29 |
  8192 |   2048 |  16384 |   47.327 |   173.09 |  293.825 |     6.97 |
  8192 |   2048 |  24576 |   55.438 |   147.77 |  305.927 |     6.69 |

```

with this pr:

```
 main: n_kv_max = 131072, n_batch = 8192, n_ubatch = 8192, flash_attn = 1, n_gpu_layers = 99, n_threads = 64, n_threads_batch = 64

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  8192 |   2048 |      0 |   35.109 |   233.33 |  265.984 |     7.70 |
|  8192 |   2048 |   8192 |   38.702 |   211.67 |  278.796 |     7.35 |
|  8192 |   2048 |  16384 |   46.606 |   175.77 |  291.971 |     7.01 |
|  8192 |   2048 |  24576 |   54.872 |   149.29 |  304.426 |     6.73 |
```

prefill: ~+1%
decode: ~+0.6%