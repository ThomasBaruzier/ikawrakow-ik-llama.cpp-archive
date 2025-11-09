## 🔀 [Pull Request #748](https://github.com/ikawrakow/ik_llama.cpp/pull/748) - Revert "CUDA: prompt processing optimizations for MoE models ([#739](https://github.com/ikawrakow/ik_llama.cpp/issues/739))"

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/revert_739` |
| **Target Branch** | `main` |
| **Created** | 2025-09-01 |
| **Updated** | 2025-09-02 |
| **Merged** | 2025-09-02 |

---

## 📄 Description

This reverts commit f22a9ef95a58c782f94f0aa7b320c21d0a542e9d.

Not sure what the issue is (works for me with the 3 tested models), but given [#746](https://github.com/ikawrakow/ik_llama.cpp/issues/746), reverting for now.

---

## 💬 Conversation

👤 **magikRUKKOLA** commented on **2025-09-01** at **21:29:10**

@ikawrakow 

Hey bro, I was thinking to create a bash script to automatically apply the PR and at least test the code for PPL  So I can download various LLMs and quick-test it and post the results.

I can provide a web host lets say where the results will be posted for each llm.

This way the bugs such as these could be fixed sooner than later.  What do you think?

---

👤 **firecoperana** commented on **2025-09-02** at **01:45:26**

This also fixed the issue for me. My issue is with -mla 1 -fa -rtr, small batch size (64 ubatch and 256 batch size), without -fmoe and with rpc-server on another pc. If I add -fmoe, the output looks fine. If I don't, the output feels like it only understands part of my prompt. I only tested Deepseek V3.