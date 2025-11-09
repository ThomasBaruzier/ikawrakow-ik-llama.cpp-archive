## 📌 [Issue #730](https://github.com/ikawrakow/ik_llama.cpp/issues/730) - Feature Request: add support for vision model InternVL3_5

| **Author** | `kiron111` |
| :--- | :--- |
| **State** | ✅ **Open** |
| **Created** | 2025-08-26 |
| **Updated** | 2025-10-04 |
| **Labels** | `enhancement` |

---

## 📄 Description

### Prerequisites

- [x] I am running the latest code. Mention the version if possible as well.
- [x] I carefully followed the [README.md](https://github.com/ggerganov/llama.cpp/blob/master/README.md).
- [x] I searched using keywords relevant to my issue to make sure that I am creating a new issue that is not already open (or closed).
- [x] I reviewed the [Discussions](https://github.com/ggerganov/llama.cpp/discussions), and have a new and useful enhancement to share.

### Feature Description

Some of them are MOE (some are based on Qwen3):
https://huggingface.co/OpenGVLab/InternVL3_5-30B-A3B
https://huggingface.co/OpenGVLab/InternVL3_5-241B-A28B

<img width="1787" height="806" alt="Image" src="https://github.com/user-attachments/assets/c82c005f-c8dc-4fb9-8a2b-db092fb60fc0" />

It should be now SOTA in open source VL models (based on InternLM Lab's report)

GGUF here:
https://huggingface.co/QuantStack/InternVL3_5-30B-A3B-gguf/tree/main


It would be great if ik_llama.cpp would support it.
Thankyou!!

### Motivation

it could be run in mainline llama.cpp (tesed) , but the speed is not optimized, could be greatly enhanced in ik-llama.cpp in hybrid CPU+GPU inference.

### Possible Implementation

_No response_

---

## 💬 Conversation

👤 **ikawrakow** commented on **2025-09-26** at **16:52:13**

This can be addressed after merging [#798](https://github.com/ikawrakow/ik_llama.cpp/issues/798).

---

👤 **kiron111** commented on **2025-10-04** at **08:00:44**

> This can be addressed after merging [[#798](https://github.com/ikawrakow/ik_llama.cpp/issues/798)](https://github.com/ikawrakow/ik_llama.cpp/pull/798).

thankyou!!