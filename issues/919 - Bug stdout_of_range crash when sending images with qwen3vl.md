## 📌 [Issue #919](https://github.com/ikawrakow/ik_llama.cpp/issues/919) - Bug: 'std::out_of_range' crash when sending images with qwen3vl

| **Author** | `MrHills-2` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-11-07 |
| **Updated** | 2025-11-09 |

---

## 📄 Description

### What happened?

llama-server crashes whenever an image is sent via the API. Text works fine. This happens both with qwen3vl 235B and qwen3vl 30B. Both with f16 mmproj and q8 mmproj. Both using sillytavern chat completion and the integrated llama.cpp webui.

I tried multiple versions of ik_llama.cpp.
Using: 
build/bin/llama-server -m models/Qwen3-VL-30B-A3B-Instruct-UD-IQ3_XXS.gguf --mmproj models/Qwen3-30B-mmproj-F16.gguf -ot "blk\.(?:[0-9]|[1-2][0-9])\.ffn.*=CPU" -c 24576 -b 8192 -ub 2048 -v -ctk q8_0 -ctv q8_0 --threads 7 -ngl 95 -sp --host 0.0.0.0 --port 8080

cmake -B ./build -DGGML_CUDA=ON -DCMAKE_CUDA_ARCHITECTURES="120" GGML_CUDA_FA_ALL_QUANTS=ON

cmake --build ./build --config Release -j 16

Idk what I'm doing wrong. I edited out the tensors loading messages from the output for brevity.

### Name and Version

./llama-server --version
version: 3964 (d62e8c51)
built with cc (GCC) 15.2.1 20250813 for x86_64-pc-linux-gnu

Commit d62e8c5

### What operating system are you seeing the problem on?

Linux

### Relevant log output

