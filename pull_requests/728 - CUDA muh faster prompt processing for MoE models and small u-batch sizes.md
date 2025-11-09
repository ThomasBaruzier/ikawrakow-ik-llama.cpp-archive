## 🔀 [Pull Request #728](https://github.com/ikawrakow/ik_llama.cpp/pull/728) - CUDA: muh faster prompt processing for MoE models and small u-batch sizes

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/add_mmq_id` |
| **Target Branch** | `main` |
| **Created** | 2025-08-25 |
| **Updated** | 2025-11-07 |
| **Merged** | 2025-08-26 |

---

## 📄 Description

`ik_llama.cpp` has excellent CUDA prompt processing (PP) performance for MoE models when using large batch and u-batch sizes (>= 2048). But for smaller batch sizes PP performance rapidly decreases with decreasing u-batch size. This is not desirable as larger u-batches result in larger CUDA compute buffers, which reduces the maximum context length one can use and/or the number of MoE layers one can offload to the GPU.

This PR remedies the situation by bringing massive performance improvement for small u-batch sizes.

For this PR I'm standing on the shoulder of giants. Actually, it is a single giant, Johannes Gaessler. The PR derived from his [PR 15525](https://github.com/ggml-org/llama.cpp/pull/15525)  in mainline `llama.cpp`. But, as it is always the case these days, it required quite a bit of adaptation. All additional `ik_llama.cpp` quants needed to be added as well to the new kernels, which of course required to get rid of the almighty assumption in mainline code that all quantized tensor data consist of blocks of a fixed size arranged contiguously in memory.

At this point it is a bit of an append-only programming style, but I'll clean up in subsequent PRs.

Here is a performance comparison between the main branch and this PR for DeepSeek-Lite quantized with `Q4_0` as a function of u-batch size for a fixed batch size of 4096 on Linux. GPU is RTX-4080. Flash attention is on, `mla = 3, fmoe = 1`.  

| model                | n_batch | n_ubatch |          test |      t/s (main)  |      t/s (PR)    |  Speedup |
| -------------------- | ------: | -------: | ------------: | ---------------: | ---------------: | -------: |
| deepseek2 16B Q4_0   |    4096 |      128 |        pp4096 |  1783.77 ± 54.27 |  3088.74 ± 51.34 |  1.732   |
| deepseek2 16B Q4_0   |    4096 |      256 |        pp4096 |  2934.71 ± 19.21 |  5222.70 ± 29.80 |  1.780   |
| deepseek2 16B Q4_0   |    4096 |      512 |        pp4096 |  4701.68 ± 44.76 |  7626.49 ± 62.89 |  1.622   |
| deepseek2 16B Q4_0   |    4096 |     1024 |        pp4096 |  6996.28 ± 64.63 |  9704.90 ± 94.28 |  1.387   |
| deepseek2 16B Q4_0   |    4096 |     2048 |        pp4096 |  9107.72 ± 37.01 | 10395.52 ± 101.25|  1.144   |
| deepseek2 16B Q4_0   |    4096 |     4096 |        pp4096 | 10256.00 ± 14.23 | 10265.86 ± 13.76 |  1.000   |

The PR massively reduces the number of kernel launches during prompt processing. Hence, I wouldn't be surprised if on Windows, where kernel launches are much more expensive, the performance impact is much higher than on Linux. 

Worth noting that the new MoE implementation becomes less efficient than the existing implementation somewhere around u-batch size of 2000 tokens. Because of that, `ik_llama.cpp` switches to the old MoE implementation for `u-batch > 2048`. I expect that the threshold where the old implementation is better will be model dependent (most likely dependent on ration between total and active experts), but haven't studied this in detail, so left for a future tuning.

---

## 💬 Conversation

👤 **DocShotgun** commented on **2025-08-26** at **05:34:56**

Very interesting! I did encounter slow prompt processing performance for GLM 4.5 Air on pure CUDA at `-ub 512` previously, so I'm curious how much this helps.

I'm having some trouble getting this PR to compile on Windows. It seems to be encountering the same error building multiple mmq instance files:
```
C:\ML\ik_llama.cpp\ggml\src\ggml-cuda\template-instances\../mmq_id_common.cuh(240): error : identifier "__nv_cvt_e8m0_t
o_bf16raw" is undefined [C:\ML\ik_llama.cpp\build\ggml\src\ggml.vcxproj]
        const nv_bfloat16 e = __nv_cvt_e8m0_to_bf16raw(x);
                              ^
