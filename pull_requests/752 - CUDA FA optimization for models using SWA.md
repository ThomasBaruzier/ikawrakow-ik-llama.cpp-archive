## 🔀 [Pull Request #752](https://github.com/ikawrakow/ik_llama.cpp/pull/752) - CUDA: FA optimization for models using SWA

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Source Branch** | `ik/cuda_swa2` |
| **Target Branch** | `main` |
| **Created** | 2025-09-02 |
| **Updated** | 2025-09-04 |

---

## 📄 Description

This PR is analogous to [#702](https://github.com/ikawrakow/ik_llama.cpp/issues/702) and implements the optimization for the CUDA back-end.

Here performance comparisons between the main branch and this PR for `Q4_0`-quantized GPT-OSS-20B running on an RTX-4080 GPU:

### Prompt processing

<img width="792" height="612" alt="u2pp" src="https://github.com/user-attachments/assets/eb879ab8-b1c5-4e1a-ab3e-4bb2269d8bc6" />

### Token generation

<img width="792" height="612" alt="u2tg" src="https://github.com/user-attachments/assets/3be6ccb3-17aa-4891-86bc-7f69b51b8ab6" />

---

## 💬 Conversation

👤 **ikawrakow** commented on **2025-09-04** at **06:43:00**

Closing in favor of [#754](https://github.com/ikawrakow/ik_llama.cpp/issues/754)