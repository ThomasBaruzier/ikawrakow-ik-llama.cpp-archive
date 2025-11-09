## 🔀 [Pull Request #810](https://github.com/ikawrakow/ik_llama.cpp/pull/810) - Fix incomplete utf-8 characters in streaming text completions

| **Author** | `loge-gh` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `streaming-broken-utf8` |
| **Target Branch** | `main` |
| **Created** | 2025-09-30 |
| **Updated** | 2025-10-13 |
| **Merged** | 2025-10-13 |

---

## 📄 Description

Closes https://github.com/ikawrakow/ik_llama.cpp/issues/780

As requested by @firecoperana I've identified the problem: when the stopping string is encountered for the first time, the variables slot.n_sent_text, pos, and stop_pos get stuck for the rest of response. This vibe-coded (sorry! not quite understand the function logic) patch should fix the bug. At least it may help somebody to make a proper patch.

---

## 💬 Conversation

👤 **ikawrakow** commented on **2025-10-07** at **07:24:58**

Has anyone tried if this fixes the issue?

@Ph0rk0z @sousekd

---

👤 **Ph0rk0z** commented on **2025-10-07** at **13:38:56**

Per this, it's not fully complete fix: https://github.com/loge-gh/ik_llama.cpp/pull/1

---

👤 **sousekd** commented on **2025-10-12** at **23:13:23**

> Has anyone tried if this fixes the issue?

Yes, this solves the issue for me.

---

👤 **ikawrakow** approved this pull request ✅ on **2025-10-13** at **13:25:21**