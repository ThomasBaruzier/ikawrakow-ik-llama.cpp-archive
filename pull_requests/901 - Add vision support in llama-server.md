## 🔀 [Pull Request #901](https://github.com/ikawrakow/ik_llama.cpp/pull/901) - Add vision support in llama-server

| **Author** | `firecoperana` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `fcp/mtmd_llama_server` |
| **Target Branch** | `main` |
| **Created** | 2025-11-05 |
| **Updated** | 2025-11-07 |
| **Merged** | 2025-11-05 |

---

## 📄 Description

This PR adds vision support for llama-server. Both llama-server and mtmd are now up to PR [#16275](https://github.com/ikawrakow/ik_llama.cpp/issues/16275) (9/26/2025) in mainline. https://github.com/ggml-org/llama.cpp/pull/12898

Updated webui to support adding pictures and files and other minor ui changes. Test with both current webui and new llama.cpp webui (launched via --webui llamacpp ) using Qwen2.5-VL-7B-Instruct-Q8_0.gguf and mmproj-F16.gguf and both works fine. 

Note that when using --mmproj for vision model, context shift is disabled. 

Other changes:
1.  Fix KV shift for qwen2vl https://github.com/ggml-org/llama.cpp/pull/13870
2.  Fix the discrepancy between the usage of slot.id in llama-server. Before the fix, we need to use slot.id+1 when porting mainline's code where they use slot.id, which is very easy to miss. After the fix, one just needs slot.id instead of slot.id+1. https://github.com/ggml-org/llama.cpp/pull/10187
3. Add --no-context-shift parameter to disable context shift
4. Simplify handle_completions,  handle_completions_oai and handle_chat_completions function by using the common function handle_completions_impl to handle inference task. In my test, the results are the same as before, but best to have more people to test.

---

## 💬 Conversation

👤 **ikawrakow** approved this pull request ✅ on **2025-11-05** at **08:43:36**

---

👤 **ikawrakow** started a conversation on `examples/server/server.cpp` on **2025-11-05** at **08:53:06**

Missed this one in the review. What is the intent here? `fprintf` needs to have a format string for this to work.

> 👤 **firecoperana** replied on **2025-11-05** at **12:57:48**
> 
> Debug purpose. Should be removed.

---

👤 **ikawrakow** started a conversation on `examples/server/server.cpp` on **2025-11-05** at **08:53:50**

What is the intent here? `fprintf` needs to have a format string.

> 👤 **firecoperana** replied on **2025-11-05** at **12:58:17**
> 
> Debug purpose. Should be removed.

> 👤 **firecoperana** replied on **2025-11-05** at **14:15:05**
> 
> I will fix this in my next PR to address https://github.com/ikawrakow/ik_llama.cpp/issues/904

---

👤 **MrHills-2** commented on **2025-11-05** at **10:21:48**

I get an error when sending images (text works fine) 

add_text: <|vision_start|>
image_tokens->nx = 13
image_tokens->ny = 18                                                                                                                                                                                                batch_f32 size = 1
add_text: <|vision_end|>
add_text: <|im_end|>
<|im_start|>assistant                                                                                                                                                                                                                                                                                                                                                                                                                     n_pos: 1
VERB [              get_new_id] new task id | tid="139746469257216" timestamp=1762337477 new_id=376
VERB [     add_waiting_task_id] waiting for task id | tid="139746469257216" timestamp=1762337477 id_task=376
VERB [              start_loop] new task may arrive | tid="139756608065536" timestamp=1762337477
VERB [              start_loop] callback_new_task | tid="139756608065536" timestamp=1762337477 id_task=376
VERB [      get_available_slot] selected slot by lcp similarity | tid="139756608065536" timestamp=1762337477 id_slot=0 max_lcp_len=53753 similarity=1.0
INFO [   launch_slot_with_task] slot is processing task | tid="139756608065536" timestamp=1762337477 id_slot=0 id_task=376                                                                                           VERB [              start_loop] update_multitasks | tid="139756608065536" timestamp=1762337477                                                                                                                       VERB [              start_loop] callback_update_slots | tid="139756608065536" timestamp=1762337477                                                                                                                   VERB [            update_slots] posting NEXT_RESPONSE | tid="139756608065536" timestamp=1762337477                                                                                                                   VERB [                    post] new task id | tid="139756608065536" timestamp=1762337477 new_id=377                                                                                                                  VERB [            update_slots] tokenizing prompt | tid="139756608065536" timestamp=1762337477 id_slot=0 id_task=376
terminate called after throwing an instance of 'std::out_of_range'
what():  vector::_M_range_check: __n (which is 18446744073709551615) >= this->size() (which is 151936)                                                                                                             ./Qwen3_235B-IQ2M: line 4: 313100 Aborted                 (core dumped) ./llama-server -m ../../models/Qwen3-VL-235B-IQ2_M.gguf --mmproj ../../models/Qwen3-VL-235B-A22B-Instruct.mmproj-Q8_0.gguf -ot "blk\.(?:[0-9]|[1-5][0-9]|[6][0-5])\.ffn.*=CPU" -c 32768 -b 8192 -ub 4096 -v -ctk q8_0 -ctv q5_1 --threads 7 -ngl 95 -mla 2 -sp --host 0.0.0.0 --port 8080

---

👤 **Ph0rk0z** commented on **2025-11-05** at **12:06:01**

Thanks! time to download qwen 235b finally.

---

👤 **firecoperana** commented on **2025-11-05** at **17:41:13**

> I get an error when sending images (text works fine)
> 
> add_text: <|vision_start|> image_tokens->nx = 13 image_tokens->ny = 18 batch_f32 size = 1 add_text: <|vision_end|> add_text: <|im_end|> <|im_start|>assistant n_pos: 1 VERB [ get_new_id] new task id | tid="139746469257216" timestamp=1762337477 new_id=376 VERB [ add_waiting_task_id] waiting for task id | tid="139746469257216" timestamp=1762337477 id_task=376 VERB [ start_loop] new task may arrive | tid="139756608065536" timestamp=1762337477 VERB [ start_loop] callback_new_task | tid="139756608065536" timestamp=1762337477 id_task=376 VERB [ get_available_slot] selected slot by lcp similarity | tid="139756608065536" timestamp=1762337477 id_slot=0 max_lcp_len=53753 similarity=1.0 INFO [ launch_slot_with_task] slot is processing task | tid="139756608065536" timestamp=1762337477 id_slot=0 id_task=376 VERB [ start_loop] update_multitasks | tid="139756608065536" timestamp=1762337477 VERB [ start_loop] callback_update_slots | tid="139756608065536" timestamp=1762337477 VERB [ update_slots] posting NEXT_RESPONSE | tid="139756608065536" timestamp=1762337477 VERB [ post] new task id | tid="139756608065536" timestamp=1762337477 new_id=377 VERB [ update_slots] tokenizing prompt | tid="139756608065536" timestamp=1762337477 id_slot=0 id_task=376 terminate called after throwing an instance of 'std::out_of_range' what(): vector::_M_range_check: __n (which is 18446744073709551615) >= this->size() (which is 151936) ./Qwen3_235B-IQ2M: line 4: 313100 Aborted (core dumped) ./llama-server -m ../../models/Qwen3-VL-235B-IQ2_M.gguf --mmproj ../../models/Qwen3-VL-235B-A22B-Instruct.mmproj-Q8_0.gguf -ot "blk.(?:[0-9]|[1-5][0-9]|[6][0-5]).ffn.*=CPU" -c 32768 -b 8192 -ub 4096 -v -ctk q8_0 -ctv q5_1 --threads 7 -ngl 95 -mla 2 -sp --host 0.0.0.0 --port 8080

What is the resolution of the image you use and does it happen with all the images? I tried test-1.jpeg inside examples\mtmd folder and it's working. I used Qwen3-VL-235B-A22B-Instruct-UD-Q2_K_XL.gguf from unsloth and their mmproj-F16.gguf for mmproj.

---

👤 **MrHills-2** commented on **2025-11-05** at **18:24:40**

I'm using iq2_M from https://huggingface.co/mradermacher/Qwen3-VL-235B-A22B-Instruct-i1-GGUF with his f16 mmproj from here https://huggingface.co/mradermacher/Qwen3-VL-235B-A22B-Instruct-GGUF.

Using latest sillytavern, chat completion. I tried the same image you mentioned test1.jpeg but it still doesn't work.

---

👤 **Thireus** commented on **2025-11-05** at **18:48:08**

Which build or commit are you using?

---

👤 **MrHills-2** commented on **2025-11-05** at **18:54:23**

Current main 320fc60

---

👤 **firecoperana** commented on **2025-11-05** at **20:48:51**

Does adding --jinja help? Can you also try the built-in webui?

---

👤 **MrHills-2** commented on **2025-11-05** at **21:12:57**

--jinja does not help and the built in web ui gives me the same error. I also downloaded a different gguf (from unsloth) and it gives me the same error.

---

👤 **Ph0rk0z** commented on **2025-11-06** at **22:48:11**

<img width="892" height="875" alt="qwenvl" src="https://github.com/user-attachments/assets/e5b3ae14-9d30-4975-9a8b-352e5cf59a1f" />

In my case it's working but I need to try jinja or see what the template is doing as it's a bit cracked out. Goes 19t/s too.

---

👤 **MrHills-2** commented on **2025-11-06** at **22:51:38**

> <img alt="qwenvl" width="892" height="875" src="https://private-user-images.githubusercontent.com/59298527/511029114-e5b3ae14-9d30-4975-9a8b-352e5cf59a1f.png?jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NjI0Njk2NTgsIm5iZiI6MTc2MjQ2OTM1OCwicGF0aCI6Ii81OTI5ODUyNy81MTEwMjkxMTQtZTViM2FlMTQtOWQzMC00OTc1LTlhOGItMzUyZTVjZjU5YTFmLnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNTExMDYlMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjUxMTA2VDIyNDkxOFomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPWFlNTBhNjhkOWU3ODVlNmZlNjM4NzJiYTE1NWNhMjM2ODdkYTA1YmJkYzg5NDZkMzI2ZjgzMjc1NDYxNzllZjEmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.l2GWJR23Nen5T5GiSW1LT1ytPrvzhuDibb8TKk9VFH4">
> 
> In my case it's working but I need to try jinja or see what the template is doing as it's a bit cracked out. Goes 19t/s too.

Can you tell me what are your build arguments and what arguments you use on llama-server? I tried qwen3vl 235b, and also qwen3vl 30b, nothing works.

---

👤 **Ph0rk0z** commented on **2025-11-07** at **01:19:42**

Sure



```
CUDA_VISIBLE_DEVICES=0,1,2,3 numactl --interleave=all ./bin/llama-server \
    -m /Qwen3-VL-235B-A22B-Instruct-UD-Q4_K_XL-00001-of-00003.gguf \
    --mmproj /mmproj-F16.gguf \
    -t 48 \
    -c 32768 \
    --host 192.168.1.211 \
    --numa distribute \
    -ngl 95 \
    -ctk q8_0 \
    -ctv q8_0 \
    --jinja \
    -v \
    -rtr \
    -amb 512 \
    -ub 1024 \
    -mqkv \
    -ot "blk\.(0|1|2|3|4|5|6|7|8|9|10|11|12|13)\.ffn_.*.=CUDA0" \
    -ot "blk\.(14|15|16|17|18|19|20|21|22|23|24|25|26|27|28|29)\.ffn_.*.=CUDA1" \
    -ot "blk\.(30|31|32|33|34|35|36|37|38|39|40|41|42|43|44|45)\.ffn_.*.=CUDA2" \
    -ot "blk\.(46|47|48|49|50|51|52|53|54|55|56|57|58|59|60)\.ffn_.*.=CUDA3" \
    -ot "\.ffn_.*_exps.=CPU"
```