## 📌 [Issue #805](https://github.com/ikawrakow/ik_llama.cpp/issues/805) - Bug: Can not load DS V3.1 Terminus on pure CPU

| **Author** | `oovloveme` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-09-28 |
| **Updated** | 2025-09-30 |

---

## 📄 Description

### What happened?

Build command:

cmake -B ./build -DGGML_CUDA=OFF -DGGML_BLAS=OFF
cmake --build ./build --config Release -j 56

Run command:

ee@ee-H12D-8D:~/cpu_ik/ik_llama.cpp$ /home/ee/cpu_ik/ik_llama.cpp/build/bin/llama-server     --model /home/ee/models/DeepSeek-V3.1-Terminus-IQ4_K/IQ4_K/DeepSeek-V3.1-Terminus-IQ4_K-00001-of-00009.gguf     --alias xiaoshuta     -c 20000  -ctk q8_0  -mla 3 -fa  -amb 512     --parallel 1   --threads 56     --host 0.0.0.0 --port 8000
INFO [                    main] build info | tid="128212134979840" timestamp=1759047858 build=3899 commit="3d4977cb"
INFO [                    main] system info | tid="128212134979840" timestamp=1759047858 n_threads=56 n_threads_batch=-1 total_threads=128 system_info="AVX = 1 | AVX_VNNI = 0 | AVX2 = 1 | AVX512 = 0 | AVX512_VBMI = 0 | AVX512_VNNI = 0 | AVX512_BF16 = 0 | FMA = 1 | NEON = 0 | SVE = 0 | ARM_FMA = 0 | F16C = 1 | FP16_VA = 0 | WASM_SIMD = 0 | BLAS = 0 | SSE3 = 1 | SSSE3 = 1 | VSX = 0 | MATMUL_INT8 = 0 | LLAMAFILE = 1 | "
llama_model_loader: additional 8 GGUFs metadata loaded.
llama_model_loader: loaded meta data with 54 key-value pairs and 1086 tensors from /home/ee/models/DeepSeek-V3.1-Terminus-IQ4_K/IQ4_K/DeepSeek-V3.1-Terminus-IQ4_K-00001-of-00009.gguf (version GGUF V3 (latest))
llama_model_loader: Dumping metadata keys/values. Note: KV overrides do not apply in this output.
llama_model_loader: - kv   0:                       general.architecture str              = deepseek2
llama_model_loader: - kv   1:                               general.type str              = model
llama_model_loader: - kv   2:                               general.name str              = DeepSeek V3.1 Terminus Bf16 Safetensors
llama_model_loader: - kv   3:                           general.finetune str              = safetensors
llama_model_loader: - kv   4:                           general.basename str              = DeepSeek-V3.1-Terminus
llama_model_loader: - kv   5:                         general.size_label str              = 256x20B
llama_model_loader: - kv   6:                      deepseek2.block_count u32              = 61
llama_model_loader: - kv   7:                   deepseek2.context_length u32              = 163840
llama_model_loader: - kv   8:                 deepseek2.embedding_length u32              = 7168
llama_model_loader: - kv   9:              deepseek2.feed_forward_length u32              = 18432
llama_model_loader: - kv  10:             deepseek2.attention.head_count u32              = 128
llama_model_loader: - kv  11:          deepseek2.attention.head_count_kv u32              = 1
llama_model_loader: - kv  12:                   deepseek2.rope.freq_base f32              = 10000.000000
llama_model_loader: - kv  13: deepseek2.attention.layer_norm_rms_epsilon f32              = 0.000001
llama_model_loader: - kv  14:                deepseek2.expert_used_count u32              = 8
llama_model_loader: - kv  15:             deepseek2.attention.key_length u32              = 576
llama_model_loader: - kv  16:           deepseek2.attention.value_length u32              = 512
llama_model_loader: - kv  17:                          general.file_type u32              = 140
llama_model_loader: - kv  18:        deepseek2.leading_dense_block_count u32              = 3
llama_model_loader: - kv  19:                       deepseek2.vocab_size u32              = 129280
llama_model_loader: - kv  20:            deepseek2.attention.q_lora_rank u32              = 1536
llama_model_loader: - kv  21:           deepseek2.attention.kv_lora_rank u32              = 512
llama_model_loader: - kv  22:         deepseek2.attention.key_length_mla u32              = 192
llama_model_loader: - kv  23:       deepseek2.attention.value_length_mla u32              = 128
llama_model_loader: - kv  24:       deepseek2.expert_feed_forward_length u32              = 2048
llama_model_loader: - kv  25:                     deepseek2.expert_count u32              = 256
llama_model_loader: - kv  26:              deepseek2.expert_shared_count u32              = 1
llama_model_loader: - kv  27:             deepseek2.expert_weights_scale f32              = 2.500000
llama_model_loader: - kv  28:              deepseek2.expert_weights_norm bool             = true
llama_model_loader: - kv  29:               deepseek2.expert_gating_func u32              = 2
llama_model_loader: - kv  30:             deepseek2.rope.dimension_count u32              = 64
llama_model_loader: - kv  31:                deepseek2.rope.scaling.type str              = yarn
llama_model_loader: - kv  32:              deepseek2.rope.scaling.factor f32              = 40.000000
llama_model_loader: - kv  33: deepseek2.rope.scaling.original_context_length u32              = 4096
llama_model_loader: - kv  34: deepseek2.rope.scaling.yarn_log_multiplier f32              = 0.100000
llama_model_loader: - kv  35:               general.quantization_version u32              = 2
llama_model_loader: - kv  36:                       tokenizer.ggml.model str              = gpt2
llama_model_loader: - kv  37:                         tokenizer.ggml.pre str              = deepseek-v3
llama_model_loader: - kv  38:                      tokenizer.ggml.tokens arr[str,129280]  = ["<｜begin▁of▁sentence｜>", "<▒...
llama_model_loader: - kv  39:                  tokenizer.ggml.token_type arr[i32,129280]  = [3, 3, 3, 1, 1, 1, 1, 1, 1, 1, 1, 1, ...
llama_model_loader: - kv  40:                      tokenizer.ggml.merges arr[str,127741]  = ["Ġ t", "Ġ a", "i n", "Ġ Ġ", "h e...
llama_model_loader: - kv  41:                tokenizer.ggml.bos_token_id u32              = 0
llama_model_loader: - kv  42:                tokenizer.ggml.eos_token_id u32              = 1
llama_model_loader: - kv  43:            tokenizer.ggml.padding_token_id u32              = 1
llama_model_loader: - kv  44:               tokenizer.ggml.add_bos_token bool             = true
llama_model_loader: - kv  45:               tokenizer.ggml.add_eos_token bool             = false
llama_model_loader: - kv  46:                    tokenizer.chat_template str              = {% if not add_generation_prompt is de...
llama_model_loader: - kv  47:                      quantize.imatrix.file str              = /mnt/data/models/ubergarm/DeepSeek-V3...
llama_model_loader: - kv  48:                   quantize.imatrix.dataset str              = ubergarm-imatrix-calibration-corpus-v...
llama_model_loader: - kv  49:             quantize.imatrix.entries_count i32              = 782
llama_model_loader: - kv  50:              quantize.imatrix.chunks_count i32              = 812
llama_model_loader: - kv  51:                                   split.no u16              = 0
llama_model_loader: - kv  52:                                split.count u16              = 9
llama_model_loader: - kv  53:                        split.tensors.count i32              = 1086
llama_model_loader: - type  f32:  361 tensors
llama_model_loader: - type q8_0:  368 tensors
llama_model_loader: - type iq4_k:  117 tensors
llama_model_loader: - type iq5_k:   58 tensors
llama_model_loader: - type iq6_k:  182 tensors
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
llm_load_print_meta: model ftype      = IQ4_K - 4.5 bpw
llm_load_print_meta: model params     = 671.026 B
llm_load_print_meta: model size       = 382.485 GiB (4.896 BPW)
llm_load_print_meta: repeating layers = 381.285 GiB (4.894 BPW, 669.173 B parameters)
llm_load_print_meta: general.name     = DeepSeek V3.1 Terminus Bf16 Safetensors
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
print_info: PAD token        = 1 '<｜end▁of▁sentence｜>'
print_info: LF token         = 201 'Ċ'
print_info: FIM PRE token    = 128801 '<｜fim▁begin｜>'
print_info: FIM SUF token    = 128800 '<｜fim▁hole｜>'
print_info: FIM MID token    = 128802 '<｜fim▁end｜>'
print_info: EOG token        = 1 '<｜end▁of▁sentence｜>'
print_info: max token length = 256
llm_load_tensors: ggml ctx size =    0.45 MiB
llm_load_tensors:        CPU buffer size = 42717.71 MiB
llm_load_tensors:        CPU buffer size = 44498.38 MiB
llm_load_tensors:        CPU buffer size = 42451.26 MiB
llm_load_tensors:        CPU buffer size = 44706.79 MiB
llm_load_tensors:        CPU buffer size = 42451.26 MiB
llm_load_tensors:        CPU buffer size = 44706.79 MiB
llm_load_tensors:        CPU buffer size = 42451.26 MiB
llm_load_tensors:        CPU buffer size = 44706.79 MiB
llm_load_tensors:        CPU buffer size = 42974.71 MiB
....................................................................................................
============ llm_prepare_mla: need to compute 61 wkv_b tensors
Computed blk.0.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
Computed blk.1.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
Computed blk.2.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
Computed blk.3.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
Computed blk.4.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
Computed blk.5.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
Computed blk.6.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
Computed blk.7.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
Computed blk.8.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
Computed blk.9.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
Computed blk.10.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
Computed blk.11.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
Computed blk.12.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
Computed blk.13.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
Computed blk.14.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
Computed blk.15.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
Computed blk.16.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
Computed blk.17.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
Computed blk.18.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
Computed blk.19.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
Computed blk.20.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
Computed blk.21.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
Computed blk.22.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
Computed blk.23.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
Computed blk.24.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
Computed blk.25.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
Computed blk.26.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
Computed blk.27.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
Computed blk.28.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
Computed blk.29.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
Computed blk.30.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
Computed blk.31.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
Computed blk.32.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
Computed blk.33.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
Computed blk.34.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
Computed blk.35.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
Computed blk.36.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
Computed blk.37.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
Computed blk.38.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
Computed blk.39.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
Computed blk.40.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
Computed blk.41.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
Computed blk.42.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
Computed blk.43.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
Computed blk.44.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
Computed blk.45.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
Computed blk.46.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
Computed blk.47.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
Computed blk.48.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
Computed blk.49.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
Computed blk.50.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
Computed blk.51.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
Computed blk.52.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
Computed blk.53.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
Computed blk.54.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
Computed blk.55.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
Computed blk.56.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
Computed blk.57.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
Computed blk.58.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
Computed blk.59.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
Computed blk.60.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
llama_new_context_with_model: n_ctx      = 20224
llama_new_context_with_model: n_batch    = 2048
llama_new_context_with_model: n_ubatch   = 512
llama_new_context_with_model: flash_attn = 1
llama_new_context_with_model: mla_attn   = 3
llama_new_context_with_model: attn_max_b = 512
llama_new_context_with_model: fused_moe  = 0
llama_new_context_with_model: fused_up_gate = 1
llama_new_context_with_model: ser        = -1, 0
llama_new_context_with_model: freq_base  = 10000.0
llama_new_context_with_model: freq_scale = 0.025
llama_kv_cache_init:        CPU KV buffer size =   720.03 MiB
llama_new_context_with_model: KV self size  =  720.03 MiB, c^KV (q8_0):  720.03 MiB, kv^T: not used
llama_new_context_with_model:        CPU  output buffer size =     0.99 MiB
llama_new_context_with_model:        CPU compute buffer size =  1280.15 MiB
llama_new_context_with_model: graph nodes  = 8178
llama_new_context_with_model: graph splits = 1
======================================= HAVE_FANCY_SIMD is NOT defined
Oops(ggml_compute_forward_sum_rows_f32, ffn_moe_weights_sum-19): found -nan for i1 = 0, i2 = 0, i3 = 0. ne00 = 256


### Name and Version

ee@ee-H12D-8D:~/cpu_ik/ik_llama.cpp$ /home/ee/cpu_ik/ik_llama.cpp/build/bin/llama-server --version
version: 3899 (3d4977cb)
built with cc (Ubuntu 13.3.0-6ubuntu2~24.04) 13.3.0 for x86_64-linux-gnu


### What operating system are you seeing the problem on?

_No response_

### Relevant log output

```shell

```

---

## 💬 Conversation

👤 **oovloveme** commented on **2025-09-28** at **08:50:52**

It is ok when I remove "-ctk q8_0“

---

👤 **ubergarm** commented on **2025-09-28** at **16:24:28**

@oovloveme 

Odd... I'd suggest referring to the modelcard example for "CPU-Only" here: https://huggingface.co/ubergarm/DeepSeek-V3.1-Terminus-GGUF#quick-start

Given you are running CPU-only you can remove `-amb 512` as that is for GPU. Also you will want to use `-fmoe` to enable `fused_moe` for better speed. Also consider `--no-mmap` which has slower startup, but imo runs more consistently once it has started and prevents you from accidentally running a model too large to fit into RAM with mmap().

You *should* be able to use `-ctk q8_0` and likely see better speed with it for CPU only case. 

Please run with `--validate-quants` to ensure your download completed successfully. If that throws an error, check each `.gguf` file with `sha256sum` and compare against the huggingface repo reported `sha256sum` to see if you have an incomplete download file.

I just tested that quant on CPU-only backend using a similar command as yours and do not see that error. fwiw I am using an `AMD EPYC 9965`. Maybe you're using an Intel processor given 56 cores?

Thanks for the report, curious to hear how you make out!

---

👤 **ikawrakow** commented on **2025-09-28** at **17:16:33**

@ubergarm  Their CPU is `AVX2` while yours supports all `AVX512` extensions necessary for the "fancy SIMD" path, so FA calculations are done in different ways. If it works with `f16` K-cache but we get NaNs with `Q8_0` K-cache, there maybe an issue with the pure `AVX2` quantized FA implementation. I'll try to look into this in the next days.

---

👤 **oovloveme** commented on **2025-09-28** at **23:48:52**

> [@ubergarm](https://github.com/ubergarm) Their CPU is `AVX2` while yours supports all `AVX512` extensions necessary for the "fancy SIMD" path, so FA calculations are done in different ways. If it works with `f16` K-cache but we get NaNs with `Q8_0` K-cache, there maybe an issue with the pure `AVX2` quantized FA implementation. I'll try to look into this in the next days.

Yes,my CPU is epyc 7B13 (64 cores 128 threads) which doesn't support AVX512.

---

👤 **ikawrakow** commented on **2025-09-29** at **06:09:09**

@oovloveme Can you try adding `--no-warmup` to your command line and let me know if that works? Thanks.

---

👤 **oovloveme** commented on **2025-09-29** at **07:20:28**

> [@oovloveme](https://github.com/oovloveme) Can you try adding `--no-warmup` to your command line and let me know if that works? Thanks.

/home/ee/cpu_ik/ik_llama.cpp/build/bin/llama-server \
    --model /home/ee/models/DeepSeek-V3.1-Terminus-IQ4_K/IQ4_K/DeepSeek-V3.1-Terminus-IQ4_K-00001-of-00009.gguf \
    --alias xiaoshuta \
    -c 40000  -mla 3 -fa  -amb 512 -ctk q8_0 \
    --parallel 1   --threads 56 \
    --host 0.0.0.0 --port 8000 \
    --jinja --chat-template-kwargs '{"thinking": true}' --no-warmup



It's works at the beginning. But then I send a request and it crashed.






INFO [                    main] HTTP server listening | tid="128549793122560" timestamp=1759130387 n_threads_http="127" port="8000" hostname="0.0.0.0"
INFO [            update_slots] all slots are idle | tid="128549793122560" timestamp=1759130387
INFO [      log_server_request] request | tid="128139002730176" timestamp=1759130495 remote_addr="192.168.5.7" remote_port=44768 status=200 method="GET" path="/v1/models" params={}
Grammar:
Grammar lazy: false
Chat format: DeepSeek V3.1
INFO [   launch_slot_with_task] slot is processing task | tid="128549793122560" timestamp=1759130498 id_slot=0 id_task=0
INFO [            update_slots] kv cache rm [p0, end) | tid="128549793122560" timestamp=1759130498 id_slot=0 id_task=0 p0=0
======================================= HAVE_FANCY_SIMD is NOT defined
Oops(ggml_compute_forward_sum_rows_f32, ffn_moe_weights_sum-3): found -nan for i1 = 0, i2 = 0, i3 = 0. ne00 = 8

---

👤 **ikawrakow** commented on **2025-09-29** at **07:23:34**

I'll try to see what goes wrong in the warm up run and push a fix. Don't see the issue at this point.

---

👤 **oovloveme** commented on **2025-09-29** at **07:26:41**

> I'll try to see what goes wrong in the warm up run and push a fix. Don't see the issue at this point.

It's works at the beginning. But then I send a request and it crashed.

---

👤 **ikawrakow** commented on **2025-09-29** at **07:28:43**

> It's works at the beginning. But then I send a request and it crashed.

I think if I fix the warmup it will also work after that.

---

👤 **ikawrakow** commented on **2025-09-29** at **10:22:04**

Does [#807](https://github.com/ikawrakow/ik_llama.cpp/issues/807) fix the issue?

---

👤 **oovloveme** commented on **2025-09-30** at **01:48:20**

> Does [[#807](https://github.com/ikawrakow/ik_llama.cpp/issues/807)](https://github.com/ikawrakow/ik_llama.cpp/pull/807) fix the issue?

llama_new_context_with_model: KV self size  = 1430.94 MiB, c^KV (q8_0): 1430.94 MiB, kv^T: not used
llama_new_context_with_model:        CPU  output buffer size =     0.99 MiB
llama_new_context_with_model:        CPU compute buffer size =  2131.98 MiB
llama_new_context_with_model: graph nodes  = 13546
llama_new_context_with_model: graph splits = 1
======================================= HAVE_FANCY_SIMD is NOT defined
INFO [                    init] initializing slots | tid="133469181147392" timestamp=1759196719 n_slots=1
INFO [                    init] new slot | tid="133469181147392" timestamp=1759196719 id_slot=0 n_ctx_slot=40192
INFO [                    main] model loaded | tid="133469181147392" timestamp=1759196719
INFO [                    main] chat template | tid="133469181147392" timestamp=1759196719 chat_template="{% if not add_generation_prompt is defined %}{% set add_generation_prompt = false %}{% endif %}{% if not thinking is defined %}{% set thinking = false %}{% endif %}{% set ns = namespace(is_first=false, is_tool=false, system_prompt='', is_first_sp=true, is_last_user=false) %}{%- for message in messages %}{%- if message['role'] == 'system' %}{%- if ns.is_first_sp %}{% set ns.system_prompt = ns.system_prompt + message['content'] %}{% set ns.is_first_sp = false %}{%- else %}{% set ns.system_prompt = ns.system_prompt + '\n\n' + message['content'] %}{%- endif %}{%- endif %}{%- endfor %}{{ bos_token }}{{ ns.system_prompt }}{%- for message in messages %}{%- if message['role'] == 'user' %}{%- set ns.is_tool = false -%}{%- set ns.is_first = false -%}{%- set ns.is_last_user = true -%}{{'<｜User｜>' + message['content']}}{%- endif %}{%- if message['role'] == 'assistant' and message['tool_calls'] is defined and message['tool_calls'] is not none %}{%- if ns.is_last_user %}{{'<｜Assistant｜></think>'}}{%- endif %}{%- set ns.is_last_user = false -%}{%- set ns.is_first = false %}{%- set ns.is_tool = false -%}{%- for tool in message['tool_calls'] %}{%- if not ns.is_first %}{%- if message['content'] is none %}{{'<｜tool▁calls▁begin｜><｜tool▁call▁begin｜>'+ tool['function']['name'] + '<｜tool▁sep｜>' + tool['function']['arguments'] + '<｜tool▁call▁end｜>'}}{%- else %}{{message['content'] + '<｜tool▁calls▁begin｜><｜tool▁call▁begin｜>' + tool['function']['name'] + '<｜tool▁sep｜>' + tool['function']['arguments'] + '<｜tool▁call▁end｜>'}}{%- endif %}{%- set ns.is_first = true -%}{%- else %}{{'<｜tool▁call▁begin｜>'+ tool['function']['name'] + '<｜tool▁sep｜>' + tool['function']['arguments'] + '<｜tool▁call▁end｜>'}}{%- endif %}{%- endfor %}{{'<｜tool▁calls▁end｜><｜end▁of▁sentence｜>'}}{%- endif %}{%- if message['role'] == 'assistant' and (message['tool_calls'] is not defined or message['tool_calls'] is none) %}{%- if ns.is_last_user %}{{'<｜Assistant｜>'}}{%- if message['prefix'] is defined and message['prefix'] and thinking %}{{'<think>'}}  {%- else %}{{'</think>'}}{%- endif %}{%- endif %}{%- set ns.is_last_user = false -%}{%- if ns.is_tool %}{{message['content'] + '<｜end▁of▁sentence｜>'}}{%- set ns.is_tool = false -%}{%- else %}{%- set content = message['content'] -%}{%- if '</think>' in content %}{%- set content = content.split('</think>', 1)[1] -%}{%- endif %}{{content + '<｜end▁of▁sentence｜>'}}{%- endif %}{%- endif %}{%- if message['role'] == 'tool' %}{%- set ns.is_last_user = false -%}{%- set ns.is_tool = true -%}{{'<｜tool▁output▁begin｜>' + message['content'] + '<｜tool▁output▁end｜>'}}{%- endif %}{%- endfor -%}{%- if add_generation_prompt and ns.is_last_user and not ns.is_tool %}{{'<｜Assistant｜>'}}{%- if not thinking %}{{'</think>'}}{%- else %}{{'<think>'}}{%- endif %}{% endif %}"
INFO [                    main] chat template | tid="133469181147392" timestamp=1759196719 chat_example="You are a helpful assistant<｜User｜>Hello<｜Assistant｜></think>Hi there<｜end▁of▁sentence｜><｜User｜>How are you?<｜Assistant｜><think>" built_in=true
INFO [                    main] HTTP server listening | tid="133469181147392" timestamp=1759196719 n_threads_http="127" port="8000" hostname="0.0.0.0"
INFO [            update_slots] all slots are idle | tid="133469181147392" timestamp=1759196719
INFO [      log_server_request] request | tid="133057694410432" timestamp=1759196745 remote_addr="192.168.5.7" remote_port=43112 status=200 method="GET" path="/v1/models" params={}
INFO [      log_server_request] request | tid="133057719588544" timestamp=1759196750 remote_addr="192.168.5.7" remote_port=43148 status=200 method="GET" path="/v1/models" params={}
INFO [      log_server_request] request | tid="133057727981248" timestamp=1759196753 remote_addr="192.168.5.7" remote_port=43180 status=200 method="GET" path="/v1/models" params={}
INFO [      log_server_request] request | tid="133058246366912" timestamp=1759196756 remote_addr="192.168.5.7" remote_port=43202 status=200 method="GET" path="/v1/models" params={}
INFO [      log_server_request] request | tid="133058237974208" timestamp=1759196759 remote_addr="192.168.5.7" remote_port=43240 status=200 method="GET" path="/v1/models" params={}
INFO [      log_server_request] request | tid="133058229581504" timestamp=1759196765 remote_addr="192.168.5.7" remote_port=43290 status=200 method="GET" path="/v1/models" params={}
Grammar:
Grammar lazy: false
Chat format: DeepSeek V3.1
INFO [   launch_slot_with_task] slot is processing task | tid="133469181147392" timestamp=1759196768 id_slot=0 id_task=0
INFO [            update_slots] kv cache rm [p0, end) | tid="133469181147392" timestamp=1759196768 id_slot=0 id_task=0 p0=0
INFO [           print_timings] prompt eval time     =     754.84 ms /    10 tokens (   75.48 ms per token,    13.25 tokens per second) | tid="133469181147392" timestamp=1759196830 id_slot=0 id_task=0 t_prompt_processing=754.835 n_prompt_tokens_processed=10 t_token=75.4835 n_tokens_second=13.247928355203456
INFO [           print_timings] generation eval time =   61781.59 ms /   279 runs   (  221.44 ms per token,     4.52 tokens per second) | tid="133469181147392" timestamp=1759196830 id_slot=0 id_task=0 t_token_generation=61781.588 n_decoded=279 t_token=221.43938351254482 n_tokens_second=4.515908526015873
INFO [           print_timings]           total time =   62536.42 ms | tid="133469181147392" timestamp=1759196830 id_slot=0 id_task=0 t_prompt_processing=754.835 t_token_generation=61781.588 t_total=62536.423
INFO [            update_slots] slot released | tid="133469181147392" timestamp=1759196830 id_slot=0 id_task=0 n_ctx=40192 n_past=288 n_system_tokens=0 n_cache_tokens=288 truncated=false
INFO [            update_slots] all slots are idle | tid="133469181147392" timestamp=1759196830
INFO [      log_server_request] request | tid="133058221188800" timestamp=1759196830 remote_addr="192.168.5.7" remote_port=43338 status=200 method="POST" path="/v1/chat/completions" params={}
INFO [            update_slots] all slots are idle | tid="133469181147392" timestamp=1759196830



It's solved now.