## 🔀 [Pull Request #860](https://github.com/ikawrakow/ik_llama.cpp/pull/860) - Faster tensor name formatting

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/format_name` |
| **Target Branch** | `main` |
| **Created** | 2025-10-23 |
| **Updated** | 2025-10-24 |
| **Merged** | 2025-10-24 |

---

## 📄 Description

When building the compute graph, tensors get assigned names via `ggml_format_name`. For small models with very fast inference, a non-negligible fraction of the overall compute time (not just graph building time) is spent in that function. Given the names being assigned, we don't need to use `snprintf` as `ggml_format_name` does. This PR replaces calls to `ggml_format_name` with calls to a new function, which is very simple but still quite a bit faster.

For e.g. Ling-mini-2.0 running on RTX-4080 we get a ~1% TG performance gain.

Here some `sweep-bench` results with this PR for Ling-mini-2.0

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  2048 |    512 |      0 |    0.142 | 14420.91 |    1.220 |   419.50 |
|  2048 |    512 |   2048 |    0.117 | 17508.91 |    1.283 |   398.98 |
|  2048 |    512 |   4096 |    0.124 | 16528.93 |    1.362 |   375.81 |
|  2048 |    512 |   6144 |    0.129 | 15815.04 |    1.447 |   353.95 |
|  2048 |    512 |   8192 |    0.136 | 15058.71 |    1.505 |   340.12 |
|  2048 |    512 |  10240 |    0.143 | 14353.90 |    1.593 |   321.36 |
|  2048 |    512 |  12288 |    0.149 | 13723.51 |    1.644 |   311.36 |
|  2048 |    512 |  14336 |    0.156 | 13142.36 |    1.725 |   296.88 |
|  2048 |    512 |  16384 |    0.163 | 12563.18 |    1.780 |   287.61 |
|  2048 |    512 |  18432 |    0.170 | 12012.86 |    1.845 |   277.50 |
|  2048 |    512 |  20480 |    0.176 | 11605.83 |    1.929 |   265.47 |
|  2048 |    512 |  22528 |    0.184 | 11130.56 |    1.987 |   257.67 |
|  2048 |    512 |  24576 |    0.189 | 10838.27 |    2.074 |   246.81 |
|  2048 |    512 |  26624 |    0.197 | 10376.34 |    2.129 |   240.44 |
|  2048 |    512 |  28672 |    0.204 | 10015.94 |    2.193 |   233.44 |
|  2048 |    512 |  30720 |    0.209 |  9791.92 |    2.273 |   225.25 |

This is with grouped expert routing enabled (`-ger`). Compared to [#838](https://github.com/ikawrakow/ik_llama.cpp/issues/838), where grouped expert routing was added on CUDA, at low context TG performance has improved by ~11% and PP performance by 8%. This is the cumulative effect of this PR, and PRs [#858](https://github.com/ikawrakow/ik_llama.cpp/issues/858), [#853](https://github.com/ikawrakow/ik_llama.cpp/issues/853), [#845](https://github.com/ikawrakow/ik_llama.cpp/issues/845), [#840](https://github.com/ikawrakow/ik_llama.cpp/issues/840).