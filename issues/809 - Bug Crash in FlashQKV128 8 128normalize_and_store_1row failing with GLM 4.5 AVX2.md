## 📌 [Issue #809](https://github.com/ikawrakow/ik_llama.cpp/issues/809) - Bug: Crash in FlashQKV<128, 8, 128>::normalize_and_store_1row failing with GLM 4.5, AVX2, -amb/-ub/-b

| **Author** | `os360` |
| :--- | :--- |
| **State** | ✅ **Open** |
| **Created** | 2025-09-29 |
| **Updated** | 2025-09-30 |

---

## 📄 Description

### What happened?

I was comparing the output of DeepSeek vs GLM-4.5 when I isolated a case where llama-server repeatedly fails when these parameters are passed:
     --attention-max-batch 2048 --batch-size 16384 --ubatch-size 1024 --no-warmup -mla 3 -fa -fmoe -thp --parallel 2
but works with:
     --no-warmup -mla 2 -fa -fmoe -thp --parallel 2
on a CPU-only dual E5 V4 AVX2 system
when sent the following two prompts via webui in order:
--- prompt one ---
Say hello
--- prompt two ---
In prone, the infant displays a “swimming” posture with alternating arm and leg movements while attempting to reach for a toy. Which of the following patterns would suggest a developmental delay rather than typical dissociation of head and limbs?

    The infant can lift one arm while the opposite leg remains still.
    The infant moves both arms simultaneously without alternating.
    The infant initiates head turning before any limb movement.
    The infant shows a brief pause between each limb movement.
    The infant’s movements become more rhythmic with repeated trials.
--- end ---
It seems to consistently fail on the second prompt regardless of what the first was in both debug and release builds.

### Name and Version

$ ./build/bin/llama-server --version
version: 3899 (3d4977cb)
built with cc (Gentoo 14.3.0 p8) 14.3.0 for x86_64-pc-linux-gnu


### What operating system are you seeing the problem on?

Linux

### Relevant log output

