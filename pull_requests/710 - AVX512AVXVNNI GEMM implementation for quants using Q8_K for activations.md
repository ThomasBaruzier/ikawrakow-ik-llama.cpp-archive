## 🔀 [Pull Request #710](https://github.com/ikawrakow/ik_llama.cpp/pull/710) - AVX512+AVXVNNI GEMM implementation for quants using Q8_K for activations 

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/q8_k_r16` |
| **Target Branch** | `main` |
| **Created** | 2025-08-20 |
| **Updated** | 2025-08-22 |
| **Merged** | 2025-08-22 |

---

## 📄 Description

This is an alternative to [#610](https://github.com/ikawrakow/ik_llama.cpp/issues/610) 

The expectation is that it will significantly improve CPU-only prompt processing speed on "true" `AVX512` CPUs that support the  `AVX512VNNI, AVX512VL, AVX512BW,` and `AVX512DQ` extensions (e.g., Zen5). These extensions are also supported by Zen4 cores, but these are not "true" `AVX512` CPUs as there 512-bit instructions are performed as two 256-bit instructions. The main benefit compared to [#610](https://github.com/ikawrakow/ik_llama.cpp/issues/610) is that on such CPUs performance is about the same as on main, unlike [#610](https://github.com/ikawrakow/ik_llama.cpp/issues/610), where we get a 10-15% performance penalty.IQ2_XXS

The PR adds a new quantization type, `Q8_K_R16`, which is only used for quantizing activation tensors. `IQ1_S, IQ1_M, IQ2_XXS, IQ2_XS, IQ2_S, IQ3_XXS, IQ3_S, IQ4_XS, Q2_K, Q3_K, IQ2_KS, IQ2_K, IQ3_KS, IQ3_K, IQ4_KSS, IQ4_KS, IQ4_K, IQ5_KS, IQ5_K, IQ6_K` all use this new quantization type for GEMM when the batch size is `>= 32` and "fancy SIMD" are available (i.e., the above `AVX512` extensions are supported by the CPU).

If you have a CPU that supports `AVX512VNNI, AVX512VL, AVX512BW,` and `AVX512DQ` extensions (Ryzen 99XX, EPYCs of the 9005 series (a.k.a., Turin), recent Intel Xeon CPUs), please test and let me know if this PR improves CPU performance.

@ubergarm  Pinging you explicitly.

---

## 💬 Conversation

👤 **sousekd** commented on **2025-08-20** at **20:24:06**

I tried to test with `ubergarm/Qwen3-235B-A22B-Instruct-2507-GGUF` **IQ5_K** (downloaded 4 weeks ago) on EPYC 9355, WS 2025 VM in QEMU.

With GPU in the mix, I saw insignificant degradation of performance on the PR710 compared to the main branch (PP 1091.02 vs 1094.84 t/s, TG 18.65 vs 18.79 t/s). When running without GPU (compiled with `-DGGML_CUDA=OFF`), the main branch worked (PP 85.46 t/s, TG 15.97 t/s), but this PR failed to run:

```
llama-sweep-bench.exe
  --alias ubergarm/Qwen3-235B-A22B-Instruct-2507-IQ5_K
  --model Z:\hfcache\ubergarm\Qwen3-235B-A22B-Instruct-2507-GGUF\Qwen3-235B-A22B-Instruct-IQ5_K-00001-of-00004.gguf
  --no-mmap -fa -fmoe -c 32768
  -amb 512 -b 8192 -ub 8192 --parallel 1
  --threads 16 --threads-batch 24
  --warmup-batch
```

**Error: `ik_llama.cpp\ggml\src\iqk\iqk_gemm_legacy_quants.cpp:1774: GGML_ASSERT(nrc_x%8 == 0) failed`.**

Compiled using:
```
cmake -B build -G Ninja `
  -DCMAKE_BUILD_TYPE=Release `
  -DCMAKE_C_COMPILER="clang-cl" `
  -DCMAKE_CXX_COMPILER="clang-cl" `
  -DGGML_CUDA=OFF `
  -DGGML_AVX512=ON `
  -DGGML_AVX512_VNNI=ON `
  -DGGML_AVX512_VBMI=ON `
  -DGGML_AVX512_BF16=ON `
  -DGGML_SCHED_MAX_COPIES=1 `
  -DGGML_BLAS=OFF `
  -DGGML_CCACHE=OFF `
  -DCMAKE_C_FLAGS='-mavx512f -mavx512bw -mavx512dq -mavx512vl -mavx512vnni -mavx512vbmi -mavx512bf16 -march=znver5' `
  -DCMAKE_CXX_FLAGS='/EHsc -mavx512f -mavx512bw -mavx512dq -mavx512vl -mavx512vnni -mavx512vbmi -mavx512bf16 -march=znver5' `
  -DCMAKE_INTERPROCEDURAL_OPTIMIZATION=ON `
  -DLLAMA_CURL=OFF `
  -DBUILD_SHARED_LIBS=OFF
  ```
  
<details> 
  <summary>Full log</summary>

   ```
