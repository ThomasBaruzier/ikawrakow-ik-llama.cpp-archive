## 📌 [Issue #804](https://github.com/ikawrakow/ik_llama.cpp/issues/804) - Bug: Got exception while trying to continue the conversation in Deepseek V3.1 Terminus

| **Author** | `magikRUKKOLA` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-09-27 |
| **Updated** | 2025-09-28 |

---

## 📄 Description

### What happened?

Using:
```
mods -m deepseekv31t "lololo"
mods -C "lalala"
```
to continue the conversation, but there seems to be an error in the chat template when it tries to pick up the part of the llm response while its not having the closing tags?

```
ulimit -n 9999
ulimit -l unlimited

export CUDA_VISIBLE_DEVICES="0,1,2"
#export GGML_CUDA_ENABLE_UNIFIED_MEMORY=1

# --verbose-prompt

/opt/ik_llama.cpp/ik_llama.cpp/build/bin/llama-server \
    --model /opt/ubergarm/DeepSeek-V3.1-Terminus/IQ5_K/DeepSeek-V3.1-Terminus-IQ5_K-00001-of-00011.gguf \
    --alias ubergarm/DeepSeek-V3.1-Terminus_IQ5_K \
    --ctx-size $((140 * 1024)) \
    -b $((16 * 512)) -ub $((8 * 512)) \
    --mlock \
    --temp 0.5 --top-k 0 --top-p 1.0 --min-p 0.1 --repeat-penalty 1.0 \
    -ctk q8_0 \
    -mla 3 -fa \
    -fmoe \
    -amb 512 \
    --split-mode layer \
    --tensor-split 10,21,20 \
    --main-gpu 1 \
    --override-tensor exps=CPU \
    --n-gpu-layers 99 \
    --threads $(grep ^cpu\\scores /proc/cpuinfo | uniq | awk '{print $4}' | xargs -I{} echo "{}-0" | bc) \
    --host 0.0.0.0 \
    --port 8080 \
    --log-enable \
    --logdir /var/log/ \
    --jinja \
    --special \
    --verbosity 1 \
    --verbose-prompt \
    --reasoning-format none \
    --prompt-cache "$HOME/.cache/ik_llama.cpp/prompt-cache.bin" --prompt-cache-all \
    --slot-save-path "$HOME/.cache/ik_llama.cpp/slot.bin" \
    --lookup-cache-dynamic "$HOME/.cache/ik_llama.cpp/slot.bin" \
    --keep -1 \
    --slot-prompt-similarity 0.35 \
    --metrics
```

### Name and Version

```
/opt/ik_llama.cpp/ik_llama.cpp/build/bin/llama-cli --version
version: 3895 (367654f9)
built with cc (Debian 15.2.0-4) 15.2.0 for x86_64-linux-gnu
```

### What operating system are you seeing the problem on?

Linux

### Relevant log output

