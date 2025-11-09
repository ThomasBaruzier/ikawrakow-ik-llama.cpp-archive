## 🔀 [Pull Request #837](https://github.com/ikawrakow/ik_llama.cpp/pull/837) - Ling-1T convert fixup

| **Author** | `ubergarm` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ug/convert-ling-1t` |
| **Target Branch** | `main` |
| **Created** | 2025-10-16 |
| **Updated** | 2025-10-17 |
| **Merged** | 2025-10-17 |

---

## 📄 Description

So far this is just a small patch to conditionally detect `moe_shared_expert_intermediate_size` in config.json and call `gguf_writer` as appropriate. This assumes the value isn't actually needed...

There may be more things that come up, but for now this is enough to get the process going with Ling-1T.

---

## 💬 Conversation

👤 **CISC** commented on **2025-10-16** at **20:04:02**

Ooohh, that was unexpected, this change is not enough though, the converted model will fail to load, I'll look closer at this and make a fix.

---

👤 **CISC** commented on **2025-10-16** at **20:11:07**

Ok, the proper fix is to use `moe_intermediate_size * num_shared_experts` if `moe_shared_expert_intermediate_size` is missing.

---

👤 **CISC** commented on **2025-10-16** at **20:13:21**

Maybe also handle the absence of shared experts, but I think we can leave that as a future problem.

---

👤 **CISC** commented on **2025-10-16** at **20:20:34**

See https://github.com/ggml-org/llama.cpp/commit/950e96470d6f1e900ac24eea5bd9080721147567

---

👤 **ubergarm** commented on **2025-10-16** at **20:52:49**

@CISC 

Thanks so much! I believe I can let the current run finish, and then run it again (into a different directory so it doesn't overwrite anything), and grab just the first GGUF which contains the kv metadata then cancel and copy over the first shard.

```
Shard (17/46):  40%|████      | 17.2G/43.0G [02:19<02:57, 145Mbyte/s]
Writing:  36%|███▌      | 714G/2.00T [1:06:30<2:27:49, 145Mbyte/s]
```

---

👤 **CISC** commented on **2025-10-16** at **20:55:29**

> Thanks so much! I believe I can let the current run finish, and then run it again (into a different directory so it doesn't overwrite anything), and grab just the first GGUF which contains the kv metadata then cancel and copy over the first shard.

Yep, sharding has saved many a day. :D

---

👤 **ubergarm** commented on **2025-10-17** at **04:13:55**

Seems like it converted fine from bf16 safetensors to bf16 GGUF, and was able to quantize a pure Q8_0 Ling-1T-Q8_0.gguf. Went straight to calculating imatrix now.

The routed experts are a bit unruly with missing data, but it settled out after 50 chunks of my usual [ubergarm-imatrix-calibration-corpus-v02.txt](https://gist.github.com/ubergarm/edfeb3ff9c6ec8b49e88cdf627b0711a?permalink_comment_id=5682584#gistcomment-5682584).

I didn't try inferencing with llama-server yet, but the per chunk perplexity values look good :crossed_fingers:

Command and truncated logs below:

<details>

<summary>👈 Details</summary>

```bash
model=/mnt/data/models/ubergarm/Ling-1T-GGUF/Ling-1T-Q8_0.gguf
numactl --interleave=all \
./build/bin/llama-imatrix \
    -m "$model" \
    -fa \
    --no-fused-up-gate \
    -ger \
    -f ubergarm-imatrix-calibration-corpus-v02.txt \
    -o /mnt/data/models/ubergarm/Ling-1T-GGUF/imatrix-Ling-1T-Q8_0.dat \
    --verbosity 1 \
    --layer-similarity \
    --seed 1337 \
    --ctx-size 512 \
    -ub 4096 -b 4096 \
    --numa distribute \
    --threads 128 \
    --threads-batch 192 \
    --no-mmap

# i know seed is not used here, i keep it just for amusement...

