## 🔀 [Pull Request #833](https://github.com/ikawrakow/ik_llama.cpp/pull/833) - Adding Ling/Ring (a.k.a., Bailing-MoE2) support

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/bailingmoe2` |
| **Target Branch** | `main` |
| **Created** | 2025-10-14 |
| **Updated** | 2025-10-16 |
| **Merged** | 2025-10-15 |

---

## 📄 Description

Derived from https://github.com/ggml-org/llama.cpp/pull/16063/files

Did not add the expert grouping thing yet, as it is not working in the mainline PR (I get a crash when I try to run it).

Nevertheless, based on a very rudimentary testing it seems to be working OK. Hence publishing it for testing by others.

~Did not yet add the necessary changes to `convert_hf_to_gguf.py`, so model conversion needs to be done with the mainline PR for now.~
The conversion scripts seems to be working fine, at least with the [model](https://huggingface.co/inclusionAI/Ling-mini-2.0) I tested with.

Here a quick `llama-sweep-bench` result using [this Q4_K_M quantized model](https://huggingface.co/inclusionAI/Ling-mini-2.0-GGUF/blob/main/Ling-mini-2.0-Q4_K_M.gguf) (RTX-4080, `-c 32768 -ub 2048 -t 1 -ngl 100 -fa -fmoe`)

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  2048 |    512 |      0 |    0.149 | 13783.36 |    1.280 |   400.03 |
|  2048 |    512 |   2048 |    0.122 | 16737.36 |    1.339 |   382.37 |
|  2048 |    512 |   4096 |    0.128 | 15956.24 |    1.419 |   360.80 |
|  2048 |    512 |   6144 |    0.135 | 15204.72 |    1.499 |   341.47 |
|  2048 |    512 |   8192 |    0.141 | 14496.75 |    1.562 |   327.73 |
|  2048 |    512 |  10240 |    0.148 | 13817.95 |    1.651 |   310.15 |
|  2048 |    512 |  12288 |    0.155 | 13198.26 |    1.700 |   301.12 |
|  2048 |    512 |  14336 |    0.162 | 12646.19 |    1.780 |   287.61 |
|  2048 |    512 |  16384 |    0.168 | 12218.99 |    1.838 |   278.50 |
|  2048 |    512 |  18432 |    0.175 | 11690.43 |    1.904 |   268.92 |
|  2048 |    512 |  20480 |    0.181 | 11331.88 |    1.985 |   257.96 |
|  2048 |    512 |  22528 |    0.188 | 10881.69 |    2.043 |   250.64 |
|  2048 |    512 |  24576 |    0.195 | 10483.32 |    2.134 |   239.97 |
|  2048 |    512 |  26624 |    0.204 | 10061.95 |    2.186 |   234.26 |
|  2048 |    512 |  28672 |    0.209 |  9803.12 |    2.248 |   227.71 |
|  2048 |    512 |  30720 |    0.214 |  9577.52 |    2.325 |   220.23 |

Closes [#813](https://github.com/ikawrakow/ik_llama.cpp/issues/813)

---

## 💬 Conversation

👤 **ikawrakow** commented on **2025-10-14** at **16:28:54**

Haha, the [Ling-mini-2.0 model](https://huggingface.co/inclusionAI/Ling-mini-2.0) that I'm using for testing seems to be yet another one of those model  where `bf16` perplexity is higher than PPL calculated with a quantized model:
* PPL(bf16) =  13.5528
* PPL(Q4_K_M) = 13.4201
* PPL(Q4_0) = 13.4292
* PPL(Q3_K_S) = 12.9360 (!!!)

This is without even using an imatrix.

---

👤 **kirnat** commented on **2025-10-14** at **16:35:32**

May I ask, do you have any idea why this is? In this case the difference is quite substantial. Would this signal an issue with the quality of the model in general or is it just pure coincidence?

---

👤 **ikawrakow** commented on **2025-10-14** at **16:40:24**

> May I ask, do you have any idea why this is? 

Not sure. It could be many things. But in this specific case I would wait for the grouped experts routing to be implemented correctly before drawing any conclusions.

---

👤 **CISC** commented on **2025-10-14** at **18:10:04**

> Did not add the expert grouping thing yet, as it is not working in the mainline PR (I get a crash when I try to run it).

It should not crash, it works for me on CUDA and CPU, do you have more details?

---

👤 **ikawrakow** commented on **2025-10-14** at **19:17:31**

@CISC I have answered your question [here](https://github.com/ggml-org/llama.cpp/pull/16063#issuecomment-3403241697)

---

👤 **ikawrakow** commented on **2025-10-15** at **06:31:30**

@CISC 

OK, it no longer crashes after your fix. But now Wikitext perplexity is much higher:
* PPL with grouped experts routing:  34.9221
* PPL with standard experts touting:  13.4201

Is this expected?

---

👤 **CISC** commented on **2025-10-15** at **07:34:02**

> @CISC
> 
> OK, it no longer crashes after your fix. But now Wikitext perplexity is much higher:
> 
> * PPL with grouped experts routing:  34.9221
> * PPL with standard experts touting:  13.4201
> 
> Is this expected?

Yes, I saw that too, and I think so, basically what group selection does is remove half the experts. The experts are divided into 8 groups and only 4 of them are used based on the highest scores within each group. I don't know how the model is trained, but I imagine each group has some sort of specialization, so due to the somewhat random nature of wikitext a higher PPL is to be expected.

In case anyone wonders, the current group selection implementation selects groups based on a single token within each group, while the disabled one (due to inefficiency (only CPU support)) selects groups based on the sum of two (like the original implementation). Both implementations yield a similarly higher PPL.

---

👤 **ikawrakow** commented on **2025-10-15** at **09:39:34**

> Yes, I saw that too, and I think so, basically what group selection does is remove half the experts. The experts are divided into 8 groups and only 4 of them are used based on the highest scores within each group. I don't know how the model is trained, but I imagine each group has some sort of specialization, so due to the somewhat random nature of wikitext a higher PPL is to be expected.

What if we took something less random? Say, [Pride and Prejudice](https://www.gutenberg.org/ebooks/1342.txt.utf-8) from project Gutenberg. We get
* PPL (standard routing, 8 experts): 31.7274
* PPL (standard routing, 4 experts): 43.6031  (using `--override-kv bailingmoe2.expert_used_count=int:4)
* PPL (grouped experts routing)     : 48.6346

