## 🗣️ [Discussion #816](https://github.com/ikawrakow/ik_llama.cpp/discussions/816) - Need a little help, crazy vram issues.

| **Author** | `RedDragonGecko` |
| :--- | :--- |
| **State** | ✅ **Answered** |
| **Created** | 2025-10-03 |
| **Updated** | 2025-10-10 |

---

## 📄 Description

Didn't want to open an issue as I assume its something I'm doing wrong. Been using llamacpp for awhile, thought I'd try out ik_llama but it isn't going well.

Tried loading up ubergarm/GLM-4.6-GGUF IQ5_K
using;
./build/bin/llama-server -m '/home/server/Downloads/GLM-4.6-IQ5_K-00001-of-00006.gguf' -c 16000 --no-warmup -ngl 999 -fa -ctk q8_0 -ctv q8_0 -ts 17,12,12.5,12,7 --jinja -t 24 --threads-batch 32 --alias GLM-4.6-IQ5K-IK --no-mmap -b 4096 -ub 4096 --parallel 1 -fmoe -ot ".ffn_(up|down|gate)_exps.=CPU"

errors out with;
ggml_backend_cuda_buffer_type_alloc_buffer: allocating 965110.94 MiB on device 0: cudaMalloc failed: out of memory

Heres the whole output;
./build/bin/llama-server -m '/home/server/Downloads/GLM-4.6-IQ5_K-00001-of-00006.gguf' -c 16000 --no-warmup -ngl 999 -fa -ctk q8_0 -ctv q8_0 -ts 17,12,12.5,12,7 --jinja -t 24 --threads-batch 32 --alias GLM-4.6-IQ5K-IK --no-mmap -b 4096 -ub 4096 --parallel 1 -fmoe -ot ".ffn_(up|down|gate)_exps.=CPU"
ggml_cuda_init: GGML_CUDA_FORCE_MMQ:    no
ggml_cuda_init: GGML_CUDA_FORCE_CUBLAS: no
ggml_cuda_init: found 5 CUDA devices:
  Device 0: NVIDIA GeForce RTX 3090, compute capability 8.6, VMM: yes
  Device 1: NVIDIA GeForce RTX 3090, compute capability 8.6, VMM: yes
  Device 2: NVIDIA GeForce RTX 3090, compute capability 8.6, VMM: yes
  Device 3: NVIDIA GeForce RTX 3090, compute capability 8.6, VMM: yes
  Device 4: NVIDIA GeForce RTX 3060, compute capability 8.6, VMM: yes
