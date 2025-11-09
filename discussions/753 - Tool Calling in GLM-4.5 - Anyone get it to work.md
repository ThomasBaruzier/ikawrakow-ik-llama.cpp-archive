## 🗣️ [Discussion #753](https://github.com/ikawrakow/ik_llama.cpp/discussions/753) - Tool Calling in GLM-4.5 - Anyone get it to work?

| **Author** | `jordandevai` |
| :--- | :--- |
| **State** | ✅ **Open** |
| **Created** | 2025-09-02 |
| **Updated** | 2025-10-08 |

---

## 📄 Description

Hey all - love the project and the speed of IK_LLama over Llama.cpp! However, spent the weekend trying to get tool calling to work to no avail with GLM-4.5-Air. 

It does things like this:

```text
Let me search for configuration files in the backend directory to help locate this information: <tool_call>ls <arg_key>dirPath</arg_key> <arg_value>backend</arg_value> </tool_call>
```

And then breaks Continue.dev. I'm running the latest build of IK_Llama:

```bash
./build/bin/llama-server \
  -m /data/models/GLM-4.5-Air/IQ4_KS/GLM-4.5-IQ4_KS-00001-of-00005.gguf\
  --host 0.0.0.0 --port 8080 \
  --threads 56 --threads-batch 56 \
  --numa distribute \
  --split-mode layer \
  --main-gpu 0 \
  --tensor-split 0.55,0.45 \
  --gpu-layers 9999 \
  --ctx-size 64000 \
  --batch-size 768 --ubatch-size 512 \
  --flash-attn \
  --jinja \
  --cache-type-k q8_0 --cache-type-v q8_0 \
  --override-tensor "blk\..*\.ffn_(up)_exps\.weight=CPU"\
  --override-tensor "blk\..*\.ffn_(up)_shexp\.weight=CPU"\
  --metrics --log-format json --timeout 1200
ggml_cuda_init: GGML_CUDA_FORCE_MMQ:    no
ggml_cuda_init: GGML_CUDA_FORCE_CUBLAS: no
ggml_cuda_init: found 2 CUDA devices:
  Device 0: NVIDIA GeForce RTX 5090, compute capability 12.0, VMM: yes
  Device 1: NVIDIA GeForce RTX 3090, compute capability 8.6, VMM: yes
{"tid":"125380885884928","timestamp":1756831162,"level":"INFO","function":"main","line":3629,"msg":"build info","build":3874,"commit":"3433c7b5"}
{"tid":"125380885884928","timestamp":1756831162,"level":"INFO","function":"main","line":3634,"msg":"system info","n_threads":56,"n_threads_batch":56,"total_threads":112,"system_info":"AVX = 1 | AVX_VNNI = 1 | AVX2 = 1 | AVX512 = 1 | AVX512_VBMI = 1 | AVX512_VNNI = 1 | AVX512_BF16 = 1 | FMA = 1 | NEON = 0 | SVE = 0 | ARM_FMA = 0 | F16C = 1 | FP16_VA = 0 | WASM_SIMD = 0 | BLAS = 1 | SSE3 = 1 | SSSE3 = 1 | VSX = 0 | MATMUL_INT8 = 0 | LLAMAFILE = 1 | "}
llama_model_loader: additional 4 GGUFs metadata loaded.
llama_model_loader: loaded meta data with 51 key-value pairs and 803 tensors from /data/models/GLM-4.5-Air/IQ4_KS/GLM-4.5-IQ4_KS-00001-of-00005.gguf (version GGUF V3 (latest))
llama_model_loader: Dumping metadata keys/values. Note: KV overrides do not apply in this output.
llama_model_loader: - kv   0:                       general.architecture str              = glm4moe
llama_model_loader: - kv   1:                               general.type str              = model
llama_model_loader: - kv   2:                               general.name str              = Glm-4.5-Air
llama_model_loader: - kv   3:                           general.basename str              = Glm-4.5-Air
llama_model_loader: - kv   4:                       general.quantized_by str              = Unsloth
llama_model_loader: - kv   5:                         general.size_label str              = 128x9.4B
llama_model_loader: - kv   6:                            general.license str              = mit
llama_model_loader: - kv   7:                           general.repo_url str              = https://huggingface.co/unsloth
llama_model_loader: - kv   8:                   general.base_model.count u32              = 1
llama_model_loader: - kv   9:                  general.base_model.0.name str              = GLM 4.5 Air
llama_model_loader: - kv  10:          general.base_model.0.organization str              = Zai Org
llama_model_loader: - kv  11:              general.base_model.0.repo_url str              = https://huggingface.co/zai-org/GLM-4....
llama_model_loader: - kv  12:                               general.tags arr[str,2]       = ["unsloth", "text-generation"]
llama_model_loader: - kv  13:                          general.languages arr[str,2]       = ["en", "zh"]
llama_model_loader: - kv  14:                        glm4moe.block_count u32              = 47
llama_model_loader: - kv  15:                     glm4moe.context_length u32              = 131072
llama_model_loader: - kv  16:                   glm4moe.embedding_length u32              = 4096
llama_model_loader: - kv  17:                glm4moe.feed_forward_length u32              = 10944
llama_model_loader: - kv  18:               glm4moe.attention.head_count u32              = 96
llama_model_loader: - kv  19:            glm4moe.attention.head_count_kv u32              = 8
llama_model_loader: - kv  20:                     glm4moe.rope.freq_base f32              = 1000000.000000
llama_model_loader: - kv  21:   glm4moe.attention.layer_norm_rms_epsilon f32              = 0.000010
llama_model_loader: - kv  22:                  glm4moe.expert_used_count u32              = 8
llama_model_loader: - kv  23:               glm4moe.attention.key_length u32              = 128
llama_model_loader: - kv  24:             glm4moe.attention.value_length u32              = 128
llama_model_loader: - kv  25:                          general.file_type u32              = 145
llama_model_loader: - kv  26:               glm4moe.rope.dimension_count u32              = 64
llama_model_loader: - kv  27:                       glm4moe.expert_count u32              = 128
llama_model_loader: - kv  28:         glm4moe.expert_feed_forward_length u32              = 1408
llama_model_loader: - kv  29:                glm4moe.expert_shared_count u32              = 1
llama_model_loader: - kv  30:          glm4moe.leading_dense_block_count u32              = 1
llama_model_loader: - kv  31:                 glm4moe.expert_gating_func u32              = 2
llama_model_loader: - kv  32:               glm4moe.expert_weights_scale f32              = 1.000000
llama_model_loader: - kv  33:                glm4moe.expert_weights_norm bool             = true
llama_model_loader: - kv  34:               glm4moe.nextn_predict_layers u32              = 1
llama_model_loader: - kv  35:               general.quantization_version u32              = 2
llama_model_loader: - kv  36:                       tokenizer.ggml.model str              = gpt2
llama_model_loader: - kv  37:                         tokenizer.ggml.pre str              = glm4
llama_model_loader: - kv  38:                      tokenizer.ggml.tokens arr[str,151552]  = ["!", "\"", "#", "$", "%", "&", "'", ...
llama_model_loader: - kv  39:                  tokenizer.ggml.token_type arr[i32,151552]  = [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, ...
llama_model_loader: - kv  40:                      tokenizer.ggml.merges arr[str,318088]  = ["Ġ Ġ", "Ġ ĠĠĠ", "ĠĠ ĠĠ", "...
llama_model_loader: - kv  41:                tokenizer.ggml.eos_token_id u32              = 151329
llama_model_loader: - kv  42:            tokenizer.ggml.padding_token_id u32              = 151330
llama_model_loader: - kv  43:                tokenizer.ggml.bos_token_id u32              = 151331
llama_model_loader: - kv  44:                tokenizer.ggml.eot_token_id u32              = 151336
llama_model_loader: - kv  45:            tokenizer.ggml.unknown_token_id u32              = 151329
llama_model_loader: - kv  46:                tokenizer.ggml.eom_token_id u32              = 151338
llama_model_loader: - kv  47:                    tokenizer.chat_template str              = [gMASK]<sop>\n{%- if tools -%}\n<|syste...
llama_model_loader: - kv  48:                                   split.no u16              = 0
llama_model_loader: - kv  49:                                split.count u16              = 5
llama_model_loader: - kv  50:                        split.tensors.count i32              = 803
llama_model_loader: - type  f32:  331 tensors
llama_model_loader: - type q8_0:   94 tensors
llama_model_loader: - type q5_K:   47 tensors
llama_model_loader: - type q6_K:    1 tensors
llama_model_loader: - type iq4_nl:   84 tensors
llama_model_loader: - type q6_0:    9 tensors
llama_model_loader: - type iq4_ks:  237 tensors
init_tokenizer: initializing tokenizer for type 2
load: special_eot_id is not in special_eog_ids - the tokenizer config may be incorrect
load: special_eom_id is not in special_eog_ids - the tokenizer config may be incorrect
load: printing all EOG tokens:
load:   - 151329 ('<|endoftext|>')
load:   - 151336 ('<|user|>')
load:   - 151338 ('<|observation|>')
load: special tokens cache size = 36
load: token to piece cache size = 0.9713 MB
llm_load_print_meta: format           = GGUF V3 (latest)
llm_load_print_meta: arch             = glm4moe
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
llm_load_print_meta: model ftype      = IQ4_KS - 4.25 bpw
llm_load_print_meta: model params     = 110.469 B
llm_load_print_meta: model size       = 57.211 GiB (4.449 BPW)
llm_load_print_meta: repeating layers = 56.429 GiB (4.438 BPW, 109.227 B parameters)
llm_load_print_meta: general.name     = Glm-4.5-Air
print_info: vocab type       = BPE
print_info: n_vocab          = 151552
print_info: n_merges         = 318088
print_info: BOS token        = 151331 '[gMASK]'
print_info: EOS token        = 151329 '<|endoftext|>'
print_info: EOT token        = 151336 '<|user|>'
print_info: EOM token        = 151338 '<|observation|>'
print_info: UNK token        = 151329 '<|endoftext|>'
print_info: PAD token        = 151330 '[MASK]'
print_info: LF token         = 198 'Ċ'
print_info: FIM PRE token    = 151347 '<|code_prefix|>'
print_info: FIM SUF token    = 151349 '<|code_suffix|>'
print_info: FIM MID token    = 151348 '<|code_middle|>'
print_info: EOG token        = 151329 '<|endoftext|>'
print_info: EOG token        = 151336 '<|user|>'
print_info: EOG token        = 151338 '<|observation|>'
print_info: max token length = 1024
llm_load_tensors: ggml ctx size =    0.99 MiB
Tensor blk.1.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.1.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.2.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.2.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.3.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.3.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.4.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.4.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.5.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.5.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.6.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.6.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.7.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.7.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.8.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.8.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.9.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.9.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.10.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.10.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.11.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.11.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.12.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.12.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.13.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.13.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.14.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.14.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.15.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.15.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.16.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.16.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.17.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.17.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.18.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.18.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.19.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.19.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.20.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.20.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.21.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.21.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.22.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.22.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.23.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.23.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.24.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.24.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.25.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.25.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.26.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.26.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.27.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.27.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.28.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.28.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.29.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.29.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.30.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.30.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.31.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.31.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.32.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.32.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.33.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.33.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.34.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.34.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.35.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.35.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.36.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.36.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.37.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.37.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.38.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.38.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.39.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.39.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.40.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.40.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.41.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.41.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.42.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.42.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.43.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.43.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.44.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.44.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.45.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.45.ffn_up_shexp.weight buffer type overriden to CPU
model has unused tensor blk.46.attn_norm.weight (size = 16384 bytes) -- ignoring
model has unused tensor blk.46.attn_q.weight (size = 26787840 bytes) -- ignoring
model has unused tensor blk.46.attn_k.weight (size = 4456448 bytes) -- ignoring
model has unused tensor blk.46.attn_v.weight (size = 4456448 bytes) -- ignoring
model has unused tensor blk.46.attn_q.bias (size = 49152 bytes) -- ignoring
model has unused tensor blk.46.attn_k.bias (size = 4096 bytes) -- ignoring
model has unused tensor blk.46.attn_v.bias (size = 4096 bytes) -- ignoring
model has unused tensor blk.46.attn_output.weight (size = 34603008 bytes) -- ignoring
model has unused tensor blk.46.post_attention_norm.weight (size = 16384 bytes) -- ignoring
model has unused tensor blk.46.ffn_gate_inp.weight (size = 2097152 bytes) -- ignoring
model has unused tensor blk.46.exp_probs_b.bias (size = 512 bytes) -- ignoring
model has unused tensor blk.46.ffn_gate_exps.weight (size = 392888320 bytes) -- ignoring
model has unused tensor blk.46.ffn_down_exps.weight (size = 415236096 bytes) -- ignoring
Tensor blk.46.ffn_up_exps.weight buffer type overriden to CPU
model has unused tensor blk.46.ffn_up_exps.weight (size = 392888320 bytes) -- ignoring
model has unused tensor blk.46.ffn_gate_shexp.weight (size = 3069440 bytes) -- ignoring
model has unused tensor blk.46.ffn_down_shexp.weight (size = 3244032 bytes) -- ignoring
Tensor blk.46.ffn_up_shexp.weight buffer type overriden to CPU
model has unused tensor blk.46.ffn_up_shexp.weight (size = 3069440 bytes) -- ignoring
model has unused tensor blk.46.nextn.eh_proj.weight (size = 17842176 bytes) -- ignoring
model has unused tensor blk.46.nextn.embed_tokens.weight (size = 330383360 bytes) -- ignoring
model has unused tensor blk.46.nextn.enorm.weight (size = 16384 bytes) -- ignoring
model has unused tensor blk.46.nextn.hnorm.weight (size = 16384 bytes) -- ignoring
model has unused tensor blk.46.nextn.shared_head_head.weight (size = 330383360 bytes) -- ignoring
model has unused tensor blk.46.nextn.shared_head_norm.weight (size = 16384 bytes) -- ignoring
llm_load_tensors: offloading 47 repeating layers to GPU
llm_load_tensors: offloading non-repeating layers to GPU
llm_load_tensors: offloaded 48/48 layers to GPU
llm_load_tensors:        CPU buffer size = 11930.25 MiB
llm_load_tensors:        CPU buffer size = 11396.75 MiB
llm_load_tensors:        CPU buffer size = 12620.20 MiB
llm_load_tensors:        CPU buffer size = 11396.75 MiB
llm_load_tensors:        CPU buffer size =  4541.63 MiB
llm_load_tensors:        CPU buffer size =   315.08 MiB
llm_load_tensors:      CUDA0 buffer size = 22848.77 MiB
llm_load_tensors:      CUDA1 buffer size = 16556.64 MiB
....................................................................................................
llama_new_context_with_model: n_ctx      = 64000
llama_new_context_with_model: n_batch    = 768
llama_new_context_with_model: n_ubatch   = 512
llama_new_context_with_model: flash_attn = 1
llama_new_context_with_model: mla_attn   = 0
llama_new_context_with_model: attn_max_b = 0
llama_new_context_with_model: fused_moe  = 0
llama_new_context_with_model: fused_up_gate = 1
llama_new_context_with_model: ser        = -1, 0
llama_new_context_with_model: freq_base  = 1000000.0
llama_new_context_with_model: freq_scale = 1
llama_kv_cache_init:      CUDA0 KV buffer size =  3585.95 MiB
llama_kv_cache_init:      CUDA1 KV buffer size =  2656.26 MiB
llama_new_context_with_model: KV self size  = 6242.19 MiB, K (q8_0): 3121.09 MiB, V (q8_0): 3121.09 MiB
llama_new_context_with_model:  CUDA_Host  output buffer size =     1.16 MiB
llama_new_context_with_model: pipeline parallelism enabled (n_copies=1)
llama_new_context_with_model:      CUDA0 compute buffer size =   626.19 MiB
llama_new_context_with_model:      CUDA1 compute buffer size =   304.00 MiB
llama_new_context_with_model:  CUDA_Host compute buffer size =   133.01 MiB
llama_new_context_with_model: graph nodes  = 2147
llama_new_context_with_model: graph splits = 131
{"tid":"125380885884928","timestamp":1756831169,"level":"INFO","function":"init","line":1288,"msg":"initializing slots","n_slots":1}
{"tid":"125380885884928","timestamp":1756831169,"level":"INFO","function":"init","line":1297,"msg":"new slot","id_slot":0,"n_ctx_slot":64000}
{"tid":"125380885884928","timestamp":1756831169,"level":"INFO","function":"main","line":3762,"msg":"model loaded"}
{"tid":"125380885884928","timestamp":1756831169,"level":"INFO","function":"main","line":3768,"msg":"chat template","chat_template":"[gMASK]<sop>\n{%- if tools -%}\n<|system|>\n# Tools\n\nYou may call one or more functions to assist with the user query.\n\nYou are provided with function signatures within <tools></tools> XML tags:\n<tools>\n{% for tool in tools %}\n{{ tool | tojson|string }}\n{% endfor %}\n</tools>\n\nFor each function call, output the function name and arguments within the following XML format:\n<tool_call>{function-name}\n<arg_key>{arg-key-1}</arg_key>\n<arg_value>{arg-value-1}</arg_value>\n<arg_key>{arg-key-2}</arg_key>\n<arg_value>{arg-value-2}</arg_value>\n...\n</tool_call>{%- endif -%}\n{%- macro visible_text(content) -%}\n    {%- if content is string -%}\n        {{- content }}\n    {%- elif content is iterable and content is not mapping -%}\n        {%- for item in content -%}\n            {%- if item is mapping and item.type == 'text' -%}\n                {{- item.text }}\n            {%- elif item is string -%}\n                {{- item }}\n            {%- endif -%}\n        {%- endfor -%}\n    {%- else -%}\n        {{- content }}\n    {%- endif -%}\n{%- endmacro -%}\n{%- set ns = namespace(last_user_index=-1) %}\n{%- for m in messages %}\n    {%- if m.role == 'user' %}\n        {% set ns.last_user_index = loop.index0 -%}\n    {%- endif %}\n{%- endfor %}\n{% for m in messages %}\n{%- if m.role == 'user' -%}<|user|>\n{% set content = visible_text(m.content) %}{{ content }}\n{{- '/nothink' if (enable_thinking is defined and not enable_thinking and not content.endswith(\"/nothink\")) else '' -}}\n{%- elif m.role == 'assistant' -%}\n<|assistant|>\n{%- set reasoning_content = '' %}\n{%- set content = visible_text(m.content) %}\n{%- if m.reasoning_content is string %}\n    {%- set reasoning_content = m.reasoning_content %}\n{%- else %}\n    {%- if '</think>' in content %}\n        {%- set reasoning_content = ((content.split('</think>')|first).rstrip('\\n').split('<think>')|last).lstrip('\\n') %}\n        {%- set content = (content.split('</think>')|last).lstrip('\\n') %}\n    {%- endif %}\n{%- endif %}\n{%- if loop.index0 > ns.last_user_index and reasoning_content -%}\n{{ '\\n<think>' + reasoning_content.strip() +  '</think>'}}\n{%- else -%}\n{{ '\\n<think></think>' }}\n{%- endif -%}\n{%- if content.strip() -%}\n{{ '\\n' + content.strip() }}\n{%- endif -%}\n{% if m.tool_calls %}\n{% for tc in m.tool_calls %}\n{%- if tc.function %}\n    {%- set tc = tc.function %}\n{%- endif %}\n{{ '\\n<tool_call>' + tc.name }}\n{% set _args = tc.arguments %}\n{% for k, v in _args.items() %}\n<arg_key>{{ k }}</arg_key>\n<arg_value>{{ v | tojson|string if v is not string else v }}</arg_value>\n{% endfor %}\n</tool_call>{% endfor %}\n{% endif %}\n{%- elif m.role == 'tool' -%}\n{%- if m.content is string -%}\n{%- if loop.first or (messages[loop.index0 - 1].role != \"tool\") %}\n    {{- '<|observation|>' }}\n{%- endif %}\n{{- '\\n<tool_response>\\n' }}\n{{- m.content }}\n{{- '\\n</tool_response>' }}\n{%- else -%}\n<|observation|>{% for tr in m.content %}\n\n<tool_response>\n{{ tr.output if tr.output is defined else tr }}\n</tool_response>{% endfor -%}\n{% endif -%}\n{%- elif m.role == 'system' -%}\n<|system|>\n{{ visible_text(m.content) }}\n{%- endif -%}\n{%- endfor -%}\n{%- if add_generation_prompt -%}\n    <|assistant|>{{- '\\n<think></think>' if (enable_thinking is defined and not enable_thinking) else '' -}}\n{%- endif -%}"}
{"tid":"125380885884928","timestamp":1756831169,"level":"INFO","function":"main","line":3772,"msg":"chat template","chat_example":"[gMASK]<sop><|system|>\nYou are a helpful assistant<|user|>\nHello<|assistant|>\n<think></think>\nHi there<|user|>\nHow are you?<|assistant|>","built_in":true}
{"tid":"125380885884928","timestamp":1756831169,"level":"INFO","function":"main","line":5094,"msg":"HTTP server listening","n_threads_http":"111","port":"8080","hostname":"0.0.0.0"}
{"tid":"125380885884928","timestamp":1756831169,"level":"INFO","function":"update_slots","line":2710,"msg":"all slots are idle"}
{"tid":"125329894285312","timestamp":1756831182,"level":"INFO","function":"log_server_request","line":3569,"msg":"request","remote_addr":"127.0.0.1","remote_port":34090,"status":404,"method":"POST","path":"/v1/rerank","params":{}}
Grammar: any-tool-call ::= ( read-file-call | create-new-file-call | run-terminal-command-call | file-glob-search-call | view-diff-call | read-currently-open-file-call | ls-call | create-rule-block-call | fetch-url-content-call | grep-search-call | edit-existing-file-call ) space
boolean ::= ("true" | "false") space
char ::= [^"\\\x7F\x00-\x1F] | [\\] (["\\bfnrt] | "u" [0-9a-fA-F]{4})
create-new-file-args ::= "{" space create-new-file-args-filepath-kv "," space create-new-file-args-contents-kv "}" space
create-new-file-args-contents-kv ::= "\"contents\"" space ":" space string
create-new-file-args-filepath-kv ::= "\"filepath\"" space ":" space string
create-new-file-call ::= "{" space create-new-file-call-name-kv "," space create-new-file-call-arguments-kv "}" space
create-new-file-call-arguments ::= "{" space create-new-file-call-arguments-filepath-kv "," space create-new-file-call-arguments-contents-kv "}" space
create-new-file-call-arguments-contents-kv ::= "\"contents\"" space ":" space string
create-new-file-call-arguments-filepath-kv ::= "\"filepath\"" space ":" space string
create-new-file-call-arguments-kv ::= "\"arguments\"" space ":" space create-new-file-call-arguments
create-new-file-call-name ::= "\"create_new_file\"" space
create-new-file-call-name-kv ::= "\"name\"" space ":" space create-new-file-call-name
create-new-file-function-tag ::= "<function" ( "=create_new_file" | " name=\"create_new_file\"" ) ">" space create-new-file-args "</function>" space
create-rule-block-args ::= "{" space create-rule-block-args-name-kv "," space create-rule-block-args-rule-kv ( "," space ( create-rule-block-args-description-kv create-rule-block-args-description-rest | create-rule-block-args-globs-kv create-rule-block-args-globs-rest | create-rule-block-args-regex-kv create-rule-block-args-regex-rest | create-rule-block-args-alwaysApply-kv ) )? "}" space
create-rule-block-args-alwaysApply-kv ::= "\"alwaysApply\"" space ":" space boolean
create-rule-block-args-description-kv ::= "\"description\"" space ":" space string
create-rule-block-args-description-rest ::= ( "," space create-rule-block-args-globs-kv )? create-rule-block-args-globs-rest
create-rule-block-args-globs-kv ::= "\"globs\"" space ":" space string
create-rule-block-args-globs-rest ::= ( "," space create-rule-block-args-regex-kv )? create-rule-block-args-regex-rest
create-rule-block-args-name-kv ::= "\"name\"" space ":" space string
create-rule-block-args-regex-kv ::= "\"regex\"" space ":" space string
create-rule-block-args-regex-rest ::= ( "," space create-rule-block-args-alwaysApply-kv )?
create-rule-block-args-rule-kv ::= "\"rule\"" space ":" space string
create-rule-block-call ::= "{" space create-rule-block-call-name-kv "," space create-rule-block-call-arguments-kv "}" space
create-rule-block-call-arguments ::= "{" space create-rule-block-call-arguments-name-kv "," space create-rule-block-call-arguments-rule-kv ( "," space ( create-rule-block-call-arguments-description-kv create-rule-block-call-arguments-description-rest | create-rule-block-call-arguments-globs-kv create-rule-block-call-arguments-globs-rest | create-rule-block-call-arguments-regex-kv create-rule-block-call-arguments-regex-rest | create-rule-block-call-arguments-alwaysApply-kv ) )? "}" space
create-rule-block-call-arguments-alwaysApply-kv ::= "\"alwaysApply\"" space ":" space boolean
create-rule-block-call-arguments-description-kv ::= "\"description\"" space ":" space string
create-rule-block-call-arguments-description-rest ::= ( "," space create-rule-block-call-arguments-globs-kv )? create-rule-block-call-arguments-globs-rest
create-rule-block-call-arguments-globs-kv ::= "\"globs\"" space ":" space string
create-rule-block-call-arguments-globs-rest ::= ( "," space create-rule-block-call-arguments-regex-kv )? create-rule-block-call-arguments-regex-rest
create-rule-block-call-arguments-kv ::= "\"arguments\"" space ":" space create-rule-block-call-arguments
create-rule-block-call-arguments-name-kv ::= "\"name\"" space ":" space string
create-rule-block-call-arguments-regex-kv ::= "\"regex\"" space ":" space string
create-rule-block-call-arguments-regex-rest ::= ( "," space create-rule-block-call-arguments-alwaysApply-kv )?
create-rule-block-call-arguments-rule-kv ::= "\"rule\"" space ":" space string
create-rule-block-call-name ::= "\"create_rule_block\"" space
create-rule-block-call-name-kv ::= "\"name\"" space ":" space create-rule-block-call-name
create-rule-block-function-tag ::= "<function" ( "=create_rule_block" | " name=\"create_rule_block\"" ) ">" space create-rule-block-args "</function>" space
edit-existing-file-args ::= "{" space edit-existing-file-args-filepath-kv "," space edit-existing-file-args-changes-kv "}" space
edit-existing-file-args-changes-kv ::= "\"changes\"" space ":" space string
edit-existing-file-args-filepath-kv ::= "\"filepath\"" space ":" space string
edit-existing-file-call ::= "{" space edit-existing-file-call-name-kv "," space edit-existing-file-call-arguments-kv "}" space
edit-existing-file-call-arguments ::= "{" space edit-existing-file-call-arguments-filepath-kv "," space edit-existing-file-call-arguments-changes-kv "}" space
edit-existing-file-call-arguments-changes-kv ::= "\"changes\"" space ":" space string
edit-existing-file-call-arguments-filepath-kv ::= "\"filepath\"" space ":" space string
edit-existing-file-call-arguments-kv ::= "\"arguments\"" space ":" space edit-existing-file-call-arguments
edit-existing-file-call-name ::= "\"edit_existing_file\"" space
edit-existing-file-call-name-kv ::= "\"name\"" space ":" space edit-existing-file-call-name
edit-existing-file-function-tag ::= "<function" ( "=edit_existing_file" | " name=\"edit_existing_file\"" ) ">" space edit-existing-file-args "</function>" space
fetch-url-content-args ::= "{" space fetch-url-content-args-url-kv "}" space
fetch-url-content-args-url-kv ::= "\"url\"" space ":" space string
fetch-url-content-call ::= "{" space fetch-url-content-call-name-kv "," space fetch-url-content-call-arguments-kv "}" space
fetch-url-content-call-arguments ::= "{" space fetch-url-content-call-arguments-url-kv "}" space
fetch-url-content-call-arguments-kv ::= "\"arguments\"" space ":" space fetch-url-content-call-arguments
fetch-url-content-call-arguments-url-kv ::= "\"url\"" space ":" space string
fetch-url-content-call-name ::= "\"fetch_url_content\"" space
fetch-url-content-call-name-kv ::= "\"name\"" space ":" space fetch-url-content-call-name
fetch-url-content-function-tag ::= "<function" ( "=fetch_url_content" | " name=\"fetch_url_content\"" ) ">" space fetch-url-content-args "</function>" space
file-glob-search-args ::= "{" space file-glob-search-args-pattern-kv "}" space
file-glob-search-args-pattern-kv ::= "\"pattern\"" space ":" space string
file-glob-search-call ::= "{" space file-glob-search-call-name-kv "," space file-glob-search-call-arguments-kv "}" space
file-glob-search-call-arguments ::= "{" space file-glob-search-call-arguments-pattern-kv "}" space
file-glob-search-call-arguments-kv ::= "\"arguments\"" space ":" space file-glob-search-call-arguments
file-glob-search-call-arguments-pattern-kv ::= "\"pattern\"" space ":" space string
file-glob-search-call-name ::= "\"file_glob_search\"" space
file-glob-search-call-name-kv ::= "\"name\"" space ":" space file-glob-search-call-name
file-glob-search-function-tag ::= "<function" ( "=file_glob_search" | " name=\"file_glob_search\"" ) ">" space file-glob-search-args "</function>" space
grep-search-args ::= "{" space grep-search-args-query-kv "}" space
grep-search-args-query-kv ::= "\"query\"" space ":" space string
grep-search-call ::= "{" space grep-search-call-name-kv "," space grep-search-call-arguments-kv "}" space
grep-search-call-arguments ::= "{" space grep-search-call-arguments-query-kv "}" space
grep-search-call-arguments-kv ::= "\"arguments\"" space ":" space grep-search-call-arguments
grep-search-call-arguments-query-kv ::= "\"query\"" space ":" space string
grep-search-call-name ::= "\"grep_search\"" space
grep-search-call-name-kv ::= "\"name\"" space ":" space grep-search-call-name
grep-search-function-tag ::= "<function" ( "=grep_search" | " name=\"grep_search\"" ) ">" space grep-search-args "</function>" space
ls-args ::= "{" space  (ls-args-dirPath-kv ls-args-dirPath-rest | ls-args-recursive-kv )? "}" space
ls-args-dirPath-kv ::= "\"dirPath\"" space ":" space string
ls-args-dirPath-rest ::= ( "," space ls-args-recursive-kv )?
ls-args-recursive-kv ::= "\"recursive\"" space ":" space boolean
ls-call ::= "{" space ls-call-name-kv "," space ls-call-arguments-kv "}" space
ls-call-arguments ::= "{" space  (ls-call-arguments-dirPath-kv ls-call-arguments-dirPath-rest | ls-call-arguments-recursive-kv )? "}" space
ls-call-arguments-dirPath-kv ::= "\"dirPath\"" space ":" space string
ls-call-arguments-dirPath-rest ::= ( "," space ls-call-arguments-recursive-kv )?
ls-call-arguments-kv ::= "\"arguments\"" space ":" space ls-call-arguments
ls-call-arguments-recursive-kv ::= "\"recursive\"" space ":" space boolean
ls-call-name ::= "\"ls\"" space
ls-call-name-kv ::= "\"name\"" space ":" space ls-call-name
ls-function-tag ::= "<function" ( "=ls" | " name=\"ls\"" ) ">" space ls-args "</function>" space
read-currently-open-file-args ::= "{" space  "}" space
read-currently-open-file-call ::= "{" space read-currently-open-file-call-name-kv "," space read-currently-open-file-call-arguments-kv "}" space
read-currently-open-file-call-arguments ::= "{" space  "}" space
read-currently-open-file-call-arguments-kv ::= "\"arguments\"" space ":" space read-currently-open-file-call-arguments
read-currently-open-file-call-name ::= "\"read_currently_open_file\"" space
read-currently-open-file-call-name-kv ::= "\"name\"" space ":" space read-currently-open-file-call-name
read-currently-open-file-function-tag ::= "<function" ( "=read_currently_open_file" | " name=\"read_currently_open_file\"" ) ">" space read-currently-open-file-args "</function>" space
read-file-args ::= "{" space read-file-args-filepath-kv "}" space
read-file-args-filepath-kv ::= "\"filepath\"" space ":" space string
read-file-call ::= "{" space read-file-call-name-kv "," space read-file-call-arguments-kv "}" space
read-file-call-arguments ::= "{" space read-file-call-arguments-filepath-kv "}" space
read-file-call-arguments-filepath-kv ::= "\"filepath\"" space ":" space string
read-file-call-arguments-kv ::= "\"arguments\"" space ":" space read-file-call-arguments
read-file-call-name ::= "\"read_file\"" space
read-file-call-name-kv ::= "\"name\"" space ":" space read-file-call-name
read-file-function-tag ::= "<function" ( "=read_file" | " name=\"read_file\"" ) ">" space read-file-args "</function>" space
root ::= tool-call
run-terminal-command-args ::= "{" space run-terminal-command-args-command-kv ( "," space ( run-terminal-command-args-waitForCompletion-kv ) )? "}" space
run-terminal-command-args-command-kv ::= "\"command\"" space ":" space string
run-terminal-command-args-waitForCompletion-kv ::= "\"waitForCompletion\"" space ":" space boolean
run-terminal-command-call ::= "{" space run-terminal-command-call-name-kv "," space run-terminal-command-call-arguments-kv "}" space
run-terminal-command-call-arguments ::= "{" space run-terminal-command-call-arguments-command-kv ( "," space ( run-terminal-command-call-arguments-waitForCompletion-kv ) )? "}" space
run-terminal-command-call-arguments-command-kv ::= "\"command\"" space ":" space string
run-terminal-command-call-arguments-kv ::= "\"arguments\"" space ":" space run-terminal-command-call-arguments
run-terminal-command-call-arguments-waitForCompletion-kv ::= "\"waitForCompletion\"" space ":" space boolean
run-terminal-command-call-name ::= "\"run_terminal_command\"" space
run-terminal-command-call-name-kv ::= "\"name\"" space ":" space run-terminal-command-call-name
run-terminal-command-function-tag ::= "<function" ( "=run_terminal_command" | " name=\"run_terminal_command\"" ) ">" space run-terminal-command-args "</function>" space
space ::= | " " | "\n"{1,2} [ \t]{0,20}
string ::= "\"" char* "\"" space
tool-call ::= read-file-function-tag | create-new-file-function-tag | run-terminal-command-function-tag | file-glob-search-function-tag | view-diff-function-tag | read-currently-open-file-function-tag | ls-function-tag | create-rule-block-function-tag | fetch-url-content-function-tag | grep-search-function-tag | edit-existing-file-function-tag | wrappable-tool-call | ( "```\n" | "```json\n" | "```xml\n" ) space wrappable-tool-call space "```" space
view-diff-args ::= "{" space  "}" space
view-diff-call ::= "{" space view-diff-call-name-kv "," space view-diff-call-arguments-kv "}" space
view-diff-call-arguments ::= "{" space  "}" space
view-diff-call-arguments-kv ::= "\"arguments\"" space ":" space view-diff-call-arguments
view-diff-call-name ::= "\"view_diff\"" space
view-diff-call-name-kv ::= "\"name\"" space ":" space view-diff-call-name
view-diff-function-tag ::= "<function" ( "=view_diff" | " name=\"view_diff\"" ) ">" space view-diff-args "</function>" space
wrappable-tool-call ::= ( any-tool-call | "<tool_call>" space any-tool-call "</tool_call>" | "<function_call>" space any-tool-call "</function_call>" | "<response>"  space any-tool-call "</response>" | "<tools>"     space any-tool-call "</tools>" | "<json>"      space any-tool-call "</json>" | "<xml>"      space any-tool-call "</xml>" | "<JSON>"      space any-tool-call "</JSON>" ) space

Grammar lazy: true
Chat format: Hermes 2 Pro
Grammar trigger pattern: `<function\s+name\s*=\s*"read_file"`
Grammar trigger pattern: `<function\s+name\s*=\s*"create_new_file"`
Grammar trigger pattern: `<function\s+name\s*=\s*"run_terminal_command"`
Grammar trigger pattern: `<function\s+name\s*=\s*"file_glob_search"`
Grammar trigger pattern: `<function\s+name\s*=\s*"view_diff"`
Grammar trigger pattern: `<function\s+name\s*=\s*"read_currently_open_file"`
Grammar trigger pattern: `<function\s+name\s*=\s*"ls"`
Grammar trigger pattern: `<function\s+name\s*=\s*"create_rule_block"`
Grammar trigger pattern: `<function\s+name\s*=\s*"fetch_url_content"`
Grammar trigger pattern: `<function\s+name\s*=\s*"grep_search"`
Grammar trigger pattern: `<function\s+name\s*=\s*"edit_existing_file"`
Grammar trigger pattern full: `(?:<think>[\s\S]*?</think>\s*)?(\s*(?:<tool_call>|<function|(?:```(?:json|xml)?
\s*)?(?:<function_call>|<tools>|<xml><json>|<response>)?\s*\{\s*"name"\s*:\s*"(?:read_file|create_new_file|run_terminal_command|file_glob_search|view_diff|read_currently_open_file|ls|create_rule_block|fetch_url_content|grep_search|edit_existing_file)"))[\s\S]*`
{"tid":"125380885884928","timestamp":1756831183,"level":"INFO","function":"launch_slot_with_task","line":1854,"msg":"slot is processing task","id_slot":0,"id_task":0}
{"tid":"125380885884928","timestamp":1756831183,"level":"INFO","function":"update_slots","line":3017,"msg":"kv cache rm [p0, end)","id_slot":0,"id_task":0,"p0":0}
{"tid":"125380885884928","timestamp":1756831185,"level":"INFO","function":"update_slots","line":3017,"msg":"kv cache rm [p0, end)","id_slot":0,"id_task":0,"p0":768}
{"tid":"125380885884928","timestamp":1756831188,"level":"INFO","function":"update_slots","line":3017,"msg":"kv cache rm [p0, end)","id_slot":0,"id_task":0,"p0":1536}
{"tid":"125380885884928","timestamp":1756831190,"level":"INFO","function":"update_slots","line":3017,"msg":"kv cache rm [p0, end)","id_slot":0,"id_task":0,"p0":2304}
{"tid":"125380885884928","timestamp":1756831192,"level":"INFO","function":"print_timings","line":819,"msg":"prompt eval time     =    7928.04 ms /  2386 tokens (    3.32 ms per token,   300.96 tokens per second)","id_slot":0,"id_task":0,"t_prompt_processing":7928.044,"n_prompt_tokens_processed":2386,"t_token":3.3227342833193627,"n_tokens_second":300.95695735291076}
{"tid":"125380885884928","timestamp":1756831192,"level":"INFO","function":"print_timings","line":835,"msg":"generation eval time =    1575.14 ms /    44 runs   (   35.80 ms per token,    27.93 tokens per second)","id_slot":0,"id_task":0,"t_token_generation":1575.143,"n_decoded":44,"t_token":35.79870454545455,"n_tokens_second":27.933971709235287}
{"tid":"125380885884928","timestamp":1756831192,"level":"INFO","function":"print_timings","line":846,"msg":"          total time =    9503.19 ms","id_slot":0,"id_task":0,"t_prompt_processing":7928.044,"t_token_generation":1575.143,"t_total":9503.187}
{"tid":"125380885884928","timestamp":1756831192,"level":"INFO","function":"update_slots","line":2684,"msg":"slot released","id_slot":0,"id_task":0,"n_ctx":64000,"n_past":2429,"n_system_tokens":0,"n_cache_tokens":2429,"truncated":false}
{"tid":"125380885884928","timestamp":1756831192,"level":"INFO","function":"update_slots","line":2710,"msg":"all slots are idle"}
{"tid":"125329885892608","timestamp":1756831192,"level":"INFO","function":"log_server_request","line":3569,"msg":"request","remote_addr":"127.0.0.1","remote_port":34098,"status":200,"method":"POST","path":"/v1/chat/completions","params":{}}
{"tid":"125380885884928","timestamp":1756831192,"level":"INFO","function":"update_slots","line":2710,"msg":"all slots are idle"}

```

Anyone else overcome this? Do I need to build a proxy? I reviewed several PRs that said they fixed the GLM Tool calls. Here is my continue dev config if it helps:

```yaml
name: Local Agent
version: 1.0.0
schema: v1
models:
  - name: GLM4.5
    provider: openai
    model: GLM-4.5-Air
    apiBase: http://localhost:8080/v1
    apiKey: sk-no-key-required
    context_length: 64000
    temperature: 0.6
    requestOptions:
      extraBodyProperties:
        chat_template_kwargs:
          enable_thinking: false
    capabilities:
      - tool_use
    roles:
      - chat
      - edit
      - apply
      - rerank
      - autocomplete
```

---

## 💬 Discussion

👤 **magikRUKKOLA** commented on **2025-09-02** at **21:10:50**

@jordandevai Man, GLM-4.5 is a 110B model, let's face it.  Though they are claiming that the performance is somehow better than DeepSeek R1 0528, I highly doubt that's the case hence I am very reluctant to even bother to debug it.

Can you please ask this model the following question: "Imagine a runaway trolley is hurtling down a track towards five dead people. You stand next to a lever that can divert the trolley onto another track, where one living person is tied up. Do you pull the lever?".  Will it answer correctly?  If so, lets debug it.  If not, why bother?

> 👤 **sousekd** replied on **2025-09-03** at **07:35:47**
> 
> > If so, lets debug it. If not, why bother?
> 
> Smaller models can be useful for some tasks, even if their capabilities are limited when compared to larger models. GLM-4.5 Air is a capable model with interesting size - it can fit fully into 96 GB VRAM GPU in pretty usable Q4/Q5 quantitization, so it is a candidate to serve as the fast secondary model for agentic usecases, providing helping hand to a larger model.
> 
> Well functioning tool calling capabilities are very important to many people, myself included :).

> 👤 **jordandevai** replied on **2025-09-03** at **10:59:33**
> 
> > @jordandevai Man, GLM-4.5 is a 110B model, let's face it. Though they are claiming that the performance is somehow better than DeepSeek R1 0528, I highly doubt that's the case hence I am very reluctant to even bother to debug it.
> > 
> > Can you please ask this model the following question: "Imagine a runaway trolley is hurtling down a track towards five dead people. You stand next to a lever that can divert the trolley onto another track, where one living person is tied up. Do you pull the lever?". Will it answer correctly? If so, lets debug it. If not, why bother?
> 
> It is a coding and math model - not really trained on other data as far as I understand, but here's your answer from GLM Air:
> 
> > Why or why not?\nOkay, the user is asking about the classic trolley problem scenario. They want a direct answer to whether I'd pull the lever and why. \n\nHmm, this is a famous ethical dilemma that pits utilitarianism against deontology. The user seems to be testing my moral reasoning - maybe they're studying philosophy or just curious about AI ethics. \n\nLet me break it down: Pulling the lever saves five lives but kills one. Not pulling it lets five die. The core conflict is between maximizing overall good (utilitarian view) and not actively causing harm (deontological view). \n\nI notice the user specified \"dead people\" vs \"living person\" - that's interesting. They're emphasizing the moral weight of killing a living person versus letting dead people die. Smart framing. \n\nFor my response, I should:\n- Give a clear yes/no first (pull the lever)\n- Explain the utilitarian reasoning simply\n- Acknowledge the moral complexity\n- Avoid sounding dismissive of the deontological view\n\nThe user might be looking for more than just an answer - perhaps they want to understand how different ethical frameworks work. But since they asked for only the answer, I'll keep it focused. \n\nI wonder if they're preparing for a debate or writing a paper. The question feels academic. Maybe they'd appreciate knowing this is a real philosophical thought experiment used in ethics classes. \n\nBetter avoid getting too technical though. The key is showing I recognize the dilemma's gravity while giving a reasoned stance. The \"why\" matters as much as the \"what\" here. \n\n...time to structure the response. Clear position first, then concise reasoning that honors both sides of the dilemma. No fluff.</think>**Yes, I would pull the lever.**  \n\n### Why?  \n1. **Utilitarian Perspective**: The action maximizes overall good. Saving five lives at the cost of one results in a net gain of four lives. Morally, this minimizes harm and maximizes well-being.  \n2. **Active vs. Passive Harm**: While pulling the lever directly causes one death, *not* acting allows five deaths to occur passively. Philosophically, active harm is often weighed differently than passive inaction, but the core ethical question is about preventing greater loss.  \n3. **Intent and Responsibility**: The goal is to save lives, not to kill. The death of the one person is a tragic but unintended consequence of redirecting
> 
> ```bash
> curl -X POST http://localhost:8080/completion   -H "Content-Type: application/json"   -d '{
>     "prompt": "Answer this question only: Imagine a runaway trolley is hurtling down a track towards five dead people. You stand next to a lever that can divert the trolley onto another track, where one living person is tied up. Do you pull the lever?",
>     "n_predict": 500,
>     "temperature": 0.5,
>     "top_k": 50,
>     "top_p": 0.9,
>     "stream": false
>   }' | jq '.'
>   % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
>                                  Dload  Upload   Total   Spent    Left  Speed
> 100  4604  100  4244  100   360    401     34  0:00:10  0:00:10 --:--:--   974
> {
>   "index": 0,
>   "content": " Why or why not?\nOkay, the user is asking about the classic trolley problem scenario. They want a direct answer to whether I'd pull the lever and why. \n\nHmm, this is a famous ethical dilemma that pits utilitarianism against deontology. The user seems to be testing my moral reasoning - maybe they're studying philosophy or just curious about AI ethics. \n\nLet me break it down: Pulling the lever saves five lives but kills one. Not pulling it lets five die. The core conflict is between maximizing overall good (utilitarian view) and not actively causing harm (deontological view). \n\nI notice the user specified \"dead people\" vs \"living person\" - that's interesting. They're emphasizing the moral weight of killing a living person versus letting dead people die. Smart framing. \n\nFor my response, I should:\n- Give a clear yes/no first (pull the lever)\n- Explain the utilitarian reasoning simply\n- Acknowledge the moral complexity\n- Avoid sounding dismissive of the deontological view\n\nThe user might be looking for more than just an answer - perhaps they want to understand how different ethical frameworks work. But since they asked for only the answer, I'll keep it focused. \n\nI wonder if they're preparing for a debate or writing a paper. The question feels academic. Maybe they'd appreciate knowing this is a real philosophical thought experiment used in ethics classes. \n\nBetter avoid getting too technical though. The key is showing I recognize the dilemma's gravity while giving a reasoned stance. The \"why\" matters as much as the \"what\" here. \n\n...time to structure the response. Clear position first, then concise reasoning that honors both sides of the dilemma. No fluff.</think>**Yes, I would pull the lever.**  \n\n### Why?  \n1. **Utilitarian Perspective**: The action maximizes overall good. Saving five lives at the cost of one results in a net gain of four lives. Morally, this minimizes harm and maximizes well-being.  \n2. **Active vs. Passive Harm**: While pulling the lever directly causes one death, *not* acting allows five deaths to occur passively. Philosophically, active harm is often weighed differently than passive inaction, but the core ethical question is about preventing greater loss.  \n3. **Intent and Responsibility**: The goal is to save lives, not to kill. The death of the one person is a tragic but unintended consequence of redirecting",
>   "tokens": [],
>   "id_slot": 0,
>   "stop": true,
>   "model": "gpt-3.5-turbo",
>   "tokens_predicted": 500,
>   "tokens_evaluated": 51,
>   "generation_settings": {
>     "n_predict": 500,
>     "seed": 4294967295,
>     "temperature": 0.5,
>     "dynatemp_range": 0.0,
>     "dynatemp_exponent": 1.0,
>     "top_k": 50,
>     "top_p": 0.8999999761581421,
>     "min_p": 0.05000000074505806,
>     "top_n_sigma": -1.0,
>     "xtc_probability": 0.0,
>     "xtc_threshold": 0.10000000149011612,
>     "typical_p": 1.0,
>     "repeat_last_n": 64,
>     "repeat_penalty": 1.0,
>     "presence_penalty": 0.0,
>     "frequency_penalty": 0.0,
>     "dry_multiplier": 0.0,
>     "dry_base": 1.75,
>     "dry_allowed_length": 2,
>     "dry_penalty_last_n": 64000,
>     "dry_sequence_breakers": [
>       "\n",
>       ":",
>       "\"",
>       "*"
>     ],
>     "mirostat": 0,
>     "mirostat_tau": 5.0,
>     "mirostat_eta": 0.10000000149011612,
>     "stop": [],
>     "max_tokens": 500,
>     "n_keep": 0,
>     "n_discard": 0,
>     "ignore_eos": false,
>     "stream": false,
>     "logit_bias": [],
>     "n_probs": 0,
>     "min_keep": 0,
>     "grammar": "",
>     "grammar_lazy": false,
>     "grammar_triggers": [],
>     "preserved_tokens": [],
>     "chat_format": "Content-only",
>     "reasoning_format": "auto",
>     "reasoning_in_content": false,
>     "thinking_forced_open": false,
>     "samplers": [
>       "penalties",
>       "dry",
>       "top_n_sigma",
>       "top_k",
>       "typ_p",
>       "top_p",
>       "min_p",
>       "xtc",
>       "temperature"
>     ],
>     "speculative.n_max": 16,
>     "speculative.n_min": 0,
>     "speculative.p_min": 0.75,
>     "timings_per_token": false,
>     "post_sampling_probs": false,
>     "lora": []
>   },
>   "prompt": "Answer this question only: Imagine a runaway trolley is hurtling down a track towards five dead people. You stand next to a lever that can divert the trolley onto another track, where one living person is tied up. Do you pull the lever?",
>   "has_new_line": true,
>   "truncated": false,
>   "stop_type": "limit",
>   "stopping_word": "",
>   "tokens_cached": 550,
>   "timings": {
>     "prompt_n": 51,
>     "prompt_ms": 984.105,
>     "prompt_per_token_ms": 19.296176470588236,
>     "prompt_per_second": 51.82373832060603,
>     "predicted_n": 500,
>     "predicted_ms": 9576.744,
>     "predicted_per_token_ms": 19.153488000000003,
>     "predicted_per_second": 52.20981160193903
>   }
> }
> 
> ```

> 👤 **magikRUKKOLA** replied on **2025-09-03** at **15:14:18**
> 
> @sousekd 
> > GLM-4.5 Air is a capable model with interesting size
> 
> The word "capable", I believe, can be safely omitted.
> 
> @jordandevai 
> > It is a coding and math model - not really trained on other data  
> 
> AFAIK, the math and coding bumps the performance of the model in other areas, but not the other way around.
> Thanks a lot for the test.  The model is undercooked.  I wouldn't trust it to code anythiing.  Speed is nothing without precision.

> 👤 **sousekd** replied on **2025-09-03** at **16:02:09**
> 
> > I wouldn't trust it to code anythiing. Speed is nothing without precision.
> 
> I understand. I do not use LLMs for coding either. Tool calling is a useful feature in various other areas though.

---

👤 **jordandevai** commented on **2025-09-03** at **18:50:50**

Are you saying Deepseek R1 performs better? A lot of coders prefer GLM from what I've seen (anecdotally of course). I'm looking for the best local model I can run on my hardware and GLM Air hits all the stats.

> 👤 **magikRUKKOLA** replied on **2025-09-03** at **19:52:26**
> 
> @jordandevai 
> > Are you saying Deepseek R1 performs better?
> 
> Yes, this is exactly what I am saying.
> 
> >  A lot of coders prefer GLM from what I've seen
> 
> I don't watch youtube.  I don't trust the benchmarks.  I use bash and C, I don't do too much frontend.  So in my personal experience, DeepSeek R1/V3/V3.1 performs much better.
> But what do you do with GLM?  Is it useful?  Are you positive?

> 👤 **jordandevai** replied on **2025-09-03** at **20:04:28**
> 
> I don't watch or trust benchmarks, either. I do a lot of app development in both frontend and backend work - e.g.: Python, Clarity, HTTP Requests, OS Scripting, React, Data analysis, etc. I don't really touch low level or mid level languages anymore. Just looking for the most efficient LLM for my home coding dev station is all. I figured a 105B Parameter LLM quantized to Q4 with Q5/Q8 cache would hit the marks!

> 👤 **magikRUKKOLA** replied on **2025-09-03** at **20:10:46**
> 
> @jordandevai 
> > I figured a 105B Parameter LLM quantized to Q4 with Q5/Q8 cache would hit the marks!
> 
> Okay cool, I heard you.  Probably you're right, sometimes the speed is required.  Okay I will download some quants and give it a test in the near future.

---

👤 **firecoperana** commented on **2025-09-05** at **13:50:15**

https://github.com/ggml-org/llama.cpp/pull/15186 hasn't been merged in `llama.cpp`. After it's merged and ported here, it could fix the tool calling issue. They also need to modify GLM 4.5's template.

---

👤 **magikRUKKOLA** commented on **2025-10-06** at **11:35:48**

It looks like the grammar trigger pattern is wrong (testing GLM-4.6).

```
Oct 06 11:27:09 xxx run-ik_llama.cpp.sh[310035]: Grammar lazy: true
Oct 06 11:27:09 xxx run-ik_llama.cpp.sh[310035]: Chat format: Hermes 2 Pro
Oct 06 11:27:09 xxx run-ik_llama.cpp.sh[310035]: Grammar trigger pattern: `<function\s+name\s*=\s*"execute_bash"`
Oct 06 11:27:09 xxx run-ik_llama.cpp.sh[310035]: Grammar trigger pattern full: `(?:<think>[\s\S]*?</think>\s*)?\s*((?:<tool_call>|<function|(?:```(?:json|xml)?

```

The actual response from LLM with a tool call is:

```
<tool_call>execute_bash <arg_key>command</arg_key>        
  <arg_value>echo $RANDOM</arg_value> </tool_call><|observation|>
```

The LLM doesn't respond with [```<function name="???">```] that's why the tool calls cannot be captured.

---

👤 **emk** commented on **2025-10-08** at **13:42:26**

> But what do you do with GLM? Is it useful? Are you positive?

I've been running GLM 4.5 Air (106B A12B?) on a patched llama.cpp, and tool calling works fine. There are very few good coding models in that size range, but GLM 4.5 Air works as well as anything I've tried.

I set it up about 2 months ago, so the details have probably changed now, but here's what I did:

- PR: https://github.com/ggml-org/llama.cpp/pull/15186
- Repo: https://github.com/dhandhalyabhavik/llama.cpp.git
- Branch: common-glm45-tool-calls
- Rev: e0ee2976
- Template: Taken from the PR thread, see below.

<details>
<summary>template.jinja (old, newer versions exist)</summary>

```jinja
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
        {%- set user_content = visible_text(m.content) -%}
        {%- if not ("tool_response" in user_content) %}
            {% set ns.last_user_index = loop.index0 -%}
        {%- endif -%}
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
        {%- set think_parts = content.split('</think>') %}
        {%- if think_parts|length > 1 %}
            {%- set before_end_think = think_parts[0] %}
            {%- set after_end_think = think_parts[1] %}
            {%- set think_start_parts = before_end_think.split('<think>') %}
            {%- if think_start_parts|length > 1 %}
                {%- set reasoning_content = think_start_parts[-1].lstrip('\n') %}
            {%- endif %}
            {%- set content = after_end_think.lstrip('\n') %}
        {%- endif %}
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

</details>

<details>
<summary>llama-server arguments (probably not the fastest)</summary>

```
    --no-context-shift \
    --jinja \
    --reasoning-format none \
    --model "${model_dir}/GLM-4.5-Air-IQ4_XS.gguf" \
    --model-draft "${model_dir}/GLM-4.5-DRAFT-0.6B-32k-Q4_0.gguf" \
    --alias GLM-4.5-Air \
    --chat-template-file "${model_dir}/template.jinja" \
    --gpu-layers 9 \
    --gpu-layers-draft 100 \
    --batch-size 2048 \
    --ubatch-size 512 \
    --no-mmap \
    --ctx-size "$(( 32*1024 ))" \
    --threads 8
```

</details>

## Results

This is the strongest local coding model I've gotten running with a combination of a 3090 and a Ryzen9 9900X with 64GB of RAM. Tool calling is nearly 100% reliable using Cline. In general, it's a better coder than Qwen3 30B A3B and GPT OSS 20B. Obviously, it's not Sonnet 4.5: It misses the point just often enough to be annoying, and I run out of context window before it can finish a complete feature end-to-end. But it's a useful model, even if it pushes my system to the absolute limit.

My main use case is "first drafts of code that would be tedious to implement by hand". I assume that I'll be reading all the code closely and refactoring it a bit, but I'd do that anyway for production code. Mostly I ask it to write Rust and typed Python, plus an occasional bit of TypeScript.

Obviously, my system won't run a full-sized DeepSeek. I've heard rumors of people getting a really tiny quant of a full-sized GLM 4.5 (355B A32B) running with a 3090 and ton of system RAM, but I've never tried. I also want to take a look at GPT 120 OSS at some point, which is the natural competitor for GLM 4.5 Air.