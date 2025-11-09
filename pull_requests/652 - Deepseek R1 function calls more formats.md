## 🔀 [Pull Request #652](https://github.com/ikawrakow/ik_llama.cpp/pull/652) - Deepseek R1 function calls (more formats)

| **Author** | `iSevenDays` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `deepseek-r1-parsing` |
| **Target Branch** | `main` |
| **Created** | 2025-07-26 |
| **Updated** | 2025-08-08 |
| **Merged** | 2025-08-07 |

---

## 📄 Description

- [x] I have read the [contributing guidelines](https://github.com/ggerganov/llama.cpp/blob/master/CONTRIBUTING.md)
- Self-reported review complexity:
  - [ ] Low
  - [x] Medium
  - [ ] High


Implemented more DeepSeek R1 supported function tool calls formats.
The diff of `examples/server/function_calls.md` shows what formats are supported. 

I was testing DeepSeek R1 and I found out that it often uses different formats with Claude Code, so I decided to support them as well. It can be useful when next version of DeepSeek is released, so we will have better support than even original llama.cpp

---

## 💬 Conversation

👤 **ikawrakow** approved this pull request ✅ on **2025-08-07** at **05:15:50**

---

👤 **raidshoebox1** commented on **2025-08-07** at **15:21:58**

Thank you for your hard work. But after this pull merged, the entire response from DeepSeek-R1-0528 is wrapped within the "thinking" tag.

<img width="1163" height="564" alt="image" src="https://github.com/user-attachments/assets/4de785f3-6532-4c46-a3b6-e9bdff780c56" />


Rolling back to pull [#648](https://github.com/ikawrakow/ik_llama.cpp/issues/648), it works normally.

<img width="1183" height="551" alt="image" src="https://github.com/user-attachments/assets/595b611e-9374-4cc1-b770-fee8b1879eb7" />

My parameters here:

/build/bin/llama-server \
-m /root/models/DeepSeek-R1-0528/DeepSeek-R1-0528-UD-Q4_K_XL-00001-of-00008.gguf \
--threads 32 \
--ctx-size 76800 \
-ot "exps=CPU" \
-ngl 99 \
--host 0.0.0.0 \
--port 8089 \
--verbose \
-a DeepSeek-R1-0528 \
-fa \
-fmoe \
-mla 3 \
-amb 512 \
-rtr

---

👤 **ikawrakow** commented on **2025-08-07** at **15:26:10**

@iSevenDays 

Do you see a fix? Else it would be better to revert.

---

👤 **iSevenDays** commented on **2025-08-07** at **18:12:02**

@raidshoebox1 thank you for the bug report!
Could you please check if the issue persists in this PR https://github.com/ikawrakow/ik_llama.cpp/pull/676?
I think I identified the issue, but I don't have a chance to run tests now

---

👤 **raidshoebox1** commented on **2025-08-08** at **01:49:32**

I tested [#676](https://github.com/ikawrakow/ik_llama.cpp/issues/676) and observed the same result as in this PR. The "Thinking" don't terminate.

---

👤 **iSevenDays** commented on **2025-08-08** at **08:43:06**

@raidshoebox1 I would be very thankful if you can test this branch https://github.com/iSevenDays/ik_llama.cpp/tree/deepseek-r1-parsing
I added additional tests, and that should work.

In case it doesn't, I think we have to revert this merge request, as I don't have other solutions available at the moment.

---

👤 **raidshoebox1** commented on **2025-08-08** at **09:15:23**

@iSevenDays Just tested this branch. It works perfectly now. Thanks for the fix!