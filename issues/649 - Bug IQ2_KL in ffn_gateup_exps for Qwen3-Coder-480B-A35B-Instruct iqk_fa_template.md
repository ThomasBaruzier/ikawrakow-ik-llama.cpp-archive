## 📌 [Issue #649](https://github.com/ikawrakow/ik_llama.cpp/issues/649) - Bug: IQ2_KL in ffn_(gate|up)_exps for Qwen3-Coder-480B-A35B-Instruct `iqk_fa_templates.h:1146: GGML_ASSERT(fms.S[j] > 0) failed`

| **Author** | `ubergarm` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-07-26 |
| **Updated** | 2025-08-28 |

---

## 📄 Description

### What happened?

I cooked a quant using IQ2_KL in ffn_(gate|up)_exps tensors for Qwen3-Coder-480B-A35B-Instruct. However, when trying to run it throws  `iqk_fa_templates.h:1146: GGML_ASSERT(fms.S[j] > 0) failed` and exits on startup. Compiled CPU-only backend.

Oddly though, I have a different quant using IQ2_KL in only the ffn_down tensors and it works fine. It is identical to the recipe except for slightly larger routed exps:
```
# Routed Experts
blk\..*\.ffn_down_exps\.weight=iq2_kl
blk\..*\.ffn_(gate|up)_exps\.weight=iq2_k
```

Finally just to test, I tried removing `-fmoe` but it failed with the same assert error. Removing `-fa` it exited with this error: `Oops(ggml_compute_forward_sum_rows_f32, ffn_moe_weights_sum-2): found nan for i1 = 0, i2 = 0, i3 = 0. ne00 = 160`

I noticed this issue before releasing the quant so no rush. If I have time I might try rolling back to earlier version to see if it was possibly a regression.

*EDIT*: Also my recent `Kimi-K2-Instruct-IQ2_KL` quants are working fine too:
```
Adding custom rule blk\..*\.ffn_down_exps\.weight -> iq3_ks
# Adding custom rule blk\..*\.ffn_down_exps\.weight -> iq2_kl # <--- i have one with all exps iq2_kl also
Adding custom rule blk\..*\.ffn_(gate|up)_exps\.weight -> iq2_kl
```

<details>

<summary>👈 IQ2_KL Quant Recipe</summary>

This is the quant that throws the error:
```bash
#!/usr/bin/env bash

# Repeating Layers [0-61]

custom="
# Attention
blk\..*\.attn_q.*=iq6_k
blk\..*\.attn_k.*=q8_0
blk\..*\.attn_v.*=q8_0
blk\..*\.attn_output.*=iq6_k

# Routed Experts
blk\..*\.ffn_down_exps\.weight=iq3_k
blk\..*\.ffn_(gate|up)_exps\.weight=iq2_kl

# Non-Repeating Layers
token_embd\.weight=iq4_k
output\.weight=iq6_k
"

custom=$(
  echo "$custom" | grep -v '^#' | \
  sed -Ez 's:\n+:,:g;s:,$::;s:^,::'
)

numactl -N 0 -m 0 \
./build/bin/llama-quantize \
    --custom-q "$custom" \
    --imatrix /mnt/raid/models/ubergarm/Qwen3-Coder-480B-A35B-Instruct-GGUF/imatrix-Qwen3-Coder-480B-A35B-Instruct-Q8_0.dat \
    /mnt/raid/models/ubergarm/Qwen3-Coder-480B-A35B-Instruct-GGUF/Qwen3-Coder-480B-A35B-Instruct-BF16-00001-of-00021.gguf \
    /mnt/raid/models/ubergarm/Qwen3-Coder-480B-A35B-Instruct-GGUF/Qwen3-Coder-480B-A35B-Instruct-IQ2_KL.gguf \
    IQ2_KL \
    192
```

</details>


<details>

<summary>👈 llama-server command, log and error</summary>

