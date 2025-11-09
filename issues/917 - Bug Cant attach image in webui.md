## 📌 [Issue #917](https://github.com/ikawrakow/ik_llama.cpp/issues/917) - Bug: Can't attach image in webui

| **Author** | `Skelectric` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-11-07 |
| **Updated** | 2025-11-07 |

---

## 📄 Description

### What happened?

I'm running unsloth/Qwen3-VL-235B-A22B-Thinking-GGUF using

`./build/bin/llama-server \
  --model ~/models/Qwen3-VL-235B-A22B-Thinking/UD-Q4_K_XL/Qwen3-VL-235B-A22B-Thinking-UD-Q4_K_XL-00001-of-00003.gguf \
  --host 0.0.0.0 \
  --port 8080 \
  --jinja \
  -mla 2 \
  -amb 512 \
  -ctk q8_0 \
  -ngl 99 \
  -ot exps=CPU \
  --ctx-size 182000 \
  --threads 64 \
  --webui llamacpp`

The option to attach an image is grayed out, as is the setting to Parse PDF as image.

<img width="1023" height="630" alt="Image" src="https://github.com/user-attachments/assets/c55daacd-55ba-4d29-8b5e-c814781cc7a9" />

<img width="1125" height="784" alt="Image" src="https://github.com/user-attachments/assets/96b0e45d-f3de-4390-bbc9-1afb91d2572c" />


### Name and Version

$ ./build/bin/llama-server --version
version: 3956 (320fc606)
built with cc (Ubuntu 13.3.0-6ubuntu2~24.04) 13.3.0 for x86_64-linux-gnu

### What operating system are you seeing the problem on?

Linux

### Relevant log output

