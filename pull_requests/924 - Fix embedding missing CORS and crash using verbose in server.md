## 🔀 [Pull Request #924](https://github.com/ikawrakow/ik_llama.cpp/pull/924) - Fix embedding missing, CORS and crash using verbose in server

| **Author** | `firecoperana` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `fcp/mtmd_fix` |
| **Target Branch** | `main` |
| **Created** | 2025-11-08 |
| **Updated** | 2025-11-09 |
| **Merged** | 2025-11-09 |

---

## 📄 Description

Fix https://github.com/ikawrakow/ik_llama.cpp/issues/915, https://github.com/ikawrakow/ik_llama.cpp/issues/918 and https://github.com/ikawrakow/ik_llama.cpp/issues/919

---

## 💬 Conversation

👤 **ikawrakow** started a conversation on `examples/server/server.cpp` on **2025-11-09** at **06:20:48**

Why cast to `int`?

---

👤 **ikawrakow** approved this pull request ✅ on **2025-11-09** at **06:23:12**

---

👤 **MrHills-2** commented on **2025-11-09** at **11:13:28**

Verbose crash is fixed as of d7385ac
Thanks