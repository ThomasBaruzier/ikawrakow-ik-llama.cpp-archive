## 📌 [Issue #671](https://github.com/ikawrakow/ik_llama.cpp/issues/671) - Feature Request: openPangu-Ultra-MoE-718B support

| **Author** | `Lissanro` |
| :--- | :--- |
| **State** | ✅ **Open** |
| **Created** | 2025-08-05 |
| **Updated** | 2025-09-26 |
| **Labels** | `enhancement`, `help wanted` |

---

## 📄 Description

### Prerequisites

- [x] I am running the latest code. Mention the version if possible as well.
- [x] I carefully followed the [README.md](https://github.com/ggerganov/llama.cpp/blob/master/README.md).
- [x] I searched using keywords relevant to my issue to make sure that I am creating a new issue that is not already open (or closed).
- [x] I reviewed the [Discussions](https://github.com/ggerganov/llama.cpp/discussions), and have a new and useful enhancement to share.

### Feature Description

Recently openPangu was released, it is based on DeepSeek V3 architecture, and is a thinking model:
https://ai.gitcode.com/ascend-tribe/openpangu-ultra-moe-718b-model/blob/main/README_EN.md

But it is not entirely identical according to https://www.reddit.com/r/LocalLLaMA/comments/1mhctvk/comment/n6xmva5/

> Pangu architecture is identical to DeepSeek V3 with the sole exception of greater hidden size (and different tokenizer). But unlike Kimi, they rename the architrecture and parameters:
> 
> attention_q_lora_dim = q_lora_rank
> num_experts_per_tok = n_routed_experts
> num_dense_layers = first_k_dense_replace
> attention_qk_dim = qk_nope_head_dim

### Motivation

It would be great if it is possible to run, quantize and generate imatrix with ik_llama.cpp for openPangu, since ik_llama.cpp gives the best performance for large MoE using CPU+GPU for inference.

### Possible Implementation

_No response_

---

## 💬 Conversation

👤 **saood06** commented on **2025-08-17** at **03:10:28**

Have you actually come across anything specific that makes you want to use this model? I ask because I am also curious, but I don't really see anything that makes it stand out (they do have case studies "covering finance, medicine and general domains" with responses compared to R1 in the paper but they are in Chinese and I feel like translating them to compare them would be unfair to both).

This is also all they say about the dataset they used:
>During the construction of the training dataset, we implement strict data quality control and emphasize
the diversity, complexity, and comprehensiveness of the corpus. For long CoT samples, we introduce
specialized tokens to structurally separate reasoning trajectories from final answers. In post-training stage,
instruction fine-tuning integrates multi-domain core tasks such as general question and answer, text generation,
semantic categorization, code programming, mathematical and logical reasoning, and tool usage to form a
multi-dimensional training space for enhanced generalization. Additionally, we set the ratio of reasoning to
non-reasoning samples to 3:1, further improving reasoning ability.

---

👤 **Lissanro** commented on **2025-08-23** at **02:16:52**

My main hope that it will be just different. Like Kimi K2 is different from DeepSeek V3, for example. While V3 was giving often output similar to R1 0528 (especially for cases with little to no thinking), sometimes even with exact matches. This is why I am at the moment mostly use R1 0528 and K2, depending on if I need thinking, or in case one of them gets stuck or if I just want a different output.

So, my expectation from this model, that it will be different in some useful way, maybe better at some things than others. Even if it is just better at medical knowledge on average, it could be useful on its own, but of course I have to test it first.

It is extremely large download, though, and it was only published at https://ai.gitcode.com/ascend-tribe/openpangu-ultra-moe-718b-model/tree/main in BF16 format (1.4TB). Unfortunately, there are no GGUF quants. I found someone download it for me if I provide a hard drive that can be later sent back to me, so perhaps I will have its weights soon to experiment with (like in a week or two), but not sure how difficult it would be to actually add support for it.

At the moment I only have its files without safetensors: https://dragon.studio/2025/08/openPangu-Ultra-MoE-718B-model.zip - the reason why I share ZIP because the gitcode side seems to require registration to download anything, so in case someone is curious what is inside and would like to inspect config files of the model or README, they can look at the ZIP file.

I also noticed activity at huggingface: https://huggingface.co/FreedomIntelligence/openPangu-Ultra-MoE-718B that was created just yesterday - currently it is empty, but I assume it is in the process of being uploaded. So probably will be available at HuggingFace soon.

---

👤 **saood06** commented on **2025-08-23** at **02:49:22**

> but not sure how difficult it would be to actually add support for it

Well actually making the code changes to me, there is nothing that stands out as complex given that it is so similar architecturally to DeepseekV3, but the main barrier is everything else, the large download, converting, quanting, and then testing. 

>So, my expectation from this model, that it will be different in some useful way, maybe better at some things than others. Even if it is just better at medical knowledge on average, it could be useful on its own, but of course I have to test it first.

I hope it does and ends up worth any time you pour into it (I do see reasons to be optimistic about it). Let me know if you need help doing it.