```shell
sudo chrt --fifo 50 sudo -u trunk numactl --balancing --interleave=0,1 --physcpubind=1-15,17-31 ./build/bin/llama-server --model /home/trunk/Public/AI/models/ubergarm/GLM-4.5-GGUF/IQ4_KSS/GLM-4.5-IQ4_KSS-00001-of-00004.gguf --threads 30 --threads-batch 30 --numa numactl --temp 0.6 --samplers 'min_p;temperature' --no-warmup --ctx-size 0 --predict 16384 --metrics --host 0.0.0.0 --model-draft /home/trunk/Public/AI/models/jukofyork/GLM-4.5-DRAFT-0.6B-v3.0-GGUF/GLM-4.5-DRAFT-0.6B-32k-Q4_0.gguf --prompt-cache /tmp/server.GLM-4.5-IQ4_KSS-00001-of-00004.gguf.cache --prompt-cache-all --lookup-cache-dynamic /tmp/server-lookup.GLM-4.5-IQ4_KSS-00001-of-00004.gguf.cache --attention-max-batch 2048 --batch-size 16384 --ubatch-size 1024 --no-warmup -mla 3 -fa -fmoe -thp --parallel 2
INFO [                    main] build info | tid="139870314715392" timestamp=1759184678 build=3899 commit="3d4977cb"
INFO [                    main] system info | tid="139870314715392" timestamp=1759184678 n_threads=30 n_threads_batch=30 total_threads=32 system_info="AVX = 1 | AVX_VNNI = 0 | AVX2 = 1 | AVX512 = 0 | AVX512_VBMI = 0 | AVX512_VNNI = 0 | AVX512_BF16 = 0 | FMA = 1 | NEON = 0 | SVE = 0 | ARM_FMA = 0 | F16C = 1 | FP16_VA = 0 | WASM_SIMD = 0 | BLAS = 0 | SSE3 = 1 | SSSE3 = 1 | VSX = 0 | MATMUL_INT8 = 0 | LLAMAFILE = 1 | "
llama_model_loader: additional 3 GGUFs metadata loaded.
llama_model_loader: loaded meta data with 50 key-value pairs and 1761 tensors from /home/trunk/Public/AI/models/ubergarm/GLM-4.5-GGUF/IQ4_KSS/GLM-4.5-IQ4_KSS-00001-of-00004.gguf (version GGUF V3 (latest))
llama_model_loader: Dumping metadata keys/values. Note: KV overrides do not apply in this output.
llama_model_loader: - kv   0:                       general.architecture str              = glm4moe
llama_model_loader: - kv   1:                               general.type str              = model
llama_model_loader: - kv   2:                               general.name str              = GLM 4.5
llama_model_loader: - kv   3:                            general.version str              = 4.5
llama_model_loader: - kv   4:                           general.basename str              = GLM
llama_model_loader: - kv   5:                         general.size_label str              = 160x21B
llama_model_loader: - kv   6:                            general.license str              = mit
llama_model_loader: - kv   7:                               general.tags arr[str,1]       = ["text-generation"]
llama_model_loader: - kv   8:                          general.languages arr[str,2]       = ["en", "zh"]
llama_model_loader: - kv   9:                        glm4moe.block_count u32              = 93
llama_model_loader: - kv  10:                     glm4moe.context_length u32              = 131072
llama_model_loader: - kv  11:                   glm4moe.embedding_length u32              = 5120
llama_model_loader: - kv  12:                glm4moe.feed_forward_length u32              = 12288
llama_model_loader: - kv  13:               glm4moe.attention.head_count u32              = 96
llama_model_loader: - kv  14:            glm4moe.attention.head_count_kv u32              = 8
llama_model_loader: - kv  15:                     glm4moe.rope.freq_base f32              = 1000000.000000
llama_model_loader: - kv  16:   glm4moe.attention.layer_norm_rms_epsilon f32              = 0.000010
llama_model_loader: - kv  17:                  glm4moe.expert_used_count u32              = 8
llama_model_loader: - kv  18:               glm4moe.attention.key_length u32              = 128
llama_model_loader: - kv  19:             glm4moe.attention.value_length u32              = 128
llama_model_loader: - kv  20:                          general.file_type u32              = 148
llama_model_loader: - kv  21:               glm4moe.rope.dimension_count u32              = 64
llama_model_loader: - kv  22:                       glm4moe.expert_count u32              = 160
llama_model_loader: - kv  23:         glm4moe.expert_feed_forward_length u32              = 1536
llama_model_loader: - kv  24:                glm4moe.expert_shared_count u32              = 1
llama_model_loader: - kv  25:          glm4moe.leading_dense_block_count u32              = 3
llama_model_loader: - kv  26:                 glm4moe.expert_gating_func u32              = 2
llama_model_loader: - kv  27:               glm4moe.expert_weights_scale f32              = 2.500000
llama_model_loader: - kv  28:                glm4moe.expert_weights_norm bool             = true
llama_model_loader: - kv  29:               glm4moe.nextn_predict_layers u32              = 1
llama_model_loader: - kv  30:                       tokenizer.ggml.model str              = gpt2
llama_model_loader: - kv  31:                         tokenizer.ggml.pre str              = glm4
llama_model_loader: - kv  32:                      tokenizer.ggml.tokens arr[str,151552]  = ["!", "\"", "#", "$", "%", "&", "'", ...
llama_model_loader: - kv  33:                  tokenizer.ggml.token_type arr[i32,151552]  = [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, ...
llama_model_loader: - kv  34:                      tokenizer.ggml.merges arr[str,318088]  = ["Ġ Ġ", "Ġ ĠĠĠ", "ĠĠ ĠĠ", "...
llama_model_loader: - kv  35:                tokenizer.ggml.eos_token_id u32              = 151329
llama_model_loader: - kv  36:            tokenizer.ggml.padding_token_id u32              = 151329
llama_model_loader: - kv  37:                tokenizer.ggml.bos_token_id u32              = 151331
llama_model_loader: - kv  38:                tokenizer.ggml.eot_token_id u32              = 151336
llama_model_loader: - kv  39:            tokenizer.ggml.unknown_token_id u32              = 151329
llama_model_loader: - kv  40:                tokenizer.ggml.eom_token_id u32              = 151338
llama_model_loader: - kv  41:                    tokenizer.chat_template str              = [gMASK]<sop>\n{%- if tools -%}\n<|syste...
llama_model_loader: - kv  42:               general.quantization_version u32              = 2
llama_model_loader: - kv  43:                      quantize.imatrix.file str              = /mnt/raid/models/ubergarm/GLM-4.5-GGU...
llama_model_loader: - kv  44:                   quantize.imatrix.dataset str              = ubergarm-imatrix-calibration-corpus-v...
llama_model_loader: - kv  45:             quantize.imatrix.entries_count i32              = 1001
llama_model_loader: - kv  46:              quantize.imatrix.chunks_count i32              = 814
llama_model_loader: - kv  47:                                   split.no u16              = 0
llama_model_loader: - kv  48:                                split.count u16              = 4
llama_model_loader: - kv  49:                        split.tensors.count i32              = 1761
llama_model_loader: - type  f32:  835 tensors
llama_model_loader: - type q8_0:   13 tensors
llama_model_loader: - type iq4_k:    1 tensors
llama_model_loader: - type iq6_k:  181 tensors
llama_model_loader: - type iq4_ks:  276 tensors
llama_model_loader: - type iq4_kss:  180 tensors
llama_model_loader: - type iq5_ks:  275 tensors
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
llm_load_print_meta: n_ctx_orig_yarn  = 131072
llm_load_print_meta: rope_finetuned   = unknown
llm_load_print_meta: ssm_d_conv       = 0
llm_load_print_meta: ssm_d_inner      = 0
llm_load_print_meta: ssm_d_state      = 0
llm_load_print_meta: ssm_dt_rank      = 0
llm_load_print_meta: model type       = 355B.A32B
llm_load_print_meta: model ftype      = IQ4_KSS - 4.0 bpw
llm_load_print_meta: model params     = 358.338 B
llm_load_print_meta: model size       = 173.726 GiB (4.164 BPW) 
llm_load_print_meta: repeating layers = 172.721 GiB (4.158 BPW, 356.786 B parameters)
llm_load_print_meta: general.name     = GLM 4.5
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
llm_load_tensors: ggml ctx size =    0.72 MiB
model has unused tensor blk.92.attn_norm.weight (size = 20480 bytes) -- ignoring
model has unused tensor blk.92.attn_q.weight (size = 41336832 bytes) -- ignoring
model has unused tensor blk.92.attn_k.weight (size = 4341760 bytes) -- ignoring
model has unused tensor blk.92.attn_v.weight (size = 4341760 bytes) -- ignoring
model has unused tensor blk.92.attn_q.bias (size = 49152 bytes) -- ignoring
model has unused tensor blk.92.attn_k.bias (size = 4096 bytes) -- ignoring
model has unused tensor blk.92.attn_v.bias (size = 4096 bytes) -- ignoring
model has unused tensor blk.92.attn_output.weight (size = 41308160 bytes) -- ignoring
model has unused tensor blk.92.attn_q_norm.weight (size = 512 bytes) -- ignoring
model has unused tensor blk.92.attn_k_norm.weight (size = 512 bytes) -- ignoring
model has unused tensor blk.92.post_attention_norm.weight (size = 20480 bytes) -- ignoring
model has unused tensor blk.92.ffn_gate_inp.weight (size = 3276800 bytes) -- ignoring
model has unused tensor blk.92.exp_probs_b.bias (size = 640 bytes) -- ignoring
model has unused tensor blk.92.ffn_gate_exps.weight (size = 630128640 bytes) -- ignoring
model has unused tensor blk.92.ffn_down_exps.weight (size = 671744000 bytes) -- ignoring
model has unused tensor blk.92.ffn_up_exps.weight (size = 630128640 bytes) -- ignoring
model has unused tensor blk.92.ffn_gate_shexp.weight (size = 4184064 bytes) -- ignoring
model has unused tensor blk.92.ffn_down_shexp.weight (size = 5181440 bytes) -- ignoring
model has unused tensor blk.92.ffn_up_shexp.weight (size = 4184064 bytes) -- ignoring
model has unused tensor blk.92.nextn.eh_proj.weight (size = 55705600 bytes) -- ignoring
model has unused tensor blk.92.nextn.embed_tokens.weight (size = 509820928 bytes) -- ignoring
model has unused tensor blk.92.nextn.enorm.weight (size = 20480 bytes) -- ignoring
model has unused tensor blk.92.nextn.hnorm.weight (size = 20480 bytes) -- ignoring
model has unused tensor blk.92.nextn.shared_head_head.weight (size = 509820928 bytes) -- ignoring
model has unused tensor blk.92.nextn.shared_head_norm.weight (size = 20480 bytes) -- ignoring
impl: using THP with page size 2 MiB ...................... done
impl: using THP with page size 2 MiB ...................... done
impl: using THP with page size 2 MiB ...................... done
impl: using THP with page size 2 MiB ...................... done
llm_load_tensors: offloading 0 repeating layers to GPU
llm_load_tensors: offloaded 0/94 layers to GPU
llm_load_tensors:        CPU buffer size = 44433.12 MiB
llm_load_tensors:        CPU buffer size = 44752.02 MiB
llm_load_tensors:        CPU buffer size = 44752.02 MiB
llm_load_tensors:        CPU buffer size = 41959.55 MiB
....................................................................................................
=====================================================================
 MLA is only available for LLM_ARCH_DEEPSEEK2 -> turning off MLA
=====================================================================
llama_new_context_with_model: n_ctx      = 131072
llama_new_context_with_model: n_batch    = 16384
llama_new_context_with_model: n_ubatch   = 1024
llama_new_context_with_model: flash_attn = 1
llama_new_context_with_model: mla_attn   = 0
llama_new_context_with_model: attn_max_b = 2048
llama_new_context_with_model: fused_moe  = 1
llama_new_context_with_model: fused_up_gate = 1
llama_new_context_with_model: ser        = -1, 0
llama_new_context_with_model: freq_base  = 1000000.0
llama_new_context_with_model: freq_scale = 1
llama_kv_cache_init:        CPU KV buffer size = 47616.00 MiB
llama_new_context_with_model: KV self size  = 47616.00 MiB, K (f16): 23808.00 MiB, V (f16): 23808.00 MiB
llama_new_context_with_model:        CPU  output buffer size =     1.73 MiB
ggml_gallocr_reserve_n: reallocating CPU buffer from size 0.00 MiB to 912.01 MiB
llama_new_context_with_model:        CPU compute buffer size =   912.01 MiB
llama_new_context_with_model: graph nodes  = 4273
llama_new_context_with_model: graph splits = 1
INFO [              load_model] loading draft model | tid="139870314715392" timestamp=1759184993 model="/home/trunk/Public/AI/models/jukofyork/GLM-4.5-DRAFT-0.6B-v3.0-GGUF/GLM-4.5-DRAFT-0.6B-32k-Q4_0.gguf"
llama_model_loader: loaded meta data with 23 key-value pairs and 291 tensors from /home/trunk/Public/AI/models/jukofyork/GLM-4.5-DRAFT-0.6B-v3.0-GGUF/GLM-4.5-DRAFT-0.6B-32k-Q4_0.gguf (version GGUF V3 (latest))
llama_model_loader: Dumping metadata keys/values. Note: KV overrides do not apply in this output.
llama_model_loader: - kv   0:                       general.architecture str              = qwen2
llama_model_loader: - kv   1:                               general.type str              = model
llama_model_loader: - kv   2:                               general.name str              = GLM 4.5 DRAFT 0.6B
llama_model_loader: - kv   3:                           general.basename str              = GLM-4.5-DRAFT
llama_model_loader: - kv   4:                         general.size_label str              = 0.6B
llama_model_loader: - kv   5:                          qwen2.block_count u32              = 24
llama_model_loader: - kv   6:                       qwen2.context_length u32              = 32768
llama_model_loader: - kv   7:                     qwen2.embedding_length u32              = 896
llama_model_loader: - kv   8:                  qwen2.feed_forward_length u32              = 4864
llama_model_loader: - kv   9:                 qwen2.attention.head_count u32              = 14
llama_model_loader: - kv  10:              qwen2.attention.head_count_kv u32              = 2
llama_model_loader: - kv  11:                       qwen2.rope.freq_base f32              = 1000000.000000
llama_model_loader: - kv  12:     qwen2.attention.layer_norm_rms_epsilon f32              = 0.000001
llama_model_loader: - kv  13:                       tokenizer.ggml.model str              = gpt2
llama_model_loader: - kv  14:                         tokenizer.ggml.pre str              = glm4
llama_model_loader: - kv  15:                      tokenizer.ggml.tokens arr[str,151552]  = ["!", "\"", "#", "$", "%", "&", "'", ...
llama_model_loader: - kv  16:                  tokenizer.ggml.token_type arr[i32,151552]  = [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, ...
llama_model_loader: - kv  17:                      tokenizer.ggml.merges arr[str,318088]  = ["Ġ Ġ", "Ġ ĠĠĠ", "ĠĠ ĠĠ", "...
llama_model_loader: - kv  18:                tokenizer.ggml.eos_token_id u32              = 151329
llama_model_loader: - kv  19:            tokenizer.ggml.padding_token_id u32              = 151329
llama_model_loader: - kv  20:                tokenizer.ggml.bos_token_id u32              = 151331
llama_model_loader: - kv  21:               general.quantization_version u32              = 2
llama_model_loader: - kv  22:                          general.file_type u32              = 2
llama_model_loader: - type  f32:  121 tensors
llama_model_loader: - type q4_0:  169 tensors
llama_model_loader: - type q8_0:    1 tensors
init_tokenizer: initializing tokenizer for type 2
load: printing all EOG tokens:
load:   - 151329 ('<|endoftext|>')
load: special tokens cache size = 36
load: token to piece cache size = 0.9713 MB
llm_load_print_meta: format           = GGUF V3 (latest)
llm_load_print_meta: arch             = qwen2
llm_load_print_meta: n_ctx_train      = 32768
llm_load_print_meta: n_embd           = 896
llm_load_print_meta: n_layer          = 24
llm_load_print_meta: n_head           = 14
llm_load_print_meta: n_head_kv        = 2
llm_load_print_meta: n_rot            = 64
llm_load_print_meta: n_swa            = 0
llm_load_print_meta: n_swa_pattern    = 1
llm_load_print_meta: n_embd_head_k    = 64
llm_load_print_meta: n_embd_head_v    = 64
llm_load_print_meta: n_gqa            = 7
llm_load_print_meta: n_embd_k_gqa     = 128
llm_load_print_meta: n_embd_v_gqa     = 128
llm_load_print_meta: f_norm_eps       = 0.0e+00
llm_load_print_meta: f_norm_rms_eps   = 1.0e-06
llm_load_print_meta: f_clamp_kqv      = 0.0e+00
llm_load_print_meta: f_max_alibi_bias = 0.0e+00
llm_load_print_meta: f_logit_scale    = 0.0e+00
llm_load_print_meta: n_ff             = 4864
llm_load_print_meta: n_expert         = 0
llm_load_print_meta: n_expert_used    = 0
llm_load_print_meta: causal attn      = 1
llm_load_print_meta: pooling type     = 0
llm_load_print_meta: rope type        = 2
llm_load_print_meta: rope scaling     = linear
llm_load_print_meta: freq_base_train  = 1000000.0
llm_load_print_meta: freq_scale_train = 1
llm_load_print_meta: n_ctx_orig_yarn  = 32768
llm_load_print_meta: rope_finetuned   = unknown
llm_load_print_meta: ssm_d_conv       = 0
llm_load_print_meta: ssm_d_inner      = 0
llm_load_print_meta: ssm_d_state      = 0
llm_load_print_meta: ssm_dt_rank      = 0
llm_load_print_meta: model type       = 1B
llm_load_print_meta: model ftype      = Q4_0
llm_load_print_meta: model params     = 629.479 M
llm_load_print_meta: model size       = 402.664 MiB (5.366 BPW) 
llm_load_print_meta: repeating layers = 192.226 MiB (4.505 BPW, 357.898 M parameters)
llm_load_print_meta: general.name     = GLM 4.5 DRAFT 0.6B
print_info: vocab type       = BPE
print_info: n_vocab          = 151552
print_info: n_merges         = 318088
print_info: BOS token        = 151331 '[gMASK]'
print_info: EOS token        = 151329 '<|endoftext|>'
print_info: EOT token        = 151329 '<|endoftext|>'
print_info: PAD token        = 151329 '<|endoftext|>'
print_info: LF token         = 198 'Ċ'
print_info: FIM PRE token    = 151347 '<|code_prefix|>'
print_info: FIM SUF token    = 151349 '<|code_suffix|>'
print_info: FIM MID token    = 151348 '<|code_middle|>'
print_info: EOG token        = 151329 '<|endoftext|>'
print_info: max token length = 1024
llm_load_tensors: ggml ctx size =    0.13 MiB
llm_load_tensors: offloading 0 repeating layers to GPU
llm_load_tensors: offloaded 0/25 layers to GPU
llm_load_tensors:        CPU buffer size =   402.66 MiB
..................................................
llama_new_context_with_model: n_ctx      = 32768
llama_new_context_with_model: n_batch    = 2048
llama_new_context_with_model: n_ubatch   = 512
llama_new_context_with_model: flash_attn = 1
llama_new_context_with_model: mla_attn   = 0
llama_new_context_with_model: attn_max_b = 0
llama_new_context_with_model: fused_moe  = 0
llama_new_context_with_model: fused_up_gate = 1
llama_new_context_with_model: ser        = -1, 0
llama_new_context_with_model: freq_base  = 1000000.0
llama_new_context_with_model: freq_scale = 1
llama_kv_cache_init:        CPU KV buffer size =   384.00 MiB
llama_new_context_with_model: KV self size  =  384.00 MiB, K (f16):  192.00 MiB, V (f16):  192.00 MiB
llama_new_context_with_model:        CPU  output buffer size =     0.58 MiB
ggml_gallocr_reserve_n: reallocating CPU buffer from size 0.00 MiB to 297.75 MiB
llama_new_context_with_model:        CPU compute buffer size =   297.75 MiB
llama_new_context_with_model: graph nodes  = 630
llama_new_context_with_model: graph splits = 1
======================================= HAVE_FANCY_SIMD is NOT defined
llama_speculative_are_compatible: vocab_type tgt: 1
llama_speculative_are_compatible: vocab_type dft: 1
INFO [                    init] initializing slots | tid="139870314715392" timestamp=1759184997 n_slots=2
INFO [                    init] new slot | tid="139870314715392" timestamp=1759184997 id_slot=0 n_ctx_slot=65536
llama_new_context_with_model: n_ctx      = 32768
llama_new_context_with_model: n_batch    = 32768
llama_new_context_with_model: n_ubatch   = 512
llama_new_context_with_model: flash_attn = 1
llama_new_context_with_model: mla_attn   = 0
llama_new_context_with_model: attn_max_b = 0
llama_new_context_with_model: fused_moe  = 0
llama_new_context_with_model: fused_up_gate = 1
llama_new_context_with_model: ser        = -1, 0
llama_new_context_with_model: freq_base  = 1000000.0
llama_new_context_with_model: freq_scale = 1
llama_kv_cache_init:        CPU KV buffer size =   384.00 MiB
llama_new_context_with_model: KV self size  =  384.00 MiB, K (f16):  192.00 MiB, V (f16):  192.00 MiB
llama_new_context_with_model:        CPU  output buffer size =     0.58 MiB
ggml_gallocr_reserve_n: reallocating CPU buffer from size 0.00 MiB to 297.75 MiB
llama_new_context_with_model:        CPU compute buffer size =   297.75 MiB
llama_new_context_with_model: graph nodes  = 630
llama_new_context_with_model: graph splits = 1
llama_speculative_are_compatible: vocab_type tgt: 1
llama_speculative_are_compatible: vocab_type dft: 1
vocab_dft_compatible = 1
INFO [                    init] new slot | tid="139870314715392" timestamp=1759184998 id_slot=1 n_ctx_slot=65536
llama_new_context_with_model: n_ctx      = 32768
llama_new_context_with_model: n_batch    = 32768
llama_new_context_with_model: n_ubatch   = 512
llama_new_context_with_model: flash_attn = 1
llama_new_context_with_model: mla_attn   = 0
llama_new_context_with_model: attn_max_b = 0
llama_new_context_with_model: fused_moe  = 0
llama_new_context_with_model: fused_up_gate = 1
llama_new_context_with_model: ser        = -1, 0
llama_new_context_with_model: freq_base  = 1000000.0
llama_new_context_with_model: freq_scale = 1
llama_kv_cache_init:        CPU KV buffer size =   384.00 MiB
llama_new_context_with_model: KV self size  =  384.00 MiB, K (f16):  192.00 MiB, V (f16):  192.00 MiB
llama_new_context_with_model:        CPU  output buffer size =     0.58 MiB
ggml_gallocr_reserve_n: reallocating CPU buffer from size 0.00 MiB to 297.75 MiB
llama_new_context_with_model:        CPU compute buffer size =   297.75 MiB
llama_new_context_with_model: graph nodes  = 630
llama_new_context_with_model: graph splits = 1
llama_speculative_are_compatible: vocab_type tgt: 1
llama_speculative_are_compatible: vocab_type dft: 1
vocab_dft_compatible = 1
INFO [                    main] model loaded | tid="139870314715392" timestamp=1759184998
INFO [                    main] chat template | tid="139870314715392" timestamp=1759184998 chat_template="[gMASK]<sop>\n{%- if tools -%}\n<|system|>\n# Tools\n\nYou may call one or more functions to assist with the user query.\n\nYou are provided with function signatures within <tools></tools> XML tags:\n<tools>\n{% for tool in tools %}\n{{ tool | tojson(ensure_ascii=False) }}\n{% endfor %}\n</tools>\n\nFor each function call, output the function name and arguments within the following XML format:\n<tool_call>{function-name}\n<arg_key>{arg-key-1}</arg_key>\n<arg_value>{arg-value-1}</arg_value>\n<arg_key>{arg-key-2}</arg_key>\n<arg_value>{arg-value-2}</arg_value>\n...\n</tool_call>{%- endif -%}\n{%- macro visible_text(content) -%}\n    {%- if content is string -%}\n        {{- content }}\n    {%- elif content is iterable and content is not mapping -%}\n        {%- for item in content -%}\n            {%- if item is mapping and item.type == 'text' -%}\n                {{- item.text }}\n            {%- elif item is string -%}\n                {{- item }}\n            {%- endif -%}\n        {%- endfor -%}\n    {%- else -%}\n        {{- content }}\n    {%- endif -%}\n{%- endmacro -%}\n{%- set ns = namespace(last_user_index=-1) %}\n{%- for m in messages %}\n    {%- if m.role == 'user' %}\n        {% set ns.last_user_index = loop.index0 -%}\n    {%- endif %}\n{%- endfor %}\n{% for m in messages %}\n{%- if m.role == 'user' -%}<|user|>\n{% set content = visible_text(m.content) %}{{ content }}\n{{- '/nothink' if (enable_thinking is defined and not enable_thinking and not content.endswith(\"/nothink\")) else '' -}}\n{%- elif m.role == 'assistant' -%}\n<|assistant|>\n{%- set reasoning_content = '' %}\n{%- set content = visible_text(m.content) %}\n{%- if m.reasoning_content is string %}\n    {%- set reasoning_content = m.reasoning_content %}\n{%- else %}\n    {%- if '</think>' in content %}\n        {%- set reasoning_content = content.split('</think>')[0].rstrip('\\n').split('<think>')[-1].lstrip('\\n') %}\n        {%- set content = content.split('</think>')[-1].lstrip('\\n') %}\n    {%- endif %}\n{%- endif %}\n{%- if loop.index0 > ns.last_user_index and reasoning_content -%}\n{{ '\\n<think>' + reasoning_content.strip() +  '</think>'}}\n{%- else -%}\n{{ '\\n<think></think>' }}\n{%- endif -%}\n{%- if content.strip() -%}\n{{ '\\n' + content.strip() }}\n{%- endif -%}\n{% if m.tool_calls %}\n{% for tc in m.tool_calls %}\n{%- if tc.function %}\n    {%- set tc = tc.function %}\n{%- endif %}\n{{ '\\n<tool_call>' + tc.name }}\n{% set _args = tc.arguments %}\n{% for k, v in _args.items() %}\n<arg_key>{{ k }}</arg_key>\n<arg_value>{{ v | tojson(ensure_ascii=False) if v is not string else v }}</arg_value>\n{% endfor %}\n</tool_call>{% endfor %}\n{% endif %}\n{%- elif m.role == 'tool' -%}\n{%- if m.content is string -%}\n{%- if loop.first or (messages[loop.index0 - 1].role != \"tool\") %}\n    {{- '<|observation|>' }}\n{%- endif %}\n{{- '\\n<tool_response>\\n' }}\n{{- m.content }}\n{{- '\\n</tool_response>' }}\n{%- else -%}\n<|observation|>{% for tr in m.content %}\n\n<tool_response>\n{{ tr.output if tr.output is defined else tr }}\n</tool_response>{% endfor -%}\n{% endif -%}\n{%- elif m.role == 'system' -%}\n<|system|>\n{{ visible_text(m.content) }}\n{%- endif -%}\n{%- endfor -%}\n{%- if add_generation_prompt -%}\n    <|assistant|>{{- '\\n<think></think>' if (enable_thinking is defined and not enable_thinking) else '' -}}\n{%- endif -%}"
INFO [                    main] chat template | tid="139870314715392" timestamp=1759184998 chat_example="[gMASK]<sop><|system|>\nYou are a helpful assistant<|user|>\nHello<|assistant|>\nHi there<|user|>\nHow are you?<|assistant|>" built_in=true
INFO [                    main] HTTP server listening | tid="139870314715392" timestamp=1759184998 n_threads_http="31" port="8080" hostname="0.0.0.0"
INFO [            update_slots] all slots are idle | tid="139870314715392" timestamp=1759184998
Grammar: 
Grammar lazy: false
Chat format: Content-only
INFO [   launch_slot_with_task] slot is processing task | tid="139870314715392" timestamp=1759185009 id_slot=0 id_task=0
INFO [            update_slots] kv cache rm [p0, end) | tid="139870314715392" timestamp=1759185009 id_slot=0 id_task=0 p0=0
llama_speculative_gen_draft: reuse_i = 0, reuse_n = 0, prompt = 0
llama_speculative_gen_draft: reuse_i = 0, reuse_n = 8, prompt = 8
llama_speculative_gen_draft: reuse_i = 0, reuse_n = 10, prompt = 10
llama_speculative_gen_draft: reuse_i = 0, reuse_n = 12, prompt = 12
llama_output_reserve: reallocating output buffer from size 1.73 MiB to 2.89 MiB
llama_speculative_gen_draft: reuse_i = 0, reuse_n = 15, prompt = 17
llama_speculative_gen_draft: reuse_i = 0, reuse_n = 17, prompt = 17
INFO [           print_timings] prompt eval time     =   13401.43 ms /     7 tokens ( 1914.49 ms per token,     0.52 tokens per second) | tid="139870314715392" timestamp=1759185081 id_slot=0 id_task=0 t_prompt_processing=13401.43 n_prompt_tokens_processed=7 t_token=1914.49 n_tokens_second=0.5223323182675281
INFO [           print_timings] generation eval time =   58516.98 ms /    15 runs   ( 3901.13 ms per token,     0.26 tokens per second) | tid="139870314715392" timestamp=1759185081 id_slot=0 id_task=0 t_token_generation=58516.979 n_decoded=15 t_token=3901.131933333333 n_tokens_second=0.25633585766619976
INFO [           print_timings]           total time =   71918.41 ms | tid="139870314715392" timestamp=1759185081 id_slot=0 id_task=0 t_prompt_processing=13401.43 t_token_generation=58516.979 t_total=71918.409
llama_speculative_gen_draft: reuse_i = 0, reuse_n = 20, prompt = 20
INFO [      log_server_request] request | tid="139631956203200" timestamp=1759185081 remote_addr="10.0.5.35" remote_port=54820 status=200 method="POST" path="/v1/chat/completions" params={}
INFO [            update_slots] slot released | tid="139870314715392" timestamp=1759185086 id_slot=0 id_task=0 n_ctx=131072 n_past=23 n_system_tokens=0 n_cache_tokens=23 truncated=false
INFO [            update_slots] all slots are idle | tid="139870314715392" timestamp=1759185086
Grammar: 
Grammar lazy: false
Chat format: Content-only
INFO [   launch_slot_with_task] slot is processing task | tid="139870314715392" timestamp=1759185092 id_slot=1 id_task=9
INFO [            update_slots] kv cache rm [p0, end) | tid="139870314715392" timestamp=1759185092 id_slot=1 id_task=9 p0=0
/home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1156: GGML_ASSERT(S > 0) failed
/home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1156: /home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1156: GGML_ASSERT(S > 0) failed
/home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1156: GGML_ASSERT(S > 0) failed
/home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1156: GGML_ASSERT(S > 0) failed
/home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1156: GGML_ASSERT(S > 0) failed
/home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1156: GGML_ASSERT(S > 0) failed
/home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1156: GGML_ASSERT(S > 0) failed
GGML_ASSERT(S > 0) failed
/home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1156: GGML_ASSERT(S > 0) failed
/home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1156: GGML_ASSERT(S > 0) failed
/home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1156: GGML_ASSERT(S > 0) failed
/home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1156: GGML_ASSERT(S > 0) failed
/home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1156: GGML_ASSERT(S > 0) failed
/home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1156: GGML_ASSERT(S > 0) failed
/home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1156: GGML_ASSERT(S > 0) failed
/home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1156: GGML_ASSERT(S > 0) failed
/home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1156: /home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1156: GGML_ASSERT(S > 0) failed
/home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1156: /home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1156: GGML_ASSERT(S > 0) failed
/home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1156: GGML_ASSERT(S > 0) failed
/home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1156: GGML_ASSERT(S > 0) failed
GGML_ASSERT(S > 0) failed
/home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1156: GGML_ASSERT(S > 0) failed
/home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1156: GGML_ASSERT(S > 0) failed
/home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1156: GGML_ASSERT(S > 0) failed
/home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1156: GGML_ASSERT(S > 0) failed
/home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1156: GGML_ASSERT(S > 0) failed
/home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1156: GGML_ASSERT(S > 0) failed
/home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1156: GGML_ASSERT(S > 0) failed
GGML_ASSERT(S > 0) failed
/home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1156: GGML_ASSERT(S > 0) failed
[New LWP 41188]
[New LWP 41187]
[New LWP 41186]
[New LWP 41185]
[New LWP 41184]
[New LWP 41183]
[New LWP 41182]
[New LWP 41181]
[New LWP 41180]
[New LWP 41179]
[New LWP 41178]
[New LWP 41177]
[New LWP 41176]
[New LWP 41175]
[New LWP 41174]
[New LWP 41173]
[New LWP 41172]
[New LWP 41171]
[New LWP 41170]
[New LWP 41169]
[New LWP 41168]
[New LWP 41167]
[New LWP 41166]
[New LWP 41165]
[New LWP 41164]
[New LWP 41163]
[New LWP 41162]
[New LWP 41161]
[New LWP 41160]
[New LWP 41159]
[New LWP 41158]
[New LWP 41157]
[New LWP 41152]
[New LWP 41151]
[New LWP 41150]
[New LWP 41149]
[New LWP 41148]
[New LWP 41147]
[New LWP 41146]
[New LWP 41145]
[New LWP 41144]
[New LWP 41143]
[New LWP 41142]
[New LWP 41141]
[New LWP 41140]
[New LWP 41139]
[New LWP 41138]
[New LWP 41137]
[New LWP 41136]
[New LWP 41135]
[New LWP 41134]
[New LWP 41133]
[New LWP 41132]
[New LWP 41131]
[New LWP 41130]
[New LWP 41129]
[New LWP 41128]
[New LWP 41127]
[New LWP 41126]
[New LWP 41125]
[New LWP 41124]
warning: process 40455 is already traced by process 41344
ptrace: Operation not permitted.
No stack.
The program is not being run.
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/usr/lib64/libthread_db.so.1".
0x00007f3618204fda in __lll_lock_wait_private () from /usr/lib64/libc.so.6
#0  0x00007f3618204fda in __lll_lock_wait_private () from /usr/lib64/libc.so.6
#1  0x00007f361825fa69 in ?? () from /usr/lib64/libc.so.6
#2  0x00007f36182494d7 in fork () from /usr/lib64/libc.so.6
warning: process 40455 is already traced by process 41344
ptrace: Operation not permitted.
No stack.
The program is not being run.
#3  0x00007f36187abcb0 in ggml_print_backtrace () at /home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/ggml.c:223
223	    int pid = fork();
#4  0x00007f36187abf07 in ggml_abort (file=0x7f36192849b8 "/home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h", line=1156, fmt=0x7f361928499a "GGML_ASSERT(%s) failed") at /home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/ggml.c:269
269	    ggml_print_backtrace();
#5  0x00007f3618b8d61c in (anonymous namespace)::FlashQKV<128, 8, 128>::normalize_and_store_1row<(anonymous namespace)::FlashMS<8, 128> > (this=0x7fffcd253fa0, fms=..., j=0, R=0x7fffcd253fa0, qkv=0x7f15491d5020, sinkf=0x0) at /home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1156
1156	        GGML_ASSERT(S > 0);
#6  0x00007f3618b4a184 in (anonymous namespace)::FlashQKV<128, 8, 128>::normalize_and_store<(anonymous namespace)::FlashMS<8, 128> > (this=0x7fffcd253fa0, fms=..., stride_qkv=12288, qkv=0x7f15491d5020, sinkf=0x0, M=0x0, S=0x0) at /home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1214
1214	                normalize_and_store_1row(fms, j, R, qkv, sinkf);
#7  0x00007f3618afa076 in (anonymous namespace)::compute_helper<128, 128, 8, 128, (anonymous namespace)::HelperF16, (anonymous namespace)::HelperF16, (anonymous namespace)::FlashQKfp32<128, 8, 128> > (kh=..., vh=..., nq1=16, nk1=256, stride_q=12288, stride_m=512, stride_qkv=12288, fms=..., fqkv=..., q=0x7f154ddd5020, mask=0x7f1550dd5020 "", qkv=0x7f15491d5020, sinkf=0x0, M=0x0, S=0x0) at /home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1373
1373	        fqkv.normalize_and_store(fms, stride_qkv, qkv, sinkf, M, S);
#8  0x00007f3618ad6cb9 in (anonymous namespace)::FlashAttn<128, 128, 8, 128>::compute<(anonymous namespace)::HelperF16, (anonymous namespace)::HelperF16> (this=0x7fffcd252ee0, kh=..., vh=..., nq1=16, nk1=256, stride_q=12288, stride_m=512, stride_qkv=12288, q=0x7f154ddd5020, mask=0x7f1550dd5020 "", qkv=0x7f15491d5020, M=0x0, S=0x0) at /home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1586
1586	            compute_helper<Dk, Dv, q_step, k_step, KHelper, VHelper, FlashQKfp32<Dk, q_step, k_step>>(
#9  0x00007f3618ab90e2 in (anonymous namespace)::iqk_flash_helper<128, 128, 128, (anonymous namespace)::HelperF16, (anonymous namespace)::HelperF16> (kh=..., vh=..., nq1=23, nk1=256, stride_q=12288, stride_m=512, stride_qkv=12288, q=0x7f154ddd5020, mask=0x7f1550dd5020 "", scale=0.0883883461, softcap=0, qkv=0x7f15491d5020, sinkf=0x0, M=0x0, S=0x0) at /home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:2074
2074	        fa.compute(kh, vh, 8*n_step, nk1, stride_q, stride_m, stride_qkv, q, (const char *)mask, qkv, M, S);
#10 0x00007f3618ab67b2 in (anonymous namespace)::iqk_flash_helper_T<128, 128, 128, (anonymous namespace)::HelperF16> (kh=..., type_v=GGML_TYPE_F16, nq1=23, nk1=256, stride_q=12288, stride_v=2048, stride_m=512, stride_qkv=12288, q=0x7f154ddd5020, v=0x7eff17fff020 "=\227r\t]\025>\025\302\222\232\230\212\225c\233\363\005U\225G\f\370\aW\227\a\230N\020\263\211\177\222\203\022:\021\220\vޛ9\231\275\231\327\024\321\002s\224\342\224\347\225ω\215\225\307\030\343\225\350\235{\220\n\025\265\017-\215\263\2204\f\277\213M\033x\025\224\025u\0165\223E\2250\022\337\030\250\231?\025\375\230\266\020\226\bO\234\225\032`\034a\222o\226\026\220B\021\214\224\377\016\200\030\257\224[\020Y\024\341\fW\226N\214,\030\r\031d\025&\030u\r\246\032\343\227\\\214Y\002ݒ\274\226\310\021\341\016\033\023Q\023`\222̔)\223k\022z\233\235\024/\033\251\f\235\030\362\213\254\021\204\224|\223\246\230\221\025\221\031"..., mask=0x7f1550dd5020 "", scale=0.0883883461, softcap=0, qkv=0x7f15491d5020, sinkf=0x0, M=0x0, S=0x0) at /home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:2131
2131	            iqk_flash_helper<Dk, Dv, k_step>(kh, vh, nq1, nk1, stride_q, stride_m, stride_qkv, q, mask, scale, softcap, qkv, sinkf, M, S);
#11 0x00007f3618ab5c80 in (anonymous namespace)::iqk_flash_helper_T<128, 128, 128> (type_k=GGML_TYPE_F16, type_v=GGML_TYPE_F16, nq1=23, nk1=256, stride_q=12288, stride_k=2048, stride_v=2048, stride_m=512, stride_qkv=12288, q=0x7f154ddd5020, k=0x7eff07fff020 "\237(\353-K\246G9 \273N\243\364\255\f\256\\\224o+{*\\\253\3359ۑ\253)\231\035\250,\202\267\275\031\250\031\212\243\200\031J\226\354\"\002\241״\030\257\024\203\006\034\366\235&\230\222\031\241:o:\345!\350\245Ũ|8\376\256a#\237\264\232.\t\251|\022\\\025f\210ҧ\f\"+!\2427\037 \271\2168\t\210\017d\234T\235\200\217U'\230\301X\023Ù?(P\020\036\242\203\245\a\222\267\224\004+\206\235X%E)\346\223ךV$D\276ՙ-\237\b'@\024\205\302,%\\\030?,\327\030_\236A\226\300!\355,\024!*\243d\"E\223\210\257N\037^\0234\002\251\027\221\232Q\250\024%"..., v=0x7eff17fff020 "=\227r\t]\025>\025\302\222\232\230\212\225c\233\363\005U\225G\f\370\aW\227\a\230N\020\263\211\177\222\203\022:\021\220\vޛ9\231\275\231\327\024\321\002s\224\342\224\347\225ω\215\225\307\030\343\225\350\235{\220\n\025\265\017-\215\263\2204\f\277\213M\033x\025\224\025u\0165\223E\2250\022\337\030\250\231?\025\375\230\266\020\226\bO\234\225\032`\034a\222o\226\026\220B\021\214\224\377\016\200\030\257\224[\020Y\024\341\fW\226N\214,\030\r\031d\025&\030u\r\246\032\343\227\\\214Y\002ݒ\274\226\310\021\341\016\033\023Q\023`\222̔)\223k\022z\233\235\024/\033\251\f\235\030\362\213\254\021\204\224|\223\246\230\221\025\221\031"..., mask=0x7f1550dd5020 "", scale=0.0883883461, softcap=0, qkv=0x7f15491d5020, sinkf=0x0, M=0x0, S=0x0) at /home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:2180
2180	            result = iqk_flash_helper_T<Dk, Dv, k_step>(kh, type_v, nq1, nk1, stride_q, stride_v, stride_m, stride_qkv, q, v, mask, scale, softcap, qkv, sinkf, M, S);
#12 0x00007f3618ab5a05 in iqk_fa_128_128 (int_type_k=1, int_type_v=1, nq=23, nk=256, stride_q=12288, stride_k=2048, stride_v=2048, stride_m=512, stride_qkv=12288, q=0x7f154ddd5020, k=0x7eff07fff020, v=0x7eff17fff020, mask=0x7f1550dd5020, scale=0.0883883461, softcap=0, qkv=0x7f15491d5020, sinkf=0x0, M=0x0, S=0x0) at /home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/iqk/fa/iqk_fa_128_128.cpp:32
32	        return iqk_flash_helper_T<128, 128, 128>(type_k, type_v, nq, nk, stride_q, stride_k, stride_v, stride_m, stride_qkv,
#13 0x00007f3618886dc6 in iqk_flash_attn_impl (int_type_k=1, int_type_v=1, Dk=128, Dv=128, nq1=23, nk1=256, stride_q=49152, stride_k=2048, stride_v=2048, stride_m=512, stride_qkv=12288, q=0x7f154ddd5020, k=0x7eff07fff020, v=0x7eff17fff020, mask=0x7f1550dd5020, sinksf=0x0, nsinks=1, scale=0.0883883461, softcap=0, qkv=0x7f15491d5020, M=0x0, S=0x0) at /home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:1357
1357	        return iqk_fa_128_128(int_type_k, int_type_v, nq1, nk1, stride_q, stride_k, stride_v, stride_m, stride_qkv,
#14 0x00007f361888a25e in iqk_flash_attn_noalibi (type_q=0, type_mask=1, max_bias=0, neq3=1, neq2=96, nbq3=5505024, nbq2=512, nek3=1, nek2=8, nbk3=2048, nbk2=256, nev3=1, nev2=8, nbv3=2048, nbv2=256, ne2=112, ne1=96, nb1=512, int_type_k_in=1, int_type_v=1, Dk=128, Dv=128, neq1=112, nek1=256, stride_q=49152, stride_k=2048, stride_v=2048, stride_m=512, q=0x7f154ddd5020, k=0x7eff07fff020, v=0x7eff17fff020, mask=0x7f1550dd5020, sinks=0x0, scale=0.0883883461, softcap=0, qkv=0x7f15491d5020, work_buffer_in=0x7efdd28dc010, barrier=0x7f36187b26c8 <ggml_barrier>, barrier_data=0x7fffcd264030, ith=0, nth=30, n_swa=0) at /home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/iqk/iqk_flash_attn.cpp:315
315	                if (!iqk_flash_attn_impl(int_type_k, int_type_v,
#15 0x00007f36187eecd1 in ggml_compute_forward_flash_attn_ext_f16 (params=0x7fffcd263f40, dst=0x7f1561f016f0) at /home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/ggml.c:19903
19903	    if (iqk_flash_attn_noalibi(q->type, mask->type, max_bias,
#16 0x00007f36187ef90a in ggml_compute_forward_flash_attn_ext (params=0x7fffcd263f40, dst=0x7f1561f016f0) at /home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/ggml.c:20116
20116	                ggml_compute_forward_flash_attn_ext_f16(params, dst);
#17 0x00007f36187f729f in ggml_compute_forward (params=0x7fffcd263f40, tensor=0x7f1561f016f0, cgraph=0x55baf4fe5e88, i=22) at /home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/ggml.c:22319
22319	                ggml_compute_forward_flash_attn_ext(params, tensor);
#18 0x00007f36187fd360 in ggml_graph_compute_thread (data=0x7fffcd263f90) at /home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/ggml.c:24428
24428	        node_n = ggml_compute_forward(&params, node, cgraph, node_n);
#19 0x00007f3618809bac in ggml_graph_compute._omp_fn.0 () at /home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/ggml.c:24490
24490	            ggml_graph_compute_thread(&worker);
#20 0x00007f3618137566 in GOMP_parallel () from /usr/lib/gcc/x86_64-pc-linux-gnu/14/libgomp.so.1
#21 0x00007f36187fd549 in ggml_graph_compute (cgraph=0x55baf4fe5e88, cplan=0x7fffcd2640a0) at /home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/ggml.c:24476
24476	        #pragma omp parallel num_threads(n_threads)
#22 0x00007f361880f67e in ggml_backend_cpu_graph_compute (backend=0x55baf59afa00, cgraph=0x55baf4fe5e88) at /home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/ggml-backend.cpp:879
879	    return ggml_graph_compute(cgraph, &cplan);
#23 0x00007f361880e589 in ggml_backend_graph_compute_async (backend=0x55baf59afa00, cgraph=0x55baf4fe5e88) at /home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/ggml-backend.cpp:319
319	    return backend->iface.graph_compute(backend, cgraph);
#24 0x00007f36188139fe in ggml_backend_sched_compute_splits (sched=0x55baf50b6140) at /home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/ggml-backend.cpp:1998
1998	            enum ggml_status ec = ggml_backend_graph_compute_async(split_backend, &split->graph);
#25 0x00007f36188146f5 in ggml_backend_sched_graph_compute_async (sched=0x55baf50b6140, graph=0x7f1561cfb030) at /home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/ggml-backend.cpp:2192
2192	    return ggml_backend_sched_compute_splits(sched);
#26 0x00007f361974eef3 in llama_graph_compute (lctx=..., gf=0x7f1561cfb030, n_threads=30) at /home/trunk/Public/AI/ikawrakow/ik_llama.cpp/src/llama.cpp:16761
16761	    ggml_backend_sched_graph_compute_async(lctx.sched, gf);
#27 0x00007f361974fb78 in llama_decode_internal (lctx=..., batch_all=...) at /home/trunk/Public/AI/ikawrakow/ik_llama.cpp/src/llama.cpp:16980
16980	        llama_graph_compute(lctx, gf, n_threads);
#28 0x00007f361976235c in llama_decode (ctx=0x55baf5ebb290, batch=...) at /home/trunk/Public/AI/ikawrakow/ik_llama.cpp/src/llama.cpp:21515
21515	    const int ret = llama_decode_internal(*ctx, batch);
#29 0x000055baf0045958 in server_context::update_slots (this=0x7fffcd2666c0) at /home/trunk/Public/AI/ikawrakow/ik_llama.cpp/examples/server/server.cpp:3175
3175	            const int ret = llama_decode(ctx, batch_view);
#30 0x000055baf0108db1 in std::__invoke_impl<void, void (server_context::*&)(), server_context*&> (__f=@0x55bafea057e0: (void (server_context::*)(struct server_context * const)) 0x55baf003fad0 <server_context::update_slots()>, __t=@0x55bafea057f0: 0x7fffcd2666c0) at /usr/lib/gcc/x86_64-pc-linux-gnu/14/include/g++-v14/bits/invoke.h:74
74	      return ((*std::forward<_Tp>(__t)).*__f)(std::forward<_Args>(__args)...);
#31 0x000055baf00fb291 in std::__invoke<void (server_context::*&)(), server_context*&> (__fn=@0x55bafea057e0: (void (server_context::*)(struct server_context * const)) 0x55baf003fad0 <server_context::update_slots()>) at /usr/lib/gcc/x86_64-pc-linux-gnu/14/include/g++-v14/bits/invoke.h:96
96	      return std::__invoke_impl<__type>(__tag{}, std::forward<_Callable>(__fn),
#32 0x000055baf00e8e49 in std::_Bind<void (server_context::*(server_context*))()>::__call<void, , 0ul>(std::tuple<>&&, std::_Index_tuple<0ul>) (this=0x55bafea057e0, __args=...) at /usr/lib/gcc/x86_64-pc-linux-gnu/14/include/g++-v14/functional:513
513		  return std::__invoke(_M_f,
#33 0x000055baf00d94a7 in std::_Bind<void (server_context::*(server_context*))()>::operator()<, void>() (this=0x55bafea057e0) at /usr/lib/gcc/x86_64-pc-linux-gnu/14/include/g++-v14/functional:598
598		  return this->__call<_Result>(
#34 0x000055baf00c4be4 in std::__invoke_impl<void, std::_Bind<void (server_context::*(server_context*))()>&>(std::__invoke_other, std::_Bind<void (server_context::*(server_context*))()>&) (__f=...) at /usr/lib/gcc/x86_64-pc-linux-gnu/14/include/g++-v14/bits/invoke.h:61
61	    { return std::forward<_Fn>(__f)(std::forward<_Args>(__args)...); }
#35 0x000055baf00afe4a in std::__invoke_r<void, std::_Bind<void (server_context::*(server_context*))()>&>(std::_Bind<void (server_context::*(server_context*))()>&) (__fn=...) at /usr/lib/gcc/x86_64-pc-linux-gnu/14/include/g++-v14/bits/invoke.h:111
111		std::__invoke_impl<__type>(__tag{}, std::forward<_Callable>(__fn),
#36 0x000055baf009028b in std::_Function_handler<void (), std::_Bind<void (server_context::*(server_context*))()> >::_M_invoke(std::_Any_data const&) (__functor=...) at /usr/lib/gcc/x86_64-pc-linux-gnu/14/include/g++-v14/bits/std_function.h:290
290		return std::__invoke_r<_Res>(*_Base::_M_get_pointer(__functor),
#37 0x000055baf005371c in std::function<void()>::operator() (this=0x7fffcd267580) at /usr/lib/gcc/x86_64-pc-linux-gnu/14/include/g++-v14/bits/std_function.h:591
591		return _M_invoker(_M_functor, std::forward<_ArgTypes>(__args)...);
#38 0x000055baf001c973 in server_queue::start_loop (this=0x7fffcd267498) at /home/trunk/Public/AI/ikawrakow/ik_llama.cpp/examples/server/server.cpp:1015
1015	            callback_update_slots();
#39 0x000055baeffe1122 in main (argc=42, argv=0x7fffcd267858) at /home/trunk/Public/AI/ikawrakow/ik_llama.cpp/examples/server/server.cpp:5146
5146	    ctx_server.queue_tasks.start_loop();
[Inferior 1 (process 40455) detached]
```

---

## 💬 Conversation

👤 **os360** commented on **2025-09-30** at **01:38:02**

Fails with or without amb/ub/b/mla and with or without the draft model. Works without flash attention at about half speed. 
Fails with --no-warmup -fa -fmoe -thp --parallel 2
but works with  --no-warmup -fa -fmoe -thp

---

👤 **ikawrakow** commented on **2025-09-30** at **06:08:55**

I think this is the same issue as [#805](https://github.com/ikawrakow/ik_llama.cpp/issues/805). Can you try the latest main branch and let me know if it has been solved?

---

👤 **os360** commented on **2025-09-30** at **14:32:35**

Still fails the same with "--parallel 2" with 
$ ./build/bin/llama-server --version
version: 3900 (e94d1a92)
built with cc (Gentoo 14.3.0 p8) 14.3.0 for x86_64-pc-linux-gnu