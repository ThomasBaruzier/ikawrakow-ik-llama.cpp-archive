## 🔀 [Pull Request #688](https://github.com/ikawrakow/ik_llama.cpp/pull/688) - Gracefully fail the decode instead of crashing for kshift Deepseek error

| **Author** | `saood06` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `s6/fix_kshift_crash` |
| **Target Branch** | `main` |
| **Created** | 2025-08-13 |
| **Updated** | 2025-08-13 |
| **Merged** | 2025-08-13 |

---

## 📄 Description

Tested working, server no longer crashes with `Deepseek2 does not support K-shift`, and just gracefully exits the decode call with a warning.

@Lissanro

Closes [#686](https://github.com/ikawrakow/ik_llama.cpp/issues/686)

---

## 💬 Conversation

👤 **ikawrakow** started a conversation on `src/llama.cpp` on **2025-08-13** at **09:56:40**

Please fix formatting: no tabs and no curly braces on new lines.

> 👤 **saood06** replied on **2025-08-13** at **10:07:00**
> 
> Done, sorry I forgot to go back and fix these things like I usually do before committing.

---

👤 **ikawrakow** approved this pull request ✅ on **2025-08-13** at **09:56:51**