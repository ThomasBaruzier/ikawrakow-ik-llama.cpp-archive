## 📌 [Issue #822](https://github.com/ikawrakow/ik_llama.cpp/issues/822) - Bug: llama.cpp vs ik-llama.cpp VRAM usage discrepancy (gpt-oss-20b, maybe other models)

| **Author** | `crat0z` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-10-10 |
| **Updated** | 2025-10-16 |
| **Labels** | `wontfix` |

---

## 📄 Description

### What happened?

Hello, I'm trying to use `gpt-oss-20b` with ik-llama just like I use it with llama.cpp, but I'm going OOM at startup (I'm GPU poor)

```
~/llamacpp/llama.cpp/build-cuda/bin/llama-server -m gpt-oss-20b-mxfp4.gguf -ngl 99 --n-cpu-moe 99 -c 131072 -fa on --reasoning-format auto --jinja
```

```
ggml_cuda_init: GGML_CUDA_FORCE_MMQ:    no
ggml_cuda_init: GGML_CUDA_FORCE_CUBLAS: no
ggml_cuda_init: found 1 CUDA devices:
  Device 0: NVIDIA GeForce RTX 3070, compute capability 8.6, VMM: yes
build: 6699 (c08002a1) with cc (GCC) 14.3.1 20250808 (Red Hat 14.3.1-3) for x86_64-redhat-linux
system info: n_threads = 10, n_threads_batch = 10, total_threads = 20

system_info: n_threads = 10 (n_threads_batch = 10) / 20 | CUDA : ARCHS = 860 | USE_GRAPHS = 1 | PEER_MAX_BATCH_SIZE = 128 | CPU : SSE3 = 1 | SSSE3 = 1 | AVX = 1 | AVX2 = 1 | F16C = 1 | FMA = 1 | BMI2 = 1 | LLAMAFILE = 1 | OPENMP = 1 | REPACK = 1 | 

main: binding port with default address family
main: HTTP server is listening, hostname: 127.0.0.1, port: 8080, http threads: 19
main: loading model
srv    load_model: loading model 'gpt-oss-20b-mxfp4.gguf'
llama_model_load_from_file_impl: using device CUDA0 (NVIDIA GeForce RTX 3070) (0000:01:00.0) - 7672 MiB free
llama_model_loader: loaded meta data with 35 key-value pairs and 459 tensors from gpt-oss-20b-mxfp4.gguf (version GGUF V3 (latest))
llama_model_loader: Dumping metadata keys/values. Note: KV overrides do not apply in this output.
llama_model_loader: - kv   0:                       general.architecture str              = gpt-oss
llama_model_loader: - kv   1:                               general.type str              = model
llama_model_loader: - kv   2:                               general.name str              = Gpt Oss 20b
llama_model_loader: - kv   3:                           general.basename str              = gpt-oss
llama_model_loader: - kv   4:                         general.size_label str              = 20B
llama_model_loader: - kv   5:                            general.license str              = apache-2.0
llama_model_loader: - kv   6:                               general.tags arr[str,2]       = ["vllm", "text-generation"]
llama_model_loader: - kv   7:                        gpt-oss.block_count u32              = 24
llama_model_loader: - kv   8:                     gpt-oss.context_length u32              = 131072
llama_model_loader: - kv   9:                   gpt-oss.embedding_length u32              = 2880
llama_model_loader: - kv  10:                gpt-oss.feed_forward_length u32              = 2880
llama_model_loader: - kv  11:               gpt-oss.attention.head_count u32              = 64
llama_model_loader: - kv  12:            gpt-oss.attention.head_count_kv u32              = 8
llama_model_loader: - kv  13:                     gpt-oss.rope.freq_base f32              = 150000.000000
llama_model_loader: - kv  14:   gpt-oss.attention.layer_norm_rms_epsilon f32              = 0.000010
llama_model_loader: - kv  15:                       gpt-oss.expert_count u32              = 32
llama_model_loader: - kv  16:                  gpt-oss.expert_used_count u32              = 4
llama_model_loader: - kv  17:               gpt-oss.attention.key_length u32              = 64
llama_model_loader: - kv  18:             gpt-oss.attention.value_length u32              = 64
llama_model_loader: - kv  19:           gpt-oss.attention.sliding_window u32              = 128
llama_model_loader: - kv  20:         gpt-oss.expert_feed_forward_length u32              = 2880
llama_model_loader: - kv  21:                  gpt-oss.rope.scaling.type str              = yarn
llama_model_loader: - kv  22:                gpt-oss.rope.scaling.factor f32              = 32.000000
llama_model_loader: - kv  23: gpt-oss.rope.scaling.original_context_length u32              = 4096
llama_model_loader: - kv  24:                       tokenizer.ggml.model str              = gpt2
llama_model_loader: - kv  25:                         tokenizer.ggml.pre str              = gpt-4o
llama_model_loader: - kv  26:                      tokenizer.ggml.tokens arr[str,201088]  = ["!", "\"", "#", "$", "%", "&", "'", ...
llama_model_loader: - kv  27:                  tokenizer.ggml.token_type arr[i32,201088]  = [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, ...
llama_model_loader: - kv  28:                      tokenizer.ggml.merges arr[str,446189]  = ["Ġ Ġ", "Ġ ĠĠĠ", "ĠĠ ĠĠ", "...
llama_model_loader: - kv  29:                tokenizer.ggml.bos_token_id u32              = 199998
llama_model_loader: - kv  30:                tokenizer.ggml.eos_token_id u32              = 200002
llama_model_loader: - kv  31:            tokenizer.ggml.padding_token_id u32              = 199999
llama_model_loader: - kv  32:                    tokenizer.chat_template str              = {#-\n  In addition to the normal input...
llama_model_loader: - kv  33:               general.quantization_version u32              = 2
llama_model_loader: - kv  34:                          general.file_type u32              = 38
llama_model_loader: - type  f32:  289 tensors
llama_model_loader: - type q8_0:   98 tensors
llama_model_loader: - type mxfp4:   72 tensors
print_info: file format = GGUF V3 (latest)
print_info: file type   = MXFP4 MoE
print_info: file size   = 11.27 GiB (4.63 BPW) 
load: printing all EOG tokens:
load:   - 199999 ('<|endoftext|>')
load:   - 200002 ('<|return|>')
load:   - 200007 ('<|end|>')
load:   - 200012 ('<|call|>')
load: special_eog_ids contains both '<|return|>' and '<|call|>' tokens, removing '<|end|>' token from EOG list
load: special tokens cache size = 21
load: token to piece cache size = 1.3332 MB
print_info: arch             = gpt-oss
print_info: vocab_only       = 0
print_info: n_ctx_train      = 131072
print_info: n_embd           = 2880
print_info: n_layer          = 24
print_info: n_head           = 64
print_info: n_head_kv        = 8
print_info: n_rot            = 64
print_info: n_swa            = 128
print_info: is_swa_any       = 1
print_info: n_embd_head_k    = 64
print_info: n_embd_head_v    = 64
print_info: n_gqa            = 8
print_info: n_embd_k_gqa     = 512
print_info: n_embd_v_gqa     = 512
print_info: f_norm_eps       = 0.0e+00
print_info: f_norm_rms_eps   = 1.0e-05
print_info: f_clamp_kqv      = 0.0e+00
print_info: f_max_alibi_bias = 0.0e+00
print_info: f_logit_scale    = 0.0e+00
print_info: f_attn_scale     = 0.0e+00
print_info: n_ff             = 2880
print_info: n_expert         = 32
print_info: n_expert_used    = 4
print_info: causal attn      = 1
print_info: pooling type     = 0
print_info: rope type        = 2
print_info: rope scaling     = yarn
print_info: freq_base_train  = 150000.0
print_info: freq_scale_train = 0.03125
print_info: n_ctx_orig_yarn  = 4096
print_info: rope_finetuned   = unknown
print_info: model type       = 20B
print_info: model params     = 20.91 B
print_info: general.name     = Gpt Oss 20b
print_info: n_ff_exp         = 2880
print_info: vocab type       = BPE
print_info: n_vocab          = 201088
print_info: n_merges         = 446189
print_info: BOS token        = 199998 '<|startoftext|>'
print_info: EOS token        = 200002 '<|return|>'
print_info: EOT token        = 199999 '<|endoftext|>'
print_info: PAD token        = 199999 '<|endoftext|>'
print_info: LF token         = 198 'Ċ'
print_info: EOG token        = 199999 '<|endoftext|>'
print_info: EOG token        = 200002 '<|return|>'
print_info: EOG token        = 200012 '<|call|>'
print_info: max token length = 256
load_tensors: loading model tensors, this can take a while... (mmap = true)
load_tensors: offloading 24 repeating layers to GPU
load_tensors: offloading output layer to GPU
load_tensors: offloaded 25/25 layers to GPU
load_tensors:        CUDA0 model buffer size =  1242.03 MiB
load_tensors:   CPU_Mapped model buffer size = 10949.33 MiB
................................................................................
llama_context: constructing llama_context
llama_context: n_seq_max     = 1
llama_context: n_ctx         = 131072
llama_context: n_ctx_per_seq = 131072
llama_context: n_batch       = 2048
llama_context: n_ubatch      = 512
llama_context: causal_attn   = 1
llama_context: flash_attn    = enabled
llama_context: kv_unified    = false
llama_context: freq_base     = 150000.0
llama_context: freq_scale    = 0.03125
llama_context:  CUDA_Host  output buffer size =     0.77 MiB
llama_kv_cache_iswa: creating non-SWA KV cache, size = 131072 cells
llama_kv_cache:      CUDA0 KV buffer size =  3072.00 MiB
llama_kv_cache: size = 3072.00 MiB (131072 cells,  12 layers,  1/1 seqs), K (f16): 1536.00 MiB, V (f16): 1536.00 MiB
llama_kv_cache_iswa: creating     SWA KV cache, size = 768 cells
llama_kv_cache:      CUDA0 KV buffer size =    18.00 MiB
llama_kv_cache: size =   18.00 MiB (   768 cells,  12 layers,  1/1 seqs), K (f16):    9.00 MiB, V (f16):    9.00 MiB
llama_context:      CUDA0 compute buffer size =   456.98 MiB
llama_context:  CUDA_Host compute buffer size =   263.15 MiB
llama_context: graph nodes  = 1352
llama_context: graph splits = 146 (with bs=512), 50 (with bs=1)
common_init_from_params: added <|endoftext|> logit bias = -inf
common_init_from_params: added <|return|> logit bias = -inf
common_init_from_params: added <|call|> logit bias = -inf
common_init_from_params: setting dry_penalty_last_n to ctx_size = 131072
common_init_from_params: warming up the model with an empty run - please wait ... (--no-warmup to disable)
srv          init: initializing slots, n_slots = 1
slot         init: id  0 | task -1 | new slot n_ctx_slot = 131072
srv          init: Enable thinking? 0
main: model loaded
main: chat template, chat_template: {#-
  In addition to the normal inputs of `messages` and `tools`, this template also accepts the
  following kwargs:
  - "builtin_tools": A list, can contain "browser" and/or "python".
  - "model_identity": A string that optionally describes the model identity.
  - "reasoning_effort": A string that describes the reasoning effort, defaults to "medium".
 #}

{#- Tool Definition Rendering ============================================== #}
{%- macro render_typescript_type(param_spec, required_params, is_nullable=false) -%}
    {%- if param_spec.type == "array" -%}
        {%- if param_spec['items'] -%}
            {%- if param_spec['items']['type'] == "string" -%}
                {{- "string[]" }}
            {%- elif param_spec['items']['type'] == "number" -%}
                {{- "number[]" }}
            {%- elif param_spec['items']['type'] == "integer" -%}
                {{- "number[]" }}
            {%- elif param_spec['items']['type'] == "boolean" -%}
                {{- "boolean[]" }}
            {%- else -%}
                {%- set inner_type = render_typescript_type(param_spec['items'], required_params) -%}
                {%- if inner_type == "object | object" or inner_type|length > 50 -%}
                    {{- "any[]" }}
                {%- else -%}
                    {{- inner_type + "[]" }}
                {%- endif -%}
            {%- endif -%}
            {%- if param_spec.nullable -%}
                {{- " | null" }}
            {%- endif -%}
        {%- else -%}
            {{- "any[]" }}
            {%- if param_spec.nullable -%}
                {{- " | null" }}
            {%- endif -%}
        {%- endif -%}
    {%- elif param_spec.type is defined and param_spec.type is iterable and param_spec.type is not string and param_spec.type is not mapping and param_spec.type[0] is defined -%}
        {#- Handle array of types like ["object", "object"] from Union[dict, list] #}
        {%- if param_spec.type | length > 1 -%}
            {{- param_spec.type | join(" | ") }}
        {%- else -%}
            {{- param_spec.type[0] }}
        {%- endif -%}
    {%- elif param_spec.oneOf -%}
        {#- Handle oneOf schemas - check for complex unions and fallback to any #}
        {%- set has_object_variants = false -%}
        {%- for variant in param_spec.oneOf -%}
            {%- if variant.type == "object" -%}
                {%- set has_object_variants = true -%}
            {%- endif -%}
        {%- endfor -%}
        {%- if has_object_variants and param_spec.oneOf|length > 1 -%}
            {{- "any" }}
        {%- else -%}
            {%- for variant in param_spec.oneOf -%}
                {{- render_typescript_type(variant, required_params) -}}
                {%- if variant.description %}
                    {{- "// " + variant.description }}
                {%- endif -%}
                {%- if variant.default is defined %}
                    {{ "// default: " + variant.default|tojson }}
                {%- endif -%}
                {%- if not loop.last %}
                    {{- " | " }}
                {% endif -%}
            {%- endfor -%}
        {%- endif -%}
    {%- elif param_spec.type == "string" -%}
        {%- if param_spec.enum -%}
            {{- '"' + param_spec.enum|join('" | "') + '"' -}}
        {%- else -%}
            {{- "string" }}
            {%- if param_spec.nullable %}
                {{- " | null" }}
            {%- endif -%}
        {%- endif -%}
    {%- elif param_spec.type == "number" -%}
        {{- "number" }}
    {%- elif param_spec.type == "integer" -%}
        {{- "number" }}
    {%- elif param_spec.type == "boolean" -%}
        {{- "boolean" }}

    {%- elif param_spec.type == "object" -%}
        {%- if param_spec.properties -%}
            {{- "{\n" }}
            {%- for prop_name, prop_spec in param_spec.properties.items() -%}
                {{- prop_name -}}
                {%- if prop_name not in (param_spec.required or []) -%}
                    {{- "?" }}
                {%- endif -%}
                {{- ": " }}
                {{ render_typescript_type(prop_spec, param_spec.required or []) }}
                {%- if not loop.last -%}
                    {{-", " }}
                {%- endif -%}
            {%- endfor -%}
            {{- "}" }}
        {%- else -%}
            {{- "object" }}
        {%- endif -%}
    {%- else -%}
        {{- "any" }}
    {%- endif -%}
{%- endmacro -%}

{%- macro render_tool_namespace(namespace_name, tools) -%}
    {{- "## " + namespace_name + "\n\n" }}
    {{- "namespace " + namespace_name + " {\n\n" }}
    {%- for tool in tools %}
        {%- set tool = tool.function %}
        {{- "// " + tool.description + "\n" }}
        {{- "type "+ tool.name + " = " }}
        {%- if tool.parameters and tool.parameters.properties %}
            {{- "(_: {\n" }}
            {%- for param_name, param_spec in tool.parameters.properties.items() %}
                {%- if param_spec.description %}
                    {{- "// " + param_spec.description + "\n" }}
                {%- endif %}
                {{- param_name }}
                {%- if param_name not in (tool.parameters.required or []) -%}
                    {{- "?" }}
                {%- endif -%}
                {{- ": " }}
                {{- render_typescript_type(param_spec, tool.parameters.required or []) }}
                {%- if param_spec.default is defined -%}
                    {%- if param_spec.enum %}
                        {{- ", // default: " + param_spec.default }}
                    {%- elif param_spec.oneOf %}
                        {{- "// default: " + param_spec.default }}
                    {%- else %}
                        {{- ", // default: " + param_spec.default|tojson }}
                    {%- endif -%}
                {%- endif -%}
                {%- if not loop.last %}
                    {{- ",\n" }}
                {%- else %}
                    {{- ",\n" }}
                {%- endif -%}
            {%- endfor %}
            {{- "}) => any;\n\n" }}
        {%- else -%}
            {{- "() => any;\n\n" }}
        {%- endif -%}
    {%- endfor %}
    {{- "} // namespace " + namespace_name }}
{%- endmacro -%}

{%- macro render_builtin_tools(browser_tool, python_tool) -%}
    {%- if browser_tool %}
        {{- "## browser\n\n" }}
        {{- "// Tool for browsing.\n" }}
        {{- "// The `cursor` appears in brackets before each browsing display: `[{cursor}]`.\n" }}
        {{- "// Cite information from the tool using the following format:\n" }}
        {{- "// `【{cursor}†L{line_start}(-L{line_end})?】`, for example: `【6†L9-L11】` or `【8†L3】`.\n" }}
        {{- "// Do not quote more than 10 words directly from the tool output.\n" }}
        {{- "// sources=web (default: web)\n" }}
        {{- "namespace browser {\n\n" }}
        {{- "// Searches for information related to `query` and displays `topn` results.\n" }}
        {{- "type search = (_: {\n" }}
        {{- "query: string,\n" }}
        {{- "topn?: number, // default: 10\n" }}
        {{- "source?: string,\n" }}
        {{- "}) => any;\n\n" }}
        {{- "// Opens the link `id` from the page indicated by `cursor` starting at line number `loc`, showing `num_lines` lines.\n" }}
        {{- "// Valid link ids are displayed with the formatting: `【{id}†.*】`.\n" }}
        {{- "// If `cursor` is not provided, the most recent page is implied.\n" }}
        {{- "// If `id` is a string, it is treated as a fully qualified URL associated with `source`.\n" }}
        {{- "// If `loc` is not provided, the viewport will be positioned at the beginning of the document or centered on the most relevant passage, if available.\n" }}
        {{- "// Use this function without `id` to scroll to a new location of an opened page.\n" }}
        {{- "type open = (_: {\n" }}
        {{- "id?: number | string, // default: -1\n" }}
        {{- "cursor?: number, // default: -1\n" }}
        {{- "loc?: number, // default: -1\n" }}
        {{- "num_lines?: number, // default: -1\n" }}
        {{- "view_source?: boolean, // default: false\n" }}
        {{- "source?: string,\n" }}
        {{- "}) => any;\n\n" }}
        {{- "// Finds exact matches of `pattern` in the current page, or the page given by `cursor`.\n" }}
        {{- "type find = (_: {\n" }}
        {{- "pattern: string,\n" }}
        {{- "cursor?: number, // default: -1\n" }}
        {{- "}) => any;\n\n" }}
        {{- "} // namespace browser\n\n" }}
    {%- endif -%}

    {%- if python_tool %}
        {{- "## python\n\n" }}
        {{- "Use this tool to execute Python code in your chain of thought. The code will not be shown to the user. This tool should be used for internal reasoning, but not for code that is intended to be visible to the user (e.g. when creating plots, tables, or files).\n\n" }}
        {{- "When you send a message containing Python code to python, it will be executed in a stateful Jupyter notebook environment. python will respond with the output of the execution or time out after 120.0 seconds. The drive at '/mnt/data' can be used to save and persist user files. Internet access for this session is UNKNOWN. Depends on the cluster.\n\n" }}
    {%- endif -%}
{%- endmacro -%}

{#- System Message Construction ============================================ #}
{%- macro build_system_message() -%}
    {%- if model_identity is not defined %}
        {%- set model_identity = "You are ChatGPT, a large language model trained by OpenAI." %}
    {%- endif %}
    {{- model_identity + "\n" }}
    {{- "Knowledge cutoff: 2024-06\n" }}
    {{- "Current date: " + strftime_now("%Y-%m-%d") + "\n\n" }}
    {%- if reasoning_effort is not defined %}
        {%- set reasoning_effort = "medium" %}
    {%- endif %}
    {{- "Reasoning: " + reasoning_effort + "\n\n" }}
    {%- if builtin_tools %}
        {{- "# Tools\n\n" }}
        {%- set available_builtin_tools = namespace(browser=false, python=false) %}
        {%- for tool in builtin_tools %}
            {%- if tool == "browser" %}
                {%- set available_builtin_tools.browser = true %}
            {%- elif tool == "python" %}
                {%- set available_builtin_tools.python = true %}
            {%- endif %}
        {%- endfor %}
        {{- render_builtin_tools(available_builtin_tools.browser, available_builtin_tools.python) }}
    {%- endif -%}
    {{- "# Valid channels: analysis, commentary, final. Channel must be included for every message." }}
    {%- if tools -%}
        {{- "\nCalls to these tools must go to the commentary channel: 'functions'." }}
    {%- endif -%}
{%- endmacro -%}

{#- Main Template Logic ================================================= #}
{#- Set defaults #}

{#- Render system message #}
{{- "<|start|>system<|message|>" }}
{{- build_system_message() }}
{{- "<|end|>" }}

{#- Extract developer message #}
{%- if messages[0].role == "developer" or messages[0].role == "system" %}
    {%- set developer_message = messages[0].content %}
    {%- set loop_messages = messages[1:] %}
{%- else %}
    {%- set developer_message = "" %}
    {%- set loop_messages = messages %}
{%- endif %}

{#- Render developer message #}
{%- if developer_message or tools %}
    {{- "<|start|>developer<|message|>" }}
    {%- if developer_message %}
        {{- "# Instructions\n\n" }}
        {{- developer_message }}
        {{- "\n\n" }}
    {%- endif %}
    {%- if tools -%}
        {{- "# Tools\n\n" }}
        {{- render_tool_namespace("functions", tools) }}
    {%- endif -%}
    {{- "<|end|>" }}
{%- endif %}

{#- Render messages #}
{%- set last_tool_call = namespace(name=none) %}
{%- for message in loop_messages -%}
    {#- At this point only assistant/user/tool messages should remain #}
    {%- if message.role == 'assistant' -%}
        {#- Checks to ensure the messages are being passed in the format we expect #}
        {%- if "content" in message %}
            {%- if false %}
                {{- raise_exception("You have passed a message containing <|channel|> tags in the content field. Instead of doing this, you should pass analysis messages (the string between '<|message|>' and '<|end|>') in the 'thinking' field, and final messages (the string between '<|message|>' and '<|end|>') in the 'content' field.") }}
            {%- endif %}
        {%- endif %}
        {%- if "thinking" in message %}
            {%- if "<|channel|>analysis<|message|>" in message.thinking or "<|channel|>final<|message|>" in message.thinking %}
                {{- raise_exception("You have passed a message containing <|channel|> tags in the thinking field. Instead of doing this, you should pass analysis messages (the string between '<|message|>' and '<|end|>') in the 'thinking' field, and final messages (the string between '<|message|>' and '<|end|>') in the 'content' field.") }}
            {%- endif %}
        {%- endif %}
        {%- if "tool_calls" in message %}
            {#- We need very careful handling here - we want to drop the tool call analysis message if the model #}
            {#- has output a later <|final|> message, but otherwise we want to retain it. This is the only case #}
            {#- when we render CoT/analysis messages in inference. #}
            {%- set future_final_message = namespace(found=false) %}
            {%- for future_message in loop_messages[loop.index:] %}
                {%- if future_message.role == 'assistant' and "tool_calls" not in future_message %}
                    {%- set future_final_message.found = true %}
                {%- endif %}
            {%- endfor %}
            {#- We assume max 1 tool call per message, and so we infer the tool call name #}
            {#- in "tool" messages from the most recent assistant tool call name #}
            {%- set tool_call = message.tool_calls[0] %}
            {%- if tool_call.function %}
                {%- set tool_call = tool_call.function %}
            {%- endif %}
            {%- if message.content and message.thinking %}
                {{- raise_exception("Cannot pass both content and thinking in an assistant message with tool calls! Put the analysis message in one or the other, but not both.") }}
            {%- elif message.content and not future_final_message.found %}
                {{- "<|start|>assistant<|channel|>analysis<|message|>" + message.content + "<|end|>" }}
            {%- elif message.thinking and not future_final_message.found %}
                {{- "<|start|>assistant<|channel|>analysis<|message|>" + message.thinking + "<|end|>" }}
            {%- endif %}
            {{- "<|start|>assistant to=" }}
            {{- "functions." + tool_call.name + "<|channel|>commentary " }}
            {{- (tool_call.content_type if tool_call.content_type is defined else "json") + "<|message|>" }}
            {{- tool_call.arguments|tojson }}
            {{- "<|call|>" }}
            {%- set last_tool_call.name = tool_call.name %}
        {%- elif loop.last and not add_generation_prompt %}
            {#- Only render the CoT if the final turn is an assistant turn and add_generation_prompt is false #}
            {#- This is a situation that should only occur in training, never in inference. #}
            {%- if "thinking" in message %}
                {{- "<|start|>assistant<|channel|>analysis<|message|>" + message.thinking + "<|end|>" }}
            {%- endif %}
            {#- <|return|> indicates the end of generation, but <|end|> does not #}
            {#- <|return|> should never be an input to the model, but we include it as the final token #}
            {#- when training, so the model learns to emit it. #}
            {{- "<|start|>assistant<|channel|>final<|message|>" + message.content + "<|return|>" }}
        {%- else %}
            {#- CoT is dropped during all previous turns, so we never render it for inference #}
            {{- "<|start|>assistant<|channel|>final<|message|>" + message.content + "<|end|>" }}
            {%- set last_tool_call.name = none %}
        {%- endif %}
    {%- elif message.role == 'tool' -%}
        {%- if last_tool_call.name is none %}
            {{- raise_exception("Message has tool role, but there was no previous assistant message with a tool call!") }}
        {%- endif %}
        {{- "<|start|>functions." + last_tool_call.name }}
        {{- " to=assistant<|channel|>commentary<|message|>" + message.content|tojson + "<|end|>" }}
    {%- elif message.role == 'user' -%}
        {{- "<|start|>user<|message|>" + message.content + "<|end|>" }}
    {%- endif -%}
{%- endfor -%}

{#- Generation prompt #}
{%- if add_generation_prompt -%}
<|start|>assistant
{%- endif -%}, example_format: '<|start|>system<|message|>You are ChatGPT, a large language model trained by OpenAI.
Knowledge cutoff: 2024-06
Current date: 2025-10-10

Reasoning: medium

# Valid channels: analysis, commentary, final. Channel must be included for every message.<|end|><|start|>developer<|message|># Instructions

You are a helpful assistant

<|end|><|start|>user<|message|>Hello<|end|><|start|>assistant<|channel|>final<|message|>Hi there<|end|><|start|>user<|message|>How are you?<|end|><|start|>assistant'
main: server is listening on http://127.0.0.1:8080 - starting the main loop
srv  update_slots: all slots are idle
```

```
Fri Oct 10 10:09:08 2025       
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 580.82.09              Driver Version: 580.82.09      CUDA Version: 13.0     |
+-----------------------------------------+------------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
|                                         |                        |               MIG M. |
|=========================================+========================+======================|
|   0  NVIDIA GeForce RTX 3070        Off |   00000000:01:00.0 Off |                  N/A |
| 65%   32C    P2             38W /  220W |    4986MiB /   8192MiB |      0%      Default |
|                                         |                        |                  N/A |
+-----------------------------------------+------------------------+----------------------+

+-----------------------------------------------------------------------------------------+
| Processes:                                                                              |
|  GPU   GI   CI              PID   Type   Process name                        GPU Memory |
|        ID   ID                                                               Usage      |
|=========================================================================================|
|    0   N/A  N/A         3401326      C   ...p/build-cuda/bin/llama-server       4972MiB |
+-----------------------------------------------------------------------------------------+
```

Loads perfectly, with 3GB to spare. Now ik-llama, with the equivalent command AFAIK:

```
~/ik-llamacpp/ik_llama.cpp/build-cuda/bin/llama-server -m gpt-oss-20b-mxfp4.gguf -ngl 99 --n-cpu-moe 99 -c 131072 --reasoning-format auto --jinja # same result with `-ot "exps=CPU"`
```

```
INFO [                    main] build info | tid="139668388159488" timestamp=1760105534 build=3902 commit="f649e36a"
INFO [                    main] system info | tid="139668388159488" timestamp=1760105534 n_threads=10 n_threads_batch=-1 total_threads=20 system_info="AVX = 1 | AVX_VNNI = 0 | AVX2 = 1 | AVX512 = 0 | AVX512_VBMI = 0 | AVX512_VNNI = 0 | AVX512_BF16 = 0 | FMA = 1 | NEON = 0 | SVE = 0 | ARM_FMA = 0 | F16C = 1 | FP16_VA = 0 | WASM_SIMD = 0 | BLAS = 1 | SSE3 = 1 | SSSE3 = 1 | VSX = 0 | MATMUL_INT8 = 0 | LLAMAFILE = 1 | "
llama_model_loader: loaded meta data with 35 key-value pairs and 459 tensors from gpt-oss-20b-mxfp4.gguf (version GGUF V3 (latest))
llama_model_loader: Dumping metadata keys/values. Note: KV overrides do not apply in this output.
llama_model_loader: - kv   0:                       general.architecture str              = gpt-oss
llama_model_loader: - kv   1:                               general.type str              = model
llama_model_loader: - kv   2:                               general.name str              = Gpt Oss 20b
llama_model_loader: - kv   3:                           general.basename str              = gpt-oss
llama_model_loader: - kv   4:                         general.size_label str              = 20B
llama_model_loader: - kv   5:                            general.license str              = apache-2.0
llama_model_loader: - kv   6:                               general.tags arr[str,2]       = ["vllm", "text-generation"]
llama_model_loader: - kv   7:                        gpt-oss.block_count u32              = 24
llama_model_loader: - kv   8:                     gpt-oss.context_length u32              = 131072
llama_model_loader: - kv   9:                   gpt-oss.embedding_length u32              = 2880
llama_model_loader: - kv  10:                gpt-oss.feed_forward_length u32              = 2880
llama_model_loader: - kv  11:               gpt-oss.attention.head_count u32              = 64
llama_model_loader: - kv  12:            gpt-oss.attention.head_count_kv u32              = 8
llama_model_loader: - kv  13:                     gpt-oss.rope.freq_base f32              = 150000.000000
llama_model_loader: - kv  14:   gpt-oss.attention.layer_norm_rms_epsilon f32              = 0.000010
llama_model_loader: - kv  15:                       gpt-oss.expert_count u32              = 32
llama_model_loader: - kv  16:                  gpt-oss.expert_used_count u32              = 4
llama_model_loader: - kv  17:               gpt-oss.attention.key_length u32              = 64
llama_model_loader: - kv  18:             gpt-oss.attention.value_length u32              = 64
llama_model_loader: - kv  19:           gpt-oss.attention.sliding_window u32              = 128
llama_model_loader: - kv  20:         gpt-oss.expert_feed_forward_length u32              = 2880
llama_model_loader: - kv  21:                  gpt-oss.rope.scaling.type str              = yarn
llama_model_loader: - kv  22:                gpt-oss.rope.scaling.factor f32              = 32.000000
llama_model_loader: - kv  23: gpt-oss.rope.scaling.original_context_length u32              = 4096
llama_model_loader: - kv  24:                       tokenizer.ggml.model str              = gpt2
llama_model_loader: - kv  25:                         tokenizer.ggml.pre str              = gpt-4o
llama_model_loader: - kv  26:                      tokenizer.ggml.tokens arr[str,201088]  = ["!", "\"", "#", "$", "%", "&", "'", ...
llama_model_loader: - kv  27:                  tokenizer.ggml.token_type arr[i32,201088]  = [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, ...
llama_model_loader: - kv  28:                      tokenizer.ggml.merges arr[str,446189]  = ["Ġ Ġ", "Ġ ĠĠĠ", "ĠĠ ĠĠ", "...
llama_model_loader: - kv  29:                tokenizer.ggml.bos_token_id u32              = 199998
llama_model_loader: - kv  30:                tokenizer.ggml.eos_token_id u32              = 200002
llama_model_loader: - kv  31:            tokenizer.ggml.padding_token_id u32              = 199999
llama_model_loader: - kv  32:                    tokenizer.chat_template str              = {#-\n  In addition to the normal input...
llama_model_loader: - kv  33:               general.quantization_version u32              = 2
llama_model_loader: - kv  34:                          general.file_type u32              = 38
llama_model_loader: - type  f32:  289 tensors
llama_model_loader: - type q8_0:   98 tensors
llama_model_loader: - type mxfp4:   72 tensors
init_tokenizer: initializing tokenizer for type 2
load: printing all EOG tokens:
load:   - 199999 ('<|endoftext|>')
load:   - 200002 ('<|return|>')
load:   - 200007 ('<|end|>')
load:   - 200012 ('<|call|>')
load: special_eog_ids contains both '<|return|>' and '<|call|>' tokens, removing '<|end|>' token from EOG list
load: special tokens cache size = 21
load: token to piece cache size = 1.3332 MB
llm_load_print_meta: format           = GGUF V3 (latest)
llm_load_print_meta: arch             = gpt-oss
llm_load_print_meta: n_ctx_train      = 131072
llm_load_print_meta: n_embd           = 2880
llm_load_print_meta: n_layer          = 24
llm_load_print_meta: n_head           = 64
llm_load_print_meta: n_head_kv        = 8
llm_load_print_meta: n_rot            = 64
llm_load_print_meta: n_swa            = 128
llm_load_print_meta: n_swa_pattern    = 1
llm_load_print_meta: n_embd_head_k    = 64
llm_load_print_meta: n_embd_head_v    = 64
llm_load_print_meta: n_gqa            = 8
llm_load_print_meta: n_embd_k_gqa     = 512
llm_load_print_meta: n_embd_v_gqa     = 512
llm_load_print_meta: f_norm_eps       = 0.0e+00
llm_load_print_meta: f_norm_rms_eps   = 1.0e-05
llm_load_print_meta: f_clamp_kqv      = 0.0e+00
llm_load_print_meta: f_max_alibi_bias = 0.0e+00
llm_load_print_meta: f_logit_scale    = 0.0e+00
llm_load_print_meta: n_ff             = 2880
llm_load_print_meta: n_expert         = 32
llm_load_print_meta: n_expert_used    = 4
llm_load_print_meta: causal attn      = 1
llm_load_print_meta: pooling type     = 0
llm_load_print_meta: rope type        = 2
llm_load_print_meta: rope scaling     = yarn
llm_load_print_meta: freq_base_train  = 150000.0
llm_load_print_meta: freq_scale_train = 0.03125
llm_load_print_meta: n_ctx_orig_yarn  = 4096
llm_load_print_meta: rope_finetuned   = unknown
llm_load_print_meta: ssm_d_conv       = 0
llm_load_print_meta: ssm_d_inner      = 0
llm_load_print_meta: ssm_d_state      = 0
llm_load_print_meta: ssm_dt_rank      = 0
llm_load_print_meta: model type       = ?B
llm_load_print_meta: model ftype      = MXFP4 - 4.25 bpw
llm_load_print_meta: model params     = 20.915 B
llm_load_print_meta: model size       = 11.266 GiB (4.627 BPW) 
llm_load_print_meta: repeating layers = 10.120 GiB (4.400 BPW, 19.756 B parameters)
llm_load_print_meta: general.name     = Gpt Oss 20b
llm_load_print_meta: n_ff_exp         = 2880
print_info: vocab type       = BPE
print_info: n_vocab          = 201088
print_info: n_merges         = 446189
print_info: BOS token        = 199998 '<|startoftext|>'
print_info: EOS token        = 200002 '<|return|>'
print_info: EOT token        = 199999 '<|endoftext|>'
print_info: PAD token        = 199999 '<|endoftext|>'
print_info: LF token         = 198 'Ċ'
print_info: EOG token        = 199999 '<|endoftext|>'
print_info: EOG token        = 200002 '<|return|>'
print_info: EOG token        = 200012 '<|call|>'
print_info: max token length = 256
ggml_cuda_init: GGML_CUDA_FORCE_MMQ:    no
ggml_cuda_init: GGML_CUDA_FORCE_CUBLAS: no
ggml_cuda_init: found 1 CUDA devices:
  Device 0: NVIDIA GeForce RTX 3070, compute capability 8.6, VMM: yes
llm_load_tensors: ggml ctx size =    0.37 MiB
Tensor blk.0.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.0.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.0.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.0.ffn_gate_exps.bias buffer type overriden to CPU
Tensor blk.0.ffn_down_exps.bias buffer type overriden to CPU
Tensor blk.0.ffn_up_exps.bias buffer type overriden to CPU
Tensor blk.1.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.1.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.1.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.1.ffn_gate_exps.bias buffer type overriden to CPU
Tensor blk.1.ffn_down_exps.bias buffer type overriden to CPU
Tensor blk.1.ffn_up_exps.bias buffer type overriden to CPU
Tensor blk.2.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.2.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.2.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.2.ffn_gate_exps.bias buffer type overriden to CPU
Tensor blk.2.ffn_down_exps.bias buffer type overriden to CPU
Tensor blk.2.ffn_up_exps.bias buffer type overriden to CPU
Tensor blk.3.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.3.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.3.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.3.ffn_gate_exps.bias buffer type overriden to CPU
Tensor blk.3.ffn_down_exps.bias buffer type overriden to CPU
Tensor blk.3.ffn_up_exps.bias buffer type overriden to CPU
Tensor blk.4.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.4.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.4.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.4.ffn_gate_exps.bias buffer type overriden to CPU
Tensor blk.4.ffn_down_exps.bias buffer type overriden to CPU
Tensor blk.4.ffn_up_exps.bias buffer type overriden to CPU
Tensor blk.5.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.5.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.5.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.5.ffn_gate_exps.bias buffer type overriden to CPU
Tensor blk.5.ffn_down_exps.bias buffer type overriden to CPU
Tensor blk.5.ffn_up_exps.bias buffer type overriden to CPU
Tensor blk.6.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.6.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.6.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.6.ffn_gate_exps.bias buffer type overriden to CPU
Tensor blk.6.ffn_down_exps.bias buffer type overriden to CPU
Tensor blk.6.ffn_up_exps.bias buffer type overriden to CPU
Tensor blk.7.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.7.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.7.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.7.ffn_gate_exps.bias buffer type overriden to CPU
Tensor blk.7.ffn_down_exps.bias buffer type overriden to CPU
Tensor blk.7.ffn_up_exps.bias buffer type overriden to CPU
Tensor blk.8.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.8.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.8.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.8.ffn_gate_exps.bias buffer type overriden to CPU
Tensor blk.8.ffn_down_exps.bias buffer type overriden to CPU
Tensor blk.8.ffn_up_exps.bias buffer type overriden to CPU
Tensor blk.9.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.9.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.9.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.9.ffn_gate_exps.bias buffer type overriden to CPU
Tensor blk.9.ffn_down_exps.bias buffer type overriden to CPU
Tensor blk.9.ffn_up_exps.bias buffer type overriden to CPU
Tensor blk.10.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.10.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.10.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.10.ffn_gate_exps.bias buffer type overriden to CPU
Tensor blk.10.ffn_down_exps.bias buffer type overriden to CPU
Tensor blk.10.ffn_up_exps.bias buffer type overriden to CPU
Tensor blk.11.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.11.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.11.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.11.ffn_gate_exps.bias buffer type overriden to CPU
Tensor blk.11.ffn_down_exps.bias buffer type overriden to CPU
Tensor blk.11.ffn_up_exps.bias buffer type overriden to CPU
Tensor blk.12.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.12.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.12.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.12.ffn_gate_exps.bias buffer type overriden to CPU
Tensor blk.12.ffn_down_exps.bias buffer type overriden to CPU
Tensor blk.12.ffn_up_exps.bias buffer type overriden to CPU
Tensor blk.13.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.13.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.13.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.13.ffn_gate_exps.bias buffer type overriden to CPU
Tensor blk.13.ffn_down_exps.bias buffer type overriden to CPU
Tensor blk.13.ffn_up_exps.bias buffer type overriden to CPU
Tensor blk.14.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.14.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.14.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.14.ffn_gate_exps.bias buffer type overriden to CPU
Tensor blk.14.ffn_down_exps.bias buffer type overriden to CPU
Tensor blk.14.ffn_up_exps.bias buffer type overriden to CPU
Tensor blk.15.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.15.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.15.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.15.ffn_gate_exps.bias buffer type overriden to CPU
Tensor blk.15.ffn_down_exps.bias buffer type overriden to CPU
Tensor blk.15.ffn_up_exps.bias buffer type overriden to CPU
Tensor blk.16.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.16.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.16.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.16.ffn_gate_exps.bias buffer type overriden to CPU
Tensor blk.16.ffn_down_exps.bias buffer type overriden to CPU
Tensor blk.16.ffn_up_exps.bias buffer type overriden to CPU
Tensor blk.17.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.17.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.17.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.17.ffn_gate_exps.bias buffer type overriden to CPU
Tensor blk.17.ffn_down_exps.bias buffer type overriden to CPU
Tensor blk.17.ffn_up_exps.bias buffer type overriden to CPU
Tensor blk.18.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.18.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.18.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.18.ffn_gate_exps.bias buffer type overriden to CPU
Tensor blk.18.ffn_down_exps.bias buffer type overriden to CPU
Tensor blk.18.ffn_up_exps.bias buffer type overriden to CPU
Tensor blk.19.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.19.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.19.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.19.ffn_gate_exps.bias buffer type overriden to CPU
Tensor blk.19.ffn_down_exps.bias buffer type overriden to CPU
Tensor blk.19.ffn_up_exps.bias buffer type overriden to CPU
Tensor blk.20.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.20.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.20.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.20.ffn_gate_exps.bias buffer type overriden to CPU
Tensor blk.20.ffn_down_exps.bias buffer type overriden to CPU
Tensor blk.20.ffn_up_exps.bias buffer type overriden to CPU
Tensor blk.21.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.21.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.21.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.21.ffn_gate_exps.bias buffer type overriden to CPU
Tensor blk.21.ffn_down_exps.bias buffer type overriden to CPU
Tensor blk.21.ffn_up_exps.bias buffer type overriden to CPU
Tensor blk.22.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.22.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.22.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.22.ffn_gate_exps.bias buffer type overriden to CPU
Tensor blk.22.ffn_down_exps.bias buffer type overriden to CPU
Tensor blk.22.ffn_up_exps.bias buffer type overriden to CPU
Tensor blk.23.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.23.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.23.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.23.ffn_gate_exps.bias buffer type overriden to CPU
Tensor blk.23.ffn_down_exps.bias buffer type overriden to CPU
Tensor blk.23.ffn_up_exps.bias buffer type overriden to CPU
llm_load_tensors: offloading 24 repeating layers to GPU
llm_load_tensors: offloading non-repeating layers to GPU
llm_load_tensors: offloaded 25/25 layers to GPU
llm_load_tensors:        CPU buffer size = 10335.57 MiB
llm_load_tensors:        CPU buffer size =   586.82 MiB
llm_load_tensors:      CUDA0 buffer size =  1242.03 MiB
................................................................................
llama_new_context_with_model: n_ctx      = 131072
llama_new_context_with_model: n_batch    = 2048
llama_new_context_with_model: n_ubatch   = 512
llama_new_context_with_model: flash_attn = 0
llama_new_context_with_model: mla_attn   = 0
llama_new_context_with_model: attn_max_b = 0
llama_new_context_with_model: fused_moe  = 0
llama_new_context_with_model: fused_up_gate = 1
llama_new_context_with_model: ser        = -1, 0
llama_new_context_with_model: freq_base  = 150000.0
llama_new_context_with_model: freq_scale = 0.03125
llama_kv_cache_init:      CUDA0 KV buffer size =  6144.00 MiB
llama_new_context_with_model: KV self size  = 6144.00 MiB, K (f16): 3072.00 MiB, V (f16): 3072.00 MiB
llama_new_context_with_model:  CUDA_Host  output buffer size =     1.53 MiB
ggml_backend_cuda_buffer_type_alloc_buffer: allocating 16929.88 MiB on device 0: cudaMalloc failed: out of memory
ggml_gallocr_reserve_n: failed to allocate CUDA0 buffer of size 17752270848
llama_new_context_with_model: failed to allocate compute buffers
llama_init_from_gpt_params: error: failed to create context with model 'gpt-oss-20b-mxfp4.gguf'
 ERR [              load_model] unable to load model | tid="139668388159488" timestamp=1760105535 model="gpt-oss-20b-mxfp4.gguf"
Segmentation fault (core dumped)
```

Along with that, when loading with 65536 ctx, ik_llama.cpp uses 5GB, llama.cpp uses 3.3GB of VRAM. It seems like ik_llama is overallocating for what is actually necessary? Perhaps something to do with SWA?

Both llama.cpp and ik_llama.cpp were built with identical cmake commands:

```
CUDACXX=/usr/local/cuda/bin/nvcc cmake -S . -B build-cuda -G Ninja   -DGGML_CUDA=ON   -DCMAKE_CUDA_ARCHITECTURES=86   -DGGML_BLAS=ON -DGGML_BLAS_VENDOR=OpenBLAS   -DGGML_CCACHE=OFF   -DCMAKE_CUDA_HOST_COMPILER=/usr/bin/g++-13   -DCMAKE_BUILD_TYPE=Release

cmake --build build-cuda -j
```





### Name and Version

user@DESKTOP-3P251OG:~/128gb/models$ ~/ik-llamacpp/ik_llama.cpp/build-cuda/bin/llama-cli --version
version: 3902 (f649e36a)
built with cc (GCC) 14.3.1 20250808 (Red Hat 14.3.1-3) for x86_64-redhat-linux

user@DESKTOP-3P251OG:~/128gb/models$ ~/llamacpp/llama.cpp/build-cuda/bin/llama-cli --version
ggml_cuda_init: GGML_CUDA_FORCE_MMQ:    no
ggml_cuda_init: GGML_CUDA_FORCE_CUBLAS: no
ggml_cuda_init: found 1 CUDA devices:
  Device 0: NVIDIA GeForce RTX 3070, compute capability 8.6, VMM: yes
version: 6699 (c08002a1)
built with cc (GCC) 14.3.1 20250808 (Red Hat 14.3.1-3) for x86_64-redhat-linux

### What operating system are you seeing the problem on?

Linux

### Relevant log output

```shell

```

---

## 💬 Conversation

👤 **ikawrakow** commented on **2025-10-10** at **15:50:43**

It is `-fa 1`, not `-fa on`.

Also add `-fmoe -ooae` to your command.

---

👤 **crat0z** commented on **2025-10-10** at **16:15:04**

Thanks for the quick reply.

I copied the wrong command above on accident, my apologies. I've been scratching my head for a few hours wondering why it isn't working, so I've been trying a bunch of different commands to get it to work similar to llama.cpp.

Running this command exactly still OOMs:

```
llama-server -m gpt-oss-20b-mxfp4.gguf -ngl 99 --n-cpu-moe 99 -fa -fmoe -ooae -c 131072 --reasoning-format auto --jinja
```

`-fa 1` doesn't work on my build, but `-fa` alone works. According to the log, `-fa` does enable flash attention:

```
llama_new_context_with_model: n_ctx      = 131072
llama_new_context_with_model: n_batch    = 2048
llama_new_context_with_model: n_ubatch   = 512
llama_new_context_with_model: flash_attn = 1
llama_new_context_with_model: mla_attn   = 0
llama_new_context_with_model: attn_max_b = 0
llama_new_context_with_model: fused_moe  = 1
llama_new_context_with_model: fused_up_gate = 1
llama_new_context_with_model: ser        = -1, 0
llama_new_context_with_model: freq_base  = 150000.0
llama_new_context_with_model: freq_scale = 0.03125
llama_kv_cache_init:      CUDA0 KV buffer size =  6144.00 MiB
llama_new_context_with_model: KV self size  = 6144.00 MiB, K (f16): 3072.00 MiB, V (f16): 3072.00 MiB
llama_new_context_with_model:  CUDA_Host  output buffer size =     1.53 MiB
ggml_backend_cuda_buffer_type_alloc_buffer: allocating 682.20 MiB on device 0: cudaMalloc failed: out of memory
ggml_gallocr_reserve_n: failed to allocate CUDA0 buffer of size 715335936
llama_new_context_with_model: failed to allocate compute buffers
llama_init_from_gpt_params: error: failed to create context with model 'gpt-oss-20b-mxfp4.gguf'
 ERR [              load_model] unable to load model | tid="139992119197696" timestamp=1760112601 model="gpt-oss-20b-mxfp4.gguf"
Segmentation fault (core dumped)
```

The same command with `-c 65536` exhibits the same behavior described in my original comment: 5GB VRAM usage compared to 3.3GB with llama.cpp. Are there any other options I'm missing? I built from main branch today.

---

👤 **magikRUKKOLA** commented on **2025-10-10** at **19:38:47**

@crat0z 

Is it the one you're using?

https://huggingface.co/bartowski/openai_gpt-oss-20b-GGUF-MXFP4-Experimental/resolve/main/openai_gpt-oss-20b-MXFP4.gguf?download=true

---

👤 **crat0z** commented on **2025-10-10** at **19:41:29**

Nope, this one:

https://huggingface.co/ggml-org/gpt-oss-20b-GGUF/blob/main/gpt-oss-20b-mxfp4.gguf

---

👤 **ikawrakow** commented on **2025-10-11** at **06:08:46**

OK, GPT-OSS uses SWA with a very small window (128 tokens). By default `llama.cpp` keeps only the KV entries necessary to compute the current prompt, which leads to a nearly 2X reduction in KV size. This has the advantage of being able to process longer prompts, but has the disadvantage of not being able to reuse the KV cache. For KV cache reuse you need `--swa-full` in `llama.cpp`, which corresponds to what `ik_llama.cpp` does, or you need to work with SWA checkpoints, which I personally find inconvenient. Hence, this functionality is not implemented in `ik_llama.cpp`. If the 128k context window is necessary for your use case then it is better to use mainline. If you want to compare, then reduce the maximum context length in `ik_llama.cpp` to a point where it does not OOM.  I'm not sure why you still get OOM at 64k tokens as the VRAM usage I see on my system is about 5 GiB, so it should fit in a 3070. Do you have a `llama.cpp` server running at the same time by any chance?

---

👤 **crat0z** commented on **2025-10-11** at **10:36:30**

Thanks again for looking into the issue

> This has the advantage of being able to process longer prompts, but has the disadvantage of not being able to reuse the KV cache

KV cache reuse works fine on mainline, but it needs `--cache-reuse N` IIRC

> I'm not sure why you still get OOM at 64k tokens as the VRAM usage I see on my system is about 5 GiB, so it should fit in a 3070.

`-c 65536` loads fine on ik-llama, I was simply pointing out that ik-llama uses ~ 5GiB compared to ~ 3.3 GiB on mainline

---

I was looking around online because I found it interesting that ik-llama uses approximately the same KV cache size at 65536 context, as mainline uses at 131072 context (the 5 GiB discussed above). What I found is that gpt-oss alternates attention layers:

"The models use alternating dense and locally banded sparse attention patterns, similar to GPT‑3" - https://openai.com/index/introducing-gpt-oss/

So from my understanding, one layer uses SWA, the next layer uses dense attention, and the next layer uses SWA, and so on. Could this be the cause of the 2x KV cache size compared to mainline?

---

👤 **ikawrakow** commented on **2025-10-11** at **11:08:37**

Yes, this is what I tried to explain above. For every second layer `llama.cpp` keeps only 128 KV cache entries, which is negligible compared to a context length of 128k, so KV cache is (nearly) 2X smaller. If `N` checkpoints (with `N` much smaller than 1000) satisfy your cache reuse requirements, then what `llama.cpp` does is better for you. But in general, being able to restart from just `N` positions is a limitation. With `--swa-full` in `llama.cpp` you will get the same KV cache size as in `ik_llama.cpp`.

---

👤 **ikawrakow** commented on **2025-10-16** at **12:02:59**

After thinking some more about this, I don't think I want to get involved with adding the `llama.cpp` approach to SWA, requiring check pointing, and all that.