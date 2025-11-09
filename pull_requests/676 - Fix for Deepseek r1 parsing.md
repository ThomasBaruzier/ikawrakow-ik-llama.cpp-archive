## 🔀 [Pull Request #676](https://github.com/ikawrakow/ik_llama.cpp/pull/676) - Fix for Deepseek r1 parsing

| **Author** | `iSevenDays` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `deepseek-r1-parsing` |
| **Target Branch** | `main` |
| **Created** | 2025-08-07 |
| **Updated** | 2025-08-08 |
| **Merged** | 2025-08-08 |

---

## 📄 Description

- [x] I have read the [contributing guidelines](https://github.com/ggerganov/llama.cpp/blob/master/CONTRIBUTING.md)
- Self-reported review complexity:
  - [x] Low
  - [ ] Medium
  - [ ] High

 The fix restores the
  missing consume_rest() call that was present in working PR [#648](https://github.com/ikawrakow/ik_llama.cpp/issues/648), ensuring that responses wrapped in
  <think> tags are properly displayed as content

---

## 💬 Conversation

👤 **iSevenDays** commented on **2025-08-08** at **10:55:18**

The fix for thinking tags has been tested by the user here https://github.com/ikawrakow/ik_llama.cpp/pull/652#issuecomment-3167144419

---

👤 **ikawrakow** approved this pull request ✅ on **2025-08-08** at **10:56:38**

---

👤 **raidshoebox1** commented on **2025-08-08** at **13:53:41**

While the https://github.com/iSevenDays/ik_llama.cpp/tree/deepseek-r1-parsing tested OK previously, the thinking tag issue persists on main branch after merging this PR. 

Could you make further fixes?

---

👤 **ikawrakow** commented on **2025-08-08** at **13:59:09**

You mean the exact same testing that you did on the PR branch no longer works on the main branch?

---

👤 **raidshoebox1** commented on **2025-08-08** at **14:20:50**

Yes. I've tested both branches using the same parameter , and can consistently reproduce the 'thinking tag' issue exclusively on the main branch.