## 📌 [Issue #895](https://github.com/ikawrakow/ik_llama.cpp/issues/895) - Bug: GPT-OSS 120b CUDA error: an illegal memory access was encountered

| **Author** | `YurkoHoshko` |
| :--- | :--- |
| **State** | ✅ **Open** |
| **Created** | 2025-11-04 |
| **Updated** | 2025-11-06 |

---

## 📄 Description

### What happened?

Unable to run [GPT-OSS 120B](https://huggingface.co/unsloth/gpt-oss-120b-GGUF?show_file_info=gpt-oss-120b-F16.gguf), even though I was able to do it few month ago. 
Confirmed that GPT-OSS 20B runs just fine.

I have built and tried it with the latest `main`.

Hardware: AMD Ryzen 9 8945HS, 96GB DDR5, RTX 5060TI 16GB, RXT 3060 12GB (not a part of the bench)

Command:

```
/ik_llama.cpp/build/bin/llama-sweep-bench -m /models/gpt-oss-120b.gguf -ngl 99  -c 16768  --temp 1 --top-p 1.0 --top-k 0 -rtr --cpu-moe

....
CUDA error: an illegal memory access was encountered
  current device: 0, in function ggml_backend_cuda_synchronize at /ik_llama.cpp/ggml/src/ggml-cuda.cu:3539
  cudaStreamSynchronize(cuda_ctx->stream())
/ik_llama.cpp/ggml/src/ggml-cuda.cu:122: CUDA error
```




### Name and Version

root@27aa253f2c92:/# /ik_llama.cpp/build/bin/llama-cli --version
version: 3947 (cd8d0b08)
built with cc (Ubuntu 13.3.0-6ubuntu2~24.04) 13.3.0 for x86_64-linux-gnu

### What operating system are you seeing the problem on?

Linux

### Relevant log output

```shell
root@27aa253f2c92:/# CUDA_VISIBLE_DEVICES=0 cuda-gdb  --args /ik_llama.cpp/build/bin/llama-sweep-bench -m /models/gpt-oss-120b.gguf -ngl 99  -c 16768  --temp 1 --top-p 1.0 --top-k 0 -rtr --cpu-moe
NVIDIA (R) cuda-gdb 12.8
Portions Copyright (C) 2007-2024 NVIDIA Corporation
Based on GNU gdb 13.2
Copyright (C) 2023 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Type "show copying" and "show warranty" for details.
This CUDA-GDB was configured as "x86_64-pc-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<https://forums.developer.nvidia.com/c/developer-tools/cuda-developer-tools/cuda-gdb>.
Find the CUDA-GDB manual and other documentation resources online at:
    <https://docs.nvidia.com/cuda/cuda-gdb/index.html>.

For help, type "help".
Type "apropos word" to search for commands related to "word"...
Reading symbols from /ik_llama.cpp/build/bin/llama-sweep-bench...
(No debugging symbols found in /ik_llama.cpp/build/bin/llama-sweep-bench)
(cuda-gdb) run
Starting program: /ik_llama.cpp/build/bin/llama-sweep-bench -m /models/gpt-oss-120b.gguf -ngl 99 -c 16768 --temp 1 --top-p 1.0 --top-k 0 -rtr --cpu-moe
warning: Error disabling address space randomization: Operation not permitted
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".
[New Thread 0x7fe2521ff000 (LWP 509)]
[New Thread 0x7fe250f2e000 (LWP 510)]
[Detaching after fork from child process 511]
ggml_cuda_init: GGML_CUDA_FORCE_MMQ:    no
ggml_cuda_init: GGML_CUDA_FORCE_CUBLAS: no
ggml_cuda_init: found 1 CUDA devices:
  Device 0: NVIDIA GeForce RTX 5060 Ti, compute capability 12.0, VMM: yes
[New Thread 0x7fe24adde000 (LWP 519)]
[New Thread 0x7fe24a5dd000 (LWP 520)]
CUDA0: using device CUDA0 - 10796 MiB free
llama_model_loader: loaded meta data with 37 key-value pairs and 687 tensors from /models/gpt-oss-120b.gguf (version GGUF V3 (latest))
llama_model_loader: Dumping metadata keys/values. Note: KV overrides do not apply in this output.
llama_model_loader: - kv   0:                       general.architecture str              = gpt-oss
llama_model_loader: - kv   1:                               general.type str              = model
llama_model_loader: - kv   2:                               general.name str              = Gpt-Oss-120B
llama_model_loader: - kv   3:                           general.basename str              = Gpt-Oss-120B
llama_model_loader: - kv   4:                       general.quantized_by str              = Unsloth
llama_model_loader: - kv   5:                         general.size_label str              = 120B
llama_model_loader: - kv   6:                            general.license str              = apache-2.0
llama_model_loader: - kv   7:                           general.repo_url str              = https://huggingface.co/unsloth
llama_model_loader: - kv   8:                               general.tags arr[str,2]       = ["vllm", "text-generation"]
llama_model_loader: - kv   9:                        gpt-oss.block_count u32              = 36
llama_model_loader: - kv  10:                     gpt-oss.context_length u32              = 131072
llama_model_loader: - kv  11:                   gpt-oss.embedding_length u32              = 2880
llama_model_loader: - kv  12:                gpt-oss.feed_forward_length u32              = 2880
llama_model_loader: - kv  13:               gpt-oss.attention.head_count u32              = 64
llama_model_loader: - kv  14:            gpt-oss.attention.head_count_kv u32              = 8
llama_model_loader: - kv  15:                     gpt-oss.rope.freq_base f32              = 150000.000000
llama_model_loader: - kv  16:   gpt-oss.attention.layer_norm_rms_epsilon f32              = 0.000010
llama_model_loader: - kv  17:                       gpt-oss.expert_count u32              = 128
llama_model_loader: - kv  18:                  gpt-oss.expert_used_count u32              = 4
llama_model_loader: - kv  19:               gpt-oss.attention.key_length u32              = 64
llama_model_loader: - kv  20:             gpt-oss.attention.value_length u32              = 64
llama_model_loader: - kv  21:                          general.file_type u32              = 1
llama_model_loader: - kv  22:           gpt-oss.attention.sliding_window u32              = 128
llama_model_loader: - kv  23:         gpt-oss.expert_feed_forward_length u32              = 2880
llama_model_loader: - kv  24:                  gpt-oss.rope.scaling.type str              = yarn
llama_model_loader: - kv  25:                gpt-oss.rope.scaling.factor f32              = 32.000000
llama_model_loader: - kv  26: gpt-oss.rope.scaling.original_context_length u32              = 4096
llama_model_loader: - kv  27:               general.quantization_version u32              = 2
llama_model_loader: - kv  28:                       tokenizer.ggml.model str              = gpt2
llama_model_loader: - kv  29:                         tokenizer.ggml.pre str              = gpt-4o
llama_model_loader: - kv  30:                      tokenizer.ggml.tokens arr[str,201088]  = ["!", "\"", "#", "$", "%", "&", "'", ...
llama_model_loader: - kv  31:                  tokenizer.ggml.token_type arr[i32,201088]  = [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, ...
llama_model_loader: - kv  32:                      tokenizer.ggml.merges arr[str,446189]  = ["Ġ Ġ", "Ġ ĠĠĠ", "ĠĠ ĠĠ", "...
llama_model_loader: - kv  33:                tokenizer.ggml.bos_token_id u32              = 199998
llama_model_loader: - kv  34:                tokenizer.ggml.eos_token_id u32              = 200002
llama_model_loader: - kv  35:            tokenizer.ggml.padding_token_id u32              = 200017
llama_model_loader: - kv  36:                    tokenizer.chat_template str              = {# Chat template fixes by Unsloth #}\n...
llama_model_loader: - type  f32:  433 tensors
llama_model_loader: - type  f16:  146 tensors
llama_model_loader: - type mxfp4:  108 tensors
load: printing all EOG tokens:
load:   - 199999 ('<|endoftext|>')
load:   - 200002 ('<|return|>')
load:   - 200007 ('<|end|>')
load:   - 200012 ('<|call|>')
load: special_eog_ids contains both '<|return|>' and '<|call|>' tokens, removing '<|end|>' token from EOG list
load: special tokens cache size = 21
load: token to piece cache size = 1.3332 MB
llm_load_print_meta: format           = GGUF V3 (latest)
llm_load_print_meta: arch             = gpt-oss
llm_load_print_meta: n_ctx_train      = 131072
llm_load_print_meta: n_embd           = 2880
llm_load_print_meta: n_layer          = 36
llm_load_print_meta: n_head           = 64
llm_load_print_meta: n_head_kv        = 8
llm_load_print_meta: n_rot            = 64
llm_load_print_meta: n_swa            = 128
llm_load_print_meta: n_swa_pattern    = 1
llm_load_print_meta: n_embd_head_k    = 64
llm_load_print_meta: n_embd_head_v    = 64
llm_load_print_meta: n_gqa            = 8
llm_load_print_meta: n_embd_k_gqa     = 512
llm_load_print_meta: n_embd_v_gqa     = 512
llm_load_print_meta: f_norm_eps       = 0.0e+00
llm_load_print_meta: f_norm_rms_eps   = 1.0e-05
llm_load_print_meta: f_clamp_kqv      = 0.0e+00
llm_load_print_meta: f_max_alibi_bias = 0.0e+00
llm_load_print_meta: f_logit_scale    = 0.0e+00
llm_load_print_meta: n_ff             = 2880
llm_load_print_meta: n_expert         = 128
llm_load_print_meta: n_expert_used    = 4
llm_load_print_meta: causal attn      = 1
llm_load_print_meta: pooling type     = 0
llm_load_print_meta: rope type        = 2
llm_load_print_meta: rope scaling     = yarn
llm_load_print_meta: freq_base_train  = 150000.0
llm_load_print_meta: freq_scale_train = 0.03125
llm_load_print_meta: n_ctx_orig_yarn  = 4096
llm_load_print_meta: rope_finetuned   = unknown
llm_load_print_meta: ssm_d_conv       = 0
llm_load_print_meta: ssm_d_inner      = 0
llm_load_print_meta: ssm_d_state      = 0
llm_load_print_meta: ssm_dt_rank      = 0
llm_load_print_meta: model type       = ?B
llm_load_print_meta: model ftype      = F16
llm_load_print_meta: model params     = 116.829 B
llm_load_print_meta: model size       = 60.868 GiB (4.475 BPW)
llm_load_print_meta: repeating layers = 58.710 GiB (4.360 BPW, 115.671 B parameters)
llm_load_print_meta: general.name     = Gpt-Oss-120B
llm_load_print_meta: n_ff_exp         = 2880
print_info: vocab type       = BPE
print_info: n_vocab          = 201088
print_info: n_merges         = 446189
print_info: BOS token        = 199998 '<|startoftext|>'
print_info: EOS token        = 200002 '<|return|>'
print_info: EOT token        = 199999 '<|endoftext|>'
print_info: PAD token        = 200017 '<|reserved_200017|>'
print_info: LF token         = 198 'Ċ'
print_info: EOG token        = 199999 '<|endoftext|>'
print_info: EOG token        = 200002 '<|return|>'
print_info: EOG token        = 200012 '<|call|>'
print_info: max token length = 256
llm_load_tensors: ggml ctx size =    0.56 MiB
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
llm_load_tensors: offloading 36 repeating layers to GPU
llm_load_tensors: offloading non-repeating layers to GPU
llm_load_tensors: offloaded 37/37 layers to GPU
llm_load_tensors:        CPU buffer size = 58244.06 MiB
llm_load_tensors:  CUDA_Host buffer size =  1104.61 MiB
llm_load_tensors:      CUDA0 buffer size =  3131.54 MiB
...................................................................................................
llama_new_context_with_model: n_ctx         = 16896
llama_new_context_with_model: n_batch       = 2048
llama_new_context_with_model: n_ubatch      = 512
llama_new_context_with_model: flash_attn    = 1
llama_new_context_with_model: mla_attn      = 0
llama_new_context_with_model: attn_max_b    = 0
llama_new_context_with_model: fused_moe     = 1
llama_new_context_with_model: grouped er    = 0
llama_new_context_with_model: fused_up_gate = 1
llama_new_context_with_model: fused_mmad    = 1
llama_new_context_with_model: rope_cache    = 0
llama_new_context_with_model: ser           = -1, 0
llama_new_context_with_model: freq_base     = 150000.0
llama_new_context_with_model: freq_scale    = 0.03125
llama_kv_cache_init:      CUDA0 KV buffer size =  1188.00 MiB
llama_new_context_with_model: KV self size  = 1188.00 MiB, K (f16):  594.00 MiB, V (f16):  594.00 MiB
llama_new_context_with_model:  CUDA_Host  output buffer size =     0.77 MiB
llama_new_context_with_model:      CUDA0 compute buffer size =   404.00 MiB
llama_new_context_with_model:  CUDA_Host compute buffer size =    78.01 MiB
llama_new_context_with_model: graph nodes  = 1409
llama_new_context_with_model: graph splits = 74
XXXXXXXXXXXXXXXXXXXXX Setting only active experts offload

main: n_kv_max = 16896, n_batch = 2048, n_ubatch = 512, flash_attn = 1, n_gpu_layers = 99, n_threads = 8, n_threads_batch = 8

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|

CUDA Exception: Warp Out-of-range Address
The exception was triggered at PC 0x7fe03b77f8f0  void k_openai_f32_f32_i32<(ggml_sort_order)1>(float const*, float*, int*, int, int, int, unsigned long)

Thread 1 "llama-sweep-ben" received signal CUDA_EXCEPTION_5, Warp Out-of-range Address.
[Switching focus to CUDA kernel 0, grid 20, block (0,0,0), thread (32,0,0), device 0, sm 0, warp 0, lane 0]
0x00007fe03b77f930 in void k_openai_f32_f32_i32<(ggml_sort_order)1>(float const*, float*, int*, int, int, int, unsigned long)<<<(1,1,1),(128,1,1)>>> ()
```

---

## 💬 Conversation

👤 **ikawrakow** commented on **2025-11-04** at **07:56:47**

This is the latest main branch?

With the same command it works foe me, and here is what I get (Ryzen-5975WX + RTX-4080)

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|   512 |    128 |      0 |    1.334 |   383.74 |    4.848 |    26.40 |
|   512 |    128 |    512 |    1.318 |   388.60 |    4.777 |    26.80 |
|   512 |    128 |   1024 |    1.327 |   385.93 |    4.783 |    26.76 |
|   512 |    128 |   1536 |    1.329 |   385.30 |    4.789 |    26.73 |
|   512 |    128 |   2048 |    1.321 |   387.57 |    4.790 |    26.72 |
|   512 |    128 |   2560 |    1.332 |   384.31 |    4.799 |    26.67 |
|   512 |    128 |   3072 |    1.336 |   383.36 |    4.797 |    26.68 |
|   512 |    128 |   3584 |    1.322 |   387.16 |    4.806 |    26.63 |
|   512 |    128 |   4096 |    1.332 |   384.49 |    4.808 |    26.62 |
|   512 |    128 |   4608 |    1.332 |   384.34 |    4.809 |    26.61 |
|   512 |    128 |   5120 |    1.346 |   380.36 |    4.823 |    26.54 |
|   512 |    128 |   5632 |    1.334 |   383.80 |    4.817 |    26.57 |
|   512 |    128 |   6144 |    1.334 |   383.72 |    4.830 |    26.50 |
|   512 |    128 |   6656 |    1.330 |   385.05 |    4.828 |    26.51 |
|   512 |    128 |   7168 |    1.335 |   383.39 |    4.836 |    26.47 |
|   512 |    128 |   7680 |    1.337 |   382.81 |    4.846 |    26.41 |
|   512 |    128 |   8192 |    1.333 |   384.22 |    4.843 |    26.43 |
|   512 |    128 |   8704 |    1.339 |   382.51 |    4.848 |    26.40 |
|   512 |    128 |   9216 |    1.339 |   382.51 |    4.850 |    26.39 |
|   512 |    128 |   9728 |    1.340 |   381.97 |    4.861 |    26.33 |
|   512 |    128 |  10240 |    1.335 |   383.44 |    4.864 |    26.32 |
|   512 |    128 |  10752 |    1.340 |   381.96 |    4.866 |    26.30 |
|   512 |    128 |  11264 |    1.346 |   380.43 |    4.867 |    26.30 |
|   512 |    128 |  11776 |    1.335 |   383.39 |    4.876 |    26.25 |
|   512 |    128 |  12288 |    1.348 |   379.83 |    4.886 |    26.20 |
|   512 |    128 |  12800 |    1.343 |   381.27 |    4.877 |    26.25 |
|   512 |    128 |  13312 |    1.345 |   380.61 |    4.889 |    26.18 |
|   512 |    128 |  13824 |    1.343 |   381.23 |    4.889 |    26.18 |
|   512 |    128 |  14336 |    1.348 |   379.75 |    4.901 |    26.12 |
|   512 |    128 |  14848 |    1.353 |   378.51 |    4.907 |    26.09 |
|   512 |    128 |  15360 |    1.353 |   378.32 |    4.909 |    26.07 |
|   512 |    128 |  15872 |    1.352 |   378.58 |    4.905 |    26.10 |

Perhaps try rebuilding from scratch?

---

👤 **YurkoHoshko** commented on **2025-11-04** at **09:07:19**

hmm, I deleted the repo, pulled and rebuilt from scratch - the same issue persists. 
For context, I am running an Ubuntu docker image `nvidia/cuda:12.8.1-cudnn-devel-ubuntu24.04` - perhaps it has something to do with it. Will experiment more  tomorrow and get back to you. Appreciate your prompt response.

Full command: 

`docker run --rm --device nvidia.com/gpu=all -v /home/yurko/Code/ik_llama.cpp:/ik_llama.cpp -it -v /home/yurko/.cache/llama.cpp/:/models -p 8000:8000 --name iktest nvidia/cuda:12.8.1-cudnn-devel-ubuntu24.04 /bin/bash`

`/ik_llama.cpp/build/bin/llama-sweep-bench -m /models/gpt-oss-120b.gguf -ngl 99  -c 16768  --temp 1 --top-p 1.0 --top-k 0 -rtr --cpu-moe`

---

👤 **YurkoHoshko** commented on **2025-11-05** at **06:35:28**

Ok, so I did some more testing and narrowed it down to RTX 5060TI. 

When I run the bench against RTX 3060 (by setting `CUDA_VISIBLE_DEVICES=1`) before running the bench - it works just fine. However, when I re-ran the bench with  RTX 5060 TI (`CUDA_VISIBLE_DEVICES=0`) I get the same error I reported originally.

As I mentioned before, gpt-oss 20b runs without issues.

<details><summary>nvidia-smi</summary>

```
root@ad967fd4ac31:/# nvidia-smi -L
GPU 0: NVIDIA GeForce RTX 5060 Ti (UUID: GPU-a259bbb8-2237-a9b8-0195-5834160801a1)
GPU 1: NVIDIA GeForce RTX 3060 (UUID: GPU-0cec721f-8bd6-a6e9-e8a9-923ae1c477a8)
root@ad967fd4ac31:/#
```

</details>

---

👤 **ikawrakow** commented on **2025-11-05** at **06:38:48**

Thanks for investigating.

Can you try rebuilding with
```
cmake -DGGML_CUDA_FUSION=0 $other_cmake_args
```
and let me know if the issue on the 5060 persists? Thanks!

---

👤 **YurkoHoshko** commented on **2025-11-05** at **06:57:56**

You are truly a master of this codebase, your suggestion worked! Thank you!

---

👤 **ikawrakow** commented on **2025-11-05** at **15:37:18**

So, I absolutely cannot make it fail. Fusions enabled or disabled, rope cache yes or no, merged QKV yes or no, all experts left on the CPU or a few experts offloaded to the GPU, it all works.

I see you are using the `f16` model from Unsloth, while I'm using the [gpt-oss-120b model from ggml-org](https://huggingface.co/ggml-org/gpt-oss-120b-GGUF). Perhaps it has something to do with that?

What is your rationale for using the `f16` model. The only tensors that are `f16` in that model are the attention tensors. The model was trained with `bf16` self-attention, so why would one want to use `f16` (or publish the `f16` model in the first place)? This can always run into numerical problems (e.g., overflow). It is also going to run slower on top of everything else.

---

👤 **YurkoHoshko** commented on **2025-11-06** at **05:03:29**

Thank you for spending time researching this issue - hope I am not wasting your time. 

I followed your suggestion and pulled the ggml-org model - the same issue persists unfortunately.

I believe it is the same model as Unsloth's - they just named it F16, but I think it is the same model under the hood (there was a bit of an explanation on [Reddit](https://www.reddit.com/r/LocalLLM/comments/1mvqbo2/comment/n9vaa1x)). 

I suspect there might be something else at play, e.g.:
- something to do with RTX 5xxx series that can not be reproduced with 4xxx
- my system is using open-source driver, not a proprietary one (likely not the case, as open is seemingly becoming a default)
- something to do with the fact that it is running in Docker (likely not, because it works when I disable fusion)?

```
❯ modinfo -l nvidia
Dual MIT/GPL
```

If it would help - happy to hop on a call if we can find time that works for both of us to do some troubleshooting.