```shell
Sep 27 10:56:10 xxx run-ik_llama.cpp.sh[716448]: VERB [              operator()] Got exception | tid="140045404782592" timestamp=1758970570 code=500 message="split method must have between 1 and 1 positional arguments and between 0 and 0 keyword arguments at row 3, column 1908:\n\n' + message['content'] %}{%- endif %}{%- endif %}{%- endfor %}{{ bos_token }}{{ ns.system_prompt }}{%- for message in messages %}{%- if message['role'] == 'user' %}{%- set ns.is_tool = false -%}{%- set ns.is_first = false -%}{%- set ns.is_last_user = true -%}{{'<｜User｜>' + message['content']}}{%- endif %}{%- if message['role'] == 'assistant' and message['tool_calls'] is defined and message['tool_calls'] is not none %}{%- if ns.is_last_user %}{{'<｜Assistant｜></think>'}}{%- endif %}{%- set ns.is_last_user = false -%}{%- set ns.is_first = false %}{%- set ns.is_tool = false -%}{%- for tool in message['tool_calls'] %}{%- if not ns.is_first %}{%- if message['content'] is none %}{{'<｜tool▁calls▁begin｜><｜tool▁call▁begin｜>'+ tool['function']['name'] + '<｜tool▁sep｜>' + tool['function']['arguments'] + '<｜tool▁call▁end｜>'}}{%- else %}{{message['content'] + '<｜tool▁calls▁begin｜><｜tool▁call▁begin｜>' + tool['function']['name'] + '<｜tool▁sep｜>' + tool['function']['arguments'] + '<｜tool▁call▁end｜>'}}{%- endif %}{%- set ns.is_first = true -%}{%- else %}{{'<｜tool▁call▁begin｜>'+ tool['function']['name'] + '<｜tool▁sep｜>' + tool['function']['arguments'] + '<｜tool▁call▁end｜>'}}{%- endif %}{%- endfor %}{{'<｜tool▁calls▁end｜><｜end▁of▁sentence｜>'}}{%- endif %}{%- if message['role'] == 'assistant' and (message['tool_calls'] is not defined or message['tool_calls'] is none) %}{%- if ns.is_last_user %}{{'<｜Assistant｜>'}}{%- if message['prefix'] is defined and message['prefix'] and thinking %}{{'<think>'}}  {%- else %}{{'</think>'}}{%- endif %}{%- endif %}{%- set ns.is_last_user = false -%}{%- if ns.is_tool %}{{message['content'] + '<｜end▁of▁sentence｜>'}}{%- set ns.is_tool = false -%}{%- else %}{%- set content = message['content'] -%}{%- if '</think>' in content %}{%- set content = content.split('</think>', 1)[1] -%}{%- endif %}{{content + '<｜end▁of▁sentence｜>'}}{%- endif %}{%- endif %}{%- if message['role'] == 'tool' %}{%- set ns.is_last_user = false -%}{%- set ns.is_tool = true -%}{{'<｜tool▁output▁begin｜>' + message['content'] + '<｜tool▁output▁end｜>'}}{%- endif %}{%- endfor -%}{%- if add_generation_prompt and ns.is_last_user and not ns.is_tool %}{{'<｜Assistant｜>'}}{%- if not thinking %}{{'</think>'}}{%- else %}{{'<think>'}}{%- endif %}{% endif %}\n                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   ^\n at row 3, column 1908:\n\n' + message['content'] %}{%- endif %}{%- endif %}{%- endfor %}{{ bos_token }}{{ ns.system_prompt }}{%- for message in messages %}{%- if message['role'] == 'user' %}{%- set ns.is_tool = false -%}{%- set ns.is_first = false -%}{%- set ns.is_last_user = true -%}{{'<｜User｜>' + message['content']}}{%- endif %}{%- if message['role'] == 'assistant' and message['tool_calls'] is defined and message['tool_calls'] is not none %}{%- if ns.is_last_user %}{{'<｜Assistant｜></think>'}}{%- endif %}{%- set ns.is_last_user = false -%}{%- set ns.is_first = false %}{%- set ns.is_tool = false -%}{%- for tool in message['tool_calls'] %}{%- if not ns.is_first %}{%- if message['content'] is none %}{{'<｜tool▁calls▁begin｜><｜tool▁call▁begin｜>'+ tool['function']['name'] + '<｜tool▁sep｜>' + tool['function']['arguments'] + '<｜tool▁call▁end｜>'}}{%- else %}{{message['content'] + '<｜tool▁calls▁begin｜><｜tool▁call▁begin｜>' + tool['function']['name'] + '<｜tool▁sep｜>' + tool['function']['arguments'] + '<｜tool▁call▁end｜>'}}{%- endif %}{%- set ns.is_first = true -%}{%- else %}{{'<｜tool▁call▁begin｜>'+ tool['function']['name'] + '<｜tool▁sep｜>' + tool['function']['arguments'] + '<｜tool▁call▁end｜>'}}{%- endif %}{%- endfor %}{{'<｜tool▁calls▁end｜><｜end▁of▁sentence｜>'}}{%- endif %}{%- if message['role'] == 'assistant' and (message['tool_calls'] is not defined or message['tool_calls'] is none) %}{%- if ns.is_last_user %}{{'<｜Assistant｜>'}}{%- if message['prefix'] is defined and message['prefix'] and thinking %}{{'<think>'}}  {%- else %}{{'</think>'}}{%- endif %}{%- endif %}{%- set ns.is_last_user = false -%}{%- if ns.is_tool %}{{message['content'] + '<｜end▁of▁sentence｜>'}}{%- set ns.is_tool = false -%}{%- else %}{%- set content = message['content'] -%}{%- if '</think>' in content %}{%- set content = content.split('</think>', 1)[1] -%}{%- endif %}{{content + '<｜end▁of▁sentence｜>'}}{%- endif %}{%- endif %}{%- if message['role'] == 'tool' %}{%- set ns.is_last_user = false -%}{%- set ns.is_tool = true -%}{{'<｜tool▁output▁begin｜>' + message['content'] + '<｜tool▁output▁end｜>'}}{%- endif %}{%- endfor -%}{%- if add_generation_prompt and ns.is_last_user and not ns.is_tool %}{{'<｜Assistant｜>'}}{%- if not thinking %}{{'</think>'}}{%- else %}{{'<think>'}}{%- endif %}{% endif %}\n                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   ^\n at row 3, column 1882:\n\n' + message['content'] %}{%- endif %}{%- endif %}{%- endfor %}{{ bos_token }}{{ ns.system_prompt }}{%- for message in messages %}{%- if message['role'] == 'user' %}{%- set ns.is_tool = false -%}{%- set ns.is_first = false -%}{%- set ns.is_last_user = true -%}{{'<｜User｜>' + message['content']}}{%- endif %}{%- if message['role'] == 'assistant' and message['tool_calls'] is defined and message['tool_calls'] is not none %}{%- if ns.is_last_user %}{{'<｜Assistant｜></think>'}}{%- endif %}{%- set ns.is_last_user = false -%}{%- set ns.is_first = false %}{%- set ns.is_tool = false -%}{%- for tool in message['tool_calls'] %}{%- if not ns.is_first %}{%- if message['content'] is none %}{{'<｜tool▁calls▁begin｜><｜tool▁call▁begin｜>'+ tool['function']['name'] + '<｜tool▁sep｜>' + tool['function']['arguments'] + '<｜tool▁call▁end｜>'}}{%- else %}{{message['content'] + '<｜tool▁calls▁begin｜><｜tool▁call▁begin｜>' + tool['function']['name'] + '<｜tool▁sep｜>' + tool['function']['arguments'] + '<｜tool▁call▁end｜>'}}{%- endif %}{%- set ns.is_first = true -%}{%- else %}{{'<｜tool▁call▁begin｜>'+ tool['function']['name'] + '<｜tool▁sep｜>' + tool['function']['arguments'] + '<｜tool▁call▁end｜>'}}{%- endif %}{%- endfor %}{{'<｜tool▁calls▁end｜><｜end▁of▁sentence｜>'}}{%- endif %}{%- if message['role'] == 'assistant' and (message['tool_calls'] is not defined or message['tool_calls'] is none) %}{%- if ns.is_last_user %}{{'<｜Assistant｜>'}}{%- if message['prefix'] is defined and message['prefix'] and thinking %}{{'<think>'}}  {%- else %}{{'</think>'}}{%- endif %}{%- endif %}{%- set ns.is_last_user = false -%}{%- if ns.is_tool %}{{message['content'] + '<｜end▁of▁sentence｜>'}}{%- set ns.is_tool = false -%}{%- else %}{%- set content = message['content'] -%}{%- if '</think>' in content %}{%- set content = content.split('</think>', 1)[1] -%}{%- endif %}{{content + '<｜end▁of▁sentence｜>'}}{%- endif %}{%- endif %}{%- if message['role'] == 'tool' %}{%- set ns.is_last_user = false -%}{%- set ns.is_tool = true -%}{{'<｜tool▁output▁begin｜>' + message['content'] + '<｜tool▁output▁end｜>'}}{%- endif %}{%- endfor -%}{%- if add_generation_prompt and ns.is_last_user and not ns.is_tool %}{{'<｜Assistant｜>'}}{%- if not thinking %}{{'</think>'}}{%- else %}{{'<think>'}}{%- endif %}{% endif %} ...
```