INFO [                    main] build info | tid="125885501370368" timestamp=1759527518 build=3901 commit="6051ba25"
INFO [                    main] system info | tid="125885501370368" timestamp=1759527518 n_threads=24 n_threads_batch=32 total_threads=48 system_info="AVX = 1 | AVX_VNNI = 0 | AVX2 = 1 | AVX512 = 0 | AVX512_VBMI = 0 | AVX512_VNNI = 0 | AVX512_BF16 = 0 | FMA = 1 | NEON = 0 | SVE = 0 | ARM_FMA = 0 | F16C = 1 | FP16_VA = 0 | WASM_SIMD = 0 | BLAS = 1 | SSE3 = 1 | SSSE3 = 1 | VSX = 0 | MATMUL_INT8 = 0 | LLAMAFILE = 1 | "
llama_model_loader: additional 5 GGUFs metadata loaded.
llama_model_loader: loaded meta data with 50 key-value pairs and 1759 tensors from /home/server/Downloads/GLM-4.6-IQ5_K-00001-of-00006.gguf (version GGUF V3 (latest))
llama_model_loader: Dumping metadata keys/values. Note: KV overrides do not apply in this output.
llama_model_loader: - kv   0:                       general.architecture str              = glm4moe
llama_model_loader: - kv   1:                               general.type str              = model
llama_model_loader: - kv   2:                               general.name str              = GLM 4.6
llama_model_loader: - kv   3:                            general.version str              = 4.6
llama_model_loader: - kv   4:                           general.basename str              = GLM
llama_model_loader: - kv   5:                         general.size_label str              = 160x19B
llama_model_loader: - kv   6:                            general.license str              = mit
llama_model_loader: - kv   7:                               general.tags arr[str,1]       = ["text-generation"]
llama_model_loader: - kv   8:                          general.languages arr[str,2]       = ["en", "zh"]
llama_model_loader: - kv   9:                        glm4moe.block_count u32              = 93
llama_model_loader: - kv  10:                     glm4moe.context_length u32              = 202752
llama_model_loader: - kv  11:                   glm4moe.embedding_length u32              = 5120
llama_model_loader: - kv  12:                glm4moe.feed_forward_length u32              = 12288
llama_model_loader: - kv  13:               glm4moe.attention.head_count u32              = 96
llama_model_loader: - kv  14:            glm4moe.attention.head_count_kv u32              = 8
llama_model_loader: - kv  15:                     glm4moe.rope.freq_base f32              = 1000000.000000
llama_model_loader: - kv  16:   glm4moe.attention.layer_norm_rms_epsilon f32              = 0.000010
llama_model_loader: - kv  17:                  glm4moe.expert_used_count u32              = 8
llama_model_loader: - kv  18:               glm4moe.attention.key_length u32              = 128
llama_model_loader: - kv  19:             glm4moe.attention.value_length u32              = 128
llama_model_loader: - kv  20:                          general.file_type u32              = 141
llama_model_loader: - kv  21:               glm4moe.rope.dimension_count u32              = 64
llama_model_loader: - kv  22:                       glm4moe.expert_count u32              = 160
llama_model_loader: - kv  23:         glm4moe.expert_feed_forward_length u32              = 1536
llama_model_loader: - kv  24:                glm4moe.expert_shared_count u32              = 1
llama_model_loader: - kv  25:          glm4moe.leading_dense_block_count u32              = 3
llama_model_loader: - kv  26:                 glm4moe.expert_gating_func u32              = 2
llama_model_loader: - kv  27:               glm4moe.expert_weights_scale f32              = 2.500000
llama_model_loader: - kv  28:                glm4moe.expert_weights_norm bool             = true
llama_model_loader: - kv  29:               glm4moe.nextn_predict_layers u32              = 1
llama_model_loader: - kv  30:               general.quantization_version u32              = 2
llama_model_loader: - kv  31:                       tokenizer.ggml.model str              = gpt2
llama_model_loader: - kv  32:                         tokenizer.ggml.pre str              = glm4
llama_model_loader: - kv  33:                      tokenizer.ggml.tokens arr[str,151552]  = ["!", "\"", "#", "$", "%", "&", "'", ...
llama_model_loader: - kv  34:                  tokenizer.ggml.token_type arr[i32,151552]  = [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, ...
llama_model_loader: - kv  35:                      tokenizer.ggml.merges arr[str,318088]  = ["Ġ Ġ", "Ġ ĠĠĠ", "ĠĠ ĠĠ", "...
llama_model_loader: - kv  36:                tokenizer.ggml.eos_token_id u32              = 151329
llama_model_loader: - kv  37:            tokenizer.ggml.padding_token_id u32              = 151329
llama_model_loader: - kv  38:                tokenizer.ggml.bos_token_id u32              = 151331
llama_model_loader: - kv  39:                tokenizer.ggml.eot_token_id u32              = 151336
llama_model_loader: - kv  40:            tokenizer.ggml.unknown_token_id u32              = 151329
llama_model_loader: - kv  41:                tokenizer.ggml.eom_token_id u32              = 151338
llama_model_loader: - kv  42:                    tokenizer.chat_template str              = [gMASK]<sop>\n{%- if tools -%}\n<|syste...
llama_model_loader: - kv  43:                      quantize.imatrix.file str              = /mnt/data/models/ubergarm/GLM-4.6-GGU...
llama_model_loader: - kv  44:                   quantize.imatrix.dataset str              = ubergarm-imatrix-calibration-corpus-v...
llama_model_loader: - kv  45:             quantize.imatrix.entries_count i32              = 1001
llama_model_loader: - kv  46:              quantize.imatrix.chunks_count i32              = 814
llama_model_loader: - kv  47:                                   split.no u16              = 0
llama_model_loader: - kv  48:                                split.count u16              = 6
llama_model_loader: - kv  49:                        split.tensors.count i32              = 1759
llama_model_loader: - type  f32:  835 tensors
llama_model_loader: - type q8_0:  652 tensors
llama_model_loader: - type iq5_k:  180 tensors
llama_model_loader: - type iq6_k:   92 tensors
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
llm_load_print_meta: n_ctx_train      = 202752
llm_load_print_meta: n_embd           = 5120
llm_load_print_meta: n_layer          = 93
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
llm_load_print_meta: n_ff             = 12288
llm_load_print_meta: n_expert         = 160
llm_load_print_meta: n_expert_used    = 8
llm_load_print_meta: causal attn      = 1
llm_load_print_meta: pooling type     = 0
llm_load_print_meta: rope type        = 2
llm_load_print_meta: rope scaling     = linear
llm_load_print_meta: freq_base_train  = 1000000.0
llm_load_print_meta: freq_scale_train = 1
llm_load_print_meta: n_ctx_orig_yarn  = 202752
llm_load_print_meta: rope_finetuned   = unknown
llm_load_print_meta: ssm_d_conv       = 0
llm_load_print_meta: ssm_d_inner      = 0
llm_load_print_meta: ssm_d_state      = 0
llm_load_print_meta: ssm_dt_rank      = 0
llm_load_print_meta: model type       = 355B.A32B
llm_load_print_meta: model ftype      = IQ5_K - 5.5 bpw
llm_load_print_meta: model params     = 356.786 B
llm_load_print_meta: model size       = 249.099 GiB (5.997 BPW) 
llm_load_print_meta: repeating layers = 247.902 GiB (5.995 BPW, 355.234 B parameters)
llm_load_print_meta: general.name     = GLM 4.6
print_info: vocab type       = BPE
print_info: n_vocab          = 151552
print_info: n_merges         = 318088
print_info: BOS token        = 151331 '[gMASK]'
print_info: EOS token        = 151329 '<|endoftext|>'
print_info: EOT token        = 151336 '<|user|>'
print_info: EOM token        = 151338 '<|observation|>'
print_info: UNK token        = 151329 '<|endoftext|>'
print_info: PAD token        = 151329 '<|endoftext|>'
print_info: LF token         = 198 'Ċ'
print_info: FIM PRE token    = 151347 '<|code_prefix|>'
print_info: FIM SUF token    = 151349 '<|code_suffix|>'
print_info: FIM MID token    = 151348 '<|code_middle|>'
print_info: EOG token        = 151329 '<|endoftext|>'
print_info: EOG token        = 151336 '<|user|>'
print_info: EOG token        = 151338 '<|observation|>'
print_info: max token length = 1024
llm_load_tensors: ggml ctx size =    4.29 MiB
Tensor blk.3.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.3.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.3.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.4.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.4.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.4.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.5.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.5.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.5.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.6.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.6.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.6.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.7.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.7.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.7.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.8.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.8.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.8.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.9.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.9.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.9.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.10.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.10.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.10.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.11.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.11.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.11.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.12.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.12.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.12.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.13.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.13.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.13.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.14.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.14.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.14.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.15.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.15.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.15.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.16.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.16.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.16.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.17.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.17.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.17.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.18.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.18.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.18.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.19.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.19.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.19.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.20.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.20.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.20.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.21.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.21.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.21.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.22.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.22.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.22.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.23.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.23.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.23.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.24.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.24.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.24.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.25.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.25.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.25.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.26.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.26.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.26.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.27.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.27.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.27.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.28.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.28.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.28.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.29.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.29.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.29.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.30.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.30.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.30.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.31.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.31.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.31.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.32.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.32.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.32.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.33.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.33.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.33.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.34.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.34.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.34.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.35.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.35.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.35.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.36.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.36.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.36.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.37.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.37.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.37.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.38.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.38.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.38.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.39.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.39.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.39.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.40.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.40.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.40.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.41.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.41.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.41.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.42.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.42.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.42.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.43.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.43.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.43.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.44.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.44.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.44.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.45.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.45.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.45.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.46.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.46.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.46.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.47.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.47.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.47.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.48.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.48.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.48.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.49.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.49.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.49.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.50.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.50.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.50.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.51.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.51.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.51.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.52.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.52.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.52.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.53.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.53.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.53.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.54.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.54.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.54.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.55.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.55.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.55.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.56.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.56.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.56.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.57.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.57.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.57.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.58.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.58.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.58.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.59.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.59.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.59.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.60.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.60.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.60.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.61.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.61.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.61.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.62.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.62.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.62.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.63.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.63.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.63.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.64.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.64.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.64.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.65.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.65.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.65.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.66.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.66.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.66.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.67.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.67.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.67.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.68.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.68.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.68.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.69.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.69.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.69.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.70.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.70.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.70.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.71.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.71.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.71.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.72.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.72.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.72.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.73.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.73.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.73.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.74.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.74.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.74.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.75.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.75.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.75.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.76.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.76.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.76.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.77.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.77.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.77.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.78.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.78.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.78.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.79.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.79.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.79.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.80.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.80.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.80.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.81.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.81.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.81.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.82.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.82.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.82.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.83.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.83.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.83.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.84.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.84.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.84.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.85.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.85.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.85.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.86.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.86.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.86.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.87.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.87.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.87.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.88.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.88.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.88.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.89.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.89.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.89.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.90.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.90.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.90.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.91.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.91.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.91.ffn_up_exps.weight buffer type overriden to CPU
model has unused tensor blk.92.attn_norm.weight (size = 20480 bytes) -- ignoring
model has unused tensor blk.92.attn_q.weight (size = 66846720 bytes) -- ignoring
model has unused tensor blk.92.attn_k.weight (size = 5570560 bytes) -- ignoring
model has unused tensor blk.92.attn_v.weight (size = 5570560 bytes) -- ignoring
model has unused tensor blk.92.attn_q.bias (size = 49152 bytes) -- ignoring
model has unused tensor blk.92.attn_k.bias (size = 4096 bytes) -- ignoring
model has unused tensor blk.92.attn_v.bias (size = 4096 bytes) -- ignoring
model has unused tensor blk.92.attn_output.weight (size = 66846720 bytes) -- ignoring
model has unused tensor blk.92.attn_q_norm.weight (size = 512 bytes) -- ignoring
model has unused tensor blk.92.attn_k_norm.weight (size = 512 bytes) -- ignoring
model has unused tensor blk.92.post_attention_norm.weight (size = 20480 bytes) -- ignoring
model has unused tensor blk.92.ffn_gate_inp.weight (size = 3276800 bytes) -- ignoring
model has unused tensor blk.92.exp_probs_b.bias (size = 640 bytes) -- ignoring
Tensor blk.92.ffn_gate_exps.weight buffer type overriden to CPU
model has unused tensor blk.92.ffn_gate_exps.weight (size = 865075200 bytes) -- ignoring
Tensor blk.92.ffn_down_exps.weight buffer type overriden to CPU
model has unused tensor blk.92.ffn_down_exps.weight (size = 1042022400 bytes) -- ignoring
Tensor blk.92.ffn_up_exps.weight buffer type overriden to CPU
model has unused tensor blk.92.ffn_up_exps.weight (size = 865075200 bytes) -- ignoring
model has unused tensor blk.92.ffn_gate_shexp.weight (size = 8355840 bytes) -- ignoring
model has unused tensor blk.92.ffn_down_shexp.weight (size = 8355840 bytes) -- ignoring
model has unused tensor blk.92.ffn_up_shexp.weight (size = 8355840 bytes) -- ignoring
model has unused tensor blk.92.nextn.eh_proj.weight (size = 55705600 bytes) -- ignoring
model has unused tensor blk.92.nextn.enorm.weight (size = 20480 bytes) -- ignoring
model has unused tensor blk.92.nextn.hnorm.weight (size = 20480 bytes) -- ignoring
model has unused tensor blk.92.nextn.shared_head_norm.weight (size = 20480 bytes) -- ignoring
llm_load_tensors: offloading 93 repeating layers to GPU
llm_load_tensors: offloading non-repeating layers to GPU
llm_load_tensors: offloaded 94/94 layers to GPU
llm_load_tensors:        CPU buffer size = 235293.75 MiB
llm_load_tensors:  CUDA_Host buffer size =   612.81 MiB
llm_load_tensors:      CUDA0 buffer size =  4954.45 MiB
llm_load_tensors:      CUDA1 buffer size =  3139.78 MiB
llm_load_tensors:      CUDA2 buffer size =  3139.78 MiB
llm_load_tensors:      CUDA3 buffer size =  3139.78 MiB
llm_load_tensors:      CUDA4 buffer size =  1934.84 MiB
....................................................................................................
llama_new_context_with_model: n_ctx      = 16128
llama_new_context_with_model: n_batch    = 4096
llama_new_context_with_model: n_ubatch   = 4096
llama_new_context_with_model: flash_attn = 1
llama_new_context_with_model: mla_attn   = 0
llama_new_context_with_model: attn_max_b = 0
llama_new_context_with_model: fused_moe  = 1
llama_new_context_with_model: fused_up_gate = 1
llama_new_context_with_model: ser        = -1, 0
llama_new_context_with_model: freq_base  = 1000000.0
llama_new_context_with_model: freq_scale = 1
llama_kv_cache_init:      CUDA0 KV buffer size =   903.67 MiB
llama_kv_cache_init:      CUDA1 KV buffer size =   635.92 MiB
llama_kv_cache_init:      CUDA2 KV buffer size =   635.92 MiB
llama_kv_cache_init:      CUDA3 KV buffer size =   635.92 MiB
llama_kv_cache_init:      CUDA4 KV buffer size =   301.22 MiB
llama_new_context_with_model: KV self size  = 3112.59 MiB, K (q8_0): 1556.30 MiB, V (q8_0): 1556.30 MiB
llama_new_context_with_model:  CUDA_Host  output buffer size =     1.16 MiB
llama_new_context_with_model: pipeline parallelism enabled (n_copies=4)
ggml_backend_cuda_buffer_type_alloc_buffer: allocating 965110.94 MiB on device 0: cudaMalloc failed: out of memory
ggml_gallocr_reserve_n: failed to allocate CUDA0 buffer of size 1011992164352
llama_new_context_with_model: failed to allocate compute buffers
llama_init_from_gpt_params: error: failed to create context with model '/home/server/Downloads/GLM-4.6-IQ5_K-00001-of-00006.gguf'
 ERR [              load_model] unable to load model | tid="125885501370368" timestamp=1759527703 model="/home/server/Downloads/GLM-4.6-IQ5_K-00001-of-00006.gguf"
Segmentation fault (core dumped)

---

## 💬 Discussion

👤 **magikRUKKOLA** commented on **2025-10-03** at **23:23:16**

```bash
/opt/ik_llama.cpp/ik_llama.cpp/build/bin/llama-server \
    --model /mnt/usb/opt/ubergarm/GLM-4.6/IQ5_K/GLM-4.6-IQ5_K-00001-of-00006.gguf \
    --alias ubergarm/GLM-4.6_IQ5_K \
    --ctx-size $((150 * 1024)) \
    -b $((16 * 512)) -ub $((8 * 512)) \
    --mlock \
    --temp 0.5 --top-k 0 --top-p 1.0 --min-p 0.1 --repeat-penalty 1.0 \
    -ctk q8_0 \
    -fa \
    -fmoe \
    -amb 512 \
    --split-mode layer \
    --tensor-split 37,44,44 \
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

Three RTX 3090.  150k ctx.  No issues.

Given that you're having:

```
ggml_backend_cuda_buffer_type_alloc_buffer: allocating 965110.94 MiB on device 0: cudaMalloc failed: out of memory
```

I believe your GGUF files are damaged or something.  Check the sha256 hashes.  Below is the script you can use.  Make sure to create the files.txt with the links to the download each of the GGUF files.

```bash
#!/usr/bin/env bash

# Color codes for pretty output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m' # No Color

# Default input file
INPUT_FILE=${1:-files.txt}

# Check if input file exists
if [[ ! -f "$INPUT_FILE" ]]; then
    echo -e "${RED}Error: Input file '$INPUT_FILE' not found${NC}"
    exit 1
fi

echo -e "${BLUE}=== GGUF File Integrity Checker ===${NC}"
echo -e "${YELLOW}Reading URLs from: $INPUT_FILE${NC}"
echo ""

# Function to test range support
test_range_support() {
    local url=$1
    local test_range="bytes=0-1" # Test with a small 2-byte range
    
    # Make HEAD request to check Accept-Ranges header
    if curl -I -v -H "Range: $test_range" "$url" | grep -iq "Accept-Ranges: bytes"; then
        # Make actual range request to test
        if curl -f -v -r "$test_range" "$url" >/dev/null 2>&1; then
            return 1
            return 0 # Range requests supported
        fi
    fi
    return 1 # Range requests not supported
}

# Process each URL in the input file
while IFS= read -r url || [[ -n "$url" ]]; do
    # Skip empty lines
    [[ -z "$url" ]] && continue
    
    echo -e "${BLUE}Processing URL:${NC} $url"
    
    # Transform URL to get the raw info link
    info_url="${url/resolve\/main/raw\/main}"
    info_url="${info_url/\?download=true/}"
    
    # Extract filename from URL
    filename=$(basename "$url" | cut -d'?' -f1)
    
    # Get the reference info
    echo -e "  ${YELLOW}Fetching reference info...${NC}"
    if ! ref_info=$(curl -v "$info_url"); then
        echo -e "  ${RED}Error: Failed to fetch reference info for $filename${NC}"
        continue
    fi
    
    # Extract reference size and sha256
    ref_size=$(echo "$ref_info" | grep -oP 'size \K\d+')
    ref_sha256=$(echo "$ref_info" | grep -oP 'oid sha256:\K[0-9a-f]+')
    
    if [[ -z "$ref_size" || -z "$ref_sha256" ]]; then
        echo -e "  ${RED}Error: Could not extract reference size or sha256${NC}"
        continue
    fi
    
    echo -e "  ${YELLOW}Reference info:${NC}"
    echo -e "    Size:   $ref_size bytes"
    echo -e "    SHA256: $ref_sha256"
    
    # Check if local file exists
    if [[ ! -f "$filename" ]]; then
        echo -e "  ${RED}File $filename not found locally${NC}"
        continue
    fi
    
    # Get local file size
    local_size=$(stat -c%s "$filename")
    
    # Check size match
    if [[ "$local_size" -ne "$ref_size" ]]; then
        echo -e "  ${RED}Size mismatch!${NC}"
        echo -e "    Local size: $local_size bytes"
        echo -e "    Expected:   $ref_size bytes"
        
        # Calculate missing bytes
        missing_bytes=$((ref_size - local_size))
        echo -e "  ${YELLOW}Missing bytes: $missing_bytes${NC}"
        
        # Test if server supports range requests
        echo -e "  ${YELLOW}Testing range request support...${NC}"
        if test_range_support "$url"; then
            echo -e "  ${GREEN}Server supports range requests${NC}"
            
            # Try three different approaches to get the missing data
            success=0
            for attempt in {1..3}; do
                patch_file="${filename}.patch"
                start_range=$local_size
                end_range=$((ref_size - 1))
                
                case $attempt in
                    1)
                        # Try exact range first
                        echo -e "  ${YELLOW}Attempt $attempt: Downloading exact missing range ($start_range-$end_range)...${NC}"
                        ;;
                    2)
                        # Try last 16MB if exact range failed
                        echo -e "  ${YELLOW}Attempt $attempt: Downloading last 16MB (${BLUE}${ref_size}${YELLOW} - 16777216)...${NC}"
                        start_range=$((ref_size - 16777216))
                        ;;
                    3)
                        # Try complete redownload if all else fails
                        echo -e "  ${YELLOW}Attempt $attempt: Downloading complete file...${NC}"
                        start_range=0
                        ;;
                esac
                
                if curl -f -L -v -r "$start_range-$end_range" -o "$patch_file" "$url"; then
                    # Verify patch size
                    patch_size=$(stat -c%s "$patch_file")
                    expected_size=$((end_range - start_range + 1))
                    
                    if [[ "$patch_size" -eq "$expected_size" ]]; then
                        echo -e "  ${GREEN}Downloaded patch size matches expected ($patch_size bytes)${NC}"
                        
                        # Ask user if they want to apply the patch
                        #read -p "  Do you want to apply this patch to the existing file? [y/N] " -n 1 -r
                        REPLY="Y"
                        echo
                        if [[ $REPLY =~ ^[Yy]$ ]]; then
                            if ! cat "$patch_file" >> "$filename"; then
                                echo -e "  ${RED}Error: Failed to append patch to file${NC}"
                                rm -f "$patch_file"
                                continue
                            fi
                            echo -e "  ${GREEN}Patch applied successfully!${NC}"
                            rm -f "$patch_file"
                            
                            # Verify new size
                            new_size=$(stat -c%s "$filename")
                            if [[ "$new_size" -eq "$ref_size" ]]; then
                                echo -e "  ${GREEN}File size now matches!${NC}"
                                success=1
                                break
                            else
                                echo -e "  ${RED}Error: File size still doesn't match after patching${NC}"
                            fi
                        else
                            echo -e "  ${YELLOW}Patch not applied${NC}"
                            rm -f "$patch_file"
                        fi
                    else
                        echo -e "  ${RED}Warning: Downloaded patch size ($patch_size) doesn't match expected ($expected_size)${NC}"
                        rm -f "$patch_file"
                    fi
                else
                    echo -e "  ${RED}Error: Failed to download missing data${NC}"
                    rm -f "$patch_file"
                fi
            done
            
            if [[ $success -eq 0 ]]; then
                echo -e "  ${RED}All attempts to download missing data failed${NC}"
            fi
        else
            echo -e "  ${RED}Server does not support range requests${NC}"
            echo -e "  ${YELLOW}You'll need to download the full file again${NC}"
        fi
        continue
    fi
    
    echo -e "  ${GREEN}Size matches!${NC} Computing SHA256 checksum..."
    
    # Compute local sha256
    local_sha256=$(sha256sum "$filename" | cut -d' ' -f1)
    
    # Check sha256 match
    if [[ "$local_sha256" == "$ref_sha256" ]]; then
        echo -e "  ${GREEN}SHA256 matches!${NC} File integrity verified."
    else
        echo -e "  ${RED}SHA256 mismatch!${NC}"
        echo -e "    Local SHA256:  $local_sha256"
        echo -e "    Expected SHA256: $ref_sha256"
        
        # If SHA256 doesn't match but size does, try patching the last 16MB
        if test_range_support "$url"; then
            echo -e "  ${YELLOW}Attempting to fix by downloading last 16MB...${NC}"
            
            patch_file="${filename}.patch"
            start_range=$((ref_size - 16777216))
            end_range=$((ref_size - 1))
            
            if curl -f -L -v -r "$start_range-$end_range" -o "$patch_file" "$url"; then
                # Verify patch size
                patch_size=$(stat -c%s "$patch_file")
                expected_size=$((end_range - start_range + 1))
                
                if [[ "$patch_size" -eq "$expected_size" ]]; then
                    echo -e "  ${GREEN}Downloaded patch size matches expected ($patch_size bytes)${NC}"
                    
                    # Create backup of original file
                    backup_file="${filename}.bak"
                    cp "$filename" "$backup_file"
                    
                    # Replace last 16MB of the file
                    truncate -s $start_range "$filename"
                    if ! cat "$patch_file" >> "$filename"; then
                        echo -e "  ${RED}Error: Failed to append patch to file${NC}"
                        # Restore from backup
                        mv "$backup_file" "$filename"
                        rm -f "$patch_file"
                        continue
                    fi
                    
                    echo -e "  ${GREEN}Patch applied successfully!${NC}"
                    rm -f "$patch_file" "$backup_file"
                    
                    # Verify new checksum
                    new_sha256=$(sha256sum "$filename" | cut -d' ' -f1)
                    if [[ "$new_sha256" == "$ref_sha256" ]]; then
                        echo -e "  ${GREEN}SHA256 now matches! File integrity verified.${NC}"
                    else
                        echo -e "  ${RED}Error: SHA256 still doesn't match after patching${NC}"
                        echo -e "    New SHA256:  $new_sha256"
                    fi
                else
                    echo -e "  ${RED}Warning: Downloaded patch size ($patch_size) doesn't match expected ($expected_size)${NC}"
                    rm -f "$patch_file"
                fi
            else
                echo -e "  ${RED}Error: Failed to download patch${NC}"
            fi
        else
            echo -e "  ${RED}Server does not support range requests - cannot attempt patching${NC}"
        fi
    fi
    
    echo ""
