## 📌 [Issue #785](https://github.com/ikawrakow/ik_llama.cpp/issues/785) - Feature Request: Add `--device` and `--device-draft` parameters

| **Author** | `MRGRD56` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-09-18 |
| **Updated** | 2025-10-27 |
| **Labels** | `enhancement` |

---

## 📄 Description

### Prerequisites

- [x] I am running the latest code. Mention the version if possible as well.
- [x] I carefully followed the [README.md](https://github.com/ggerganov/llama.cpp/blob/master/README.md).
- [x] I searched using keywords relevant to my issue to make sure that I am creating a new issue that is not already open (or closed).
- [x] I reviewed the [Discussions](https://github.com/ggerganov/llama.cpp/discussions), and have a new and useful enhancement to share.

### Feature Description

With the original llama.cpp, I can specify which device(s) the model will be running on, using `-dev`/`--device` and `-devd`/`--device-draft`.  
This way, using speculative decoding, I can run the target model on `CUDA0` and the draft model on `CUDA1`, making use of 2 GPUs. Like this:

```powershell
C:\apps\llama.cpp\llama-server.exe --port 15900 `
      --model "E:\AI\LLM\gguf\unsloth\Qwen3-235B-A22B-Instruct-2507-GGUF\Qwen3-235B-A22B-Instruct-2507-UD-Q4_K_XL-00001-of-00003.gguf" `
      --ctx-size 16384 `
      -ctk q8_0 -ctv q8_0 `
      -fa on `
      --n-gpu-layers 999 `
      -mg 0 `
      -dev CUDA0 `
      -ot "blk\.(0?[0-9]|1[0-6])\.ffn_.*_exps.=CUDA0" `
      -ot ".ffn_.*_exps.=CPU" `
      --parallel 1 `
      -tb 30 -t 15 `
      --jinja --reasoning-budget 0 --no-mmap `
      --model-draft "E:\AI\LLM\gguf\unsloth\Qwen3-4B-Instruct-2507-GGUF\Qwen3-4B-Instruct-2507-UD-Q6_K_XL.gguf" `
      -devd CUDA1 `
      -ngld 999 `
      --ctx-size-draft 16384 `
      -ctkd q8_0 -ctvd q8_0
```

If anything, my GPUs are different: RTX 5090 (32GB) and RTX 4070 Ti Super (16GB).

In ik_llama.cpp, as far as I understand, there's no way to specify which GPUs to use, except this environment variable `CUDA_VISIBLE_DEVICES`. But it doesn't let you use different cards for the target and the draft model.

### Motivation

Using different GPUs for the target and the draft model when using speculative decoding, making it more effective.

### Possible Implementation

_No response_

---

## 💬 Conversation

👤 **saood06** commented on **2025-09-18** at **06:38:33**

I made an attempt at this a while ago posted the code diff in [this](https://github.com/ikawrakow/ik_llama.cpp/pull/645#issuecomment-3162600584) comment along with my thoughts, this could be useful for someone attempting this feature, I do not think I will have time in the near future to work on it though.

---

👤 **firecoperana** commented on **2025-10-21** at **21:12:29**

I can try this.

---

👤 **firecoperana** commented on **2025-10-25** at **15:00:43**

https://github.com/ikawrakow/ik_llama.cpp/pull/866

---

👤 **ikawrakow** commented on **2025-10-27** at **12:20:24**

@MRGRD56 and people upvoting this issue: please test [#866](https://github.com/ikawrakow/ik_llama.cpp/issues/866)