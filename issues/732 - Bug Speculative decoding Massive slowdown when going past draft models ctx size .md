## 📌 [Issue #732](https://github.com/ikawrakow/ik_llama.cpp/issues/732) - Bug: (Speculative decoding) Massive slowdown when going past draft model's ctx size (when -cd < -c)

| **Author** | `alint77` |
| :--- | :--- |
| **State** | ✅ **Open** |
| **Created** | 2025-08-26 |
| **Updated** | 2025-09-04 |

---

## 📄 Description

### What happened?

When using speculative decoding in llama-server, when specifying different context sizes for the target model (-c) and the draft model (-cd), with the draft context being smaller than the target (e.g. -cd < -c). At the start of generation, performance is excellent, high throughput and low GPU power consumption. But once the draft fills its context window, token generation slows drastically (lower than half) and GPU power draw increases.

This bug is also present in the mainline llama.cpp. I know there's some kind of sliding window mechanism going on in this situation and clearly there's too much overhead when trying to pass only the latest {cd} tokens to the draft model. I tried to fix the code myself but wasn't successful.

I think fixing this would greatly improve the feasability of using SD on llama.cpp in general, because the small model doesn't really need a big ctx window to come up with high quality draft tokens and running the draft model with same context size as the target model is a complete waste of precious VRAM. 

in my testing on qwen3 30B coder on a partially offloaded setting, running the qwen3 0.6B q4KM draft model with -ctkd q8_0 -ctvd q8_0 -cd 1024 has pretty much the same acceptance rate and speedup as running with no -cd and it only uses around 1.5GB extra vram (compared to no draft model) for 1.2-1.8x speedup in TG. I'd imagine the gains to be even more pronounced with bigger models... 

### Name and Version

./llama-server --version
version: 3860 (0cc32ff0)
built with cc (Ubuntu 13.3.0-6ubuntu2~24.04) 13.3.0 for x86_64-linux-gnu

### What operating system are you seeing the problem on?

Linux

### Relevant log output

```shell

```

---

## 💬 Conversation

👤 **ikawrakow** commented on **2025-08-28** at **11:13:48**

Yes, I know, KV cache handling could use some love in `ik_llama.cpp`. But I also see that mainline developers have spent a huge amount of time totally rewriting KV cache handling in `llama.cpp`, so I'm surprised it is still inadequate when the context becomes full.

---

👤 **nimishchaudhari** commented on **2025-08-28** at **16:31:42**

Hello, 

Not sure if I'm also talking about the same thing, but till day before yesterday I was getting some beefed up speeds using a draft model ( https://github.com/ikawrakow/ik_llama.cpp/pull/677#issuecomment-3229022320 I only posted about my speeds yesterday )


Today after git pull + rebuilding I'm getting much much lower speeds for the SAME config 

Today:
Prompt
- Tokens: 27
- Time: 900.123 ms
- Speed: 30.0 t/s
Generation
- Tokens: 37
- Time: 13573.686 ms
- Speed: 2.7 t/s

Day before yesterday:
Tokens: 1
**- Time: 103.512 ms
Speed: 9.7 t/s**
Generation
Tokens: 3537
Time: 338829.416 ms
Speed: 10.4 t/s


Without Draft model:

Prompt
- Tokens: 27
- Time: 11474.484 ms
- Speed: 2.4 t/s
Generation
- Tokens: 48
- Time: 8327.126 ms
- Speed: 5.8 t/s




Draft model's speed has increased but tg is poor. 

Does anyone else confirm this slowdown ?

---

👤 **ChicoPinto70** commented on **2025-09-01** at **15:55:28**

> Draft model's speed has increased but tg is poor.
> 
> Does anyone else confirm this slowdown ?

Today, I updated ik_llama.cpp and I noticed this slowdown in spec dec too (in GLM 4.5).

---

👤 **DocShotgun** commented on **2025-09-03** at **03:40:28**

> When using speculative decoding in llama-server, when specifying different context sizes for the target model (-c) and the draft model (-cd), with the draft context being smaller than the target (e.g. -cd < -c). At the start of generation, performance is excellent, high throughput and low GPU power consumption. But once the draft fills its context window, token generation slows drastically (lower than half) and GPU power draw increases.

My suspicion is that the draft model starts generating gibberish once you exceed `-cd`, therefore token gen speed drops due to no drafts being accepted.

I'm not sure what the correct behavior is when the context exceeds `-cd` but is less than `-c`... would the earliest tokens (sysprompt, earlier convo turns, etc.) be simply dropped from the draft model's context?

---

👤 **nimishchaudhari** commented on **2025-09-04** at **09:18:53**

In my case I didn't set the draft model's context length, default binding is the same ctx size as the model's ctx size.