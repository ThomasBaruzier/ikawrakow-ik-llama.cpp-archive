## 🔀 [Pull Request #836](https://github.com/ikawrakow/ik_llama.cpp/pull/836) - Grouped expert routing (CPU only)

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/try_grouped_topk_playing1` |
| **Target Branch** | `main` |
| **Created** | 2025-10-16 |
| **Updated** | 2025-10-17 |
| **Merged** | 2025-10-16 |

---

## 📄 Description

This PR adds grouped experts routing as used by the BailingMoeV2 arch (Ling/Ring models).

It is CPU only, so for now disabled by default. It is enabled via `-ger` or `--grouped-expert-routing`.

Quick testing with Ling-mini-2.0 with full GPU offload shows only 20-30% performance degradation when using grouped expert routing (which runs on the CPU). For larger models and/or hybrid GPU/CPU inference the impact will be even less, so it is possible to try this option even before CUDA support is added.

The implementation in this PR is based on my interpretation of the [original Python implementation](https://huggingface.co/inclusionAI/Ling-1T/blob/99177e9391daf85b6c32aac2b5f4486365db14af/modeling_bailing_moe_v2.py#L315-L329), which clearly differs from @CISC's interpretation in the [llama.cpp BailingMoeV2 PR](https://github.com/ggml-org/llama.cpp/pull/16063).

The following table shows perplexities computed for Wikitext2 and the plain text version of the [Pride and Prejudice novel](https://www.gutenberg.org/ebooks/1342.txt.utf-8) from Project Gutenberg (column P&P in the table).

| routing | Wiki2 | P&P |
| ---: | ---: | ---: |
| standard | 13.4016 | 31.7274 |
| grouped (this PR) | 13.5730 |31.7236 |
| grouped (@CISC PR) | 34.9221 |48.6346 |

Based on this, my guess is that the implementation in this PR is more likely to be correct than the implementation in @CISC's [llama.cpp PR](https://github.com/ggml-org/llama.cpp/pull/16063).

In terms of performance when running CPU-only, grouped expert routing in this PR is about the same or even very slightly better than standard expert top_k routing.

---

## 💬 Conversation

👤 **CISC** commented on **2025-10-16** at **09:59:10**

So, was it my interpretation or implementation that was wrong?

---

👤 **ikawrakow** commented on **2025-10-16** at **10:29:46**

> So, was it my interpretation or implementation that was wrong?

Hard to say.

My implementation does this (in plain English)
* **For each token**, group the `n_expert` probabilities into `n_groups` groups of `n_expert/n_group` elements
* Determine group scores for each group by using the sum of the top-2 elements in the group (i.e., top-2 highest probability experts in the group)
* Select the `n_used_groups` groups with the highest group score
* Apply normal top_k expert selection using only the experts in the selected `n_used_groups`. This can be done either by setting the score of experts not in the selected groups to `-INFINITY`, and then applying normal top_k expert selection (this is done in the Python implementation), or by directly fusing the top_k expert selection with the group selection to take advantage of the fact that we already know which experts participate in the top_k selection (the approach used in this PR).

In your PR you group **all tokens** into groups. There are matrix transpositions and what not. I can't tell if this is so because your interpretation of the Python implementation is different from mine, or if it is simply an incorrect implementation of the above algorithm using the limited capabilities of the underlying `ggml` toolkit.

---

👤 **CISC** commented on **2025-10-16** at **10:51:36**

> In your PR you group **all tokens** into groups. There are matrix transpositions and what not. I can't tell if this is so because your interpretation of the Python implementation is different from mine, or if it is simply an incorrect implementation of the above algorithm using the limited capabilities of the underlying `ggml` toolkit.

Probably a little bit of both. :) So, AFAICT our interpretations are basically the same, it's just that I got things mixed up on the first step (I struggled a bit with this because it wasn't possible to use `top_k` the same way as in the Python code).

Ahwell, thanks for the reality check, I'll look into fixing the implementation when I have time.

---

👤 **ikawrakow** commented on **2025-10-16** at **11:56:54**

@ubergarm In case you are still into quant cooking, the Ling-1T/Ring-1T models are an opportunity to publish GGUFs before they become available for mainline. The PR there is still WIP, while I think the models should be functional in `ik_llama.cpp` after merging this PR. Just make sure to add `-ger` when computing the imatrix.

---

👤 **ubergarm** commented on **2025-10-16** at **19:53:34**

> In case you are still into quant cooking, the Ling-1T/Ring-1T models are an opportunity to publish GGUFs

Thanks! I almost started downloading it a couple days ago, but am having hiccups with huggingface changing their public repo size allowances recently. I just subscribed at $9/mo to `PRO` level and hope that allows me to continue uploading quants as I'm over 26TB already...

Working through it now and comparing `Ling-1T` with `Ling-flash-2.0`:

```
Ling-flash-2.0$ grep moe_shared_expert_intermediate_size *.json
config.json:    "moe_shared_expert_intermediate_size": 1024

Ling-1T$ grep moe_shared_expert_intermediate_size *.json
# nothing
```

Created a small patch PR here: https://github.com/ikawrakow/ik_llama.cpp/pull/837 and will update if anything else comes up along the way. Hopefully I'll be able to upload imatrix dat and quants to huggingface if all goes well. :crossed_fingers:

---

👤 **CISC** commented on **2025-10-16** at **23:48:36**

@ikawrakow I think I got it right now: https://github.com/ggml-org/llama.cpp/commit/bc6c48af11cb1240d7a38a3a8c766c9de2dd8912

---

👤 **ikawrakow** commented on **2025-10-17** at **16:04:25**

> @ikawrakow I think I got it right now: https://github.com/ggml-org/llama.cpp/commit/bc6c48af11cb1240d7a38a3a8c766c9de2dd8912

For some reason when I try to use the same implementation in `ik_llama.cpp` I get an assert in the `ggml_set_rows` op, so I ended up making a dedicated CUDA implementation (PR [#838](https://github.com/ikawrakow/ik_llama.cpp/issues/838)).

---

👤 **CISC** commented on **2025-10-17** at **18:19:03**

> > @ikawrakow I think I got it right now: [ggml-org/llama.cpp@bc6c48a](https://github.com/ggml-org/llama.cpp/commit/bc6c48af11cb1240d7a38a3a8c766c9de2dd8912)
> 
> For some reason when I try to use the same implementation in `ik_llama.cpp` I get an assert in the `ggml_set_rows` op, so I ended up making a dedicated CUDA implementation (PR [#838](https://github.com/ikawrakow/ik_llama.cpp/issues/838)).

Weird, though awesome with the CUDA implementation. I think it's possible to optimize away the masking (and thus set rows) though, I'll give it a go...

---

👤 **CISC** commented on **2025-10-17** at **20:13:14**

> I think it's possible to optimize away the masking (and thus set rows) though, I'll give it a go...

Guess not, doing so skews the ids for mulmat ops later.