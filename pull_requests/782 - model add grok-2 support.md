## 🔀 [Pull Request #782](https://github.com/ikawrakow/ik_llama.cpp/pull/782) - model : add grok-2 support

| **Author** | `firecoperana` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `fcp/grok2` |
| **Target Branch** | `main` |
| **Created** | 2025-09-17 |
| **Updated** | 2025-10-26 |
| **Merged** | 2025-09-23 |
| **Assignees** | `firecoperana` |

---

## 📄 Description

Add support for grok-2.

Test with unsloth/grok-2-UD-Q2_K_XL.gguf and looks ok.
Use `--jinja` for loading the model with `llama-server`

https://github.com/ggml-org/llama.cpp/pull/15539

---

## 💬 Conversation

👤 **ikawrakow** started a conversation on `common/common.h` on **2025-09-23** at **06:42:25**

Why is this change required?

> 👤 **firecoperana** replied on **2025-09-23** at **12:36:24**
> 
> This copies from mainline.

---

👤 **ikawrakow** started a conversation on `src/llama-vocab.h` on **2025-09-23** at **06:45:13**

Align '=' as the other entries

---

👤 **ikawrakow** approved this pull request ✅ on **2025-09-23** at **06:47:16**