```shell
ggml_cuda_init: GGML_CUDA_FORCE_MMQ:    no
ggml_cuda_init: GGML_CUDA_FORCE_CUBLAS: no
ggml_cuda_init: found 1 CUDA devices:
  Device 0: NVIDIA GeForce RTX 5090, compute capability 12.0, VMM: yes
INFO [                    main] build info | tid="140019725488128" timestamp=1762539575 build=3964 commit="d62e8c51"
INFO [                    main] system info | tid="140019725488128" timestamp=1762539575 n_threads=7 n_threads_batch=-1 total_threads=16 system_info="AVX = 1 | AVX_VNNI = 0 | AVX2 = 1 | AVX512 = 1 | AVX512_VBMI = 1 | AVX512_VNNI = 1 | AVX512_BF16 = 1 | FMA = 1 | NEON = 0 | SVE = 0 | ARM_FMA = 0 | F16C = 1 | FP16_VA = 0 | WASM_SIMD = 0 | BLAS = 1 | SSE3 = 1 | SSSE3 = 1 | VSX = 0 | MATMUL_INT8 = 0 | LLAMAFILE = 1 | "
CUDA0: using device CUDA0 - 31085 MiB free
llama_model_loader: loaded meta data with 45 key-value pairs and 579 tensors from models/Qwen3-VL-30B-A3B-Instruct-UD-IQ3_XXS.gguf (version GGUF V3 (latest))
llama_model_loader: Dumping metadata keys/values. Note: KV overrides do not apply in this output.
llama_model_loader: - kv   0:                       general.architecture str              = qwen3vlmoe
llama_model_loader: - kv   1:                               general.type str              = model
llama_model_loader: - kv   2:                               general.name str              = Qwen3-Vl-30B-A3B-Instruct
llama_model_loader: - kv   3:                           general.finetune str              = Instruct
llama_model_loader: - kv   4:                           general.basename str              = Qwen3-Vl-30B-A3B-Instruct
llama_model_loader: - kv   5:                       general.quantized_by str              = Unsloth
llama_model_loader: - kv   6:                         general.size_label str              = 30B-A3B
llama_model_loader: - kv   7:                            general.license str              = apache-2.0
llama_model_loader: - kv   8:                           general.repo_url str              = https://huggingface.co/unsloth
llama_model_loader: - kv   9:                   general.base_model.count u32              = 1
llama_model_loader: - kv  10:                  general.base_model.0.name str              = Qwen3 VL 30B A3B Instruct
llama_model_loader: - kv  11:          general.base_model.0.organization str              = Qwen
llama_model_loader: - kv  12:              general.base_model.0.repo_url str              = https://huggingface.co/Qwen/Qwen3-VL-...
llama_model_loader: - kv  13:                               general.tags arr[str,2]       = ["unsloth", "image-text-to-text"]
llama_model_loader: - kv  14:                     qwen3vlmoe.block_count u32              = 48
llama_model_loader: - kv  15:                  qwen3vlmoe.context_length u32              = 262144
llama_model_loader: - kv  16:                qwen3vlmoe.embedding_length u32              = 2048
llama_model_loader: - kv  17:             qwen3vlmoe.feed_forward_length u32              = 6144
llama_model_loader: - kv  18:            qwen3vlmoe.attention.head_count u32              = 32
llama_model_loader: - kv  19:         qwen3vlmoe.attention.head_count_kv u32              = 4
llama_model_loader: - kv  20:                  qwen3vlmoe.rope.freq_base f32              = 5000000.000000
llama_model_loader: - kv  21: qwen3vlmoe.attention.layer_norm_rms_epsilon f32              = 0.000001
llama_model_loader: - kv  22:               qwen3vlmoe.expert_used_count u32              = 8
llama_model_loader: - kv  23:            qwen3vlmoe.attention.key_length u32              = 128
llama_model_loader: - kv  24:          qwen3vlmoe.attention.value_length u32              = 128
llama_model_loader: - kv  25:                    qwen3vlmoe.expert_count u32              = 128
llama_model_loader: - kv  26:      qwen3vlmoe.expert_feed_forward_length u32              = 768
llama_model_loader: - kv  27:         qwen3vlmoe.rope.dimension_sections arr[i32,4]       = [24, 20, 20, 0]
llama_model_loader: - kv  28:              qwen3vlmoe.n_deepstack_layers u32              = 3
llama_model_loader: - kv  29:                       tokenizer.ggml.model str              = gpt2
llama_model_loader: - kv  30:                         tokenizer.ggml.pre str              = qwen2
llama_model_loader: - kv  31:                      tokenizer.ggml.tokens arr[str,151936]  = ["!", "\"", "#", "$", "%", "&", "'", ...
llama_model_loader: - kv  32:                  tokenizer.ggml.token_type arr[i32,151936]  = [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, ...
llama_model_loader: - kv  33:                      tokenizer.ggml.merges arr[str,151387]  = ["Ġ Ġ", "ĠĠ ĠĠ", "i n", "Ġ t",...
llama_model_loader: - kv  34:                tokenizer.ggml.eos_token_id u32              = 151645
llama_model_loader: - kv  35:            tokenizer.ggml.padding_token_id u32              = 151654
llama_model_loader: - kv  36:                tokenizer.ggml.bos_token_id u32              = 151643
llama_model_loader: - kv  37:               tokenizer.ggml.add_bos_token bool             = false
llama_model_loader: - kv  38:                    tokenizer.chat_template str              = {%- if tools %}\n    {{- '<|im_start|>...
llama_model_loader: - kv  39:               general.quantization_version u32              = 2
llama_model_loader: - kv  40:                          general.file_type u32              = 23
llama_model_loader: - kv  41:                      quantize.imatrix.file str              = Qwen3-VL-30B-A3B-Instruct-GGUF/imatri...
llama_model_loader: - kv  42:                   quantize.imatrix.dataset str              = unsloth_calibration_Qwen3-VL-30B-A3B-...
llama_model_loader: - kv  43:             quantize.imatrix.entries_count u32              = 384
llama_model_loader: - kv  44:              quantize.imatrix.chunks_count u32              = 694
llama_model_loader: - type  f32:  241 tensors
llama_model_loader: - type q4_K:    1 tensors
llama_model_loader: - type q5_K:   18 tensors
llama_model_loader: - type q6_K:   11 tensors
llama_model_loader: - type iq3_xxs:   89 tensors
llama_model_loader: - type iq3_s:   42 tensors
llama_model_loader: - type iq4_xs:  177 tensors
load: printing all EOG tokens:
load:   - 151643 ('<|endoftext|>')
load:   - 151645 ('<|im_end|>')
load:   - 151662 ('<|fim_pad|>')
load:   - 151663 ('<|repo_name|>')
load:   - 151664 ('<|file_sep|>')
load: special tokens cache size = 26
load: token to piece cache size = 0.9311 MB
llm_load_print_meta: format           = GGUF V3 (latest)
llm_load_print_meta: arch             = qwen3vlmoe
llm_load_print_meta: n_ctx_train      = 262144
llm_load_print_meta: n_embd           = 8192
llm_load_print_meta: n_layer          = 48
llm_load_print_meta: n_head           = 32
llm_load_print_meta: n_head_kv        = 4
llm_load_print_meta: n_rot            = 128
llm_load_print_meta: n_swa            = 0
llm_load_print_meta: n_swa_pattern    = 1
llm_load_print_meta: n_embd_head_k    = 128
llm_load_print_meta: n_embd_head_v    = 128
llm_load_print_meta: n_gqa            = 8
llm_load_print_meta: n_embd_k_gqa     = 512
llm_load_print_meta: n_embd_v_gqa     = 512
llm_load_print_meta: f_norm_eps       = 0.0e+00
llm_load_print_meta: f_norm_rms_eps   = 1.0e-06
llm_load_print_meta: f_clamp_kqv      = 0.0e+00
llm_load_print_meta: f_max_alibi_bias = 0.0e+00
llm_load_print_meta: f_logit_scale    = 0.0e+00                                                                                                                                                                      llm_load_print_meta: n_ff             = 6144                                                                                                                                                                         llm_load_print_meta: n_expert         = 128                                                                                                                                                                          llm_load_print_meta: n_expert_used    = 8                                                                                                                                                                            llm_load_print_meta: causal attn      = 1                                                                                                                                                                            llm_load_print_meta: pooling type     = 0                                                                                                                                                                            llm_load_print_meta: rope type        = 40                                                                                                                                                                           llm_load_print_meta: rope scaling     = linear                                                                                                                                                                       llm_load_print_meta: freq_base_train  = 5000000.0                                                                                                                                                                    llm_load_print_meta: freq_scale_train = 1                                                                                                                                                                            llm_load_print_meta: n_ctx_orig_yarn  = 262144                                                                                                                                                                       llm_load_print_meta: rope_finetuned   = unknown                                                                                                                                                                      llm_load_print_meta: mrope sections   = [24, 20, 20, 0]                                                                                                                                                              llm_load_print_meta: ssm_d_conv       = 0                                                                                                                                                                            llm_load_print_meta: ssm_d_inner      = 0                                                                                                                                                                            llm_load_print_meta: ssm_d_state      = 0                                                                                                                                                                            llm_load_print_meta: ssm_dt_rank      = 0                                                                                                                                                                            llm_load_print_meta: model type       = 30B.A3B                                                                                                                                                                      llm_load_print_meta: model ftype      = IQ3_XXS - 3.0625 bpw                                                                                                                                                         llm_load_print_meta: model params     = 30.532 B                                                                                                                                                                     llm_load_print_meta: model size       = 11.997 GiB (3.375 BPW)                                                                                                                                                       llm_load_print_meta: repeating layers = 11.597 GiB (3.331 BPW, 29.910 B parameters)                                                                                                                                  llm_load_print_meta: general.name     = Qwen3-Vl-30B-A3B-Instruct                                                                                                                                                    llm_load_print_meta: n_ff_exp         = 768                                                                                                                                                                          print_info: vocab type       = BPE                                                                                                                                                                                   print_info: n_vocab          = 151936                                                                                                                                                                                print_info: n_merges         = 151387
print_info: BOS token        = 151643 '<|endoftext|>'                                                                                                                                                                print_info: EOS token        = 151645 '<|im_end|>'
print_info: EOT token        = 151645 '<|im_end|>'
print_info: PAD token        = 151654 '<|vision_pad|>'
print_info: LF token         = 198 'Ċ'
print_info: FIM PRE token    = 151659 '<|fim_prefix|>'
print_info: FIM SUF token    = 151661 '<|fim_suffix|>'
print_info: FIM MID token    = 151660 '<|fim_middle|>'
print_info: FIM PAD token    = 151662 '<|fim_pad|>'
print_info: FIM REP token    = 151663 '<|repo_name|>'
print_info: FIM SEP token    = 151664 '<|file_sep|>'
print_info: EOG token        = 151643 '<|endoftext|>'
print_info: EOG token        = 151645 '<|im_end|>'
print_info: EOG token        = 151662 '<|fim_pad|>'
print_info: EOG token        = 151663 '<|repo_name|>'
print_info: EOG token        = 151664 '<|file_sep|>'
print_info: max token length = 256
llm_load_tensors: ggml ctx size =    0.51 MiB
Tensor blk.0.ffn_norm.weight buffer type overriden to CPU

-- more tensors --

Tensor blk.29.ffn_up_exps.weight buffer type overriden to CPU
llm_load_tensors: offloading 48 repeating layers to GPU
llm_load_tensors: offloading non-repeating layers to GPU
llm_load_tensors: offloaded 49/49 layers to GPU
llm_load_tensors:        CPU buffer size =  7181.49 MiB
llm_load_tensors:        CPU buffer size =   166.92 MiB
llm_load_tensors:      CUDA0 buffer size =  5231.69 MiB
...................................................................................................
llama_new_context_with_model: n_ctx         = 24576
llama_new_context_with_model: n_batch       = 8192
llama_new_context_with_model: n_ubatch      = 2048
llama_new_context_with_model: flash_attn    = 1
llama_new_context_with_model: mla_attn      = 0
llama_new_context_with_model: attn_max_b    = 0
llama_new_context_with_model: fused_moe     = 1
llama_new_context_with_model: grouped er    = 0
llama_new_context_with_model: fused_up_gate = 1
llama_new_context_with_model: fused_mmad    = 1
llama_new_context_with_model: rope_cache    = 0
llama_new_context_with_model: ser           = -1, 0
llama_new_context_with_model: freq_base     = 5000000.0
llama_new_context_with_model: freq_scale    = 1
llama_kv_cache_init:      CUDA0 KV buffer size =  1224.02 MiB
llama_new_context_with_model: KV self size  = 1224.00 MiB, K (q8_0):  612.00 MiB, V (q8_0):  612.00 MiB
llama_new_context_with_model:  CUDA_Host  output buffer size =     0.58 MiB
llama_new_context_with_model:      CUDA0 compute buffer size =  1219.00 MiB
llama_new_context_with_model:  CUDA_Host compute buffer size =   112.05 MiB
llama_new_context_with_model: graph nodes  = 1781
llama_new_context_with_model: graph splits = 122
XXXXXXXXXXXXXXXXXXXXX Setting only active experts offload
======================================= HAVE_FANCY_SIMD is defined
clip_model_loader: model name:   Qwen3-Vl-30B-A3B-Instruct
clip_model_loader: description:
clip_model_loader: GGUF version: 3
clip_model_loader: alignment:    32
clip_model_loader: n_tensors:    352
clip_model_loader: n_kv:         31

clip_model_loader: has vision encoder
clip_model_loader: tensor[0]: n_dims = 1, name = v.blk.0.attn_out.bias, tensor_size=4608, offset=0, shape:[1152, 1, 1, 1], type = f32

-- more clip tensors --

clip_model_loader: tensor[351]: n_dims = 2, name = v.position_embd.weight, tensor_size=10616832, offset=1072862144, shape:[1152, 2304, 1, 1], type = f32
clip_ctx: have 2 back-ends:
  0:  CPU
  1:  CUDA0
clip_ctx: CLIP using CUDA0 backend
load_hparams: projector:          qwen3vl_merger
load_hparams: n_embd:             1152
load_hparams: n_head:             16
load_hparams: n_ff:               4304
load_hparams: n_layer:            27
load_hparams: ffn_op:             gelu
load_hparams: projection_dim:     2048

--- vision hparams ---
load_hparams: image_size:         1024
load_hparams: patch_size:         16
load_hparams: has_llava_proj:     0
load_hparams: minicpmv_version:   0
load_hparams: proj_scale_factor:  0
load_hparams: n_wa_pattern:       0
load_hparams: spatial_merge_size: 2

load_hparams: model size:         1033.29 MiB
load_hparams: metadata size:      0.12 MiB
load_tensors: loaded 352 tensors from models/Qwen3-30B-mmproj-F16.gguf
alloc_compute_meta:      CUDA0 compute buffer size =     3.00 MiB
alloc_compute_meta:        CPU compute buffer size =     0.19 MiB
INFO [              load_model] loaded multimodal model, '%s'
 | ="models/Qwen3-30B-mmproj-F16.gguf"
WARN [              load_model] %s                                                                                                                                                                                    | ="ctx_shift is not supported by multimodal, it will be disabled"                                                                                                                                                  INFO [                    init] initializing slots | tid="140019725488128" timestamp=1762539577 n_slots=1
INFO [                    init] new slot | tid="140019725488128" timestamp=1762539577 id_slot=0 n_ctx_slot=24576                                                                                                     INFO [                    main] model loaded | tid="140019725488128" timestamp=1762539577
INFO [                    main] chat template | tid="140019725488128" timestamp=1762539577 chat_template="{%- if tools %}\n    {{- '<|im_start|>system\\n' }}\n    {%- if messages[0].role == 'system' %}\n        {%- if messages[0].content is string %}\n            {{- messages[0].content }}\n        {%- else %}\n            {%- for content in messages[0].content %}\n                {%- if 'text' in content %}\n                    {{- content.text }}\n                {%- endif %}\n            {%- endfor %}\n        {%- endif %}\n        {{- '\\n\\n' }}\n    {%- endif %}\n    {{- \"# Tools\\n\\nYou may call one or more functions to assist with the user query.\\n\\nYou are provided with function signatures within <tools></tools> XML tags:\\n<tools>\" }}\n    {%- for tool in tools %}\n        {{- \"\\n\" }}\n        {{- tool | tojson }}\n    {%- endfor %}\n    {{- \"\\n</tools>\\n\\nFor each function call, return a json object with function name and arguments within <tool_call></tool_call> XML tags:\\n<tool_call>\\n{\\\"name\\\": <function-name>, \\\"arguments\\\": <args-json-object>}\\n</tool_call><|im_end|>\\n\" }}\n{%- else %}\n    {%- if messages[0].role == 'system' %}\n        {{- '<|im_start|>system\\n' }}\n        {%- if messages[0].content is string %}\n            {{- messages[0].content }}\n        {%- else %}\n            {%- for content in messages[0].content %}\n                {%- if 'text' in content %}\n                    {{- content.text }}\n                {%- endif %}\n            {%- endfor %}\n        {%- endif %}\n        {{- '<|im_end|>\\n' }}\n    {%- endif %}\n{%- endif %}\n{%- set image_count = namespace(value=0) %}\n{%- set video_count = namespace(value=0) %}\n{%- for message in messages %}\n    {%- if message.role == \"user\" %}\n        {{- '<|im_start|>' + message.role + '\\n' }}\n        {%- if message.content is string %}\n            {{- message.content }}\n        {%- else %}\n            {%- for content in message.content %}\n                {%- if content.type == 'image' or 'image' in content or 'image_url' in content %}\n                    {%- set image_count.value = image_count.value + 1 %}\n                    {%- if add_vision_id %}Picture {{ image_count.value }}: {% endif -%}\n                    <|vision_start|><|image_pad|><|vision_end|>\n                {%- elif content.type == 'video' or 'video' in content %}\n                    {%- set video_count.value = video_count.value + 1 %}\n                    {%- if add_vision_id %}Video {{ video_count.value }}: {% endif -%}\n                    <|vision_start|><|video_pad|><|vision_end|>\n                {%- elif 'text' in content %}\n                    {{- content.text }}\n                {%- endif %}\n            {%- endfor %}\n        {%- endif %}\n        {{- '<|im_end|>\\n' }}\n    {%- elif message.role == \"assistant\" %}\n        {{- '<|im_start|>' + message.role + '\\n' }}\n        {%- if message.content is string %}\n            {{- message.content }}\n        {%- else %}\n            {%- for content_item in message.content %}\n                {%- if 'text' in content_item %}\n                    {{- content_item.text }}\n                {%- endif %}\n            {%- endfor %}\n        {%- endif %}\n        {%- if message.tool_calls %}\n            {%- for tool_call in message.tool_calls %}\n                {%- if (loop.first and message.content) or (not loop.first) %}\n                    {{- '\\n' }}\n                {%- endif %}\n                {%- if tool_call.function %}\n                    {%- set tool_call = tool_call.function %}\n                {%- endif %}\n                {{- '<tool_call>\\n{\"name\": \"' }}\n                {{- tool_call.name }}\n                {{- '\", \"arguments\": ' }}\n                {%- if tool_call.arguments is string %}\n                    {{- tool_call.arguments }}\n                {%- else %}\n                    {{- tool_call.arguments | tojson }}\n                {%- endif %}\n                {{- '}\\n</tool_call>' }}\n            {%- endfor %}\n        {%- endif %}\n        {{- '<|im_end|>\\n' }}\n    {%- elif message.role == \"tool\" %}\n        {%- if loop.first or (messages[loop.index0 - 1].role != \"tool\") %}\n            {{- '<|im_start|>user' }}\n        {%- endif %}\n        {{- '\\n<tool_response>\\n' }}\n        {%- if message.content is string %}\n            {{- message.content }}\n        {%- else %}\n            {%- for content in message.content %}\n                {%- if content.type == 'image' or 'image' in content or 'image_url' in content %}\n                    {%- set image_count.value = image_count.value + 1 %}\n                    {%- if add_vision_id %}Picture {{ image_count.value }}: {% endif -%}\n                    <|vision_start|><|image_pad|><|vision_end|>\n                {%- elif content.type == 'video' or 'video' in content %}\n                    {%- set video_count.value = video_count.value + 1 %}\n                    {%- if add_vision_id %}Video {{ video_count.value }}: {% endif -%}\n                    <|vision_start|><|video_pad|><|vision_end|>\n                {%- elif 'text' in content %}\n                    {{- content.text }}\n                {%- endif %}\n            {%- endfor %}\n        {%- endif %}\n        {{- '\\n</tool_response>' }}\n        {%- if loop.last or (messages[loop.index0 + 1].role != \"tool\") %}\n            {{- '<|im_end|>\\n' }}\n        {%- endif %}\n    {%- endif %}\n{%- endfor %}\n{%- if add_generation_prompt %}\n    {{- '<|im_start|>assistant\\n' }}\n{%- endif %}\n"
INFO [                    main] chat template | tid="140019725488128" timestamp=1762539577 chat_example="<|im_start|>system\nYou are a helpful assistant<|im_end|>\n<|im_start|>user\nHello<|im_end|>\n<|im_start|>assistant\nHi there<|im_end|>\n<|im_start|>user\nHow are you?<|im_end|>\n<|im_start|>assistant\n" built_in=true
INFO [                    main] HTTP server listening | tid="140019725488128" timestamp=1762539577 n_threads_http="15" port="8080" hostname="0.0.0.0"
VERB [              start_loop] new task may arrive | tid="140019725488128" timestamp=1762539577
VERB [              start_loop] update_multitasks | tid="140019725488128" timestamp=1762539577
VERB [              start_loop] callback_update_slots | tid="140019725488128" timestamp=1762539577
INFO [            update_slots] all slots are idle | tid="140019725488128" timestamp=1762539577
VERB [          kv_cache_clear] clearing KV cache | tid="140019725488128" timestamp=1762539577
VERB [              start_loop] wait for new task | tid="140019725488128" timestamp=1762539577                                                                                                                       add_text: <|im_start|>system
-- Start of the Chat --
<|im_end|>
<|im_start|>assistant
Greetings! I'm an AI model. How can I help you?<|im_end|>
<|im_start|>user
Hey can you see this image?

add_text: <|vision_start|>
image_tokens->nx = 20
image_tokens->ny = 16
batch_f32 size = 1
add_text: <|vision_end|>
add_text: <|im_end|>
<|im_start|>assistant

VERB [              get_new_id] new task id | tid="140009716600832" timestamp=1762539609 new_id=0
VERB [     add_waiting_task_id] waiting for task id | tid="140009716600832" timestamp=1762539609 id_task=0
VERB [              start_loop] new task may arrive | tid="140019725488128" timestamp=1762539609
VERB [              start_loop] callback_new_task | tid="140019725488128" timestamp=1762539609 id_task=0
VERB [      get_available_slot] selected slot by lru | tid="140019725488128" timestamp=1762539609 id_slot=0 t_last=-1
INFO [   launch_slot_with_task] slot is processing task | tid="140019725488128" timestamp=1762539609 id_slot=0 id_task=0
VERB [              start_loop] update_multitasks | tid="140019725488128" timestamp=1762539609
VERB [              start_loop] callback_update_slots | tid="140019725488128" timestamp=1762539609
VERB [            update_slots] posting NEXT_RESPONSE | tid="140019725488128" timestamp=1762539609
VERB [                    post] new task id | tid="140019725488128" timestamp=1762539609 new_id=1
VERB [            update_slots] tokenizing prompt | tid="140019725488128" timestamp=1762539609 id_slot=0 id_task=0
terminate called after throwing an instance of 'std::out_of_range'
  what():  vector::_M_range_check: __n (which is 18446744073709551615) >= this->size() (which is 151936)
./Qwen3-VL-30B-A3B-Instruct-UD-IQ3_XXS: line 5: 102005 Aborted                    (core dumped) build/bin/llama-server -m models/Qwen3-VL-30B-A3B-Instruct-UD-IQ3_XXS.gguf --mmproj models/Qwen3-30B-mmproj-F16.gguf -ot "blk\.(?:[0-9]|[1-2][0-9])\.ffn.*=CPU" -c 24576 -b 8192 -ub 2048 -v -ctk q8_0 -ctv q8_0 --threads 7 -ngl 95 -sp --host 0.0.0.0 --port 8080
```

