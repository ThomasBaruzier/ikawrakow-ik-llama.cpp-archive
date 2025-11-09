## 📌 [Issue #778](https://github.com/ikawrakow/ik_llama.cpp/issues/778) - Bug: response_format type must be one of "text" or "json_object", but got: json_schema', 'type': 'server_error'

| **Author** | `whoisjeremylam` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-09-13 |
| **Updated** | 2025-09-14 |

---

## 📄 Description

### What happened?

I am trying to use [Maestro](https://github.com/murtaza-nasir/maestro) which supports OpenAI API compatible endpoints. The API call is failing with the aforementioned error.

I found a related issue in `llama.cpp` from September, 2024: https://github.com/ggml-org/llama.cpp/pull/9527

If I run `llama-server` from `llama.cpp` then the API calls work, so I'm not sure if this is a case of functionality that needs to be brought to `ik_llama.cpp` or if there is a something that caused this not to work.

### Name and Version

version: 3850 (f6f56db0)
built with cc (Ubuntu 11.4.0-1ubuntu1~22.04) 11.4.0 for x86_64-linux-gnu

### What operating system are you seeing the problem on?

Linux

### Relevant log output

```shell
openai.InternalServerError: Error code: 500 - {'error': {'code': 500, 'message': 'response_format type must be one of "text" or "json_object", but got: json_schema', 'type': 'server_error'}}
```

---

## 💬 Conversation

👤 **firecoperana** commented on **2025-09-13** at **12:22:54**

Please update to latest main and try again. There has been some updates for the API of llama-server.

---

👤 **whoisjeremylam** commented on **2025-09-13** at **14:36:59**

Thanks, I've switched to `main` and at `head` now.

That's fixed the error!

Although, it seems that something in the response parsing isn't quite right. You can see `<|channel|>analysis<|message|>` in the second response.

<img width="630" height="746" alt="Image" src="https://github.com/user-attachments/assets/e9fc1d13-9455-4857-9b84-afaeee3fa675" />

This is with `gpt-oss-120b`:

```
~/ik_llama.cpp/build/bin/llama-server \
  -t 23 \
  -m ~/models/unsloth/gpt-oss-120b-GGUF/gpt-oss-120b-Q8_0.gguf \
  --alias gpt-oss-120b \
  --jinja \
  --no-mmap \
  --host 0.0.0.0 \
  --port 5000 \
  -c 50000 \
  -ngl 999 \
  --temp 1.0 --top-p 1.0 --top-k 0 \
  --chat-template-kwargs '{"reasoning_effort":"medium"}'
```

---

👤 **firecoperana** commented on **2025-09-13** at **21:47:43**

Does adding `--reasoning-format auto` fix the issue?

---

👤 **whoisjeremylam** commented on **2025-09-14** at **03:19:58**

It does. All fully operational with `gpt-oss-120b` now!