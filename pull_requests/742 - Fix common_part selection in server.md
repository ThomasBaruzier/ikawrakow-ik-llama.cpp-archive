## 🔀 [Pull Request #742](https://github.com/ikawrakow/ik_llama.cpp/pull/742) - Fix "common_part" selection in server

| **Author** | `saood06` |
| :--- | :--- |
| **State** | 📝 **Draft** |
| **Source Branch** | `s6/fix_prompt_tokenization` |
| **Target Branch** | `main` |
| **Created** | 2025-08-30 |
| **Updated** | 2025-10-05 |

---

## 📄 Description

This is my attempt to address https://github.com/ggml-org/llama.cpp/issues/11970 

It has some similarities to https://github.com/ggml-org/llama.cpp/pull/12067 but is not a port it is implemented differently.

It matches tokens in cache directly to the string passed in the prompt.

I do not know how to reliably reproduce the issue so I'm not sure if it fixes it.

@vnicolici Do you mind testing?

---

## 💬 Conversation

👤 **ikawrakow** commented on **2025-09-02** at **12:36:46**

@saood06 This is draft because it is not ready, or because you are not sure that it works?

---

👤 **saood06** commented on **2025-09-02** at **20:02:41**

@ikawrakow I think it is ready (it is fully implemented and conceptually makes sense to me).

This is meant to fix situations where TG will generate something in two tokens that PP will later consolidate in one which forces reprocessing from there, but I haven't tested by forcing the situation yet to see if my PR actually fixes that bug (I did compile and inference with it normally very briefly and that worked so there seems to be no regressions ).

Edit: To clarify my implementation matches the common part of the existing tokens in the cache against the prompt string directly. Before this the code would determine the slot by matching the common part of the prompt string and the cached part. But then only use the tokens after matching the common part of the tokens in the cache vs the tokenized version of the prompt, which in extreme situations could lead to far less reuse than just the expected issues with token and string boundaries.

---

👤 **ikawrakow** commented on **2025-09-03** at **05:37:39**

I understand the implementation and it LGTM.

But I wonder if it wouldn't be better to first try the tokens directly. If the full prompt is matched, there would be no need to do string matching, and the behavior will be the same as before. If we find a shorter match, one can do string matching from the point of disagreement.

---

👤 **saood06** commented on **2025-09-03** at **05:58:39**

>But I wonder if it wouldn't be better to first try the tokens directly.  If the full prompt is matched, there would be no need to do string matching, and the behavior will be the same as before. 

I can see how that might result in better performance for the average user.

>If we find a shorter match, one can do string matching from the point of disagreement.

But I don't think starting from the point of disagreement makes sense because if you compare tokens directly and everything matches you don't need to detokenize anything. In order to know where the point of disagreement is on the string wouldn't you need to do string matching, at which point comparing tokens becomes redundant.

---

👤 **ikawrakow** commented on **2025-09-03** at **14:39:08**

So, my understanding is that the `tokens -> text` conversion is not necessarily reversible, i.e., after `text -> tokens'`, `tokens'` may not be the same as `tokens`. But if `tokens[j] == tokens'[j]` up to `j_match`, then one can convert `tokens'[j_match...end]` to string and do string matching from there. No? This may not be more efficient than just matching strings as we now need two conversions (`text -> tokens'` and then `tokens'[j_match...] -> text'`), but is guaranteed to be not worse than the existing implementation.

Alternatively one can match tokens and strings, and one takes the result with the greater matched length.

All of this is only to avoid a potential failure mode in the string matching approach that we both don't see.

---

👤 **ikawrakow** approved this pull request ✅ on **2025-09-05** at **09:55:20**

---

👤 **saood06** commented on **2025-09-18** at **06:43:11**

Life got a bit hectic, sorry I haven't gotten back to this.

