## 🔀 [Pull Request #840](https://github.com/ikawrakow/ik_llama.cpp/pull/840) - Various fused ops around expert selection

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/fused_bailingmoev2` |
| **Target Branch** | `main` |
| **Created** | 2025-10-19 |
| **Updated** | 2025-10-20 |
| **Merged** | 2025-10-19 |

---

## 📄 Description

Nothing earth shattering, just 1-2% performance gains.

---

## 💬 Conversation

👤 **ikawrakow** commented on **2025-10-19** at **16:02:17**

OK, here some graphs for Ling-mini-2.0-Q4_K_M (RTX-4080) showing the cumulative effect of the changes since grouped expert routing was added in Ling/Ring in [#838](https://github.com/ikawrakow/ik_llama.cpp/issues/838). Also shown results for the not yet merged [PR 16063](https://github.com/ggml-org/llama.cpp/pull/16063) in mainline. u-batch size is 2048 and FA is on.

### Prompt processing  

<img width="792" height="612" alt="u1" src="https://github.com/user-attachments/assets/cb5db941-8185-4342-995d-5dc40662140f" />

### Token generation

<img width="792" height="612" alt="u1a" src="https://github.com/user-attachments/assets/ce53f54b-9308-468f-99b5-6dddce7a8e93" />

---

👤 **magikRUKKOLA** commented on **2025-10-19** at **19:46:17**

@ikawrakow 
Is it possible to use Ling-mini-2.0-Q2_K.gguf as a draft model for speculative decoding with ubergarm/smol-IQ4-KSS ?

---

👤 **ikawrakow** commented on **2025-10-20** at **09:25:09**

> Is it possible to use Ling-mini-2.0-Q2_K.gguf as a draft model for speculative decoding with ubergarm/smol-IQ4-KSS ?

If the vocabulary is the same it should be possible. But I haven't tried myself.