## 🔀 [Pull Request #897](https://github.com/ikawrakow/ik_llama.cpp/pull/897) - sweep-bench: QoL improvement

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/sweep_bench_n_predict` |
| **Target Branch** | `main` |
| **Created** | 2025-11-04 |
| **Updated** | 2025-11-05 |
| **Merged** | 2025-11-04 |

---

## 📄 Description

When using `llama-sweep-bench` to evaluate performance over a range of context lengths, one often uses a larger u-batch (which determines the step between consecutive context lengths) to cover the range more quickly (with fewer steps). One may also set the u-batch size to a larger value for MoE models (because this improves PP performance). But using a large u-batch results in a slow benchmark because the number of generated tokens used to determine TG performance is hard-coded to u-batch/4, which can get annoying rather quickly.

This PR adds a QoL improvement for `llama-sweep-bench` by allowing the number of TG tokens to be set via `-n` (or the equivalent `--predict` or `--n-predict`). If present, the number of TG tokens is taken from the command line argument, else it is set to u-batch/4 as before.

Here an example run with `-c 16384 -ub 2048 -n 64`, which finishes in 7.2 seconds instead of the 25 seconds required on the main branch. 

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  2048 |     64 |      0 |    0.264 |  7764.02 |    0.248 |   258.56 |
|  2048 |     64 |   2048 |    0.288 |  7107.58 |    0.266 |   240.49 |
|  2048 |     64 |   4096 |    0.317 |  6452.77 |    0.288 |   222.30 |
|  2048 |     64 |   6144 |    0.346 |  5911.81 |    0.311 |   205.56 |
|  2048 |     64 |   8192 |    0.374 |  5471.47 |    0.329 |   194.50 |
|  2048 |     64 |  10240 |    0.405 |  5059.66 |    0.355 |   180.21 |
|  2048 |     64 |  12288 |    0.436 |  4698.85 |    0.369 |   173.43 |
|  2048 |     64 |  14336 |    0.467 |  4385.75 |    0.388 |   165.09 |

---

## 💬 Conversation

👤 **ubergarm** commented on **2025-11-05** at **15:16:59**

Very handy! In one quick test on my local gaming rig can now get through a sweep in a quarter of time!

I added your patch on this mainline fork/branch I use for testing: https://github.com/ubergarm/llama.cpp/tree/ug/port-sweep-bench

Thanks!