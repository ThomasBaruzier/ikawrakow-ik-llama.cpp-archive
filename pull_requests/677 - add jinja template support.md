## 🔀 [Pull Request #677](https://github.com/ikawrakow/ik_llama.cpp/pull/677) - add jinja template support

| **Author** | `firecoperana` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `fcp/jinja` |
| **Target Branch** | `main` |
| **Created** | 2025-08-08 |
| **Updated** | 2025-10-26 |
| **Merged** | 2025-08-09 |
| **Assignees** | `firecoperana` |

---

## 📄 Description

Test with the example from https://github.com/ggml-org/llama.cpp/pull/11016
It looks ok. Not sure whether it will have conflict with existing function call features. 

- [x] I have read the [contributing guidelines](https://github.com/ggerganov/llama.cpp/blob/master/CONTRIBUTING.md)
- Self-reported review complexity:
  - [ ] Low
  - [x] Medium
  - [ ] High

---

## 💬 Conversation

👤 **ikawrakow** commented on **2025-08-08** at **11:36:49**

Can someone please check if this works?

Especially people using function/tool calling.

Thanks!

---

👤 **espen96** commented on **2025-08-08** at **17:12:53**

I have a version of Gemma3 27B here which can see tools in mainline. Could be related to how mainline does tools, enabling `--jinja` here with this pr did not let it see the tools. Again, that could be something else.

using a custom template through `--chat-template-file` , ripped from Qwen3, it did see and list the tools correctly. the rest of the output was as expected, broken. I did after all feed it the wrong template.

GLM-air does not appear to be able to see tools with `--chat-template chatglm4` 
using jinja I get 


`500: Unknown argument ensure_ascii for function tojson at row 11, column 37:
{% for tool in tools %}
{{ tool | tojson(ensure_ascii=False) }}
^
{% endfor %}
at row 11, column 1:
{% for tool in tools %}
{{ tool | tojson(ensure_ascii=False) }}
^
{% endfor %}
at row 10, column 24:
<tools>
{% for tool in tools %}
^
{{ tool | tojson(ensure_ascii=False) }}
at row 10, column 1:
<tools>
{% for tool in tools %}
^
{{ tool | tojson(ensure_ascii=False) }}
at row 2, column 17:
[gMASK]<sop>
{%- if tools -%}
^
<|system|>
at row 2, column 1:
[gMASK]<sop>
{%- if tools -%}
^
<|system|>
at row 1, column 1:
[gMASK]<sop>
^
{%- if tools -%}`


and in the cli the message `tools param is not fully supported yet` not sure if that was missed or expected. 
Looking at the original commit from mainline, it has the same message.

besides that the model appears to be working, not having any jinja flag appears to trigger the error 
`500: tools param requires --jinja flag`  which is required in mainline.


Qwen3-30B-A3B-Instruct-2507

it does see the tools without  `--jinja` 
Using the tool without jinja appears to work just fine
using tools with it, appears to work fine

---

👤 **firecoperana** commented on **2025-08-08** at **18:01:56**

"tools param is not fully supported yet" is expected because this pr only adds support for jinja template. Tool calls support from mainline has not been fully integrated here.

---

👤 **RodriMora** commented on **2025-08-08** at **18:13:07**

Seing similar behaviour, using this gist https://gist.github.com/RodriMora/099913a7cea971d1bd09c623fc12c7bf

I tested against this PR and the result with `--jinja` is:

`
openai.InternalServerError: Error code: 500 - {'error': {'code': 500, 'message': 'Unknown argument ensure_ascii for function tojson at row 11, column 37:\n{% for tool in tools %}\n{{ tool | tojson(ensure_ascii=False) }}\n                                    ^\n{% endfor %}\n at row 11, column 1:\n{% for tool in tools %}\n{{ tool | tojson(ensure_ascii=False) }}\n^\n{% endfor %}\n at row 10, column 24:\n<tools>\n{% for tool in tools %}\n                       ^\n{{ tool | tojson(ensure_ascii=False) }}\n at row 10, column 1:\n<tools>\n{% for tool in tools %}\n^\n{{ tool | tojson(ensure_ascii=False) }}\n at row 2, column 17:\n[gMASK]<sop>\n{%- if tools -%}\n                ^\n<|system|>\n at row 2, column 1:\n[gMASK]<sop>\n{%- if tools -%}\n^\n<|system|>\n at row 1, column 1:\n[gMASK]<sop>\n^\n{%- if tools -%}\n', 'type': 'server_error'}}
`


Without `--jinja`:

`openai.InternalServerError: Error code: 500 - {'error': {'code': 500, 'message': 'tools param requires --jinja flag', 'type': 'server_error'}}`

Mainline llama.cpp with `--jinja`:

`ChatCompletion(id='chatcmpl-3nMluDyMeeRz1mvYFM7piNgihpD9qXGg', choices=[Choice(finish_reason='tool_calls', index=0, logprobs=None, message=ChatCompletionMessage(content=None, refusal=None, role='assistant', annotations=None, audio=None, function_call=None, tool_calls=[ChatCompletionMessageToolCall(id='1xW6qLjhIgfCdkc1xfSE6NE0HvWZaLsW', function=Function(arguments='{"a":2,"b":3}', name='add'), type='function')], reasoning_content="Okay, let's see. The user wants to add 2 and 3. I need to check the available tools. There's a function called add that takes two numbers a and b. The required parameters are a and b, both numbers. The user provided 2 and 3, so I should call add with a=2 and b=3. I'll make sure to structure the tool call correctly in JSON inside the specified XML tags."))], created=1754676258, model='GLM-4.5', object='chat.completion', service_tier=None, system_fingerprint='b6119-cd6983d56', usage=CompletionUsage(completion_tokens=118, prompt_tokens=180, total_tokens=298, completion_tokens_details=None, prompt_tokens_details=None), timings={'prompt_n': 129, 'prompt_ms': 792.932, 'prompt_per_token_ms': 6.14675968992248, 'prompt_per_second': 162.68734267251165, 'predicted_n': 118, 'predicted_ms': 3029.918, 'predicted_per_token_ms': 25.677271186440677, 'predicted_per_second': 38.94494834513673})`

---

👤 **firecoperana** commented on **2025-08-08** at **19:28:34**

If you remove tool_choice, does it still report error?

---

👤 **RodriMora** commented on **2025-08-08** at **21:06:48**

> If you remove tool_choice, does it still report error?

I had to remove booth the call of tools=tools and tool_choice="auto". But this was the response:

`ChatCompletion(id='chatcmpl-9bcdLnnPQr5CaNKIQhfZ1nBFEWfj9agC', choices=[Choice(finish_reason='stop', index=0, logprobs=None, message=ChatCompletionMessage(content='\n<think>First, the user said "add 2 and 3". That seems straightforward. Adding 2 and 3 should give 5. But let me make sure there\'s no trick here. Is there any context I\'m missing? The message is very simple: "add 2 and 3". No other words or numbers.\n\nPerhaps this is a test of basic arithmetic. Or maybe it\'s part of a larger conversation, but since this is a new interaction, I should take it at face value.\n\nIn mathematics, addition is commutative, so 2 + 3 is the same as 3 + 2, both equal to 5.\n\nI could respond by saying "2 + 3 = 5" to be clear and educational.\n\nSince the user said "add", it might be that they want me to perform the operation and state the result.\n\nAlso, considering the platform, this could be a simple query from someone learning math or just a quick check.\n\nI should respond in a friendly and helpful manner. Something like: "Adding 2 and 3 gives 5."\n\nTo make it engaging, I could add a bit more, like "The sum of 2 and 3 is 5."\n\nBut I shouldn\'t overcomplicate it. Keep it simple.\n\nFinally, ensure the response is clear and correct.</think>Adding 2 and 3 is simple arithmetic:  \n**2 + 3 = 5**  \n\nSo, the result is **5**! 😊', refusal=None, role='assistant', annotations=None, audio=None, function_call=None, tool_calls=None))], created=1754686493, model='GLM-4.5', object='chat.completion', service_tier=None, system_fingerprint=None, usage=CompletionUsage(completion_tokens=307, prompt_tokens=11, total_tokens=318, completion_tokens_details=None, prompt_tokens_details=None))`

So `finish_reason='stop'` instead of` tool_calls`
No `tool_calls `or `function_call` field

Using it with Opencode, it fails the load the tools:
<img width="756" height="232" alt="image" src="https://github.com/user-attachments/assets/52019b01-5700-470b-9815-aa9baa7aa110" />

---

👤 **Hisma** commented on **2025-08-09** at **02:44:53**

hmmm... I can't even get it to load my model with the `--jinja` flag.
```
(base) root@hisma-super-server:/home/hisma/ik_llama.cpp/build# llama-server \
-m /home/hisma/llama.cpp/models/zai-org_GLM-4.5-Air-IQ5_K/IQ5_K/GLM-4.5-Air-IQ5_K-00001-of-00002.gguf   \
--jinja \
-ub 4096 \
-b 2048 \
-c 131072 \
-ctk q8_0 \
-ctv q8_0 \
-t 1 \
-ngl 99 \
-fa \
-fmoe \
--temp 0.6 \
--top-p 1.0 \
--host 0.0.0.0 \
--port 8888
INFO [                    main] build info | tid="134565362118656" timestamp=1754706799 build=3831 commit="093f8796"
INFO [                    main] system info | tid="134565362118656" timestamp=1754706799 n_threads=1 n_threads_batch=-1 total_threads=128 system_info="AVX = 1 | AVX_VNNI = 0 | AVX2 = 1 | AVX512 = 0 | AVX512_VBMI = 0 | AVX512_VNNI = 0 | AVX512_BF16 = 0 | FMA = 1 | NEON = 0 | SVE = 0 | ARM_FMA = 0 | F16C = 1 | FP16_VA = 0 | WASM_SIMD = 0 | BLAS = 1 | SSE3 = 1 | SSSE3 = 1 | VSX = 0 | MATMUL_INT8 = 0 | LLAMAFILE = 1 | "
llama_model_loader: additional 1 GGUFs metadata loaded.
llama_model_loader: loaded meta data with 48 key-value pairs and 803 tensors from /home/hisma/llama.cpp/models/zai-org_GLM-4.5-Air-IQ5_K/IQ5_K/GLM-4.5-Air-IQ5_K-00001-of-00002.gguf (version GGUF V3 (latest))
llama_model_loader: Dumping metadata keys/values. Note: KV overrides do not apply in this output.
llama_model_loader: - kv   0:                       general.architecture str              = glm4moe
llama_model_loader: - kv   1:                               general.type str              = model
llama_model_loader: - kv   2:                               general.name str              = GLM 4.5 Air
llama_model_loader: - kv   3:                         general.size_label str              = 128x9.4B
llama_model_loader: - kv   4:                            general.license str              = mit
llama_model_loader: - kv   5:                               general.tags arr[str,1]       = ["text-generation"]
llama_model_loader: - kv   6:                          general.languages arr[str,2]       = ["en", "zh"]
llama_model_loader: - kv   7:                        glm4moe.block_count u32              = 47
llama_model_loader: - kv   8:                     glm4moe.context_length u32              = 131072
llama_model_loader: - kv   9:                   glm4moe.embedding_length u32              = 4096
llama_model_loader: - kv  10:                glm4moe.feed_forward_length u32              = 10944
llama_model_loader: - kv  11:               glm4moe.attention.head_count u32              = 96
llama_model_loader: - kv  12:            glm4moe.attention.head_count_kv u32              = 8
llama_model_loader: - kv  13:                     glm4moe.rope.freq_base f32              = 1000000.000000
llama_model_loader: - kv  14:   glm4moe.attention.layer_norm_rms_epsilon f32              = 0.000010
llama_model_loader: - kv  15:                  glm4moe.expert_used_count u32              = 8
llama_model_loader: - kv  16:               glm4moe.attention.key_length u32              = 128
llama_model_loader: - kv  17:             glm4moe.attention.value_length u32              = 128
llama_model_loader: - kv  18:                          general.file_type u32              = 141
llama_model_loader: - kv  19:               glm4moe.rope.dimension_count u32              = 64
llama_model_loader: - kv  20:                       glm4moe.expert_count u32              = 128
llama_model_loader: - kv  21:         glm4moe.expert_feed_forward_length u32              = 1408
llama_model_loader: - kv  22:                glm4moe.expert_shared_count u32              = 1
llama_model_loader: - kv  23:          glm4moe.leading_dense_block_count u32              = 1
llama_model_loader: - kv  24:                 glm4moe.expert_gating_func u32              = 2
llama_model_loader: - kv  25:               glm4moe.expert_weights_scale f32              = 1.000000
llama_model_loader: - kv  26:                glm4moe.expert_weights_norm bool             = true
llama_model_loader: - kv  27:               glm4moe.nextn_predict_layers u32              = 1
llama_model_loader: - kv  28:                       tokenizer.ggml.model str              = gpt2
llama_model_loader: - kv  29:                         tokenizer.ggml.pre str              = glm4
llama_model_loader: - kv  30:                      tokenizer.ggml.tokens arr[str,151552]  = ["!", "\"", "#", "$", "%", "&", "'", ...
llama_model_loader: - kv  31:                  tokenizer.ggml.token_type arr[i32,151552]  = [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, ...
llama_model_loader: - kv  32:                      tokenizer.ggml.merges arr[str,318088]  = ["Ġ Ġ", "Ġ ĠĠĠ", "ĠĠ ĠĠ", "...
llama_model_loader: - kv  33:                tokenizer.ggml.eos_token_id u32              = 151329
llama_model_loader: - kv  34:            tokenizer.ggml.padding_token_id u32              = 151329
llama_model_loader: - kv  35:                tokenizer.ggml.bos_token_id u32              = 151331
llama_model_loader: - kv  36:                tokenizer.ggml.eot_token_id u32              = 151336
llama_model_loader: - kv  37:            tokenizer.ggml.unknown_token_id u32              = 151329
llama_model_loader: - kv  38:                tokenizer.ggml.eom_token_id u32              = 151338
llama_model_loader: - kv  39:                    tokenizer.chat_template str              = [gMASK]<sop>\n{%- if tools -%}\n<|syste...
llama_model_loader: - kv  40:               general.quantization_version u32              = 2
llama_model_loader: - kv  41:                      quantize.imatrix.file str              = /mnt/raid/models/ubergarm/GLM-4.5-Air...
llama_model_loader: - kv  42:                   quantize.imatrix.dataset str              = ubergarm-imatrix-calibration-corpus-v...
llama_model_loader: - kv  43:             quantize.imatrix.entries_count i32              = 503
llama_model_loader: - kv  44:              quantize.imatrix.chunks_count i32              = 814
llama_model_loader: - kv  45:                                   split.no u16              = 0
llama_model_loader: - kv  46:                                split.count u16              = 2
llama_model_loader: - kv  47:                        split.tensors.count i32              = 803
llama_model_loader: - type  f32:  331 tensors
llama_model_loader: - type q8_0:  333 tensors
llama_model_loader: - type q6_0:   45 tensors
llama_model_loader: - type iq5_k:   90 tensors
llama_model_loader: - type iq6_k:    2 tensors
llama_model_loader: - type iq5_ks:    2 tensors
llm_load_vocab: special tokens cache size = 36
llm_load_vocab: token to piece cache size = 0.9713 MB
llm_load_print_meta: format           = GGUF V3 (latest)
llm_load_print_meta: arch             = glm4moe
llm_load_print_meta: vocab type       = BPE
llm_load_print_meta: n_vocab          = 151552
llm_load_print_meta: n_merges         = 318088
llm_load_print_meta: vocab_only       = 0
llm_load_print_meta: n_ctx_train      = 131072
llm_load_print_meta: n_embd           = 4096
llm_load_print_meta: n_layer          = 47
llm_load_print_meta: n_head           = 96
llm_load_print_meta: n_head_kv        = 8
llm_load_print_meta: n_rot            = 64
llm_load_print_meta: n_swa            = 0
llm_load_print_meta: n_swa_pattern    = 1
llm_load_print_meta: n_embd_head_k    = 128
llm_load_print_meta: n_embd_head_v    = 128
llm_load_print_meta: n_gqa            = 12
llm_load_print_meta: n_embd_k_gqa     = 1024
llm_load_print_meta: n_embd_v_gqa     = 1024
llm_load_print_meta: f_norm_eps       = 0.0e+00
llm_load_print_meta: f_norm_rms_eps   = 1.0e-05
llm_load_print_meta: f_clamp_kqv      = 0.0e+00
llm_load_print_meta: f_max_alibi_bias = 0.0e+00
llm_load_print_meta: f_logit_scale    = 0.0e+00
llm_load_print_meta: n_ff             = 10944
llm_load_print_meta: n_expert         = 128
llm_load_print_meta: n_expert_used    = 8
llm_load_print_meta: causal attn      = 1
llm_load_print_meta: pooling type     = 0
llm_load_print_meta: rope type        = 2
llm_load_print_meta: rope scaling     = linear
llm_load_print_meta: freq_base_train  = 1000000.0
llm_load_print_meta: freq_scale_train = 1
llm_load_print_meta: n_ctx_orig_yarn  = 131072
llm_load_print_meta: rope_finetuned   = unknown
llm_load_print_meta: ssm_d_conv       = 0
llm_load_print_meta: ssm_d_inner      = 0
llm_load_print_meta: ssm_d_state      = 0
llm_load_print_meta: ssm_dt_rank      = 0
llm_load_print_meta: model type       = 106B.A12B
llm_load_print_meta: model ftype      = IQ5_K - 5.5 bpw
llm_load_print_meta: model params     = 110.469 B
llm_load_print_meta: model size       = 77.704 GiB (6.042 BPW) 
llm_load_print_meta: repeating layers = 76.747 GiB (6.036 BPW, 109.227 B parameters)
llm_load_print_meta: general.name     = GLM 4.5 Air
llm_load_print_meta: BOS token        = 151331 '[gMASK]'
llm_load_print_meta: EOS token        = 151329 '<|endoftext|>'
llm_load_print_meta: UNK token        = 151329 '<|endoftext|>'
llm_load_print_meta: PAD token        = 151329 '<|endoftext|>'
llm_load_print_meta: LF token         = 128 'Ä'
llm_load_print_meta: FIM PRE token    = 151347 '<|code_prefix|>'
llm_load_print_meta: FIM SUF token    = 151349 '<|code_suffix|>'
llm_load_print_meta: FIM MID token    = 151348 '<|code_middle|>'
llm_load_print_meta: EOT token        = 151336 '<|user|>'
llm_load_print_meta: max token length = 1024
ggml_cuda_init: GGML_CUDA_FORCE_MMQ:    no
ggml_cuda_init: GGML_CUDA_FORCE_CUBLAS: no
ggml_cuda_init: found 6 CUDA devices:
Illegal instruction (core dumped)
```

This is the quant here - 
https://huggingface.co/ubergarm/GLM-4.5-Air-GGUF
by @ubergarm 

I removed all flags, included `--jinja` to the bare minimum.  Still doesn't work.  
It's worth noting that to get that model to work, I have to use the branch from  https://github.com/ikawrakow/ik_llama.cpp/pull/668, which was just merged a couple days ago, so I'm not sure if this branch includes those changes.

---

👤 **Hisma** commented on **2025-08-09** at **03:14:06**

FYI model loads fine using the latest mainline branch that merged GLM-4.5 support.

No `--jinja` flag of course since it's not recognized, but wanted to isolate the issue of the GLM 4.5 model not loading to the changes specific to this PR.

---

👤 **firecoperana** commented on **2025-08-09** at **03:31:39**

After pull the latest commit with GLM4.5 support, GLM-4.5-Air-UD-Q2_K_XL.gguf works for me with --jinja flag.
Without --jinja, I got openai.InternalServerError: Error code: 500 - {'error': {'code': 500, 'message': 'tools param requires --jinja flag', 'type': 'server_error'}}

messages = [
    {"role":"user", "content": """Print a hello world message with python (using single quotes '"'"' for strings)."""
}
]

tools = [
      {
        "type": "function",
        "function": {
          "name": "ipython",
          "description": "Runs code in an ipython interpreter and returns the result of the execution after 60 seconds.",
          "parameters": {
            "type": "object",
            "properties": {
              "code": {
                "type": "string",
                "description": "The code to run in the ipython interpreter."
              }
            },
            "required": ["code"]
          }
        }
      }
    ],


result_raw = asyncio.run(aclient.chat.completions.create(
    model="gpt-3.5-turbo",
    tools=tools,
    messages=messages,
    **create_kwargs
))

Response:
ChatCompletion(id='chatcmpl-wlV6LrgPDHEAGH4sIiAsZyy4QQcxjJgv', choices=[Choice(finish_reason='stop', index=0, logprobs=None, message=ChatCompletionMessage(content='\n<think>The user wants me to print a hello world message using Python, and they specifically mention using single quotes \'"\'"\' for strings. This is a bit unusual because typically we use single quotes like \'hello world\' or double quotes like "hello world". \n\nThe user is asking for single quotes \'"\'"\' which looks like they want to use triple quotes or maybe they want to escape quotes in some way. Let me think about this:\n\n1. They say "single quotes \'"\'"\'" - this looks like they want me to use single quotes but they\'re showing me what appears to be escaped quotes\n2. In Python, we can use single quotes \'hello world\' or double quotes "hello world"\n3. We can also use triple quotes: \'\'\'hello world\'\'\' or """hello world"""\n\nI think the user might be asking for a string that uses single quotes, so I\'ll use \'hello world\'. The way they wrote \'"\'"\'" might be a typo or they might be showing me how to escape quotes.\n\nLet me run a simple Python print statement with single quotes.</think>\nI\'ll print a hello world message using Python with single quotes as requested.\n<tool_call>ipython\n<arg_key>code</arg_key>\n<arg_value>print(\'hello world\')</arg_value>\n</tool_call>', refusal=None, role='assistant', annotations=None, audio=None, function_call=None, tool_calls=None))], created=1754709734, model='gpt-3.5-turbo', object='chat.completion', service_tier=None, system_fingerprint=None, usage=CompletionUsage(completion_tokens=253, prompt_tokens=206, total_tokens=459, completion_tokens_details=None, prompt_tokens_details=None))

---

👤 **Hisma** commented on **2025-08-09** at **03:38:38**

I'm confused @firecoperana, how are you using the `--jinja` flag on the mainline branch?
Are you saying you merged your changes w/ this PR w/ the latest mainline branch and now it works?  
Can you please update your branch so I can test it myself?

---

👤 **Hisma** commented on **2025-08-09** at **04:07:34**

OK I rebased your branch locally and re-compiled.  I successfully was able to pass the --jinja command and load the model.  Tool calling works fine w/ the latest commits from main merged with this PR branch.

As stated, please update your remote branch so others can easily test.

---

👤 **ikawrakow** commented on **2025-08-09** at **05:46:47**

@firecoperana I took the liberty of rebasing on the latest main branch for easier testing (including GLM4-MoE).

---

👤 **Hisma** commented on **2025-08-09** at **06:13:20**

So as a sanity check I pulled the latest version of this branch after the forced rebase and re-compiled.  As expected, the `--jinja` flag now works.

Here's some successful tool calling in cline - 
<img width="767" height="1375" alt="image" src="https://github.com/user-attachments/assets/8ef05734-158b-41d7-9c99-f488e097174d" />

Others that were having problems earlier should try now.

---

👤 **ikawrakow** approved this pull request ✅ on **2025-08-09** at **06:34:10**

---

👤 **RodriMora** commented on **2025-08-09** at **07:12:52**

Tested some agents with tool calling:

-Cline **works** with tool calling and MCP's
-Roocode **works** with tool calling and MCP's
-Opencode **does not work**, error logs:
```
AI_RetryError: Failed after 4 attempts. Last error: Unknown argument ensure_ascii for function tojson at row 11, column 37:
{% for tool in tools %}
{{ tool | tojson(ensure_ascii=False) }}
                                    ^
{% endfor %}
 at row 11, column 1:
{% for tool in tools %}
{{ tool | tojson(ensure_ascii=False) }}
^
{% endfor %}
 at row 10, column 24:
<tools>
{% for tool in tools %}
                       ^
{{ tool | tojson(ensure_ascii=False) }}
 at row 10, column 1:
<tools>
{% for tool in tools %}
^
{{ tool | tojson(ensure_ascii=False) }}
 at row 2, column 17:
[gMASK]<sop>
{%- if tools -%}
                ^
<|system|>
 at row 2, column 1:
[gMASK]<sop>
{%- if tools -%}
^
<|system|>
 at row 1, column 1:
[gMASK]<sop>
^
{%- if tools -%}
```

---

👤 **Hisma** commented on **2025-08-09** at **12:52:26**

> Tested some agents with tool calling:
> 
> -Cline **works** with tool calling and MCP's -Roocode **works** with tool calling and MCP's -Opencode **does not work**, error logs:
> 
> ```
> AI_RetryError: Failed after 4 attempts. Last error: Unknown argument ensure_ascii for function tojson at row 11, column 37:
> {% for tool in tools %}
> {{ tool | tojson(ensure_ascii=False) }}
>                                     ^
> {% endfor %}
>  at row 11, column 1:
> {% for tool in tools %}
> {{ tool | tojson(ensure_ascii=False) }}
> ^
> {% endfor %}
>  at row 10, column 24:
> <tools>
> {% for tool in tools %}
>                        ^
> {{ tool | tojson(ensure_ascii=False) }}
>  at row 10, column 1:
> <tools>
> {% for tool in tools %}
> ^
> {{ tool | tojson(ensure_ascii=False) }}
>  at row 2, column 17:
> [gMASK]<sop>
> {%- if tools -%}
>                 ^
> <|system|>
>  at row 2, column 1:
> [gMASK]<sop>
> {%- if tools -%}
> ^
> <|system|>
>  at row 1, column 1:
> [gMASK]<sop>
> ^
> {%- if tools -%}
> ```

it looks like it's having problems doing tool calls in json format.  As far as I know the chat_template uses xml tags for tool calling. It's possible this is an opencode problem not liking tool calls using xml and expecting something like json natively.

```
# Tools\n\nYou may call one or more functions to assist with the user query.\n\nYou are provided with function signatures within <tools></tools> XML tags:\n<tools>\n{% for tool in tools %}\n{{ tool | tojson(ensure_ascii=False) }}\n{% endfor %}\n</tools>\n\nFor each function call, output the function name and arguments within the following XML format:\n<tool_call>{function-name}
```

I would look into opencode and see what tool calling format it expects and if it supports xml.  I have seen this problem pop up on other apps...

---

👤 **tomt610** commented on **2025-08-09** at **16:53:06**

Text completion looks broken to me.
On latest merged main branch I get 500 error using completions endpoint while using GLM4.5 or Deepseek V3 model.

Request URL
http://127.0.0.1:8000/api/backends/text-completions/generate
Request Method
POST
Status Code
500 Internal Server Error

[json.exception.out_of_range.403] key 'messages' not found

<img width="336" height="134" alt="image" src="https://github.com/user-attachments/assets/b6590e6f-237d-4f61-9285-b6b78ccd1ade" />

It works fine on Commit 7117c23 which was just before jinja merge

---

👤 **RodriMora** commented on **2025-08-09** at **16:53:31**

> it looks like it's having problems doing tool calls in json format. As far as I know the chat_template uses xml tags for tool calling. It's possible this is an opencode problem not liking tool calls using xml and expecting something like json natively.

I believe you are sending the` <tool_call>…</tool_call>` markup but not converting those tags into the structured `tool_calls` array and then flipping the `finish_reason` to "tool_calls".

Example of qwen3-coder (has good support for tools, seems like glm and gpt-oss don't atm) response with mainline llama.cpp:

```
{
  "id": "chatcmpl-74O0JXpAVzB2ERgklS7sYknRk9wSYF1i",
  "object": "chat.completion",
  "created": 1754757361,
  "model": "modeltest",
  "system_fingerprint": "b6122-34c9d765b",
  "choices": [
    {
      "index": 0,
      "finish_reason": "tool_calls",
      "message": {
        "role": "assistant",
        "content": null,
        "tool_calls": [
          {
            "id": "MHuWt9AGCrF2x2FSkfgRO5Q6lqzWGhN6",
            "type": "function",
            "function": {
              "name": "add",
              "arguments": "{\"a\":2,\"b\":3}"
            }
          }
        ]
      }
    }
  ],
  "usage": {
    "prompt_tokens": 316,
    "completion_tokens": 28,
    "total_tokens": 344
  },
  "timings": {
    "prompt_n": 316,
    "prompt_ms": 475.035,
    "prompt_per_token_ms": 1.5032753164556962,
    "prompt_per_second": 665.2141421158441,
    "predicted_n": 28,
    "predicted_ms": 408.269,
    "predicted_per_token_ms": 14.581035714285715,
    "predicted_per_second": 68.58223377234128
  }
}
```


Response with ik_llama.cpp (pulled last commit in main branch):

```
{
  "id": "chatcmpl-FH1F1kCiklkuyuTs4y7xeqevFt5oVbT1",
  "object": "chat.completion",
  "created": 1754757338,
  "model": "modeltest",
  "choices": [
    {
      "index": 0,
      "finish_reason": "stop",
      "message": {
        "role": "assistant",
        "content": "<tool_call>\n<function=add>\n<parameter=a>\n2\n</parameter>\n<parameter=b>\n3\n</parameter>\n</function>\n</tool_call>"
      }
    }
  ],
  "usage": {
    "prompt_tokens": 316,
    "completion_tokens": 29,
    "total_tokens": 345
  }
}

```


The openai compliant tool call response should be something like this?:

```
{
  "choices": [
    {
      "finish_reason": "tool_calls",
      "message": {
        "role": "assistant",
        "content": null,
        "tool_calls": [
          {
            "id": "xxxx",
            "type": "function",
            "function": {
              "name": "add",
              "arguments": "{\"a\":2,\"b\":3}"
            }
          }
        ]
      }
    }
  ]
}
```

---

👤 **RodriMora** commented on **2025-08-09** at **16:58:26**

> Text completion looks broken to me. On latest merged main branch I get 500 error using completions endpoint while using GLM4.5 or Deepseek V3 model.
> 
> Request URL http://127.0.0.1:8000/api/backends/text-completions/generate Request Method POST Status Code 500 Internal Server Error
> 
> [json.exception.out_of_range.403] key 'messages' not found
> 
> <img alt="image" width="336" height="134" src="https://private-user-images.githubusercontent.com/908712/476297263-b6590e6f-237d-4f61-9285-b6b78ccd1ade.png?jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NTQ3NTg2ODgsIm5iZiI6MTc1NDc1ODM4OCwicGF0aCI6Ii85MDg3MTIvNDc2Mjk3MjYzLWI2NTkwZTZmLTIzN2QtNGY2MS05Mjg1LWI2Yjc4Y2NkMWFkZS5wbmc_WC1BbXotQWxnb3JpdGhtPUFXUzQtSE1BQy1TSEEyNTYmWC1BbXotQ3JlZGVudGlhbD1BS0lBVkNPRFlMU0E1M1BRSzRaQSUyRjIwMjUwODA5JTJGdXMtZWFzdC0xJTJGczMlMkZhd3M0X3JlcXVlc3QmWC1BbXotRGF0ZT0yMDI1MDgwOVQxNjUzMDhaJlgtQW16LUV4cGlyZXM9MzAwJlgtQW16LVNpZ25hdHVyZT1mZmZjZmU2YmNlZmU5NTdmMzdlZDMzY2ExMTEwODNiYmU5ZjA3MmMxZTUyZjk0ZWFiMTkwYzZmM2IyMTY2Yjk5JlgtQW16LVNpZ25lZEhlYWRlcnM9aG9zdCJ9.1kyq5MQBJx8rVUtW1zAVrt2dtiu5AtvGD2PTp1BO4nE">
> It works fine on Commit [7117c23](https://github.com/ikawrakow/ik_llama.cpp/commit/7117c23de46f37fe7b8300ec0f6fa6c1ead06e18) which was just before jinja merge

just tested, can confirm. Chat completions is fine, but text completions is broken

---

👤 **firecoperana** commented on **2025-08-09** at **20:32:21**

Can you give an example of how you do text completions? I never used it. Tool calls are not expected to work like mainline.

---

👤 **tomt610** commented on **2025-08-09** at **22:31:19**

> Can you give an example of how you do text completions? I never used it. Tool calls are not expected to work like mainline.

curl -X POST http://localhost:8080/completion -H "Content-Type: application/json" -d "{\"prompt\": \"Once upon a time\", \"n_predict\": 128}"
{"error":{"code":500,"message":"[json.exception.out_of_range.403] key 'messages' not found","type":"server_error"}}

---

👤 **firecoperana** commented on **2025-08-10** at **14:01:36**

Fixed in https://github.com/ikawrakow/ik_llama.cpp/pull/684

---

👤 **ubergarm** commented on **2025-08-10** at **15:45:16**

Thanks for digging into all this folks and @firecoperana 

I noticed another open mainline lcpp PR discussing GLM-4.5 Tool Calling here: https://github.com/ggml-org/llama.cpp/pull/15186

Not sure if that effects anything here as I tend to stick to `/chat/completions/` llama-server endpoint and don't mess around with clients doing much function/tool calling so far.

---

👤 **RodriMora** commented on **2025-08-10** at **17:18:11**

I made some changes to make tool calling compliant with openai's api, so it should be now compatible with more agents. If someone can test it it's here (it also has the text completion fix):
https://github.com/RodriMora/ik_llama.cpp

I tested it with Qwen3-Coder-30B-A3B-Instruct (note: the og model uploaded 10 days ago had the chat template wrong for tool calling, and the update was pushed 3 days ago). So I'm using my [own quants](https://huggingface.co/bullerwins/Qwen3-Coder-30B-A3B-Instruct-GGUF) to test as most HF quants are outdated _i think_ 

Testing with both:
- https://github.com/Teachings/FastAgentAPI/blob/master/test_function_call.py
- And this [gist](https://gist.github.com/RodriMora/099913a7cea971d1bd09c623fc12c7bf) I made for simple tests

With my fork I have the same results as with mainline llama.cpp I posted [before](https://github.com/ikawrakow/ik_llama.cpp/pull/677#issuecomment-3171917475) and works fine with Opencode, and still works with Cline and Roocode

---

👤 **trilog-inc** commented on **2025-08-11** at **14:52:19**

I am using Qwen3-235 and the opening <think> tags are no longer returned, which breaks the jinja template when trying to do function calls 

'Value is not callable: null at row 39, column 78:\n            {%- if \'</think>\' in content %}\n                {%- set reasoning_content = ((content.split(\'</think>\')|first).rstrip(\'\\n\').split(\'<think>\')|last).lstrip(\'\\n\') %}

Anyone else?

---

👤 **ubergarm** commented on **2025-08-11** at **15:02:57**

@trilog-inc 

> I am using Qwen3-235 and the opening tags are no longer returned, which breaks the jinja template when trying to do function calls

"no longer returned" seems to imply that you had this working before? I don't use tool/function call stuff, but which llama-server API endpoint are you using with your client e.g. the `/text/completions` or the `/chat/completions` ? 

fwiw this PR was merged about 8 hours ago which effects some code possibly in the path for tool/function calling and JSON stuff: https://github.com/ikawrakow/ik_llama.cpp/pull/684

So you could try rolling back one commit e.g. `git checkout d60c8f4d3bdde` and build (assuming you're reported issue above is with tip of `main@d99cf7cb`

---

👤 **trilog-inc** commented on **2025-08-11** at **15:40:15**

Using chat/completions. I should specify that this affects non-function calling queries too.

Before the jinja template update, the qwen3 model would properly send out the <think> tag ( I checked out the commit you mentioned and it behaves the same as current HEAD). I deleted my previous ik_llama folder but my last pull must have been last week and it was behaving as expected then. Using mainline llama.cpp correctly outputs the <think> token.

I will try to take a look at how llama.cpp handles this tonight. This definitely feels like a consequence of using the jinja template too strictly. 

Current ik_llama output to "What is the capital of France?":

INFO [format_partial_response_oaicompat] DEBUG: Streaming finish_reason check | tid="139886531170304" timestamp=1754925900 generated_text="Okay, the user asked, \"What is the capital of France?\" Hmm, this seems like a very basic geography question. Maybe they're a student doing homework, or someone testing if I know simple facts.  \n\nBut wait—why would anyone ask this in 2024? It's one of the most well-known capitals globally. Could it be a trick? Like, maybe they're checking if I'll overcomplicate it? Or perhaps they're very young or new to learning geography.  \n\nI should just answer directly: Paris. No need for fluff. But since they might be learning, I'll add one extra fact—like the Seine River—to make it slightly educational without overwhelming them.  \n\n...Though part of me wonders if this is a bot testing response accuracy. Either way, short and correct is safest.  \n\n*Types \"Paris\" then pauses*  \nShould I say \"definitely Paris\" to sound confident? Nah, that's overkill. Just clean and factual.\n</think>\n\nThe capital of France is **Paris**.  \n\nIt has been the political, cultural, and economic center of France for centuries and is renowned worldwide for landmarks like the Eiffel Tower, the Louvre Museum, and Notre-Dame Cathedral. 🇫🇷  \n\n*Fun fact:* Paris is built along the Seine River and is often called \"La Ville Lumière\" (The City of Light) due to its early adoption of street lighting and its role as a center of education and ideas during the Age of Enlightenment." model_name="qwen3" tool_calls_count=0

---

👤 **firecoperana** commented on **2025-08-11** at **16:13:34**

Do you use jinja flag when the openning tag is not returned? What do you send to chat completions?

---

👤 **trilog-inc** commented on **2025-08-11** at **16:38:49**

This is my full command. Happens with or without --jinja

./build/bin/llama-server --model /mnt/home_extend/models/unsloth_Qwen3-235B-A22B-Thinking-2507-GGUF/Q6_K/Qwen3-235B-A22B-Thinking-2507-Q6_K-00001-of-00004.gguf --temp 0.6 --top-k 20 --top-p 0.8 --min-p 0 --repeat-penalty 1.0 --presence-penalty 2.0 -fmoe -fa --cache-type-k q8_0 --cache-type-v q8_0 --n-gpu-layers 99 --alias qwen3 --host 0.0.0.0 --port 8099 --no-mmap --ctx-size 120000  --override-tensor "blk\.(?:[1-9]?[01235789])\.ffn_.*_exps\.weight=CPU" --jinja -ub 4096 -amb 4096

---

👤 **firecoperana** commented on **2025-08-11** at **16:52:09**

Do you have this issue when using the built-in webui?

---

👤 **nimishchaudhari** commented on **2025-08-27** at **17:03:35**

Hello, 

Reporting that GLM 4.5 Air's template didn't work only by using --jinja flag on claude code (using claude code router). 

I managed to fix it using the template below: 

```
[gMASK]<sop>
{%- if tools -%}
<|system|>
# Tools

You may call one or more functions to assist with the user query.

You are provided with function signatures within <tools></tools> XML tags:
<tools>
{% for tool in tools %}
{{ tool | tojson }}
{% endfor %}
</tools>

For each function call, output the function name and arguments within the following XML format:
<tool_call>{function-name}
<arg_key>{arg-key-1}</arg_key>
<arg_value>{arg-value-1}</arg_value>
<arg_key>{arg-key-2}</arg_key>
<arg_value>{arg-value-2}</arg_value>
...
</tool_call>{%- endif -%}
{%- macro visible_text(content) -%}
    {%- if content is string -%}
        {{- content }}
    {%- elif content is iterable and content is not mapping -%}
        {%- for item in content -%}
            {%- if item is mapping and item.type == 'text' -%}
                {{- item.text }}
            {%- elif item is string -%}
                {{- item }}
            {%- endif -%}
        {%- endfor -%}
    {%- else -%}
        {{- content }}
    {%- endif -%}
{%- endmacro -%}
{%- set ns = namespace(last_user_index=-1) %}
{%- for m in messages %}
    {%- if m.role == 'user' %}
        {% set ns.last_user_index = loop.index0 -%}
    {%- endif %}
{%- endfor %}
{% for m in messages %}
{%- if m.role == 'user' -%}<|user|>
{%- set user_content = visible_text(m.content) -%}
{{ user_content }}
{%- if enable_thinking is defined and not enable_thinking -%}
{%- if not user_content.endswith("/nothink") -%}
{{- '/nothink' -}}
{%- endif -%}
{%- endif -%}
{%- elif m.role == 'assistant' -%}
<|assistant|>
{%- set reasoning_content = '' %}
{%- set content = visible_text(m.content) %}
{%- if m.reasoning_content is string %}
    {%- set reasoning_content = m.reasoning_content %}
{%- else %}
    {%- if '</think>' in content %}
        {%- set reasoning_content = content.split('</think>')[0].rstrip('\n').split('<think>')[-1].lstrip('\n') %}
        {%- set content = content.split('</think>')[-1].lstrip('\n') %}
    {%- endif %}
{%- endif %}
{%- if loop.index0 > ns.last_user_index and reasoning_content -%}
{{ '\n<think>' + reasoning_content.strip() +  '</think>'}}
{%- else -%}
{{ '\n<think></think>' }}
{%- endif -%}
{%- if content.strip() -%}
{{ '\n' + content.strip() }}
{%- endif -%}
{% if m.tool_calls %}
{% for tc in m.tool_calls %}
{%- if tc.function %}
    {%- set tc = tc.function %}
{%- endif %}
{{ '\n<tool_call>' + tc.name }}
{% set _args = tc.arguments %}
{% for k, v in _args.items() %}
<arg_key>{{ k }}</arg_key>
<arg_value>{{ v | tojson if v is not string else v }}</arg_value>
{% endfor %}
</tool_call>{% endfor %}
{% endif %}
{%- elif m.role == 'tool' -%}
{%- if m.content is string -%}
{%- if loop.first or (messages[loop.index0 - 1].role != "tool") %}
    {{- '<|observation|>' }}
{%- endif %}
{{- '\n<tool_response>\n' }}
{{- m.content }}
{{- '\n</tool_response>' }}
{%- else -%}
<|observation|>{% for tr in m.content %}

<tool_response>
{{ tr.output if tr.output is defined else tr }}
</tool_response>{% endfor -%}
{% endif -%}
{%- elif m.role == 'system' -%}
<|system|>
{{ visible_text(m.content) }}
{%- endif -%}
{%- endfor -%}
{%- if add_generation_prompt -%}
    <|assistant|>{{- '\n<think></think>' if (enable_thinking is defined and not enable_thinking) else '' -}}
{%- endif -%}
```


Found this working template in https://github.com/ggml-org/llama.cpp/pull/15186 

My final running command: 

``` 
     /home/nimish/Programs/ik_llama.cpp/build/bin/llama-server \
      -m '/home/nimish/Dev/Models/ik_llama/GLM-4.5-Air-IQ2_KL.gguf' \ 
      -ctkd q8_0 -ctvd q8_0 -md '/home/nimish/Dev/Models/ik_llama/GLM-4.5-DRAFT-0.6B-32k-Q4_0.gguf' \ # Using a draft model
      -c 32768 \
      --chat-template-file '/home/nimish/Dev/Models/ik_llama/glm_4.5_template.txt' --jinja \
      -fa -fmoe -amb 512 \
      -ctk q8_0 -ctv q8_0 \
      -ngl 24 \
      --gpu-layers-draft 99\
      --threads 8 \
      --port ${PORT}
 ```

Funny story: I have a RTX 3090 and a RTX 2060 with 32GB DDR5 on a i7 14700F but I manage to make glm 4.5 air runnable on my setup using this draft model I mentioned above:
< With Draft model >
Prompt
- Tokens: 1
**- Time: 103.512 ms
- Speed: 9.7 t/s**
Generation
- Tokens: 3537
- Time: 338829.416 ms
- Speed: 10.4 t/s

< Without Draft model >
Prompt
- Tokens: 31
**- Time: 9176.051 ms
- Speed: 3.4 t/s**
Generation
- Tokens: 2645
- Time: 318602.521 ms
- Speed: 8.3 t/s

Hope this helps.

---

👤 **ubergarm** commented on **2025-08-28** at **15:32:23**

@nimishchaudhari 

Thanks for the link to a working jinja chat template for Air, I opened a discussion on the huggingface repo here: https://huggingface.co/ubergarm/GLM-4.5-Air-GGUF/discussions/6 with your notes and some comments and feedback for your command. Cheers!