## 📌 [Issue #859](https://github.com/ikawrakow/ik_llama.cpp/issues/859) - Bug: [server] Text completions keep goin and goin.

| **Author** | `Ph0rk0z` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-10-23 |
| **Updated** | 2025-10-27 |

---

## 📄 Description

### What happened?

I use server with the --jinja flag and set a preset and stopping strings. It's not the same chat preset as the one in the metadata. My client (sillytavern) correctly stops streaming but the server keeps generating in the background. I have to manually send a stop command to the server.

I also tried disabling streaming and the behavior is the same. Just goes till the max output tokens is reached, ignoring all custom stopping strings.




### Name and Version

latest GIT

### What operating system are you seeing the problem on?

Linux

### Relevant log output

```shell
INFO [           print_timings] generation eval time =  175058.09 ms /  2048 runs   (   85.48 ms per token,    11.70 tokens per second) | tid="139933603020800" timestamp=1761227346 id_slot=0 id_task=12620 t_token_generation=175058.094 n_decoded=2048 t_token=85.4775849609375 n_tokens_second=11.698973484767862
INFO [           print_timings]           total time =  178251.15 ms | tid="139933603020800" timestamp=1761227346 id_slot=0 id_task=12620 t_prompt_processing=3193.055 t_token_generation=175058.094 t_total=178251.149


Actual reply was only 650 tokens before it emitted <im-end>
```

---

## 💬 Conversation

👤 **Ph0rk0z** commented on **2025-10-23** at **14:18:37**

On a hunch I revert: https://github.com/ikawrakow/ik_llama.cpp/commit/41bdd865552c9b592e7ab1b77e453485d295238d

Now the server stops correctly... so far. I'll keep testing.

---

👤 **firecoperana** commented on **2025-10-26** at **22:19:46**

I can reproduce this. Will fix.

---

👤 **firecoperana** commented on **2025-10-27** at **00:31:08**

Can you test https://github.com/ikawrakow/ik_llama.cpp/pull/870?

---

👤 **Ph0rk0z** commented on **2025-10-27** at **12:47:29**

heh.. i was asleep. i will pull and test today to see what happens.

---

👤 **Ph0rk0z** commented on **2025-10-27** at **14:24:40**

Ok, the stopping bug appears fixed.