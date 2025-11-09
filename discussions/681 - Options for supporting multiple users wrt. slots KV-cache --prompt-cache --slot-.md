## 🗣️ [Discussion #681](https://github.com/ikawrakow/ik_llama.cpp/discussions/681) - Options for supporting multiple users wrt. slots, KV-cache, --prompt-cache, --slot-save

| **Author** | `usrlocalben` |
| :--- | :--- |
| **State** | ✅ **Open** |
| **Created** | 2025-08-08 |
| **Updated** | 2025-09-09 |

---

## 📄 Description

--prompt-cache doesn't seem to do anything, despite having some implementation that reads and populates the value
--slot-save doesn't seem to do anything, and reading the /slot/ handlers I'm not even sure how it's intended to be used.
With -np 1, two users clobber each other's KV-cache, even if users don't mind waiting for a request to be filled. (e.g. 10s of minutes to rebuild a deep ctx slot)
Can KV-cache be (slot-wise) split across CUDA-devs?

Currently it seems like the only way to get two non-clobbering slots would be to avoid GPU use and go CPU only where there's "unlimited" ctx space.

What's the state of affairs wrt. these params and what options are there?

Could an LRU-cache of slots be implemented?

CPU backends don't offer any meaningful concurrency, but swapping out slots would go a long way.

Have I missed something? 😅

---

## 💬 Discussion

👤 **saood06** commented on **2025-08-08** at **23:38:31**

What do you mean "--slot-save doesn't seem to do anything". I find it really useful, and am glad I made it work with MLA models as well (see [#497](https://github.com/ikawrakow/ik_llama.cpp/issues/497)), and did and have a UI for managing it with mikupad (still a WIP but screenshots here https://github.com/ikawrakow/ik_llama.cpp/pull/558#issuecomment-3115444257)

I do agree the state of KV state management has a lot of potential upgrades, I had being able to automatically use any saves in the `--slot-save-path` as cache (similar to how it caches the prompt now from the RAM). Creating those save files would still be manual (a frontend has the tools to automate it though). Since there is interest I'll move it up the list of things I plan to do.

> 👤 **usrlocalben** replied on **2025-08-12** at **00:38:13**
> 
> I've been reading over slot save/restore as I think about a slot disk/mem spill + LRU.
> 
> Two things strike me as odd.
> 
> 1. Due to the asynchrony, there's a race wrt. the state of the slot between time-of-request and time-of-action. One might examine the slot content, then issue save/restore and either get a surprising save-state (slot was modified before save) or accidental invalidation (less harmful, potential delays).  save could take the last id_task as a param and give HTTP 409 if upon filling the request the slot's id_task doesn't match.
> 
> 2. Loading the slot only loads the KV and token list. The prompt text (string) is not restored. This means the current LRU impl. won't match it since it compares by prompt text, so reuse would only occur by the nature of having one slot to choose from. (or issuing restores on all slots)
> 
> It seems like either a) rebuilding the prompt text from tokens, or b) storing the prompt string with the slot file would set up the slot functions as being nearly ready to build an LRU spill file.

