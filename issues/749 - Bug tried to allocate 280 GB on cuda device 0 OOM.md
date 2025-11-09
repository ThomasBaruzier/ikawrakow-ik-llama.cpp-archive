## 📌 [Issue #749](https://github.com/ikawrakow/ik_llama.cpp/issues/749) - Bug: tried to allocate 280 GB on cuda device 0 (OOM)

| **Author** | `ghost` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-09-01 |
| **Updated** | 2025-09-02 |

---

## 📄 Description

### What happened?

```
llama_kv_cache_init:      CUDA0 KV buffer size =   379.23 MiB
llama_kv_cache_init:      CUDA1 KV buffer size =    82.44 MiB
llama_kv_cache_init:      CUDA2 KV buffer size =    65.95 MiB
llama_kv_cache_init:      CUDA3 KV buffer size =    82.44 MiB
llama_kv_cache_init:      CUDA4 KV buffer size =    82.44 MiB
llama_kv_cache_init:      CUDA5 KV buffer size =    65.95 MiB
llama_kv_cache_init:      CUDA6 KV buffer size =    82.44 MiB
llama_kv_cache_init:      CUDA7 KV buffer size =    82.44 MiB
llama_kv_cache_init:      CUDA8 KV buffer size =    32.98 MiB
llama_kv_cache_init:      CUDA9 KV buffer size =    32.98 MiB
llama_kv_cache_init:     CUDA10 KV buffer size =    16.49 MiB
llama_new_context_with_model: KV self size  = 1005.79 MiB, c^KV (f16): 1005.79 MiB, kv^T (f16):    0.00 MiB
llama_new_context_with_model:  CUDA_Host  output buffer size =     0.99 MiB
llama_new_context_with_model: pipeline parallelism enabled (n_copies=4)
ggml_backend_cuda_buffer_type_alloc_buffer: allocating 279356.51 MiB on device 0: cudaMalloc failed: out of memory
ggml_gallocr_reserve_n: failed to allocate CUDA0 buffer of size 292926534144
llama_new_context_with_model: failed to allocate compute buffers
llama_init_from_gpt_params: error: failed to create context with model '/<...>/DeepSeek-V3.1-GGUF/UD-Q2_K_XL/DeepSeek-V3.1-UD-Q2_K_XL-00001-of-00006.gguf'
```


### Name and Version

commit d10d90ae272fbf5ca29b040057d162e41b94c87c

`cmake -B build -DGGML_CUDA=ON -DCMAKE_CUDA_ARCHITECTURES="61;75;86" -DCMAKE_BUILD_TYPE=Release -DLLAMA_CURL=OFF -DGGML_BLAS=OFF`

`cmake --build build --config Release -j 8 --target llama-server`

`./llama-server -m "/<...>/DeepSeek-V3.1-GGUF/UD-Q2_K_XL/DeepSeek-V3.1-UD-Q2_K_XL-00001-of-00006.gguf" -ts 23,5,4,5,5,4,5,5,2,2,2 -sm layer -c 15000 -b 1024 -ub 1024 -ngl 62 -ncmoe 19 -ot 'blk\.(19)\.ffn_(up|down)_exps*=CPU' -ot 'blk\.(60)\.ffn_(up)_exps*=CPU' -t 7 -mla 3 -fmoe -amb 1024 --no-mmap --port 5001`

using these ggufs:
https://huggingface.co/unsloth/DeepSeek-V3.1-GGUF

tried "-mla 2", disabling -fa and -amb, still no luck. The same command works for llama.cpp (ofc without ik_llama-specific args)

### What operating system are you seeing the problem on?

Linux

### Relevant log output