llama_model_loader: loaded meta data with 43 key-value pairs and 1103 tensors from /mnt/data/models/ubergarm/Ling-1T-GGUF/Ling-1T-Q8_0.gguf (version GGUF V3 (latest))
llama_model_loader: Dumping metadata keys/values. Note: KV overrides do not apply in this output.
llama_model_loader: - kv   0:                       general.architecture str              = bailingmoe2
llama_model_loader: - kv   1:                               general.type str              = model
llama_model_loader: - kv   2:                               general.name str              = Ling 1T
llama_model_loader: - kv   3:                           general.basename str              = Ling
llama_model_loader: - kv   4:                         general.size_label str              = 1T
llama_model_loader: - kv   5:                    bailingmoe2.block_count u32              = 80
llama_model_loader: - kv   6:                 bailingmoe2.context_length u32              = 32768
llama_model_loader: - kv   7:               bailingmoe2.embedding_length u32              = 8192
llama_model_loader: - kv   8:            bailingmoe2.feed_forward_length u32              = 18432
llama_model_loader: - kv   9:           bailingmoe2.attention.head_count u32              = 64
llama_model_loader: - kv  10:        bailingmoe2.attention.head_count_kv u32              = 8
llama_model_loader: - kv  11:                 bailingmoe2.rope.freq_base f32              = 600000.000000
llama_model_loader: - kv  12: bailingmoe2.attention.layer_norm_rms_epsilon f32              = 0.000001
llama_model_loader: - kv  13:              bailingmoe2.expert_used_count u32              = 8
llama_model_loader: - kv  14:           bailingmoe2.attention.key_length u32              = 128
llama_model_loader: - kv  15:         bailingmoe2.attention.value_length u32              = 128
llama_model_loader: - kv  16:                          general.file_type u32              = 7
llama_model_loader: - kv  17:           bailingmoe2.rope.dimension_count u32              = 64
llama_model_loader: - kv  18:              bailingmoe2.rope.scaling.type str              = none
llama_model_loader: - kv  19:      bailingmoe2.leading_dense_block_count u32              = 4
llama_model_loader: - kv  20:                     bailingmoe2.vocab_size u32              = 157184
llama_model_loader: - kv  21:     bailingmoe2.expert_feed_forward_length u32              = 2048
llama_model_loader: - kv  22: bailingmoe2.expert_shared_feed_forward_length u32              = 2048
llama_model_loader: - kv  23:           bailingmoe2.expert_weights_scale f32              = 2.500000
llama_model_loader: - kv  24:                   bailingmoe2.expert_count u32              = 256
llama_model_loader: - kv  25:            bailingmoe2.expert_shared_count u32              = 1
llama_model_loader: - kv  26:             bailingmoe2.expert_group_count u32              = 8
llama_model_loader: - kv  27:        bailingmoe2.expert_group_used_count u32              = 4
llama_model_loader: - kv  28:            bailingmoe2.expert_weights_norm bool             = true
llama_model_loader: - kv  29:             bailingmoe2.expert_gating_func u32              = 2
llama_model_loader: - kv  30:                       tokenizer.ggml.model str              = gpt2
llama_model_loader: - kv  31:                         tokenizer.ggml.pre str              = bailingmoe2
llama_model_loader: - kv  32:                      tokenizer.ggml.tokens arr[str,157184]  = ["!", "\"", "#", "$", "%", "&", "'", ...
llama_model_loader: - kv  33:                  tokenizer.ggml.token_type arr[i32,157184]  = [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, ...
llama_model_loader: - kv  34:                      tokenizer.ggml.merges arr[str,156635]  = ["Ġ Ġ", "Ġ t", "i n", "Ġ a", "h e...
llama_model_loader: - kv  35:                tokenizer.ggml.bos_token_id u32              = 156891
llama_model_loader: - kv  36:                tokenizer.ggml.eos_token_id u32              = 156895
llama_model_loader: - kv  37:            tokenizer.ggml.padding_token_id u32              = 156892
llama_model_loader: - kv  38:                tokenizer.ggml.cls_token_id u32              = 156893
llama_model_loader: - kv  39:               tokenizer.ggml.add_bos_token bool             = false
llama_model_loader: - kv  40:               tokenizer.ggml.add_eos_token bool             = false
llama_model_loader: - kv  41:                    tokenizer.chat_template str              = {% set thinking_option = 'off' %}\n{{-...
llama_model_loader: - kv  42:               general.quantization_version u32              = 2
llama_model_loader: - type  f32:  473 tensors
llama_model_loader: - type q8_0:  630 tensors
init_tokenizer: initializing tokenizer for type 2
load: special_eos_id is not in special_eog_ids - the tokenizer config may be incorrect
load: printing all EOG tokens:
load:   - 156892 ('<|endoftext|>')
load:   - 156895 ('<|role_end|>')
load: special tokens cache size = 262
load: token to piece cache size = 1.0010 MB
llm_load_print_meta: format           = GGUF V3 (latest)
llm_load_print_meta: arch             = bailingmoe2
llm_load_print_meta: n_ctx_train      = 32768
llm_load_print_meta: n_embd           = 8192
llm_load_print_meta: n_layer          = 80
llm_load_print_meta: n_head           = 64
llm_load_print_meta: n_head_kv        = 8
llm_load_print_meta: n_rot            = 64
llm_load_print_meta: n_swa            = 0
llm_load_print_meta: n_swa_pattern    = 1
llm_load_print_meta: n_embd_head_k    = 128
llm_load_print_meta: n_embd_head_v    = 128
llm_load_print_meta: n_gqa            = 8
llm_load_print_meta: n_embd_k_gqa     = 1024
llm_load_print_meta: n_embd_v_gqa     = 1024
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
llm_load_print_meta: rope type        = 2
llm_load_print_meta: rope scaling     = none
llm_load_print_meta: freq_base_train  = 600000.0
llm_load_print_meta: freq_scale_train = 1
llm_load_print_meta: n_ctx_orig_yarn  = 32768
llm_load_print_meta: rope_finetuned   = unknown
llm_load_print_meta: ssm_d_conv       = 0
llm_load_print_meta: ssm_d_inner      = 0
llm_load_print_meta: ssm_d_state      = 0
llm_load_print_meta: ssm_dt_rank      = 0
llm_load_print_meta: model type       = ?B
llm_load_print_meta: model ftype      = Q8_0
llm_load_print_meta: model params     = 999.705 B
llm_load_print_meta: model size       = 989.678 GiB (8.504 BPW) 
llm_load_print_meta: repeating layers = 987.130 GiB (8.504 BPW, 997.130 B parameters)
llm_load_print_meta: general.name     = Ling 1T
llm_load_print_meta: n_layer_dense_lead   = 4
llm_load_print_meta: n_ff_exp             = 2048
llm_load_print_meta: n_ff_shexp           = 2048
llm_load_print_meta: n_expert_shared      = 1
llm_load_print_meta: n_expert_groups      = 8
llm_load_print_meta: n_group_used         = 4
llm_load_print_meta: expert_weights_scale = 2.5
llm_load_print_meta: expert_weights_norm  = 1
llm_load_print_meta: expert_gating_func   = sigmoid
llm_load_print_meta: nextn_predict_layers = 0
print_info: vocab type       = BPE
print_info: n_vocab          = 157184
print_info: n_merges         = 156635
print_info: BOS token        = 156891 '<|startoftext|>'
print_info: EOS token        = 156895 '<|role_end|>'
print_info: EOT token        = 156892 '<|endoftext|>'
print_info: PAD token        = 156892 '<|endoftext|>'
print_info: LF token         = 198 'Ċ'
print_info: EOG token        = 156892 '<|endoftext|>'
print_info: EOG token        = 156895 '<|role_end|>'
print_info: max token length = 154
llm_load_tensors: ggml ctx size =    0.47 MiB
llm_load_tensors:        CPU buffer size = 1013430.68 MiB
....................................................................................................
llama_new_context_with_model: n_ctx      = 512
llama_new_context_with_model: n_batch    = 512
llama_new_context_with_model: n_ubatch   = 512
llama_new_context_with_model: flash_attn = 1
llama_new_context_with_model: mla_attn   = 0
llama_new_context_with_model: attn_max_b = 0
llama_new_context_with_model: fused_moe  = 0
llama_new_context_with_model: grouped er = 1
llama_new_context_with_model: fused_up_gate = 0
llama_new_context_with_model: ser        = -1, 0
llama_new_context_with_model: freq_base  = 600000.0
llama_new_context_with_model: freq_scale = 1
llama_kv_cache_init:        CPU KV buffer size =   160.00 MiB
llama_new_context_with_model: KV self size  =  160.00 MiB, K (f16):   80.00 MiB, V (f16):   80.00 MiB
llama_new_context_with_model:        CPU  output buffer size =     0.60 MiB
llama_new_context_with_model:        CPU compute buffer size =   323.00 MiB
llama_new_context_with_model: graph nodes  = 3681
llama_new_context_with_model: graph splits = 1

system_info: n_threads = 128 (n_threads_batch = 192) / 768 | AVX = 1 | AVX_VNNI = 1 | AVX2 = 1 | AVX512 = 1 | AVX512_VBMI = 1 | AVX512_VNNI = 1 | AVX512_BF16 = 1 | FMA = 1 | NEON = 0 | SVE = 0 | ARM_FMA = 0 | F16C = 1 | FP16_VA = 0 | WASM_SIMD = 0 | BLAS = 0 | SSE3 = 1 | SSSE3 = 1 | VSX = 0 | MATMUL_INT8 = 0 | LLAMAFILE = 1 | 
compute_imatrix: tokenizing the input ..
compute_imatrix: tokenization took 890.437 ms
compute_imatrix: computing over 864 chunks with batch_size 512
compute_imatrix: 16.03 seconds per pass - ETA 3 hours 50.87 minutes
======================================= HAVE_FANCY_SIMD is defined
[1]5.1943,[2]5.9431,[3]3.8346,[4]2.9940,[5]2.7136,[6]2.3430,[7]2.1198,[8]1.9431,[9]1.8248,
save_imatrix: entry '             blk.79.ffn_down_exps.weight' has partial data (91.41%) 22 out of 256 experts are missing data - skipping
save_imatrix: entry '               blk.79.ffn_up_exps.weight' has partial data (91.41%) 22 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.78.ffn_down_exps.weight' has partial data (92.97%) 18 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.78.ffn_gate_exps.weight' has partial data (92.97%) 18 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.77.ffn_down_exps.weight' has partial data (91.80%) 21 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.76.ffn_down_exps.weight' has partial data (94.14%) 15 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.76.ffn_gate_exps.weight' has partial data (94.14%) 15 out of 256 experts are missing data - skipping
save_imatrix: entry '               blk.76.ffn_up_exps.weight' has partial data (94.14%) 15 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.75.ffn_gate_exps.weight' has partial data (92.97%) 18 out of 256 experts are missing data - skipping
save_imatrix: entry '               blk.75.ffn_up_exps.weight' has partial data (92.97%) 18 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.74.ffn_down_exps.weight' has partial data (92.58%) 19 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.73.ffn_gate_exps.weight' has partial data (95.31%) 12 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.73.ffn_up_exps.weight' has partial data (95.31%) 12 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.72.ffn_gate_exps.weight' has partial data (96.09%) 10 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.72.ffn_up_exps.weight' has partial data (96.09%) 10 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.75.ffn_down_exps.weight' has partial data (92.97%) 18 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.71.ffn_down_exps.weight' has partial data (95.70%) 11 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.71.ffn_gate_exps.weight' has partial data (95.70%) 11 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.70.ffn_gate_exps.weight' has partial data (95.31%) 12 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.69.ffn_gate_exps.weight' has partial data (94.53%) 14 out of 256 experts are missing data - skipping
save_imatrix: entry '               blk.69.ffn_up_exps.weight' has partial data (94.53%) 14 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.68.ffn_down_exps.weight' has partial data (94.92%) 13 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.68.ffn_gate_exps.weight' has partial data (94.92%) 13 out of 256 experts are missing data - skipping
save_imatrix: entry '               blk.68.ffn_up_exps.weight' has partial data (94.92%) 13 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.67.ffn_down_exps.weight' has partial data (96.09%) 10 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.67.ffn_gate_exps.weight' has partial data (96.09%) 10 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.67.ffn_up_exps.weight' has partial data (96.09%) 10 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.77.ffn_up_exps.weight' has partial data (91.80%) 21 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.66.ffn_down_exps.weight' has partial data (95.70%) 11 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.65.ffn_gate_exps.weight' has partial data (95.31%) 12 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.65.ffn_up_exps.weight' has partial data (95.31%) 12 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.77.ffn_gate_exps.weight' has partial data (91.80%) 21 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.64.ffn_down_exps.weight' has partial data (94.53%) 14 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.64.ffn_gate_exps.weight' has partial data (94.53%) 14 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.63.ffn_down_exps.weight' has partial data (96.09%) 10 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.78.ffn_up_exps.weight' has partial data (92.97%) 18 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.63.ffn_gate_exps.weight' has partial data (96.09%) 10 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.63.ffn_up_exps.weight' has partial data (96.09%) 10 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.62.ffn_down_exps.weight' has partial data (96.48%) 9 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.29.ffn_down_exps.weight' has partial data (98.44%) 4 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.42.ffn_up_exps.weight' has partial data (96.09%) 10 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.31.ffn_up_exps.weight' has partial data (98.44%) 4 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.27.ffn_down_exps.weight' has partial data (98.44%) 4 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.27.ffn_gate_exps.weight' has partial data (98.44%) 4 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.27.ffn_up_exps.weight' has partial data (98.44%) 4 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.25.ffn_down_exps.weight' has partial data (99.61%) 1 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.25.ffn_gate_exps.weight' has partial data (99.61%) 1 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.23.ffn_gate_exps.weight' has partial data (99.61%) 1 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.23.ffn_up_exps.weight' has partial data (99.61%) 1 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.53.ffn_up_exps.weight' has partial data (96.48%) 9 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.44.ffn_gate_exps.weight' has partial data (94.53%) 14 out of 256 experts are missing data - skipping
save_imatrix: entry '               blk.39.ffn_up_exps.weight' has partial data (96.48%) 9 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.20.ffn_up_exps.weight' has partial data (99.61%) 1 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.28.ffn_up_exps.weight' has partial data (99.22%) 2 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.21.ffn_gate_exps.weight' has partial data (99.61%) 1 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.19.ffn_up_exps.weight' has partial data (99.61%) 1 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.18.ffn_down_exps.weight' has partial data (99.61%) 1 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.17.ffn_down_exps.weight' has partial data (99.61%) 1 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.17.ffn_gate_exps.weight' has partial data (99.61%) 1 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.62.ffn_gate_exps.weight' has partial data (96.48%) 9 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.71.ffn_up_exps.weight' has partial data (95.70%) 11 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.51.ffn_down_exps.weight' has partial data (93.75%) 16 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.79.ffn_gate_exps.weight' has partial data (91.41%) 22 out of 256 experts are missing data - skipping
save_imatrix: entry '               blk.66.ffn_up_exps.weight' has partial data (95.70%) 11 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.25.ffn_up_exps.weight' has partial data (99.61%) 1 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.21.ffn_up_exps.weight' has partial data (99.61%) 1 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.20.ffn_gate_exps.weight' has partial data (99.61%) 1 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.49.ffn_up_exps.weight' has partial data (96.09%) 10 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '              blk.7.ffn_down_exps.weight' has partial data (99.61%) 1 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.15.ffn_up_exps.weight' has partial data (99.61%) 1 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.24.ffn_gate_exps.weight' has partial data (98.83%) 3 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.38.ffn_up_exps.weight' has partial data (97.66%) 6 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.41.ffn_up_exps.weight' has partial data (96.09%) 10 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.29.ffn_gate_exps.weight' has partial data (98.44%) 4 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.58.ffn_up_exps.weight' has partial data (95.31%) 12 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.24.ffn_up_exps.weight' has partial data (98.83%) 3 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.54.ffn_gate_exps.weight' has partial data (94.92%) 13 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.24.ffn_down_exps.weight' has partial data (98.83%) 3 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.50.ffn_gate_exps.weight' has partial data (94.92%) 13 out of 256 experts are missing data - skipping
save_imatrix: entry '               blk.14.ffn_up_exps.weight' has partial data (98.83%) 3 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.21.ffn_down_exps.weight' has partial data (99.61%) 1 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.69.ffn_down_exps.weight' has partial data (94.53%) 14 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.37.ffn_down_exps.weight' has partial data (96.88%) 8 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '                blk.7.ffn_up_exps.weight' has partial data (99.61%) 1 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.44.ffn_down_exps.weight' has partial data (94.53%) 14 out of 256 experts are missing data - skipping
save_imatrix: entry '               blk.74.ffn_up_exps.weight' has partial data (92.58%) 19 out of 256 experts are missing data - skipping
save_imatrix: entry '               blk.30.ffn_up_exps.weight' has partial data (96.88%) 8 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.23.ffn_down_exps.weight' has partial data (99.61%) 1 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '              blk.4.ffn_gate_exps.weight' has partial data (97.66%) 6 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.56.ffn_up_exps.weight' has partial data (94.53%) 14 out of 256 experts are missing data - skipping
save_imatrix: entry '               blk.59.ffn_up_exps.weight' has partial data (95.70%) 11 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '                blk.4.ffn_up_exps.weight' has partial data (97.66%) 6 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '              blk.7.ffn_gate_exps.weight' has partial data (99.61%) 1 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.36.ffn_gate_exps.weight' has partial data (97.27%) 7 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.41.ffn_gate_exps.weight' has partial data (96.09%) 10 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.15.ffn_gate_exps.weight' has partial data (99.61%) 1 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.14.ffn_gate_exps.weight' has partial data (98.83%) 3 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.64.ffn_up_exps.weight' has partial data (94.53%) 14 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.28.ffn_gate_exps.weight' has partial data (99.22%) 2 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.58.ffn_down_exps.weight' has partial data (95.31%) 12 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.18.ffn_up_exps.weight' has partial data (99.61%) 1 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.70.ffn_up_exps.weight' has partial data (95.31%) 12 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.17.ffn_up_exps.weight' has partial data (99.61%) 1 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.37.ffn_up_exps.weight' has partial data (96.88%) 8 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.55.ffn_up_exps.weight' has partial data (94.92%) 13 out of 256 experts are missing data - skipping
save_imatrix: entry '               blk.29.ffn_up_exps.weight' has partial data (98.44%) 4 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.48.ffn_down_exps.weight' has partial data (95.70%) 11 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '              blk.4.ffn_down_exps.weight' has partial data (97.66%) 6 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.61.ffn_down_exps.weight' has partial data (95.31%) 12 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.73.ffn_down_exps.weight' has partial data (95.31%) 12 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.38.ffn_down_exps.weight' has partial data (97.66%) 6 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.18.ffn_gate_exps.weight' has partial data (99.61%) 1 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.52.ffn_up_exps.weight' has partial data (94.92%) 13 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.14.ffn_down_exps.weight' has partial data (98.83%) 3 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.44.ffn_up_exps.weight' has partial data (94.53%) 14 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.49.ffn_down_exps.weight' has partial data (96.09%) 10 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.51.ffn_up_exps.weight' has partial data (93.75%) 16 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.15.ffn_down_exps.weight' has partial data (99.61%) 1 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.30.ffn_gate_exps.weight' has partial data (96.88%) 8 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.62.ffn_up_exps.weight' has partial data (96.48%) 9 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.47.ffn_down_exps.weight' has partial data (94.92%) 13 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.43.ffn_gate_exps.weight' has partial data (94.92%) 13 out of 256 experts are missing data - skipping
save_imatrix: entry '               blk.60.ffn_up_exps.weight' has partial data (94.92%) 13 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.20.ffn_down_exps.weight' has partial data (99.61%) 1 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.40.ffn_gate_exps.weight' has partial data (96.48%) 9 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.31.ffn_gate_exps.weight' has partial data (98.44%) 4 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.31.ffn_down_exps.weight' has partial data (98.44%) 4 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.46.ffn_gate_exps.weight' has partial data (92.97%) 18 out of 256 experts are missing data - skipping
save_imatrix: entry '               blk.33.ffn_up_exps.weight' has partial data (98.83%) 3 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.33.ffn_down_exps.weight' has partial data (98.83%) 3 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.34.ffn_up_exps.weight' has partial data (99.61%) 1 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.61.ffn_up_exps.weight' has partial data (95.31%) 12 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.34.ffn_gate_exps.weight' has partial data (99.61%) 1 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.34.ffn_down_exps.weight' has partial data (99.61%) 1 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.30.ffn_down_exps.weight' has partial data (96.88%) 8 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.51.ffn_gate_exps.weight' has partial data (93.75%) 16 out of 256 experts are missing data - skipping
save_imatrix: entry '               blk.35.ffn_up_exps.weight' has partial data (98.44%) 4 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.35.ffn_down_exps.weight' has partial data (98.44%) 4 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.66.ffn_gate_exps.weight' has partial data (95.70%) 11 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.36.ffn_up_exps.weight' has partial data (97.27%) 7 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.36.ffn_down_exps.weight' has partial data (97.27%) 7 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.50.ffn_up_exps.weight' has partial data (94.92%) 13 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.37.ffn_gate_exps.weight' has partial data (96.88%) 8 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.39.ffn_down_exps.weight' has partial data (96.48%) 9 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.40.ffn_down_exps.weight' has partial data (96.48%) 9 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.38.ffn_gate_exps.weight' has partial data (97.66%) 6 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.40.ffn_up_exps.weight' has partial data (96.48%) 9 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.39.ffn_gate_exps.weight' has partial data (96.48%) 9 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.28.ffn_down_exps.weight' has partial data (99.22%) 2 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.74.ffn_gate_exps.weight' has partial data (92.58%) 19 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.41.ffn_down_exps.weight' has partial data (96.09%) 10 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.42.ffn_gate_exps.weight' has partial data (96.09%) 10 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.35.ffn_gate_exps.weight' has partial data (98.44%) 4 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.42.ffn_down_exps.weight' has partial data (96.09%) 10 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.55.ffn_down_exps.weight' has partial data (94.92%) 13 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.46.ffn_down_exps.weight' has partial data (92.97%) 18 out of 256 experts are missing data - skipping
save_imatrix: entry '               blk.43.ffn_up_exps.weight' has partial data (94.92%) 13 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.19.ffn_gate_exps.weight' has partial data (99.61%) 1 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.43.ffn_down_exps.weight' has partial data (94.92%) 13 out of 256 experts are missing data - skipping
save_imatrix: entry '               blk.45.ffn_up_exps.weight' has partial data (94.53%) 14 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.45.ffn_gate_exps.weight' has partial data (94.53%) 14 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.45.ffn_down_exps.weight' has partial data (94.53%) 14 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.19.ffn_down_exps.weight' has partial data (99.61%) 1 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.46.ffn_up_exps.weight' has partial data (92.97%) 18 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.47.ffn_gate_exps.weight' has partial data (94.92%) 13 out of 256 experts are missing data - skipping
save_imatrix: entry '               blk.48.ffn_up_exps.weight' has partial data (95.70%) 11 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.65.ffn_down_exps.weight' has partial data (95.31%) 12 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.53.ffn_down_exps.weight' has partial data (96.48%) 9 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.47.ffn_up_exps.weight' has partial data (94.92%) 13 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.72.ffn_down_exps.weight' has partial data (96.09%) 10 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.52.ffn_gate_exps.weight' has partial data (94.92%) 13 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.52.ffn_down_exps.weight' has partial data (94.92%) 13 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.56.ffn_down_exps.weight' has partial data (94.53%) 14 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.53.ffn_gate_exps.weight' has partial data (96.48%) 9 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.54.ffn_up_exps.weight' has partial data (94.92%) 13 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.59.ffn_down_exps.weight' has partial data (95.70%) 11 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.54.ffn_down_exps.weight' has partial data (94.92%) 13 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.70.ffn_down_exps.weight' has partial data (95.31%) 12 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.55.ffn_gate_exps.weight' has partial data (94.92%) 13 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.56.ffn_gate_exps.weight' has partial data (94.53%) 14 out of 256 experts are missing data - skipping
save_imatrix: entry '               blk.57.ffn_up_exps.weight' has partial data (94.53%) 14 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.49.ffn_gate_exps.weight' has partial data (96.09%) 10 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.57.ffn_gate_exps.weight' has partial data (94.53%) 14 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.57.ffn_down_exps.weight' has partial data (94.53%) 14 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.50.ffn_down_exps.weight' has partial data (94.92%) 13 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.48.ffn_gate_exps.weight' has partial data (95.70%) 11 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.58.ffn_gate_exps.weight' has partial data (95.31%) 12 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.60.ffn_gate_exps.weight' has partial data (94.92%) 13 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.59.ffn_gate_exps.weight' has partial data (95.70%) 11 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.60.ffn_down_exps.weight' has partial data (94.92%) 13 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.33.ffn_gate_exps.weight' has partial data (98.83%) 3 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.61.ffn_gate_exps.weight' has partial data (95.31%) 12 out of 256 experts are missing data Storing **but be aware**
save_imatrix: warning: storing only 639 out of 705 entries

save_imatrix: stored collected data after 10 chunks in /mnt/data/models/ubergarm/Ling-1T-GGUF/imatrix-Ling-1T-Q8_0.dat
[10]1.7215,[11]1.6757,[12]1.6679,[13]1.7836,[14]1.8437,[15]1.8665,[16]1.8998,[17]1.8512,[18]1.7926,[19]1.7478,
save_imatrix: entry '             blk.79.ffn_down_exps.weight' has partial data (92.58%) 19 out of 256 experts are missing data - skipping
save_imatrix: entry '               blk.79.ffn_up_exps.weight' has partial data (92.58%) 19 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.78.ffn_down_exps.weight' has partial data (94.92%) 13 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.78.ffn_gate_exps.weight' has partial data (94.92%) 13 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.77.ffn_down_exps.weight' has partial data (92.97%) 18 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.76.ffn_down_exps.weight' has partial data (95.70%) 11 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.76.ffn_gate_exps.weight' has partial data (95.70%) 11 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.76.ffn_up_exps.weight' has partial data (95.70%) 11 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.75.ffn_gate_exps.weight' has partial data (94.92%) 13 out of 256 experts are missing data - skipping
save_imatrix: entry '               blk.75.ffn_up_exps.weight' has partial data (94.92%) 13 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.74.ffn_down_exps.weight' has partial data (94.92%) 13 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.75.ffn_down_exps.weight' has partial data (94.92%) 13 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.69.ffn_gate_exps.weight' has partial data (96.09%) 10 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.69.ffn_up_exps.weight' has partial data (96.09%) 10 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.68.ffn_down_exps.weight' has partial data (96.09%) 10 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.68.ffn_gate_exps.weight' has partial data (96.09%) 10 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.68.ffn_up_exps.weight' has partial data (96.09%) 10 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.77.ffn_up_exps.weight' has partial data (92.97%) 18 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.77.ffn_gate_exps.weight' has partial data (92.97%) 18 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.64.ffn_down_exps.weight' has partial data (96.48%) 9 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.64.ffn_gate_exps.weight' has partial data (96.48%) 9 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.78.ffn_up_exps.weight' has partial data (94.92%) 13 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.44.ffn_gate_exps.weight' has partial data (97.66%) 6 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.51.ffn_down_exps.weight' has partial data (94.53%) 14 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.79.ffn_gate_exps.weight' has partial data (92.58%) 19 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.54.ffn_gate_exps.weight' has partial data (96.48%) 9 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.50.ffn_gate_exps.weight' has partial data (97.27%) 7 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.69.ffn_down_exps.weight' has partial data (96.09%) 10 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.44.ffn_down_exps.weight' has partial data (97.66%) 6 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.74.ffn_up_exps.weight' has partial data (94.92%) 13 out of 256 experts are missing data - skipping
save_imatrix: entry '               blk.56.ffn_up_exps.weight' has partial data (94.92%) 13 out of 256 experts are missing data - skipping
save_imatrix: entry '               blk.64.ffn_up_exps.weight' has partial data (96.48%) 9 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.55.ffn_up_exps.weight' has partial data (96.88%) 8 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.52.ffn_up_exps.weight' has partial data (96.88%) 8 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.44.ffn_up_exps.weight' has partial data (97.66%) 6 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.51.ffn_up_exps.weight' has partial data (94.53%) 14 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.47.ffn_down_exps.weight' has partial data (95.70%) 11 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.43.ffn_gate_exps.weight' has partial data (97.27%) 7 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.60.ffn_up_exps.weight' has partial data (95.70%) 11 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.46.ffn_gate_exps.weight' has partial data (96.09%) 10 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.51.ffn_gate_exps.weight' has partial data (94.53%) 14 out of 256 experts are missing data - skipping
save_imatrix: entry '               blk.50.ffn_up_exps.weight' has partial data (97.27%) 7 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.74.ffn_gate_exps.weight' has partial data (94.92%) 13 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.55.ffn_down_exps.weight' has partial data (96.88%) 8 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.46.ffn_down_exps.weight' has partial data (96.09%) 10 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.43.ffn_up_exps.weight' has partial data (97.27%) 7 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.43.ffn_down_exps.weight' has partial data (97.27%) 7 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.45.ffn_up_exps.weight' has partial data (97.27%) 7 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.45.ffn_gate_exps.weight' has partial data (97.27%) 7 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.45.ffn_down_exps.weight' has partial data (97.27%) 7 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.46.ffn_up_exps.weight' has partial data (96.09%) 10 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.47.ffn_gate_exps.weight' has partial data (95.70%) 11 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.47.ffn_up_exps.weight' has partial data (95.70%) 11 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.52.ffn_gate_exps.weight' has partial data (96.88%) 8 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.52.ffn_down_exps.weight' has partial data (96.88%) 8 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.56.ffn_down_exps.weight' has partial data (94.92%) 13 out of 256 experts are missing data - skipping
save_imatrix: entry '               blk.54.ffn_up_exps.weight' has partial data (96.48%) 9 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.54.ffn_down_exps.weight' has partial data (96.48%) 9 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.55.ffn_gate_exps.weight' has partial data (96.88%) 8 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.56.ffn_gate_exps.weight' has partial data (94.92%) 13 out of 256 experts are missing data - skipping
save_imatrix: entry '               blk.57.ffn_up_exps.weight' has partial data (97.66%) 6 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.57.ffn_gate_exps.weight' has partial data (97.66%) 6 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.57.ffn_down_exps.weight' has partial data (97.66%) 6 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.50.ffn_down_exps.weight' has partial data (97.27%) 7 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.60.ffn_gate_exps.weight' has partial data (95.70%) 11 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.60.ffn_down_exps.weight' has partial data (95.70%) 11 out of 256 experts are missing data Storing **but be aware**
save_imatrix: warning: storing only 684 out of 705 entries

save_imatrix: stored collected data after 20 chunks in /mnt/data/models/ubergarm/Ling-1T-GGUF/imatrix-Ling-1T-Q8_0.dat
[20]1.7036,[21]1.6736,[22]1.6445,[23]1.6221,[24]1.5897,[25]1.5658,[26]1.5504,[27]1.5339,[28]1.5108,[29]1.5538,
save_imatrix: entry '             blk.79.ffn_down_exps.weight' has partial data (92.97%) 18 out of 256 experts are missing data - skipping
save_imatrix: entry '               blk.79.ffn_up_exps.weight' has partial data (92.97%) 18 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.78.ffn_down_exps.weight' has partial data (95.31%) 12 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.78.ffn_gate_exps.weight' has partial data (95.31%) 12 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.77.ffn_down_exps.weight' has partial data (93.75%) 16 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.75.ffn_gate_exps.weight' has partial data (96.09%) 10 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.75.ffn_up_exps.weight' has partial data (96.09%) 10 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.74.ffn_down_exps.weight' has partial data (96.48%) 9 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.75.ffn_down_exps.weight' has partial data (96.09%) 10 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.77.ffn_up_exps.weight' has partial data (93.75%) 16 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.77.ffn_gate_exps.weight' has partial data (93.75%) 16 out of 256 experts are missing data - skipping
save_imatrix: entry '               blk.78.ffn_up_exps.weight' has partial data (95.31%) 12 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.51.ffn_down_exps.weight' has partial data (94.92%) 13 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.79.ffn_gate_exps.weight' has partial data (92.97%) 18 out of 256 experts are missing data - skipping
save_imatrix: entry '               blk.74.ffn_up_exps.weight' has partial data (96.48%) 9 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.56.ffn_up_exps.weight' has partial data (95.70%) 11 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.51.ffn_up_exps.weight' has partial data (94.92%) 13 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.51.ffn_gate_exps.weight' has partial data (94.92%) 13 out of 256 experts are missing data - skipping
save_imatrix: entry '             blk.74.ffn_gate_exps.weight' has partial data (96.48%) 9 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.56.ffn_down_exps.weight' has partial data (95.70%) 11 out of 256 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.56.ffn_gate_exps.weight' has partial data (95.70%) 11 out of 256 experts are missing data Storing **but be aware**
save_imatrix: warning: storing only 696 out of 705 entries

save_imatrix: stored collected data after 30 chunks in /mnt/data/models/ubergarm/Ling-1T-GGUF/imatrix-Ling-1T-Q8_0.dat
[30]1.6130,[31]1.6574,[32]1.6895,[33]1.7194,[34]1.7353,[35]1.7721,[36]1.7964,[37]1.8460,
```

</details>

---

👤 **ikawrakow** approved this pull request ✅ on **2025-10-17** at **04:51:52**

---

👤 **ubergarm** commented on **2025-10-17** at **13:33:51**

Okay managed to complete imatrix, but i can't upload to it to huggingface for some reason as I list on the model card: https://huggingface.co/ubergarm/Ling-1T-GGUF (but i include a link for it from it off a personal site anyway if someone else wants to try it).

I left it quantizing the listed `smol-IQ2_KS 264.984 GiB (2.277 BPW)` which gives a low perplexity against wiki.test.raw of `Final estimate: PPL = 2.4429 +/- 0.01191`.

It seems to inference okay in a few short tests, however if I cancel a generation, and then try to send another prompt, its tends to throw `iqk_fa_templates.h:1157: GGML_ASSERT(S > 0) failed`. This seems to happen both with and without `-ger`:

<details>

<summary>👈 Details</summary>

```bash
# compiled cpu-only
model=/mnt/data/models/ubergarm/Ling-1T-GGUF/Ling-1T-smol-IQ2_KS.gguf
numactl -N "$SOCKET" -m "$SOCKET" \
./build/bin/llama-server \
    --model "$model"\
    --alias ubergarm/Ling-1T-smol-IQ2_KS \
    --ctx-size 65536 \
    -fa -fmoe -ger \
    -ctk q8_0 -ctv q8_0 \
    -ub 4096 -b 4096 \
    --parallel 2 \
    --threads 128 \
    --threads-batch 192 \
    --numa numactl \
    --host 127.0.0.1 \
    --port 8080 \
    --no-mmap \
    --no-display-prompt \
    --validate-quants


INFO [            update_slots] all slots are idle | tid="125374515894464" timestamp=1760707131
INFO [      log_server_request] request | tid="125064682927808" timestamp=1760707133 remote_addr="127.0.0.1" remote_port=50932 status=200 method="GET"
 path="/v1/models" params={}
Grammar:
Grammar lazy: false
Chat format: Content-only
INFO [   launch_slot_with_task] slot is processing task | tid="125374515894464" timestamp=1760707137 id_slot=1 id_task=461
INFO [            update_slots] kv cache rm [p0, end) | tid="125374515894464" timestamp=1760707137 id_slot=1 id_task=461 p0=0
/home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1157: /home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1157: /hom
e/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1157: /home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1157: /home/w/
projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1157: /home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1157: GGML_ASSERT(
S > 0) failed/home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1157: /home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates
.h:1157: /home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1157: /home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1
157: /home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1157: /home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1157:
 /home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1157: /home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1157: /ho
me/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1157: /home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1157: /home/w
/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1157: /home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1157: /home/w/pro
jects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1157: /home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1157: /home/w/project
s/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1157: /home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1157: /home/w/projects/ik
_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1157: /home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1157: /home/w/projects/ik_lla
ma.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1157: /home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1157: GGML_ASSERT(S > 0) failed
/home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1157: GGML_ASSERT(S > 0) failed
/home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1157: GGML_ASSERT(S > 0) failed
```

</details>

---

👤 **ikawrakow** commented on **2025-10-17** at **13:46:09**

Looks like something goes wrong with the KV cache when generation is canceled.
Does it also happen with  `--parallel 1` ?

---

👤 **ubergarm** commented on **2025-10-17** at **13:53:07**

> Looks like something goes wrong with the KV cache when generation is canceled. Does it also happen with `--parallel 1` ?

Right, I just dialed it back to `--parallel 1` and have some folks kicking the tires at https://llm.ubergarm.com right now. I'll report back after some coffee!

---

👤 **ubergarm** commented on **2025-10-17** at **14:27:29**

@ikawrakow 

> Does it also happen with --parallel 1 ?

So far so good with `--parallel 1` even canceling/closing my client mid streaming then sending new prompts. Mostly short prompts so far, still having folks poke at it.

---

👤 **ikawrakow** commented on **2025-10-17** at **16:07:35**

> Okay managed to complete imatrix, but i can't upload to it to huggingface for some reason as I list on the model card: https://huggingface.co/ubergarm/Ling-1T-GGUF (but i include a link for it from it off a personal site anyway if someone else wants to try it).

Did they somehow disable upload of `imatrix.dat` files because `llama.cpp` now uses GGUF for that?

---

👤 **ubergarm** commented on **2025-10-17** at **16:21:06**

> Did they somehow disable upload of imatrix.dat files because llama.cpp now uses GGUF for that?

No, i got 403 after uploading 250GB gguf's too 💀. Pretty sure it is the recent changes with public storage limits. I'm in contact with them trying to clear it up to release the splits so other folks can test targeting 256GB RAM + 24GB VRAM rigs on the initial quant.

---

👤 **ubergarm** commented on **2025-10-17** at **19:17:55**

Just did a comparison between ik and mainline implementation in terms of perplexity on wiki.test.raw and seems like both implementations give very similar results within the noise floor:

* ik/main@dbfd1515
   - Final estimate: PPL = 2.5880 +/- 0.01280
* mainline lcpp/cisc/bailingmoe2@bc6c48af1 
  - Final estimate: PPL = 2.5870 +/- 0.01279

The ik_llama.cpp recipe used to cook this `smol-IQ2_XXS` 249.92 GiB (2.15 BPW)

<details>

<summary>👈 Details</summary>

```bash
#!/usr/bin/env bash

custom="
# 80 Repeating Layers [0-79]

# Attention
blk\.(0|1|2|3)\.attn_qkv.*=q8_0
blk\.(0|1|2|3)\.attn_output.*=q8_0
blk\..*\.attn_qkv.*=q6_K
blk\..*\.attn_output.*=q6_K

# First 4 Dense Layers [0-3]
blk\..*\.ffn_down\.weight=q5_K
blk\..*\.ffn_(gate|up)\.weight=q4_K

# Shared Expert Layers [3-79]
blk\..*\.ffn_down_shexp\.weight=q5_K
blk\..*\.ffn_(gate|up)_shexp\.weight=q4_K

# Routed Experts Layers [3-79]
blk\..*\.ffn_down_exps\.weight=iq2_xxs
blk\..*\.ffn_(gate|up)_exps\.weight=iq2_xxs

# Non-Repeating Layers
token_embd\.weight=q4_K
output\.weight=q6_K
"

custom=$(
  echo "$custom" | grep -v '^#' | \
  sed -Ez 's:\n+:,:g;s:,$::;s:^,::'
)

numactl -N ${SOCKET} -m ${SOCKET} \
./build/bin/llama-quantize \
    --custom-q "$custom" \
    --imatrix /mnt/data/models/ubergarm/Ling-1T-GGUF/imatrix-Ling-1T-Q8_0.dat \
    /mnt/data/models/ubergarm/Ling-1T-GGUF/Ling-1T-BF16-00001-of-00046.gguf \
    /mnt/data/models/ubergarm/Ling-1T-GGUF/Ling-1T-smol-IQ2_XXS.gguf \
    IQ2_XXS \
    192
```

</details>

Compare to `smol-IQ2_KS` 264.984 GiB (2.277 BPW)
* Final estimate: PPL = 2.4429 +/- 0.01191

I'll run perplexity on the pure Q8_0 for baseline to see how that is looking...

Still waiting for huggingface to address the public quota 403 issue so i can release....

---

👤 **CISC** commented on **2025-10-17** at **20:32:29**

@ubergarm Hope HF fixes the quota soon. :)

Do you have a good `-ts`/`-ot` strategy for multiple 24GB cards? Unfortunately I don't have enough RAM to run this on my single 24GB card, so can't test locally. :(

---

👤 **ubergarm** commented on **2025-10-17** at **22:58:44**

@CISC

Well in the mean time, there is now this repo which has the imatrix (dat format which should work on both ik and mainline), a `smol-IQ2_KS` for ik_llama.cpp and that `smol-IQ2_XXS` which I tested your open mainline PR:

https://huggingface.co/ubergarm2/Ling-1T-GGUF

Hopefully I can get that namespace fixed up soon to release more sizes, but at least some quants are available for testing.

About the smallest possible model is like 180GB or so with IQ1_S probably for routed exps haha... But I added an example Hybrid CPU+GPU that if you have enough CUDA cards you can just keep offloading more routed experts across them in the pattern shown. Keep in mind layers [0-3] are the dense layers and shexp/exps begin at [4-79].

So tl;dr; hopefully something like this will do the trick, but I am not able to test now:
```bash
      -ngl 99 \
      -ot "blk\.(4|5|6)\.ffn_.*=CUDA0" \
      -ot "blk\.(7|8|9|10|11)\.ffn_.*=CUDA1" \
      -ot exps=CPU \
```

Also congrats @ikawrakow and thanks both of you helping push this out so more folks can continue to test!

<img width="1217" height="616" alt="ling-1t-gguf-first" src="https://github.com/user-attachments/assets/896f3e2b-f55e-4f60-b848-24d5a60b2904" />