## 📌 [Issue #847](https://github.com/ikawrakow/ik_llama.cpp/issues/847) - Feature Request: Select quants to compile

| **Author** | `wallentri88` |
| :--- | :--- |
| **State** | ✅ **Open** |
| **Created** | 2025-10-20 |
| **Updated** | 2025-11-04 |
| **Labels** | `enhancement` |

---

## 📄 Description

### Prerequisites

- [x] I am running the latest code. Mention the version if possible as well.
- [x] I carefully followed the [README.md](https://github.com/ggerganov/llama.cpp/blob/master/README.md).
- [x] I searched using keywords relevant to my issue to make sure that I am creating a new issue that is not already open (or closed).
- [x] I reviewed the [Discussions](https://github.com/ggerganov/llama.cpp/discussions), and have a new and useful enhancement to share.

### Feature Description

Current implementation compiles all quants, but it makes compilation painful.

more context here https://github.com/ikawrakow/ik_llama.cpp/discussions/846

### Motivation

Reduces compile time

### Possible Implementation

_No response_

---

## 💬 Conversation

👤 **MrHills-2** commented on **2025-11-01** at **23:14:09**

Compile time is pretty short for me with an 8 cores, even with GGML_CUDA_FA_ALL_QUANTS set. Less then a minute. You are using the -j argument for multi core compilation, right? -j 16 for example of you have 16 cores.

---

👤 **wallentri88** commented on **2025-11-04** at **15:45:06**

Yes, I'm using -j arg and I have plenty of RAM. Probably my cpu is kinda old