## 🔀 [Pull Request #791](https://github.com/ikawrakow/ik_llama.cpp/pull/791) - Update webui to handle reasoning content and include usage stats in server only when requested

| **Author** | `firecoperana` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `fcp/webui_reasoning_handle` |
| **Target Branch** | `main` |
| **Created** | 2025-09-24 |
| **Updated** | 2025-10-26 |
| **Merged** | 2025-09-24 |
| **Assignees** | `firecoperana` |

---

## 📄 Description

The following changes are included in this PR:

1. Change reasoning format's default value to auto and make current webui compatible with this change (
https://github.com/ggml-org/llama.cpp/pull/15052). This should be fine with most 3rd party front end as they should be updated by now. If not, use --reasoning-format none to start server.

2. Add config in current webui for reasoning format. When encountering parsing issue for reasoning content, switch between none and auto to see which one fix them.

3. server : include usage statistics only when user request them (
https://github.com/ggml-org/llama.cpp/pull/16052). This could be a breaking change, but is compatible with openai API.

3. server : only attempt to enable thinking if using jinja (
https://github.com/ggml-org/llama.cpp/pull/15967)

---

## 💬 Conversation

👤 **ikawrakow** approved this pull request ✅ on **2025-09-24** at **05:41:57**