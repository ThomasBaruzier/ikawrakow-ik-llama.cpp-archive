## 📌 [Issue #675](https://github.com/ikawrakow/ik_llama.cpp/issues/675) - Bug: ik_llama.cpp server crashes when prompted via API. Oops(ggml_compute_forward_sum_rows_f32, ffn_moe_weights_sum-45): found -nan for i1 = 0, i2 = 0, i3 = 0. ne00 = 8

| **Author** | `1aienthusiast` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-08-07 |
| **Updated** | 2025-08-25 |

---

## 📄 Description

### What happened?

The llama-server works fine (also tested in log output) but when connected via external UI (SillyTavern in this case) it processes prompt and then crashes when trying to generate. Works normally with llama.cpp (albeit different quant)

Crashes with IQ4_KSS ( https://huggingface.co/ubergarm/GLM-4.5-Air-GGUF/tree/main/IQ4_KSS ) , works with Q3_K_XL from unsloth ( https://huggingface.co/unsloth/GLM-4.5-Air-GGUF/tree/main/UD-Q3_K_XL )

### Name and Version

./llama-server --version
version: 3829 (dee40cff)
built with cc (Debian 12.2.0-14+deb12u1) 12.2.0 for x86_64-linux-gnu


### What operating system are you seeing the problem on?

Linux

### Relevant log output

```shell
IQ4_KSS-00001-of-00002.gguf (version GGUF V3 (latest))
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
llm_load_print_meta: model ftype      = IQ4_KSS - 4.0 bpw
llm_load_print_meta: model params     = 110.469 B
llm_load_print_meta: model size       = 54.801 GiB (4.261 BPW) 
llm_load_print_meta: repeating layers = 53.997 GiB (4.246 BPW, 109.227 B parameters)
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
llm_load_tensors: ggml ctx size =    0.66 MiB
Tensor blk.3.attn_norm.weight buffer type overriden to CPU
Tensor blk.3.attn_q.weight buffer type overriden to CPU
Tensor blk.3.attn_k.weight buffer type overriden to CPU
Tensor blk.3.attn_v.weight buffer type overriden to CPU
Tensor blk.3.attn_q.bias buffer type overriden to CPU
Tensor blk.3.attn_k.bias buffer type overriden to CPU
Tensor blk.3.attn_v.bias buffer type overriden to CPU
Tensor blk.3.attn_output.weight buffer type overriden to CPU
Tensor blk.3.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.3.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.3.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.3.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.3.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.3.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.3.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.3.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.3.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.3.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.3.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.4.attn_norm.weight buffer type overriden to CPU
Tensor blk.4.attn_q.weight buffer type overriden to CPU
Tensor blk.4.attn_k.weight buffer type overriden to CPU
Tensor blk.4.attn_v.weight buffer type overriden to CPU
Tensor blk.4.attn_q.bias buffer type overriden to CPU
Tensor blk.4.attn_k.bias buffer type overriden to CPU
Tensor blk.4.attn_v.bias buffer type overriden to CPU
Tensor blk.4.attn_output.weight buffer type overriden to CPU
Tensor blk.4.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.4.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.4.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.4.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.4.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.4.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.4.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.4.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.4.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.4.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.4.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.5.attn_norm.weight buffer type overriden to CPU
Tensor blk.5.attn_q.weight buffer type overriden to CPU
Tensor blk.5.attn_k.weight buffer type overriden to CPU
Tensor blk.5.attn_v.weight buffer type overriden to CPU
Tensor blk.5.attn_q.bias buffer type overriden to CPU
Tensor blk.5.attn_k.bias buffer type overriden to CPU
Tensor blk.5.attn_v.bias buffer type overriden to CPU
Tensor blk.5.attn_output.weight buffer type overriden to CPU
Tensor blk.5.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.5.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.5.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.5.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.5.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.5.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.5.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.5.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.5.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.5.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.5.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.6.attn_norm.weight buffer type overriden to CPU
Tensor blk.6.attn_q.weight buffer type overriden to CPU
Tensor blk.6.attn_k.weight buffer type overriden to CPU
Tensor blk.6.attn_v.weight buffer type overriden to CPU
Tensor blk.6.attn_q.bias buffer type overriden to CPU
Tensor blk.6.attn_k.bias buffer type overriden to CPU
Tensor blk.6.attn_v.bias buffer type overriden to CPU
Tensor blk.6.attn_output.weight buffer type overriden to CPU
Tensor blk.6.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.6.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.6.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.6.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.6.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.6.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.6.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.6.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.6.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.6.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.6.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.7.attn_norm.weight buffer type overriden to CPU
Tensor blk.7.attn_q.weight buffer type overriden to CPU
Tensor blk.7.attn_k.weight buffer type overriden to CPU
Tensor blk.7.attn_v.weight buffer type overriden to CPU
Tensor blk.7.attn_q.bias buffer type overriden to CPU
Tensor blk.7.attn_k.bias buffer type overriden to CPU
Tensor blk.7.attn_v.bias buffer type overriden to CPU
Tensor blk.7.attn_output.weight buffer type overriden to CPU
Tensor blk.7.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.7.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.7.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.7.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.7.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.7.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.7.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.7.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.7.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.7.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.7.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.8.attn_norm.weight buffer type overriden to CPU
Tensor blk.8.attn_q.weight buffer type overriden to CPU
Tensor blk.8.attn_k.weight buffer type overriden to CPU
Tensor blk.8.attn_v.weight buffer type overriden to CPU
Tensor blk.8.attn_q.bias buffer type overriden to CPU
Tensor blk.8.attn_k.bias buffer type overriden to CPU
Tensor blk.8.attn_v.bias buffer type overriden to CPU
Tensor blk.8.attn_output.weight buffer type overriden to CPU
Tensor blk.8.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.8.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.8.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.8.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.8.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.8.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.8.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.8.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.8.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.8.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.8.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.9.attn_norm.weight buffer type overriden to CPU
Tensor blk.9.attn_q.weight buffer type overriden to CPU
Tensor blk.9.attn_k.weight buffer type overriden to CPU
Tensor blk.9.attn_v.weight buffer type overriden to CPU
Tensor blk.9.attn_q.bias buffer type overriden to CPU
Tensor blk.9.attn_k.bias buffer type overriden to CPU
Tensor blk.9.attn_v.bias buffer type overriden to CPU
Tensor blk.9.attn_output.weight buffer type overriden to CPU
Tensor blk.9.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.9.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.9.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.9.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.9.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.9.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.9.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.9.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.9.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.9.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.9.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.10.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.10.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.10.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.11.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.11.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.11.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.12.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.12.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.12.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.13.attn_norm.weight buffer type overriden to CPU
Tensor blk.13.attn_q.weight buffer type overriden to CPU
Tensor blk.13.attn_k.weight buffer type overriden to CPU
Tensor blk.13.attn_v.weight buffer type overriden to CPU
Tensor blk.13.attn_q.bias buffer type overriden to CPU
Tensor blk.13.attn_k.bias buffer type overriden to CPU
Tensor blk.13.attn_v.bias buffer type overriden to CPU
Tensor blk.13.attn_output.weight buffer type overriden to CPU
Tensor blk.13.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.13.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.13.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.13.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.13.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.13.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.13.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.13.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.13.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.13.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.13.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.14.attn_norm.weight buffer type overriden to CPU
Tensor blk.14.attn_q.weight buffer type overriden to CPU
Tensor blk.14.attn_k.weight buffer type overriden to CPU
Tensor blk.14.attn_v.weight buffer type overriden to CPU
Tensor blk.14.attn_q.bias buffer type overriden to CPU
Tensor blk.14.attn_k.bias buffer type overriden to CPU
Tensor blk.14.attn_v.bias buffer type overriden to CPU
Tensor blk.14.attn_output.weight buffer type overriden to CPU
Tensor blk.14.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.14.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.14.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.14.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.14.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.14.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.14.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.14.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.14.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.14.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.14.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.15.attn_norm.weight buffer type overriden to CPU
Tensor blk.15.attn_q.weight buffer type overriden to CPU
Tensor blk.15.attn_k.weight buffer type overriden to CPU
Tensor blk.15.attn_v.weight buffer type overriden to CPU
Tensor blk.15.attn_q.bias buffer type overriden to CPU
Tensor blk.15.attn_k.bias buffer type overriden to CPU
Tensor blk.15.attn_v.bias buffer type overriden to CPU
Tensor blk.15.attn_output.weight buffer type overriden to CPU
Tensor blk.15.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.15.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.15.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.15.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.15.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.15.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.15.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.15.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.15.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.15.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.15.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.16.attn_norm.weight buffer type overriden to CPU
Tensor blk.16.attn_q.weight buffer type overriden to CPU
Tensor blk.16.attn_k.weight buffer type overriden to CPU
Tensor blk.16.attn_v.weight buffer type overriden to CPU
Tensor blk.16.attn_q.bias buffer type overriden to CPU
Tensor blk.16.attn_k.bias buffer type overriden to CPU
Tensor blk.16.attn_v.bias buffer type overriden to CPU
Tensor blk.16.attn_output.weight buffer type overriden to CPU
Tensor blk.16.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.16.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.16.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.16.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.16.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.16.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.16.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.16.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.16.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.16.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.16.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.17.attn_norm.weight buffer type overriden to CPU
Tensor blk.17.attn_q.weight buffer type overriden to CPU
Tensor blk.17.attn_k.weight buffer type overriden to CPU
Tensor blk.17.attn_v.weight buffer type overriden to CPU
Tensor blk.17.attn_q.bias buffer type overriden to CPU
Tensor blk.17.attn_k.bias buffer type overriden to CPU
Tensor blk.17.attn_v.bias buffer type overriden to CPU
Tensor blk.17.attn_output.weight buffer type overriden to CPU
Tensor blk.17.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.17.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.17.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.17.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.17.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.17.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.17.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.17.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.17.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.17.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.17.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.18.attn_norm.weight buffer type overriden to CPU
Tensor blk.18.attn_q.weight buffer type overriden to CPU
Tensor blk.18.attn_k.weight buffer type overriden to CPU
Tensor blk.18.attn_v.weight buffer type overriden to CPU
Tensor blk.18.attn_q.bias buffer type overriden to CPU
Tensor blk.18.attn_k.bias buffer type overriden to CPU
Tensor blk.18.attn_v.bias buffer type overriden to CPU
Tensor blk.18.attn_output.weight buffer type overriden to CPU
Tensor blk.18.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.18.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.18.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.18.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.18.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.18.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.18.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.18.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.18.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.18.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.18.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.19.attn_norm.weight buffer type overriden to CPU
Tensor blk.19.attn_q.weight buffer type overriden to CPU
Tensor blk.19.attn_k.weight buffer type overriden to CPU
Tensor blk.19.attn_v.weight buffer type overriden to CPU
Tensor blk.19.attn_q.bias buffer type overriden to CPU
Tensor blk.19.attn_k.bias buffer type overriden to CPU
Tensor blk.19.attn_v.bias buffer type overriden to CPU
Tensor blk.19.attn_output.weight buffer type overriden to CPU
Tensor blk.19.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.19.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.19.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.19.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.19.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.19.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.19.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.19.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.19.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.19.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.19.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.20.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.20.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.20.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.21.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.21.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.21.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.22.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.22.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.22.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.23.attn_norm.weight buffer type overriden to CPU
Tensor blk.23.attn_q.weight buffer type overriden to CPU
Tensor blk.23.attn_k.weight buffer type overriden to CPU
Tensor blk.23.attn_v.weight buffer type overriden to CPU
Tensor blk.23.attn_q.bias buffer type overriden to CPU
Tensor blk.23.attn_k.bias buffer type overriden to CPU
Tensor blk.23.attn_v.bias buffer type overriden to CPU
Tensor blk.23.attn_output.weight buffer type overriden to CPU
Tensor blk.23.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.23.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.23.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.23.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.23.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.23.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.23.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.23.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.23.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.23.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.23.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.24.attn_norm.weight buffer type overriden to CPU
Tensor blk.24.attn_q.weight buffer type overriden to CPU
Tensor blk.24.attn_k.weight buffer type overriden to CPU
Tensor blk.24.attn_v.weight buffer type overriden to CPU
Tensor blk.24.attn_q.bias buffer type overriden to CPU
Tensor blk.24.attn_k.bias buffer type overriden to CPU
Tensor blk.24.attn_v.bias buffer type overriden to CPU
Tensor blk.24.attn_output.weight buffer type overriden to CPU
Tensor blk.24.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.24.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.24.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.24.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.24.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.24.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.24.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.24.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.24.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.24.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.24.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.25.attn_norm.weight buffer type overriden to CPU
Tensor blk.25.attn_q.weight buffer type overriden to CPU
Tensor blk.25.attn_k.weight buffer type overriden to CPU
Tensor blk.25.attn_v.weight buffer type overriden to CPU
Tensor blk.25.attn_q.bias buffer type overriden to CPU
Tensor blk.25.attn_k.bias buffer type overriden to CPU
Tensor blk.25.attn_v.bias buffer type overriden to CPU
Tensor blk.25.attn_output.weight buffer type overriden to CPU
Tensor blk.25.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.25.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.25.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.25.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.25.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.25.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.25.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.25.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.25.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.25.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.25.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.26.attn_norm.weight buffer type overriden to CPU
Tensor blk.26.attn_q.weight buffer type overriden to CPU
Tensor blk.26.attn_k.weight buffer type overriden to CPU
Tensor blk.26.attn_v.weight buffer type overriden to CPU
Tensor blk.26.attn_q.bias buffer type overriden to CPU
Tensor blk.26.attn_k.bias buffer type overriden to CPU
Tensor blk.26.attn_v.bias buffer type overriden to CPU
Tensor blk.26.attn_output.weight buffer type overriden to CPU
Tensor blk.26.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.26.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.26.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.26.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.26.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.26.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.26.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.26.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.26.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.26.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.26.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.27.attn_norm.weight buffer type overriden to CPU
Tensor blk.27.attn_q.weight buffer type overriden to CPU
Tensor blk.27.attn_k.weight buffer type overriden to CPU
Tensor blk.27.attn_v.weight buffer type overriden to CPU
Tensor blk.27.attn_q.bias buffer type overriden to CPU
Tensor blk.27.attn_k.bias buffer type overriden to CPU
Tensor blk.27.attn_v.bias buffer type overriden to CPU
Tensor blk.27.attn_output.weight buffer type overriden to CPU
Tensor blk.27.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.27.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.27.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.27.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.27.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.27.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.27.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.27.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.27.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.27.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.27.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.28.attn_norm.weight buffer type overriden to CPU
Tensor blk.28.attn_q.weight buffer type overriden to CPU
Tensor blk.28.attn_k.weight buffer type overriden to CPU
Tensor blk.28.attn_v.weight buffer type overriden to CPU
Tensor blk.28.attn_q.bias buffer type overriden to CPU
Tensor blk.28.attn_k.bias buffer type overriden to CPU
Tensor blk.28.attn_v.bias buffer type overriden to CPU
Tensor blk.28.attn_output.weight buffer type overriden to CPU
Tensor blk.28.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.28.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.28.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.28.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.28.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.28.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.28.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.28.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.28.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.28.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.28.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.29.attn_norm.weight buffer type overriden to CPU
Tensor blk.29.attn_q.weight buffer type overriden to CPU
Tensor blk.29.attn_k.weight buffer type overriden to CPU
Tensor blk.29.attn_v.weight buffer type overriden to CPU
Tensor blk.29.attn_q.bias buffer type overriden to CPU
Tensor blk.29.attn_k.bias buffer type overriden to CPU
Tensor blk.29.attn_v.bias buffer type overriden to CPU
Tensor blk.29.attn_output.weight buffer type overriden to CPU
Tensor blk.29.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.29.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.29.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.29.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.29.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.29.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.29.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.29.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.29.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.29.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.29.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.30.attn_norm.weight buffer type overriden to CPU
Tensor blk.30.attn_q.weight buffer type overriden to CPU
Tensor blk.30.attn_k.weight buffer type overriden to CPU
Tensor blk.30.attn_v.weight buffer type overriden to CPU
Tensor blk.30.attn_q.bias buffer type overriden to CPU
Tensor blk.30.attn_k.bias buffer type overriden to CPU
Tensor blk.30.attn_v.bias buffer type overriden to CPU
Tensor blk.30.attn_output.weight buffer type overriden to CPU
Tensor blk.30.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.30.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.30.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.30.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.30.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.30.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.30.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.30.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.30.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.30.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.30.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.31.attn_norm.weight buffer type overriden to CPU
Tensor blk.31.attn_q.weight buffer type overriden to CPU
Tensor blk.31.attn_k.weight buffer type overriden to CPU
Tensor blk.31.attn_v.weight buffer type overriden to CPU
Tensor blk.31.attn_q.bias buffer type overriden to CPU
Tensor blk.31.attn_k.bias buffer type overriden to CPU
Tensor blk.31.attn_v.bias buffer type overriden to CPU
Tensor blk.31.attn_output.weight buffer type overriden to CPU
Tensor blk.31.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.31.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.31.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.31.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.31.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.31.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.31.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.31.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.31.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.31.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.31.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.32.attn_norm.weight buffer type overriden to CPU
Tensor blk.32.attn_q.weight buffer type overriden to CPU
Tensor blk.32.attn_k.weight buffer type overriden to CPU
Tensor blk.32.attn_v.weight buffer type overriden to CPU
Tensor blk.32.attn_q.bias buffer type overriden to CPU
Tensor blk.32.attn_k.bias buffer type overriden to CPU
Tensor blk.32.attn_v.bias buffer type overriden to CPU
Tensor blk.32.attn_output.weight buffer type overriden to CPU
Tensor blk.32.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.32.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.32.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.32.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.32.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.32.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.32.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.32.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.32.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.32.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.32.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.33.attn_norm.weight buffer type overriden to CPU
Tensor blk.33.attn_q.weight buffer type overriden to CPU
Tensor blk.33.attn_k.weight buffer type overriden to CPU
Tensor blk.33.attn_v.weight buffer type overriden to CPU
Tensor blk.33.attn_q.bias buffer type overriden to CPU
Tensor blk.33.attn_k.bias buffer type overriden to CPU
Tensor blk.33.attn_v.bias buffer type overriden to CPU
Tensor blk.33.attn_output.weight buffer type overriden to CPU
Tensor blk.33.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.33.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.33.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.33.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.33.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.33.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.33.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.33.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.33.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.33.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.33.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.34.attn_norm.weight buffer type overriden to CPU
Tensor blk.34.attn_q.weight buffer type overriden to CPU
Tensor blk.34.attn_k.weight buffer type overriden to CPU
Tensor blk.34.attn_v.weight buffer type overriden to CPU
Tensor blk.34.attn_q.bias buffer type overriden to CPU
Tensor blk.34.attn_k.bias buffer type overriden to CPU
Tensor blk.34.attn_v.bias buffer type overriden to CPU
Tensor blk.34.attn_output.weight buffer type overriden to CPU
Tensor blk.34.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.34.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.34.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.34.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.34.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.34.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.34.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.34.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.34.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.34.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.34.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.35.attn_norm.weight buffer type overriden to CPU
Tensor blk.35.attn_q.weight buffer type overriden to CPU
Tensor blk.35.attn_k.weight buffer type overriden to CPU
Tensor blk.35.attn_v.weight buffer type overriden to CPU
Tensor blk.35.attn_q.bias buffer type overriden to CPU
Tensor blk.35.attn_k.bias buffer type overriden to CPU
Tensor blk.35.attn_v.bias buffer type overriden to CPU
Tensor blk.35.attn_output.weight buffer type overriden to CPU
Tensor blk.35.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.35.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.35.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.35.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.35.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.35.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.35.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.35.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.35.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.35.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.35.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.36.attn_norm.weight buffer type overriden to CPU
Tensor blk.36.attn_q.weight buffer type overriden to CPU
Tensor blk.36.attn_k.weight buffer type overriden to CPU
Tensor blk.36.attn_v.weight buffer type overriden to CPU
Tensor blk.36.attn_q.bias buffer type overriden to CPU
Tensor blk.36.attn_k.bias buffer type overriden to CPU
Tensor blk.36.attn_v.bias buffer type overriden to CPU
Tensor blk.36.attn_output.weight buffer type overriden to CPU
Tensor blk.36.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.36.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.36.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.36.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.36.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.36.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.36.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.36.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.36.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.36.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.36.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.37.attn_norm.weight buffer type overriden to CPU
Tensor blk.37.attn_q.weight buffer type overriden to CPU
Tensor blk.37.attn_k.weight buffer type overriden to CPU
Tensor blk.37.attn_v.weight buffer type overriden to CPU
Tensor blk.37.attn_q.bias buffer type overriden to CPU
Tensor blk.37.attn_k.bias buffer type overriden to CPU
Tensor blk.37.attn_v.bias buffer type overriden to CPU
Tensor blk.37.attn_output.weight buffer type overriden to CPU
Tensor blk.37.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.37.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.37.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.37.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.37.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.37.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.37.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.37.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.37.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.37.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.37.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.38.attn_norm.weight buffer type overriden to CPU
Tensor blk.38.attn_q.weight buffer type overriden to CPU
Tensor blk.38.attn_k.weight buffer type overriden to CPU
Tensor blk.38.attn_v.weight buffer type overriden to CPU
Tensor blk.38.attn_q.bias buffer type overriden to CPU
Tensor blk.38.attn_k.bias buffer type overriden to CPU
Tensor blk.38.attn_v.bias buffer type overriden to CPU
Tensor blk.38.attn_output.weight buffer type overriden to CPU
Tensor blk.38.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.38.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.38.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.38.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.38.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.38.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.38.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.38.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.38.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.38.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.38.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.39.attn_norm.weight buffer type overriden to CPU
Tensor blk.39.attn_q.weight buffer type overriden to CPU
Tensor blk.39.attn_k.weight buffer type overriden to CPU
Tensor blk.39.attn_v.weight buffer type overriden to CPU
Tensor blk.39.attn_q.bias buffer type overriden to CPU
Tensor blk.39.attn_k.bias buffer type overriden to CPU
Tensor blk.39.attn_v.bias buffer type overriden to CPU
Tensor blk.39.attn_output.weight buffer type overriden to CPU
Tensor blk.39.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.39.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.39.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.39.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.39.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.39.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.39.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.39.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.39.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.39.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.39.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.40.attn_norm.weight buffer type overriden to CPU
Tensor blk.40.attn_q.weight buffer type overriden to CPU
Tensor blk.40.attn_k.weight buffer type overriden to CPU
Tensor blk.40.attn_v.weight buffer type overriden to CPU
Tensor blk.40.attn_q.bias buffer type overriden to CPU
Tensor blk.40.attn_k.bias buffer type overriden to CPU
Tensor blk.40.attn_v.bias buffer type overriden to CPU
Tensor blk.40.attn_output.weight buffer type overriden to CPU
Tensor blk.40.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.40.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.40.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.40.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.40.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.40.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.40.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.40.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.40.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.40.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.40.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.41.attn_norm.weight buffer type overriden to CPU
Tensor blk.41.attn_q.weight buffer type overriden to CPU
Tensor blk.41.attn_k.weight buffer type overriden to CPU
Tensor blk.41.attn_v.weight buffer type overriden to CPU
Tensor blk.41.attn_q.bias buffer type overriden to CPU
Tensor blk.41.attn_k.bias buffer type overriden to CPU
Tensor blk.41.attn_v.bias buffer type overriden to CPU
Tensor blk.41.attn_output.weight buffer type overriden to CPU
Tensor blk.41.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.41.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.41.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.41.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.41.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.41.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.41.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.41.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.41.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.41.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.41.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.42.attn_norm.weight buffer type overriden to CPU
Tensor blk.42.attn_q.weight buffer type overriden to CPU
Tensor blk.42.attn_k.weight buffer type overriden to CPU
Tensor blk.42.attn_v.weight buffer type overriden to CPU
Tensor blk.42.attn_q.bias buffer type overriden to CPU
Tensor blk.42.attn_k.bias buffer type overriden to CPU
Tensor blk.42.attn_v.bias buffer type overriden to CPU
Tensor blk.42.attn_output.weight buffer type overriden to CPU
Tensor blk.42.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.42.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.42.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.42.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.42.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.42.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.42.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.42.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.42.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.42.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.42.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.43.attn_norm.weight buffer type overriden to CPU
Tensor blk.43.attn_q.weight buffer type overriden to CPU
Tensor blk.43.attn_k.weight buffer type overriden to CPU
Tensor blk.43.attn_v.weight buffer type overriden to CPU
Tensor blk.43.attn_q.bias buffer type overriden to CPU
Tensor blk.43.attn_k.bias buffer type overriden to CPU
Tensor blk.43.attn_v.bias buffer type overriden to CPU
Tensor blk.43.attn_output.weight buffer type overriden to CPU
Tensor blk.43.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.43.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.43.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.43.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.43.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.43.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.43.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.43.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.43.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.43.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.43.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.44.attn_norm.weight buffer type overriden to CPU
Tensor blk.44.attn_q.weight buffer type overriden to CPU
Tensor blk.44.attn_k.weight buffer type overriden to CPU
Tensor blk.44.attn_v.weight buffer type overriden to CPU
Tensor blk.44.attn_q.bias buffer type overriden to CPU
Tensor blk.44.attn_k.bias buffer type overriden to CPU
Tensor blk.44.attn_v.bias buffer type overriden to CPU
Tensor blk.44.attn_output.weight buffer type overriden to CPU
Tensor blk.44.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.44.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.44.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.44.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.44.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.44.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.44.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.44.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.44.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.44.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.44.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.45.attn_norm.weight buffer type overriden to CPU
Tensor blk.45.attn_q.weight buffer type overriden to CPU
Tensor blk.45.attn_k.weight buffer type overriden to CPU
Tensor blk.45.attn_v.weight buffer type overriden to CPU
Tensor blk.45.attn_q.bias buffer type overriden to CPU
Tensor blk.45.attn_k.bias buffer type overriden to CPU
Tensor blk.45.attn_v.bias buffer type overriden to CPU
Tensor blk.45.attn_output.weight buffer type overriden to CPU
Tensor blk.45.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.45.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.45.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.45.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.45.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.45.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.45.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.45.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.45.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.45.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.45.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.46.attn_norm.weight buffer type overriden to CPU
model has unused tensor blk.46.attn_norm.weight (size = 16384 bytes) -- ignoring
Tensor blk.46.attn_q.weight buffer type overriden to CPU
model has unused tensor blk.46.attn_q.weight (size = 33079296 bytes) -- ignoring
Tensor blk.46.attn_k.weight buffer type overriden to CPU
model has unused tensor blk.46.attn_k.weight (size = 2756608 bytes) -- ignoring
Tensor blk.46.attn_v.weight buffer type overriden to CPU
model has unused tensor blk.46.attn_v.weight (size = 2756608 bytes) -- ignoring
Tensor blk.46.attn_q.bias buffer type overriden to CPU
model has unused tensor blk.46.attn_q.bias (size = 49152 bytes) -- ignoring
Tensor blk.46.attn_k.bias buffer type overriden to CPU
model has unused tensor blk.46.attn_k.bias (size = 4096 bytes) -- ignoring
Tensor blk.46.attn_v.bias buffer type overriden to CPU
model has unused tensor blk.46.attn_v.bias (size = 4096 bytes) -- ignoring
Tensor blk.46.attn_output.weight buffer type overriden to CPU
model has unused tensor blk.46.attn_output.weight (size = 33046528 bytes) -- ignoring
Tensor blk.46.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.46.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.46.post_attention_norm.weight buffer type overriden to CPU
model has unused tensor blk.46.post_attention_norm.weight (size = 16384 bytes) -- ignoring
Tensor blk.46.ffn_gate_inp.weight buffer type overriden to CPU
model has unused tensor blk.46.ffn_gate_inp.weight (size = 2097152 bytes) -- ignoring
Tensor blk.46.exp_probs_b.bias buffer type overriden to CPU
model has unused tensor blk.46.exp_probs_b.bias (size = 512 bytes) -- ignoring
Tensor blk.46.ffn_gate_exps.weight buffer type overriden to CPU
model has unused tensor blk.46.ffn_gate_exps.weight (size = 369819648 bytes) -- ignoring
Tensor blk.46.ffn_down_exps.weight buffer type overriden to CPU
model has unused tensor blk.46.ffn_down_exps.weight (size = 415236096 bytes) -- ignoring
Tensor blk.46.ffn_up_exps.weight buffer type overriden to CPU
model has unused tensor blk.46.ffn_up_exps.weight (size = 369819648 bytes) -- ignoring
Tensor blk.46.ffn_gate_shexp.weight buffer type overriden to CPU
model has unused tensor blk.46.ffn_gate_shexp.weight (size = 3790336 bytes) -- ignoring
Tensor blk.46.ffn_down_shexp.weight buffer type overriden to CPU
model has unused tensor blk.46.ffn_down_shexp.weight (size = 4685824 bytes) -- ignoring
Tensor blk.46.ffn_up_shexp.weight buffer type overriden to CPU
model has unused tensor blk.46.ffn_up_shexp.weight (size = 3790336 bytes) -- ignoring
Tensor blk.46.nextn.eh_proj.weight buffer type overriden to CPU
model has unused tensor blk.46.nextn.eh_proj.weight (size = 16793600 bytes) -- ignoring
Tensor blk.46.nextn.embed_tokens.weight buffer type overriden to CPU
model has unused tensor blk.46.nextn.embed_tokens.weight (size = 310984704 bytes) -- ignoring
Tensor blk.46.nextn.enorm.weight buffer type overriden to CPU
model has unused tensor blk.46.nextn.enorm.weight (size = 16384 bytes) -- ignoring
Tensor blk.46.nextn.hnorm.weight buffer type overriden to CPU
model has unused tensor blk.46.nextn.hnorm.weight (size = 16384 bytes) -- ignoring
Tensor blk.46.nextn.shared_head_head.weight buffer type overriden to CPU
model has unused tensor blk.46.nextn.shared_head_head.weight (size = 310984704 bytes) -- ignoring
Tensor blk.46.nextn.shared_head_norm.weight buffer type overriden to CPU
model has unused tensor blk.46.nextn.shared_head_norm.weight (size = 16384 bytes) -- ignoring
llm_load_tensors: offloading 47 repeating layers to GPU
llm_load_tensors: offloading non-repeating layers to GPU
llm_load_tensors: offloaded 48/48 layers to GPU
llm_load_tensors:        CPU buffer size = 50397.01 MiB
llm_load_tensors:  CUDA_Host buffer size =   333.00 MiB
llm_load_tensors:      CUDA0 buffer size =  3593.55 MiB
....................................................................................................
llama_new_context_with_model: n_ctx      = 16384
llama_new_context_with_model: n_batch    = 2048
llama_new_context_with_model: n_ubatch   = 512
llama_new_context_with_model: flash_attn = 0
llama_new_context_with_model: mla_attn   = 0
llama_new_context_with_model: attn_max_b = 0
llama_new_context_with_model: fused_moe  = 0
llama_new_context_with_model: ser        = -1, 0
llama_new_context_with_model: freq_base  = 1000000.0
llama_new_context_with_model: freq_scale = 1
llama_kv_cache_init:      CUDA0 KV buffer size =  3008.00 MiB
llama_new_context_with_model: KV self size  = 3008.00 MiB, K (f16): 1504.00 MiB, V (f16): 1504.00 MiB
llama_new_context_with_model:  CUDA_Host  output buffer size =     1.16 MiB
llama_new_context_with_model:      CUDA0 compute buffer size =  3176.00 MiB
llama_new_context_with_model:  CUDA_Host compute buffer size =    40.01 MiB
llama_new_context_with_model: graph nodes  = 2422
llama_new_context_with_model: graph splits = 649
INFO [                    init] initializing slots | tid="140128153567232" timestamp=1754577337 n_slots=1
INFO [                    init] new slot | tid="140128153567232" timestamp=1754577337 id_slot=0 n_ctx_slot=16384
INFO [                    main] model loaded | tid="140128153567232" timestamp=1754577337
INFO [                    main] chat template | tid="140128153567232" timestamp=1754577337 chat_example="[gMASK]<sop><|system|>\nYou are a helpful assistant<|user|>\nHello<|assistant|>\nHi there<|user|>\nHow are you?<|assistant|>" built_in=true
INFO [                    main] HTTP server listening | tid="140128153567232" timestamp=1754577337 n_threads_http="11" port="8080" hostname="127.0.0.1"
INFO [            update_slots] all slots are idle | tid="140128153567232" timestamp=1754577337
INFO [   launch_slot_with_task] slot is processing task | tid="140128153567232" timestamp=1754577390 id_slot=0 id_task=0
INFO [            update_slots] kv cache rm [p0, end) | tid="140128153567232" timestamp=1754577390 id_slot=0 id_task=0 p0=0
INFO [           print_timings] prompt eval time     =     909.47 ms /    14 tokens (   64.96 ms per token,    15.39 tokens per second) | tid="140128153567232" timestamp=1754577406 id_slot=0 id_task=0 t_prompt_processing=909.47 n_prompt_tokens_processed=14 t_token=64.96214285714287 n_tokens_second=15.393580876774383
INFO [           print_timings] generation eval time =   15316.76 ms /    85 runs   (  180.20 ms per token,     5.55 tokens per second) | tid="140128153567232" timestamp=1754577406 id_slot=0 id_task=0 t_token_generation=15316.758 n_decoded=85 t_token=180.19715294117645 n_tokens_second=5.549477245772245
INFO [           print_timings]           total time =   16226.23 ms | tid="140128153567232" timestamp=1754577406 id_slot=0 id_task=0 t_prompt_processing=909.47 t_token_generation=15316.758 t_total=16226.228
INFO [            update_slots] slot released | tid="140128153567232" timestamp=1754577406 id_slot=0 id_task=0 n_ctx=16384 n_past=98 n_system_tokens=0 n_cache_tokens=98 truncated=false
INFO [            update_slots] all slots are idle | tid="140128153567232" timestamp=1754577406
INFO [format_partial_response_oaicompat] DEBUG: Streaming finish_reason check | tid="140062683193344" timestamp=1754577406 generated_text="\n<think>The user has greeted me with \"hello\". This is a simple greeting, and I should respond politely and professionally to establish a positive interaction. I'll acknowledge their greeting and introduce myself as an AI assistant, while also expressing that I'm here to help them with any questions or tasks they might have.</think>Hello! I'm here to help you with your questions and tasks. How can I assist you today?" model_name="gpt-3.5-turbo-0613" tool_calls_count=0
INFO [      log_server_request] request | tid="140062683193344" timestamp=1754577406 remote_addr="127.0.0.1" remote_port=43128 status=200 method="POST" path="/v1/chat/completions" params={}
INFO [            update_slots] all slots are idle | tid="140128153567232" timestamp=1754577406
INFO [      log_server_request] request | tid="140062674800640" timestamp=1754577466 remote_addr="127.0.0.1" remote_port=52118 status=200 method="POST" path="/tokenize" params={}
INFO [      log_server_request] request | tid="140062666407936" timestamp=1754577466 remote_addr="127.0.0.1" remote_port=52128 status=200 method="POST" path="/tokenize" params={}
INFO [      log_server_request] request | tid="140062658015232" timestamp=1754577466 remote_addr="127.0.0.1" remote_port=52138 status=200 method="POST" path="/tokenize" params={}
INFO [      log_server_request] request | tid="140062574505984" timestamp=1754577466 remote_addr="127.0.0.1" remote_port=52150 status=200 method="POST" path="/tokenize" params={}
INFO [   launch_slot_with_task] slot is processing task | tid="140128153567232" timestamp=1754577466 id_slot=0 id_task=87
INFO [            update_slots] kv cache rm [p0, end) | tid="140128153567232" timestamp=1754577466 id_slot=0 id_task=87 p0=0
Oops(ggml_compute_forward_sum_rows_f32, ffn_moe_weights_sum-45): found -nan for i1 = 0, i2 = 0, i3 = 0. ne00 = 8
```

---

## 💬 Conversation

👤 **1aienthusiast** commented on **2025-08-07** at **14:58:05**

Also, pc specs:
rtx 3060 12gb, 64gb ddr4 ram, i5 12400f, no swap
tried powerlimiting (sudo nvidia-smi -pl 100) because an user in [#572](https://github.com/ikawrakow/ik_llama.cpp/issues/572) solved it similarly, my ram is not overclocked (3200MHz) nor my cpu, nor my gpu

---

👤 **ikawrakow** commented on **2025-08-07** at **15:12:22**

Does it work with flash attention? (add `-fa` to your command line).

Having a `NaN` in this particular function is only possible if we have NaNs coming out of self-attention, or if there are NaNs in the model weights.

---

👤 **1aienthusiast** commented on **2025-08-07** at **15:18:12**

> Does it work with flash attention? (add `-fa` to your command line).
> 
> Having a `NaN` in this particular function is only possible if we have NaNs coming out of self-attention, or if there are NaNs in the model weights.

`./llama-server -ngl 999 --model ~/TND/AI/GLM4.5-AIR-IQLLAMA/GLM-4.5-Air-IQ4_KSS-00001-of-00002.gguf -ot "8|7|6|5|4|3|9|[1-9][0-9]\.ffn_.*_exps\.=CPU" --no-mmap -c 16384 -fa
ggml_cuda_init: GGML_CUDA_FORCE_MMQ:    no
ggml_cuda_init: GGML_CUDA_FORCE_CUBLAS: no
ggml_cuda_init: found 1 CUDA devices:
  Device 0: NVIDIA GeForce RTX 3060, compute capability 8.6, VMM: yes
INFO [                    main] build info | tid="140550555570176" timestamp=1754579747 build=3829 commit="dee40cff"
INFO [                    main] system info | tid="140550555570176" timestamp=1754579747 n_threads=6 n_threads_batch=-1 total_threads=12 system_info="AVX = 1 | AVX_VNNI = 1 | AVX2 = 1 | AVX512 = 0 | AVX512_VBMI = 0 | AVX512_VNNI = 0 | AVX512_BF16 = 0 | FMA = 1 | NEON = 0 | SVE = 0 | ARM_FMA = 0 | F16C = 1 | FP16_VA = 0 | WASM_SIMD = 0 | BLAS = 1 | SSE3 = 1 | SSSE3 = 1 | VSX = 0 | MATMUL_INT8 = 0 | LLAMAFILE = 1 | "
llama_model_loader: additional 1 GGUFs metadata loaded.
llama_model_loader: loaded meta data with 48 key-value pairs and 803 tensors from /home/john/TND/AI/GLM4.5-AIR-IQLLAMA/GLM-4.5-Air-IQ4_KSS-00001-of-00002.gguf (version GGUF V3 (latest))
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
llm_load_print_meta: model ftype      = IQ4_KSS - 4.0 bpw
llm_load_print_meta: model params     = 110.469 B
llm_load_print_meta: model size       = 54.801 GiB (4.261 BPW) 
llm_load_print_meta: repeating layers = 53.997 GiB (4.246 BPW, 109.227 B parameters)
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
llm_load_tensors: ggml ctx size =    0.66 MiB
Tensor blk.3.attn_norm.weight buffer type overriden to CPU
Tensor blk.3.attn_q.weight buffer type overriden to CPU
Tensor blk.3.attn_k.weight buffer type overriden to CPU
Tensor blk.3.attn_v.weight buffer type overriden to CPU
Tensor blk.3.attn_q.bias buffer type overriden to CPU
Tensor blk.3.attn_k.bias buffer type overriden to CPU
Tensor blk.3.attn_v.bias buffer type overriden to CPU
Tensor blk.3.attn_output.weight buffer type overriden to CPU
Tensor blk.3.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.3.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.3.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.3.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.3.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.3.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.3.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.3.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.3.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.3.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.3.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.4.attn_norm.weight buffer type overriden to CPU
Tensor blk.4.attn_q.weight buffer type overriden to CPU
Tensor blk.4.attn_k.weight buffer type overriden to CPU
Tensor blk.4.attn_v.weight buffer type overriden to CPU
Tensor blk.4.attn_q.bias buffer type overriden to CPU
Tensor blk.4.attn_k.bias buffer type overriden to CPU
Tensor blk.4.attn_v.bias buffer type overriden to CPU
Tensor blk.4.attn_output.weight buffer type overriden to CPU
Tensor blk.4.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.4.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.4.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.4.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.4.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.4.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.4.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.4.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.4.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.4.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.4.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.5.attn_norm.weight buffer type overriden to CPU
Tensor blk.5.attn_q.weight buffer type overriden to CPU
Tensor blk.5.attn_k.weight buffer type overriden to CPU
Tensor blk.5.attn_v.weight buffer type overriden to CPU
Tensor blk.5.attn_q.bias buffer type overriden to CPU
Tensor blk.5.attn_k.bias buffer type overriden to CPU
Tensor blk.5.attn_v.bias buffer type overriden to CPU
Tensor blk.5.attn_output.weight buffer type overriden to CPU
Tensor blk.5.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.5.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.5.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.5.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.5.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.5.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.5.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.5.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.5.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.5.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.5.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.6.attn_norm.weight buffer type overriden to CPU
Tensor blk.6.attn_q.weight buffer type overriden to CPU
Tensor blk.6.attn_k.weight buffer type overriden to CPU
Tensor blk.6.attn_v.weight buffer type overriden to CPU
Tensor blk.6.attn_q.bias buffer type overriden to CPU
Tensor blk.6.attn_k.bias buffer type overriden to CPU
Tensor blk.6.attn_v.bias buffer type overriden to CPU
Tensor blk.6.attn_output.weight buffer type overriden to CPU
Tensor blk.6.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.6.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.6.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.6.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.6.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.6.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.6.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.6.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.6.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.6.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.6.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.7.attn_norm.weight buffer type overriden to CPU
Tensor blk.7.attn_q.weight buffer type overriden to CPU
Tensor blk.7.attn_k.weight buffer type overriden to CPU
Tensor blk.7.attn_v.weight buffer type overriden to CPU
Tensor blk.7.attn_q.bias buffer type overriden to CPU
Tensor blk.7.attn_k.bias buffer type overriden to CPU
Tensor blk.7.attn_v.bias buffer type overriden to CPU
Tensor blk.7.attn_output.weight buffer type overriden to CPU
Tensor blk.7.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.7.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.7.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.7.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.7.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.7.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.7.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.7.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.7.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.7.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.7.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.8.attn_norm.weight buffer type overriden to CPU
Tensor blk.8.attn_q.weight buffer type overriden to CPU
Tensor blk.8.attn_k.weight buffer type overriden to CPU
Tensor blk.8.attn_v.weight buffer type overriden to CPU
Tensor blk.8.attn_q.bias buffer type overriden to CPU
Tensor blk.8.attn_k.bias buffer type overriden to CPU
Tensor blk.8.attn_v.bias buffer type overriden to CPU
Tensor blk.8.attn_output.weight buffer type overriden to CPU
Tensor blk.8.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.8.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.8.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.8.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.8.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.8.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.8.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.8.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.8.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.8.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.8.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.9.attn_norm.weight buffer type overriden to CPU
Tensor blk.9.attn_q.weight buffer type overriden to CPU
Tensor blk.9.attn_k.weight buffer type overriden to CPU
Tensor blk.9.attn_v.weight buffer type overriden to CPU
Tensor blk.9.attn_q.bias buffer type overriden to CPU
Tensor blk.9.attn_k.bias buffer type overriden to CPU
Tensor blk.9.attn_v.bias buffer type overriden to CPU
Tensor blk.9.attn_output.weight buffer type overriden to CPU
Tensor blk.9.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.9.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.9.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.9.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.9.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.9.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.9.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.9.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.9.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.9.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.9.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.10.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.10.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.10.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.11.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.11.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.11.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.12.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.12.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.12.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.13.attn_norm.weight buffer type overriden to CPU
Tensor blk.13.attn_q.weight buffer type overriden to CPU
Tensor blk.13.attn_k.weight buffer type overriden to CPU
Tensor blk.13.attn_v.weight buffer type overriden to CPU
Tensor blk.13.attn_q.bias buffer type overriden to CPU
Tensor blk.13.attn_k.bias buffer type overriden to CPU
Tensor blk.13.attn_v.bias buffer type overriden to CPU
Tensor blk.13.attn_output.weight buffer type overriden to CPU
Tensor blk.13.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.13.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.13.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.13.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.13.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.13.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.13.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.13.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.13.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.13.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.13.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.14.attn_norm.weight buffer type overriden to CPU
Tensor blk.14.attn_q.weight buffer type overriden to CPU
Tensor blk.14.attn_k.weight buffer type overriden to CPU
Tensor blk.14.attn_v.weight buffer type overriden to CPU
Tensor blk.14.attn_q.bias buffer type overriden to CPU
Tensor blk.14.attn_k.bias buffer type overriden to CPU
Tensor blk.14.attn_v.bias buffer type overriden to CPU
Tensor blk.14.attn_output.weight buffer type overriden to CPU
Tensor blk.14.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.14.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.14.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.14.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.14.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.14.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.14.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.14.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.14.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.14.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.14.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.15.attn_norm.weight buffer type overriden to CPU
Tensor blk.15.attn_q.weight buffer type overriden to CPU
Tensor blk.15.attn_k.weight buffer type overriden to CPU
Tensor blk.15.attn_v.weight buffer type overriden to CPU
Tensor blk.15.attn_q.bias buffer type overriden to CPU
Tensor blk.15.attn_k.bias buffer type overriden to CPU
Tensor blk.15.attn_v.bias buffer type overriden to CPU
Tensor blk.15.attn_output.weight buffer type overriden to CPU
Tensor blk.15.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.15.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.15.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.15.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.15.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.15.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.15.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.15.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.15.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.15.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.15.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.16.attn_norm.weight buffer type overriden to CPU
Tensor blk.16.attn_q.weight buffer type overriden to CPU
Tensor blk.16.attn_k.weight buffer type overriden to CPU
Tensor blk.16.attn_v.weight buffer type overriden to CPU
Tensor blk.16.attn_q.bias buffer type overriden to CPU
Tensor blk.16.attn_k.bias buffer type overriden to CPU
Tensor blk.16.attn_v.bias buffer type overriden to CPU
Tensor blk.16.attn_output.weight buffer type overriden to CPU
Tensor blk.16.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.16.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.16.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.16.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.16.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.16.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.16.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.16.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.16.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.16.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.16.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.17.attn_norm.weight buffer type overriden to CPU
Tensor blk.17.attn_q.weight buffer type overriden to CPU
Tensor blk.17.attn_k.weight buffer type overriden to CPU
Tensor blk.17.attn_v.weight buffer type overriden to CPU
Tensor blk.17.attn_q.bias buffer type overriden to CPU
Tensor blk.17.attn_k.bias buffer type overriden to CPU
Tensor blk.17.attn_v.bias buffer type overriden to CPU
Tensor blk.17.attn_output.weight buffer type overriden to CPU
Tensor blk.17.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.17.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.17.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.17.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.17.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.17.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.17.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.17.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.17.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.17.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.17.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.18.attn_norm.weight buffer type overriden to CPU
Tensor blk.18.attn_q.weight buffer type overriden to CPU
Tensor blk.18.attn_k.weight buffer type overriden to CPU
Tensor blk.18.attn_v.weight buffer type overriden to CPU
Tensor blk.18.attn_q.bias buffer type overriden to CPU
Tensor blk.18.attn_k.bias buffer type overriden to CPU
Tensor blk.18.attn_v.bias buffer type overriden to CPU
Tensor blk.18.attn_output.weight buffer type overriden to CPU
Tensor blk.18.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.18.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.18.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.18.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.18.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.18.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.18.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.18.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.18.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.18.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.18.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.19.attn_norm.weight buffer type overriden to CPU
Tensor blk.19.attn_q.weight buffer type overriden to CPU
Tensor blk.19.attn_k.weight buffer type overriden to CPU
Tensor blk.19.attn_v.weight buffer type overriden to CPU
Tensor blk.19.attn_q.bias buffer type overriden to CPU
Tensor blk.19.attn_k.bias buffer type overriden to CPU
Tensor blk.19.attn_v.bias buffer type overriden to CPU
Tensor blk.19.attn_output.weight buffer type overriden to CPU
Tensor blk.19.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.19.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.19.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.19.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.19.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.19.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.19.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.19.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.19.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.19.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.19.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.20.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.20.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.20.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.21.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.21.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.21.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.22.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.22.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.22.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.23.attn_norm.weight buffer type overriden to CPU
Tensor blk.23.attn_q.weight buffer type overriden to CPU
Tensor blk.23.attn_k.weight buffer type overriden to CPU
Tensor blk.23.attn_v.weight buffer type overriden to CPU
Tensor blk.23.attn_q.bias buffer type overriden to CPU
Tensor blk.23.attn_k.bias buffer type overriden to CPU
Tensor blk.23.attn_v.bias buffer type overriden to CPU
Tensor blk.23.attn_output.weight buffer type overriden to CPU
Tensor blk.23.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.23.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.23.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.23.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.23.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.23.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.23.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.23.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.23.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.23.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.23.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.24.attn_norm.weight buffer type overriden to CPU
Tensor blk.24.attn_q.weight buffer type overriden to CPU
Tensor blk.24.attn_k.weight buffer type overriden to CPU
Tensor blk.24.attn_v.weight buffer type overriden to CPU
Tensor blk.24.attn_q.bias buffer type overriden to CPU
Tensor blk.24.attn_k.bias buffer type overriden to CPU
Tensor blk.24.attn_v.bias buffer type overriden to CPU
Tensor blk.24.attn_output.weight buffer type overriden to CPU
Tensor blk.24.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.24.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.24.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.24.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.24.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.24.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.24.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.24.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.24.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.24.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.24.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.25.attn_norm.weight buffer type overriden to CPU
Tensor blk.25.attn_q.weight buffer type overriden to CPU
Tensor blk.25.attn_k.weight buffer type overriden to CPU
Tensor blk.25.attn_v.weight buffer type overriden to CPU
Tensor blk.25.attn_q.bias buffer type overriden to CPU
Tensor blk.25.attn_k.bias buffer type overriden to CPU
Tensor blk.25.attn_v.bias buffer type overriden to CPU
Tensor blk.25.attn_output.weight buffer type overriden to CPU
Tensor blk.25.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.25.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.25.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.25.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.25.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.25.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.25.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.25.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.25.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.25.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.25.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.26.attn_norm.weight buffer type overriden to CPU
Tensor blk.26.attn_q.weight buffer type overriden to CPU
Tensor blk.26.attn_k.weight buffer type overriden to CPU
Tensor blk.26.attn_v.weight buffer type overriden to CPU
Tensor blk.26.attn_q.bias buffer type overriden to CPU
Tensor blk.26.attn_k.bias buffer type overriden to CPU
Tensor blk.26.attn_v.bias buffer type overriden to CPU
Tensor blk.26.attn_output.weight buffer type overriden to CPU
Tensor blk.26.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.26.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.26.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.26.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.26.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.26.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.26.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.26.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.26.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.26.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.26.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.27.attn_norm.weight buffer type overriden to CPU
Tensor blk.27.attn_q.weight buffer type overriden to CPU
Tensor blk.27.attn_k.weight buffer type overriden to CPU
Tensor blk.27.attn_v.weight buffer type overriden to CPU
Tensor blk.27.attn_q.bias buffer type overriden to CPU
Tensor blk.27.attn_k.bias buffer type overriden to CPU
Tensor blk.27.attn_v.bias buffer type overriden to CPU
Tensor blk.27.attn_output.weight buffer type overriden to CPU
Tensor blk.27.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.27.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.27.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.27.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.27.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.27.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.27.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.27.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.27.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.27.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.27.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.28.attn_norm.weight buffer type overriden to CPU
Tensor blk.28.attn_q.weight buffer type overriden to CPU
Tensor blk.28.attn_k.weight buffer type overriden to CPU
Tensor blk.28.attn_v.weight buffer type overriden to CPU
Tensor blk.28.attn_q.bias buffer type overriden to CPU
Tensor blk.28.attn_k.bias buffer type overriden to CPU
Tensor blk.28.attn_v.bias buffer type overriden to CPU
Tensor blk.28.attn_output.weight buffer type overriden to CPU
Tensor blk.28.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.28.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.28.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.28.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.28.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.28.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.28.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.28.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.28.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.28.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.28.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.29.attn_norm.weight buffer type overriden to CPU
Tensor blk.29.attn_q.weight buffer type overriden to CPU
Tensor blk.29.attn_k.weight buffer type overriden to CPU
Tensor blk.29.attn_v.weight buffer type overriden to CPU
Tensor blk.29.attn_q.bias buffer type overriden to CPU
Tensor blk.29.attn_k.bias buffer type overriden to CPU
Tensor blk.29.attn_v.bias buffer type overriden to CPU
Tensor blk.29.attn_output.weight buffer type overriden to CPU
Tensor blk.29.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.29.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.29.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.29.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.29.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.29.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.29.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.29.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.29.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.29.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.29.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.30.attn_norm.weight buffer type overriden to CPU
Tensor blk.30.attn_q.weight buffer type overriden to CPU
Tensor blk.30.attn_k.weight buffer type overriden to CPU
Tensor blk.30.attn_v.weight buffer type overriden to CPU
Tensor blk.30.attn_q.bias buffer type overriden to CPU
Tensor blk.30.attn_k.bias buffer type overriden to CPU
Tensor blk.30.attn_v.bias buffer type overriden to CPU
Tensor blk.30.attn_output.weight buffer type overriden to CPU
Tensor blk.30.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.30.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.30.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.30.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.30.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.30.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.30.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.30.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.30.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.30.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.30.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.31.attn_norm.weight buffer type overriden to CPU
Tensor blk.31.attn_q.weight buffer type overriden to CPU
Tensor blk.31.attn_k.weight buffer type overriden to CPU
Tensor blk.31.attn_v.weight buffer type overriden to CPU
Tensor blk.31.attn_q.bias buffer type overriden to CPU
Tensor blk.31.attn_k.bias buffer type overriden to CPU
Tensor blk.31.attn_v.bias buffer type overriden to CPU
Tensor blk.31.attn_output.weight buffer type overriden to CPU
Tensor blk.31.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.31.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.31.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.31.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.31.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.31.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.31.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.31.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.31.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.31.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.31.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.32.attn_norm.weight buffer type overriden to CPU
Tensor blk.32.attn_q.weight buffer type overriden to CPU
Tensor blk.32.attn_k.weight buffer type overriden to CPU
Tensor blk.32.attn_v.weight buffer type overriden to CPU
Tensor blk.32.attn_q.bias buffer type overriden to CPU
Tensor blk.32.attn_k.bias buffer type overriden to CPU
Tensor blk.32.attn_v.bias buffer type overriden to CPU
Tensor blk.32.attn_output.weight buffer type overriden to CPU
Tensor blk.32.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.32.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.32.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.32.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.32.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.32.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.32.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.32.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.32.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.32.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.32.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.33.attn_norm.weight buffer type overriden to CPU
Tensor blk.33.attn_q.weight buffer type overriden to CPU
Tensor blk.33.attn_k.weight buffer type overriden to CPU
Tensor blk.33.attn_v.weight buffer type overriden to CPU
Tensor blk.33.attn_q.bias buffer type overriden to CPU
Tensor blk.33.attn_k.bias buffer type overriden to CPU
Tensor blk.33.attn_v.bias buffer type overriden to CPU
Tensor blk.33.attn_output.weight buffer type overriden to CPU
Tensor blk.33.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.33.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.33.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.33.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.33.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.33.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.33.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.33.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.33.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.33.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.33.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.34.attn_norm.weight buffer type overriden to CPU
Tensor blk.34.attn_q.weight buffer type overriden to CPU
Tensor blk.34.attn_k.weight buffer type overriden to CPU
Tensor blk.34.attn_v.weight buffer type overriden to CPU
Tensor blk.34.attn_q.bias buffer type overriden to CPU
Tensor blk.34.attn_k.bias buffer type overriden to CPU
Tensor blk.34.attn_v.bias buffer type overriden to CPU
Tensor blk.34.attn_output.weight buffer type overriden to CPU
Tensor blk.34.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.34.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.34.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.34.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.34.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.34.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.34.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.34.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.34.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.34.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.34.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.35.attn_norm.weight buffer type overriden to CPU
Tensor blk.35.attn_q.weight buffer type overriden to CPU
Tensor blk.35.attn_k.weight buffer type overriden to CPU
Tensor blk.35.attn_v.weight buffer type overriden to CPU
Tensor blk.35.attn_q.bias buffer type overriden to CPU
Tensor blk.35.attn_k.bias buffer type overriden to CPU
Tensor blk.35.attn_v.bias buffer type overriden to CPU
Tensor blk.35.attn_output.weight buffer type overriden to CPU
Tensor blk.35.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.35.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.35.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.35.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.35.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.35.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.35.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.35.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.35.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.35.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.35.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.36.attn_norm.weight buffer type overriden to CPU
Tensor blk.36.attn_q.weight buffer type overriden to CPU
Tensor blk.36.attn_k.weight buffer type overriden to CPU
Tensor blk.36.attn_v.weight buffer type overriden to CPU
Tensor blk.36.attn_q.bias buffer type overriden to CPU
Tensor blk.36.attn_k.bias buffer type overriden to CPU
Tensor blk.36.attn_v.bias buffer type overriden to CPU
Tensor blk.36.attn_output.weight buffer type overriden to CPU
Tensor blk.36.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.36.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.36.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.36.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.36.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.36.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.36.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.36.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.36.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.36.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.36.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.37.attn_norm.weight buffer type overriden to CPU
Tensor blk.37.attn_q.weight buffer type overriden to CPU
Tensor blk.37.attn_k.weight buffer type overriden to CPU
Tensor blk.37.attn_v.weight buffer type overriden to CPU
Tensor blk.37.attn_q.bias buffer type overriden to CPU
Tensor blk.37.attn_k.bias buffer type overriden to CPU
Tensor blk.37.attn_v.bias buffer type overriden to CPU
Tensor blk.37.attn_output.weight buffer type overriden to CPU
Tensor blk.37.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.37.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.37.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.37.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.37.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.37.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.37.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.37.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.37.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.37.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.37.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.38.attn_norm.weight buffer type overriden to CPU
Tensor blk.38.attn_q.weight buffer type overriden to CPU
Tensor blk.38.attn_k.weight buffer type overriden to CPU
Tensor blk.38.attn_v.weight buffer type overriden to CPU
Tensor blk.38.attn_q.bias buffer type overriden to CPU
Tensor blk.38.attn_k.bias buffer type overriden to CPU
Tensor blk.38.attn_v.bias buffer type overriden to CPU
Tensor blk.38.attn_output.weight buffer type overriden to CPU
Tensor blk.38.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.38.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.38.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.38.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.38.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.38.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.38.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.38.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.38.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.38.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.38.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.39.attn_norm.weight buffer type overriden to CPU
Tensor blk.39.attn_q.weight buffer type overriden to CPU
Tensor blk.39.attn_k.weight buffer type overriden to CPU
Tensor blk.39.attn_v.weight buffer type overriden to CPU
Tensor blk.39.attn_q.bias buffer type overriden to CPU
Tensor blk.39.attn_k.bias buffer type overriden to CPU
Tensor blk.39.attn_v.bias buffer type overriden to CPU
Tensor blk.39.attn_output.weight buffer type overriden to CPU
Tensor blk.39.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.39.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.39.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.39.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.39.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.39.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.39.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.39.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.39.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.39.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.39.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.40.attn_norm.weight buffer type overriden to CPU
Tensor blk.40.attn_q.weight buffer type overriden to CPU
Tensor blk.40.attn_k.weight buffer type overriden to CPU
Tensor blk.40.attn_v.weight buffer type overriden to CPU
Tensor blk.40.attn_q.bias buffer type overriden to CPU
Tensor blk.40.attn_k.bias buffer type overriden to CPU
Tensor blk.40.attn_v.bias buffer type overriden to CPU
Tensor blk.40.attn_output.weight buffer type overriden to CPU
Tensor blk.40.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.40.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.40.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.40.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.40.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.40.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.40.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.40.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.40.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.40.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.40.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.41.attn_norm.weight buffer type overriden to CPU
Tensor blk.41.attn_q.weight buffer type overriden to CPU
Tensor blk.41.attn_k.weight buffer type overriden to CPU
Tensor blk.41.attn_v.weight buffer type overriden to CPU
Tensor blk.41.attn_q.bias buffer type overriden to CPU
Tensor blk.41.attn_k.bias buffer type overriden to CPU
Tensor blk.41.attn_v.bias buffer type overriden to CPU
Tensor blk.41.attn_output.weight buffer type overriden to CPU
Tensor blk.41.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.41.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.41.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.41.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.41.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.41.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.41.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.41.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.41.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.41.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.41.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.42.attn_norm.weight buffer type overriden to CPU
Tensor blk.42.attn_q.weight buffer type overriden to CPU
Tensor blk.42.attn_k.weight buffer type overriden to CPU
Tensor blk.42.attn_v.weight buffer type overriden to CPU
Tensor blk.42.attn_q.bias buffer type overriden to CPU
Tensor blk.42.attn_k.bias buffer type overriden to CPU
Tensor blk.42.attn_v.bias buffer type overriden to CPU
Tensor blk.42.attn_output.weight buffer type overriden to CPU
Tensor blk.42.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.42.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.42.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.42.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.42.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.42.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.42.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.42.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.42.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.42.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.42.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.43.attn_norm.weight buffer type overriden to CPU
Tensor blk.43.attn_q.weight buffer type overriden to CPU
Tensor blk.43.attn_k.weight buffer type overriden to CPU
Tensor blk.43.attn_v.weight buffer type overriden to CPU
Tensor blk.43.attn_q.bias buffer type overriden to CPU
Tensor blk.43.attn_k.bias buffer type overriden to CPU
Tensor blk.43.attn_v.bias buffer type overriden to CPU
Tensor blk.43.attn_output.weight buffer type overriden to CPU
Tensor blk.43.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.43.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.43.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.43.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.43.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.43.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.43.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.43.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.43.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.43.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.43.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.44.attn_norm.weight buffer type overriden to CPU
Tensor blk.44.attn_q.weight buffer type overriden to CPU
Tensor blk.44.attn_k.weight buffer type overriden to CPU
Tensor blk.44.attn_v.weight buffer type overriden to CPU
Tensor blk.44.attn_q.bias buffer type overriden to CPU
Tensor blk.44.attn_k.bias buffer type overriden to CPU
Tensor blk.44.attn_v.bias buffer type overriden to CPU
Tensor blk.44.attn_output.weight buffer type overriden to CPU
Tensor blk.44.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.44.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.44.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.44.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.44.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.44.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.44.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.44.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.44.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.44.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.44.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.45.attn_norm.weight buffer type overriden to CPU
Tensor blk.45.attn_q.weight buffer type overriden to CPU
Tensor blk.45.attn_k.weight buffer type overriden to CPU
Tensor blk.45.attn_v.weight buffer type overriden to CPU
Tensor blk.45.attn_q.bias buffer type overriden to CPU
Tensor blk.45.attn_k.bias buffer type overriden to CPU
Tensor blk.45.attn_v.bias buffer type overriden to CPU
Tensor blk.45.attn_output.weight buffer type overriden to CPU
Tensor blk.45.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.45.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.45.post_attention_norm.weight buffer type overriden to CPU
Tensor blk.45.ffn_gate_inp.weight buffer type overriden to CPU
Tensor blk.45.exp_probs_b.bias buffer type overriden to CPU
Tensor blk.45.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.45.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.45.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.45.ffn_gate_shexp.weight buffer type overriden to CPU
Tensor blk.45.ffn_down_shexp.weight buffer type overriden to CPU
Tensor blk.45.ffn_up_shexp.weight buffer type overriden to CPU
Tensor blk.46.attn_norm.weight buffer type overriden to CPU
model has unused tensor blk.46.attn_norm.weight (size = 16384 bytes) -- ignoring
Tensor blk.46.attn_q.weight buffer type overriden to CPU
model has unused tensor blk.46.attn_q.weight (size = 33079296 bytes) -- ignoring
Tensor blk.46.attn_k.weight buffer type overriden to CPU
model has unused tensor blk.46.attn_k.weight (size = 2756608 bytes) -- ignoring
Tensor blk.46.attn_v.weight buffer type overriden to CPU
model has unused tensor blk.46.attn_v.weight (size = 2756608 bytes) -- ignoring
Tensor blk.46.attn_q.bias buffer type overriden to CPU
model has unused tensor blk.46.attn_q.bias (size = 49152 bytes) -- ignoring
Tensor blk.46.attn_k.bias buffer type overriden to CPU
model has unused tensor blk.46.attn_k.bias (size = 4096 bytes) -- ignoring
Tensor blk.46.attn_v.bias buffer type overriden to CPU
model has unused tensor blk.46.attn_v.bias (size = 4096 bytes) -- ignoring
Tensor blk.46.attn_output.weight buffer type overriden to CPU
model has unused tensor blk.46.attn_output.weight (size = 33046528 bytes) -- ignoring
Tensor blk.46.attn_q_norm.weight buffer type overriden to CPU
Tensor blk.46.attn_k_norm.weight buffer type overriden to CPU
Tensor blk.46.post_attention_norm.weight buffer type overriden to CPU
model has unused tensor blk.46.post_attention_norm.weight (size = 16384 bytes) -- ignoring
Tensor blk.46.ffn_gate_inp.weight buffer type overriden to CPU
model has unused tensor blk.46.ffn_gate_inp.weight (size = 2097152 bytes) -- ignoring
Tensor blk.46.exp_probs_b.bias buffer type overriden to CPU
model has unused tensor blk.46.exp_probs_b.bias (size = 512 bytes) -- ignoring
Tensor blk.46.ffn_gate_exps.weight buffer type overriden to CPU
model has unused tensor blk.46.ffn_gate_exps.weight (size = 369819648 bytes) -- ignoring
Tensor blk.46.ffn_down_exps.weight buffer type overriden to CPU
model has unused tensor blk.46.ffn_down_exps.weight (size = 415236096 bytes) -- ignoring
Tensor blk.46.ffn_up_exps.weight buffer type overriden to CPU
model has unused tensor blk.46.ffn_up_exps.weight (size = 369819648 bytes) -- ignoring
Tensor blk.46.ffn_gate_shexp.weight buffer type overriden to CPU
model has unused tensor blk.46.ffn_gate_shexp.weight (size = 3790336 bytes) -- ignoring
Tensor blk.46.ffn_down_shexp.weight buffer type overriden to CPU
model has unused tensor blk.46.ffn_down_shexp.weight (size = 4685824 bytes) -- ignoring
Tensor blk.46.ffn_up_shexp.weight buffer type overriden to CPU
model has unused tensor blk.46.ffn_up_shexp.weight (size = 3790336 bytes) -- ignoring
Tensor blk.46.nextn.eh_proj.weight buffer type overriden to CPU
model has unused tensor blk.46.nextn.eh_proj.weight (size = 16793600 bytes) -- ignoring
Tensor blk.46.nextn.embed_tokens.weight buffer type overriden to CPU
model has unused tensor blk.46.nextn.embed_tokens.weight (size = 310984704 bytes) -- ignoring
Tensor blk.46.nextn.enorm.weight buffer type overriden to CPU
model has unused tensor blk.46.nextn.enorm.weight (size = 16384 bytes) -- ignoring
Tensor blk.46.nextn.hnorm.weight buffer type overriden to CPU
model has unused tensor blk.46.nextn.hnorm.weight (size = 16384 bytes) -- ignoring
Tensor blk.46.nextn.shared_head_head.weight buffer type overriden to CPU
model has unused tensor blk.46.nextn.shared_head_head.weight (size = 310984704 bytes) -- ignoring
Tensor blk.46.nextn.shared_head_norm.weight buffer type overriden to CPU
model has unused tensor blk.46.nextn.shared_head_norm.weight (size = 16384 bytes) -- ignoring
llm_load_tensors: offloading 47 repeating layers to GPU
llm_load_tensors: offloading non-repeating layers to GPU
llm_load_tensors: offloaded 48/48 layers to GPU
llm_load_tensors:        CPU buffer size = 50397.01 MiB
llm_load_tensors:  CUDA_Host buffer size =   333.00 MiB
llm_load_tensors:      CUDA0 buffer size =  3593.55 MiB
....................................................................................................
llama_new_context_with_model: n_ctx      = 16384
llama_new_context_with_model: n_batch    = 2048
llama_new_context_with_model: n_ubatch   = 512
llama_new_context_with_model: flash_attn = 1
llama_new_context_with_model: mla_attn   = 0
llama_new_context_with_model: attn_max_b = 0
llama_new_context_with_model: fused_moe  = 0
llama_new_context_with_model: ser        = -1, 0
llama_new_context_with_model: freq_base  = 1000000.0
llama_new_context_with_model: freq_scale = 1
llama_kv_cache_init:      CUDA0 KV buffer size =  3008.00 MiB
llama_new_context_with_model: KV self size  = 3008.00 MiB, K (f16): 1504.00 MiB, V (f16): 1504.00 MiB
llama_new_context_with_model:  CUDA_Host  output buffer size =     1.16 MiB
llama_new_context_with_model:      CUDA0 compute buffer size =   538.00 MiB
llama_new_context_with_model:  CUDA_Host compute buffer size =    40.01 MiB
llama_new_context_with_model: graph nodes  = 2239
llama_new_context_with_model: graph splits = 649
INFO [                    init] initializing slots | tid="140550555570176" timestamp=1754579786 n_slots=1
INFO [                    init] new slot | tid="140550555570176" timestamp=1754579786 id_slot=0 n_ctx_slot=16384
INFO [                    main] model loaded | tid="140550555570176" timestamp=1754579786
INFO [                    main] chat template | tid="140550555570176" timestamp=1754579786 chat_example="[gMASK]<sop><|system|>\nYou are a helpful assistant<|user|>\nHello<|assistant|>\nHi there<|user|>\nHow are you?<|assistant|>" built_in=true
INFO [                    main] HTTP server listening | tid="140550555570176" timestamp=1754579786 n_threads_http="11" port="8080" hostname="127.0.0.1"
INFO [            update_slots] all slots are idle | tid="140550555570176" timestamp=1754579786
INFO [      log_server_request] request | tid="140548844736512" timestamp=1754579817 remote_addr="127.0.0.1" remote_port=42190 status=200 method="GET" path="/v1/models" params={}
INFO [   launch_slot_with_task] slot is processing task | tid="140550555570176" timestamp=1754579820 id_slot=0 id_task=0
INFO [            update_slots] kv cache rm [p0, end) | tid="140550555570176" timestamp=1754579820 id_slot=0 id_task=0 p0=0
Oops(ggml_compute_forward_sum_rows_f32, ffn_moe_weights_sum-45): found -nan for i1 = 0, i2 = 0, i3 = 0. ne00 = 8
`

still crashes the same

---

👤 **skrew** commented on **2025-08-12** at **17:34:19**

Got the same crash: 
`Oops(ggml_compute_forward_sum_rows_f32, ffn_moe_weights_sum-1): found -nan for i1 = 0, i2 = 0, i3 = 0. ne00 = 128`

Using  1x3090 and 1x1080 12gb with `ubergarm` model `GLM-4.5-Air-IQ4_K`, current `main` version.

Running with `./llama-server -m /xxx/GLM-4.5-Air/IQ4_K/GLM-4.5-Air-IQ4_K-00001-of-00002.gguf --no-mmap -b 2048 -ub 1024 -ngl 22 --n-cpu-moe 29 -rtr -c 8192 -fmoe -fa --jinja`

Last logs:
```
============ Repacked 259 tensors
llama_new_context_with_model: n_ctx      = 8192
llama_new_context_with_model: n_batch    = 2048
llama_new_context_with_model: n_ubatch   = 1024
llama_new_context_with_model: flash_attn = 1
llama_new_context_with_model: mla_attn   = 0
llama_new_context_with_model: attn_max_b = 0
llama_new_context_with_model: fused_moe  = 1
llama_new_context_with_model: ser        = -1, 0
llama_new_context_with_model: freq_base  = 1000000.0
llama_new_context_with_model: freq_scale = 1
llama_kv_cache_init:  CUDA_Host KV buffer size =   800.00 MiB
llama_kv_cache_init:      CUDA0 KV buffer size =   480.00 MiB
llama_kv_cache_init:      CUDA1 KV buffer size =   224.00 MiB
llama_new_context_with_model: KV self size  = 1504.00 MiB, K (f16):  752.00 MiB, V (f16):  752.00 MiB
llama_new_context_with_model:  CUDA_Host  output buffer size =     1.16 MiB
llama_new_context_with_model:      CUDA0 compute buffer size =  1114.27 MiB
llama_new_context_with_model:      CUDA1 compute buffer size =   224.01 MiB
llama_new_context_with_model:  CUDA_Host compute buffer size =   204.01 MiB
llama_new_context_with_model: graph nodes  = 2149
llama_new_context_with_model: graph splits = 511
Oops(ggml_compute_forward_sum_rows_f32, ffn_moe_weights_sum-1): found -nan for i1 = 0, i2 = 0, i3 = 0. ne00 = 128
```

---

👤 **ikawrakow** commented on **2025-08-15** at **06:24:22**

Based on reports elsewhere, this appears to be due to a corrupted model download.

---

👤 **skrew** commented on **2025-08-22** at **17:04:03**

Not sure I tried with 2 different quant... It would be bad luck that these 2 downloads on HF are corrupted while all the others work. Besides when I use other models on ik_llama they work, it's just the 2 models from `ubergarm`. Unless these models are corrupted from the source?

---

👤 **ikawrakow** commented on **2025-08-23** at **06:08:07**

Is this by any chance fixed on the latest?

[#719](https://github.com/ikawrakow/ik_llama.cpp/issues/719) adds a fix for a problem that may have been the root cause of this issue (if your CPU is `AVX2`).

---

👤 **1aienthusiast** commented on **2025-08-25** at **23:21:54**

I can confirm that on the latest commit, the GLM-4.5-Air-IQ4_KSS-00001-of-00002.gguf quant now works on SillyTavern