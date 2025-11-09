## 🔀 [Pull Request #696](https://github.com/ikawrakow/ik_llama.cpp/pull/696) - Better CPU prompt processing performance for SWA models

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/cpu_swa_v1` |
| **Target Branch** | `main` |
| **Created** | 2025-08-16 |
| **Updated** | 2025-08-17 |
| **Merged** | 2025-08-17 |

---

## 📄 Description

This PR is a follow up of [#692](https://github.com/ikawrakow/ik_llama.cpp/issues/692) and uses the same technique to improve prompt processing performance for models utilizing SWA. As [#682](https://github.com/ikawrakow/ik_llama.cpp/issues/682) it is implemented only on the CPU and requires FA.

Here some performance comparisons on a Ryzen-7950X CPU

### Gemma3-270M-it, Q8_0

 
<img width="792" height="612" alt="g3_swa_pp" src="https://github.com/user-attachments/assets/f8ca95e5-0aee-4aa0-a600-d5404c423626" />

### GPT-OSS-20B, MXFP4

<img width="792" height="612" alt="gpt_oss_swa_pp" src="https://github.com/user-attachments/assets/6559ef7a-4118-4a16-bde5-5c3997b992ad" />

---

## 💬 Conversation

👤 **ikawrakow** commented on **2025-08-16** at **05:35:56**

Just for fun, here a CPU-only comparison with mainline `llama.cpp` for GPT-OSS-20B-MXFP4 with `Q8_0` KV cache:

 
<img width="792" height="612" alt="gpt_oss_swa_pp1" src="https://github.com/user-attachments/assets/28613474-bd33-4035-ba48-887bdc8539bd" />