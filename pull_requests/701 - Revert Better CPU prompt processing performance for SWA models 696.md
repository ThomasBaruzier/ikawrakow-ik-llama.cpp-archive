## 🔀 [Pull Request #701](https://github.com/ikawrakow/ik_llama.cpp/pull/701) - Revert "Better CPU prompt processing performance for SWA models ([#696](https://github.com/ikawrakow/ik_llama.cpp/issues/696))"

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/reverts` |
| **Target Branch** | `main` |
| **Created** | 2025-08-17 |
| **Updated** | 2025-08-17 |
| **Merged** | 2025-08-17 |

---

## 📄 Description

Clearly I did not test well enough. The PR leads to a segmentation fault with hybrid GPU/CPU inference.

---

## 💬 Conversation

👤 **Ph0rk0z** commented on **2025-08-17** at **12:51:44**

Me too. Was just in the process of recompiling with the commit removed to see if that's it. GLM-4.5 gave segfault.

---

👤 **usrlocalben** commented on **2025-08-17** at **13:37:28**

here as well, was just about to report

---

👤 **calvin2021y** commented on **2025-08-17** at **20:29:53**

pure cpu also crash.