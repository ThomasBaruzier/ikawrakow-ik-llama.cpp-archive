## 📌 [Issue #904](https://github.com/ikawrakow/ik_llama.cpp/issues/904) - Bug: KV cache ignore image for Qwen3-VL

| **Author** | `calvin2021y` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-11-05 |
| **Updated** | 2025-11-06 |

---

## 📄 Description

### What happened?

If send diff image, but same input text. response ignore the new image context.

### Name and Version

./llama-server --version
version: 3956 (320fc606)

### What operating system are you seeing the problem on?

_No response_

### Relevant log output

```shell

```

---

## 💬 Conversation

👤 **firecoperana** commented on **2025-11-05** at **13:21:40**

Let me look into it.

---

👤 **firecoperana** commented on **2025-11-05** at **17:31:21**

Can you try https://github.com/ikawrakow/ik_llama.cpp/pull/906?

---

👤 **calvin2021y** commented on **2025-11-06** at **06:16:55**

confirm [#906](https://github.com/ikawrakow/ik_llama.cpp/issues/906) fixed this