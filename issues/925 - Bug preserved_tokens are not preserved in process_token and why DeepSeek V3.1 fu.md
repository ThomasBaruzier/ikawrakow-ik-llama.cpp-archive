## 📌 [Issue #925](https://github.com/ikawrakow/ik_llama.cpp/issues/925) - Bug: preserved_tokens are not preserved in `process_token` (and why DeepSeek V3.1 function calling fails)

| **Author** | `KiruyaMomochi` |
| :--- | :--- |
| **State** | ✅ **Open** |
| **Created** | 2025-11-08 |
| **Updated** | 2025-11-08 |

---

## 📄 Description

### What happened?

Background:

1. DeepSeek V3.1 tool calling works on mainline, but fails on ik. Applied almost all the new changes of `common/chat.cpp` but it still fails.
2. Setting `--special` works.

Reason:

common/chat.cpp: common_chat_params_init_deepseek_r1: set `data.preserved_tokens`
https://github.com/ikawrakow/ik_llama.cpp/blob/e5fc02c71a361730842f23544c6852fb6cc2e3b9/common/chat.cpp#L1327-L1335

Therefore we use `common_token_to_piece(ctx, result.tok, accept_special_token(slot, result.tok));` to get text to send.
- In mainline:
https://github.com/ggml-org/llama.cpp/blob/aa3b7a90b407c556778a7e13a4b0d28cf964fd1c/tools/server/server.cpp#L4281
- In our fork:
https://github.com/ikawrakow/ik_llama.cpp/blob/e5fc02c71a361730842f23544c6852fb6cc2e3b9/examples/server/server.cpp#L3338

However, in the `process_token` function, we are manually calculate `token_str` which only consider `params.special` and ignored `preserved_tokens`:
https://github.com/ikawrakow/ik_llama.cpp/blob/e5fc02c71a361730842f23544c6852fb6cc2e3b9/examples/server/server.cpp#L1980

**Fix**:

To fix, just set `const std::string token_str = result.text_to_send;` as in mainline:
https://github.com/ggml-org/llama.cpp/blob/aa3b7a90b407c556778a7e13a4b0d28cf964fd1c/tools/server/server.cpp#L2857

I have confirmed this fix works for DeepSeek V3.1 tool calling.

### Name and Version

Built with gcc (GCC) 14.3.0 for x86_64-unknown-linux-gnu on latest main branch (e5fc02c)

### What operating system are you seeing the problem on?

Linux

### Relevant log output

```shell

```

---

## 💬 Conversation

👤 **firecoperana** commented on **2025-11-08** at **23:33:06**

Thanks! https://github.com/ikawrakow/ik_llama.cpp/pull/926 created