---

## 💬 Conversation

👤 **Ph0rk0z** commented on **2025-11-07** at **22:58:35**

Hmm.. so you have 5090 and I have 3090s. I also don't have fancy SIMD.

---

👤 **firecoperana** commented on **2025-11-08** at **13:48:35**

Can you remove `-v` ?

---

👤 **MrHills-2** commented on **2025-11-08** at **16:09:48**

> Can you remove `-v` ?

Removing -v works. Thanks. Apparently vision doesn't like verbose outputs.

---

👤 **Ph0rk0z** commented on **2025-11-08** at **17:18:27**

That is weird because I was using -v with no issues.

---

👤 **MrHills-2** commented on **2025-11-08** at **17:24:55**

Well at any rate this doesn't work: 

build/bin/llama-server -m models/Qwen3-VL-30B-A3B-Instruct-UD-IQ3_XXS.gguf --mmproj models/Qwen3-30B-mmproj-F16.gguf -ot "blk\.(?:[0-9]|[1-2][0-9])\.ffn.*=CPU" -c 24576 -b 8192 -ub 2048 -v -ctk q8_0 -ctv q8_0 --threads 7 -ngl 95 -sp --host 0.0.0.0 --port 8080

This does: 

build/bin/llama-server -m models/Qwen3-VL-30B-A3B-Instruct-UD-IQ3_XXS.gguf --mmproj models/Qwen3-30B-mmproj-F16.gguf -ot "blk\.(?:[0-9]|[1-2][0-9])\.ffn.*=CPU" -c 24576 -b 8192 -ub 2048 -ctk q8_0 -ctv q8_0 --threads 7 -ngl 95 -sp --host 0.0.0.0 --port 8080

