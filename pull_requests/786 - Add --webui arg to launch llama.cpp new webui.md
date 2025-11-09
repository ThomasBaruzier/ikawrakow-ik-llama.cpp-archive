## 🔀 [Pull Request #786](https://github.com/ikawrakow/ik_llama.cpp/pull/786) - Add --webui arg to launch llama.cpp new webui

| **Author** | `firecoperana` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `fcp/alt_frontend` |
| **Target Branch** | `main` |
| **Created** | 2025-09-19 |
| **Updated** | 2025-10-27 |
| **Merged** | 2025-10-27 |
| **Assignees** | `firecoperana` |

---

## 📄 Description

This PR adds `--webui` parameter in llama-server so user can select different built-in webui. This is useful for those who wants to try new llama.cpp webui without losing access to the default webui. (https://github.com/ggml-org/llama.cpp/pull/14839) New llama.cpp webui is still in beta phase, which expects more features and bug fixes incoming. If you encounter bugs, please confirm if mainline has it. If it does, submit issues there. To request more features, also submit feature request there. I just port it without doing much modification. New upgrade in llama.cpp includes better UI, conversation branching...
Use with caution and either start server in new ip:port or make backups to your conversation history. 

To launch it, use `--webui llamacpp`. By default, it still use current webui.
`--webui` accepts three values:
`none`: disable webui
`auto`: default webui 
`llamacpp`: llamacpp webui

---

## 💬 Conversation

👤 **ikawrakow** commented on **2025-09-23** at **06:55:42**

Unless changes 2-5 are tightly coupled with change 1 and are difficult to disentangle, I think it would be better to have them in a separate PR.

I do see quite a few bug reports in mainline related to the new webui. Perhaps it would be better to keep change 1 on a branch until most issues are fixed?

---

👤 **firecoperana** commented on **2025-09-23** at **13:22:41**

Yes, that is possible.

---

👤 **firecoperana** commented on **2025-09-24** at **00:30:28**

Separate PR is created for 2-5

---

👤 **firecoperana** commented on **2025-10-26** at **16:48:33**

There has been a lot of updates for new webui. I don't see many bug reports any more. I think it is ready to be added as an alternative webui. Testing is welcome. 

Some notable PRs: 
Centralize CoT parsing in backend for streaming mode https://github.com/ggml-org/llama.cpp/pull/16394
Add server-driven parameter defaults and syncing https://github.com/ggml-org/llama.cpp/pull/16515
Enable per-conversation loading states to allow having parallel conversations https://github.com/ggml-org/llama.cpp/pull/16327
Introduce OpenAI-compatible model selector in JSON payload https://github.com/ggml-org/llama.cpp/pull/16562

---

👤 **ikawrakow** commented on **2025-10-26** at **17:04:29**

Did they fix the one where the UI was losing tokens under certain conditions?

---

👤 **firecoperana** commented on **2025-10-26** at **17:07:53**

Yes. https://github.com/ggml-org/llama.cpp/issues/16417 was closed.

---

👤 **ikawrakow** commented on **2025-10-26** at **17:14:37**

Kind of hard to review a 30k LOC PR, so I guess we merge and hope for the best?

It would be nice if a few people tried and gave feedback...

---

👤 **firecoperana** commented on **2025-10-26** at **17:30:48**

Most of the code is to create the html file for the webui, and not much code change in cpp. Bugs outside of the new webui should be easy to fix.

---

👤 **Nexesenex** commented on **2025-10-27** at **00:52:54**

I compiled then launched both the default and llama.cpp UI and had a little chat with each.
Nothing odd happened on my side.

---

👤 **ikawrakow** approved this pull request ✅ on **2025-10-27** at **12:21:54**