---

## 💬 Conversation

👤 **magikRUKKOLA** commented on **2025-09-28** at **02:31:50**

Uh oh!  I compared the template embedded into the model and the one at /opt/ik_llama.cpp/ik_llama.cpp/models/templates/deepseek-ai-DeepSeek-V3.1.jinja and it turned out that the only difference is that in the embedded template the newlines are escaped.  The --jinja flag in my command line triggered the server to pickup the template with not escaped newlines.  So I removed the --jinja flag and that solved the issue.

---

👤 **magikRUKKOLA** commented on **2025-09-28** at **03:11:12**

Hm.  Wait a second.  In the unsloth blog they are saying:

https://unsloth.ai/blog/deepseek-v3.1
```
2. llama.cpp's jinja renderer via minja does not allow the use of extra arguments in the .split() command, so using .split(text, 1) works in Python, but not in minja. We had to change this to make llama.cpp function correctly without erroring out. We fixed it in all our quants and you will get the following error when using other quants:
terminate called after throwing an instance of 'std::runtime_error' what(): split method must have between 1 and 1 positional arguments and between 0 and 0 keyword arguments at row 3, column 1908
```

Ha!

```
{#- Unsloth template fixes #}{% if not add_generation_prompt is defined %}{% set add_generation_prompt = false %}{% endif %}{% if enable_thinking is defined and enable_thinking is false %}{% set thinking = false %}{% elif enable_thinking is defined and enable_thinking is true %}{% set thinking = true %}{% elif not thinking is defined %}{% set thinking = false %}{% endif %}{% set ns = namespace(is_first=false, is_tool=false, system_prompt='', is_first_sp=true, is_last_user=false) %}{%- for message in messages %}{%- if message['role'] == 'system' %}{%- if ns.is_first_sp %}{% set ns.system_prompt = ns.system_prompt + message['content'] %}{% set ns.is_first_sp = false %}{%- else %}{% set ns.system_prompt = ns.system_prompt + ' ' + message['content'] %}{%- endif %}{%- endif %}{%- endfor %}{{ bos_token }}{{ ns.system_prompt }}{%- for message in messages %}{%- if message['role'] == 'user' %}{%- set ns.is_tool = false -%}{%- set ns.is_first = false -%}{%- set ns.is_last_user = true -%}{{'<｜User｜>' + message['content']}}{%- endif %}{%- if message['role'] == 'assistant' and message['tool_calls'] is defined and message['tool_calls'] is not none %}{%- if ns.is_last_user %}{{'<｜Assistant｜></think>'}}{%- endif %}{%- set ns.is_last_user = false -%}{%- set ns.is_first = false %}{%- set ns.is_tool = false -%}{%- for tool in message['tool_calls'] %}{%- if not ns.is_first %}{%- if message['content'] is none %}{{'<｜tool▁calls▁begin｜><｜tool▁call▁begin｜>'+ tool['function']['name'] + '<｜tool▁sep｜>' + tool['function']['arguments'] + '<｜tool▁call▁end｜>'}}{%- else %}{{message['content'] + '<｜tool▁calls▁begin｜><｜tool▁call▁begin｜>' + tool['function']['name'] + '<｜tool▁sep｜>' + tool['function']['arguments'] + '<｜tool▁call▁end｜>'}}{%- endif %}{%- set ns.is_first = true -%}{%- else %}{{'<｜tool▁call▁begin｜>'+ tool['function']['name'] + '<｜tool▁sep｜>' + tool['function']['arguments'] + '<｜tool▁call▁end｜>'}}{%- endif %}{%- endfor %}{{'<｜tool▁calls▁end｜><｜end▁of▁sentence｜>'}}{%- endif %}{%- if message['role'] == 'assistant' and (message['tool_calls'] is not defined or message['tool_calls'] is none) %}{%- if ns.is_last_user %}{{'<｜Assistant｜>'}}{%- if message['prefix'] is defined and message['prefix'] and thinking %}{{'<think>'}} {%- else %}{{'</think>'}}{%- endif %}{%- endif %}{%- set ns.is_last_user = false -%}{%- if ns.is_tool %}{{message['content'] + '<｜end▁of▁sentence｜>'}}{%- set ns.is_tool = false -%}{%- else %}{%- set content = message['content'] -%}{%- if '</think>' in content %}{%- set splitted = content.split('</think>') -%}{%- set content = splitted[1:] | join('</think>') -%}{%- endif %}{{content + '<｜end▁of▁sentence｜>'}}{%- endif %}{%- endif %}{%- if message['role'] == 'tool' %}{%- set ns.is_last_user = false -%}{%- set ns.is_tool = true -%}{{'<｜tool▁output▁begin｜>' + message['content'] + '<｜tool▁output▁end｜>'}}{%- endif %}{%- endfor -%}{%- if add_generation_prompt and ns.is_last_user and not ns.is_tool %}{{'<｜Assistant｜>'}}{%- if not thinking %}{{'</think>'}}{%- else %}{{'<think>'}}{%- endif %}{% endif %}{#- Copyright 2025-present Unsloth. Apache 2.0 License. #}
```