Maybe it's the combination of -sp and -v?

---

👤 **firecoperana** commented on **2025-11-08** at **22:17:52**

See if https://github.com/ikawrakow/ik_llama.cpp/pull/924 fixed it.

---

👤 **MrHills-2** commented on **2025-11-08** at **23:14:03**

> See if [[#924](https://github.com/ikawrakow/ik_llama.cpp/issues/924)](https://github.com/ikawrakow/ik_llama.cpp/pull/924) fixed it.

No, I get the same error while using

build/bin/llama-server -m models/Qwen3-VL-30B-A3B-Instruct-UD-IQ3_XXS.gguf --mmproj models/Qwen3-30B-mmproj-F16.gguf -ot "blk\.(?:[0-9]|[1-2][0-9])\.ffn.*=CPU" -c 24576 -b 8192 -ub 2048 -ctk q8_0 -ctv q8_0 --threads 7 -ngl 95 -sp --host 0.0.0.0 --port 8080 -v

It only works without -v

Using cb68182

---

👤 **firecoperana** commented on **2025-11-09** at **00:22:02**

pushed a new commit

---

👤 **MrHills-2** commented on **2025-11-09** at **11:06:09**

> pushed a new commit

Using commit d7385ac the -v flag no longer causes a crash.

This works:
build/bin/llama-server -m models/Qwen3-VL-30B-A3B-Instruct-UD-IQ3_XXS.gguf --mmproj models/Qwen3-30B-mmproj-F16.gguf -ot "blk.(?:[0-9]|[1-2][0-9]).ffn.*=CPU" -c 24576 -b 8192 -ub 2048 -ctk q8_0 -ctv q8_0 --threads 7 -ngl 95 -sp --host 0.0.0.0 --port 8080 -v

Thank you for your efforts.