```bash
# compile CPU-only backend

model=/mnt/raid/hf/Qwen3-Coder-480B-A35B-Instruct-GGUF/IQ2_KL/Qwen3-480B-A35B-Instruct-IQ2_KL-00001-of-00004.gguf

numactl -N 1 -m 1 \
./build/bin/llama-server \
    --model "$model"\
    --alias ubergarm/Qwen3-Coder-480B-A35B-Instruct \
    --ctx-size 196608 \
    -ctk q8_0 -ctv q8_0 \
    -fa -fmoe \
    -ub 4096 -b 4096 \
    --parallel 1 \
    --threads 128 \
    --threads-batch 192 \
    --numa numactl \
    --host 127.0.0.1 \
    --port 8080 \
    --no-mmap

INFO [                    main] build info | tid="127586578487488" timestamp=1753302334 build=3821 commit="1b052109"
INFO [                    main] system info | tid="127586578487488" timestamp=1753302334 n_threads=128 n_threads_batch=192 total_threads=768 system_info="AVX = 1 | AVX_VNNI = 1 | AVX2 = 1 | AVX512 = 1 | AVX512_VBMI = 1 | AVX512_VNNI = 1 | AVX512_BF16 = 1 | FMA = 1 | NEON = 0 | SVE = 0 | ARM_FMA = 0 | F16C = 1 | FP16_VA = 0 | WASM_SIMD = 0 | BLAS = 0 | SSE3 = 1 | SSSE3 = 1 | VSX = 0 | MATMUL_INT8 = 0 | LLAMAFILE = 1 | "
llama_model_loader: additional 3 GGUFs metadata loaded.
llama_model_loader: loaded meta data with 41 key-value pairs and 747 tensors from /mnt/raid/hf/Qwen3-Coder-480B-A35B-Instruct-GGUF/IQ2_KL/Qwen3-480B-A35B-Instruct-IQ2_KL-00001-of-00004.gguf (version GGUF V3 (latest))
llama_model_loader: Dumping metadata keys/values. Note: KV overrides do not apply in this output.
llama_model_loader: - kv   0:                       general.architecture str              = qwen3moe
llama_model_loader: - kv   1:                               general.type str              = model
llama_model_loader: - kv   2:                               general.name str              = Qwen3 Coder 480B A35B Instruct
llama_model_loader: - kv   3:                           general.finetune str              = Instruct
llama_model_loader: - kv   4:                           general.basename str              = Qwen3-Coder
llama_model_loader: - kv   5:                         general.size_label str              = 480B-A35B
llama_model_loader: - kv   6:                            general.license str              = apache-2.0
llama_model_loader: - kv   7:                       general.license.link str              = https://huggingface.co/Qwen/Qwen3-Cod...
llama_model_loader: - kv   8:                               general.tags arr[str,1]       = ["text-generation"]
llama_model_loader: - kv   9:                       qwen3moe.block_count u32              = 62
llama_model_loader: - kv  10:                    qwen3moe.context_length u32              = 262144
llama_model_loader: - kv  11:                  qwen3moe.embedding_length u32              = 6144
llama_model_loader: - kv  12:               qwen3moe.feed_forward_length u32              = 8192
llama_model_loader: - kv  13:              qwen3moe.attention.head_count u32              = 96
llama_model_loader: - kv  14:           qwen3moe.attention.head_count_kv u32              = 8
llama_model_loader: - kv  15:                    qwen3moe.rope.freq_base f32              = 10000000.000000
llama_model_loader: - kv  16:  qwen3moe.attention.layer_norm_rms_epsilon f32              = 0.000001
llama_model_loader: - kv  17:                 qwen3moe.expert_used_count u32              = 8
llama_model_loader: - kv  18:              qwen3moe.attention.key_length u32              = 128
llama_model_loader: - kv  19:            qwen3moe.attention.value_length u32              = 128
llama_model_loader: - kv  20:                          general.file_type u32              = 155
llama_model_loader: - kv  21:                      qwen3moe.expert_count u32              = 160
llama_model_loader: - kv  22:        qwen3moe.expert_feed_forward_length u32              = 2560
llama_model_loader: - kv  23: qwen3moe.expert_shared_feed_forward_length u32              = 0
llama_model_loader: - kv  24:               general.quantization_version u32              = 2
llama_model_loader: - kv  25:                       tokenizer.ggml.model str              = gpt2
llama_model_loader: - kv  26:                         tokenizer.ggml.pre str              = qwen2
llama_model_loader: - kv  27:                      tokenizer.ggml.tokens arr[str,151936]  = ["!", "\"", "#", "$", "%", "&", "'", ...
llama_model_loader: - kv  28:                  tokenizer.ggml.token_type arr[i32,151936]  = [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, ...
llama_model_loader: - kv  29:                      tokenizer.ggml.merges arr[str,151387]  = ["Ġ Ġ", "ĠĠ ĠĠ", "i n", "Ġ t",...
llama_model_loader: - kv  30:                tokenizer.ggml.eos_token_id u32              = 151645
llama_model_loader: - kv  31:            tokenizer.ggml.padding_token_id u32              = 151643
llama_model_loader: - kv  32:               tokenizer.ggml.add_bos_token bool             = false
llama_model_loader: - kv  33:                    tokenizer.chat_template str              = {% macro render_item_list(item_list, ...
llama_model_loader: - kv  34:                      quantize.imatrix.file str              = /mnt/raid/models/ubergarm/Qwen3-Coder...
llama_model_loader: - kv  35:                   quantize.imatrix.dataset str              = ubergarm-imatrix-calibration-corpus-v...
llama_model_loader: - kv  36:             quantize.imatrix.entries_count i32              = 497
llama_model_loader: - kv  37:              quantize.imatrix.chunks_count i32              = 840
llama_model_loader: - kv  38:                                   split.no u16              = 0
llama_model_loader: - kv  39:                                split.count u16              = 4
llama_model_loader: - kv  40:                        split.tensors.count i32              = 747
llama_model_loader: - type  f32:  311 tensors
llama_model_loader: - type q8_0:  124 tensors
llama_model_loader: - type iq3_k:   62 tensors
llama_model_loader: - type iq4_k:    1 tensors
llama_model_loader: - type iq6_k:  125 tensors
llama_model_loader: - type iq2_kl:  124 tensors
llm_load_vocab: special tokens cache size = 26
llm_load_vocab: token to piece cache size = 0.9311 MB
llm_load_print_meta: format           = GGUF V3 (latest)
llm_load_print_meta: arch             = qwen3moe
llm_load_print_meta: vocab type       = BPE
llm_load_print_meta: n_vocab          = 151936
llm_load_print_meta: n_merges         = 151387
llm_load_print_meta: vocab_only       = 0
llm_load_print_meta: n_ctx_train      = 262144
llm_load_print_meta: n_embd           = 6144
llm_load_print_meta: n_layer          = 62
llm_load_print_meta: n_head           = 96
llm_load_print_meta: n_head_kv        = 8
llm_load_print_meta: n_rot            = 128
llm_load_print_meta: n_swa            = 0
llm_load_print_meta: n_swa_pattern    = 1
llm_load_print_meta: n_embd_head_k    = 128
llm_load_print_meta: n_embd_head_v    = 128
llm_load_print_meta: n_gqa            = 12
llm_load_print_meta: n_embd_k_gqa     = 1024
llm_load_print_meta: n_embd_v_gqa     = 1024
llm_load_print_meta: f_norm_eps       = 0.0e+00
llm_load_print_meta: f_norm_rms_eps   = 1.0e-06
llm_load_print_meta: f_clamp_kqv      = 0.0e+00
llm_load_print_meta: f_max_alibi_bias = 0.0e+00
llm_load_print_meta: f_logit_scale    = 0.0e+00
llm_load_print_meta: n_ff             = 8192
llm_load_print_meta: n_expert         = 160
llm_load_print_meta: n_expert_used    = 8
llm_load_print_meta: causal attn      = 1
llm_load_print_meta: pooling type     = 0
llm_load_print_meta: rope type        = 2
llm_load_print_meta: rope scaling     = linear
llm_load_print_meta: freq_base_train  = 10000000.0
llm_load_print_meta: freq_scale_train = 1
llm_load_print_meta: n_ctx_orig_yarn  = 262144
llm_load_print_meta: rope_finetuned   = unknown
llm_load_print_meta: ssm_d_conv       = 0
llm_load_print_meta: ssm_d_inner      = 0
llm_load_print_meta: ssm_d_state      = 0
llm_load_print_meta: ssm_dt_rank      = 0
llm_load_print_meta: model type       = ?B
llm_load_print_meta: model ftype      = IQ2_KL - 2.6875 bpw
llm_load_print_meta: model params     = 480.155 B
llm_load_print_meta: model size       = 169.597 GiB (3.034 BPW) 
llm_load_print_meta: repeating layers = 168.388 GiB (3.024 BPW, 478.288 B parameters)
llm_load_print_meta: general.name     = Qwen3 Coder 480B A35B Instruct
llm_load_print_meta: BOS token        = 11 ','
llm_load_print_meta: EOS token        = 151645 '<|im_end|>'
llm_load_print_meta: PAD token        = 151643 '<|endoftext|>'
llm_load_print_meta: LF token         = 148848 'ÄĬ'
llm_load_print_meta: EOT token        = 151645 '<|im_end|>'
llm_load_print_meta: max token length = 256
llm_load_print_meta: n_ff_exp         = 2560
llm_load_tensors: ggml ctx size =    0.33 MiB
llm_load_tensors:        CPU buffer size = 173666.87 MiB
....................................................................................................
llama_new_context_with_model: n_ctx      = 196608
llama_new_context_with_model: n_batch    = 4096
llama_new_context_with_model: n_ubatch   = 4096
llama_new_context_with_model: flash_attn = 1
llama_new_context_with_model: mla_attn   = 0
llama_new_context_with_model: attn_max_b = 0
llama_new_context_with_model: fused_moe  = 1
llama_new_context_with_model: ser        = -1, 0
llama_new_context_with_model: freq_base  = 10000000.0
llama_new_context_with_model: freq_scale = 1
llama_kv_cache_init:        CPU KV buffer size = 25296.00 MiB
llama_new_context_with_model: KV self size  = 25296.00 MiB, K (q8_0): 12648.00 MiB, V (q8_0): 12648.00 MiB
llama_new_context_with_model:        CPU  output buffer size =     2.32 MiB
llama_new_context_with_model:        CPU compute buffer size =  5184.05 MiB
llama_new_context_with_model: graph nodes  = 2424
llama_new_context_with_model: graph splits = 1
/home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1146: /home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1146: /home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1146: /home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1146: GGML_ASSERT(fms.S[j] > 0) failed/home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1146: /home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1146: GGML_ASSERT(fms.S[j] > 0) failed
/home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1146: GGML_ASSERT(fms.S[j] > 0) failed
/home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1146: GGML_ASSERT(fms.S[j] > 0) failed
/home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1146: GGML_ASSERT(fms.S[j] > 0) failed
/home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1146: GGML_ASSERT(fms.S[j] > 0) failed
/home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1146: GGML_ASSERT(fms.S[j] > 0) failed
/home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1146: GGML_ASSERT(fms.S[j] > 0) failed
/home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1146: GGML_ASSERT(fms.S[j] > 0) failed
/home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1146: GGML_ASSERT(fms.S[j] > 0) failed
/home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1146: GGML_ASSERT(fms.S[j] > 0) failed
/home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1146: GGML_ASSERT(fms.S[j] > 0) failed
/home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1146: GGML_ASSERT(fms.S[j] > 0) failed
/home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1146: GGML_ASSERT(fms.S[j] > 0) failed
/home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1146: GGML_ASSERT(fms.S[j] > 0) failed
GGML_ASSERT(fms.S[j] > 0) failed
/home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1146: GGML_ASSERT(fms.S[j] > 0) failed
/home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1146: GGML_ASSERT(fms.S[j] > 0) failed
/home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1146: GGML_ASSERT(fms.S[j] > 0) failed
/home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1146: GGML_ASSERT(fms.S[j] > 0) failed
/home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1146: GGML_ASSERT(fms.S[j] > 0) failed
/home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1146: GGML_ASSERT(fms.S[j] > 0) failed
/home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1146: GGML_ASSERT(fms.S[j] > 0) failed
/home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1146: GGML_ASSERT(fms.S[j] > 0) failed
/home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1146: GGML_ASSERT(fms.S[j] > 0) failed
/home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1146: GGML_ASSERT(fms.S[j] > 0) failed
/home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1146: GGML_ASSERT(fms.S[j] > 0) failed
/home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1146: GGML_ASSERT(fms.S[j] > 0) failed
GGML_ASSERT(fms.S[j] > 0) failed
GGML_ASSERT(fms.S[j] > 0) failed

GGML_ASSERT(fms.S[j] > 0) failed
/home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1146: GGML_ASSERT(fms.S[j] > 0) failed
Could not attach to process.  If your uid matches the uid of the target
process, check the setting of /proc/sys/kernel/yama/ptrace_scope, or try
again as the root user.  For more details, see /etc/sysctl.d/10-ptrace.conf
ptrace: Inappropriate ioctl for device.
No stack.
The program is not being run.
Could not attach to process.  If your uid matches the uid of the target
process, check the setting of /proc/sys/kernel/yama/ptrace_scope, or try
again as the root user.  For more details, see /etc/sysctl.d/10-ptrace.conf
ptrace: Inappropriate ioctl for device.
No stack.
The program is not being run.
Could not attach to process.  If your uid matches the uid of the target
process, check the setting of /proc/sys/kernel/yama/ptrace_scope, or try
again as the root user.  For more details, see /etc/sysctl.d/10-ptrace.conf
ptrace: Inappropriate ioctl for device.
No stack.
The program is not being run.
Could not attach to process.  If your uid matches the uid of the target
process, check the setting of /proc/sys/kernel/yama/ptrace_scope, or try
again as the root user.  For more details, see /etc/sysctl.d/10-ptrace.conf
warning: process 4140403 is a zombie - the process has already terminated
ptrace: Inappropriate ioctl for device.
No stack.
The program is not being run.
./myscripts/api-server-Qwen3-Coder-480B-A35B-Instruct.sh: line 34: 4140403 Aborted                 (core dumped) numactl -N 1 -m 1 ./build/bin/llama-server --model "$model" --alias ubergarm/Qwen3-Coder-480B-A35B-Instruct --ctx-size 196608 -ctk q8_0 -ctv q8_0 -fa -fmoe -ub 4096 -b 4096 --parallel 3 --threads 128 --threads-batch 192 --numa numactl --host 127.0.0.1 --port 8080 --no-mmap
```