---

👤 **magikRUKKOLA** commented on **2025-09-28** at **03:42:24**

Hmm...

```diff
--- chat_template.jinja.original.formatted      2025-09-28 03:37:10.686917528 +0000
+++ chat_template.jinja.formatted       2025-09-28 03:37:07.546761731 +0000
@@ -1,7 +1,11 @@
 {% if not add_generation_prompt is defined %}
 {% set add_generation_prompt = false %}
 {% endif %}
-{% if not thinking is defined %}
+{% if enable_thinking is defined and enable_thinking is false %}
+{% set thinking = false %}
+{% elif enable_thinking is defined and enable_thinking is true %}
+{% set thinking = true %}
+{% elif not thinking is defined %}
 {% set thinking = false %}
 {% endif %}
 {% set ns = namespace(is_first=false, is_tool=false, system_prompt='', is_first_sp=true, is_last_user=false) %}
@@ -11,7 +15,7 @@
 {% set ns.system_prompt = ns.system_prompt + message['content'] %}
 {% set ns.is_first_sp = false %}
 {%- else %}
-{% set ns.system_prompt = ns.system_prompt + '\n\n' + message['content'] %}
+{% set ns.system_prompt = ns.system_prompt + ' ' + message['content'] %}
 {%- endif %}
 {%- endif %}
 {%- endfor %}
@@ -25,7 +29,7 @@
 {%- endif %}
 {%- if message['role'] == 'assistant' and message['tool_calls'] is defined and message['tool_calls'] is not none %}
 {%- if ns.is_last_user %}
-{{'### Assistant: <response>'}}
+{{'### Assistant: '}}
 {%- endif %}
 {%- set ns.is_last_user = false -%}
 {%- set ns.is_first = false %}
@@ -48,7 +52,7 @@
 {%- if ns.is_last_user %}
 {{'### Assistant: '}}
 {%- if message['prefix'] is defined and message['prefix'] and thinking %}
-{{'<thinking>  '}}
+{{'<thinking> '}}
 {%- else %}
 {{'<response>'}}
 {%- endif %}
@@ -60,7 +64,8 @@
 {%- else %}
 {%- set content = message['content'] -%}
 {%- if '<response>' in content %}
-{%- set content = content.split('<response>', 1)[1] -%}
+{%- set splitted = content.split('<response>') -%}
+{%- set content = splitted[1:] | join('<response>') -%}
 {%- endif %}
 {{content + ''}}
 {%- endif %}
```

so ... the chat template is not compatible with minja and its required to chat it to use split + join?

---

👤 **magikRUKKOLA** commented on **2025-09-28** at **04:07:27**

The tool calling doesn't work for Deepseek-V3.1-Terminus for sure.

Related: https://github.com/ikawrakow/ik_llama.cpp/pull/771

With the chat_template from unsloth I was able to make the Deepseek to generate the tool calls but its outputting the:

```bash
echo 'Hello, World!'
```

instead of solving any problem. lol--

[EDIT]:  what a f**king day today.  I had used --chat-template and set it to a path to the file instead of using --chat-template-file. :D

Now we are back to the different issue https://github.com/ikawrakow/ik_llama.cpp/issues/750:

```
Sep 28 04:22:48 xxx run-ik_llama.cpp.sh[735431]: [1759033368] Partial parse: (?:<｜tool▁call▁begin｜>)?([^\n<]+)(?:<｜tool>
Sep 28 04:22:48 xxx run-ik_llama.cpp.sh[735431]: terminate called after throwing an instance of 'std::runtime_error'
Sep 28 04:22:48 xxx run-ik_llama.cpp.sh[735431]:   what():  Invalid diff:
```