## 📌 [Issue #693](https://github.com/ikawrakow/ik_llama.cpp/issues/693) - Bug: gpt-oss chat template error when using --jinja flag.

| **Author** | `genericgod` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-08-15 |
| **Updated** | 2025-09-01 |
| **Labels** | `help wanted` |

---

## 📄 Description

### What happened?

With `llama-server --jinja` in combination with the gpt-oss-20B model I get a chat template error. It then falls back to chatml which causes the model to produce nonsense.

I am using gpt-oss-20b-UD-Q6_K_XL.gguf from unsloth but loading the official chat template from OpenAI with `--chat-template-file` causes the same issue.



### Name and Version

version: 3838 (fc06bc9d)
built with cc (Ubuntu 14.2.0-4ubuntu2) 14.2.0 for x86_64-linux-gnu

### What operating system are you seeing the problem on?

Linux

### Relevant log output

```shell
ERR [validate_model_chat_template] failed to apply template: %s
 | ="Object is not callable: null at row 203, column 40:\n    {{- \"Knowledge cutoff: 2024-06\\n\" }}\n    {{- \"Current date: \" + strftime_now(\"%Y-%m-%d\") + \"\\n\\n\" }}\n                                       ^\n    {%- if reasoning_effort is not defined %}\n at row 203, column 52:\n    {{- \"Knowledge cutoff: 2024-06\\n\" }}\n    {{- \"Current date: \" + strftime_now(\"%Y-%m-%d\") + \"\\n\\n\" }}\n                                                   ^\n    {%- if reasoning_effort is not defined %}\n at row 203, column 62:\n    {{- \"Knowledge cutoff: 2024-06\\n\" }}\n    {{- \"Current date: \" + strftime_now(\"%Y-%m-%d\") + \"\\n\\n\" }}\n                                                             ^\n    {%- if reasoning_effort is not defined %}\n at row 203, column 5:\n    {{- \"Knowledge cutoff: 2024-06\\n\" }}\n    {{- \"Current date: \" + strftime_now(\"%Y-%m-%d\") + \"\\n\\n\" }}\n    ^\n    {%- if reasoning_effort is not defined %}\n at row 197, column 37:\n{#- System Message Construction ============================================ #}\n{%- macro build_system_message() -%}\n                                    ^\n    {%- if model_identity is not defined %}\n at row 231, column 25:\n{{- \"<|start|>system<|message|>\" }}\n{{- build_system_message() }}\n                        ^\n{{- \"<|end|>\" }}\n at row 231, column 1:\n{{- \"<|start|>system<|message|>\" }}\n{{- build_system_message() }}\n^\n{{- \"<|end|>\" }}\n at row 1, column 37:\n{# Chat template fixes by Unsloth #}\n                                    ^\n{#-\n"
WARN [              load_model] %s: The chat template that comes with this model is not yet supported, falling back to chatml. This may cause the model to output suboptimal responses
```

---

## 💬 Conversation

👤 **ikawrakow** commented on **2025-08-15** at **15:00:17**

Does it work without `--jinja`?

---

👤 **genericgod** commented on **2025-08-15** at **17:08:30**

Yes it works without `--jinja`, but jinja is necessary for open-webui to correctly format the thinking block.

---

👤 **espen96** commented on **2025-08-18** at **08:16:54**

Right, all of this gets into the not so fun world of the Harmony response format.

Proper handling of it is also required for any kind of tool use. It's a can of worms

---

👤 **ultoris** commented on **2025-08-18** at **21:40:47**