If we did have the situation where "each group has some sort of specialization", one would expect that no relevant groups get discarded when reading a well known novel (that with almost 100% probability has been in the training data), so the grouped expert routing shouldn't be having a 50% higher PPL, and it shouldn't be higher compared to just using the top 4  experts with standard routing.

For comparison, LlaMA-3.1-8B-Instruct gets PPL = 2.7745 on Pride and Prejudice.

---

👤 **CISC** commented on **2025-10-15** at **09:52:27**

> What if we took something less random? Say, [Pride and Prejudice](https://www.gutenberg.org/ebooks/1342.txt.utf-8) from project Gutenberg. We get
> 
> * PPL (standard routing, 8 experts): 31.7274
> * PPL (standard routing, 4 experts): 43.6031  (using `--override-kv bailingmoe2.expert_used_count=int:4)
> * PPL (grouped experts routing)     : 48.6346
> 
> If we did have the situation where "each group has some sort of specialization", one would expect that no relevant groups get discarded when reading a well known novel (that with almost 100% probability has been in the training data), so the grouped expert routing shouldn't be having a 50% higher PPL, and it shouldn't be higher compared to just using the top 4 experts with standard routing.

True, but a PPL of ~32 with standard routing is extremely high to begin with, I would say this model does not have it in its training data...

---

👤 **ikawrakow** commented on **2025-10-15** at **11:19:30**

OK, I'll merge this PR without enabling grouped expert routing.

As far as I can tell, the grouping does not improve inference (if anything, it makes matters worse). It also results in a significant performance degradation.

If we get reports that `llama.cpp` (where grouped experts routing may be enabled when the BailingMoeV2 PR is merged) works better than `ik_llama.cpp`, I'll look into implementing it in a more efficient way.

---

👤 **ikawrakow** commented on **2025-10-15** at **16:01:11**

@CISC 

Here some experimental results of Wikitext perplexity computed with different u-batch sizes.

| u-batch | PPL (grouped) | diff (grouped) | PPL (standard) | diff (standard) |
| ---: | ---: | ---: | ---: | ---: |
|   128 | 31.1416 | -3.68% |  13.3810 |  -0.14% |
|  256  | 32.1615 | -0.52% |  13.3813 |  -0.14% |
|  512  | 31.1469 | -3.66% |  13.3957 |  -0.03% |
| 1024  | 32.0534|  -0.86%|   13.4281|   +0.21% |
| 2048  | 34.9221|  +8.01% |  13.4016 |  +0.02% |
| 4096  | 32.5617|  +0.71% |  13.4092  | +0.07% |

The "diff" column shows the difference to the average of the 6 results in the respective experts routing type. Differences in the range of 0.1% between different parameters / hardware / etc. are normal, and this is what we see for the standard routing approach. For the grouped routing we have results varying wildly, and well beyond the expected 0.1 range.

Looking at the code this is kind of expected, considering that the selected experts for each token will depend on the tokens present in the batch. But perhaps I'm misunderstanding the implementation?

Can you explain in plain English what you are trying to accomplish? Thanks.

---

👤 **CISC** commented on **2025-10-15** at **19:02:07**

> Looking at the code this is kind of expected, considering that the selected experts for each token will depend on the tokens present in the batch. But perhaps I'm misunderstanding the implementation?

That's correct.

> Can you explain in plain English what you are trying to accomplish? Thanks.

I'm merely reproducing the original implementation:
https://huggingface.co/inclusionAI/Ling-1T/blob/99177e9391daf85b6c32aac2b5f4486365db14af/modeling_bailing_moe_v2.py#L315-L329

All the experts are "grouped" into 8 groups and each group are then scored and the 4 lowest scoring groups are masked out (setting the tokens to -inf) before proceeding to the normal topk selection.

---

👤 **ikawrakow** commented on **2025-10-16** at **06:09:05**

> I'm merely reproducing the original implementation:

As far as I can tell, the code in your `llama.cpp` PR doesn't do what the code you linked above does. Neither the enabled version nor the disabled top-2 version. That's why I asked for plain English explanation rather than a link to a piece of code.

---

👤 **CISC** commented on **2025-10-16** at **06:53:51**

> > I'm merely reproducing the original implementation:
> 
> As far as I can tell, the code in your `llama.cpp` PR doesn't do what the code you linked above does. Neither the enabled version nor the disabled top-2 version. That's why I asked for plain English explanation rather than a link to a piece of code.

Ok, that was my intention at least, I might have done something wrong of course. My understanding of what the original (and my) code does is what I wrote in plain English though.