</details>


### Name and Version

$ ./build/bin/llama-server --version
version: 3822 (4e9c78c0)
built with cc (Ubuntu 13.3.0-6ubuntu2~24.04) 13.3.0 for x86_64-linux-gnu

### What operating system are you seeing the problem on?

Linux

### Relevant log output

```shell

```

---

## 💬 Conversation

👤 **dc740** commented on **2025-07-28** at **08:57:13**

I'm having the same issue with your Kimi quants and this was the first result in google.

```
numactl --interleave=all -- ./ik_llama.cpp/build/bin/llama-cli \
    --numa numactl  \
    --model models/ubergarm/Kimi-Dev-72B-GGUF/Kimi-Dev-72B-smol-IQ3_K.gguf \
    --threads -1 \
    --n-gpu-layers 99 \
    --no-mmap \
    --cache-type-k q4_1 \
    --cache-type-v q4_1 -fa \
    -ot ".ffn_(up|down)_exps.=CPU" \
    --temp 0.6 \
    --min_p 0.01 \
    -ser 5,1 \
    -amb 512 \
    -fmoe \
    -rtr \
    --ctx-size 16384 \
 --prompt "<｜User｜>Create a Flappy Bird game in Python.<｜Assistant｜>"
```

---

👤 **ubergarm** commented on **2025-07-28** at **19:41:07**

