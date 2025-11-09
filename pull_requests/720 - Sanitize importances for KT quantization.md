## 🔀 [Pull Request #720](https://github.com/ikawrakow/ik_llama.cpp/pull/720) - Sanitize importances for KT quantization

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/sanitize_importance_kt_quants` |
| **Target Branch** | `main` |
| **Created** | 2025-08-23 |
| **Updated** | 2025-08-27 |
| **Merged** | 2025-08-27 |

---

## 📄 Description

This PR adds more sanity checks on model weights and imatrix values for trellis quantization (`IQ1_KT, IQ2_KT, IQ3_KT, IQ4_KT`).

@ubergarm Does this fix the problem reported in [#650](https://github.com/ikawrakow/ik_llama.cpp/issues/650) ?

---

## 💬 Conversation

👤 **ubergarm** commented on **2025-08-23** at **16:36:09**

Yes, this PR fixes the problem! Additional notes in the Issue.