```shell
ggml_cuda_init: GGML_CUDA_FORCE_MMQ:    no
ggml_cuda_init: GGML_CUDA_FORCE_CUBLAS: no
ggml_cuda_init: found 11 CUDA devices:
  Device 0: NVIDIA GeForce RTX 3090, compute capability 8.6, VMM: yes
  Device 1: NVIDIA GeForce RTX 3090, compute capability 8.6, VMM: yes
  Device 2: NVIDIA GeForce RTX 2080 Ti, compute capability 7.5, VMM: yes
  Device 3: NVIDIA GeForce RTX 3090, compute capability 8.6, VMM: yes
  Device 4: NVIDIA GeForce RTX 3090, compute capability 8.6, VMM: yes
  Device 5: NVIDIA GeForce RTX 2080 Ti, compute capability 7.5, VMM: yes
  Device 6: Tesla P40, compute capability 6.1, VMM: yes
  Device 7: Tesla P40, compute capability 6.1, VMM: yes
  Device 8: NVIDIA GeForce RTX 3060, compute capability 8.6, VMM: yes
  Device 9: NVIDIA GeForce RTX 3060, compute capability 8.6, VMM: yes
  Device 10: NVIDIA GeForce RTX 3070 Ti, compute capability 8.6, VMM: yes
INFO [                    main] build info | tid="135466039586816" timestamp=1756751941 build=3871 commit="d10d90ae"
INFO [                    main] system info | tid="135466039586816" timestamp=1756751941 n_threads=7 n_threads_batch=-1 total_threads=8 system_info="AVX = 1 | AVX_VNNI = 0 | AVX2 = 1 | AVX512 = 0 | AVX512_VBMI = 0 | AVX512_VNNI = 0 | AVX512_BF16 = 0 | FMA = 1 | NEON = 0 | SVE = 0 | ARM_FMA = 0 | F16C = 1 | FP16_VA = 0 | WASM_SIMD = 0 | BLAS = 1 | SSE3 = 1 | SSSE3 = 1 | VSX = 0 | MATMUL_INT8 = 0 | LLAMAFILE = 1 | "
llama_model_loader: additional 5 GGUFs metadata loaded.
llama_model_loader: loaded meta data with 63 key-value pairs and 1086 tensors from /<...>/DeepSeek-V3.1-GGUF/UD-Q2_K_XL/DeepSeek-V3.1-UD-Q2_K_XL-00001-of-00006.gguf (version GGUF V3 (latest))
llama_model_loader: Dumping metadata keys/values. Note: KV overrides do not apply in this output.
llama_model_loader: - kv   0:                       general.architecture str              = deepseek2
llama_model_loader: - kv   1:                               general.type str              = model
llama_model_loader: - kv   2:                               general.name str              = Deepseek-V3.1
llama_model_loader: - kv   3:                           general.basename str              = Deepseek-V3.1
llama_model_loader: - kv   4:                       general.quantized_by str              = Unsloth
llama_model_loader: - kv   5:                         general.size_label str              = 256x20B
llama_model_loader: - kv   6:                            general.license str              = mit
llama_model_loader: - kv   7:                           general.repo_url str              = https://huggingface.co/unsloth
llama_model_loader: - kv   8:                   general.base_model.count u32              = 1
llama_model_loader: - kv   9:                  general.base_model.0.name str              = DeepSeek V3.1
llama_model_loader: - kv  10:               general.base_model.0.version str              = V3.1
llama_model_loader: - kv  11:          general.base_model.0.organization str              = Deepseek Ai
llama_model_loader: - kv  12:              general.base_model.0.repo_url str              = https://huggingface.co/deepseek-ai/De...
llama_model_loader: - kv  13:                               general.tags arr[str,1]       = ["unsloth"]
llama_model_loader: - kv  14:                      deepseek2.block_count u32              = 61
llama_model_loader: - kv  15:                   deepseek2.context_length u32              = 163840
llama_model_loader: - kv  16:                 deepseek2.embedding_length u32              = 7168
llama_model_loader: - kv  17:              deepseek2.feed_forward_length u32              = 18432
llama_model_loader: - kv  18:             deepseek2.attention.head_count u32              = 128
llama_model_loader: - kv  19:          deepseek2.attention.head_count_kv u32              = 1
llama_model_loader: - kv  20:                   deepseek2.rope.freq_base f32              = 10000.000000
llama_model_loader: - kv  21: deepseek2.attention.layer_norm_rms_epsilon f32              = 0.000001
llama_model_loader: - kv  22:                deepseek2.expert_used_count u32              = 8
llama_model_loader: - kv  23:        deepseek2.leading_dense_block_count u32              = 3
llama_model_loader: - kv  24:                       deepseek2.vocab_size u32              = 129280
llama_model_loader: - kv  25:            deepseek2.attention.q_lora_rank u32              = 1536
llama_model_loader: - kv  26:           deepseek2.attention.kv_lora_rank u32              = 512
llama_model_loader: - kv  27:             deepseek2.attention.key_length u32              = 576
llama_model_loader: - kv  28:           deepseek2.attention.value_length u32              = 512
llama_model_loader: - kv  29:         deepseek2.attention.key_length_mla u32              = 192
llama_model_loader: - kv  30:       deepseek2.attention.value_length_mla u32              = 128
llama_model_loader: - kv  31:       deepseek2.expert_feed_forward_length u32              = 2048
llama_model_loader: - kv  32:                     deepseek2.expert_count u32              = 256
llama_model_loader: - kv  33:              deepseek2.expert_shared_count u32              = 1
llama_model_loader: - kv  34:             deepseek2.expert_weights_scale f32              = 2.500000
llama_model_loader: - kv  35:              deepseek2.expert_weights_norm bool             = true
llama_model_loader: - kv  36:               deepseek2.expert_gating_func u32              = 2
llama_model_loader: - kv  37:             deepseek2.rope.dimension_count u32              = 64
llama_model_loader: - kv  38:                deepseek2.rope.scaling.type str              = yarn
llama_model_loader: - kv  39:              deepseek2.rope.scaling.factor f32              = 40.000000
llama_model_loader: - kv  40: deepseek2.rope.scaling.original_context_length u32              = 4096
llama_model_loader: - kv  41: deepseek2.rope.scaling.yarn_log_multiplier f32              = 0.100000
llama_model_loader: - kv  42:                       tokenizer.ggml.model str              = gpt2
llama_model_loader: - kv  43:                         tokenizer.ggml.pre str              = deepseek-v3
llama_model_loader: - kv  44:                      tokenizer.ggml.tokens arr[str,129280]  = ["<｜begin▁of▁sentence｜>", "<�...
llama_model_loader: - kv  45:                  tokenizer.ggml.token_type arr[i32,129280]  = [3, 3, 3, 1, 1, 1, 1, 1, 1, 1, 1, 1, ...
llama_model_loader: - kv  46:                      tokenizer.ggml.merges arr[str,127741]  = ["Ġ t", "Ġ a", "i n", "Ġ Ġ", "h e...
llama_model_loader: - kv  47:                tokenizer.ggml.bos_token_id u32              = 0
llama_model_loader: - kv  48:                tokenizer.ggml.eos_token_id u32              = 1
llama_model_loader: - kv  49:            tokenizer.ggml.padding_token_id u32              = 2
llama_model_loader: - kv  50:               tokenizer.ggml.add_bos_token bool             = true
llama_model_loader: - kv  51:               tokenizer.ggml.add_sep_token bool             = false
llama_model_loader: - kv  52:               tokenizer.ggml.add_eos_token bool             = false
llama_model_loader: - kv  53:                    tokenizer.chat_template str              = {#- Unsloth template fixes #}{% if no...
llama_model_loader: - kv  54:               general.quantization_version u32              = 2
llama_model_loader: - kv  55:                          general.file_type u32              = 10
llama_model_loader: - kv  56:                      quantize.imatrix.file str              = DeepSeek-V3.1-GGUF/imatrix_unsloth.gguf
llama_model_loader: - kv  57:                   quantize.imatrix.dataset str              = unsloth_calibration_DeepSeek-V3.1-BF1...
llama_model_loader: - kv  58:             quantize.imatrix.entries_count u32              = 781
llama_model_loader: - kv  59:              quantize.imatrix.chunks_count u32              = 84
llama_model_loader: - kv  60:                                   split.no u16              = 0
llama_model_loader: - kv  61:                        split.tensors.count i32              = 1086
llama_model_loader: - kv  62:                                split.count u16              = 6
llama_model_loader: - type  f32:  361 tensors
llama_model_loader: - type q8_0:  183 tensors
llama_model_loader: - type q2_K:  116 tensors
llama_model_loader: - type q3_K:   42 tensors
llama_model_loader: - type q4_K:  343 tensors
llama_model_loader: - type q5_K:   28 tensors
llama_model_loader: - type q6_K:   13 tensors
================= Adjusted mainline llama.cpp MLA tensors to ik_llama.cpp
init_tokenizer: initializing tokenizer for type 2
load: special_eos_id is not in special_eog_ids - the tokenizer config may be incorrect
load: printing all EOG tokens:
load:   - 1 ('<｜end▁of▁sentence｜>')
load: special tokens cache size = 818
load: token to piece cache size = 0.8223 MB
llm_load_print_meta: format           = GGUF V3 (latest)
llm_load_print_meta: arch             = deepseek2
llm_load_print_meta: n_ctx_train      = 163840
llm_load_print_meta: n_embd           = 7168
llm_load_print_meta: n_layer          = 61
llm_load_print_meta: n_head           = 128
llm_load_print_meta: n_head_kv        = 128
llm_load_print_meta: n_rot            = 64
llm_load_print_meta: n_swa            = 0
llm_load_print_meta: n_swa_pattern    = 1
llm_load_print_meta: n_embd_head_k    = 192
llm_load_print_meta: n_embd_head_v    = 128
llm_load_print_meta: n_gqa            = 1
llm_load_print_meta: n_embd_k_gqa     = 24576
llm_load_print_meta: n_embd_v_gqa     = 16384
llm_load_print_meta: f_norm_eps       = 0.0e+00
llm_load_print_meta: f_norm_rms_eps   = 1.0e-06
llm_load_print_meta: f_clamp_kqv      = 0.0e+00
llm_load_print_meta: f_max_alibi_bias = 0.0e+00
llm_load_print_meta: f_logit_scale    = 0.0e+00
llm_load_print_meta: n_ff             = 18432
llm_load_print_meta: n_expert         = 256
llm_load_print_meta: n_expert_used    = 8
llm_load_print_meta: causal attn      = 1
llm_load_print_meta: pooling type     = 0
llm_load_print_meta: rope type        = 0
llm_load_print_meta: rope scaling     = yarn
llm_load_print_meta: freq_base_train  = 10000.0
llm_load_print_meta: freq_scale_train = 0.025
llm_load_print_meta: n_ctx_orig_yarn  = 4096
llm_load_print_meta: rope_finetuned   = unknown
llm_load_print_meta: ssm_d_conv       = 0
llm_load_print_meta: ssm_d_inner      = 0
llm_load_print_meta: ssm_d_state      = 0
llm_load_print_meta: ssm_dt_rank      = 0
llm_load_print_meta: model type       = 671B
llm_load_print_meta: model ftype      = Q2_K - Medium
llm_load_print_meta: model params     = 671.026 B
llm_load_print_meta: model size       = 238.158 GiB (3.049 BPW) 
llm_load_print_meta: repeating layers = 236.964 GiB (3.042 BPW, 669.173 B parameters)
llm_load_print_meta: general.name     = Deepseek-V3.1
llm_load_print_meta: n_layer_dense_lead   = 3
llm_load_print_meta: n_lora_q             = 1536
llm_load_print_meta: n_lora_kv            = 512
llm_load_print_meta: n_ff_exp             = 2048
llm_load_print_meta: n_expert_shared      = 1
llm_load_print_meta: expert_weights_scale = 2.5
llm_load_print_meta: expert_weights_norm  = 1
llm_load_print_meta: expert_gating_func   = sigmoid
llm_load_print_meta: rope_yarn_log_mul    = 0.1000
print_info: vocab type       = BPE
print_info: n_vocab          = 129280
print_info: n_merges         = 127741
print_info: BOS token        = 0 '<｜begin▁of▁sentence｜>'
print_info: EOS token        = 1 '<｜end▁of▁sentence｜>'
print_info: EOT token        = 1 '<｜end▁of▁sentence｜>'
print_info: PAD token        = 2 '<｜▁pad▁｜>'
print_info: LF token         = 201 'Ċ'
print_info: FIM PRE token    = 128801 '<｜fim▁begin｜>'
print_info: FIM SUF token    = 128800 '<｜fim▁hole｜>'
print_info: FIM MID token    = 128802 '<｜fim▁end｜>'
print_info: EOG token        = 1 '<｜end▁of▁sentence｜>'
print_info: max token length = 256
llm_load_tensors: ggml ctx size =    5.35 MiB
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
Tensor blk.19.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.19.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.60.ffn_up_exps.weight buffer type overriden to CPU
llm_load_tensors: offloading 61 repeating layers to GPU
llm_load_tensors: offloading non-repeating layers to GPU
llm_load_tensors: offloaded 62/62 layers to GPU
llm_load_tensors:        CPU buffer size = 68068.00 MiB
llm_load_tensors:  CUDA_Host buffer size =   497.11 MiB
llm_load_tensors:      CUDA0 buffer size = 17216.38 MiB
llm_load_tensors:      CUDA1 buffer size = 20642.89 MiB
llm_load_tensors:      CUDA2 buffer size = 17087.62 MiB
llm_load_tensors:      CUDA3 buffer size = 20648.25 MiB
llm_load_tensors:      CUDA4 buffer size = 20646.50 MiB
llm_load_tensors:      CUDA5 buffer size = 16613.47 MiB
llm_load_tensors:      CUDA6 buffer size = 20650.11 MiB
llm_load_tensors:      CUDA7 buffer size = 21129.72 MiB
llm_load_tensors:      CUDA8 buffer size =  8066.06 MiB
llm_load_tensors:      CUDA9 buffer size =  8545.67 MiB
llm_load_tensors:     CUDA10 buffer size =  4061.62 MiB
....................................................................................................
============ llm_prepare_mla: need to compute 61 wkv_b tensors
Computed blk.0.attn_kv_b.weight as 512 x 32768 and stored in buffer CUDA0
Computed blk.1.attn_kv_b.weight as 512 x 32768 and stored in buffer CUDA0
Computed blk.2.attn_kv_b.weight as 512 x 32768 and stored in buffer CUDA0
Computed blk.3.attn_kv_b.weight as 512 x 32768 and stored in buffer CUDA0
Computed blk.4.attn_kv_b.weight as 512 x 32768 and stored in buffer CUDA0
Computed blk.5.attn_kv_b.weight as 512 x 32768 and stored in buffer CUDA0
Computed blk.6.attn_kv_b.weight as 512 x 32768 and stored in buffer CUDA0
Computed blk.7.attn_kv_b.weight as 512 x 32768 and stored in buffer CUDA0
Computed blk.8.attn_kv_b.weight as 512 x 32768 and stored in buffer CUDA0
Computed blk.9.attn_kv_b.weight as 512 x 32768 and stored in buffer CUDA0
Computed blk.10.attn_kv_b.weight as 512 x 32768 and stored in buffer CUDA0
Computed blk.11.attn_kv_b.weight as 512 x 32768 and stored in buffer CUDA0
Computed blk.12.attn_kv_b.weight as 512 x 32768 and stored in buffer CUDA0
Computed blk.13.attn_kv_b.weight as 512 x 32768 and stored in buffer CUDA0
Computed blk.14.attn_kv_b.weight as 512 x 32768 and stored in buffer CUDA0
Computed blk.15.attn_kv_b.weight as 512 x 32768 and stored in buffer CUDA0
Computed blk.16.attn_kv_b.weight as 512 x 32768 and stored in buffer CUDA0
Computed blk.17.attn_kv_b.weight as 512 x 32768 and stored in buffer CUDA0
Computed blk.18.attn_kv_b.weight as 512 x 32768 and stored in buffer CUDA0
Computed blk.19.attn_kv_b.weight as 512 x 32768 and stored in buffer CUDA0
Computed blk.20.attn_kv_b.weight as 512 x 32768 and stored in buffer CUDA0
Computed blk.21.attn_kv_b.weight as 512 x 32768 and stored in buffer CUDA0
Computed blk.22.attn_kv_b.weight as 512 x 32768 and stored in buffer CUDA0
Computed blk.23.attn_kv_b.weight as 512 x 32768 and stored in buffer CUDA1
Computed blk.24.attn_kv_b.weight as 512 x 32768 and stored in buffer CUDA1
Computed blk.25.attn_kv_b.weight as 512 x 32768 and stored in buffer CUDA1
Computed blk.26.attn_kv_b.weight as 512 x 32768 and stored in buffer CUDA1
Computed blk.27.attn_kv_b.weight as 512 x 32768 and stored in buffer CUDA1
Computed blk.28.attn_kv_b.weight as 512 x 32768 and stored in buffer CUDA2
Computed blk.29.attn_kv_b.weight as 512 x 32768 and stored in buffer CUDA2
Computed blk.30.attn_kv_b.weight as 512 x 32768 and stored in buffer CUDA2
Computed blk.31.attn_kv_b.weight as 512 x 32768 and stored in buffer CUDA2
Computed blk.32.attn_kv_b.weight as 512 x 32768 and stored in buffer CUDA3
Computed blk.33.attn_kv_b.weight as 512 x 32768 and stored in buffer CUDA3
Computed blk.34.attn_kv_b.weight as 512 x 32768 and stored in buffer CUDA3
Computed blk.35.attn_kv_b.weight as 512 x 32768 and stored in buffer CUDA3
Computed blk.36.attn_kv_b.weight as 512 x 32768 and stored in buffer CUDA3
Computed blk.37.attn_kv_b.weight as 512 x 32768 and stored in buffer CUDA4
Computed blk.38.attn_kv_b.weight as 512 x 32768 and stored in buffer CUDA4
Computed blk.39.attn_kv_b.weight as 512 x 32768 and stored in buffer CUDA4
Computed blk.40.attn_kv_b.weight as 512 x 32768 and stored in buffer CUDA4
Computed blk.41.attn_kv_b.weight as 512 x 32768 and stored in buffer CUDA4
Computed blk.42.attn_kv_b.weight as 512 x 32768 and stored in buffer CUDA5
Computed blk.43.attn_kv_b.weight as 512 x 32768 and stored in buffer CUDA5
Computed blk.44.attn_kv_b.weight as 512 x 32768 and stored in buffer CUDA5
Computed blk.45.attn_kv_b.weight as 512 x 32768 and stored in buffer CUDA5
Computed blk.46.attn_kv_b.weight as 512 x 32768 and stored in buffer CUDA6
Computed blk.47.attn_kv_b.weight as 512 x 32768 and stored in buffer CUDA6
Computed blk.48.attn_kv_b.weight as 512 x 32768 and stored in buffer CUDA6
Computed blk.49.attn_kv_b.weight as 512 x 32768 and stored in buffer CUDA6
Computed blk.50.attn_kv_b.weight as 512 x 32768 and stored in buffer CUDA6
Computed blk.51.attn_kv_b.weight as 512 x 32768 and stored in buffer CUDA7
Computed blk.52.attn_kv_b.weight as 512 x 32768 and stored in buffer CUDA7
Computed blk.53.attn_kv_b.weight as 512 x 32768 and stored in buffer CUDA7
Computed blk.54.attn_kv_b.weight as 512 x 32768 and stored in buffer CUDA7
Computed blk.55.attn_kv_b.weight as 512 x 32768 and stored in buffer CUDA7
Computed blk.56.attn_kv_b.weight as 512 x 32768 and stored in buffer CUDA8
Computed blk.57.attn_kv_b.weight as 512 x 32768 and stored in buffer CUDA8
Computed blk.58.attn_kv_b.weight as 512 x 32768 and stored in buffer CUDA9
Computed blk.59.attn_kv_b.weight as 512 x 32768 and stored in buffer CUDA9
Computed blk.60.attn_kv_b.weight as 512 x 32768 and stored in buffer CUDA10
llama_new_context_with_model: n_ctx      = 15008
llama_new_context_with_model: n_batch    = 1024
llama_new_context_with_model: n_ubatch   = 1024
llama_new_context_with_model: flash_attn = 0
llama_new_context_with_model: mla_attn   = 3
llama_new_context_with_model: attn_max_b = 1024
llama_new_context_with_model: fused_moe  = 1
llama_new_context_with_model: fused_up_gate = 1
llama_new_context_with_model: ser        = -1, 0
llama_new_context_with_model: freq_base  = 10000.0
llama_new_context_with_model: freq_scale = 0.025
llama_kv_cache_init:      CUDA0 KV buffer size =   379.23 MiB
llama_kv_cache_init:      CUDA1 KV buffer size =    82.44 MiB
llama_kv_cache_init:      CUDA2 KV buffer size =    65.95 MiB
llama_kv_cache_init:      CUDA3 KV buffer size =    82.44 MiB
llama_kv_cache_init:      CUDA4 KV buffer size =    82.44 MiB
llama_kv_cache_init:      CUDA5 KV buffer size =    65.95 MiB
llama_kv_cache_init:      CUDA6 KV buffer size =    82.44 MiB
llama_kv_cache_init:      CUDA7 KV buffer size =    82.44 MiB
llama_kv_cache_init:      CUDA8 KV buffer size =    32.98 MiB
llama_kv_cache_init:      CUDA9 KV buffer size =    32.98 MiB
llama_kv_cache_init:     CUDA10 KV buffer size =    16.49 MiB
llama_new_context_with_model: KV self size  = 1005.79 MiB, c^KV (f16): 1005.79 MiB, kv^T (f16):    0.00 MiB
llama_new_context_with_model:  CUDA_Host  output buffer size =     0.99 MiB
llama_new_context_with_model: pipeline parallelism enabled (n_copies=4)
ggml_backend_cuda_buffer_type_alloc_buffer: allocating 279356.51 MiB on device 0: cudaMalloc failed: out of memory
ggml_gallocr_reserve_n: failed to allocate CUDA0 buffer of size 292926534144
llama_new_context_with_model: failed to allocate compute buffers
llama_init_from_gpt_params: error: failed to create context with model '/<...>/DeepSeek-V3.1-GGUF/UD-Q2_K_XL/DeepSeek-V3.1-UD-Q2_K_XL-00001-of-00006.gguf'
 ERR [              load_model] unable to load model | tid="135466039586816" timestamp=1756752409 model="/<...>/DeepSeek-V3.1-GGUF/UD-Q2_K_XL/DeepSeek-V3.1-UD-Q2_K_XL-00001-of-00006.gguf"
```