done < "$INPUT_FILE"

echo -e "${BLUE}=== Verification complete ===${NC}"
```

> 👤 **RedDragonGecko** replied on **2025-10-04** at **01:14:55**
> 
> All files integrity verified.

---

👤 **ikawrakow** commented on **2025-10-04** at **04:56:59**

Try removing --parallel 1. Also, what were the cmake build commands?

---

👤 **RedDragonGecko** commented on **2025-10-05** at **01:41:06**

removing --parallel 1; the result is the same.
Build commands used;
cmake -B ./build -DGGML_CUDA=ON -DGGML_BLAS=OFF
cmake --build ./build --config Release -j $(nproc)

./build/bin/llama-server --version
version: 3901 (6051ba25)
built with cc (Ubuntu 14.2.0-19ubuntu2) 14.2.0 for x86_64-linux-gnu

nvcc --version
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2023 NVIDIA Corporation
Built on Tue_Aug_15_22:02:13_PDT_2023
Cuda compilation tools, release 12.2, V12.2.140
Build cuda_12.2.r12.2/compiler.33191640_0

actually, looking closer the amount it tried to allocate decreased a bit;
ggml_backend_cuda_buffer_type_alloc_buffer: allocating 913480.94 MiB on device 0: cudaMalloc failed: out of memory

> 👤 **magikRUKKOLA** replied on **2025-10-05** at **13:37:16**
> 
> Have you tried to upgrade cuda or something?

> 👤 **RedDragonGecko** replied on **2025-10-05** at **17:19:46**
> 
> Is there a recommend version for Ik_llama.cpp I missed somewhere?

> 👤 **magikRUKKOLA** replied on **2025-10-05** at **20:13:59**
> 
> @RedDragonGecko 
> > Is there a recommend version for Ik_llama.cpp I missed somewhere?
> 
> I don't think so.
> But at least I can tell that cuda 13 works great.

---

👤 **ikawrakow** commented on **2025-10-07** at **07:43:09**

This crazy VRAM allocation is happening with a fresh build folder? 

Try rebuilding in a new folder adding `-DGGML_SCHED_MAX_COPIES=1` to the `cmake` command line. (this shouldn't normally be required, but the only known crazy VRAM allocation issue is when `GGML_SCHED_MAX_COPIES` somehow ends up being greater than 1).

> 👤 **RedDragonGecko** replied on **2025-10-07** at **20:29:34**
> 
> Yes, this seems to have fixed my problem. Thank you all for your time.

> 👤 **RedDragonGecko** replied on **2025-10-07** at **21:56:58**
> 
> When I  use -ot ".ffn_(up|down|gate)_exps.=CPU at let the remaining use about 29 of vram split across two of my 3090's I get about 60% of the speed I am seeing using llama.cpp. That is when I cram as much as I can of the gate unto 4 3090's an 1 3060 12g.
> However, when I do the same with ik_llama.cpp the speed drops to 0.3t/s
> 
> Not sure if its something I'm doing wrong or if ik_llama.cpp is more cpu focused. I'll play around with it some more with different models.

> 👤 **ikawrakow** replied on **2025-10-08** at **05:11:34**
> 
> >  I get about 60% of the speed I am seeing using llama.cpp.
> 
> 60% of prompt processing speed, 60% of token generation speed, or both?
> 
> > That is when I cram as much as I can of the gate unto 4 3090's an 1 3060 12g. However, when I do the same with ik_llama.cpp the speed drops to 0.3t/s
> 
> In `ik_llama.cpp` you absolutely do not want to have `ffn_up_exps` and `ffn_gate_exps` on different devices (be it VRAM and RAM, or VRAM on two different GPUs). If you post your tensor overrides perhaps we can help to improve them.

> 👤 **RedDragonGecko** replied on **2025-10-08** at **22:55:16**
> 
> ## ik_llama.cpp  model size       = 249.099 GiB (5.997 BPW)
> ./build/bin/llama-server -m '/media/server/NVME Raid 1/GLM-4.6-IQ5_K-00001-of-00006.gguf' -c 32768 --no-warmup -ngl 999 -fa -ctk q8_0 -ctv q8_0 -ts 50,50 --jinja -t 24 --threads-batch 32 --alias GLM-4.6-IQ5K-IK --no-mmap -b 4096 -ub 4096 --parallel 1 -fmoe -ot ".ffn_(up|down|gate)_exps.=CPU"
> 
> ### prompt eval time     =   69520.45 ms / 10864 tokens (    6.40 ms per token,   156.27 tokens per second) ###
> ### generation eval time =  420711.94 ms /  2000 runs   (  210.36 ms per token,     4.75 tokens per second) ###
> 
> With [ -ts 17,12,12.5,12,7 -ot ".ffn_(up|down)_exps.=CPU" -ot "blk.(3|4|5|6|7|8|9|10).ffn_gate_exps.=CPU" ]
> Unusable speeds.
> 
> --------
> 
> ## llama.cpp  file size   = 234.74 GiB (5.65 BPW)
> ./build/bin/llama-server -m '/home/server/Downloads/GLM-4.6-UD-Q5_K_XL-00001-of-00006.gguf' -c 32768 --no-warmup -ngl 999 -fa on -ctk q8_0 -ctv q8_0 -ts 50,50 --jinja -t 24 --threads-batch 32 --alias GLM-4.6-UD-Q5_KXL --no-mmap -b 4096 -ub 4096 --parallel 1 -ot ".ffn_(up|down|gate)_exps.=CPU"
> 
> ### prompt eval time =   46887.17 ms / 10864 tokens (    4.32 ms per token,   231.71 tokens per second) ###
> ###               eval time =  399485.27 ms /  2000 tokens (  199.74 ms per token,     5.01 tokens per second) ###
> 
> with [ -ts 16,12.5,12.5,12.5,5 -ot ".ffn_(up|down)_exps.=CPU" -ot "blk.(3|4|5|6|7|8|9|10).ffn_gate_exps.=CPU"]
> 
> ### prompt eval time =   44457.57 ms / 10864 tokens (    4.09 ms per token,   244.37 tokens per second) ###
> ###                eval time =  310982.29 ms /  2000 tokens (  155.49 ms per token,     6.43 tokens per second) ###
> 
> Hardware, Amd Epyc 7402p 24 core 48 thread, 512gb 2400 ddr4 8 channel, 3090x4 3060 12gx1, operator competence level- just enough to be dangerous.

> 👤 **ikawrakow** replied on **2025-10-09** at **05:59:35**
> 
> Try this:
> * Add `-ooae` to your command line. This triggers offloading only the active experts, which is what `llama.cpp` does. This may give you a better PP speed. Offloading only active experts to the GPU is the default behavior in `llama.cpp`. 
> * In the case of loading some of the experts tensors onto GPUs, try
> ```
> -ot ".ffn_(up|gate)_exps.=CPU" -ot "blk.(3|4|5|6|7|8|9|10).ffn_down_exps.=CPU"
> ```
> I'm not sure if this is the best strategy (most people use tensor overrides that put specific layers of experts on specific GPUs), but it should be much better compared to what you used. The `-fmoe` flag in `ik_llama.cpp` fuses the `ffn_up` + `ffn_gate` matrix multiplications and the `SILU` op with the result into a single op. But with the overrides you are using, `ffn_gate_exps` for most layers is on a GPU. Hence, for token generation the `ffn_gate_exps` tensors need to be copied from VRAM to RAM, which is very slow. With the above override `ffn_up_exps` and `ffn_gate_exps` will be both on the same device, so the fused op can be run without needing to copy the tensors.  
> 
> The performance comparison you are running is a bit of apples to oranges. The `IQ5_K` quantization type will generally be of better quality compared to `Q5_K`, but it will be somewhat slower. I also see that ubergarm has used `Q8_0` for all attention tensors (vs `Q5_K/Q6_K` in Unsloth's model), which will result in a higher size for the active model weights, which will lead to a lower TG performance (which is memory bound). For a real performance comparison you should run Unsloth's `Q5_K_XL` model with `ik_llama.cpp` as well, and only then decide if the better quantization quality of the `IQ5_K` model is worth the performance hit.
> 
> As a side note, performance with `Q8_0` KV cache and attention computed on the GPU is generally slower than `fp16`, so you may be better off using `fp16` KV cache (and loading fewer experts layers onto the GPUs to offset the higher VRAM usage with `fp16` KV cache).

> 👤 **magikRUKKOLA** replied on **2025-10-09** at **13:32:44**
> 
> @RedDragonGecko 
> 
> > ## ik_llama.cpp model size = 249.099 GiB (5.997 BPW)
> > 
> > ./build/bin/llama-server -m '/media/server/NVME Raid 1/GLM-4.6-IQ5_K-00001-of-00006.gguf' -c 32768 --no-warmup -ngl 999 -fa -ctk q8_0 -ctv q8_0 -ts 50,50 --jinja -t 24 --threads-batch 32 --alias GLM-4.6-IQ5K-IK --no-mmap -b 4096 -ub 4096 --parallel 1 -fmoe -ot ".ffn_(up|down|gate)_exps.=CPU"
> > ### prompt eval time = 69520.45 ms / 10864 tokens ( 6.40 ms per token, 156.27 tokens per second)
> 
> [EDIT]:  Ah.  You're using the 4k batches.  That's why.
> 
> Wow this is really weird.  I am getting around 300 tps with two RTX 3090 (and 12C CPU) at around the same context you are testing (10k) .
> https://github.com/ikawrakow/ik_llama.cpp/discussions/715#discussioncomment-14611362
> 
> With the similar quant from THIREUS I am getting about 10% more speed.  I never thought of using llama.cpp mainline because it just doesn't support much quants.
> 
> ```
> export MALLOC_CONF="background_thread:true,percpu_arena:phycpu,metadata_thp:auto,dirty_decay_ms:10000,muzzy_decay_ms:60000"
> export LD_PRELOAD=/usr/local/lib/libjemalloc.so
> 
> #    --seed 3407 \
> #    -fmoe \
>       
> ulimit -n 9999
> ulimit -l unlimited
> 
> export CUDA_VISIBLE_DEVICES="0,1"
> #export GGML_CUDA_ENABLE_UNIFIED_MEMORY=1
> 
> # --verbose-prompt
> 
> /opt/ik_llama.cpp/ik_llama.cpp/build/bin/llama-server \
>     --model /opt/THIREUS/GLM-4.6-5.4976bpw/GLM-4.6-THIREUS-BF16-SPECIAL_TENSOR-00001-of-01760.gguf \
>     --alias THIREUS/GLM-4.6-5.4976bpw \
>     --ctx-size $((72 * 1024)) \
>     -b $((16 * 512)) -ub $((16 * 512)) \
>     --mlock \
>     --temp 0.5 --top-k 0 --top-p 1.0 --min-p 0.1 --repeat-penalty 1.1 \
>     -ctk q8_0 \
>     -fa \
>     -fmoe \
>     -amb 512 \
>     --split-mode layer \
>     --tensor-split 18,20 \
>     --main-gpu 1 \
>     --override-tensor exps=CPU \
>     --n-gpu-layers 99 \
>     --threads $(grep ^cpu\\scores /proc/cpuinfo | uniq | awk '{print $4}' | xargs -I{} echo "{}-0" | bc) \
>     --host 0.0.0.0 \
>     --port 8080 \
>     --log-enable \
>     --logdir /var/log/ \
>     --jinja \
>     --chat-template-file /opt/THIREUS/GLM-4.6-5.4976bpw/chat_template2.jinja \
>     --special \
>     --verbosity 1 \
>     --verbose-prompt \
>     --reasoning-format auto \
>     --prompt-cache "$HOME/.cache/ik_llama.cpp/prompt-cache.bin" --prompt-cache-all \
>     --slot-save-path "$HOME/.cache/ik_llama.cpp/slot.bin" \
>     --lookup-cache-dynamic "$HOME/.cache/ik_llama.cpp/slot.bin" \
>     --keep -1 \
>     --slot-prompt-similarity 0.35 \
>     --metrics
> ```

> 👤 **magikRUKKOLA** replied on **2025-10-09** at **13:42:24**
> 
> @RedDragonGecko 
> 
> > ./build/bin/llama-server -m '/home/server/Downloads/GLM-4.6-UD-Q5_K_XL-00001-of-00006.gguf' -c 32768 --no-warmup -ngl 999 -fa on -ctk q8_0 -ctv q8_0 -ts 50,50 --jinja -t 24 --threads-batch 32 --alias GLM-4.6-UD-Q5_KXL --no-mmap -b 4096 -ub 4096 --parallel 1 -ot ".ffn_(up|down|gate)_exps.=CPU"
> 
> Can you do a perplexity for that quant?  I've got the perplexity for IQ5_K and THIREUS/GLM-4.6-5.4976bpw and was wondering what's up with unsloth.  Usually, it performs much worse.  So yeah, your comparison is more like apples to oranges.

---

👤 **RedDragonGecko** commented on **2025-10-10** at **00:17:23**

ik_llama.cpp - - IQ5_K using -c 32768 --no-warmup -ngl 999 -fa -ctk q8_0 -ctv q8_0 -ts 50,50 --jinja -t 24 --threads-batch 32 --alias GLM-4.6-IQ5K-IK --no-mmap -b 4096 -ub 4096 --parallel 1 -fmoe -ooae -ot ".ffn_(up|gate)_exps.=CPU" -ot "blk.(3|4|5|6|7|8|9|10|11|12|13|14|15|16|17|18|19).ffn_down_exps.=CPU" -ts 20.50,11.75,11.75,11.75,5.5 (This fills the vram of my 4x3090, 3060_12g)

prompt eval time     =   62751.52 ms / 10864 tokens (    5.78 ms per token,   173.13 tokens per second)
generation eval time =  359663.83 ms /  1946 runs   (  184.82 ms per token,     5.41 tokens per second)
--------
Ik_llama.cpp - - UD_Q5_K_XL, exact same settings as above (didn't quite fill my vram, could load more if I fiddled)

prompt eval time     =   61631.81 ms / 10864 tokens (    5.67 ms per token,   176.27 tokens per second)
generation eval time =  303086.29 ms /  1774 runs   (  170.85 ms per token,     5.85 tokens per second)

---

👤 **RedDragonGecko** commented on **2025-10-10** at **00:18:40**

Perplexity of UD_Q5_K_XL using ik_llama.cpp

perplexity: 220.70 seconds per pass - ETA 29.42 minutes
[1]4.0626,[2]3.0639,[3]3.2953,[4]3.2905,[5]2.9277,[6]3.2110,[7]3.1151,[8]3.2022,
Final estimate: PPL = 3.2022 +/- 0.01847

llama_print_timings:        load time =  375469.35 ms
llama_print_timings:      sample time =       0.00 ms /     1 runs   (    0.00 ms per token,      inf tokens per second)
llama_print_timings: prompt eval time = 1702994.10 ms / 262144 tokens (    6.50 ms per token,   153.93 tokens per second)
llama_print_timings:        eval time =       0.00 ms /     1 runs   (    0.00 ms per token,      inf tokens per second)
llama_print_timings:       total time = 1973518.09 ms / 262145 tokens

> 👤 **magikRUKKOLA** replied on **2025-10-10** at **05:45:26**
> 
> @RedDragonGecko 
> 
> > Perplexity of UD_Q5_K_XL using ik_llama.cpp
> > 
> > perplexity: 220.70 seconds per pass - ETA 29.42 minutes [1]4.0626,[2]3.0639,[3]3.2953,[4]3.2905,[5]2.9277,[6]3.2110,[7]3.1151,[8]3.2022, Final estimate: PPL = 3.2022 +/- 0.01847
> > 
> > llama_print_timings: load time = 375469.35 ms llama_print_timings: sample time = 0.00 ms / 1 runs ( 0.00 ms per token, inf tokens per second) llama_print_timings: prompt eval time = 1702994.10 ms / 262144 tokens ( 6.50 ms per token, 153.93 tokens per second) llama_print_timings: eval time = 0.00 ms / 1 runs ( 0.00 ms per token, inf tokens per second) llama_print_timings: total time = 1973518.09 ms / 262145 tokens
> 
> Unfortunately this score doesn't make sense since the baseline is 3.4454.  Your result is 3.2022?  Doesn't make sense.
> 
> Please make sure that the file you're testing is this one: https://github.com/Thireus/GGUF-Tool-Suite/blob/main/wiki.test.raw
> And also make sure that ctx=512.

> 👤 **RedDragonGecko** replied on **2025-10-10** at **21:36:35**
> 
> Sorry about that.
> 
> Final estimate: PPL = 3.4807 +/- 0.02024
> 
> llama_print_timings:        load time =  181845.90 ms
> llama_print_timings:      sample time =       0.00 ms /     1 runs   (    0.00 ms per token,      inf tokens per second)
> llama_print_timings: prompt eval time = 1950286.18 ms / 289280 tokens (    6.74 ms per token,   148.33 tokens per second)
> llama_print_timings:        eval time =       0.00 ms /     1 runs   (    0.00 ms per token,      inf tokens per second)
> llama_print_timings:       total time = 2110103.57 ms / 289281 tokens

> 👤 **magikRUKKOLA** replied on **2025-10-10** at **22:07:31**
> 
> @RedDragonGecko 
> 
> > 3.4807
> 
> You do see a problem here, right?  The unsloth quants are not that efficient you see:
> 
> ![glm46-ppl-log](https://github.com/user-attachments/assets/469d4b6b-ab05-479a-84e9-e324633bdb5f)

> 👤 **RedDragonGecko** replied on **2025-10-10** at **22:25:41**
> 
> Hmm, interesting. But that also shows the IQ5_K beating Q8 and BF16 if I'm understanding this correctly.

> 👤 **magikRUKKOLA** replied on **2025-10-10** at **22:32:57**
> 
> @RedDragonGecko 
> > Hmm, interesting. But that also shows the IQ5_K beating Q8 and BF16 if I'm understanding this correctly.
> 
> Yeah, that's funny.  Its better to think of 0.00xx differences as of noise though.