@dc740 

Heya that is the older dense Kimi-Dev-72B model and I don't think it is related to this issue. Please open a discussion on that model's hugging face and I'll help you workshop your command as I don't know where you got that command given that model has no routed experts psure. Please include your RAM, CPU, VRAM/GPU and OS information as well in that ticket over there (not here). thanks!

---

👤 **ubergarm** commented on **2025-07-31** at **20:44:04**

Got another assert when working on Qwen3-Coder-30B-A3B-Instruct today and couldn't release my `IQ3_K` but the rest of the series worked fine:  https://huggingface.co/ubergarm/Qwen3-Coder-30B-A3B-Instruct-GGUF

Just logging this here before I forget for completeness.

The imatrix had no missing data psure, though a few of the routed exps were not 100%.  Same `qk_fa_templates.h:1146: /home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1146: GGML_ASSERT(fms.S[j] > 0) failed`.


<details>

<summary>👈 Details</summary>

```bash
## recipe
#!/usr/bin/env bash

custom="
# 48 Repeating Layers [0-47]

# Attention
blk\.(0)\.attn_q.*=q8_0
blk\.(0)\.attn_k.*=q8_0
blk\.(0)\.attn_v.*=q8_0
blk\.(0)\.attn_output.*=q8_0

blk\..*\.attn_q.*=iq5_k
blk\..*\.attn_k.*=iq6_k
blk\..*\.attn_v.*=iq6_k
blk\..*\.attn_output.*=iq5_k

# Routed Experts
blk\.(0|47)\.ffn_down_exps\.weight=q8_0
blk\.(0|47)\.ffn_(gate|up)_exps\.weight=q8_0

blk\..*\.ffn_down_exps\.weight=iq4_k
blk\..*\.ffn_(gate|up)_exps\.weight=iq3_k

# Non-Repeating Layers
token_embd\.weight=iq4_k
output\.weight=iq6_k
"

custom=$(
  echo "$custom" | grep -v '^#' | \
  sed -Ez 's:\n+:,:g;s:,$::;s:^,::'
)

./build/bin/llama-quantize \
    --custom-q "$custom" \
    --imatrix /mnt/raid/models/ubergarm/Qwen3-Coder-30B-A3B-Instruct-GGUF/imatrix-Qwen3-Coder-30B-A3B-Instruct-BF16.dat \
    /mnt/raid/models/ubergarm/Qwen3-Coder-30B-A3B-Instruct-GGUF/Qwen3-Coder-30B-A3B-Instruct-BF16-00001-of-00002.gguf \
    /mnt/raid/models/ubergarm/Qwen3-Coder-30B-A3B-Instruct-GGUF/Qwen3-Coder-30B-A3B-Instruct-IQ3_K.gguf \
    IQ3_K \
    192


## perplexity test errors out

model=/mnt/raid/models/ubergarm/Qwen3-Coder-30B-A3B-Instruct-GGUF/Qwen3-Coder-30B-A3B-Instruct-IQ3_K.gguf

numactl -N 0 -m 0 \
    ./build/bin/llama-perplexity \
        -m "$model" \
        -f wiki.test.raw \
        --seed 1337 \
        -fa -fmoe \
        --ctx-size 512 \
        -ub 4096 -b 4096 \
        --numa numactl \
        --threads 128 \
        --threads-batch 192 \
        --no-mmap

llama_model_loader: - kv  34:                      quantize.imatrix.file str              = /mnt/raid/models/ubergarm/Qwen3-Coder...
llama_model_loader: - kv  35:                   quantize.imatrix.dataset str              = ubergarm-imatrix-calibration-corpus-v...
llama_model_loader: - kv  36:             quantize.imatrix.entries_count i32              = 385
llama_model_loader: - kv  37:              quantize.imatrix.chunks_count i32              = 840
llama_model_loader: - type  f32:  241 tensors
llama_model_loader: - type q8_0:   10 tensors
llama_model_loader: - type iq3_k:   92 tensors
llama_model_loader: - type iq4_k:   47 tensors
llama_model_loader: - type iq5_k:   94 tensors
llama_model_loader: - type iq6_k:   95 tensors
llm_load_vocab: special tokens cache size = 26
llm_load_vocab: token to piece cache size = 0.9311 MB
llm_load_print_meta: format           = GGUF V3 (latest)
llm_load_print_meta: arch             = qwen3moe
...

llm_load_print_meta: max token length = 256
llm_load_print_meta: n_ff_exp         = 768
llm_load_tensors: ggml ctx size =    0.25 MiB
llm_load_tensors:        CPU buffer size = 14857.44 MiB
................................................................................................
llama_new_context_with_model: n_ctx      = 4096
llama_new_context_with_model: n_batch    = 4096
llama_new_context_with_model: n_ubatch   = 4096
llama_new_context_with_model: flash_attn = 1
llama_new_context_with_model: mla_attn   = 0
llama_new_context_with_model: attn_max_b = 0
llama_new_context_with_model: fused_moe  = 1
llama_new_context_with_model: ser        = -1, 0
llama_new_context_with_model: freq_base  = 10000000.0
llama_new_context_with_model: freq_scale = 1
llama_kv_cache_init:        CPU KV buffer size =   384.00 MiB
llama_new_context_with_model: KV self size  =  384.00 MiB, K (f16):  192.00 MiB, V (f16):  192.00 MiB
llama_new_context_with_model:        CPU  output buffer size =     4.64 MiB
llama_new_context_with_model:        CPU compute buffer size =  2406.00 MiB
llama_new_context_with_model: graph nodes  = 1878
llama_new_context_with_model: graph splits = 1
/home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1146: /home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1146: /hom
e/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1146: GGML_ASSERT(fms.S[j] > 0) failed/home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/i
qk_fa_templates.h:1146:
GGML_ASSERT(fms.S[j] > 0) failed
/home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1146: /home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1146: GGML
_ASSERT(fms.S[j] > 0) failed
...
```

