## 📌 [Issue #873](https://github.com/ikawrakow/ik_llama.cpp/issues/873) - Bug: Ling-1T works with shorter context, but fails to generate useful output at higher context length

| **Author** | `Lissanro` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-10-27 |
| **Updated** | 2025-10-28 |

---

## 📄 Description

### What happened?

I recently downloaded https://huggingface.co/ubergarm/Ling-1T-GGUF/tree/main/smol-IQ4-KSS and when testing it with shorter context length, it works fine. But when I gave it a task that resulted in 78K context, it fails completely to generate any useful output, sometimes it can generate one token like "I" and then fail, sometimes not even that.

I tested in Roo Code with temperature 0.6. I tried with and without "-ctk q8_0 -ctv q8_0" (for f16 context), with and without -ger, but the outcome is the same, even though with short context it looks promising, the model does not really work unfortunately. Please let me know if I missed some important option to try. My full command is shown at the beginning of the log.

Here, it works normally with the initial request:

```
INFO [            update_slots] kv cache rm [p0, end) | tid="140455660474368" timestamp=1761602482 id_slot=0 id_task=0 p0=8192
INFO [           print_timings] prompt eval time     =   96267.58 ms / 11358 tokens (    8.48 ms per token,   117.98 tokens per second) | tid="140455660474368" timestamp=1761602568 id_slot=0 id_task=0 t_prompt_processing=96267.577 n_prompt_tokens_processed=11358 t_token=8.475750748371192 n_tokens_second=117.98364884575831
INFO [           print_timings] generation eval time =   54357.39 ms /   264 runs   (  205.90 ms per token,     4.86 tokens per second) | tid="140455660474368" timestamp=1761602568 id_slot=0 id_task=0 t_token_generation=54357.389 n_decoded=264 t_token=205.89920075757576 n_tokens_second=4.8567454187323085
```

But after it reads few files and builds up context, it fails entirely:

```
INFO [            update_slots] kv cache rm [p0, end) | tid="140455660474368" timestamp=1761603165 id_slot=0 id_task=268 p0=73061
WARN [           process_token] n_predict is not set and self-context extend is disabled. Limiting generated tokens to n_ctx_train to avoid EOS-less generation infinite loop | tid="140455660474368" timestamp=1761603206 id_slot=0 params.n_predict=-1 slot.n_prompt_tokens=76173 slot.n_decoded=1 slot.n_predict=-1 n_slots=1 slot.n_ctx=131072 n_ctx=131072 n_ctx_train=32768 ga_n=1
INFO [           print_timings] prompt eval time     =  636923.73 ms / 64552 tokens (    9.87 ms per token,   101.35 tokens per second) | tid="140455660474368" timestamp=1761603206 id_slot=0 id_task=268 t_prompt_processing=636923.734 n_prompt_tokens_processed=64552 t_token=9.866831918453341 n_tokens_second=101.34965389749472
INFO [           print_timings] generation eval time =       0.11 ms /     1 runs   (    0.11 ms per token,  9009.01 tokens per second) | tid="140455660474368" timestamp=1761603206 id_slot=0 id_task=268 t_token_generation=0.111 n_decoded=1 t_token=0.111 n_tokens_second=9009.009009009009
```

I suspect this message may contain a clue why it fails: `n_predict is not set and self-context extend is disabled. Limiting generated tokens to n_ctx_train to avoid EOS-less generation infinite loop` - but I do not know why it happens given this is 128K context model according to the official model card: https://huggingface.co/inclusionAI/Ling-1T so this is why I think there is might be a bug, maybe it does not read correctly meta information or something else is wrong?

I tried to run it with simpler task, and it failed even sooner with similar error (around 41K context length, much smaller than 128K that the model is supposed to support):

```
...
INFO [            update_slots] kv cache rm [p0, end) | tid="140455660474368" timestamp=1761610867 id_slot=0 id_task=583 p0=41519
WARN [           process_token] n_predict is not set and self-context extend is disabled. Limiting generated tokens to n_ctx_train to avoid EOS-less generation infinite loop | tid="140455660474368" timestamp=1761610898 id_slot=0 params.n_predict=-1 slot.n_prompt_tokens=43513 slot.n_decoded=1 slot.n_predict=-1 n_slots=1 slot.n_ctx=131072 n_ctx=131072 n_ctx_train=32768 ga_n=1
...

### Name and Version

latest git

### What operating system are you seeing the problem on?

_No response_

### Relevant log output

```shell
> ~/pkgs/ik_llama.cpp/build/bin/llama-server \
--model /mnt/neuro/models/Ling-1T/Ling-1T-smol-IQ4_KSS-00001-of-00011.gguf \
--ctx-size 131072 --n-gpu-layers 62 --tensor-split 12,26,31,31 -b 4096 -ub 4096 -ger \
-ot "blk\.(4)\.ffn_.*=CUDA0" \
-ot "blk\.(5)\.ffn_.*=CUDA1" \
-ot "blk\.(6)\.ffn_.*=CUDA2" \
-ot "blk\.(7)\.ffn_.*=CUDA3" \
-ot exps=CPU \
--threads 64 --host 0.0.0.0 --port 5000 \
--slot-save-path /var/cache/ik_llama.cpp/ling \
--jinja
ggml_cuda_init: GGML_CUDA_FORCE_MMQ:    no
ggml_cuda_init: GGML_CUDA_FORCE_CUBLAS: no
ggml_cuda_init: found 4 CUDA devices:
  Device 0: NVIDIA GeForce RTX 3090, compute capability 8.6, VMM: yes
  Device 1: NVIDIA GeForce RTX 3090, compute capability 8.6, VMM: yes
  Device 2: NVIDIA GeForce RTX 3090, compute capability 8.6, VMM: yes
  Device 3: NVIDIA GeForce RTX 3090, compute capability 8.6, VMM: yes
