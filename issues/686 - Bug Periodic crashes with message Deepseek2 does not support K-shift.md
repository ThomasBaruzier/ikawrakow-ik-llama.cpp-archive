## 📌 [Issue #686](https://github.com/ikawrakow/ik_llama.cpp/issues/686) - Bug: Periodic crashes with message "Deepseek2 does not support K-shift"

| **Author** | `Lissanro` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-08-12 |
| **Updated** | 2025-08-15 |
| **Labels** | `enhancement` |

---

## 📄 Description

### What happened?

Sometimes, especially when using Cline, the backend crashes with message "Deepseek2 does not support K-shift".

I understand that it does not support K-shift, but in such a case I think it could just reevaluate context normally, probably some part of it will be a cache hit but even if not, it still would be better without crashing, maybe display it as warning instead. This because these crashes break agentic workflow, not allowing to continue automatically.

Please let me know if I missed something and if it is misconfiguration on my side (like missing some command line option). This is how I run it:

```
numactl --cpunodebind=0 --interleave=all ~/pkgs/ik_llama.cpp/build/bin/llama-server \
--model /mnt/neuro/models/DeepSeek-R1-256x21B-0528-IQ4_K-163840seq/DeepSeek-R1-256x21B-0528-IQ4_XS-163840seq.gguf \
--ctx-size 102400 --n-gpu-layers 62 --tensor-split 15,25,30,30 -mla 3 -fa -ctk q8_0 -amb 1024 -fmoe -b 4096 -ub 4096 \
-ot "blk\.3\.ffn_up_exps=CUDA0, blk\.3\.ffn_gate_exps=CUDA0, blk\.3\.ffn_down_exps=CUDA0" \
-ot "blk\.4\.ffn_up_exps=CUDA1, blk\.4\.ffn_gate_exps=CUDA1, blk\.4\.ffn_down_exps=CUDA1" \
-ot "blk\.5\.ffn_up_exps=CUDA2, blk\.5\.ffn_gate_exps=CUDA2, blk\.5\.ffn_down_exps=CUDA2" \
-ot "blk\.6\.ffn_up_exps=CUDA3, blk\.6\.ffn_gate_exps=CUDA3, blk\.6\.ffn_down_exps=CUDA3" \
-ot "ffn_down_exps=CPU, ffn_up_exps=CPU, gate_exps=CPU" \
--threads 64 --host 0.0.0.0 --port 5000
```

### Name and Version

Latest git

### What operating system are you seeing the problem on?

Linux

### Relevant log output

```shell
...
INFO [           print_timings]           total time =   70700.13 ms | tid="134518025084928" timestamp=1754958412 id_slot=0 id_task=8124 t_prompt_processing=48951.336 t_token_generation=21748.792 t_total=70700.128
INFO [            update_slots] slot released | tid="134518025084928" timestamp=1754958412 id_slot=0 id_task=8124 n_ctx=102400 n_past=99887 n_system_tokens=0 n_cache_tokens=99887 truncated=false
INFO [            update_slots] all slots are idle | tid="134518025084928" timestamp=1754958412
INFO [format_partial_response_oaicompat] DEBUG: Streaming finish_reason check | tid="134067588681728" timestamp=1754958412 generated_text="<think>\n<attempt_completion>\n<result>\nI've made the following improvements:\n\n1. Added Space Grotesk font to all text elements in header\n2. Ensured consistent use of Tailwind color classes\n3. Preserved all animations and hover effects\n4. Fixed font family issues\n\nThe design should now much more closely match the reference. You can run the development server to verify.\n</result>\n<command>yarn dev</command>\n</attempt_completion>" model_name="x" tool_calls_count=0
INFO [      log_server_request] request | tid="134067588681728" timestamp=1754958412 remote_addr="127.0.0.1" remote_port=49610 status=200 method="POST" path="/v1/chat/completions" params={}
INFO [            update_slots] all slots are idle | tid="134518025084928" timestamp=1754958412
INFO [   launch_slot_with_task] slot is processing task | tid="134518025084928" timestamp=1754959245 id_slot=0 id_task=8225
INFO [            update_slots] kv cache rm [p0, end) | tid="134518025084928" timestamp=1754959246 id_slot=0 id_task=8225 p0=99887
INFO [            update_slots] slot context shift | tid="134518025084928" timestamp=1754959375 id_slot=0 id_task=8225 n_keep=1 n_left=102398 n_discard=51199 n_ctx=102400 n_past=102399 n_system_tokens=0 n_cache_tokens=102399
/home/lissanro/pkgs/ik_llama.cpp/src/llama.cpp:18969: Deepseek2 does not support K-shift
```

---

## 💬 Conversation

👤 **saood06** commented on **2025-08-12** at **01:11:41**

>Please let me know if I missed something and if it is misconfiguration on my side (like missing some command line option). This is how I run it:

