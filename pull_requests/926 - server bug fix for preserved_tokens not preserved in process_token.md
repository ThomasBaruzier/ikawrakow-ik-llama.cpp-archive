## 🔀 [Pull Request #926](https://github.com/ikawrakow/ik_llama.cpp/pull/926) - server: bug fix for preserved_tokens not preserved in process_token

| **Author** | `firecoperana` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `fcp/deepseek3.1_toolcall_fix` |
| **Target Branch** | `main` |
| **Created** | 2025-11-08 |
| **Updated** | 2025-11-09 |
| **Merged** | 2025-11-09 |

---

## 📄 Description

Address https://github.com/ikawrakow/ik_llama.cpp/issues/925.
Thanks to @KiruyaMomochi, this PR fixes DeepSeek V3.1 function calling. Don't have the model to test, but should be good as it follows mainline.

---

## 💬 Conversation

👤 **firecoperana** commented on **2025-11-08** at **23:32:13**

@kirnat  Can you test this?

---

👤 **ikawrakow** approved this pull request ✅ on **2025-11-09** at **06:17:26**

---

👤 **magikRUKKOLA** commented on **2025-11-09** at **10:19:11**

@firecoperana 

lol that's was the reason I always ran the server with --special enabled.  I can retest with --special disabled.  Should I?