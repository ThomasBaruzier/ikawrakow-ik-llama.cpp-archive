## 🔀 [Pull Request #799](https://github.com/ikawrakow/ik_llama.cpp/pull/799) - sync: vendor

| **Author** | `firecoperana` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `fcp/sync_jinja_vendor` |
| **Target Branch** | `main` |
| **Created** | 2025-09-26 |
| **Updated** | 2025-10-26 |
| **Merged** | 2025-09-26 |
| **Assignees** | `firecoperana` |

---

## 📄 Description

Some users still report underwhelming results with tool callings after Deepseek V3.1 tool call support is merged (https://github.com/ikawrakow/ik_llama.cpp/pull/771). This PR is the initial attempt to fix it. It simply updates minja.hpp and json.hpp from mainline. This is needed even if it does not fix Deepseek v3.1 tool calling issue.

---

## 💬 Conversation

👤 **firecoperana** commented on **2025-09-26** at **13:37:14**

@kirnat Can you test this?

---

👤 **kirnat** commented on **2025-09-26** at **14:38:38**

Thanks a lot, unfortunately this PR didn't seem to solve this particular issue with tools calls being picked up properly. Hopefully someone else can confirm so it's not local to my specific setup. When you tried originally with DS 3.1, do you remember which quantized version you used? I could test with the same if it would be meaningful to rule out any issues with chat template?

---

👤 **ubergarm** commented on **2025-09-26** at **15:11:17**

Just had a report come in about the new Kimi-K2 tool-calling as moonshot just released a tool-calling test harness verifier tool and someone was having an issue: https://huggingface.co/ubergarm/Kimi-K2-Instruct-0905-GGUF/discussions/1#68d5df5f268bd3e07916baba

I asked them to try again with this PR and report back if possible.

---

👤 **firecoperana** commented on **2025-09-26** at **16:14:31**

> Thanks a lot, unfortunately this PR didn't seem to solve this particular issue with tools calls being picked up properly. Hopefully someone else can confirm so it's not local to my specific setup. When you tried originally with DS 3.1, do you remember which quantized version you used? I could test with the same if it would be meaningful to rule out any issues with chat template?

I don't have the model to test as the previous PR is a cherrypick from the mainline.

---

👤 **ikawrakow** approved this pull request ✅ on **2025-09-26** at **16:22:41**