>So, my understanding is that the tokens -> text conversion is not necessarily reversible, i.e., after text -> tokens', tokens' may not be the same as tokens. But if tokens[j] == tokens'[j] up to j_match, then one can convert tokens'[j_match...end] to string and do string matching from there. No? This may not be more efficient than just matching strings as we now need two conversions (text -> tokens' and then tokens'[j_match...] -> text'), but is guaranteed to be not worse than the existing implementation.

Yes this could be implemented, as it would be equivalent but potentially faster.

---

👤 **usrlocalben** commented on **2025-10-05** at **19:40:57**

I tried this out. I finally hit a prompt that could benefit from this because the cache & tokenizer disagree only 20KT deep into a 100KT prompt.

I noticed that it refuses to reuse the prompt, starting at p0=0 each time.

I added some debug logging in the new common_part() and was surprised to see that the very first token (which I expected to be BOS or some special chat template token) was a token from well past the beginning of the prompt.

Could it be related to fragmentation? 🤔

---

👤 **usrlocalben** commented on **2025-10-05** at **19:46:44**

Thinking a moment more, I realize this may be unrelated to this patch as my sample doesn't have any output tokens, just 100K input tokens, so it shouldn't be due to the tokenizer/decode mismatch problem.

---

👤 **usrlocalben** commented on **2025-10-05** at **20:08:43**

I compared with main/HEAD as of now, and it appears to be a fault/regression in this PR.

my figures were off wrt. tokens but the gist is the same.

these are cold-starts / re-gen pairs for the same prompt

main/HEAD
```
INFO [            update_slots] kv cache rm [p0, end) | tid="140196405805056" timestamp=1759694453 id_slot=0 id_task=2 p0=65536
INFO [            update_slots] kv cache rm [p0, end) | tid="140196405805056" timestamp=1759694490 id_slot=0 id_task=2 p0=69632
INFO [            update_slots] kv cache rm [p0, end) | tid="140196405805056" timestamp=1759694528 id_slot=0 id_task=2 p0=73728
INFO [           print_timings] prompt eval time     =  580853.17 ms / 76425 tokens (    7.60 ms per token,   131.57 tokens per second) | tid="140196405805056" timestamp=1759694583 id_slot=0 id_task=2 t_prompt_processing=580853.171 n_prompt_tokens_processed=76425 t_token=7.600303186130192 n_tokens_second=131.5736985104623
INFO [           print_timings] generation eval time =   30540.66 ms /   403 runs   (   75.78 ms per token,    13.20 tokens per second) | tid="140196405805056" timestamp=1759694583 id_slot=0 id_task=2 t_token_generation=30540.659 n_decoded=403 t_token=75.7832729528536 n_tokens_second=13.195524038954103
INFO [           print_timings]           total time =  611393.83 ms | tid="140196405805056" timestamp=1759694583 id_slot=0 id_task=2 t_prompt_processing=580853.171 t_token_generation=30540.659 t_total=611393.83
INFO [            update_slots] slot released | tid="140196405805056" timestamp=1759694583 id_slot=0 id_task=2 n_ctx=131072 n_past=76827 n_system_tokens=0 n_cache_tokens=76827 truncated=true
INFO [            update_slots] all slots are idle | tid="140196405805056" timestamp=1759694583
INFO [      log_server_request] request | tid="140186308542464" timestamp=1759694583 remote_addr="127.0.0.1" remote_port=44778 status=200 method="POST" path="/v1/chat/completions" params={}
INFO [            update_slots] all slots are idle | tid="140196405805056" timestamp=1759694583
Grammar: 
Grammar lazy: false
Chat format: Content-only
INFO [   launch_slot_with_task] slot is processing task | tid="140196405805056" timestamp=1759694613 id_slot=0 id_task=425
INFO [            update_slots] we have to evaluate at least 1 token to generate logits | tid="140196405805056" timestamp=1759694613 id_slot=0 id_task=425
INFO [            update_slots] kv cache rm [p0, end) | tid="140196405805056" timestamp=1759694613 id_slot=0 id_task=425 p0=76424
INFO [           print_timings] prompt eval time     =     259.95 ms /     1 tokens (  259.95 ms per token,     3.85 tokens per second) | tid="140196405805056" timestamp=1759694639 id_slot=0 id_task=425 t_prompt_processing=259.954 n_prompt_tokens_processed=1 t_token=259.954 n_tokens_second=3.8468344399393737
INFO [           print_timings] generation eval time =   25420.14 ms /   336 runs   (   75.66 ms per token,    13.22 tokens per second) | tid="140196405805056" timestamp=1759694639 id_slot=0 id_task=425 t_token_generation=25420.139 n_decoded=336 t_token=75.65517559523809 n_tokens_second=13.217866353917263
INFO [           print_timings]           total time =   25680.09 ms | tid="140196405805056" timestamp=1759694639 id_slot=0 id_task=425
```


pr
```
INFO [            update_slots] kv cache rm [p0, end) | tid="140172995358720" timestamp=1759692609 id_slot=0 id_task=2 p0=57344
INFO [            update_slots] kv cache rm [p0, end) | tid="140172995358720" timestamp=1759692643 id_slot=0 id_task=2 p0=61440
INFO [            update_slots] kv cache rm [p0, end) | tid="140172995358720" timestamp=1759692677 id_slot=0 id_task=2 p0=65536
INFO [            update_slots] kv cache rm [p0, end) | tid="140172995358720" timestamp=1759692714 id_slot=0 id_task=2 p0=69632
INFO [            update_slots] kv cache rm [p0, end) | tid="140172995358720" timestamp=1759692752 id_slot=0 id_task=2 p0=73728
INFO [           print_timings] prompt eval time     =  579941.19 ms / 76425 tokens (    7.59 ms per token,   131.78 tokens per second) | tid="140172995358720" timestamp=1759692799 id_slot=0 id_task=2 t_prompt_processing=579941.191 n_prompt_tokens_processed=76425 t_token=7.588370179914949 n_tokens_second=131.7806032508562
INFO [           print_timings] generation eval time =   23004.48 ms /   303 runs   (   75.92 ms per token,    13.17 tokens per second) | tid="140172995358720" timestamp=1759692799 id_slot=0 id_task=2 t_token_generation=23004.476 n_decoded=303 t_token=75.92236303630362 n_tokens_second=13.171349784276766
INFO [           print_timings]           total time =  602945.67 ms | tid="140172995358720" timestamp=1759692799 id_slot=0 id_task=2 t_prompt_processing=579941.191 t_token_generation=23004.476 t_total=602945.667
INFO [      log_server_request] request | tid="140162879156224" timestamp=1759692799 remote_addr="127.0.0.1" remote_port=34622 status=200 method="POST" path="/v1/chat/completions" params={}
INFO [            update_slots] slot released | tid="140172995358720" timestamp=1759692799 id_slot=0 id_task=2 n_ctx=131072 n_past=76727 n_system_tokens=0 n_cache_tokens=76727 truncated=true
INFO [            update_slots] all slots are idle | tid="140172995358720" timestamp=1759692799
Grammar: 
Grammar lazy: false
Chat format: Content-only
INFO [   launch_slot_with_task] slot is processing task | tid="140172995358720" timestamp=1759692831 id_slot=0 id_task=325
INFO [            update_slots] kv cache rm [p0, end) | tid="140172995358720" timestamp=1759692831 id_slot=0 id_task=325 p0=0
INFO [            update_slots] kv cache rm [p0, end) | tid="140172995358720" timestamp=1759692857 id_slot=0 id_task=325 p0=4096
INFO [            update_slots] kv cache rm [p0, end) | tid="140172995358720" timestamp=1759692884 id_slot=0 id_task=325 p0=8192
INFO [            update_slots] kv cache rm [p0, end) | tid="140172995358720" timestamp=1759692911 id_slot=0 id_task=325 p0=12288
INFO [            update_slots] kv cache rm [p0, end) | tid="140172995358720" timestamp=1759692938 id_slot=0 id_task=325 p0=16384
INFO [            update_slots] kv cache rm [p0, end) | tid="140172995358720" timestamp=1759692966 id_slot=0 id_task=325 p0=20480
```