```shell
$ ./build/bin/llama-server   --model ~/models/Qwen3-VL-235B-A22B-Thinking/UD-Q4_K_XL/Qwen3-VL-235B-A22B-Thinking-UD-Q4_K_XL-00001-of-00003.gguf   --host 0.0.0.0   --port 8080   --jinja   -mla 2   -amb 512   -ctk q8_0   -ngl 99   -ot exps=CPU   --ctx-size 182000   --threads 64   --webui llamacpp
ggml_cuda_init: GGML_CUDA_FORCE_MMQ:    no
ggml_cuda_init: GGML_CUDA_FORCE_CUBLAS: no
ggml_cuda_init: found 1 CUDA devices:
  Device 0: NVIDIA GeForce RTX 5090, compute capability 12.0, VMM: yes
INFO [                    main] build info | tid="123793531027456" timestamp=1762534922 build=3956 commit="320fc606"
INFO [                    main] system info | tid="123793531027456" timestamp=1762534922 n_threads=64 n_threads_batch=-1 total_threads=128 system_info="AVX = 1 | AVX_VNNI = 1 | AVX2 = 1 | AVX512 = 1 | AVX512_VBMI = 1 | AVX512_VNNI = 1 | AVX512_BF16 = 1 | FMA = 1 | NEON = 0 | SVE = 0 | ARM_FMA = 0 | F16C = 1 | FP16_VA = 0 | WASM_SIMD = 0 | BLAS = 1 | SSE3 = 1 | SSSE3 = 1 | VSX = 0 | MATMUL_INT8 = 0 | LLAMAFILE = 1 | "
CUDA0: using device CUDA0 - 31602 MiB free
llama_model_loader: additional 2 GGUFs metadata loaded.
llama_model_loader: loaded meta data with 48 key-value pairs and 1131 tensors from /home/philkir/models/Qwen3-VL-235B-A22B-Thinking/UD-Q4_K_XL/Qwen3-VL-235B-A22B-Thinking-UD-Q4_K_XL-00001-of-00003.gguf (version GGUF V3 (latest))
llama_model_loader: Dumping metadata keys/values. Note: KV overrides do not apply in this output.
llama_model_loader: - kv   0:                       general.architecture str              = qwen3vlmoe
llama_model_loader: - kv   1:                               general.type str              = model
llama_model_loader: - kv   2:                               general.name str              = Qwen3-Vl-235B-A22B-Thinking
llama_model_loader: - kv   3:                           general.finetune str              = Thinking
llama_model_loader: - kv   4:                           general.basename str              = Qwen3-Vl-235B-A22B-Thinking
llama_model_loader: - kv   5:                       general.quantized_by str              = Unsloth
llama_model_loader: - kv   6:                         general.size_label str              = 235B-A22B
llama_model_loader: - kv   7:                            general.license str              = apache-2.0
llama_model_loader: - kv   8:                           general.repo_url str              = https://huggingface.co/unsloth
llama_model_loader: - kv   9:                   general.base_model.count u32              = 1
llama_model_loader: - kv  10:                  general.base_model.0.name str              = Qwen3 VL 235B A22B Thinking
llama_model_loader: - kv  11:          general.base_model.0.organization str              = Qwen
llama_model_loader: - kv  12:              general.base_model.0.repo_url str              = https://huggingface.co/Qwen/Qwen3-VL-...
llama_model_loader: - kv  13:                               general.tags arr[str,1]       = ["unsloth"]
llama_model_loader: - kv  14:                     qwen3vlmoe.block_count u32              = 94
llama_model_loader: - kv  15:                  qwen3vlmoe.context_length u32              = 262144
llama_model_loader: - kv  16:                qwen3vlmoe.embedding_length u32              = 4096
llama_model_loader: - kv  17:             qwen3vlmoe.feed_forward_length u32              = 12288
llama_model_loader: - kv  18:            qwen3vlmoe.attention.head_count u32              = 64
llama_model_loader: - kv  19:         qwen3vlmoe.attention.head_count_kv u32              = 4
llama_model_loader: - kv  20:                  qwen3vlmoe.rope.freq_base f32              = 5000000.000000
llama_model_loader: - kv  21: qwen3vlmoe.attention.layer_norm_rms_epsilon f32              = 0.000001
llama_model_loader: - kv  22:               qwen3vlmoe.expert_used_count u32              = 8
llama_model_loader: - kv  23:            qwen3vlmoe.attention.key_length u32              = 128
llama_model_loader: - kv  24:          qwen3vlmoe.attention.value_length u32              = 128
llama_model_loader: - kv  25:                    qwen3vlmoe.expert_count u32              = 128
llama_model_loader: - kv  26:      qwen3vlmoe.expert_feed_forward_length u32              = 1536
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
llama_model_loader: - kv  38:                    tokenizer.chat_template str              = {# Unsloth template fixes #}\n{%- set ...
llama_model_loader: - kv  39:               general.quantization_version u32              = 2
llama_model_loader: - kv  40:                          general.file_type u32              = 15
llama_model_loader: - kv  41:                      quantize.imatrix.file str              = Qwen3-VL-235B-A22B-Thinking-GGUF/imat...
llama_model_loader: - kv  42:                   quantize.imatrix.dataset str              = unsloth_calibration_Qwen3-VL-235B-A22...
llama_model_loader: - kv  43:             quantize.imatrix.entries_count u32              = 752
llama_model_loader: - kv  44:              quantize.imatrix.chunks_count u32              = 79
llama_model_loader: - kv  45:                                   split.no u16              = 0
llama_model_loader: - kv  46:                        split.tensors.count i32              = 1131
llama_model_loader: - kv  47:                                split.count u16              = 3
llama_model_loader: - type  f32:  471 tensors
llama_model_loader: - type q4_K:  611 tensors
llama_model_loader: - type q5_K:   38 tensors
llama_model_loader: - type q6_K:   11 tensors
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
llm_load_print_meta: n_embd           = 16384
llm_load_print_meta: n_layer          = 94
llm_load_print_meta: n_head           = 64
llm_load_print_meta: n_head_kv        = 4
llm_load_print_meta: n_rot            = 128
llm_load_print_meta: n_swa            = 0
llm_load_print_meta: n_swa_pattern    = 1
llm_load_print_meta: n_embd_head_k    = 128
llm_load_print_meta: n_embd_head_v    = 128
llm_load_print_meta: n_gqa            = 16
llm_load_print_meta: n_embd_k_gqa     = 512
llm_load_print_meta: n_embd_v_gqa     = 512
llm_load_print_meta: f_norm_eps       = 0.0e+00
llm_load_print_meta: f_norm_rms_eps   = 1.0e-06
llm_load_print_meta: f_clamp_kqv      = 0.0e+00
llm_load_print_meta: f_max_alibi_bias = 0.0e+00
llm_load_print_meta: f_logit_scale    = 0.0e+00
llm_load_print_meta: n_ff             = 12288
llm_load_print_meta: n_expert         = 128
llm_load_print_meta: n_expert_used    = 8
llm_load_print_meta: causal attn      = 1
llm_load_print_meta: pooling type     = 0
llm_load_print_meta: rope type        = 40
llm_load_print_meta: rope scaling     = linear
llm_load_print_meta: freq_base_train  = 5000000.0
llm_load_print_meta: freq_scale_train = 1
llm_load_print_meta: n_ctx_orig_yarn  = 262144
llm_load_print_meta: rope_finetuned   = unknown
llm_load_print_meta: mrope sections   = [24, 20, 20, 0]
llm_load_print_meta: ssm_d_conv       = 0
llm_load_print_meta: ssm_d_inner      = 0
llm_load_print_meta: ssm_d_state      = 0
llm_load_print_meta: ssm_dt_rank      = 0
llm_load_print_meta: model type       = 235B.A22B
llm_load_print_meta: model ftype      = Q4_K - Medium
llm_load_print_meta: model params     = 235.094 B
llm_load_print_meta: model size       = 125.001 GiB (4.567 BPW) 
llm_load_print_meta: repeating layers = 124.199 GiB (4.562 BPW, 233.849 B parameters)
llm_load_print_meta: general.name     = Qwen3-Vl-235B-A22B-Thinking
llm_load_print_meta: n_ff_exp         = 1536
print_info: vocab type       = BPE
print_info: n_vocab          = 151936
print_info: n_merges         = 151387
print_info: BOS token        = 151643 '<|endoftext|>'
print_info: EOS token        = 151645 '<|im_end|>'
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
llm_load_tensors: ggml ctx size =    0.99 MiB
Tensor blk.0.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.0.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.0.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.1.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.1.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.1.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.2.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.2.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.2.ffn_up_exps.weight buffer type overriden to CPU
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
Tensor blk.92.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.92.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.92.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.93.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.93.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.93.ffn_up_exps.weight buffer type overriden to CPU
llm_load_tensors: offloading 94 repeating layers to GPU
llm_load_tensors: offloading non-repeating layers to GPU
llm_load_tensors: offloaded 95/95 layers to GPU
llm_load_tensors:        CPU buffer size = 46698.60 MiB
llm_load_tensors:        CPU buffer size = 47339.13 MiB
llm_load_tensors:        CPU buffer size = 33055.51 MiB
llm_load_tensors:        CPU buffer size =   333.84 MiB
llm_load_tensors:      CUDA0 buffer size =  4402.90 MiB
....................................................................................................
=====================================================================
 MLA is only available for LLM_ARCH_DEEPSEEK2 -> turning off MLA
=====================================================================
llama_new_context_with_model: n_ctx         = 182016
llama_new_context_with_model: n_batch       = 2048
llama_new_context_with_model: n_ubatch      = 512
llama_new_context_with_model: flash_attn    = 1
llama_new_context_with_model: mla_attn      = 0
llama_new_context_with_model: attn_max_b    = 512
llama_new_context_with_model: fused_moe     = 1
llama_new_context_with_model: grouped er    = 0
llama_new_context_with_model: fused_up_gate = 1
llama_new_context_with_model: fused_mmad    = 1
llama_new_context_with_model: rope_cache    = 0
llama_new_context_with_model: ser           = -1, 0
llama_new_context_with_model: freq_base     = 5000000.0
llama_new_context_with_model: freq_scale    = 1
llama_kv_cache_init:      CUDA0 KV buffer size = 25584.94 MiB
llama_new_context_with_model: KV self size  = 25584.89 MiB, K (q8_0): 8876.39 MiB, V (f16): 16708.50 MiB
llama_new_context_with_model:  CUDA_Host  output buffer size =     0.58 MiB
llama_new_context_with_model:      CUDA0 compute buffer size =  1097.76 MiB
llama_new_context_with_model:  CUDA_Host compute buffer size =   185.76 MiB
llama_new_context_with_model: graph nodes  = 3483
llama_new_context_with_model: graph splits = 190
XXXXXXXXXXXXXXXXXXXXX Setting only active experts offload
INFO [                    init] initializing slots | tid="123793531027456" timestamp=1762534926 n_slots=1
INFO [                    init] new slot | tid="123793531027456" timestamp=1762534926 id_slot=0 n_ctx_slot=182016
INFO [                    main] model loaded | tid="123793531027456" timestamp=1762534926
INFO [                    main] chat template | tid="123793531027456" timestamp=1762534926 chat_template="{# Unsloth template fixes #}\n{%- set image_count = namespace(value=0) %}\n{%- set video_count = namespace(value=0) %}\n{%- macro render_content(content, do_vision_count) %}\n    {%- if content is string %}\n        {{- content }}\n    {%- else %}\n        {%- for item in content %}\n            {%- if 'image' in item or 'image_url' in item or item.type == 'image' %}\n                {%- if do_vision_count %}\n                    {%- set image_count.value = image_count.value + 1 %}\n                {%- endif %}\n                {%- if add_vision_id %}Picture {{ image_count.value }}: {% endif -%}\n                <|vision_start|><|image_pad|><|vision_end|>\n            {%- elif 'video' in item or item.type == 'video' %}\n                {%- if do_vision_count %}\n                    {%- set video_count.value = video_count.value + 1 %}\n                {%- endif %}\n                {%- if add_vision_id %}Video {{ video_count.value }}: {% endif -%}\n                <|vision_start|><|video_pad|><|vision_end|>\n            {%- elif 'text' in item %}\n                {{- item.text }}\n            {%- endif %}\n        {%- endfor %}\n    {%- endif %}\n{%- endmacro %}\n{%- if tools %}\n    {{- '<|im_start|>system\\n' }}\n    {%- if messages[0].role == 'system' %}\n        {{- render_content(messages[0].content, false) + '\\n\\n' }}\n    {%- endif %}\n    {{- \"# Tools\\n\\nYou may call one or more functions to assist with the user query.\\n\\nYou are provided with function signatures within <tools></tools> XML tags:\\n<tools>\" }}\n    {%- for tool in tools %}\n        {{- \"\\n\" }}\n        {{- tool | tojson }}\n    {%- endfor %}\n    {{- \"\\n</tools>\\n\\nFor each function call, return a json object with function name and arguments within <tool_call></tool_call> XML tags:\\n<tool_call>\\n{\\\"name\\\": <function-name>, \\\"arguments\\\": <args-json-object>}\\n</tool_call><|im_end|>\\n\" }}\n{%- else %}\n    {%- if messages[0].role == 'system' %}\n        {{- '<|im_start|>system\\n' + render_content(messages[0].content, false) + '<|im_end|>\\n' }}\n    {%- endif %}\n{%- endif %}\n{%- set ns = namespace(multi_step_tool=true, last_query_index=messages|length - 1) %}\n{%- for message in messages[::-1] %}\n    {%- set index = (messages|length - 1) - loop.index0 %}\n    {%- if ns.multi_step_tool and message.role == \"user\" %}\n        {%- set content = render_content(message.content, false) %}\n        {%- if not(content.startswith('<tool_response>') and content.endswith('</tool_response>')) %}\n            {%- set ns.multi_step_tool = false %}\n            {%- set ns.last_query_index = index %}\n        {%- endif %}\n    {%- endif %}\n{%- endfor %}\n{%- for message in messages %}\n    {%- set content = render_content(message.content, True) %}\n    {%- if (message.role == \"user\") or (message.role == \"system\" and not loop.first) %}\n        {{- '<|im_start|>' + message.role + '\\n' + content + '<|im_end|>' + '\\n' }}\n    {%- elif message.role == \"assistant\" %}\n        {%- set reasoning_content = '' %}\n        {%- if message.reasoning_content is string %}\n            {%- set reasoning_content = message.reasoning_content %}\n        {%- else %}\n            {%- if '</think>' in content %}\n                {# Unsloth template fixes - must change to for loop since llama.cpp will error out if not #}\n                {%- set parts = content.split('</think>') %}\n                {%- for part in parts %}\n                    {%- if loop.index0 == 0 -%}\n                        {%- set reasoning_content = part.rstrip('\\n') %}\n                        {%- set reasoning_content = (reasoning_content.split('<think>')|last) %}\n                        {%- set reasoning_content = reasoning_content.lstrip('\\n') -%}\n                    {%- else -%}\n                        {%- set content = part.lstrip('\\n') %}\n                    {%- endif %}\n                {%- endfor %}\n            {%- endif %}\n        {%- endif %}\n        {%- if loop.index0 > ns.last_query_index %}\n            {%- if loop.last or (not loop.last and reasoning_content) %}\n                {{- '<|im_start|>' + message.role + '\\n<think>\\n' + reasoning_content.strip('\\n') + '\\n</think>\\n\\n' + content.lstrip('\\n') }}\n            {%- else %}\n                {{- '<|im_start|>' + message.role + '\\n' + content }}\n            {%- endif %}\n        {%- else %}\n            {{- '<|im_start|>' + message.role + '\\n' + content }}\n        {%- endif %}\n        {%- if message.tool_calls %}\n            {%- for tool_call in message.tool_calls %}\n                {%- if (loop.first and content) or (not loop.first) %}\n                    {{- '\\n' }}\n                {%- endif %}\n                {%- if tool_call.function %}\n                    {%- set tool_call = tool_call.function %}\n                {%- endif %}\n                {{- '<tool_call>\\n{\"name\": \"' }}\n                {{- tool_call.name }}\n                {{- '\", \"arguments\": ' }}\n                {%- if tool_call.arguments is string %}\n                    {{- tool_call.arguments }}\n                {%- else %}\n                    {{- tool_call.arguments | tojson }}\n                {%- endif %}\n                {{- '}\\n</tool_call>' }}\n            {%- endfor %}\n        {%- endif %}\n        {{- '<|im_end|>\\n' }}\n    {%- elif message.role == \"tool\" %}\n        {%- if loop.first or (messages[loop.index0 - 1].role != \"tool\") %}\n            {{- '<|im_start|>user' }}\n        {%- endif %}\n        {{- '\\n<tool_response>\\n' }}\n        {{- content }}\n        {{- '\\n</tool_response>' }}\n        {%- if loop.last or (messages[loop.index0 + 1].role != \"tool\") %}\n            {{- '<|im_end|>\\n' }}\n        {%- endif %}\n    {%- endif %}\n{%- endfor %}\n{%- if add_generation_prompt %}\n    {{- '<|im_start|>assistant\\n<think>\\n' }}\n{%- endif %}\n{# Copyright 2025-present Unsloth. Apache 2.0 License. #}"
INFO [                    main] chat template | tid="123793531027456" timestamp=1762534926 chat_example="<|im_start|>system\nYou are a helpful assistant<|im_end|>\n<|im_start|>user\nHello<|im_end|>\n<|im_start|>assistant\nHi there<|im_end|>\n<|im_start|>user\nHow are you?<|im_end|>\n<|im_start|>assistant\n<think>\n" built_in=true
INFO [                    main] HTTP server listening | tid="123793531027456" timestamp=1762534926 n_threads_http="127" port="8080" hostname="0.0.0.0"
INFO [            update_slots] all slots are idle | tid="123793531027456" timestamp=1762534926
```

---

## 💬 Conversation

👤 **ikawrakow** commented on **2025-11-07** at **17:06:49**

You need an `mmproj` file to enable the vision capabilities, no?

---

👤 **Skelectric** commented on **2025-11-07** at **18:59:48**

Yep that was it, thanks