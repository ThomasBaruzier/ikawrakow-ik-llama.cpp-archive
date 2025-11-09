## 🔀 [Pull Request #672](https://github.com/ikawrakow/ik_llama.cpp/pull/672) - Port cpu moe options from mainline

| **Author** | `TheLegendOfKitty` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `port-cpu-moe-options` |
| **Target Branch** | `main` |
| **Created** | 2025-08-06 |
| **Updated** | 2025-08-08 |
| **Merged** | 2025-08-08 |

---

## 📄 Description

- [x] I have read the [contributing guidelines](https://github.com/ggerganov/llama.cpp/blob/master/CONTRIBUTING.md)
- Self-reported review complexity:
  - [x] Low
  - [ ] Medium
  - [ ] High

Simple port of https://github.com/ggml-org/llama.cpp/pull/15077.

---

## 💬 Conversation

👤 **ikawrakow** commented on **2025-08-07** at **05:31:38**

Do people think that this really useful?

`-cmoe` is the same as `-ot exps=CPU`, and now we have `-fmoe` and `-cmoe`.

The regex required with `-ot` for `-ncmoe X` is slightly more complicated, but it isn't clear to me why keeping the first `X` routed MoE tensors on the CPU is better than any alternative option of having `X` layers on the CPU.

---

👤 **enbeec** commented on **2025-08-07** at **07:59:08**

> Do people think that this really useful?

I did before I looked at mainline and realized it's just what I'm doing in bash, while wondering if it's even any faster than some alternatives.

> The regex required with -ot for -ncmoe X is slightly more complicated, but it isn't clear to me why keeping the first X routed MoE tensors on the CPU is better than any alternative option of having X layers on the CPU.

Yeah, speaking as a noob here but with some user-facing experience as a dev, this feels like solving a papercut by potentially misleading people. Speaking as someone who has been on the internet, I get the feeling we're looking at something like blogspam making one piece of isolated advice into a pseudo best practice. The most credible source I can find for this exact advice are Unsloth guides for big MoE models and [this Reddit post](https://www.reddit.com/r/LocalLLaMA/comments/1ki7tg7/dont_offload_gguf_layers_offload_tensors_200_gen/).

As I understand it with my beginner mind, it's a combo of `FFN on CPU is fast` and `prefer to offload routed tensors to reduce PCIe chatter, probably` with the caveat `splitting layers increases PCIe chatter drastically doing roundtrips mid-layer`. Like this [top level comment on that Reddit post](https://www.reddit.com/r/LocalLLaMA/comments/1ki7tg7/comment/mrd6xjw/) points out, this will work best when the CPU is heavily bottlenecked. The last thing I've seen is the people that do statistical analysis on activations but that seems like a lot, to me at least.

I do think we're going to see a lot of users like me trying to cram fairly sizeable MoEs into modest-to-meager hardware. I've got a 3090 on PCIe gen3, ~192GB DDR4 2666MHz and 16 threads dedicated to inference. Until I can afford to upgrade the host and leapfrog the GPU this a topic that really matters to me.

If someone with deeper technical knowledge can help fill in some of the details about the tradeoffs of whole-layer vs. expert-tensor offload, I can happily test on my setup and share results on Reddit to get some evidence-based thinking going out there. I also am fuzzy on *exactly* why I'm told to offload from GPU to CPU starting with the last layers and working forward -- prompt processing? I think there is a need for "hybrid MoE strategies" and it would be a good backdoor for getting people to think beyond VRAM+RAM+TC.

Also, I know there are people who have done statistical analysis of activations to guide offloading choices but that seems *heavily* workload dependent. I think any efforts on this front should be leading towards teaching someone how to read the architecture diagram and deduce which layers/tensors belong where **on their hardware**.

---

👤 **ikawrakow** approved this pull request ✅ on **2025-08-08** at **11:38:11**

Given the 3 :rocket: , I'll merge.