INFO [                    main] build info | tid="140455660474368" timestamp=1761602325 build=3929 commit="f76e9853"
INFO [                    main] system info | tid="140455660474368" timestamp=1761602325 n_threads=64 n_threads_batch=-1 total_threads=128 system_info="AVX = 1 | AVX_VNNI = 0 | AVX2 = 1 | AVX512 = 0 | AVX512_VBMI = 0 | AVX512_VNNI = 0 | AVX512_BF16 = 0 | FMA = 1 | NEON = 0 | SVE = 0 | ARM_FMA = 0 | F16C = 1 | FP16_VA = 0 | WASM_SIMD = 0 | BLAS = 1 | SSE3 = 1 | SSSE3 = 1 | VSX = 0 | MATMUL_INT8 = 0 | LLAMAFILE = 1 | "
llama_model_loader: additional 10 GGUFs metadata loaded.
llama_model_loader: loaded meta data with 50 key-value pairs and 1103 tensors from /mnt/neuro/models/Ling-1T/Ling-1T-smol-IQ4_KSS-00001-of-00011.gguf (version GGUF V3 (latest))
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
llama_model_loader: - kv  16:                          general.file_type u32              = 148
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
llama_model_loader: - kv  43:                      quantize.imatrix.file str              = /mnt/data/models/ubergarm/Ling-1T-GGU...
llama_model_loader: - kv  44:                   quantize.imatrix.dataset str              = ubergarm-imatrix-calibration-corpus-v...
llama_model_loader: - kv  45:             quantize.imatrix.entries_count i32              = 705
llama_model_loader: - kv  46:              quantize.imatrix.chunks_count i32              = 864
llama_model_loader: - kv  47:                                   split.no u16              = 0
llama_model_loader: - kv  48:                                split.count u16              = 11
llama_model_loader: - kv  49:                        split.tensors.count i32              = 1103
llama_model_loader: - type  f32:  473 tensors
llama_model_loader: - type iq4_k:    1 tensors
llama_model_loader: - type iq6_k:  161 tensors
llama_model_loader: - type iq4_kss:  228 tensors
llama_model_loader: - type iq5_ks:  240 tensors
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
llm_load_print_meta: model ftype      = IQ4_KSS - 4.0 bpw
llm_load_print_meta: model params     = 999.705 B
llm_load_print_meta: model size       = 471.923 GiB (4.055 BPW) 
llm_load_print_meta: repeating layers = 470.255 GiB (4.051 BPW, 997.130 B parameters)
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
llm_load_tensors: ggml ctx size =    2.36 MiB
Tensor blk.4.ffn_norm.weight buffer type overriden to CUDA0
Tensor blk.4.ffn_gate_inp.weight buffer type overriden to CUDA0
Tensor blk.4.ffn_gate_exps.weight buffer type overriden to CUDA0
Tensor blk.4.ffn_down_exps.weight buffer type overriden to CUDA0
Tensor blk.4.ffn_up_exps.weight buffer type overriden to CUDA0
Tensor blk.4.ffn_gate_shexp.weight buffer type overriden to CUDA0
Tensor blk.4.ffn_down_shexp.weight buffer type overriden to CUDA0
Tensor blk.4.ffn_up_shexp.weight buffer type overriden to CUDA0
Tensor blk.5.ffn_norm.weight buffer type overriden to CUDA1
Tensor blk.5.ffn_gate_inp.weight buffer type overriden to CUDA1
Tensor blk.5.ffn_gate_exps.weight buffer type overriden to CUDA1
Tensor blk.5.ffn_down_exps.weight buffer type overriden to CUDA1
Tensor blk.5.ffn_up_exps.weight buffer type overriden to CUDA1
Tensor blk.5.ffn_gate_shexp.weight buffer type overriden to CUDA1
Tensor blk.5.ffn_down_shexp.weight buffer type overriden to CUDA1
Tensor blk.5.ffn_up_shexp.weight buffer type overriden to CUDA1
Tensor blk.6.ffn_norm.weight buffer type overriden to CUDA2
Tensor blk.6.ffn_gate_inp.weight buffer type overriden to CUDA2
Tensor blk.6.ffn_gate_exps.weight buffer type overriden to CUDA2
Tensor blk.6.ffn_down_exps.weight buffer type overriden to CUDA2
Tensor blk.6.ffn_up_exps.weight buffer type overriden to CUDA2
Tensor blk.6.ffn_gate_shexp.weight buffer type overriden to CUDA2
Tensor blk.6.ffn_down_shexp.weight buffer type overriden to CUDA2
Tensor blk.6.ffn_up_shexp.weight buffer type overriden to CUDA2
Tensor blk.7.ffn_norm.weight buffer type overriden to CUDA3
Tensor blk.7.ffn_gate_inp.weight buffer type overriden to CUDA3
Tensor blk.7.ffn_gate_exps.weight buffer type overriden to CUDA3
Tensor blk.7.ffn_down_exps.weight buffer type overriden to CUDA3
Tensor blk.7.ffn_up_exps.weight buffer type overriden to CUDA3
Tensor blk.7.ffn_gate_shexp.weight buffer type overriden to CUDA3
Tensor blk.7.ffn_down_shexp.weight buffer type overriden to CUDA3
Tensor blk.7.ffn_up_shexp.weight buffer type overriden to CUDA3
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
llm_load_tensors: offloading 62 repeating layers to GPU
llm_load_tensors: offloaded 62/81 layers to GPU
llm_load_tensors:        CPU buffer size = 17170.28 MiB
llm_load_tensors:        CPU buffer size = 43907.32 MiB
llm_load_tensors:        CPU buffer size = 44553.77 MiB
llm_load_tensors:        CPU buffer size = 44013.15 MiB
llm_load_tensors:        CPU buffer size = 43907.32 MiB
llm_load_tensors:        CPU buffer size = 44638.58 MiB
llm_load_tensors:        CPU buffer size = 43907.32 MiB
llm_load_tensors:        CPU buffer size = 44553.77 MiB
llm_load_tensors:        CPU buffer size = 44013.15 MiB
llm_load_tensors:        CPU buffer size = 43907.32 MiB
llm_load_tensors:        CPU buffer size = 39672.03 MiB
llm_load_tensors:        CPU buffer size = 40258.11 MiB
llm_load_tensors:        CPU buffer size = 31595.32 MiB
llm_load_tensors:        CPU buffer size =  6545.32 MiB
llm_load_tensors:        CPU buffer size =  1811.28 MiB
llm_load_tensors:      CUDA0 buffer size =  7466.47 MiB
llm_load_tensors:      CUDA1 buffer size =  8737.36 MiB
llm_load_tensors:      CUDA2 buffer size =  9213.94 MiB
llm_load_tensors:      CUDA3 buffer size =  9213.94 MiB
....................................................................................................
llama_new_context_with_model: n_ctx         = 131072
llama_new_context_with_model: n_batch       = 4096
llama_new_context_with_model: n_ubatch      = 4096
llama_new_context_with_model: flash_attn    = 1
llama_new_context_with_model: mla_attn      = 0
llama_new_context_with_model: attn_max_b    = 0
llama_new_context_with_model: fused_moe     = 1
llama_new_context_with_model: grouped er    = 1
llama_new_context_with_model: fused_up_gate = 1
llama_new_context_with_model: fused_mmad    = 1
llama_new_context_with_model: ser           = -1, 0
llama_new_context_with_model: freq_base     = 600000.0
llama_new_context_with_model: freq_scale    = 1
llama_kv_cache_init:  CUDA_Host KV buffer size =  9216.00 MiB
llama_kv_cache_init:      CUDA0 KV buffer size =  4096.00 MiB
llama_kv_cache_init:      CUDA1 KV buffer size =  8192.00 MiB
llama_kv_cache_init:      CUDA2 KV buffer size =  9728.00 MiB
llama_kv_cache_init:      CUDA3 KV buffer size =  9728.00 MiB
llama_new_context_with_model: KV self size  = 40960.00 MiB, K (f16): 20480.00 MiB, V (f16): 20480.00 MiB
llama_new_context_with_model:  CUDA_Host  output buffer size =     1.20 MiB
llama_new_context_with_model:      CUDA0 compute buffer size =  5892.02 MiB
llama_new_context_with_model:      CUDA1 compute buffer size =  1568.02 MiB
llama_new_context_with_model:      CUDA2 compute buffer size =  1568.02 MiB
llama_new_context_with_model:      CUDA3 compute buffer size =  1568.03 MiB
llama_new_context_with_model:  CUDA_Host compute buffer size =  1359.98 MiB
llama_new_context_with_model: graph nodes  = 3217
llama_new_context_with_model: graph splits = 384
XXXXXXXXXXXXXXXXXXXXX Setting only active experts offload
======================================= HAVE_FANCY_SIMD is NOT defined
INFO [                    init] initializing slots | tid="140455660474368" timestamp=1761602387 n_slots=1
INFO [                    init] new slot | tid="140455660474368" timestamp=1761602387 id_slot=0 n_ctx_slot=131072
INFO [                    main] model loaded | tid="140455660474368" timestamp=1761602387
INFO [                    main] chat template | tid="140455660474368" timestamp=1761602387 chat_template="{% set thinking_option = 'off' %}\n{{- '<role>SYSTEM</role>' }}\n{%- if messages[0].role == 'system' %}\n    {{- messages[0].content + '\\n' }}\n{%- endif %}\n{%- if tools %}\n    {{- \"# Tools\\n\\nYou may call one or more functions to assist with the user query.\\n\\nYou are provided with function signatures within <tools></tools> XML tags:\\n<tools>\" }}\n    {%- for tool in tools %}\n        {{- \"\\n\" }}\n        {{- tool | tojson }}\n    {%- endfor %}\n    {{- \"\\n</tools>\\n\\nFor each function call, return a json object with function name and arguments within <tool_call></tool_call> XML tags:\\n<tool_call>\\n{\\\"name\\\": <function-name>, \\\"arguments\\\": <args-json-object>}\\n</tool_call>\\n\" }}\n{%- endif %}\n{{- 'detailed thinking ' + thinking_option + '<|role_end|>' }}\n{%- set ns = namespace(multi_step_tool=true, last_query_index=messages|length - 1) %}\n{%- for message in messages[::-1] %}\n    {%- set index = (messages|length - 1) - loop.index0 %}\n    {%- if ns.multi_step_tool and message.role == \"user\" and message.content is string and not(message.content.startswith('<tool_response>') and message.content.endswith('</tool_response>')) %}\n        {%- set ns.multi_step_tool = false %}\n        {%- set ns.last_query_index = index %}\n    {%- endif %}\n{%- endfor %}\n{%- for message in messages %}\n    {%- if message.content is string %}\n        {%- set content = message.content %}\n    {%- else %}\n        {%- set content = '' %}\n    {%- endif %}\n    {%- if message.role == \"user\" %}\n        {{- '<role>HUMAN</role>' + message.content + '<|role_end|>' }}\n    {%- elif message.role == \"system\" and not loop.first %}\n        {{- '<role>SYSTEM</role>' + message.content + '<|role_end|>' }}\n    {%- elif message.role == \"assistant\" %}\n        {%- set reasoning_content = '' %}\n        {%- if message.reasoning_content is string %}\n            {%- set reasoning_content = message.reasoning_content %}\n        {%- else %}\n            {%- if '</think>' in content %}\n                {%- set reasoning_content = content.split('</think>')[0].rstrip('\\n').split('<think>')[-1].lstrip('\\n') %}\n                {%- set content = content.split('</think>')[-1].lstrip('\\n') %}\n            {%- endif %}\n        {%- endif %}\n        {%- if loop.index0 > ns.last_query_index %}\n            {%- if reasoning_content %}\n                {{- '<role>ASSISTANT</role>' + '\\n<think>\\n' + reasoning_content.strip('\\n') + '\\n</think>\\n\\n' + content.lstrip('\\n') }}\n            {%- else %}\n                {{- '<role>ASSISTANT</role>' + content }}\n            {%- endif %}\n        {%- else %}\n            {{- '<role>ASSISTANT</role>' + content }}\n        {%- endif %}\n        {%- if message.tool_calls %}\n            {%- for tool_call in message.tool_calls %}\n                {%- if (loop.first and content) or (not loop.first) %}\n                    {{- '\\n' }}\n                {%- endif %}\n                {%- if tool_call.function %}\n                    {%- set tool_call = tool_call.function %}\n                {%- endif %}\n                {{- '<tool_call>\\n{\"name\": \"' }}\n                {{- tool_call.name }}\n                {{- '\", \"arguments\": ' }}\n                {%- if tool_call.arguments is string %}\n                    {{- tool_call.arguments }}\n                {%- else %}\n                    {{- tool_call.arguments | tojson }}\n                {%- endif %}\n                {{- '}\\n</tool_call>' }}\n            {%- endfor %}\n        {%- endif %}\n        {{- '<|role_end|>' }}\n    {%- elif message.role == \"tool\" %}\n        {%- if loop.first or (messages[loop.index0 - 1].role != \"tool\") %}\n            {{- '<role>OBSERVATION</role>' }}\n        {%- endif %}\n        {{- '\\n<tool_response>\\n' }}\n        {{- content }}\n        {{- '\\n</tool_response>' }}\n        {%- if loop.last or (messages[loop.index0 + 1].role != \"tool\") %}\n            {{- '<|role_end|>' }}\n        {%- endif %}\n    {%- endif %}\n{%- endfor %}\n{%- if add_generation_prompt %}\n    {{- '<role>ASSISTANT</role>' }}\n{%- endif %}"
INFO [                    main] chat template | tid="140455660474368" timestamp=1761602387 chat_example="<role>SYSTEM</role>You are a helpful assistant\ndetailed thinking off<|role_end|><role>HUMAN</role>Hello<|role_end|><role>ASSISTANT</role>Hi there<|role_end|><role>HUMAN</role>How are you?<|role_end|><role>ASSISTANT</role>" built_in=true
INFO [                    main] HTTP server listening | tid="140455660474368" timestamp=1761602387 n_threads_http="127" port="5000" hostname="0.0.0.0"
INFO [            update_slots] all slots are idle | tid="140455660474368" timestamp=1761602387
Grammar: 
Grammar lazy: false
Chat format: Hermes 2 Pro
INFO [   launch_slot_with_task] slot is processing task | tid="140455660474368" timestamp=1761602417 id_slot=0 id_task=0
INFO [            update_slots] kv cache rm [p0, end) | tid="140455660474368" timestamp=1761602417 id_slot=0 id_task=0 p0=0
INFO [            update_slots] kv cache rm [p0, end) | tid="140455660474368" timestamp=1761602449 id_slot=0 id_task=0 p0=4096
INFO [            update_slots] kv cache rm [p0, end) | tid="140455660474368" timestamp=1761602482 id_slot=0 id_task=0 p0=8192
INFO [           print_timings] prompt eval time     =   96267.58 ms / 11358 tokens (    8.48 ms per token,   117.98 tokens per second) | tid="140455660474368" timestamp=1761602568 id_slot=0 id_task=0 t_prompt_processing=96267.577 n_prompt_tokens_processed=11358 t_token=8.475750748371192 n_tokens_second=117.98364884575831
INFO [           print_timings] generation eval time =   54357.39 ms /   264 runs   (  205.90 ms per token,     4.86 tokens per second) | tid="140455660474368" timestamp=1761602568 id_slot=0 id_task=0 t_token_generation=54357.389 n_decoded=264 t_token=205.89920075757576 n_tokens_second=4.8567454187323085
INFO [           print_timings]           total time =  150624.97 ms | tid="140455660474368" timestamp=1761602568 id_slot=0 id_task=0 t_prompt_processing=96267.577 t_token_generation=54357.389 t_total=150624.96600000001
INFO [            update_slots] slot released | tid="140455660474368" timestamp=1761602568 id_slot=0 id_task=0 n_ctx=131072 n_past=11621 n_system_tokens=0 n_cache_tokens=11621 truncated=false
INFO [            update_slots] all slots are idle | tid="140455660474368" timestamp=1761602568
INFO [      log_server_request] request | tid="139940159545344" timestamp=1761602568 remote_addr="127.0.0.1" remote_port=38908 status=200 method="POST" path="/v1/chat/completions" params={}
INFO [            update_slots] all slots are idle | tid="140455660474368" timestamp=1761602568
Grammar: 
Grammar lazy: false
Chat format: Hermes 2 Pro
INFO [   launch_slot_with_task] slot is processing task | tid="140455660474368" timestamp=1761602569 id_slot=0 id_task=268
INFO [            update_slots] kv cache rm [p0, end) | tid="140455660474368" timestamp=1761602569 id_slot=0 id_task=268 p0=11621
INFO [            update_slots] kv cache rm [p0, end) | tid="140455660474368" timestamp=1761602603 id_slot=0 id_task=268 p0=15717
INFO [            update_slots] kv cache rm [p0, end) | tid="140455660474368" timestamp=1761602638 id_slot=0 id_task=268 p0=19813
INFO [            update_slots] kv cache rm [p0, end) | tid="140455660474368" timestamp=1761602674 id_slot=0 id_task=268 p0=23909
INFO [            update_slots] kv cache rm [p0, end) | tid="140455660474368" timestamp=1761602711 id_slot=0 id_task=268 p0=28005
INFO [            update_slots] kv cache rm [p0, end) | tid="140455660474368" timestamp=1761602748 id_slot=0 id_task=268 p0=32101
INFO [            update_slots] kv cache rm [p0, end) | tid="140455660474368" timestamp=1761602786 id_slot=0 id_task=268 p0=36197
INFO [            update_slots] kv cache rm [p0, end) | tid="140455660474368" timestamp=1761602825 id_slot=0 id_task=268 p0=40293
INFO [            update_slots] kv cache rm [p0, end) | tid="140455660474368" timestamp=1761602865 id_slot=0 id_task=268 p0=44389
INFO [            update_slots] kv cache rm [p0, end) | tid="140455660474368" timestamp=1761602905 id_slot=0 id_task=268 p0=48485
INFO [            update_slots] kv cache rm [p0, end) | tid="140455660474368" timestamp=1761602947 id_slot=0 id_task=268 p0=52581
INFO [            update_slots] kv cache rm [p0, end) | tid="140455660474368" timestamp=1761602988 id_slot=0 id_task=268 p0=56677
INFO [            update_slots] kv cache rm [p0, end) | tid="140455660474368" timestamp=1761603031 id_slot=0 id_task=268 p0=60773
INFO [            update_slots] kv cache rm [p0, end) | tid="140455660474368" timestamp=1761603075 id_slot=0 id_task=268 p0=64869
INFO [            update_slots] kv cache rm [p0, end) | tid="140455660474368" timestamp=1761603119 id_slot=0 id_task=268 p0=68965
INFO [            update_slots] kv cache rm [p0, end) | tid="140455660474368" timestamp=1761603165 id_slot=0 id_task=268 p0=73061
WARN [           process_token] n_predict is not set and self-context extend is disabled. Limiting generated tokens to n_ctx_train to avoid EOS-less generation infinite loop | tid="140455660474368" timestamp=1761603206 id_slot=0 params.n_predict=-1 slot.n_prompt_tokens=76173 slot.n_decoded=1 slot.n_predict=-1 n_slots=1 slot.n_ctx=131072 n_ctx=131072 n_ctx_train=32768 ga_n=1
INFO [           print_timings] prompt eval time     =  636923.73 ms / 64552 tokens (    9.87 ms per token,   101.35 tokens per second) | tid="140455660474368" timestamp=1761603206 id_slot=0 id_task=268 t_prompt_processing=636923.734 n_prompt_tokens_processed=64552 t_token=9.866831918453341 n_tokens_second=101.34965389749472
INFO [           print_timings] generation eval time =       0.11 ms /     1 runs   (    0.11 ms per token,  9009.01 tokens per second) | tid="140455660474368" timestamp=1761603206 id_slot=0 id_task=268 t_token_generation=0.111 n_decoded=1 t_token=0.111 n_tokens_second=9009.009009009009
INFO [           print_timings]           total time =  636923.85 ms | tid="140455660474368" timestamp=1761603206 id_slot=0 id_task=268 t_prompt_processing=636923.734 t_token_generation=0.111 t_total=636923.8450000001
INFO [      log_server_request] request | tid="139940151152640" timestamp=1761603206 remote_addr="127.0.0.1" remote_port=46178 status=200 method="POST" path="/v1/chat/completions" params={}
INFO [            update_slots] slot released | tid="140455660474368" timestamp=1761603206 id_slot=0 id_task=268 n_ctx=131072 n_past=76173 n_system_tokens=0 n_cache_tokens=76173 truncated=true
INFO [            update_slots] all slots are idle | tid="140455660474368" timestamp=1761603206
Grammar: 
Grammar lazy: false
Chat format: Hermes 2 Pro
INFO [   launch_slot_with_task] slot is processing task | tid="140455660474368" timestamp=1761603206 id_slot=0 id_task=278
INFO [            update_slots] we have to evaluate at least 1 token to generate logits | tid="140455660474368" timestamp=1761603206 id_slot=0 id_task=278
INFO [            update_slots] kv cache rm [p0, end) | tid="140455660474368" timestamp=1761603206 id_slot=0 id_task=278 p0=76172
WARN [           process_token] n_predict is not set and self-context extend is disabled. Limiting generated tokens to n_ctx_train to avoid EOS-less generation infinite loop | tid="140455660474368" timestamp=1761603206 id_slot=0 params.n_predict=-1 slot.n_prompt_tokens=76173 slot.n_decoded=1 slot.n_predict=-1 n_slots=1 slot.n_ctx=131072 n_ctx=131072 n_ctx_train=32768 ga_n=1
INFO [           print_timings] prompt eval time     =     460.27 ms /     1 tokens (  460.27 ms per token,     2.17 tokens per second) | tid="140455660474368" timestamp=1761603206 id_slot=0 id_task=278 t_prompt_processing=460.269 n_prompt_tokens_processed=1 t_token=460.269 n_tokens_second=2.1726425199177
INFO [           print_timings] generation eval time =       0.10 ms /     1 runs   (    0.10 ms per token,  9803.92 tokens per second) | tid="140455660474368" timestamp=1761603206 id_slot=0 id_task=278 t_token_generation=0.102 n_decoded=1 t_token=0.102 n_tokens_second=9803.921568627451
INFO [           print_timings]           total time =     460.37 ms | tid="140455660474368" timestamp=1761603206 id_slot=0 id_task=278 t_prompt_processing=460.269 t_token_generation=0.102 t_total=460.371
INFO [      log_server_request] request | tid="139940142759936" timestamp=1761603206 remote_addr="127.0.0.1" remote_port=33524 status=200 method="POST" path="/v1/chat/completions" params={}
INFO [            update_slots] slot released | tid="140455660474368" timestamp=1761603206 id_slot=0 id_task=278 n_ctx=131072 n_past=76173 n_system_tokens=0 n_cache_tokens=76173 truncated=true
INFO [            update_slots] all slots are idle | tid="140455660474368" timestamp=1761603206
Grammar: 
Grammar lazy: false
Chat format: Hermes 2 Pro
INFO [   launch_slot_with_task] slot is processing task | tid="140455660474368" timestamp=1761603206 id_slot=0 id_task=286
INFO [            update_slots] we have to evaluate at least 1 token to generate logits | tid="140455660474368" timestamp=1761603207 id_slot=0 id_task=286
INFO [            update_slots] kv cache rm [p0, end) | tid="140455660474368" timestamp=1761603207 id_slot=0 id_task=286 p0=76172
WARN [           process_token] n_predict is not set and self-context extend is disabled. Limiting generated tokens to n_ctx_train to avoid EOS-less generation infinite loop | tid="140455660474368" timestamp=1761603207 id_slot=0 params.n_predict=-1 slot.n_prompt_tokens=76173 slot.n_decoded=1 slot.n_predict=-1 n_slots=1 slot.n_ctx=131072 n_ctx=131072 n_ctx_train=32768 ga_n=1
INFO [           print_timings] prompt eval time     =     443.03 ms /     1 tokens (  443.03 ms per token,     2.26 tokens per second) | tid="140455660474368" timestamp=1761603207 id_slot=0 id_task=286 t_prompt_processing=443.029 n_prompt_tokens_processed=1 t_token=443.029 n_tokens_second=2.257188581334405
INFO [           print_timings] generation eval time =       0.10 ms /     1 runs   (    0.10 ms per token, 10101.01 tokens per second) | tid="140455660474368" timestamp=1761603207 id_slot=0 id_task=286 t_token_generation=0.099 n_decoded=1 t_token=0.099 n_tokens_second=10101.0101010101
INFO [           print_timings]           total time =     443.13 ms | tid="140455660474368" timestamp=1761603207 id_slot=0 id_task=286 t_prompt_processing=443.029 t_token_generation=0.099 t_total=443.128
INFO [            update_slots] slot released | tid="140455660474368" timestamp=1761603207 id_slot=0 id_task=286 n_ctx=131072 n_past=76173 n_system_tokens=0 n_cache_tokens=76173 truncated=true
INFO [            update_slots] all slots are idle | tid="140455660474368" timestamp=1761603207
INFO [      log_server_request] request | tid="139940134367232" timestamp=1761603207 remote_addr="127.0.0.1" remote_port=60640 status=200 method="POST" path="/v1/chat/completions" params={}
INFO [            update_slots] all slots are idle | tid="140455660474368" timestamp=1761603207
Grammar: 
Grammar lazy: false
Chat format: Hermes 2 Pro
INFO [   launch_slot_with_task] slot is processing task | tid="140455660474368" timestamp=1761603208 id_slot=0 id_task=292
INFO [            update_slots] kv cache rm [p0, end) | tid="140455660474368" timestamp=1761603208 id_slot=0 id_task=292 p0=76173
WARN [           process_token] n_predict is not set and self-context extend is disabled. Limiting generated tokens to n_ctx_train to avoid EOS-less generation infinite loop | tid="140455660474368" timestamp=1761603226 id_slot=0 params.n_predict=-1 slot.n_prompt_tokens=76825 slot.n_decoded=1 slot.n_predict=-1 n_slots=1 slot.n_ctx=131072 n_ctx=131072 n_ctx_train=32768 ga_n=1
INFO [           print_timings] prompt eval time     =   18455.11 ms /   652 tokens (   28.31 ms per token,    35.33 tokens per second) | tid="140455660474368" timestamp=1761603226 id_slot=0 id_task=292 t_prompt_processing=18455.107 n_prompt_tokens_processed=652 t_token=28.305378834355828 n_tokens_second=35.328974250867255
INFO [           print_timings] generation eval time =       0.12 ms /     1 runs   (    0.12 ms per token,  8403.36 tokens per second) | tid="140455660474368" timestamp=1761603226 id_slot=0 id_task=292 t_token_generation=0.119 n_decoded=1 t_token=0.119 n_tokens_second=8403.361344537816
INFO [           print_timings]           total time =   18455.23 ms | tid="140455660474368" timestamp=1761603226 id_slot=0 id_task=292 t_prompt_processing=18455.107 t_token_generation=0.119 t_total=18455.226
INFO [            update_slots] slot released | tid="140455660474368" timestamp=1761603226 id_slot=0 id_task=292 n_ctx=131072 n_past=76825 n_system_tokens=0 n_cache_tokens=76825 truncated=true
INFO [            update_slots] all slots are idle | tid="140455660474368" timestamp=1761603226
INFO [      log_server_request] request | tid="139940125974528" timestamp=1761603226 remote_addr="127.0.0.1" remote_port=60644 status=200 method="POST" path="/v1/chat/completions" params={}
INFO [            update_slots] all slots are idle | tid="140455660474368" timestamp=1761603226
Grammar: 
Grammar lazy: false
Chat format: Hermes 2 Pro
INFO [   launch_slot_with_task] slot is processing task | tid="140455660474368" timestamp=1761603228 id_slot=0 id_task=295
INFO [            update_slots] kv cache rm [p0, end) | tid="140455660474368" timestamp=1761603228 id_slot=0 id_task=295 p0=76825
WARN [           process_token] n_predict is not set and self-context extend is disabled. Limiting generated tokens to n_ctx_train to avoid EOS-less generation infinite loop | tid="140455660474368" timestamp=1761603246 id_slot=0 params.n_predict=-1 slot.n_prompt_tokens=77477 slot.n_decoded=1 slot.n_predict=-1 n_slots=1 slot.n_ctx=131072 n_ctx=131072 n_ctx_train=32768 ga_n=1
INFO [           print_timings] prompt eval time     =   18473.72 ms /   652 tokens (   28.33 ms per token,    35.29 tokens per second) | tid="140455660474368" timestamp=1761603246 id_slot=0 id_task=295 t_prompt_processing=18473.722 n_prompt_tokens_processed=652 t_token=28.333929447852764 n_tokens_second=35.293375097882276
INFO [           print_timings] generation eval time =       0.14 ms /     1 runs   (    0.14 ms per token,  6993.01 tokens per second) | tid="140455660474368" timestamp=1761603246 id_slot=0 id_task=295 t_token_generation=0.143 n_decoded=1 t_token=0.143 n_tokens_second=6993.006993006994
INFO [           print_timings]           total time =   18473.87 ms | tid="140455660474368" timestamp=1761603246 id_slot=0 id_task=295 t_prompt_processing=18473.722 t_token_generation=0.143 t_total=18473.865
INFO [            update_slots] slot released | tid="140455660474368" timestamp=1761603246 id_slot=0 id_task=295 n_ctx=131072 n_past=77477 n_system_tokens=0 n_cache_tokens=77477 truncated=true
INFO [            update_slots] all slots are idle | tid="140455660474368" timestamp=1761603246
INFO [      log_server_request] request | tid="139940117581824" timestamp=1761603246 remote_addr="127.0.0.1" remote_port=52886 status=200 method="POST" path="/v1/chat/completions" params={}
INFO [            update_slots] all slots are idle | tid="140455660474368" timestamp=1761603246
```

---

## 💬 Conversation

👤 **Lissanro** commented on **2025-10-28** at **01:14:24**

Upon closer investigation, I noticed in the middle of the official model card https://huggingface.co/inclusionAI/Ling-1T a note "32K -> 128K (YaRN)". But https://huggingface.co/ubergarm/Ling-1T-GGUF the example command provided by @ubergarm uses `--ctx-size 65536`, which is 64K context, implying that YaRN supposed to work for this model.

If this is not a bug and the issue caused by the quant lacking YaRN meta information (despite the example command implies it should be there), then my apologies for the noise, and please feel free to close if this is the case.

But if ik_llama.cpp fails to utilize YaRN, then this still may be a bug. I am not sure how to check if the GGUF has right YaRN parameters though, or if I can force them manually in order to test this.

---

👤 **ikawrakow** commented on **2025-10-28** at **04:59:23**

`ik_llama.cpp` works fine with YaRN. My guess is that something is missing in the meta parameters and this is why it is not working beyond the original training context of 32k (this was reported somewhere else as well, but I'm not finding the discussion atm).

---

👤 **CISC** commented on **2025-10-28** at **11:33:55**

Much like `Qwen`, `inclusionAI` do not include `YaRN` settings in their default config, you have to add it before conversion, or provide it on the command line: `--rope-scaling yarn --rope-scale 4 --yarn-orig-ctx 32768` (there are more YaRN settings, but they are fine left at default).

Edit: Forgot the RoPE Scaling Factor.

---

👤 **ikawrakow** commented on **2025-10-28** at **12:04:49**

@CISC  Thank you for this! It didn't even occur to me that the YaRN parameters may not be included in the model file, considering that they advertise a context of 128k while having trained with a context of 32k!

@magikRUKKOLA IIRC, you were having trouble running Ling-1T with a context > 32k, so this is relevant for you.

---

👤 **Lissanro** commented on **2025-10-28** at **15:21:25**

@CISC Thanks, but unfortunately it did not work:

```
> CUDA_VISIBLE_DEVICES="0,1,2,3" numactl --cpunodebind=0 --interleave=all /home/lissanro/pkgs/ik_llama.cpp/build/bin/llama-server \
--jinja --rope-scaling yarn --rope-scale 4 --yarn-orig-ctx 32768 \
--model /mnt/neuro/models/Ling-1T/Ling-1T-smol-IQ4_KSS-00001-of-00011.gguf \
--ctx-size 131072 --n-gpu-layers 62 --tensor-split 12,26,31,31 -ctk q8_0 -ctv q8_0 -b 4096 -ub 4096 -ger \
-ot "blk\.(4|5)\.ffn_.*=CUDA0" \
-ot "blk\.(7|8)\.ffn_.*=CUDA1" \
-ot "blk\.(9|10)\.ffn_.*=CUDA2" \
-ot "blk\.(11|12)\.ffn_.*=CUDA3" \
-ot exps=CPU \
--threads 64 --host 0.0.0.0 --port 5000 \
--slot-save-path /var/cache/ik_llama.cpp/ling
...
llm_load_print_meta: rope type        = 2
llm_load_print_meta: rope scaling     = none
llm_load_print_meta: freq_base_train  = 600000.0
llm_load_print_meta: freq_scale_train = 1
llm_load_print_meta: n_ctx_orig_yarn  = 32768
llm_load_print_meta: rope_finetuned   = unknown
...
INFO [            update_slots] kv cache rm [p0, end) | tid="138784445362176" timestamp=1761664760 id_slot=0 id_task=688 p0=69385
WARN [           process_token] n_predict is not set and self-context extend is disabled. Limiting generated tokens to n_ctx_train to avoid EOS-less generation infinite loop | tid="138784445362176" timestamp=1761664790 id_slot=0 params.n_predict=-1 slot.n_prompt_tokens=70645 slot.n_decoded=1 slot.n_predict=-1 n_slots=1 slot.n_ctx=131072 n_ctx=131072 n_ctx_train=32768 ga_n=1
```

Currently it seems to ignore `--rope-scaling yarn --rope-scale 4 --yarn-orig-ctx 32768` as far as I can tell. Maybe some other command-line options are needed to force it to use YaRN?

---

👤 **CISC** commented on **2025-10-28** at **17:33:20**

@Lissanro It works (at least on mainline, just tested it), it just doesn't tell you as the scaling type is injected straight into context params and doesn't show in the printed metadata.

---

👤 **Lissanro** commented on **2025-10-28** at **18:26:33**

@CISC I tested with ik_llama.cpp. It fails to generate useful output beyond 32K with exactly the same error `Limiting generated tokens to n_ctx_train to avoid EOS-less generation infinite loop`. As far as I can tell these options do not have any effect at all.

Since you mention they work on the mainline llama.cpp, then I guess it means ik_llama.cpp has YaRN-related bug after all (but if there is something wrong with my command that I shared in the previous message, please let me know).

---

👤 **magikRUKKOLA** commented on **2025-10-28** at **18:39:26**

@ikawrakow 

Unfortunately I am not able to run Ling-T1 with my DDR4 system yet beyond 32k ctx because I have to use the spec. decoding to get more or less acceptable speed.  So 32k ctx itself with f16 KV cache both for Ling-flash-2.0 (and the model itself with IQ1_KT quantisation) and Ling-T1 takes 64 GB of VRAM by itself.  I strictly need to add a fourth GPU for the KV cache beyond 32k ctx.  For that I need liquid cooling setup.  I ordered two 60mm thick radiators for that.  Still waiting for them though.

---

👤 **CISC** commented on **2025-10-28** at **18:54:07**

@Lissanro Oh, you're right, this happens with `llama-server` (I was testing with `llama-cli`), looks like it just cuts off because you're past the training context.

---

👤 **CISC** commented on **2025-10-28** at **19:12:44**

To fix this without editing metadata you have to use the following option: `--override-kv bailingmoe2.context_length=int:131072`.

---

👤 **Lissanro** commented on **2025-10-28** at **23:53:15**

@CISC Thanks, it worked! Here is full command for reference that worked for me:

```
numactl --cpunodebind=0 --interleave=all ~/pkgs/ik_llama.cpp/build/bin/llama-server \
--jinja --rope-scaling yarn --rope-scale 4 --yarn-orig-ctx 32768 --override-kv bailingmoe2.context_length=int:131072 \
--model /mnt/neuro/models/Ling-1T/Ling-1T-smol-IQ4_KSS-00001-of-00011.gguf \
--ctx-size 131072 --n-gpu-layers 62 --tensor-split 12,26,31,31 -ctk q8_0 -ctv q8_0 -b 4096 -ub 4096 -ger \
-ot "blk\.(4|5)\.ffn_.*=CUDA0" \
-ot "blk\.(7|8)\.ffn_.*=CUDA1" \
-ot "blk\.(9|10)\.ffn_.*=CUDA2" \
-ot "blk\.(11|12)\.ffn_.*=CUDA3" \
-ot exps=CPU \
--threads 64 --host 0.0.0.0 --port 5000 \
--slot-save-path /var/cache/ik_llama.cpp/ling
```

Since this turned out to be an issue with the quant metainformation and not a bug in ik_llama.cpp, I think it is safe to close this issue. Thanks again to everyone for helping figuring this out! I finally can test the Ling-1T model.