> 👤 **saood06** replied on **2025-08-12** at **01:00:16**
> 
> Thank you for the thoughtful reply.
> 
> >Loading the slot only loads the KV and token list. The prompt text (string) is not restored. This means the current LRU impl. won't match it since it compares by prompt text, so this system would only work by the nature of having one slot to choose from. (or issuing restores on all slots)
> 
> Yes, but from my perspective it is the responsibility of the application to manage the state of the prompt. I don't think what you are describing is a roadblock from this being implemented. You can already do this manually by loading from a slot then sending your prompt, but this would just mean the slot will do the restore for you (there may be extremely fringe situations where it results in performance degradation but rare enough that I think it can still be on by default).
> 
> I did recently add a way to peer into what is stored in the so that a more informed decision can be made in my mikupad PR (code [here](https://github.com/ikawrakow/ik_llama.cpp/blob/4a5695ef6fe58125cbeceb165697792bc510a9ff/examples/server/server.cpp#L3797C66-L3797C67)). 
> 
> In my mikupad PR I was considering automatically pulling the prompt into the currently editing document but ultimately that felt invasive (what if you had a draft in there), I do feel like a copy or insert in button would work but for now copy and paste works and feels fine.
> 
> The new UI I pushed yesterday integrates slot and cache management, I would really appreciate if you tried it, because although there are improvements (like discussed above), the new stuff in mikupad works well and does the basics at addressing this problem.
> 
> >b) storing the prompt string with the slot file would set up the slot functions as being nearly ready to build an LRU spill file.
> 
> That already exists? Look at the outputs of the endpoint added in [#502](https://github.com/ikawrakow/ik_llama.cpp/issues/502) (again this has been updated in my mikupad PR with more stuff see the code for that version [here](https://github.com/ikawrakow/ik_llama.cpp/blob/4a5695ef6fe58125cbeceb165697792bc510a9ff/examples/server/server.cpp#L3736))
> 
> On related topics, I was thinking about this as well as https://github.com/ggml-org/llama.cpp/pull/12067.

> 👤 **usrlocalben** replied on **2025-09-08** at **16:24:32**
> 
> https://github.com/usrlocalben/ik_llama.cpp/commit/78ca4bddc2e1fd5df7339bd535b1cdaeda0cefd8
> 
> I used your slot_save/load to implement a disk LRU cache, although I didn't actually bother with the "LRU" part so it's just a disk spool.
> 
> I also repurposed _slot-save-path_.
> 
> It solves my problem (multi-user/chat with long context) but I don't intend to promote it as a PR, at least for now.
> 
> Also yes, I have been following the KV-based-tokenization and find it interesting. I'll probably test your PR at some point soon.

---

👤 **magikRUKKOLA** commented on **2025-09-09** at **01:00:26**

As related to the KV-cache.  I just recenly realized that ik_llama.cpp does have the RPC to store/restore the KV-cache.  The options that I am using right now are:

```
     --prompt-cache "$HOME/.cache/ik_llama.cpp/prompt-cache.bin" --prompt-cache-all \
     --slot-save-path "$HOME/.cache/ik_llama.cpp/slot.bin" \
     --lookup-cache-dynamic "$HOME/.cache/ik_llama.cpp/slot.bin" \
     --keep -1 \
     --slot-prompt-similarity 0.35 \
```

Very sadly, I do not understand what they are and how they work.  I tried to find the documentation regarding them, but I failed.  At least, I was able to do the basic functionality of saving and restoring the KV-cache for the system with multiple GPUs.  Its working.  And the KV-cache itself is kinda compressed/optimal (unlike what is done in ktransformers for example, where you have to employ the ZFS etc).

I think there should be just three options.
1.) path where to store the KV-cache
2.) the limit in GB how much data would be used for the KV-cache
3.) autosave on/off

The software should estimate how long it takes to store/restore the KV-cache, what is the approx. prefill speed of the current setup and based on that, it should determine the minimal length of the prompt for which the KV-cache should be swapped to the storage device to be automatically restored later on.  The old/least used dumps are automatically deleted if the storage quota is exhausted.

This is all anyone would ever need from such a system.  What are the state of affairs regarding this?  The exposed RPC for us to trigger it and do the slot management?
Why not to implement just three abovementioned options?  Too much hustle?

> 👤 **usrlocalben** replied on **2025-09-09** at **12:14:29**
> 
> try my [spool-to-disk mod](https://github.com/usrlocalben/ik_llama.cpp/commit/78ca4bddc2e1fd5df7339bd535b1cdaeda0cefd8).
> 
> all it does is spool all kv-sessions to a dir (I repurposed slot-save-path for this).
> on start, it looks for the longest prefix match to reuse.
> on finish it tries to append a new suffix if it finds a match.
> 
> use crontab to do GC if you need it.
> 
> even without any interesting algorithm for the query, the lookup time is basically free relative to the PP time for e.g. 100K tokens. loading a 100K token slot (K2, F16) takes a couple seconds from ssd (less from tmpfs!)
> 
> I understand that multiuser doesn't seem to be in-scope for (ik_)llama.cpp, but it puzzles me this isn't already a feature.

> 👤 **magikRUKKOLA** replied on **2025-09-09** at **12:37:25**
> 
> @usrlocalben 
> 
> > I understand that multiuser doesn't seem to be in-scope for (ik_)llama.cpp, but it puzzles me this isn't already a feature.
> 
> Its not really about the multiuser stuff.  In the agentic workflows the code that is sent to the prefill most of the times is constant.  So that constant part could be stored and reused later on at some point.
> It puzzles me as well why its not the feature.
> You pointed out that:
> 
> ```
> not implemented:
> - no GC/LRU as it could be handled with cron etc.
> - no guard against config / KV-blob mismatch.
> - no awareness of chat-tempalte when extending
> ```
> 
> I believe this should be implemented and merged into the master.  Its a very useful feature.