</details>

---

👤 **ikawrakow** commented on **2025-08-23** at **07:29:22**

Does this still happen on the latest main branch? There have been some fixes and changes since this issue was open, so it would be useful to reproduce.

---

👤 **ubergarm** commented on **2025-08-23** at **16:19:55**

I just tried again compiling CPU-only on that big AMD EPYC 9965 with version ik_llama.cpp at latest tip of `main@e008c0e1`

### Qwen3-Coder-480B-A35B-Instruct-IQ2_KL

Still failing with same assert on this one.

<details>

<summary>👈 Details</summary>

```bash
model=/mnt/raid/models/ubergarm/Qwen3-Coder-480B-A35B-Instruct-GGUF/Qwen3-Coder-480B-A35B-Instruct-og-busted-IQ2_KL.gguf

numactl -N 0 -m 0 \
./build/bin/llama-server \
    --model "$model" \
    --alias ubergarm/Qwen3-Coder-480B-A35B-Instruct \
    --ctx-size 131072 \
    -fa -fmoe \
    -ctk q8_0 -ctv q8_0 \
    -ub 4096 -b 4096 \
    --parallel 1 \
    --threads 128 \
    --threads-batch 192 \
    --numa numactl \
    --host 127.0.0.1 \
    --port 8080 \
    --no-mmap

llama_model_loader: - type  f32:  311 tensors
llama_model_loader: - type q8_0:  124 tensors
llama_model_loader: - type iq3_k:   62 tensors
llama_model_loader: - type iq4_k:    1 tensors
llama_model_loader: - type iq6_k:  125 tensors
llama_model_loader: - type iq2_kl:  124 tensors

llm_load_print_meta: model type       = ?B
llm_load_print_meta: model ftype      = IQ2_KL - 2.6875 bpw
llm_load_print_meta: model params     = 480.155 B
llm_load_print_meta: model size       = 169.597 GiB (3.034 BPW)
llm_load_print_meta: repeating layers = 168.388 GiB (3.024 BPW, 478.288 B parameters)
llm_load_print_meta: general.name     = Qwen3 Coder 480B A35B Instruct
llm_load_print_meta: n_ff_exp         = 2560

llm_load_tensors: ggml ctx size =    0.33 MiB
llm_load_tensors:        CPU buffer size = 173666.87 MiB
....................................................................................................
llama_new_context_with_model: n_ctx      = 131072
llama_new_context_with_model: n_batch    = 4096
llama_new_context_with_model: n_ubatch   = 4096
llama_new_context_with_model: flash_attn = 1
llama_new_context_with_model: mla_attn   = 0
llama_new_context_with_model: attn_max_b = 0
llama_new_context_with_model: fused_moe  = 1
llama_new_context_with_model: ser        = -1, 0
llama_new_context_with_model: freq_base  = 10000000.0
llama_new_context_with_model: freq_scale = 1
llama_kv_cache_init:        CPU KV buffer size = 16864.00 MiB
llama_new_context_with_model: KV self size  = 16864.00 MiB, K (q8_0): 8432.00 MiB, V (q8_0): 8432.00 MiB
llama_new_context_with_model:        CPU  output buffer size =     1.16 MiB
llama_new_context_with_model:        CPU compute buffer size =  3648.05 MiB
llama_new_context_with_model: graph nodes  = 2424
llama_new_context_with_model: graph splits = 1
======================================= HAVE_FANCY_SIMD is defined
/home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1161: /home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1161: /hom
e/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1161: /home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1161: GGML_ASS
ERT(S > 0) failed
/home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1161: GGML_ASSERT(S > 0) failed
```