```

---

👤 **ikawrakow** commented on **2025-08-26** at **05:45:12**

@DocShotgun Thanks! There was the `fp8` header include missing. Can you try now?

---

👤 **DocShotgun** commented on **2025-08-26** at **06:34:06**

> @DocShotgun Thanks! There was the `fp8` header include missing. Can you try now?

Built successfully now!

Here are some preliminary numbers on Windows 11 with RTX PRO 6000 96gb on GLM-4.5-Air-Q5_K_S. Ran the same tests that you did above. It's a HUGE speedup on Windows lol.

ik_llama.cpp PR vs ik_llama.cpp main `50f7119dfd15325c6889927499e4233f7453cb44`:

`./llama-bench -m "C:\ML\GGUF\GLM-4.5-Air-Q5_K_S.gguf" -ngl 999 -fa 1 -fmoe 1 --mmap 0 -t 1 -n 0 -p 4096 -b 4096 -ub 128,256,512,1024,2048,4096`

| model                          | n_batch | n_ubatch |          test |       t/s (main) |         t/s (PR) |          Speedup |
| ------------------------------ | ------: | -------: | ------------: | ---------------: | ---------------: | ---------------: |
| glm4moe 106B.A12B Q5_K - Small |    4096 |      128 |        pp4096 |    395.70 ± 2.04 |    988.92 ± 6.59 |            2.499 |
| glm4moe 106B.A12B Q5_K - Small |    4096 |      256 |        pp4096 |    665.76 ± 4.57 |   1619.13 ± 9.21 |            2.432 |
| glm4moe 106B.A12B Q5_K - Small |    4096 |      512 |        pp4096 |   1090.70 ± 6.14 |   2421.81 ± 9.80 |            2.220 |
| glm4moe 106B.A12B Q5_K - Small |    4096 |     1024 |        pp4096 |   1716.40 ± 5.39 |  3181.34 ± 15.91 |            1.853 |
| glm4moe 106B.A12B Q5_K - Small |    4096 |     2048 |        pp4096 |   2464.98 ± 6.85 |   3630.40 ± 3.02 |            1.473 |
| glm4moe 106B.A12B Q5_K - Small |    4096 |     4096 |        pp4096 |   3261.20 ± 8.64 |  3244.31 ± 17.58 |            0.995 |

I compared it to current mainline llama.cpp as well (with Johannes's original PR merged).

ik_llama.cpp PR vs llama.cpp main `34bdbbd7c2b70b848718e95bc48010f6aecd2816`:

`./llama-bench -m "C:\ML\GGUF\GLM-4.5-Air-Q5_K_S.gguf" -ngl 999 -fa 1 -fmoe 1 --mmap 0 -t 1 -n 0 -p 4096 -b 4096 -ub 128,256,512,1024,2048,4096`
(same thing for main llama.cpp but with no `-fmoe`)

| model                          | n_batch | n_ubatch |          test |  t/s (lcpp main) |         t/s (PR) |          Speedup |
| ------------------------------ | ------: | -------: | ------------: | ---------------: | ---------------: | ---------------: |
| glm4moe 106B.A12B Q5_K - Small |    4096 |      128 |        pp4096 |   1019.57 ± 4.95 |    988.92 ± 6.59 |            0.970 |
| glm4moe 106B.A12B Q5_K - Small |    4096 |      256 |        pp4096 |   1651.01 ± 4.99 |   1619.13 ± 9.21 |            0.981 |
| glm4moe 106B.A12B Q5_K - Small |    4096 |      512 |        pp4096 |   2458.26 ± 3.79 |   2421.81 ± 9.80 |            0.985 |
| glm4moe 106B.A12B Q5_K - Small |    4096 |     1024 |        pp4096 |   3257.00 ± 1.83 |  3181.34 ± 15.91 |            0.977 |
| glm4moe 106B.A12B Q5_K - Small |    4096 |     2048 |        pp4096 |   3792.66 ± 5.86 |   3630.40 ± 3.02 |            0.957 |
| glm4moe 106B.A12B Q5_K - Small |    4096 |     4096 |        pp4096 |  3926.86 ± 15.30 |  3244.31 ± 17.58 |            0.826 |

---

👤 **Ph0rk0z** commented on **2025-08-26** at **11:34:48**

No real changes for hybrid inference. Something made PP fall when I went from batch 1024 to 512, previously it was over 100 on qwen-235b and now it's 66, but I don't think that it was this commit.

---

👤 **usrlocalben** commented on **2025-08-26** at **13:31:21**

> No real changes for hybrid inference. Something made PP fall when I went from batch 1024 to 512, previously it was over 100 on qwen-235b and now it's 66, but I don't think that it was this commit.

I _think_ something happened in the CUDA graphs PR, but I have not taken the time to measure commits yet and find it.

---

👤 **Ph0rk0z** commented on **2025-08-26** at **14:47:23**

I may also be mistaking qwen 512 perf for GLM perf. I tried rolling back to https://github.com/ikawrakow/ik_llama.cpp/commit/23fe18ce83237879dc5e55c444de560f0ed736d5 with no change. I'll try to roll back to before cuda graphs later today.

---

👤 **ikawrakow** commented on **2025-08-26** at **16:09:09**

Because @DocShotgun brought up a comparison with mainline `llama.cpp`: for the MoE models I can run fully offloaded to my 16 GB RTX-4080 GPU, `ik_llama.cpp` is still faster (but yes, Johannes has been able to close the gap quite a bit since April, when the `llama.cpp` performance improvement initiative kicked in). Here is what I get for 3 different models:

  | model                | n_ubatch |            test |    t/s (llama.cpp)   |  t/s (ik_llama.cpp)  |  Speedup |
| -------------------- | -------: | --------------: | -------------------: | -------------------: | -------: |
| deepseek2 16B Q4_0   |      256 |          pp4096 |       5379.33 ± 7.51 |    5192.36 ± 116.79  |  0.965   |
| deepseek2 16B Q4_0   |      512 |          pp4096 |       7268.16 ± 5.54 |    7458.49 ± 144.03  |  1.026   |
| deepseek2 16B Q4_0   |     1024 |          pp4096 |      8463.82 ± 15.63 |    9635.03 ± 149.52  |  1.138   |
| deepseek2 16B Q4_0   |     2048 |          pp4096 |      8372.37 ± 23.71 |    10363.95 ± 140.65 |  1.238   |
| deepseek2 16B Q4_0   |     4096 |          pp4096 |      7471.83 ± 10.94 |    10257.31 ± 48.72  |  1.373   |
| qwen3moe 30B.A3B IQ2_XXS |      256 |          pp4096 |      3758.21 ± 67.68 |  3990.02 ± 18.09 |  1.061   |
| qwen3moe 30B.A3B IQ2_XXS |      512 |          pp4096 |     5167.30 ± 212.46 |  5576.11 ± 15.25 |  1.079   |
| qwen3moe 30B.A3B IQ2_XXS |     1024 |          pp4096 |     6039.60 ± 231.07 |   6518.85 ± 8.76 |  1.079   |
| qwen3moe 30B.A3B IQ2_XXS |     2048 |          pp4096 |      6299.04 ± 24.53 |  6587.33 ± 13.69 |  1.046   |
| qwen3moe 30B.A3B IQ2_XXS |     4096 |          pp4096 |       5846.55 ± 5.21 | 6082.93 ± 190.64 |  1.040   |
| gpt-oss 20B MXFP4 MoE    |      256 |          pp3072 |      4881.85 ± 23.06 |  5474.21 ± 35.55 |  1.119   |
| gpt-oss 20B MXFP4 MoE    |      512 |          pp3072 |       6573.29 ± 8.64 |  7282.90 ± 26.87 |  1.108   |
| gpt-oss 20B MXFP4 MoE    |     1024 |          pp3072 |      7512.86 ± 20.92 |  8266.53 ± 31.21 |  1.100   |
| gpt-oss 20B MXFP4 MoE    |     1536 |          pp3072 |      7562.65 ± 23.94 |  8878.06 ± 42.00 |  1.174   |
| gpt-oss 20B MXFP4 MoE    |     3072 |          pp3072 |      7200.01 ± 11.84 |  9364.51 ± 29.43 |  1.301   |

---

👤 **ikawrakow** commented on **2025-08-26** at **16:16:23**

> No real changes for hybrid inference. 

Of course not. For hybrid inference such small batches may not even be offloaded to the GPU (batch size must be greater than `32 * num_total_expert / num_active_experts` for that to happen). But even if they do get offloaded, for such small batch sizes total processing time is totally dominated by the time it takes to copy the tensors to VRAM. For large batches, where data transfer time is a smaller fraction of the overall time, the improvement is much less (and it eventually completely goes away), so performance improvements will be none to very small.

> Something made PP fall when I went from batch 1024 to 512,
>  may also be mistaking qwen 512 perf for GLM perf

Yes, it can be very useful to take notes, so one does not live with the concept that something got broken along the way.

---

👤 **DocShotgun** commented on **2025-08-26** at **19:40:44**

> Because @DocShotgun brought up a comparison with mainline `llama.cpp`: for the MoE models I can run fully offloaded to my 16 GB RTX-4080 GPU, `ik_llama.cpp` is still faster (but yes, Johannes has been able to close the gap quite a bit since April, when the `llama.cpp` performance improvement initiative kicked in). Here is what I get for 3 different models:

Do you think mainline `llama.cpp` still has some other Windows-specific optimization that hasn't been ported over yet?

---

👤 **Ph0rk0z** commented on **2025-08-27** at **11:56:08**

Funny enough, I keep notes with benchmarks but the file is getting unwieldy. Got a small 512 bump from `32 * num_total_expert / num_active_experts` I'll have to clear the cache to get a proper reading on 1024 as loading different models in/out of memory has a detrimental effect after a while.

---

👤 **Nexesenex** commented on **2025-11-06** at **02:06:23**

This PR might have provoked a perplexity bump on GLM 4.5 Air. I inquried about a perplexity bump on the recent main, as I was trying your commit allowing to quantize ffn_gate_imp, the PPL was higher than expected, including on the source model (the following quant in my 'expose').

I rolled back from main to isolate the problem on PR 728. OS used, Win 11. Full Cuda offload on 3 Ampere GPUs.

```
Q:\LLAMA_IK_GLM2>llama-perplexity -m X:\text-generation-webui\models\zai-org_GLM-4.5-Air-bf16-iMat-IQ4_XS_L-00001-of-00005.gguf -f wiki.test.raw --parallel 1 -ngl 150 -b 512 -mg 0 -ts 18,18,12 --no-mmap -fa --override-kv glm4moe.expert_used_count=int:7

