## 🔀 [Pull Request #874](https://github.com/ikawrakow/ik_llama.cpp/pull/874) - Fix assistant prefill in OAI chat API

| **Author** | `jarrodfeaks` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `fix-assistant-prefill` |
| **Target Branch** | `main` |
| **Created** | 2025-10-28 |
| **Updated** | 2025-10-29 |
| **Merged** | 2025-10-29 |

---

## 📄 Description

Support for prefilling assistant messages in the v1/chat/completions endpoint was added in [#723](https://github.com/ikawrakow/ik_llama.cpp/issues/723) (directly ported over from mainline), but until now it has not been functional. I had a look and realised the author missed a few lines from the source implementation, responsible for actually appending the prefilled content to the chat. This change simply restores them.

- [x] I have read the [contributing guidelines](https://github.com/ggerganov/llama.cpp/blob/master/CONTRIBUTING.md)
- Self-reported review complexity:
  - [x] Low
  - [ ] Medium
  - [ ] High

---

## 💬 Conversation

👤 **firecoperana** commented on **2025-10-28** at **16:43:49**

Thanks for this PR. It was indeed what I missed. I will test it when I'm home.

---

👤 **firecoperana** commented on **2025-10-29** at **00:23:34**

LGTM

---

👤 **firecoperana** approved this pull request ✅ on **2025-10-29** at **02:32:04**