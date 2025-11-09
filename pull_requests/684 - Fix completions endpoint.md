## 🔀 [Pull Request #684](https://github.com/ikawrakow/ik_llama.cpp/pull/684) - Fix completions endpoint

| **Author** | `firecoperana` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `fcp/fix_completions` |
| **Target Branch** | `main` |
| **Created** | 2025-08-10 |
| **Updated** | 2025-09-06 |
| **Merged** | 2025-08-11 |
| **Assignees** | `firecoperana` |

---

## 📄 Description

Fix the text completions issues after adding jinja template

- [x] I have read the [contributing guidelines](https://github.com/ggerganov/llama.cpp/blob/master/CONTRIBUTING.md)
- Self-reported review complexity:
  - [x] Low
  - [ ] Medium
  - [ ] High

---

## 💬 Conversation

👤 **ubergarm** commented on **2025-08-10** at **16:10:49**

Thanks, I saw a few people trying to use GLM-4.5-Air with ik_llama.cpp using SillyTavern client and `/text/completions/` endpoint who said they were seeing this before:

<img width="309" height="89" alt="image" src="https://github.com/user-attachments/assets/6c92c682-f57a-42e2-9db5-7a6b69b20d30" />

I suggested they try this PR to see if it fixes, not 100% it is relevant but seems related at quick glance.

Thanks!

---

👤 **firecoperana** commented on **2025-08-10** at **16:21:08**

Yes, this PR is meant to fix this error.

---

👤 **ikawrakow** approved this pull request ✅ on **2025-08-11** at **06:42:50**