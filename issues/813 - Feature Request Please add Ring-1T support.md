## 📌 [Issue #813](https://github.com/ikawrakow/ik_llama.cpp/issues/813) - Feature Request: Please add Ring-1T support

| **Author** | `Alexey-Akishin` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-10-01 |
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

Recently Ring-1T was released: https://huggingface.co/inclusionAI/Ring-1T-preview - it has one trillion parameters model with 50B active ones, which is more than in Kimi K2 or DeepSeek, and also it supports reasoning.

### Motivation

I think it could be a cool alternative to Kimi K2 (also 1T) model, and it also has [MIT License](https://github.com/inclusionAI/Ling-V2/blob/main/LICENSE) according to its model card. From its description in the model card, it sounds like more releases are coming. But it is offered in BF16 format, which is very big download. On huggingface there is also 3-bit MLX quant exists, but no GGUFs, hence I am assuming it is not supported just yet.

### Possible Implementation

_No response_

---

## 💬 Conversation

👤 **Lissanro** commented on **2025-10-14** at **00:52:04**

There is PR for llama.cpp for it: https://github.com/ggml-org/llama.cpp/pull/16063 - based on the most recent comments, sounds like it is very close to being accepted. Unfortunately, the patch does not apply to ik_llama.cpp due to some files being different or even not existing at all. I am not sure how much work would it take to convert it to a patch for ik_llama.cpp.

By the way, two more models were released since the Ring-1T-Preview, so in total there are three already:

https://huggingface.co/inclusionAI/Ring-1T-preview (the initial preview)
https://huggingface.co/inclusionAI/Ling-1T (no thinking)
https://huggingface.co/inclusionAI/Ring-1T (thinking)

---

👤 **ikawrakow** commented on **2025-10-14** at **08:14:33**

Is there a small model that I can use for testing? 1T is way beyond my hardware capabilities.

---

👤 **kirnat** commented on **2025-10-14** at **08:35:18**

They have a llama.cpp fork and GGUF of Ling-mini-2.0
https://huggingface.co/inclusionAI/Ling-mini-2.0-GGUF

---

👤 **ikawrakow** commented on **2025-10-14** at **13:37:17**

See [#833](https://github.com/ikawrakow/ik_llama.cpp/issues/833)

---

👤 **kirnat** commented on **2025-10-14** at **16:30:37**

ikawrakow is like the AI agent that actually works

---

👤 **ikawrakow** commented on **2025-10-14** at **16:33:57**

Haha, you are not the first one to suspect an AI (or even AGI) behind the ikawrakow account 😄