llama_model_loader: - type  f32:  331 tensors
llama_model_loader: - type q4_K:    3 tensors
llama_model_loader: - type q5_K:   93 tensors
llama_model_loader: - type q6_K:   53 tensors
llama_model_loader: - type iq4_nl:   45 tensors
llama_model_loader: - type iq4_xs:  136 tensors
llama_model_loader: - type q6_0:  142 tensors
```

Before this PR 728 :

Final estimate: PPL = 4.6508 +/- 0.02854 / orig before Muh PR.

```
llama_print_timings:        load time =   29100.68 ms
llama_print_timings:      sample time =       0.00 ms /     1 runs   (    0.00 ms per token,      inf tokens per second)
llama_print_timings: prompt eval time =  650424.94 ms / 289280 tokens (    2.25 ms per token,   444.76 tokens per second)
llama_print_timings:        eval time =       0.00 ms /     1 runs   (    0.00 ms per token,      inf tokens per second)
llama_print_timings:       total time =  659193.06 ms / 289281 tokens
```

After this PR 728 :

Final estimate: PPL = 4.6772 +/- 0.02874  / after Muh PR.

```
llama_print_timings:        load time =   29011.64 ms
llama_print_timings:      sample time =       0.00 ms /     1 runs   (    0.00 ms per token,      inf tokens per second)
llama_print_timings: prompt eval time =  456817.57 ms / 289280 tokens (    1.58 ms per token,   633.25 tokens per second)
llama_print_timings:        eval time =       0.00 ms /     1 runs   (    0.00 ms per token,      inf tokens per second)
llama_print_timings:       total time =  465547.22 ms / 289281 tokens
```

I made a compromise for myself between speed and quality by shelving 12 PRs, making a clean rollback.

Here's the modified branch :

https://github.com/Nexesenex/ik_llama.cpp.nxs/tree/Rolled-back-to-pre-PR728


final estimate: PPL = 4.6544 +/- 0.02857 / branch rolled back from main until pre-PR 728.

```
llama_print_timings:        load time =   29011.65 ms
llama_print_timings:      sample time =       0.00 ms /     1 runs   (    0.00 ms per token,      inf tokens per second)
llama_print_timings: prompt eval time =  513583.20 ms / 289280 tokens (    1.78 ms per token,   563.26 tokens per second)
llama_print_timings:        eval time =       0.00 ms /     1 runs   (    0.00 ms per token,      inf tokens per second)
llama_print_timings:       total time =  522307.88 ms / 289281 tokens
```

Disabling MMAD on this branche helps quality furthermore:

Final estimate: PPL = 4.6511 +/- 0.02854 / branch rolled back from main until pre-PR 728. -no-mmad

```
llama_print_timings:        load time =   29681.95 ms
llama_print_timings:      sample time =       0.00 ms /     1 runs   (    0.00 ms per token,      inf tokens per second)
llama_print_timings: prompt eval time =  530091.86 ms / 289280 tokens (    1.83 ms per token,   545.72 tokens per second)
llama_print_timings:        eval time =       0.00 ms /     1 runs   (    0.00 ms per token,      inf tokens per second)
llama_print_timings:       total time =  538881.31 ms / 289281 tokens
```

---

👤 **ikawrakow** commented on **2025-11-06** at **06:26:42**

@Nexesenex

What PPL do we get with mainline `llama.cpp`? Or is this a mix that cannot be computed with `llama.cpp`?

Btw, you don't need to roll back 12 PR and all that. To arrive at the behavior prior to this PR, all you need to do is change https://github.com/ikawrakow/ik_llama.cpp/blob/575e2c28216243227cb19d137f69e4f018b672b7/ggml/src/ggml-cuda.cu#L2688

to
```c++
    if (false && src1->ne[2] <= 32*src0->ne[2]
```

---

👤 **ikawrakow** commented on **2025-11-06** at **10:59:00**

OK, I can confirm the issue exists in mainline `llama.cpp` as well. Here is a table with perplexities for a few models. PPL has been computed with today's `llama.cpp` build (`build: 6967 (b7f9010d2)`). PPL* is also with today's build, but disabling the effect of mainline [PR 15525](https://github.com/ggml-org/llama.cpp/pull/15525), on which this PR is based, by changing
```c++
        if (ggml_cuda_should_use_mmq(src0->type, cc, ne12)) {
            ggml_cuda_mul_mat_q(ctx, src0, src1, ids, dst);
            return;
        }
```
on lines 2285-2288 in `ggml-cuda.cu` to
```c++
        if (false && ggml_cuda_should_use_mmq(src0->type, cc, ne12)) {
            ggml_cuda_mul_mat_q(ctx, src0, src1, ids, dst);
            return;
        }
```

| Model | PPL | PPL* | PPL vs PPL*  |
| ---: | ---: | ---: | ---: |
| GLM-4.5-Air-UD-IQ2_XXS |  5.5193 | 5.4985 | +0.38% |
| Ling-mini-2.0-Q4_K_M | 13.6225 | 13.5614 | +0.45% |
| Qwe3-30B-A3B IQ2_XXS | 13.5195 | 13.4481 | +0.53% |
| DeepSeek-Lite Q4_0 | 7.1254  | 7.0966 | +0.41% |

The PPL increase appears to be systematic. Differences of 0.4% are well beyond the typical range caused by numerical round off. Hence, it is likely that there is an issue in mainline PR 15525, so pinging @JohannesGaessler as the author of 15525.

---

👤 **Nexesenex** commented on **2025-11-06** at **11:26:46**

> Btw, you don't need to roll back 12 PR and all that. To arrive at the behavior prior to this PR, all you need to do is change
> 
> https://github.com/ikawrakow/ik_llama.cpp/blob/575e2c28216243227cb19d137f69e4f018b672b7/ggml/src/ggml-cuda.cu#L2688
> 
> to
> 
> ```c++
>     if (false && src1->ne[2] <= 32*src0->ne[2]
> ```

I suspected that PR first, and saw that line, but then I tried to undo the commit and saw conflicts, so I concluded that it could be a more complex issue and that changing a trigger might not cut the deal, so I rolled-back everything to be absolutely certain of myself. I didn't check on mainline, though, I was sleepy. :D

My variances are in the same order of magnitude than yours, and they indeed went way beyond the 0.01% or so which is "acceptable".

Anyway, showtime for the big guys! ^^

---

👤 **ikawrakow** commented on **2025-11-06** at **16:01:34**

@Nexesenex See [#910](https://github.com/ikawrakow/ik_llama.cpp/issues/910). With this, using `-cuda mmq-id-size=0` will undo the effect of this PR.

---

👤 **JohannesGaessler** commented on **2025-11-06** at **22:48:13**

Thank you for notifying me. I am seeing an issue specifically on my RTX 5090, with no other GPU being affected. To be clear, the hardware you were using is pre-Blackwell, right? (Could be the same bug, but manifesting differently depending on SM count.)

---

👤 **ikawrakow** commented on **2025-11-07** at **05:37:38**

@JohannesGaessler 

I'm using RTX-4080. @Nexesenex is using 3090s IIRC.

>  I am seeing an issue specifically on my RTX 5090, with no other GPU being affected.

You mean you computed perplexity on a bunch of MoE models, and the result is the same pre- and post MUL-MAT-ID optimization on all of the many GPUs that you have, except on the 5090?

---

👤 **JohannesGaessler** commented on **2025-11-07** at **06:49:02**

No, I mean that on my RTX 5090 (which I didnt yet have back then) there is a very clear problem with ppl increasing by 50% so I'll debug that first. I think it has to do with the SM count not being a power of 2 which would affect all GPUs in question.

---

👤 **ikawrakow** commented on **2025-11-07** at **07:00:19**

I'm not a GPU guru, but the SM count being a power of 2 is not something I would expect. It is 76 on my 4080, and 82 on the 3090 (which is a pretty popular choice among LLM enthusiasts).

Anyway, I hope you will find the issue. Thanks for looking into it!

---

👤 **JohannesGaessler** commented on **2025-11-07** at **07:06:51**

The reason I suspect it is simply because for my implementations of stream-k decompositions that has historically been a frequent source of bugs. The testing I did was simply GraniteMoe with master (+50% ppl vs. RTX 4090 with 128 SMs), with your patch (fixes ppl), and with a patch that overrides the SM count (from the perspective of ggml) to 32 (also fixes ppl). The issue has persisted since back when I implemented this feature on the upstream repository.