The same happens with the ggml-org GGUFs for the 120b (I've also tried the F16 quant from Unsloth and produces a segfault with or without --jinja).

Aside from the tool calling, the thinking block problem is not limited to open-webui, aider also fails recognize it without --jinja.

---

👤 **gopinath87607** commented on **2025-08-20** at **05:10:26**

i am getting good speed around 18t/sec but i cant set the resaoning choice like hard and medium and low. in llama.cpp i am getting 12t/sec i am using ggml org quant.

---

👤 **sousekd** commented on **2025-08-22** at **08:00:52**

I’m also having issues with **gpt-oss**. It either babbles nonsense or outputs tokens that tools like **ik\_llama’s** embedded WebUI can’t handle - depending on whether the `--jinja` flag is used.

I’ve been off for a while. Is anyone running this successfully (ideally with tool calling) who can share their settings? The model is heavily censored, but the speed is great - I’m seeing \~15,000 t/s (pp) and \~250 t/s (tg).

---

👤 **ddxfish** commented on **2025-08-24** at **16:32:59**

I am having the same tool calling issue with `--jinja`. I found different chat templates embedded in the different quants for gpt oss 120b. --jijna seems to load the model's embedded template. With `llama-server --jinja` loops in thoughts and never returns cURL requests. Without --jinja it's 8 seconds for a hello. These are lmstudio then the Unsloth quant.
`strings gpt-oss-120b-MXFP4-00001-of-00002.gguf | grep -A 20 -B 5 "chat_template"`
`strings gpt-oss-120b-Q4_K_M-00001-of-00002.gguf | grep -A 20 -B 5 "chat_template"`

I tried to override with `--chat-template chatml` and a few other templates but none matched Harmony chat template format.
I tried Unsloth and LM Studio quants, not others.

```
./ik_llama.cpp/build/bin/llama-server \
    -m "unsloth/gpt-oss-120b-GGUF/gpt-oss-120b-Q4_K_M-00001-of-00002.gguf" \
    -c 16768 \
    -ngl 99 \
    -t 28 \
    -cnv \
    -fa -fmoe \
    --cont-batching \
    -ot "blk.(1[6-9]|[2-9][0-9]).ffn.*=CPU" \
    -p "$SYSTEM_PROMPT" \
    --jinja
```

```
ERR [validate_model_chat_template] failed to apply template: %s
 | ="Object is not callable: null at row 203, column 40:\n    {{- \"Knowledge cutoff: 2024-06\\n\" }}\n    {{- \"Current date: \" + strftime_now(\"%Y-%m-%d\") + \"\\n\\n\" }}\n                                       ^\n    {%- if reasoning_effort is not defined %}\n at row 203, column 52:\n    {{- \"Knowledge cutoff: 2024-06\\n\" }}\n    {{- \"Current date: \" + strftime_now(\"%Y-%m-%d\") + \"\\n\\n\" }}\n                                                   ^\n    {%- if reasoning_effort is not defined %}\n at row 203, column 62:\n    {{- \"Knowledge cutoff: 2024-06\\n\" }}\n    {{- \"Current date: \" + strftime_now(\"%Y-%m-%d\") + \"\\n\\n\" }}\n                                                             ^\n    {%- if reasoning_effort is not defined %}\n at row 203, column 5:\n    {{- \"Knowledge cutoff: 2024-06\\n\" }}\n    {{- \"Current date: \" + strftime_now(\"%Y-%m-%d\") + \"\\n\\n\" }}\n    ^\n    {%- if reasoning_effort is not defined %}\n at row 197, column 37:\n{#- System Message Construction ============================================ #}\n{%- macro build_system_message() -%}\n                                    ^\n    {%- if model_identity is not defined %}\n at row 231, column 25:\n{{- \"<|start|>system<|message|>\" }}\n{{- build_system_message() }}\n                        ^\n{{- \"<|end|>\" }}\n at row 231, column 1:\n{{- \"<|start|>system<|message|>\" }}\n{{- build_system_message() }}\n^\n{{- \"<|end|>\" }}\n at row 1, column 37:\n{# Chat template fixes by Unsloth #}\n                                    ^\n{#-\n"
WARN [              load_model] %s: The chat template that comes with this model is not yet supported, falling back to chatml. This may cause the model to output suboptimal responses
```

---

👤 **ikawrakow** commented on **2025-08-24** at **16:47:12**

Well, yes, GPT-OSS models need more work. Inference works, but all the stuff with chat templates, tool calling, etc., is not there yet. At some level I'm hoping that someone else will do this part.

---

👤 **firecoperana** commented on **2025-08-26** at **02:48:44**

jinja is working for gpt-oss in https://github.com/ikawrakow/ik_llama.cpp/pull/723, but tool calling does not work for me.

---

👤 **gopinath87607** commented on **2025-08-26** at **04:09:35**

> jinja is working for gpt-oss in [[#723](https://github.com/ikawrakow/ik_llama.cpp/issues/723)](https://github.com/ikawrakow/ik_llama.cpp/pull/723), but tool calling does not work for me.

i will check it.

---

👤 **gopinath87607** commented on **2025-08-26** at **05:10:23**

> jinja is working for gpt-oss in [[#723](https://github.com/ikawrakow/ik_llama.cpp/issues/723)](https://github.com/ikawrakow/ik_llama.cpp/pull/723), but tool calling does not work for me.

this is really great getting 19t/sec

CUDA_VISIBLE_DEVICES="0" ./bin/llama-server \
  --model "/home/gopi/gpt-oss-120b-mxfp4-00001-of-00003.gguf" \
  --ctx-size 101072 \
  -ngl 99 \
  --override-tensor ".*_exps\..*=CPU" \
  --override-tensor ".*\.ffn_.*_exps.*=CPU" \
  --threads 28 \
  --threads-batch 28 \
  --host 127.0.0.1 \
  -fa \
  -ser 30,1 \
  -fmoe \
  --jinja \
  --temp 0.6 \
  --top-p 1.0 \
  --chat-template-kwargs '{"reasoning_effort": "high"}' \
  --port 8080

in the above code i just played with ser for more accurate answer. getting good accuracy when i increase above 20.

---

👤 **firecoperana** commented on **2025-08-30** at **14:14:23**

Tool call also works now with https://github.com/ikawrakow/ik_llama.cpp/pull/723

---

👤 **genericgod** commented on **2025-09-01** at **10:04:47**

Looks like with d10d90ae it works now.
Closing, since the initial issue has been resolved.

Thank you.