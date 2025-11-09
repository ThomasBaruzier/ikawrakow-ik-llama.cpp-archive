## 🔀 [Pull Request #841](https://github.com/ikawrakow/ik_llama.cpp/pull/841) - Change --n-cpu-moe to not keep expert biases on CPU

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/n_cpu_moe` |
| **Target Branch** | `main` |
| **Created** | 2025-10-19 |
| **Updated** | 2025-10-19 |
| **Merged** | 2025-10-19 |

---

## 📄 Description

The `--n-cpu-moe` and `--cpu-moe` command line options add tensor overrides of type
```
\\.ffn_(up|down|gate)_exps=CPU
```
When the routed experts have biases (e.g. gpt-oss models), this results in a significantly lower PP performance compared to using
```
\\.ffn_(up|down|gate)_exps\\.weight=CPU
```
and thus having the biases loaded to the GPU (see table below).  This PR changes the `--n-cpu-moe` and `--cpu-moe` regular expressions to the above version.

Here is an example using GPT-OSS-20B-MXFP4 on Ryzen-7950X/RTX-4080. The model does fully fit on the RTX-4080, but for the sake of an example `--cpu-moe` is added to the `llama-sweep-bench` command.

### On main branch

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  2048 |    512 |      0 |    0.925 |  2215.17 |   15.054 |    34.01 |
|  2048 |    512 |   2048 |    0.914 |  2240.81 |   15.080 |    33.95 |
|  2048 |    512 |   4096 |    0.920 |  2226.23 |   15.145 |    33.81 |
|  2048 |    512 |   6144 |    0.928 |  2206.20 |   15.261 |    33.55 |
|  2048 |    512 |   8192 |    0.935 |  2190.92 |   15.304 |    33.45 |
|  2048 |    512 |  10240 |    0.946 |  2165.40 |   15.349 |    33.36 |
|  2048 |    512 |  12288 |    0.952 |  2150.17 |   15.322 |    33.42 |
|  2048 |    512 |  14336 |    0.962 |  2128.93 |   15.375 |    33.30 |
|  2048 |    512 |  16384 |    0.972 |  2107.13 |   15.522 |    32.99 |

### This PR

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  2048 |    512 |      0 |    0.693 |  2954.21 |   15.016 |    34.10 |
|  2048 |    512 |   2048 |    0.686 |  2983.95 |   15.054 |    34.01 |
|  2048 |    512 |   4096 |    0.696 |  2942.64 |   15.086 |    33.94 |
|  2048 |    512 |   6144 |    0.703 |  2912.48 |   15.130 |    33.84 |
|  2048 |    512 |   8192 |    0.711 |  2879.00 |   15.159 |    33.78 |
|  2048 |    512 |  10240 |    0.718 |  2852.11 |   15.224 |    33.63 |
|  2048 |    512 |  12288 |    0.727 |  2816.66 |   15.273 |    33.52 |
|  2048 |    512 |  14336 |    0.735 |  2788.08 |   15.301 |    33.46 |
|  2048 |    512 |  16384 |    0.744 |  2750.86 |   15.369 |    33.31 |