Unfortunately it is not. As I mentioned [here](https://github.com/ikawrakow/ik_llama.cpp/issues/437#issuecomment-2955189143) it sucks.

I know for a while I used [#290](https://github.com/ikawrakow/ik_llama.cpp/issues/290) to allow me to set KV size absurdly high, this way context overflow resulted in paging and bad performance not crashing, but that PR is so old now.

---

👤 **ikawrakow** commented on **2025-08-12** at **05:16:57**

It is known that K-shift does not work for DeepSeek models, so I would consider this a feature request rather than a bug.

---

👤 **saood06** commented on **2025-08-12** at **05:29:36**

>It is known that K-shift does not work for DeepSeek models, so I would consider this a feature request rather than a bug.

The solution doesn't have to be to implement it, it would be just as well solved by handling it more gracefully, and not crashing.

---

👤 **ikawrakow** commented on **2025-08-12** at **05:35:11**

So, the feature request is:

Implement K-shift for DeepSeek models. If this is not possible/too hard, handle the situation gracefully by, e.g., recomputing the cache for the new context.

---

👤 **saood06** commented on **2025-08-12** at **05:38:25**

>Implement K-shift for DeepSeek models. If this is not possible/too hard, handle the situation gracefully by, e.g., recomputing the cache for the new context.

Or just give empty or error responses (could be something about n_ctx has been hit). This allows the user to either save the cache to disk and relaunch with higher context, or manually trim the context so that their request and it's reply can fit.

---

👤 **Lissanro** commented on **2025-08-12** at **22:30:49**

@ikawrakow 
> If this is not possible/too hard, handle the situation gracefully by, e.g., recomputing the cache for the new context

Yes, that's what I meant, because when it crashes I not just lose the whole context - entire workflow breaks, and cannot continue automatically, so if I left it unattended, I end up finding it stopped early. Cline for example likes to crash ik_llama.cpp often this way, ignoring its context limit setting or maybe just assuming that it can context shift. In some cases it crashes again after restart, so I end up recomputing cache multiple times in addition to the need to intervene malually. But if it just recomputed it automatically, even if K-shift still not implemented, it would have been great improvement for use cases like this.

My apologies if it was supposed to be filed as feature request, I filed it as a bug because it crashes and my intention was to suggest graceful handling rather than requesting actual K-shift implementation.

@saood06
> save the cache to disk and relaunch with higher context

This sounds interesting, but is this possible? I mean, saving cache to disk and then reusing it on the next llama-server launch. This is something I tried to look into in the past, but did not find a solution - otherwise, could be useful to have cache saved for my some frequently used long prompts.

---

👤 **saood06** commented on **2025-08-13** at **00:58:10**

> [@saood06](https://github.com/saood06)
> 
> > save the cache to disk and relaunch with higher context
> 
> This sounds interesting, but is this possible? I mean, saving cache to disk and then reusing it on the next llama-server launch. This is something I tried to look into in the past, but did not find a solution - otherwise, could be useful to have cache saved for my some frequently used long prompts.

Yes this is very possible (and a feature I use regularly) by passing `--slot-save-path` to server.

There are two ways to make use of it, the manual way is to manage it with calls like:
```
curl --header "Content-Type: application/json" \
   --request POST \
   --data '{"filename":"test.bin"}' [...]:8080/slots/0?action=save
```
and
```
curl --header "Content-Type: application/json" \
   --request POST \
   --data '{"filename":"test.bin"}' [...]:8080/slots/0?action=restore
```
alongside
using `/list` to view what is saved

There is an automatic way with a GUI via the mikupad PR I have open (it is based on old code and has not been merged with main in a while so some model support/new stuff may be missing) via [#558](https://github.com/ikawrakow/ik_llama.cpp/issues/558) which I would really appreciate some feedback of (you don't even have to use mikupad for inference you can just use it to manage the prompts saved in disk or memory) this also updates `/list` with more metadata and adds `/slots/list` which displays what is currently in the slots memory.

---

👤 **Lissanro** commented on **2025-08-13** at **10:06:20**

@saood06 
> manual way is to manage it with calls

Wow, thank you so much, I did not know about this method. I used option --slot-save-path and it seems to save the cache. I will test this out more thoroughly later, it opens up a lot of possibilities from alternating models more often to advanced workflows with model switching without losing the cache for each. Awesome!

And many thanks for coming up with the patch for this issue too!

---

👤 **saood06** commented on **2025-08-13** at **10:24:10**

>I will test this out more thoroughly later

The GUI in [#558](https://github.com/ikawrakow/ik_llama.cpp/issues/558) makes it really easy to view/save/load the saved and in memory cache, going back to the manual route to test the fix for this was much less convenient than the GUI I used to create a reproduction of the issue. It will be nice when I finish that so that it is merged in.

> it opens up a lot of possibilities from alternating models more often to advanced workflows with model switching without losing the cache for each. Awesome!

It opened up possibilities for me, especially since I have extremely low PP of 10 t/s at 0 context. I'm sure it will for you as well, especially with your faster disks (my server currently only has HDDs and can only access my network shares at gigabit speeds).



> And many thanks for coming up with the patch for this issue too!

Your welcome, even though I did take the easy fix and didn't implement K-shift for MLA. Like I said earlier, this bug annoyed me as well.

---

👤 **saood06** commented on **2025-08-15** at **02:20:32**

@Lissanro 

>This because these crashes break agentic workflow, not allowing to continue automatically.
>But if it just recomputed it automatically, even if K-shift still not implemented, it would have been great improvement for use cases like this.

My fix doesn't actually address this. There are a lot of potential choices on what to do next, and none of them are actually applied automatically. It just stops processing as the n_ctx has been hit instead of crashing.

For example one way you could continue with much higher context in the same  (with crippled PP performance) is to save the KV cache to a file, kill the server, relaunch it but remove `-b 4096 -ub 4096 -amb 1024` and replace it with `-b 32 -ub 1` (`n_batch is less than GGML_KQ_MASK_PAD - increasing to 32` means 32 is the lowest it can be set to ), and a much higher context value. Restore the KV cache from disk and TG speed will be fine (and the restore will put everything in the cache from when Cline hit the old cap) .

This is a workflow I probably will end up using myself now to stretch the limits of KV cache as the last parts of the KV cache are pretty much always TG (besides maybe like building system prompts to save to disk for later use to ultimately end up doing TG).