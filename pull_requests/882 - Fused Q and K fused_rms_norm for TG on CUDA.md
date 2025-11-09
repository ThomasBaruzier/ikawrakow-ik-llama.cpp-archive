## 🔀 [Pull Request #882](https://github.com/ikawrakow/ik_llama.cpp/pull/882) - Fused Q and K fused_rms_norm for TG on CUDA

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/fused_rms_rms` |
| **Target Branch** | `main` |
| **Created** | 2025-10-30 |
| **Updated** | 2025-10-31 |
| **Merged** | 2025-10-31 |

---

## 📄 Description

When Q, K, V model tensors are merged into a single tensor ([#878](https://github.com/ikawrakow/ik_llama.cpp/issues/878)), models which use `FUSED_RMS_NORM` for the `Q` and `K` portions of the result of the matrix multiplication of  QKV with the current activations (e.g., Qwen3-MoE, GPT-OSS, Ling/Ring, etc.) end up having two consecutive `FUSED_RMS_NORM` operations. This PR merges these two into a single CUDA kernel.  

We get in the range of 1% better TG when running with the model fully offloaded to the GPU (for models using `RMS_NORM` for `Q` and `K`).

Just for fun, here is a sweep bench for Ling-mini-2.0 with grouped expert routing enabled on RTX-4080 with this PR. A comparison with the main branch will look disappointing, so it is better to compare to PR [#838](https://github.com/ikawrakow/ik_llama.cpp/issues/838) where grouped expert routing was added for Ling/Ring models. We see TG performance has improved by about 17% as the result of multiple baby steps such as this PR.

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  2048 |    512 |      0 |    0.145 | 14132.42 |    1.166 |   439.18 |
|  2048 |    512 |   2048 |    0.118 | 17426.23 |    1.227 |   417.25 |
|  2048 |    512 |   4096 |    0.124 | 16542.81 |    1.310 |   390.79 |
|  2048 |    512 |   6144 |    0.138 | 14832.20 |    1.390 |   368.37 |
|  2048 |    512 |   8192 |    0.137 | 14998.17 |    1.473 |   347.68 |
|  2048 |    512 |  10240 |    0.149 | 13784.10 |    1.550 |   330.35 |
|  2048 |    512 |  12288 |    0.151 | 13596.68 |    1.592 |   321.54 |
|  2048 |    512 |  14336 |    0.156 | 13097.06 |    1.666 |   307.41 |
|  2048 |    512 |  16384 |    0.164 | 12493.67 |    1.725 |   296.76 |
|  2048 |    512 |  18432 |    0.171 | 12007.43 |    1.793 |   285.62 |
|  2048 |    512 |  20480 |    0.176 | 11618.34 |    1.878 |   272.67 |
|  2048 |    512 |  22528 |    0.184 | 11133.76 |    1.935 |   264.56 |
|  2048 |    512 |  24576 |    0.190 | 10770.67 |    2.016 |   253.96 |
|  2048 |    512 |  26624 |    0.198 | 10342.86 |    2.071 |   247.20 |
|  2048 |    512 |  28672 |    0.214 |  9555.94 |    2.160 |   237.01 |
|  2048 |    512 |  30720 |    0.211 |  9717.77 |    2.216 |   231.02 |

I experimented with adding the fusion also for prompt processing, but didn't see any performance improvement, so decided to keep the simpler TG-only fusion in this PR.

---

## 💬 Conversation

👤 **magikRUKKOLA** commented on **2025-10-30** at **21:42:25**

@ikawrakow 
> Just for fun, here is a sweep bench for Ling-mini-2.0 with grouped expert routing enabled on RTX-4080 with this PR.

Hey, Mr. Ivan!  Each time you're mentioning the improvements such and such I feel like something should be automated in this regard.  May be I can simply make a bash script that checks for the recent releases of ik_llama.cpp to checkout the performance of the widely used LLMs etc.?  I can upload the results per PR etc.  It would be much more convenient rather than manually testing this or that PR.

---

👤 **magikRUKKOLA** commented on **2025-10-30** at **21:44:42**

@ikawrakow 

Actually I am pretty sure I mentioned it several times in the past but I never got any response regarding this issue.

You do understand that your code is like enterprise-level etc.?  I just suggest to put more effort into testing and automation.  Of course I would not do any movements towards it without an approval from the person in charge.

---

👤 **Nexesenex** commented on **2025-10-30** at **23:38:48**

LGTM.
I repeated my tests at 10k context with merged QKV and fused QK RMS Norm, and my TG now closes on 10.5t/s with a Q5_K_S quant of L3.3 70B, instead of hovering around 10.25 before those 2 PRs.

TG is the hardest thing to improve, a boost of 3-5% is really damn good!

---

👤 **ikawrakow** commented on **2025-10-31** at **06:22:11**

@magikRUKKOLA 

Yes, if I can get help with that, automation and testing is definitely good.

But if I'm to choose between working on stuff that is fun for me, or working on automation and testing, I will pick the former every day of the week.