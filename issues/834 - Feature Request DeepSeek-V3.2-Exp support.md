## 📌 [Issue #834](https://github.com/ikawrakow/ik_llama.cpp/issues/834) - Feature Request: DeepSeek-V3.2-Exp support?

| **Author** | `kkontosis` |
| :--- | :--- |
| **State** | ✅ **Open** |
| **Created** | 2025-10-15 |
| **Updated** | 2025-10-15 |
| **Labels** | `enhancement` |

---

## 📄 Description

### Prerequisites

- [x] I am running the latest code. Mention the version if possible as well.
- [x] I carefully followed the [README.md](https://github.com/ggerganov/llama.cpp/blob/master/README.md).
- [x] I searched using keywords relevant to my issue to make sure that I am creating a new issue that is not already open (or closed).
- [x] I reviewed the [Discussions](https://github.com/ggerganov/llama.cpp/discussions), and have a new and useful enhancement to share.

### Feature Description

Integration of the sparse attention DeepSeek-V3.2-Exp model

### Motivation

Based on my understanding, DeepSeek-V3.2-Exp is an experimental sparse attention model with better efficiency. I'm wondering what this improved efficiency would translate to on a CPU-GPU deployment. Right now I don't think there is a way to run DeepSeek-V3.2-Exp with a CPU (more RAM) - GPU (limited VRAM) setup. I think it would be interesting if feasible.

### Possible Implementation

_No response_

---

## 💬 Conversation

👤 **ikawrakow** commented on **2025-10-15** at **16:13:47**

When you run a MoE model in a hybrid setup, where the experts are stored in RAM and attention related tensors are in VRAM, performance degradation with context length is much slower compared to the case where all tensors are on the GPU. 

For token generation this is so because the GPU is much faster than the CPU, so the time spent computing self-attention is much less than the time spent computing the FFN part of the network on the CPU.

For prompt processing (a.k.a., prefill) this is so because a significant time is spent copying tensors from RAM to VRAM, so the time taken to compute self-attention is again a much smaller fraction of the overall computation.

Given that most `ik_llama.cpp` users are on a hybrid CPU/GPU setup with consumer-grade hardware, performance gains will be much more modest compared to what we see in the DeepSeek-V3.2-Exp HF page.