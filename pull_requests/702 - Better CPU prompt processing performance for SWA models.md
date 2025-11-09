## 🔀 [Pull Request #702](https://github.com/ikawrakow/ik_llama.cpp/pull/702) - Better CPU prompt processing performance for SWA models

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Source Branch** | `ik/cpu_swa_v1` |
| **Target Branch** | `main` |
| **Created** | 2025-08-18 |
| **Updated** | 2025-09-04 |

---

## 📄 Description

This PR is a fixed version of [#696](https://github.com/ikawrakow/ik_llama.cpp/issues/696), see there for details.

The crashes we were getting on [#696](https://github.com/ikawrakow/ik_llama.cpp/issues/696) are due to the back-end not allocating a buffer for the tensor containing the mask bounds when this tensor is not used in the graph. Now we only create the mask bounds if they are actually used (SWA models with FA enabled).

---

## 💬 Conversation

👤 **ikawrakow** commented on **2025-09-04** at **10:11:03**

Closing in favor of [#757](https://github.com/ikawrakow/ik_llama.cpp/issues/757)