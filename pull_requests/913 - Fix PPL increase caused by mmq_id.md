## 🔀 [Pull Request #913](https://github.com/ikawrakow/ik_llama.cpp/pull/913) - Fix PPL increase caused by mmq_id

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/fix_mmq_id` |
| **Target Branch** | `main` |
| **Created** | 2025-11-07 |
| **Updated** | 2025-11-07 |
| **Merged** | 2025-11-07 |

---

## 📄 Description

@Nexesenex noticed that the `mmq_id` MoE matrix multiplication approach added in [#728](https://github.com/ikawrakow/ik_llama.cpp/issues/728) leads to a non-negligible increase in perplexity (https://github.com/ikawrakow/ik_llama.cpp/pull/728#issuecomment-3494522254). As PR [#728](https://github.com/ikawrakow/ik_llama.cpp/issues/728) is derived from [PR 15525](https://github.com/ggml-org/llama.cpp/pull/15525) in mainline `llama.cpp`, the same issue exists there (see also https://github.com/ikawrakow/ik_llama.cpp/pull/728#issuecomment-3496574276). As a result, I added the ability to disable `mmq_id` via a command line parameter in [#910](https://github.com/ikawrakow/ik_llama.cpp/issues/910). 

I don't like the [#910](https://github.com/ikawrakow/ik_llama.cpp/issues/910) solution because disabling `mmq_id` leads to a significantly lower PP performance for small batch sizes. So, after some [back-and-fort](https://github.com/ikawrakow/ik_llama.cpp/pull/728#issuecomment-3501039000) with @JohannesGaessler, I went ahead and changed the `mmq_id` implementation to pretend that the number of streaming multiprocessors (SM) is a power of 2. This does in fact fix the PPL increase for all models I tried (GLM-4.5-AIR, Ling-mini-2.0, Qwen3-30B-A3B, DeepSeek-Lite). Based on experimentation with these models, it seems using the lowest power of two that is >= number of SM gives better performance than using the highest power of two that is <= number of SM.

I don't see any negative impact on performance. If anything, on my RTX-4080 GPU PP performance is very slightly better.

Please try and report any performance degradation (or performance improvement). To make sure that you are using  `mmq_id` rather than the original `ik_llama.cpp` MoE matrix multiplication implementation, add `-cuda  mmq-id-size=1000` to your command line.

---

## 💬 Conversation

👤 **Nexesenex** commented on **2025-11-07** at **14:30:28**

Testing right now.

Yesterday, with this command line (equivalent to pre-PR 728)

`llama-perplexity -m X:\text-generation-webui\models\zai-org_GLM-4.5-Air-bf16-iMat-IQ4_XS_L-00001-of-00005.gguf -f wiki.test.raw --parallel 1 -ngl 150 -b 512 -mg 0 -ts 18,18,12 --no-mmap -no-fmoe -no-mmad -cuda fusion=1,offload-batch-size=1,mmq-id-size=0 -c 512 --override-kv glm4moe.expert_used_count=int:7`

I obtained: Final estimate: PPL = 4.6508 +/- 0.02854

---

Today, with this command line, activating MMQ_ID and the proposed fix :

`llama-perplexity -m X:\text-generation-webui\models\zai-org_GLM-4.5-Air-bf16-iMat-IQ4_XS_L-00001-of-00005.gguf -f wiki.test.raw --parallel 1 -ngl 150 -b 512 -mg 0 -ts 18,18,12 --no-mmap -no-fmoe -no-mmad -cuda fusion=1,offload-batch-size=1,mmq-id-size=1000 -c 512 --override-kv glm4moe.expert_used_count=int:7`

I obtained: PPL over 565 chunks for n_ctx=512 = 4.6520 +/- 0.02855

The final PPL bump is 1/20th of what it was, the variance on individual values are 1/10th of what they were, and we're back in the realm of an "acceptable" trade-off for the performance boost.

With FMOE activated :

Final estimate: PPL over 565 chunks for n_ctx=512 = 4.6520 +/- 0.02855 (no variance, I had one yesterday without MMQ_ID).

With MMAD activated :

Final estimate: PPL over 565 chunks for n_ctx=512 = 4.6513 +/- 0.02855 (same final PPL than without MMQ_ID with the same options, but the intermediary variances are higher, especially in the beginning of the test (+0.02 PPL, very much like before the proposed MMQ_ID fix)).

MMAD repeatedly causes variances no matter which other options are chosen. I'll deactivate it for now on my side, and also, I got a PP speed drop of 1.5% during my perplexity test with MMAD on. I would suggest you to review PR [#858](https://github.com/ikawrakow/ik_llama.cpp/issues/858) again.

---

👤 **ikawrakow** commented on **2025-11-07** at **17:03:10**

> but the intermediary variances are higher, especially in the beginning of the test

You mean they differ from one PPL run to another PPL run with the same parameters, or they differ from the run without the MMAD fusion? If the latter, this is very normal. If the former, I cannot reproduce. There is a data race in the `add + fused_rms_norm` fusion, but this should be solved after [#916](https://github.com/ikawrakow/ik_llama.cpp/issues/916) has been merged.

---

👤 **Nexesenex** commented on **2025-11-07** at **19:18:30**

I'm sorry, I wondered afterwards if I was ambiguous, and I was.

The latter, of course. With MMQ_ID fixed and activated, MMAD modifies quite a bit the intermediary values of the PPL test, even if we end up with a final PPL similar to no-mmad.

---

👤 **JohannesGaessler** commented on **2025-11-07** at **22:25:24**

Again thank you for reporting this issue, it was undetected on the main repository for months. Fix here: https://github.com/ggml-org/llama.cpp/pull/17089 . 

Rounding up the number of SMs is only a complete fix for a subset of tensor shapes (they need to be sufficiently large); I use it for debugging but I would strongly advise you to take over the PR I linked since there is simply an index in the stream-k fixup that is being calculated correctly.