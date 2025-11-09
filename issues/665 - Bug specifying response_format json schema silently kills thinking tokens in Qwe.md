## 📌 [Issue #665](https://github.com/ikawrakow/ik_llama.cpp/issues/665) - Bug: specifying response_format json schema silently kills thinking tokens in Qwen3 thinking models

| **Author** | `SlapDrone` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-08-01 |
| **Updated** | 2025-09-26 |

---

## 📄 Description

### What happened?

Setup: built from source, main branch, hosting the model via llama-server like so:

```bash
llama-server -m models/Qwen3-30B-A3B-Thinking-2507-IQ5_K.gguf \
-t 24 \
-c 65536 \
-b 4096 \
-ub 4096 \
-fa \
-ot "blk\\.[0-2].*\\.ffn_.*_exps.weight=CUDA0" \
-ot "blk\\..*\\.ffn_.*_exps.weight=CPU" \
-ngl 99 \
-sm layer \
-ts 1 \
-amb 2048 \
-fmoe \
--top-k 20 \
--min-p 0
```

My use case requires reasoning model + structured generation. vLLM exposes a `--reasoning-parser` which when set correctly allows the backend to smartly apply the structured generation constraints to the model output, i.e. after its generated the <think>...</think> CoT.

It seems that mainline llamacpp can do something similar by using the `--jinja` argument with `--chat-template` or `--reasoning-format`. The necessary template to achieve this behaviour should already be present in the GGUF file afaik.

ik_llamacpp doesn't seem to support these arguments, at least not in the same way. However, I can still enforce a JSON schema at request-time. In this case, the whole response is constrained, thus nuking the thinking tags.

Here is a [standalone gist](https://gist.github.com/SlapDrone/510ae0af4b5c5be83c56755732f32995) for a minimal reproduction with outputs.

Is this intended behaviour? Perhaps it should at the very least raise a warning if the response_format violates the model-supplied template. Is there a workaround for now that enables both behaviours (thinking + constraining the non-thinking output)?

Thanks for any inputs.

### Name and Version

version: 3826 (ae0ba31f)
built with cc (GCC) 15.1.1 20250425 for x86_64-pc-linux-gnu


### What operating system are you seeing the problem on?

Linux

### Relevant log output

```shell

```

---

## 💬 Conversation

👤 **ikawrakow** commented on **2025-08-01** at **14:54:17**

Support for jinja templates was added to llama.cpp well after I forked the project. I think the reasoning related arguments are also quite recent. The two code bases are now so massively diverged that nothing that is being done in mainline is just a matter of copy/paste, and requires a significant effort to port over. My main interests have been computational efficiency and SOTA quantization. Hence, none of this is available here.

If you have been successfully using vLLM, why are you looking into ik_llama.cpp?

---

👤 **SlapDrone** commented on **2025-08-01** at **15:18:42**

Hey, thanks for your response.

Thought i'd switch to gain the ability to run large MoE models with offloading. Using a server board with tons of system RAM and cores to play with, but where VRAM is too limited for running large models on GPU with vLLM. Throughput/quality are both very important in my use-case (I need a smarter model than what I can run on vLLM alone to even get "good enough" outputs), and I gather that this project represents the bleeding edge of perf for quantized models.

I will make a quick browse to get a feel for how divergent this feature is, as it would be really nice to have.

---

👤 **ikawrakow** commented on **2025-08-01** at **15:38:37**

Thanks! This project can use any help it can get.

---

👤 **firecoperana** commented on **2025-09-02** at **15:10:27**

@SlapDrone Can you try the latest main? The support for `--jinja` argument with `--chat-template` or `--reasoning-format` was recently merged.

---

👤 **ikawrakow** commented on **2025-09-26** at **16:57:03**

No user response for >1 month -> closing