C:\Users\Administrator\Source\repos\ik_llama.cpp\build-p710-cpu\bin>llama-sweep-bench.exe --alias ubergarm/Qwen3-235B-A22B-Instruct-2507-IQ5_K --model Z:\hfcache\ubergarm\Qwen3-235B-A22B-Instruct-2507-GGUF\Qwen3-235B-A22B-Instruct-IQ5_K-00001-of-00004.gguf --no-mmap -fa -fmoe -c 32768 -amb 512 -b 8192 -ub 8192 --parallel 1 --threads 16 --threads-batch 24 --warmup-batch
llama_model_loader: additional 3 GGUFs metadata loaded.
llama_model_loader: loaded meta data with 42 key-value pairs and 1131 tensors from Z:\hfcache\ubergarm\Qwen3-235B-A22B-Instruct-2507-GGUF\Qwen3-235B-A22B-Instruct-IQ5_K-00001-of-00004.gguf (version GGUF V3 (latest))
llama_model_loader: Dumping metadata keys/values. Note: KV overrides do not apply in this output.
llama_model_loader: - kv   0:                       general.architecture str              = qwen3moe
llama_model_loader: - kv   1:                               general.type str              = model
llama_model_loader: - kv   2:                               general.name str              = Qwen3 235B A22B Instruct 2507
llama_model_loader: - kv   3:                            general.version str              = 2507
llama_model_loader: - kv   4:                           general.finetune str              = Instruct
llama_model_loader: - kv   5:                           general.basename str              = Qwen3
llama_model_loader: - kv   6:                         general.size_label str              = 235B-A22B
llama_model_loader: - kv   7:                            general.license str              = apache-2.0
llama_model_loader: - kv   8:                       general.license.link str              = https://huggingface.co/Qwen/Qwen3-235...
llama_model_loader: - kv   9:                               general.tags arr[str,1]       = ["text-generation"]
llama_model_loader: - kv  10:                       qwen3moe.block_count u32              = 94
llama_model_loader: - kv  11:                    qwen3moe.context_length u32              = 262144
llama_model_loader: - kv  12:                  qwen3moe.embedding_length u32              = 4096
llama_model_loader: - kv  13:               qwen3moe.feed_forward_length u32              = 12288
llama_model_loader: - kv  14:              qwen3moe.attention.head_count u32              = 64
llama_model_loader: - kv  15:           qwen3moe.attention.head_count_kv u32              = 4
llama_model_loader: - kv  16:                    qwen3moe.rope.freq_base f32              = 5000000.000000
llama_model_loader: - kv  17:  qwen3moe.attention.layer_norm_rms_epsilon f32              = 0.000001
llama_model_loader: - kv  18:                 qwen3moe.expert_used_count u32              = 8
llama_model_loader: - kv  19:              qwen3moe.attention.key_length u32              = 128
llama_model_loader: - kv  20:            qwen3moe.attention.value_length u32              = 128
llama_model_loader: - kv  21:                          general.file_type u32              = 141
llama_model_loader: - kv  22:                      qwen3moe.expert_count u32              = 128
llama_model_loader: - kv  23:        qwen3moe.expert_feed_forward_length u32              = 1536
llama_model_loader: - kv  24:               general.quantization_version u32              = 2
llama_model_loader: - kv  25:                       tokenizer.ggml.model str              = gpt2
llama_model_loader: - kv  26:                         tokenizer.ggml.pre str              = qwen2
llama_model_loader: - kv  27:                      tokenizer.ggml.tokens arr[str,151936]  = ["!", "\"", "#", "$", "%", "&", "'", ...
llama_model_loader: - kv  28:                  tokenizer.ggml.token_type arr[i32,151936]  = [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, ...
llama_model_loader: - kv  29:                      tokenizer.ggml.merges arr[str,151387]  = ["─á ─á", "─á─á ─á─á", "i n", "─á t",...
llama_model_loader: - kv  30:                tokenizer.ggml.eos_token_id u32              = 151645
llama_model_loader: - kv  31:            tokenizer.ggml.padding_token_id u32              = 151643
llama_model_loader: - kv  32:                tokenizer.ggml.bos_token_id u32              = 151643
llama_model_loader: - kv  33:               tokenizer.ggml.add_bos_token bool             = false
llama_model_loader: - kv  34:                    tokenizer.chat_template str              = {%- if tools %}\n    {{- '<|im_start|>...
llama_model_loader: - kv  35:                      quantize.imatrix.file str              = /mnt/raid/models/ubergarm/Qwen3-235B-...
llama_model_loader: - kv  36:                   quantize.imatrix.dataset str              = ubergarm-imatrix-calibration-corpus-v...
llama_model_loader: - kv  37:             quantize.imatrix.entries_count i32              = 753
llama_model_loader: - kv  38:              quantize.imatrix.chunks_count i32              = 840
llama_model_loader: - kv  39:                                   split.no u16              = 0
llama_model_loader: - kv  40:                                split.count u16              = 4
llama_model_loader: - kv  41:                        split.tensors.count i32              = 1131
llama_model_loader: - type  f32:  471 tensors
llama_model_loader: - type q8_0:  188 tensors
llama_model_loader: - type iq5_k:  188 tensors
llama_model_loader: - type iq6_k:  284 tensors
init_tokenizer: initializing tokenizer for type 2
load: control token: 151661 '<|fim_suffix|>' is not marked as EOG
load: control token: 151649 '<|box_end|>' is not marked as EOG
load: control token: 151647 '<|object_ref_end|>' is not marked as EOG
load: control token: 151654 '<|vision_pad|>' is not marked as EOG
load: control token: 151659 '<|fim_prefix|>' is not marked as EOG
load: control token: 151648 '<|box_start|>' is not marked as EOG
load: control token: 151644 '<|im_start|>' is not marked as EOG
load: control token: 151646 '<|object_ref_start|>' is not marked as EOG
load: control token: 151650 '<|quad_start|>' is not marked as EOG
load: control token: 151651 '<|quad_end|>' is not marked as EOG
load: control token: 151652 '<|vision_start|>' is not marked as EOG
load: control token: 151653 '<|vision_end|>' is not marked as EOG
load: control token: 151655 '<|image_pad|>' is not marked as EOG
load: control token: 151656 '<|video_pad|>' is not marked as EOG
load: control token: 151660 '<|fim_middle|>' is not marked as EOG
load: printing all EOG tokens:
load:   - 151643 ('<|endoftext|>')
load:   - 151645 ('<|im_end|>')
load:   - 151662 ('<|fim_pad|>')
load:   - 151663 ('<|repo_name|>')
load:   - 151664 ('<|file_sep|>')
load: special tokens cache size = 26
load: token to piece cache size = 0.9311 MB
llm_load_print_meta: format           = GGUF V3 (latest)
llm_load_print_meta: arch             = qwen3moe
llm_load_print_meta: n_ctx_train      = 262144
llm_load_print_meta: n_embd           = 4096
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
llm_load_print_meta: rope type        = 2
llm_load_print_meta: rope scaling     = linear
llm_load_print_meta: freq_base_train  = 5000000.0
llm_load_print_meta: freq_scale_train = 1
llm_load_print_meta: n_ctx_orig_yarn  = 262144
llm_load_print_meta: rope_finetuned   = unknown
llm_load_print_meta: ssm_d_conv       = 0
llm_load_print_meta: ssm_d_inner      = 0
llm_load_print_meta: ssm_d_state      = 0
llm_load_print_meta: ssm_dt_rank      = 0
llm_load_print_meta: model type       = ?B
llm_load_print_meta: model ftype      = IQ5_K - 5.5 bpw
llm_load_print_meta: model params     = 235.094 B
llm_load_print_meta: model size       = 161.722 GiB (5.909 BPW)
llm_load_print_meta: repeating layers = 160.762 GiB (5.905 BPW, 233.849 B parameters)
llm_load_print_meta: general.name     = Qwen3 235B A22B Instruct 2507
llm_load_print_meta: n_ff_exp         = 1536
print_info: vocab type       = BPE
print_info: n_vocab          = 151936
print_info: n_merges         = 151387
print_info: BOS token        = 151643 '<|endoftext|>'
print_info: EOS token        = 151645 '<|im_end|>'
print_info: EOT token        = 151645 '<|im_end|>'
print_info: PAD token        = 151643 '<|endoftext|>'
print_info: LF token         = 198 '─è'
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
llm_load_tensors: ggml ctx size =    0.50 MiB
llm_load_tensors:        CPU buffer size = 165603.53 MiB
....................................................................................................
llama_new_context_with_model: n_ctx      = 32768
llama_new_context_with_model: n_batch    = 8192
llama_new_context_with_model: n_ubatch   = 8192
llama_new_context_with_model: flash_attn = 1
llama_new_context_with_model: mla_attn   = 0
llama_new_context_with_model: attn_max_b = 512
llama_new_context_with_model: fused_moe  = 1
llama_new_context_with_model: ser        = -1, 0
llama_new_context_with_model: freq_base  = 5000000.0
llama_new_context_with_model: freq_scale = 1
llama_kv_cache_init:        CPU KV buffer size =  6016.00 MiB
llama_new_context_with_model: KV self size  = 6016.00 MiB, K (f16): 3008.00 MiB, V (f16): 3008.00 MiB
llama_new_context_with_model:        CPU  output buffer size =     0.58 MiB
llama_new_context_with_model:        CPU compute buffer size =  5004.00 MiB
llama_new_context_with_model: graph nodes  = 3672
llama_new_context_with_model: graph splits = 1

main: n_kv_max = 32768, n_batch = 8192, n_ubatch = 8192, flash_attn = 1, n_gpu_layers = -1, n_threads = 16, n_threads_batch = 24

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
C:\Users\Administrator\source\repos\ik_llama.cpp\ggml\src\iqk\iqk_gemm_legacy_quants.cpp:1774: GGML_ASSERT(nrc_x%8 == 0) failed
C:\Users\Administrator\source\repos\ik_llama.cpp\ggml\src\iqk\iqk_gemm_legacy_quants.cpp:1774:
C:\Users\Administrator\Source\repos\ik_llama.cpp\build-p710-cpu\bin>
   ```

</details>
  
  Not sure what I am doing wrong.

---

👤 **ubergarm** commented on **2025-08-20** at **21:38:55**

Did one quick comparison between this PR and main on my AMD 9950X compiled CPU only and running this [Qwen3-30B-A3B-Thinking-2705-IQ4_KSS](https://huggingface.co/ubergarm/Qwen3-30B-A3B-Thinking-2507-GGUF#iq4_kss-15531-gib-4370-bpw) which isn't perfect as it has some `q8_0` tensors not involved with this PR.

Despite that, still seeing Prompt Processing uplift of around 15-33% at low kv-cache depth depending on how I'm running it. I'll try to get some more testing in soon including on that remote AMD EPYC 9965 192-Core rig where it could definitely help given I run it CPU-only.

<details>

<summary>👈 Details</summary>

```bash
cmake -B build -DGGML_CUDA=0 -DGGML_VULKAN=0 -DGGML_BLAS=0
cmake --build build --config Release -j $(nproc)

./build/bin/llama-sweep-bench \
  --model "$model" \
  -fa -fmoe \
  -ctk q8_0 -ctv q8_0 \
  -c 10240 \
  -rtr \
  --threads 16 \
  --warmup-batch
```

## ik_llama.cpp ik/q8_k_r16@f3edfe0f -rtr -ctk q8_0 -ctv q8_0
|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|   512 |    128 |      0 |    0.777 |   659.28 |    3.319 |    38.56 |
|   512 |    128 |    512 |    0.817 |   626.93 |    3.370 |    37.99 |
|   512 |    128 |   1024 |    0.859 |   595.83 |    3.433 |    37.29 |
|   512 |    128 |   1536 |    0.923 |   554.74 |    3.492 |    36.66 |
|   512 |    128 |   2048 |    0.951 |   538.13 |    3.561 |    35.95 |
|   512 |    128 |   2560 |    0.999 |   512.52 |    3.628 |    35.28 |
|   512 |    128 |   3072 |    1.041 |   491.98 |    3.691 |    34.68 |
|   512 |    128 |   3584 |    1.095 |   467.56 |    3.776 |    33.90 |
|   512 |    128 |   4096 |    1.129 |   453.40 |    3.842 |    33.31 |
|   512 |    128 |   4608 |    1.176 |   435.52 |    3.904 |    32.78 |
|   512 |    128 |   5120 |    1.214 |   421.64 |    3.970 |    32.24 |
|   512 |    128 |   5632 |    1.267 |   404.09 |    4.023 |    31.82 |
|   512 |    128 |   6144 |    1.307 |   391.60 |    4.043 |    31.66 |
|   512 |    128 |   6656 |    1.351 |   378.96 |    4.116 |    31.10 |
|   512 |    128 |   7168 |    1.392 |   367.87 |    4.250 |    30.12 |
|   512 |    128 |   7680 |    1.439 |   355.70 |    4.322 |    29.61 |
|   512 |    128 |   8192 |    1.592 |   321.69 |    4.399 |    29.10 |
|   512 |    128 |   8704 |    1.566 |   326.97 |    4.519 |    28.33 |
|   512 |    128 |   9216 |    1.587 |   322.69 |    4.584 |    27.92 |
|   512 |    128 |   9728 |    1.636 |   312.99 |    4.706 |    27.20 |

## ik_llama.cpp main@0cb66969 -rtr -ctk q8_0 -ctv q8_0
|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|   512 |    128 |      0 |    0.895 |   572.05 |    3.329 |    38.45 |
|   512 |    128 |    512 |    0.947 |   540.56 |    3.383 |    37.83 |
|   512 |    128 |   1024 |    0.986 |   519.32 |    3.437 |    37.24 |
|   512 |    128 |   1536 |    1.034 |   495.17 |    3.497 |    36.60 |
|   512 |    128 |   2048 |    1.078 |   474.92 |    3.569 |    35.86 |
|   512 |    128 |   2560 |    1.138 |   449.93 |    3.639 |    35.18 |
|   512 |    128 |   3072 |    1.192 |   429.44 |    3.712 |    34.49 |
|   512 |    128 |   3584 |    1.223 |   418.58 |    3.770 |    33.95 |
|   512 |    128 |   4096 |    1.263 |   405.43 |    3.817 |    33.53 |
|   512 |    128 |   4608 |    1.302 |   393.13 |    3.885 |    32.94 |
|   512 |    128 |   5120 |    1.341 |   381.81 |    4.003 |    31.98 |
|   512 |    128 |   5632 |    1.389 |   368.52 |    4.075 |    31.41 |
|   512 |    128 |   6144 |    1.429 |   358.37 |    4.154 |    30.81 |
|   512 |    128 |   6656 |    1.488 |   344.03 |    4.234 |    30.23 |
|   512 |    128 |   7168 |    1.525 |   335.75 |    4.325 |    29.59 |
|   512 |    128 |   7680 |    1.588 |   322.38 |    4.383 |    29.21 |
|   512 |    128 |   8192 |    1.636 |   312.89 |    4.439 |    28.84 |
|   512 |    128 |   8704 |    1.686 |   303.68 |    4.505 |    28.41 |
|   512 |    128 |   9216 |    1.693 |   302.50 |    4.486 |    28.53 |
|   512 |    128 |   9728 |    1.734 |   295.20 |    4.546 |    28.16 |

## ik_llama.cpp ik/q8_k_r16@f3edfe0f --no-mmap -ub 4096 -b 4096 -ctk q8_0 -ctv q8_0
|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  4096 |   1024 |      0 |    7.005 |   584.71 |   27.322 |    37.48 |
|  4096 |   1024 |   4096 |    9.823 |   416.96 |   31.609 |    32.40 |

## ik_llama.cpp main@0cb66969 --no-mmap -ub 4096 -b 4096 -ctk q8_0 -ctv q8_0
|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  4096 |   1024 |      0 |    9.327 |   439.15 |   27.215 |    37.63 |
|  4096 |   1024 |   4096 |   11.943 |   342.97 |   31.609 |    32.40 |

## ik_llama.cpp ik/q8_k_r16@f3edfe0f --no-mmap -ub 4096 -b 4096
|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  4096 |   1024 |      0 |    8.699 |   470.87 |   27.942 |    36.65 |
|  4096 |   1024 |   4096 |   13.666 |   299.72 |   34.688 |    29.52 |

## ik_llama.cpp main@0cb66969 --no-mmap -ub 4096 -b 4096
|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  4096 |   1024 |      0 |   10.817 |   378.66 |   27.710 |    36.95 |
|  4096 |   1024 |   4096 |   15.396 |   266.05 |   34.648 |    29.55 |


</details>

<img width="2222" height="1111" alt="sweep-bench-pr710-Qwen3-30B" src="https://github.com/user-attachments/assets/07bbb97d-b306-49e6-a0c5-e320a25d4445" />

@sousekd  

Hrmm, your `EPYC 9355` is Zen5 so should support the CPU flags assuming they are all passed through with QEMU and supported on that version of windows. (Maybe you can check they are passing through correctly with CPU-Z or however you do `lscpu` in Windows?)

Likely unrelated to the ASSERT you mention, a few things with your command:
* `--alias` can remove this as it is only for llama-server, but hurts nothing
* `-amb 512` can remove this, it only applies to MLA models with mode 2/3, does nothing here
*  `-b 8192 -ub 8192` i try not to go over 4096 on these as going higher can get weird
*  `--parallel 1` not needed here, only for llama-server, but doesn't hurt anything 

Everything else looks reasonable to me. Not sure what is up with that ASSERT when compiling CPU-only which is a good way to test this PR.

---

👤 **sousekd** commented on **2025-08-21** at **00:21:43**

@ubergarm Yeah, thank you for the notes. I have a universal semi-interactive script for running these tests more easily, so there are params that only affect the llama-server or some other models. Anyway, this is what CPU-Z shows inside the VM:

<img width="400" height="400" alt="image" src="https://github.com/user-attachments/assets/738ff60a-c72d-4019-b770-fcc985173ba1" />

I didn’t see any significant gains last time when I tested [#610](https://github.com/ikawrakow/ik_llama.cpp/issues/610) on Windows on bare metal either. Not sure why - it might just be Windows. With Proxmox in the way, things got even more complicated and obscured. I expected some drop in performance due to virtualization, and I did see that on most metrics (around 5–10%), but interestingly RAM read bandwidth doubled when measured by OCCT, now well over 1000 GB/s - probably an impossible value for the hardware and difficult to explain. And don’t even get me started on CPU threads and core pinning - it’s a mess!

Good to see improvements with this PR from other people. I’m sure I’ll figure out the oddities of my setup eventually.
I can boot up another machine with 9950X if helpful, but I assume your results show the gains clearly.

---

👤 **ikawrakow** commented on **2025-08-21** at **05:16:31**

@sousekd 

Thanks for testing. Short of downloading the exact same model and running the exact same command as you I'm absolutely not able to reproduce the assert. But I have pushed a change that hopefully fixes that.

Also, because there is some doubt if you are actually enabling the faster "fancy" SIMD path, I have added a hack that will tell us if it is enabled. If you pull the latest, rebuild, and run any command, you will in then output either
```
======================================= HAVE_FANCY_SIMD is defined
```
or
```
======================================= HAVE_FANCY_SIMD is NOT defined
```

Oh, one more thing: `-b 8192 -ub 8192` is good for the GPU (if you have enough VRAM), but not for the CPU. Running CPU-only PP performance is maximized somewhere around `u-batch = 256-1024` (depending on model, number of threads, etc.). So, unless you want to quickly cover a wide range of context lengths (as the `N_KV` step in `sweep-bench` is determined by `-ub`), you don't really want to use such large u-batches when running CPU-only.

---

👤 **ikawrakow** commented on **2025-08-21** at **05:44:51**

@ubergarm 

Thanks for testing! 

It looks like in your case `-rtr` is faster than no `-rtr`? This is interesting. On my 7950X, with this PR and the model you have used I get

### With -rtr

 |    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|   512 |    128 |      0 |    1.153 |   443.88 |    5.179 |    24.72 |
|   512 |    128 |    512 |    1.194 |   428.89 |    5.263 |    24.32 |
|   512 |    128 |   1024 |    1.265 |   404.77 |    5.375 |    23.81 |
|   512 |    128 |   1536 |    1.348 |   379.75 |    5.484 |    23.34 |
|   512 |    128 |   2048 |    1.423 |   359.90 |    5.573 |    22.97 |
|   512 |    128 |   2560 |    1.497 |   342.03 |    5.666 |    22.59 |
|   512 |    128 |   3072 |    1.579 |   324.29 |    5.752 |    22.25 |
|   512 |    128 |   3584 |    1.686 |   303.63 |    5.845 |    21.90 |
|   512 |    128 |   4096 |    1.738 |   294.67 |    5.958 |    21.48 |

### Without -rtr

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|   512 |    128 |      0 |    1.008 |   508.03 |    4.855 |    26.37 |
|   512 |    128 |    512 |    1.046 |   489.32 |    5.102 |    25.09 |
|   512 |    128 |   1024 |    1.116 |   458.62 |    5.006 |    25.57 |
|   512 |    128 |   1536 |    1.194 |   428.88 |    5.098 |    25.11 |
|   512 |    128 |   2048 |    1.264 |   405.05 |    5.193 |    24.65 |
|   512 |    128 |   2560 |    1.336 |   383.25 |    5.263 |    24.32 |
|   512 |    128 |   3072 |    1.428 |   358.63 |    5.433 |    23.56 |
|   512 |    128 |   3584 |    1.514 |   338.27 |    5.551 |    23.06 |
|   512 |    128 |   4096 |    1.590 |   322.10 |    5.615 |    22.80 |

In any case, it would be useful to have the benchmark results for the Qwen3-14B model that you used in [#610](https://github.com/ikawrakow/ik_llama.cpp/issues/610) to know if this PR is as good as [#610](https://github.com/ikawrakow/ik_llama.cpp/issues/610)

---

👤 **saood06** commented on **2025-08-21** at **05:51:41**

>(as the N_KV step in sweep-bench is determined by -ub), you don't really want to use such large u-batches when running CPU-only.

I've been thinking about improvements to sweep bench that would sweep with varying or multiple batch sizes, and then do TG afterwards testing at varying points (I don't like how little data you get about TG when testing with large batch sizes, or how little you can configure how quick/thorough to be) but with a configurable amount of tokens at a time (independent of batch size). I haven't gotten around to doing it, and not even sure how much people would want it.

---

👤 **ikawrakow** commented on **2025-08-21** at **05:58:16**

> I've been thinking about improvements to sweep bench ...

I have been thinking about that too. On a number of occasions I have wished that I can define the step size of the sweep independently from the batch/u-batch being used and the number of TG tokens. But it never became painful enough so I sit down and do it.

In any case, for the tests being done for this PR running CPU only, my point was to not use such huge batch and u-batch sizes. It is also not necessary to go to very high context lengths.

---

👤 **saood06** commented on **2025-08-21** at **06:04:55**

>But it never became painful enough so I sit down and do it.

Yep. Although I have occasionally hard coded 32 instead of ubatch/4, but the better solution would do TG differently (PP pass to build the KV, TG breaks it down by removing from the cache until you are wherever you want to be to collect data).

---

👤 **ubergarm** commented on **2025-08-21** at **17:07:22**

@ikawrakow 

> It looks like in your case -rtr is faster than no -rtr? This is interesting. On my 7950X, with this PR and the model you have used I get

No, good point, I had not tried the default batches without rtr, and in like your 7950X without `-rtr` is faster:

<img width="2083" height="1111" alt="sweep-bench-pr710-take-2-Qwen3-30B" src="https://github.com/user-attachments/assets/98a83409-93db-4617-8507-827065c8fd50" />

> ... it would be useful to have the benchmark results for the Qwen3-14B model that you used in https://github.com/ikawrakow/ik_llama.cpp/pull/610 to know if this PR is as good as https://github.com/ikawrakow/ik_llama.cpp/pull/610

I'll try to keep it simple and just use default batches without `-rtr` and do a 3-way comparison between main, this PR710, and the previous PR610. For this model, and if I can dig up that earlier model I'll run it too and update. (DeepSeek-V3.1 Instruct is out today too, working on that).

---

👤 **ubergarm** commented on **2025-08-21** at **17:15:50**

So with this quant, PR710 > PR610 > main for PP speed. I do have that old `Qwen3-14B-Q8_0_R8.gguf` used on PR610 as well, and will do this 3-wa comparison with that on my AMD 9950X next.

<img width="2376" height="1082" alt="sweep-bench-pr710-take-3-Qwen3-30B" src="https://github.com/user-attachments/assets/88e202b4-8c70-40e7-917b-a5cb6d096b22" />

<details>

<summary>👈 Details</summary>

```bash
./build/bin/llama-sweep-bench \
  --model "$model" \
  -fa -fmoe \
  -ctk q8_0 -ctv q8_0 \
  -c 10240 \
  --no-mmap \
  --threads 16 \
  --warmup-batch
```

## ik_llama.cpp PR710 ik/q8_k_r16@f3edfe0f --no-mmap -ctk q8_0 -ctv q8_0
|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|   512 |    128 |      0 |    0.643 |   796.48 |    3.365 |    38.03 |
|   512 |    128 |    512 |    0.682 |   750.46 |    3.441 |    37.20 |
|   512 |    128 |   1024 |    0.724 |   707.49 |    3.504 |    36.53 |
|   512 |    128 |   1536 |    0.779 |   657.40 |    3.567 |    35.88 |
|   512 |    128 |   2048 |    0.814 |   628.92 |    3.633 |    35.24 |
|   512 |    128 |   2560 |    0.864 |   592.40 |    3.720 |    34.41 |
|   512 |    128 |   3072 |    0.903 |   566.81 |    3.787 |    33.80 |
|   512 |    128 |   3584 |    0.955 |   536.13 |    3.840 |    33.33 |
|   512 |    128 |   4096 |    0.991 |   516.86 |    3.905 |    32.78 |
|   512 |    128 |   4608 |    1.035 |   494.51 |    3.974 |    32.21 |
|   512 |    128 |   5120 |    1.077 |   475.48 |    4.044 |    31.65 |
|   512 |    128 |   5632 |    1.125 |   455.02 |    4.134 |    30.96 |
|   512 |    128 |   6144 |    1.170 |   437.64 |    4.193 |    30.53 |
|   512 |    128 |   6656 |    1.333 |   384.11 |    4.261 |    30.04 |
|   512 |    128 |   7168 |    1.259 |   406.70 |    4.286 |    29.86 |
|   512 |    128 |   7680 |    1.309 |   391.21 |    4.351 |    29.42 |
|   512 |    128 |   8192 |    1.355 |   377.74 |    4.417 |    28.98 |
|   512 |    128 |   8704 |    1.389 |   368.61 |    4.463 |    28.68 |
|   512 |    128 |   9216 |    1.430 |   358.14 |    4.549 |    28.14 |
|   512 |    128 |   9728 |    1.491 |   343.34 |    4.622 |    27.69 |

## ik_llama.cpp main@0cb66969 --no-mmap -ctk q8_0 -ctv q8_0
|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|   512 |    128 |      0 |    0.879 |   582.80 |    3.372 |    37.96 |
|   512 |    128 |    512 |    0.924 |   554.25 |    3.416 |    37.47 |
|   512 |    128 |   1024 |    0.970 |   527.58 |    3.486 |    36.72 |
|   512 |    128 |   1536 |    1.028 |   497.90 |    3.547 |    36.09 |
|   512 |    128 |   2048 |    1.064 |   481.02 |    3.595 |    35.61 |
|   512 |    128 |   2560 |    1.115 |   459.01 |    3.674 |    34.84 |
|   512 |    128 |   3072 |    1.151 |   444.80 |    3.727 |    34.35 |
|   512 |    128 |   3584 |    1.205 |   424.78 |    3.816 |    33.55 |
|   512 |    128 |   4096 |    1.241 |   412.52 |    3.884 |    32.96 |
|   512 |    128 |   4608 |    1.287 |   397.76 |    3.958 |    32.34 |
|   512 |    128 |   5120 |    1.329 |   385.18 |    4.024 |    31.81 |
|   512 |    128 |   5632 |    1.385 |   369.76 |    4.090 |    31.29 |
|   512 |    128 |   6144 |    1.420 |   360.46 |    4.158 |    30.79 |
|   512 |    128 |   6656 |    1.471 |   348.06 |    4.255 |    30.08 |
|   512 |    128 |   7168 |    1.512 |   338.66 |    4.267 |    29.99 |
|   512 |    128 |   7680 |    1.577 |   324.59 |    4.336 |    29.52 |
|   512 |    128 |   8192 |    1.595 |   321.02 |    4.451 |    28.75 |
|   512 |    128 |   8704 |    1.641 |   312.01 |    4.512 |    28.37 |
|   512 |    128 |   9216 |    1.688 |   303.38 |    4.614 |    27.74 |
|   512 |    128 |   9728 |    1.740 |   294.29 |    4.665 |    27.44 |

## ik_llama.cpp PR610 ik/q8_k_r8_avx512+main@0cb66969 --no-mmap -ctk q8_0 -ctv q8_0
|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|   512 |    128 |      0 |    0.673 |   760.21 |    3.395 |    37.70 |
|   512 |    128 |    512 |    0.714 |   716.80 |    3.456 |    37.04 |
|   512 |    128 |   1024 |    0.757 |   676.24 |    3.519 |    36.38 |
|   512 |    128 |   1536 |    0.807 |   634.65 |    3.584 |    35.71 |
|   512 |    128 |   2048 |    0.849 |   603.16 |    3.650 |    35.06 |
|   512 |    128 |   2560 |    0.899 |   569.74 |    3.709 |    34.51 |
|   512 |    128 |   3072 |    0.938 |   545.95 |    3.804 |    33.65 |
|   512 |    128 |   3584 |    0.991 |   516.59 |    3.852 |    33.23 |
|   512 |    128 |   4096 |    1.030 |   497.07 |    3.922 |    32.64 |
|   512 |    128 |   4608 |    1.073 |   477.09 |    3.988 |    32.10 |
|   512 |    128 |   5120 |    1.110 |   461.37 |    4.057 |    31.55 |
|   512 |    128 |   5632 |    1.160 |   441.51 |    4.150 |    30.84 |
|   512 |    128 |   6144 |    1.198 |   427.27 |    4.224 |    30.30 |
|   512 |    128 |   6656 |    1.245 |   411.32 |    4.292 |    29.82 |
|   512 |    128 |   7168 |    1.291 |   396.62 |    4.368 |    29.31 |
|   512 |    128 |   7680 |    1.387 |   369.16 |    4.286 |    29.86 |
|   512 |    128 |   8192 |    1.374 |   372.56 |    4.398 |    29.11 |
|   512 |    128 |   8704 |    1.430 |   358.10 |    4.405 |    29.06 |
|   512 |    128 |   9216 |    1.461 |   350.53 |    4.568 |    28.02 |
|   512 |    128 |   9728 |    1.517 |   337.53 |    4.631 |    27.64 |


</details>

---

👤 **ubergarm** commented on **2025-08-21** at **17:42:06**

Okay for the test quant I used on PR610 `Qwen3-14B-Q8_K_R8.gguf` it is looking like for PP speed for PR610 > PR710 > main in this case.

Should I try to make a `Q8_K_R16` test quant or is this sufficient? I'll try to get some data with this on the bigger rig too soon with the new DeepSeek-V3.1 for some "real world" examples of quant recipes I'll release.

<img width="2087" height="1082" alt="sweep-bench-pr710-take-4-Qwen3-30B" src="https://github.com/user-attachments/assets/75f302b0-3f44-4a14-912c-1220a933ddad" />


<details>

<summary>👈 Details</summary>

```bash
/build/bin/llama-sweep-bench \
  --model "$model" \
  -fa \
  -c 8704 \
  --warmup-batch \
  --threads 16
```

## ik_llama.cpp PR710 ik/q8_k_r16@f3edfe0f
|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|   512 |    128 |      0 |    2.083 |   245.82 |   21.395 |     5.98 |
|   512 |    128 |    512 |    2.154 |   237.66 |   21.599 |     5.93 |
|   512 |    128 |   1024 |    2.246 |   227.98 |   21.790 |     5.87 |
|   512 |    128 |   1536 |    2.329 |   219.86 |   22.056 |     5.80 |
|   512 |    128 |   2048 |    2.410 |   212.46 |   22.366 |     5.72 |
|   512 |    128 |   2560 |    2.488 |   205.79 |   22.578 |     5.67 |
|   512 |    128 |   3072 |    2.572 |   199.06 |   22.818 |     5.61 |
|   512 |    128 |   3584 |    2.645 |   193.58 |   23.056 |     5.55 |
|   512 |    128 |   4096 |    2.703 |   189.38 |   23.301 |     5.49 |

## ik_llama.cpp main@0cb66969
|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|   512 |    128 |      0 |    2.864 |   178.77 |   21.413 |     5.98 |
|   512 |    128 |    512 |    2.943 |   173.98 |   21.619 |     5.92 |
|   512 |    128 |   1024 |    3.043 |   168.26 |   21.832 |     5.86 |
|   512 |    128 |   1536 |    3.110 |   164.61 |   22.047 |     5.81 |
|   512 |    128 |   2048 |    3.176 |   161.22 |   22.277 |     5.75 |
|   512 |    128 |   2560 |    3.248 |   157.62 |   22.507 |     5.69 |
|   512 |    128 |   3072 |    3.327 |   153.89 |   22.797 |     5.61 |
|   512 |    128 |   3584 |    3.380 |   151.50 |   23.015 |     5.56 |
|   512 |    128 |   4096 |    3.451 |   148.37 |   23.211 |     5.51 |

## ik_llama.cpp PR610 ik/q8_k_r8_avx512+main@0cb66969
|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|   512 |    128 |      0 |    1.591 |   321.80 |   21.441 |     5.97 |
|   512 |    128 |    512 |    1.662 |   308.05 |   21.649 |     5.91 |
|   512 |    128 |   1024 |    1.730 |   295.89 |   21.882 |     5.85 |
|   512 |    128 |   1536 |    1.809 |   282.97 |   22.154 |     5.78 |
|   512 |    128 |   2048 |    1.874 |   273.28 |   22.402 |     5.71 |
|   512 |    128 |   2560 |    1.941 |   263.81 |   22.678 |     5.64 |
|   512 |    128 |   3072 |    2.022 |   253.24 |   22.974 |     5.57 |
|   512 |    128 |   3584 |    2.091 |   244.81 |   23.156 |     5.53 |
|   512 |    128 |   4096 |    2.202 |   232.47 |   23.408 |     5.47 |

</details>

---

👤 **ubergarm** commented on **2025-08-21** at **17:57:21**

One more quick test using a "pure" IQ2_KT which is *not* involved in these PRs pretty sure given the speed is the same across all three test cases.

<img width="2087" height="1082" alt="sweep-bench-pr710-take-5-Qwen3-14B" src="https://github.com/user-attachments/assets/10e0eb95-6598-4441-a42e-1c7d5373c1b9" />


<details>

<summary>👈 Details</summary>

```bash
./build/bin/llama-sweep-bench \
  --model "$model" \
  -fa \
  -ctk q8_0 -ctv q8_0 \
  -c 8704 \
  --warmup-batch \
  --threads 16
```

## ik_llama.cpp PR710 ik/q8_k_r16@f3edfe0f
|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|   512 |    128 |      0 |    1.905 |   268.81 |    7.923 |    16.16 |
|   512 |    128 |    512 |    1.943 |   263.50 |    8.011 |    15.98 |
|   512 |    128 |   1024 |    1.992 |   257.03 |    8.158 |    15.69 |
|   512 |    128 |   1536 |    2.042 |   250.78 |    8.279 |    15.46 |
|   512 |    128 |   2048 |    2.093 |   244.66 |    8.395 |    15.25 |
|   512 |    128 |   2560 |    2.145 |   238.69 |    8.510 |    15.04 |
|   512 |    128 |   3072 |    2.185 |   234.31 |    8.637 |    14.82 |
|   512 |    128 |   3584 |    2.244 |   228.15 |    8.751 |    14.63 |
|   512 |    128 |   4096 |    2.286 |   224.00 |    8.956 |    14.29 |

## ik_llama.cpp main@0cb66969
|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|   512 |    128 |      0 |    1.892 |   270.65 |    7.893 |    16.22 |
|   512 |    128 |    512 |    1.933 |   264.82 |    8.022 |    15.96 |
|   512 |    128 |   1024 |    1.989 |   257.36 |    8.130 |    15.74 |
|   512 |    128 |   1536 |    2.034 |   251.74 |    8.263 |    15.49 |
|   512 |    128 |   2048 |    2.084 |   245.64 |    8.383 |    15.27 |
|   512 |    128 |   2560 |    2.137 |   239.59 |    8.479 |    15.10 |
|   512 |    128 |   3072 |    2.184 |   234.48 |    8.630 |    14.83 |
|   512 |    128 |   3584 |    2.228 |   229.76 |    8.664 |    14.77 |
|   512 |    128 |   4096 |    2.278 |   224.71 |    8.806 |    14.54 |

## ik_llama.cpp PR610 ik/q8_k_r8_avx512+main@0cb66969
|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|   512 |    128 |      0 |    1.902 |   269.23 |    7.921 |    16.16 |
|   512 |    128 |    512 |    1.951 |   262.48 |    8.029 |    15.94 |
|   512 |    128 |   1024 |    1.996 |   256.53 |    8.142 |    15.72 |
|   512 |    128 |   1536 |    2.055 |   249.10 |    8.201 |    15.61 |
|   512 |    128 |   2048 |    2.104 |   243.33 |    8.343 |    15.34 |
|   512 |    128 |   2560 |    2.143 |   238.93 |    8.457 |    15.14 |
|   512 |    128 |   3072 |    2.194 |   233.39 |    8.616 |    14.86 |
|   512 |    128 |   3584 |    2.249 |   227.63 |    8.679 |    14.75 |
|   512 |    128 |   4096 |    2.296 |   223.03 |    8.822 |    14.51 |


</details>

---

👤 **ubergarm** commented on **2025-08-21** at **18:11:29**

Okay sorry for spamming this thread lol, one more example of a "pure" IQ2_KL which *is* used in this PR psure (there is a commit comment for it, but I believe it was omitted in the PR description).

For this case again we see PR710 > PR610 > main for PP speed.

<img width="2087" height="1082" alt="sweep-bench-pr710-take-6-Qwen3-14B" src="https://github.com/user-attachments/assets/a9e526c5-2659-4da8-a35c-bc5ce427edc0" />


<details>

<summary>👈 Details</summary>

```bash
./build/bin/llama-sweep-bench \
  --model "$model" \
  -fa \
  -ctk q8_0 -ctv q8_0 \
  -c 8704 \
  --warmup-batch \
  --threads 16
```

## ik_llama.cpp PR710 ik/q8_k_r16@f3edfe0f
|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|   512 |    128 |      0 |    1.477 |   346.69 |    8.217 |    15.58 |
|   512 |    128 |    512 |    1.514 |   338.11 |    8.038 |    15.92 |
|   512 |    128 |   1024 |    1.561 |   328.03 |    8.120 |    15.76 |
|   512 |    128 |   1536 |    1.606 |   318.84 |    8.243 |    15.53 |
|   512 |    128 |   2048 |    1.666 |   307.41 |    8.352 |    15.33 |
|   512 |    128 |   2560 |    1.685 |   303.89 |    8.470 |    15.11 |
|   512 |    128 |   3072 |    1.735 |   295.11 |    8.677 |    14.75 |
|   512 |    128 |   3584 |    1.785 |   286.77 |    8.848 |    14.47 |
|   512 |    128 |   4096 |    1.826 |   280.38 |    8.997 |    14.23 |

## ik_llama.cpp main@0cb66969
|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|   512 |    128 |      0 |    3.061 |   167.25 |    7.935 |    16.13 |
|   512 |    128 |    512 |    3.001 |   170.61 |    8.021 |    15.96 |
|   512 |    128 |   1024 |    3.053 |   167.70 |    8.107 |    15.79 |
|   512 |    128 |   1536 |    3.112 |   164.51 |    8.210 |    15.59 |
|   512 |    128 |   2048 |    3.273 |   156.45 |    8.344 |    15.34 |
|   512 |    128 |   2560 |    3.217 |   159.14 |    8.544 |    14.98 |
|   512 |    128 |   3072 |    3.260 |   157.07 |    8.697 |    14.72 |
|   512 |    128 |   3584 |    3.301 |   155.09 |    8.854 |    14.46 |
|   512 |    128 |   4096 |    3.357 |   152.53 |    9.013 |    14.20 |

## ik_llama.cpp PR610 ik/q8_k_r8_avx512+main@0cb66969
|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|   512 |    128 |      0 |    1.601 |   319.88 |    7.879 |    16.24 |
|   512 |    128 |    512 |    1.648 |   310.74 |    8.003 |    15.99 |
|   512 |    128 |   1024 |    1.695 |   302.02 |    8.120 |    15.76 |
|   512 |    128 |   1536 |    1.748 |   292.86 |    8.243 |    15.53 |
|   512 |    128 |   2048 |    1.790 |   286.08 |    8.410 |    15.22 |
|   512 |    128 |   2560 |    1.838 |   278.61 |    8.537 |    14.99 |
|   512 |    128 |   3072 |    1.884 |   271.70 |    8.710 |    14.70 |
|   512 |    128 |   3584 |    1.932 |   265.01 |    8.939 |    14.32 |
|   512 |    128 |   4096 |    1.983 |   258.16 |    9.063 |    14.12 |


</details>

---

👤 **ikawrakow** commented on **2025-08-21** at **19:10:07**

@ubergarm  Thanks for the thorough testing! It seems to be even slightly better than [#610](https://github.com/ikawrakow/ik_llama.cpp/issues/610)!

> Should I try to make a Q8_K_R16 test quant or is this sufficient? 

I haven't even implemented the ability to quantize a model to `Q8_K_R16`. It is only there to be used for quantizing the activations. The lower `Q8_K_R8` performance compared to [#610](https://github.com/ikawrakow/ik_llama.cpp/issues/610) is expected because here we are back to the original `Q8_K_R8` implementation (except for removing the template specialization with 16 columns). To get better there I need to special-case the implementation, which I didn't want to do in this PR.

So, the only thing left to resolve before merging is the assert observed by @sousekd

---

👤 **sousekd** commented on **2025-08-21** at **19:10:37**

I tried again **ubergarm/Qwen3-235B-A22B-Instruct-2507-IQ5_K** on the current version, this time with default batch size, and I finally see quite significant improvement. `HAVE_FANCY_SIMD is defined` :).

```
.\bin\llama-sweep-bench.exe `
    --model xyz `
    --no-mmap -fa -fmoe `
    -c 32768 -amb 512 `
    --parallel 1 --threads 16 --threads-batch 24 `
    --warmup-batch
```

EPYC 9355 CPU only, 12x DDR5 6400, Windows VM on Proxmox:

<img width="1979" height="1180" alt="image" src="https://github.com/user-attachments/assets/07afc66d-2c92-4349-891b-858099c94c3a" />
<img width="1979" height="1180" alt="image" src="https://github.com/user-attachments/assets/0293ec4a-5f9e-4fe3-8f95-b3b4a839e7f2" />

---

👤 **sousekd** commented on **2025-08-21** at **19:42:45**

> So, the only thing left to resolve before merging is the assert observed by @sousekd

All sorted. I don't see the assert now even with the `-b 8192 -ub 8192` I used yesterday.