</details>

### Qwen3-Coder-30B-A3B-Instruct-IQ3_K


Still failing with same assert on this one.

<details>

<summary>👈 Details</summary>

```bash
model=/mnt/raid/models/ubergarm/Qwen3-Coder-30B-A3B-Instruct-GGUF/Qwen3-Coder-30B-A3B-Instruct-IQ3_K.gguf

numactl -N 1 -m 1 \
./build/bin/llama-server \
    --model "$model"\
    --alias ubergarm/Qwen3-Coder-30B-A3B-Instruct \
    --ctx-size 196608 \
    -ctk q8_0 -ctv q8_0 \
    -fa -fmoe \
    --parallel 1 \
    --threads 128 \
    --threads-batch 192 \
    --numa numactl \
    --host 127.0.0.1 \
    --port 8080 \
    --no-mmap

llama_new_context_with_model: freq_scale = 1
llama_kv_cache_init:        CPU KV buffer size =  9792.00 MiB
llama_new_context_with_model: KV self size  = 9792.00 MiB, K (q8_0): 4896.00 MiB, V (q8_0): 4896.00 MiB
llama_new_context_with_model:        CPU  output buffer size =     1.16 MiB
llama_new_context_with_model:        CPU compute buffer size =   600.01 MiB
llama_new_context_with_model: graph nodes  = 1878
llama_new_context_with_model: graph splits = 1
======================================= HAVE_FANCY_SIMD is defined
/home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1161: /home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1161: /home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1161: GGML_ASSERT(S > 0) failed 
/home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1161: GGML_ASSERT(S > 0) failed
/home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1161: GGML_ASSERT(S > 0) failed/home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1161: GGML_ASSERT(S > 0) failed 
GGML_ASSERT(S > 0) failed
/home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1161: GGML_ASSERT(S > 0) failed
```

</details> 

---

### DeepSeek-V3.1-IQ4_K
### DeepSeek-V3.1-IQ4_KS
These two new ones I cooked today are failing now too for `llama-perplexity` but not for `llama-server`. I rolled back a couple commits and they seem to run okay on `main@9351cc34`. 

