## 📌 [Issue #912](https://github.com/ikawrakow/ik_llama.cpp/issues/912) - Bug: [Server] Verbose flag doesn't log text completions contents.

| **Author** | `Ph0rk0z` |
| :--- | :--- |
| **State** | ✅ **Open** |
| **Created** | 2025-11-07 |
| **Updated** | 2025-11-09 |

---

## 📄 Description

### What happened?

I run the server with -v and text completions do not show the prompts I have sent. Chat completions do show up in the terminal. Not sure if they're fully logged. 

I switch the name behavior in sillytavern but don't see name appended to the start of the prompt. Maybe ST is broken in that regard but I should definitely be able to see the other requests.

Anything else I can do besides compile with debug llama-server? When I do that it prints individual token stats and it's a bit much.

### Name and Version

HEAD

### What operating system are you seeing the problem on?

_No response_

### Relevant log output

```shell
add_text: <|im_start|>system

vs

INFO [            update_slots] kv cache rm [p0, end) | tid="139703764496384" timestamp=1762478090 id_slot=0 id_task=0 p0=0
INFO [           print_timings] prompt eval time     =    8739.27 ms /  1703 tokens (    5.13 ms per token,   194.87 tokens per second) | tid="139703764496384" timestamp=1762478102 id_slot=0 id_task=0 t_prompt_processing=8739.271 n_prompt_tokens_processed=1703 t_token=5.131691720493247 n_tokens_second=194.86751240463877
```

---

## 💬 Conversation

👤 **firecoperana** commented on **2025-11-09** at **00:04:48**

Do you want to log the prompt or the final generated message in verbose?

---

👤 **Ph0rk0z** commented on **2025-11-09** at **01:57:43**

At least log the prompt so I can see what my client is sending and if the template is right, etc.