---

## 💬 Conversation

👤 **juanqui** commented on **2025-09-01** at **20:53:01**

Similar issue here, but entirely different model.

In my case it's trying to allocate `180317.55 MiB`.

```
./ik_llama.cpp/build/bin/llama-server     --model ./ubergarm/GLM-4.5-Air-GGUF/IQ4_KSS/GLM-4.5-Air-IQ4_KSS-00001-of-00002.gguf     --alias glm-4.5-air       --ctx-size 128000     -fa -fmoe     -ctk q4_0 -ctv q4_0     -ub 4096 -b 4096     -ngl 99     --parallel 1     --threads 30     --host 0.0.0.0     --port 3001     --chat-template-file glm_4_5_chat_template.jinja     --jinja     --no-mmap -ot ".ffn_(up|down)_exps.=CPU"ggml_cuda_init: GGML_CUDA_FORCE_MMQ:    no
ggml_cuda_init: GGML_CUDA_FORCE_CUBLAS: no
ggml_cuda_init: found 2 CUDA devices:
  Device 0: NVIDIA GeForce RTX 4090, compute capability 8.9, VMM: yes
  Device 1: NVIDIA GeForce RTX 4090, compute capability 8.9, VMM: yes
INFO [                    main] build info | tid="136570102431744" timestamp=1756759605 build=3871 commit="d10d90ae"
INFO [                    main] system info | tid="136570102431744" timestamp=1756759605 n_threads=30 n_threads_batch=-1 total_threads=64 system_info="AVX = 1 | AVX_VNNI = 0 | AVX2 = 1 | AVX512 = 1 | AVX512_VBMI = 1 | AVX512_VNNI = 1 | AVX512_BF16 = 1 | FMA = 1 | NEON = 0 | SVE = 0 | ARM_FMA = 0 | F16C = 1 | FP16_VA = 0 | WASM_SIMD = 0 | BLAS = 1 | SSE3 = 1 | SSSE3 = 1 | VSX = 0 | MATMUL_INT8 = 0 | LLAMAFILE = 1 | "
llama_model_loader: additional 1 GGUFs metadata loaded.
llama_model_loader: loaded meta data with 48 key-value pairs and 803 tensors from ./ubergarm/GLM-4.5-Air-GGUF/IQ4_KSS/GLM-4.5-Air-IQ4_KSS-00001-of-00002.gguf (version GGUF V3 (latest))
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
llama_model_loader: - kv  18:                          general.file_type u32              = 148
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
llama_model_loader: - type q8_0:    8 tensors
llama_model_loader: - type iq4_nl:   46 tensors
llama_model_loader: - type q6_0:   47 tensors
llama_model_loader: - type iq4_k:    1 tensors
llama_model_loader: - type iq6_k:    1 tensors
llama_model_loader: - type iq4_kss:   95 tensors
llama_model_loader: - type iq5_ks:  274 tensors
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
llm_load_print_meta: model ftype      = IQ4_KSS - 4.0 bpw
llm_load_print_meta: model params     = 110.469 B
llm_load_print_meta: model size       = 54.801 GiB (4.261 BPW)
llm_load_print_meta: repeating layers = 53.997 GiB (4.246 BPW, 109.227 B parameters)
llm_load_print_meta: general.name     = GLM 4.5 Air
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
llm_load_tensors: ggml ctx size =    0.99 MiB
Tensor blk.1.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.1.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.2.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.2.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.3.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.3.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.4.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.4.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.5.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.5.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.6.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.6.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.7.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.7.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.8.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.8.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.9.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.9.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.10.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.10.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.11.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.11.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.12.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.12.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.13.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.13.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.14.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.14.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.15.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.15.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.16.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.16.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.17.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.17.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.18.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.18.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.19.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.19.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.20.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.20.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.21.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.21.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.22.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.22.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.23.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.23.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.24.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.24.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.25.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.25.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.26.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.26.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.27.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.27.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.28.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.28.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.29.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.29.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.30.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.30.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.31.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.31.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.32.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.32.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.33.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.33.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.34.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.34.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.35.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.35.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.36.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.36.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.37.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.37.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.38.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.38.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.39.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.39.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.40.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.40.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.41.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.41.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.42.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.42.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.43.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.43.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.44.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.44.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.45.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.45.ffn_up_exps.weight buffer type overriden to CPU
model has unused tensor blk.46.attn_norm.weight (size = 16384 bytes) -- ignoring
model has unused tensor blk.46.attn_q.weight (size = 33079296 bytes) -- ignoring
model has unused tensor blk.46.attn_k.weight (size = 2756608 bytes) -- ignoring
model has unused tensor blk.46.attn_v.weight (size = 2756608 bytes) -- ignoring
model has unused tensor blk.46.attn_q.bias (size = 49152 bytes) -- ignoring
model has unused tensor blk.46.attn_k.bias (size = 4096 bytes) -- ignoring
model has unused tensor blk.46.attn_v.bias (size = 4096 bytes) -- ignoring
model has unused tensor blk.46.attn_output.weight (size = 33046528 bytes) -- ignoring
model has unused tensor blk.46.post_attention_norm.weight (size = 16384 bytes) -- ignoring
model has unused tensor blk.46.ffn_gate_inp.weight (size = 2097152 bytes) -- ignoring
model has unused tensor blk.46.exp_probs_b.bias (size = 512 bytes) -- ignoring
model has unused tensor blk.46.ffn_gate_exps.weight (size = 369819648 bytes) -- ignoring
Tensor blk.46.ffn_down_exps.weight buffer type overriden to CPU
model has unused tensor blk.46.ffn_down_exps.weight (size = 415236096 bytes) -- ignoring
Tensor blk.46.ffn_up_exps.weight buffer type overriden to CPU
model has unused tensor blk.46.ffn_up_exps.weight (size = 369819648 bytes) -- ignoring
model has unused tensor blk.46.ffn_gate_shexp.weight (size = 3790336 bytes) -- ignoring
model has unused tensor blk.46.ffn_down_shexp.weight (size = 4685824 bytes) -- ignoring
model has unused tensor blk.46.ffn_up_shexp.weight (size = 3790336 bytes) -- ignoring
model has unused tensor blk.46.nextn.eh_proj.weight (size = 16793600 bytes) -- ignoring
model has unused tensor blk.46.nextn.embed_tokens.weight (size = 310984704 bytes) -- ignoring
model has unused tensor blk.46.nextn.enorm.weight (size = 16384 bytes) -- ignoring
model has unused tensor blk.46.nextn.hnorm.weight (size = 16384 bytes) -- ignoring
model has unused tensor blk.46.nextn.shared_head_head.weight (size = 310984704 bytes) -- ignoring
model has unused tensor blk.46.nextn.shared_head_norm.weight (size = 16384 bytes) -- ignoring
llm_load_tensors: offloading 47 repeating layers to GPU
llm_load_tensors: offloading non-repeating layers to GPU
llm_load_tensors: offloaded 48/48 layers to GPU
llm_load_tensors:        CPU buffer size = 33690.94 MiB
llm_load_tensors:  CUDA_Host buffer size =   333.00 MiB
llm_load_tensors:      CUDA0 buffer size = 10678.72 MiB
llm_load_tensors:      CUDA1 buffer size =  9620.91 MiB
....................................................................................................
llama_new_context_with_model: n_ctx      = 128000
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
llama_kv_cache_init:      CUDA0 KV buffer size =  3515.63 MiB
llama_kv_cache_init:      CUDA1 KV buffer size =  3093.76 MiB
llama_new_context_with_model: KV self size  = 6609.38 MiB, K (q4_0): 3304.69 MiB, V (q4_0): 3304.69 MiB
llama_new_context_with_model:  CUDA_Host  output buffer size =     1.16 MiB
llama_new_context_with_model: pipeline parallelism enabled (n_copies=4)
ggml_backend_cuda_buffer_type_alloc_buffer: allocating 180317.55 MiB on device 0: cudaMalloc failed: out of memory
ggml_gallocr_reserve_n: failed to allocate CUDA0 buffer of size 189076659200
llama_new_context_with_model: failed to allocate compute buffers
llama_init_from_gpt_params: error: failed to create context with model './ubergarm/GLM-4.5-Air-GGUF/IQ4_KSS/GLM-4.5-Air-IQ4_KSS-00001-of-00002.gguf'
 ERR [              load_model] unable to load model | tid="136570102431744" timestamp=1756759620 model="./ubergarm/GLM-4.5-Air-GGUF/IQ4_KSS/GLM-4.5-Air-IQ4_KSS-00001-of-00002.gguf"
```

---

👤 **firecoperana** commented on **2025-09-02** at **00:53:34**

recompile the code with -DGGML_SCHED_MAX_COPIES=1

---

👤 **ikawrakow** commented on **2025-09-02** at **12:12:59**

Or just try the latest main branch and, if It still doesn't work, reopen the issue.

---

👤 **juanqui** commented on **2025-09-02** at **12:50:00**

Can confirm `-DGGML_SCHED_MAX_COPIES=1` fixed the issue for me. Thank you!