## 📌 [Issue #812](https://github.com/ikawrakow/ik_llama.cpp/issues/812) - Feature Request: Add GLM 4.6 support

| **Author** | `Geechan` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-10-01 |
| **Updated** | 2025-10-01 |
| **Labels** | `enhancement` |

---

## 📄 Description

### Prerequisites

- [x] I am running the latest code. Mention the version if possible as well.
- [x] I carefully followed the [README.md](https://github.com/ggerganov/llama.cpp/blob/master/README.md).
- [x] I searched using keywords relevant to my issue to make sure that I am creating a new issue that is not already open (or closed).
- [x] I reviewed the [Discussions](https://github.com/ggerganov/llama.cpp/discussions), and have a new and useful enhancement to share.

### Feature Description

GLM 4.6 marks the `layer.nextn.shared_head_head` and `layer.nextn.embed_tokens` tensors as not required, which prevents the model from loading with the error `llama_model_load: error loading model: missing tensor 'blk.92.nextn.embed_tokens.weight'`.

### Motivation

Adds GLM 4.6 support and allows it to load.

### Possible Implementation

A simple change to the code is needed to mark the tensors as not required, allowing the model to load. See here:
https://github.com/ggml-org/llama.cpp/pull/16359