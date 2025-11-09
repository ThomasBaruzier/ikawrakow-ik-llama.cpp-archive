## 📌 [Issue #737](https://github.com/ikawrakow/ik_llama.cpp/issues/737) - Bug: Moonlight 16b (deepseek 2) key not found

| **Author** | `hardWorker254` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-08-28 |
| **Updated** | 2025-08-28 |

---

## 📄 Description

### What happened?

Moonlight 16B does not work(https://huggingface.co/moonshotai/Moonlight-16B-A3B-Instruct)

Command:./llama-server -m /home/prof/.llama-runner/tensors/Moonlight-16B-IQ4_XS.gguf -fa --cache-type-k q8_0 --cache-type-v q8_0 -fmoe 

It works with current llama.cpp version (version: 6301 (da54f9f1)
built with cc (GCC) 15.2.1 20250813 for x86_64-pc-linux-gnu
)


### Name and Version

version: 3865 (dac5b483)
built with cc (GCC) 15.2.1 20250813 for x86_64-pc-linux-gnu


### What operating system are you seeing the problem on?

Linux

### Relevant log output

```shell
./llama-server -m /home/prof/.llama-runner/tensors/Moonlight-16B-IQ4_XS.gguf -fa --cache-type-k q8_0 --cache-type-v q8_0 -fmoe 
INFO [                    main] build info | tid="140477161699648" timestamp=1756360003 build=3865 commit="dac5b483"
INFO [                    main] system info | tid="140477161699648" timestamp=1756360003 n_threads=4 n_threads_batch=-1 total_threads=4 system_info="AVX = 1 | AVX_VNNI = 0 | AVX2 = 1 | AVX512 = 0 | AVX512_VBMI = 0 | AVX512_VNNI = 0 | AVX512_BF16 = 0 | FMA = 1 | NEON = 0 | SVE = 0 | ARM_FMA = 0 | F16C = 1 | FP16_VA = 0 | WASM_SIMD = 0 | BLAS = 1 | SSE3 = 1 | SSSE3 = 1 | VSX = 0 | MATMUL_INT8 = 0 | LLAMAFILE = 1 | "
llama_model_loader: loaded meta data with 44 key-value pairs and 430 tensors from /home/prof/.llama-runner/tensors/Moonlight-16B-IQ4_XS.gguf (version GGUF V3 (latest))
llama_model_loader: Dumping metadata keys/values. Note: KV overrides do not apply in this output.
llama_model_loader: - kv   0:                       general.architecture str              = deepseek2
llama_model_loader: - kv   1:                               general.type str              = model
llama_model_loader: - kv   2:                               general.name str              = Moonlight 16B A3B Instruct
llama_model_loader: - kv   3:                           general.finetune str              = Instruct
llama_model_loader: - kv   4:                           general.basename str              = Moonlight
llama_model_loader: - kv   5:                         general.size_label str              = 16B-A3B
llama_model_loader: - kv   6:                            general.license str              = mit
llama_model_loader: - kv   7:                      deepseek2.block_count u32              = 27
llama_model_loader: - kv   8:                   deepseek2.context_length u32              = 8192
llama_model_loader: - kv   9:                 deepseek2.embedding_length u32              = 2048
llama_model_loader: - kv  10:              deepseek2.feed_forward_length u32              = 11264
llama_model_loader: - kv  11:             deepseek2.attention.head_count u32              = 16
llama_model_loader: - kv  12:          deepseek2.attention.head_count_kv u32              = 1
llama_model_loader: - kv  13:                   deepseek2.rope.freq_base f32              = 50000.000000
llama_model_loader: - kv  14: deepseek2.attention.layer_norm_rms_epsilon f32              = 0.000010
llama_model_loader: - kv  15:                deepseek2.expert_used_count u32              = 6
llama_model_loader: - kv  16:        deepseek2.leading_dense_block_count u32              = 1
llama_model_loader: - kv  17:                       deepseek2.vocab_size u32              = 163840
llama_model_loader: - kv  18:           deepseek2.attention.kv_lora_rank u32              = 512
llama_model_loader: - kv  19:             deepseek2.attention.key_length u32              = 576
llama_model_loader: - kv  20:           deepseek2.attention.value_length u32              = 512
llama_model_loader: - kv  21:         deepseek2.attention.key_length_mla u32              = 192
llama_model_loader: - kv  22:       deepseek2.attention.value_length_mla u32              = 128
llama_model_loader: - kv  23:       deepseek2.expert_feed_forward_length u32              = 1408
llama_model_loader: - kv  24:                     deepseek2.expert_count u32              = 64
llama_model_loader: - kv  25:              deepseek2.expert_shared_count u32              = 2
llama_model_loader: - kv  26:             deepseek2.expert_weights_scale f32              = 2.446000
llama_model_loader: - kv  27:              deepseek2.expert_weights_norm bool             = true
llama_model_loader: - kv  28:               deepseek2.expert_gating_func u32              = 2
llama_model_loader: - kv  29:             deepseek2.rope.dimension_count u32              = 64
llama_model_loader: - kv  30:                       tokenizer.ggml.model str              = gpt2
llama_model_loader: - kv  31:                         tokenizer.ggml.pre str              = kimi-k2
llama_model_loader: - kv  32:                      tokenizer.ggml.tokens arr[str,163840]  = ["!", "\"", "#", "$", "%", "&", "'", ...
llama_model_loader: - kv  33:                  tokenizer.ggml.token_type arr[i32,163840]  = [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, ...
llama_model_loader: - kv  34:                      tokenizer.ggml.merges arr[str,163328]  = ["Ġ Ġ", "ĠĠ ĠĠ", "Ġ t", "i n",...
llama_model_loader: - kv  35:                tokenizer.ggml.bos_token_id u32              = 163584
llama_model_loader: - kv  36:                tokenizer.ggml.eos_token_id u32              = 163586
llama_model_loader: - kv  37:                    tokenizer.chat_template str              = {%- for message in messages -%}{%- if...
llama_model_loader: - kv  38:               general.quantization_version u32              = 2
llama_model_loader: - kv  39:                          general.file_type u32              = 30
llama_model_loader: - kv  40:                      quantize.imatrix.file str              = Moonlight-16B-A3B-Instruct/Moonlight-...
llama_model_loader: - kv  41:                   quantize.imatrix.dataset str              = calibration_datav3.txt
llama_model_loader: - kv  42:             quantize.imatrix.entries_count u32              = 320
llama_model_loader: - kv  43:              quantize.imatrix.chunks_count u32              = 126
llama_model_loader: - type  f32:  134 tensors
llama_model_loader: - type q6_K:    1 tensors
llama_model_loader: - type iq4_nl:   53 tensors
llama_model_loader: - type iq4_xs:  242 tensors
================= Adjusted mainline llama.cpp MLA tensors to ik_llama.cpp
llama_model_load: error loading model: error loading model hyperparameters: key not found in model: deepseek2.rope.scaling.yarn_log_multiplier
llama_load_model_from_file: failed to load model
llama_init_from_gpt_params: error: failed to load model '/home/prof/.llama-runner/tensors/Moonlight-16B-IQ4_XS.gguf'
 ERR [              load_model] unable to load model | tid="140477161699648" timestamp=1756360003 model="/home/prof/.llama-runner/tensors/Moonlight-16B-IQ4_XS.gguf"
free(): invalid pointer
[1]    2000176 IOT instruction (core dumped)  ./llama-server -m /home/prof/.llama-runner/tensors/Moonlight-16B-IQ4_XS.gguf
```

---

## 💬 Conversation

👤 **ikawrakow** commented on **2025-08-28** at **06:46:18**

When you run with mainline `llama.cpp`, what does it sat in the log about `rope_yarn_log_mul` ? I see that their "fix" it is to just not require this parameter to be present in the metadata, but then it just gets used unconditionally.

---

👤 **ikawrakow** commented on **2025-08-28** at **06:49:10**

Does [#738](https://github.com/ikawrakow/ik_llama.cpp/issues/738) fix it?

---

👤 **hardWorker254** commented on **2025-08-28** at **07:14:39**

> When you run with mainline `llama.cpp`, what does it sat in the log about `rope_yarn_log_mul` ? I see that their "fix" it is to just not require this parameter to be present in the metadata, but then it just gets used unconditionally.

print_info: rope_yarn_log_mul    = 0.0000

---

👤 **hardWorker254** commented on **2025-08-28** at **07:30:03**

> Does [[#738](https://github.com/ikawrakow/ik_llama.cpp/issues/738)](https://github.com/ikawrakow/ik_llama.cpp/pull/738) fix it?

Yeah, It works now, thanks!

---

👤 **hardWorker254** commented on **2025-08-28** at **07:41:44**

Now I have error with changing cache type

./llama-server -m /home/prof/.llama-runner/tensors/Moonlight-16B-IQ4_XS.gguf --cache-type-k q8_0 --cache-type-v q8_0 -fa



INFO [                    main] build info | tid="140190723975488" timestamp=1756366748 build=0 commit="unknown"
INFO [                    main] system info | tid="140190723975488" timestamp=1756366748 n_threads=4 n_threads_batch=-1 total_threads=4 system_info="AVX = 1 | AVX_VNNI = 0 | AVX2 = 1 | AVX512 = 0 | AVX512_VBMI = 0 | AVX512_VNNI = 0 | AVX512_BF16 = 0 | FMA = 1 | NEON = 0 | SVE = 0 | ARM_FMA = 0 | F16C = 1 | FP16_VA = 0 | WASM_SIMD = 0 | BLAS = 1 | SSE3 = 1 | SSSE3 = 1 | VSX = 0 | MATMUL_INT8 = 0 | LLAMAFILE = 1 | "
llama_model_loader: loaded meta data with 44 key-value pairs and 430 tensors from /home/prof/.llama-runner/tensors/Moonlight-16B-IQ4_XS.gguf (version GGUF V3 (latest))
llama_model_loader: Dumping metadata keys/values. Note: KV overrides do not apply in this output.
llama_model_loader: - kv   0:                       general.architecture str              = deepseek2
llama_model_loader: - kv   1:                               general.type str              = model
llama_model_loader: - kv   2:                               general.name str              = Moonlight 16B A3B Instruct
llama_model_loader: - kv   3:                           general.finetune str              = Instruct
llama_model_loader: - kv   4:                           general.basename str              = Moonlight
llama_model_loader: - kv   5:                         general.size_label str              = 16B-A3B
llama_model_loader: - kv   6:                            general.license str              = mit
llama_model_loader: - kv   7:                      deepseek2.block_count u32              = 27
llama_model_loader: - kv   8:                   deepseek2.context_length u32              = 8192
llama_model_loader: - kv   9:                 deepseek2.embedding_length u32              = 2048
llama_model_loader: - kv  10:              deepseek2.feed_forward_length u32              = 11264
llama_model_loader: - kv  11:             deepseek2.attention.head_count u32              = 16
llama_model_loader: - kv  12:          deepseek2.attention.head_count_kv u32              = 1
llama_model_loader: - kv  13:                   deepseek2.rope.freq_base f32              = 50000.000000
llama_model_loader: - kv  14: deepseek2.attention.layer_norm_rms_epsilon f32              = 0.000010
llama_model_loader: - kv  15:                deepseek2.expert_used_count u32              = 6
llama_model_loader: - kv  16:        deepseek2.leading_dense_block_count u32              = 1
llama_model_loader: - kv  17:                       deepseek2.vocab_size u32              = 163840
llama_model_loader: - kv  18:           deepseek2.attention.kv_lora_rank u32              = 512
llama_model_loader: - kv  19:             deepseek2.attention.key_length u32              = 576
llama_model_loader: - kv  20:           deepseek2.attention.value_length u32              = 512
llama_model_loader: - kv  21:         deepseek2.attention.key_length_mla u32              = 192
llama_model_loader: - kv  22:       deepseek2.attention.value_length_mla u32              = 128
llama_model_loader: - kv  23:       deepseek2.expert_feed_forward_length u32              = 1408
llama_model_loader: - kv  24:                     deepseek2.expert_count u32              = 64
llama_model_loader: - kv  25:              deepseek2.expert_shared_count u32              = 2
llama_model_loader: - kv  26:             deepseek2.expert_weights_scale f32              = 2.446000
llama_model_loader: - kv  27:              deepseek2.expert_weights_norm bool             = true
llama_model_loader: - kv  28:               deepseek2.expert_gating_func u32              = 2
llama_model_loader: - kv  29:             deepseek2.rope.dimension_count u32              = 64
llama_model_loader: - kv  30:                       tokenizer.ggml.model str              = gpt2
llama_model_loader: - kv  31:                         tokenizer.ggml.pre str              = kimi-k2
llama_model_loader: - kv  32:                      tokenizer.ggml.tokens arr[str,163840]  = ["!", "\"", "#", "$", "%", "&", "'", ...
llama_model_loader: - kv  33:                  tokenizer.ggml.token_type arr[i32,163840]  = [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, ...
llama_model_loader: - kv  34:                      tokenizer.ggml.merges arr[str,163328]  = ["Ġ Ġ", "ĠĠ ĠĠ", "Ġ t", "i n",...
llama_model_loader: - kv  35:                tokenizer.ggml.bos_token_id u32              = 163584
llama_model_loader: - kv  36:                tokenizer.ggml.eos_token_id u32              = 163586
llama_model_loader: - kv  37:                    tokenizer.chat_template str              = {%- for message in messages -%}{%- if...
llama_model_loader: - kv  38:               general.quantization_version u32              = 2
llama_model_loader: - kv  39:                          general.file_type u32              = 30
llama_model_loader: - kv  40:                      quantize.imatrix.file str              = Moonlight-16B-A3B-Instruct/Moonlight-...
llama_model_loader: - kv  41:                   quantize.imatrix.dataset str              = calibration_datav3.txt
llama_model_loader: - kv  42:             quantize.imatrix.entries_count u32              = 320
llama_model_loader: - kv  43:              quantize.imatrix.chunks_count u32              = 126
llama_model_loader: - type  f32:  134 tensors
llama_model_loader: - type q6_K:    1 tensors
llama_model_loader: - type iq4_nl:   53 tensors
llama_model_loader: - type iq4_xs:  242 tensors
================= Adjusted mainline llama.cpp MLA tensors to ik_llama.cpp
init_tokenizer: initializing tokenizer for type 2
load: printing all EOG tokens:
load:   - 163586 ('<|im_end|>')
load: special tokens cache size = 256
load: token to piece cache size = 1.0607 MB
llm_load_print_meta: format           = GGUF V3 (latest)
llm_load_print_meta: arch             = deepseek2
llm_load_print_meta: n_ctx_train      = 8192
llm_load_print_meta: n_embd           = 2048
llm_load_print_meta: n_layer          = 27
llm_load_print_meta: n_head           = 16
llm_load_print_meta: n_head_kv        = 16
llm_load_print_meta: n_rot            = 64
llm_load_print_meta: n_swa            = 0
llm_load_print_meta: n_swa_pattern    = 1
llm_load_print_meta: n_embd_head_k    = 192
llm_load_print_meta: n_embd_head_v    = 128
llm_load_print_meta: n_gqa            = 1
llm_load_print_meta: n_embd_k_gqa     = 3072
llm_load_print_meta: n_embd_v_gqa     = 2048
llm_load_print_meta: f_norm_eps       = 0.0e+00
llm_load_print_meta: f_norm_rms_eps   = 1.0e-05
llm_load_print_meta: f_clamp_kqv      = 0.0e+00
llm_load_print_meta: f_max_alibi_bias = 0.0e+00
llm_load_print_meta: f_logit_scale    = 0.0e+00
llm_load_print_meta: n_ff             = 11264
llm_load_print_meta: n_expert         = 64
llm_load_print_meta: n_expert_used    = 6
llm_load_print_meta: causal attn      = 1
llm_load_print_meta: pooling type     = 0
llm_load_print_meta: rope type        = 0
llm_load_print_meta: rope scaling     = linear
llm_load_print_meta: freq_base_train  = 50000.0
llm_load_print_meta: freq_scale_train = 1
llm_load_print_meta: n_ctx_orig_yarn  = 8192
llm_load_print_meta: rope_finetuned   = unknown
llm_load_print_meta: ssm_d_conv       = 0
llm_load_print_meta: ssm_d_inner      = 0
llm_load_print_meta: ssm_d_state      = 0
llm_load_print_meta: ssm_dt_rank      = 0
llm_load_print_meta: model type       = 16B
llm_load_print_meta: model ftype      = IQ4_XS - 4.25 bpw
llm_load_print_meta: model params     = 15.960 B
llm_load_print_meta: model size       = 8.139 GiB (4.380 BPW) 
llm_load_print_meta: repeating layers = 7.716 GiB (4.335 BPW, 15.289 B parameters)
llm_load_print_meta: general.name     = Moonlight 16B A3B Instruct
llm_load_print_meta: n_layer_dense_lead   = 1
llm_load_print_meta: n_lora_q             = 0
llm_load_print_meta: n_lora_kv            = 512
llm_load_print_meta: n_ff_exp             = 1408
llm_load_print_meta: n_expert_shared      = 2
llm_load_print_meta: expert_weights_scale = 2.4
llm_load_print_meta: expert_weights_norm  = 1
llm_load_print_meta: expert_gating_func   = sigmoid
llm_load_print_meta: rope_yarn_log_mul    = 0.0000
print_info: vocab type       = BPE
print_info: n_vocab          = 163840
print_info: n_merges         = 163328
print_info: BOS token        = 163584 '[BOS]'
print_info: EOS token        = 163586 '<|im_end|>'
print_info: EOT token        = 163586 '<|im_end|>'
print_info: LF token         = 198 'Ċ'
print_info: EOG token        = 163586 '<|im_end|>'
print_info: max token length = 512
llm_load_tensors: ggml ctx size =    0.18 MiB
llm_load_tensors:        CPU buffer size =  8334.06 MiB
......................................................................................
============ llm_prepare_mla: need to compute 27 wkv_b tensors
Computed blk.0.attn_kv_b.weight as 512 x 4096 and stored in buffer CPU
Computed blk.1.attn_kv_b.weight as 512 x 4096 and stored in buffer CPU
Computed blk.2.attn_kv_b.weight as 512 x 4096 and stored in buffer CPU
Computed blk.3.attn_kv_b.weight as 512 x 4096 and stored in buffer CPU
Computed blk.4.attn_kv_b.weight as 512 x 4096 and stored in buffer CPU
Computed blk.5.attn_kv_b.weight as 512 x 4096 and stored in buffer CPU
Computed blk.6.attn_kv_b.weight as 512 x 4096 and stored in buffer CPU
Computed blk.7.attn_kv_b.weight as 512 x 4096 and stored in buffer CPU
Computed blk.8.attn_kv_b.weight as 512 x 4096 and stored in buffer CPU
Computed blk.9.attn_kv_b.weight as 512 x 4096 and stored in buffer CPU
Computed blk.10.attn_kv_b.weight as 512 x 4096 and stored in buffer CPU
Computed blk.11.attn_kv_b.weight as 512 x 4096 and stored in buffer CPU
Computed blk.12.attn_kv_b.weight as 512 x 4096 and stored in buffer CPU
Computed blk.13.attn_kv_b.weight as 512 x 4096 and stored in buffer CPU
Computed blk.14.attn_kv_b.weight as 512 x 4096 and stored in buffer CPU
Computed blk.15.attn_kv_b.weight as 512 x 4096 and stored in buffer CPU
Computed blk.16.attn_kv_b.weight as 512 x 4096 and stored in buffer CPU
Computed blk.17.attn_kv_b.weight as 512 x 4096 and stored in buffer CPU
Computed blk.18.attn_kv_b.weight as 512 x 4096 and stored in buffer CPU
Computed blk.19.attn_kv_b.weight as 512 x 4096 and stored in buffer CPU
Computed blk.20.attn_kv_b.weight as 512 x 4096 and stored in buffer CPU
Computed blk.21.attn_kv_b.weight as 512 x 4096 and stored in buffer CPU
Computed blk.22.attn_kv_b.weight as 512 x 4096 and stored in buffer CPU
Computed blk.23.attn_kv_b.weight as 512 x 4096 and stored in buffer CPU
Computed blk.24.attn_kv_b.weight as 512 x 4096 and stored in buffer CPU
Computed blk.25.attn_kv_b.weight as 512 x 4096 and stored in buffer CPU
Computed blk.26.attn_kv_b.weight as 512 x 4096 and stored in buffer CPU
llama_new_context_with_model: n_ctx      = 8192
llama_new_context_with_model: n_batch    = 2048
llama_new_context_with_model: n_ubatch   = 512
llama_new_context_with_model: flash_attn = 1
llama_new_context_with_model: mla_attn   = 0
llama_new_context_with_model: attn_max_b = 0
llama_new_context_with_model: fused_moe  = 0
llama_new_context_with_model: ser        = -1, 0
llama_new_context_with_model: freq_base  = 50000.0
llama_new_context_with_model: freq_scale = 1
llama_kv_cache_init:        CPU KV buffer size =  1147.50 MiB
llama_new_context_with_model: KV self size  = 1147.50 MiB, K (q8_0):  688.50 MiB, V (q8_0):  459.00 MiB
llama_new_context_with_model:        CPU  output buffer size =     1.25 MiB
llama_new_context_with_model:        CPU compute buffer size =   324.00 MiB
llama_new_context_with_model: graph nodes  = 1497
llama_new_context_with_model: graph splits = 378
======================================= HAVE_FANCY_SIMD is NOT defined
/home/prof/Загрузки/ik_llama.cpp-ik-optional_yarn_log_multiplier/ggml/src/./iqk/fa/iqk_fa_templates.h:1156: GGML_ASSERT(S > 0) failed
ptrace: Операция не позволена.
No stack.
The program is not being run.
[1]    2058934 IOT instruction (core dumped)  ./llama-server -m /home/prof/.llama-runner/tensors/Moonlight-16B-IQ4_XS.gguf 


Error with q5_0 cache:
llama_new_context_with_model: KV self size  =   83.53 MiB, c^KV (q5_0):   83.53 MiB, kv^T: not used
llama_new_context_with_model:        CPU  output buffer size =     1.25 MiB
llama_new_context_with_model:        CPU compute buffer size =   324.00 MiB
llama_new_context_with_model: graph nodes  = 1470
llama_new_context_with_model: graph splits = 324
======================================= HAVE_FANCY_SIMD is NOT defined
Oops(ggml_compute_forward_sum_rows_f32, ffn_moe_weights_sum-1): found -nan for i1 = 0, i2 = 0, i3 = 0. ne00 = 64

---

👤 **ikawrakow** commented on **2025-08-28** at **11:07:31**

Add `-mla 3` to your command line arguments. This will make the KV cache much smaller (apart from `mla = 0` apparently being broken). It is also useful to use `-fmoe`, which will make it run somewhat faster.