Further testing suggests it is specifically [#722](https://github.com/ikawrakow/ik_llama.cpp/issues/722) that is causing this somehow? This is likely a *different* issue than the original, sorry for tagging it in here as it came up while exploring the other stuff.

<details>

<summary>👈 Details</summary>

```bash
#model=/mnt/raid/models/ubergarm/DeepSeek-V3.1-GGUF/DeepSeek-V3.1-IQ4_KS.gguf
model=/mnt/raid/models/ubergarm/DeepSeek-V3.1-GGUF/DeepSeek-V3.1-IQ4_K.gguf

SOCKET=1

numactl -N "$SOCKET" -m "$SOCKET" \
./build/bin/llama-perplexity \
    -m "$model" \
    -f wiki.test.raw \
    --seed 1337 \
    -fa -fmoe \
    -mla 3 \
    --ctx-size 512 \
    -ub 4096 -b 4096 \
    --numa numactl \
    --threads 128 \
    --threads-batch 192 \
    --no-mmap

llm_load_print_meta: model type       = 671B
llm_load_print_meta: model ftype      = IQ4_K - 4.5 bpw
llm_load_print_meta: model params     = 671.026 B
llm_load_print_meta: model size       = 384.765 GiB (4.925 BPW)
llm_load_print_meta: repeating layers = 383.336 GiB (4.921 BPW, 669.173 B parameters)
llm_load_print_meta: general.name     = DeepSeek V3.1 Bf16 Safetensors

system_info: n_threads = 128 (n_threads_batch = 192) / 768 | AVX = 1 | AVX_VNNI = 1 | AVX2 = 1 | AVX512 = 1 | AVX512_VBMI = 1 | AVX512_VNNI = 1 | AVX5
12_BF16 = 1 | FMA = 1 | NEON = 0 | SVE = 0 | ARM_FMA = 0 | F16C = 1 | FP16_VA = 0 | WASM_SIMD = 0 | BLAS = 0 | SSE3 = 1 | SSSE3 = 1 | VSX = 0 | MATMUL
_INT8 = 0 | LLAMAFILE = 1 |
perplexity: tokenizing the input ..
perplexity: tokenization took 493.281 ms
perplexity: calculating perplexity over 561 chunks, n_ctx=512, batch_size=2048, n_seq=4
ttn_kv_b.weight as 512 x 32768 and stored in buffer CPU
Computed blk.56.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
Computed blk.57.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
Computed blk.58.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
Computed blk.59.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
Computed blk.60.attn_kv_b.weight as 512 x 32768 and stored in buffer CPU
======================================= HAVE_FANCY_SIMD is defined
/home/w/projects/ik_llama.cpp/ggml/src/iqk/iqk_gemm_legacy_quants.cpp:1774: GGML_ASSERT(nrc_x%8 == 0) failed/home/w/projects/ik_llama.cpp/ggml/src/iqk
/iqk_gemm_legacy_quants.cpp:1774: GGML_ASSERT(nrc_x%8 == 0) failed
/home/w/projects/ik_llama.cpp/ggml/src/iqk/iqk_gemm_legacy_quants.cpp:1774: GGML_ASSERT(nrc_x%8 == 0) failed/home/w/projects/ik_llama.cpp/ggml/src/iqk
/iqk_gemm_legacy_quants.cpp:1774: /home/w/projects/ik_llama.cpp/ggml/src/iqk/iqk_gemm_legacy_quants.cpp:1774: /home/w/projects/ik_llama.cpp/ggml/src/i
qk/iqk_gemm_legacy_quants.cpp:1774: /home/w/projects/ik_llama.cpp/ggml/src/iqk/iqk_gemm_legacy_quants.cpp:1774: /home/w/projects/ik_llama.cpp/ggml/src
/iqk/iqk_gemm_legacy_quants.cpp:1774: /home/w/projects/ik_llama.cpp/ggml/src/iqk/iqk_gemm_legacy_quants.cpp:1774: /home/w/projects/ik_llama.cpp/ggml/s
rc/iqk/iqk_gemm_legacy_quants.cpp:1774: GGML_ASSERT(nrc_x%8 == 0) failed/home/w/projects/ik_llama.cpp/ggml/src/iqk/iqk_gemm_legacy_quants.cpp:1774: /h
ome/w/projects/ik_llama.cpp/ggml/src/iqk/iqk_gemm_legacy_quants.cpp:1774: GGML_ASSERT(nrc_x%8 == 0) failed/home/w/projects/ik_llama.cpp/ggml/src/iqk/i
qk_gemm_legacy_quants.cpp:1774: /home/w/projects/ik_llama.cpp/ggml/src/iqk/iqk_gemm_legacy_quants.cpp:1774: /home/w/projects/ik_llama.cpp/ggml/src/iqk
/iqk_gemm_legacy_quants.cpp:1774: /home/w/projects/ik_llama.cpp/ggml/src/iqk/iqk_gemm_legacy_quants.cpp:1774: /home/w/projects/ik_llama.cpp/ggml/src/i
qk/iqk_gemm_legacy_quants.cpp:1774: /home/w/projects/ik_llama.cpp/ggml/src/iqk/iqk_gemm_legacy_quants.cpp:1774: /home/w/projects/ik_llama.cpp/ggml/src
/iqk/iqk_gemm_legacy_quants.cpp:1774: /home/w/projects/ik_llama.cpp/ggml/src/iqk/iqk_gemm_legacy_quants.cpp:1774: GGML_ASSERT(nrc_x%8 == 0) failedGGML
_ASSERT(nrc_x%8 == 0) failedGGML_ASSERT(nrc_x%8 == 0) failed/home/w/projects/ik_llama.cpp/ggml/src/iqk/iqk_gemm_legacy_quants.cpp:1774: GGML_ASSERT(nr
c_x%8 == 0) failed
/home/w/projects/ik_llama.cpp/ggml/src/iqk/iqk_gemm_legacy_quants.cpp:1774: GGML_ASSERT(nrc_x%8 == 0) failed
/home/w/projects/ik_llama.cpp/ggml/src/iqk/iqk_gemm_legacy_quants.cpp:1774: GGML_ASSERT(nrc_x%8 == 0) failed
```

</details>

I'll take a look at the KT specific stuff next.

---

👤 **ikawrakow** commented on **2025-08-24** at **05:31:03**

@ubergarm  Still failing with [#724](https://github.com/ikawrakow/ik_llama.cpp/issues/724) ? If so, I would need to get the Qwen3-Coder-30B-A3B-Instruct-IQ3_K model, which I did not find published in you HF repository.

---

👤 **ubergarm** commented on **2025-08-24** at **15:58:56**

I just tried again compiling CPU-only on that big AMD EPYC 9965 with ik_llama.cpp running PR724 `ik/fix_avx2_gemm_mess@04740cae`.

### Qwen3-Coder-480B-A35B-Instruct-IQ2_KL

Still failing on this one, the line number changed slightly:

```bash
======================================= HAVE_FANCY_SIMD is defined
/home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1156: /home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1156: GGML
_ASSERT(S > 0) failed
/home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1156: GGML_ASSERT(S > 0) failed
/home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1156: GGML_ASSERT(S > 0) failed
/home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1156: GGML_ASSERT(S > 0) failed
```

### Qwen3-Coder-30B-A3B-Instruct-IQ3_K

Still failing here too.

```
llama_new_context_with_model: graph splits = 1
======================================= HAVE_FANCY_SIMD is defined
/home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1156: /home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1156: /home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1156: GGML_ASSERT(S > 0) failed
/home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1156: GGML_ASSERT(S > 0) failed
/home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1156: GGML_ASSERT(S > 0) failed
```

So went ahead and upload the "broken" [Qwen3-Coder-30B-A3B-Instruct-IQ3_K here](https://huggingface.co/ubergarm/Qwen3-Coder-30B-A3B-Instruct-GGUF/blob/main/Qwen3-Coder-30B-A3B-Instruct-IQ3_K.gguf) if you would like to test locally. I sure hope it isn't these two files are corrupt or anything, but the others created in the same way were working fine.

---

👤 **ikawrakow** commented on **2025-08-24** at **16:09:09**

> So went ahead and upload the "broken" [Qwen3-Coder-30B-A3B-Instruct-IQ3_K here](https://huggingface.co/ubergarm/Qwen3-Coder-30B-A3B-Instruct-GGUF/blob/main/Qwen3-Coder-30B-A3B-Instruct-IQ3_K.gguf)

Downloading now, but somehow getting just 3 MB/s. Is something up with HF?

---

👤 **ubergarm** commented on **2025-08-24** at **23:54:03**

> Is something up with HF?

I've heard various complaints about "xet" but I don't think I have that enabled.. i definitely have issues downloading larger quants with the `hf` cli command timing out occasionally... for small single file quants i typically just use `wget ...` 

thanks for looking at this one!

---

👤 **ikawrakow** commented on **2025-08-25** at **08:35:50**

@ubergarm 

There are indeed NaNs in [Qwen3-Coder-30B-A3B-Instruct-IQ3_K](https://huggingface.co/ubergarm/Qwen3-Coder-30B-A3B-Instruct-GGUF/blob/main/Qwen3-Coder-30B-A3B-Instruct-IQ3_K.gguf). 3200 NaN block scales in `blk.1.ffn_gate_exps.weight` and 3200 NaN block scales in `blk.1.ffn_up_exps.weight` to be more precise.

You can use PR [#727](https://github.com/ikawrakow/ik_llama.cpp/issues/727) to check this model and all the others where people have reported issues.
Just run any command that loads a model CPU-only (the check for NaNs is done only for tensors stored in RAM, so CPU-only will check all tensors, while a CUDA build will check only tensors set to CPU via `-ot`).

I did check the sha256 checksum, and it matches the checksum posted in the repository, so the file was not corrupted when downloading.

In another thread you mentioned that you are checking all of your models before publishing, so I was wondering how this one could have produced a PPL value that is not NaN. So I ran a perplexity calculation on the GPU and it was fine. Somehow when running Wikitext2 perplexity on the GPU never activates the experts containing NaNs, but for some reason these experts are triggered when running on the CPU (or hybrid, but the op was not offloaded to the GPU). 

**Update** I refined my NaN checking a bit, so all NaNs in both tensors are in expert 27.

---

👤 **ubergarm** commented on **2025-08-25** at **15:51:18**

Okay, I just used PR727 and confirmed that NaN blocks exist in *both* of the models for which I had trouble:

## Qwen3-Coder-480B-A35B-Instruct-IQ2_KL
```bash
llm_load_tensors: ggml ctx size =    0.33 MiB
llm_load_tensors:        CPU buffer size = 173666.87 MiB
....................................................................................................
check_tensor_for_blocks_256_fp16: found 61440 NaN block scales out of 9830400 blocks in tensor blk.1.ffn_down_exps.weight
    there are 12288 NaN block scales for expert 20
    there are 6144 NaN block scales for expert 84
    there are 36864 NaN block scales for expert 97
    there are 6144 NaN block scales for expert 129
check_tensor_for_blocks_256_fp16: found 6144 NaN block scales out of 9830400 blocks in tensor blk.2.ffn_down_exps.weight
    there are 6144 NaN block scales for expert 100
check_tensor_for_blocks_256_fp16: found 36864 NaN block scales out of 9830400 blocks in tensor blk.7.ffn_down_exps.weight
    there are 6144 NaN block scales for expert 37
    there are 12288 NaN block scales for expert 54
    there are 18432 NaN block scales for expert 146
check_tensor_for_blocks_256_fp16: found 36864 NaN block scales out of 9830400 blocks in tensor blk.8.ffn_down_exps.weight
.
.
.
# it had 1000+ lines of NaN blocks...
```

Just to be sure, I tried it on the similar `Qwen3-Coder-480B-A35B-Instruct-IQ3_K` and it is fine with no NaN blocks detected... So not sure why the IQ2_KL is busted...

## Qwen3-Coder-30B-A3B-Instruct-IQ3_K
```bash
llm_load_tensors: ggml ctx size =    0.25 MiB
llm_load_tensors:        CPU buffer size = 14857.44 MiB
................................................................................................
check_tensor_for_blocks_256_fp16: found 3200 NaN block scales out of 786432 blocks in tensor blk.1.ffn_gate_exps.weight
    there are 3200 NaN block scales for expert 27
check_tensor_for_blocks_256_fp16: found 3200 NaN block scales out of 786432 blocks in tensor blk.1.ffn_up_exps.weight
    there are 3200 NaN block scales for expert 27
Found 2 bad tensors in model
llama_model_load: error loading model: Bad tensors in model
llama_load_model_from_file: failed to load model
llama_init_from_gpt_params: error: failed to load model '/mnt/raid/models/ubergarm/Qwen3-Coder-30B-A3B-Instruct-GGUF/Qwen3-Coder-30B-A3B-Instruct-IQ3_K.gguf'
 ERR [              load_model] unable to load model | tid="129560015714496" timestamp=1756136000 model="/mnt/raid/models/ubergarm/Qwen3-Coder-30B-A3B-Instruct-GGUF/Qwen3-Coder-30B-A3B-Instruct-IQ3_K.gguf"
```

> In another thread you mentioned that you are checking all of your models before publishing, so I was wondering how this one could have produced a PPL value that is not NaN.

Correct, I never released either of these two models as I was running `llama-perplexity` on CPU backend and so it threw that error:
```
/home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1146: /home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1146: /hom
e/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1146: GGML_ASSERT(fms.S[j] > 0) failed/home/w/projects/ik_llama.cpp/ggml/src/./iqk/fa/i
qk_fa_templates.h:1146:
GGML_ASSERT(fms.S[j] > 0) failed
```

I went ahead and [deleted the test quant uploaded to hf](https://huggingface.co/ubergarm/Qwen3-Coder-30B-A3B-Instruct-GGUF/commit/26aa6cfbebfa513bc49fdba169840c67b219e152) as it indeed seems bugged with the NaN blocks.

> So I ran a perplexity calculation on the GPU and it was fine. Somehow when running Wikitext2 perplexity on the GPU never activates the experts containing NaNs, but for some reason these experts are triggered when running on the CPU (or hybrid, but the op was not offloaded to the GPU).

Huh, that is interesting and a little spooky haha... 😅

So having PR727 with an option e.g. `--check-tensors` defaulting to off could help prevent a situation like this. I could run that on each quant before releasing to make sure there are no NaN blocks and not worry about GPU/CPU backend differences possibly missing something.

Thanks!

---

👤 **ikawrakow** commented on **2025-08-25** at **15:54:56**

> So having PR727 with an option e.g. --check-tensors defaulting to off could help prevent a situation like this. I could run that on each quant before releasing to make sure there are no NaN blocks and not worry about GPU/CPU backend differences possibly missing something.

Yes, I'll polish [#727](https://github.com/ikawrakow/ik_llama.cpp/issues/727), and will also hide the check behind a command line option. But I think I need to take more seriously the possibility of NaNs in the quantization routines, and prevent having NaNs in the first place.

---

👤 **ikawrakow** commented on **2025-08-27** at **15:32:32**

This should also be solved now (assuming a new model is generated that does not contain NaNs and the latest main branch is used)

---

👤 **ubergarm** commented on **2025-08-28** at **14:32:06**

Yes, this is now complete. The two models in question have been re-quantized, validated to have no NaNs, and released onto huggingface now! Thanks!