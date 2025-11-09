## 🔀 [Pull Request #823](https://github.com/ikawrakow/ik_llama.cpp/pull/823) - Refactor file llama.cpp

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/refactor_llama.cpp` |
| **Target Branch** | `main` |
| **Created** | 2025-10-11 |
| **Updated** | 2025-10-12 |
| **Merged** | 2025-10-11 |

---

## 📄 Description

`src/llama.cpp` is nearly 23 kLOC, and it takes in the range of 60 seconds to compile on my Ryzen-7950X. Hence, adding changes to this file is extremely annoying.

This PR refactors `src/llama.cpp` into several files. Recompiling these files from scratch now takes about 6 seconds on my system.

In addition to being able to compile the files in parallel, there were two main culprits causing the extremely long compilation time (C++ compilation is slow, but 60 seconds for 23 kLOC is way slower than normal):
* The inlined operators of the `LLM_TN` struct. Making those not inline reduces compilation time by a factor of 2!
* The excessive use of lambdas in giant switch statements. Replacing those with class methods results in another 1.5X speedup.

These two changes reduce the compilation time of `src/llama.cpp` to less than 20 seconds. Splitting into several files so compilation can be done in parallel brings it down to 6 seconds.

Hopefully nothing was broken with this refactoring.

There is a lot of duplicated code when creating the model tensors. I have extracted a few repetitions as separate functions here and there, but left a more complete tensor creation de-duplication for a future PR.

---

## 💬 Conversation

👤 **usrlocalben** commented on **2025-10-12** at **04:00:44**

I'm seeing TG throughput ~1/3 of normal after the last two commits. Could this have caused previously internal bindings to become external?

---

👤 **ikawrakow** commented on **2025-10-12** at **05:05:52**

I see zero performance impact on my end, as expected.

Someone else seeing performance drop?

---

👤 **sousekd** commented on **2025-10-12** at **08:33:50**

@usrlocalben @ikawrakow ~I noticed drop to 1/3 TG speed yesterday after updating Ubuntu Server 24.04, CUDA and ik_llama.cpp. Rollback of ik_llama.cpp update **did not** help. Rollback of the system and CUDA update **did** help. => I think it is the latest CUDA what is causing the issue.~

---

👤 **ikawrakow** commented on **2025-10-12** at **08:36:50**

> I think it is the latest CUDA what is causing the issue.

Interesting. What are the CUDA and driver versions that lead to such performance drop?

---

👤 **sousekd** commented on **2025-10-12** at **08:52:24**

@ikawrakow 

I am new to Linux, so bear with me:

- I have no issues with driver version 580.95.05 and CUDA 13.0 (nvidia-smi).
- I have this source registered in `apt`:
  https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2404/x86_64

Output of `apt list --upgradable`:

```
Listing... Done
cuda-13-0/unknown 13.0.2-1 amd64 [upgradable from: 13.0.1-1]
cuda-command-line-tools-13-0/unknown 13.0.2-1 amd64 [upgradable from: 13.0.1-1]
cuda-compiler-13-0/unknown 13.0.2-1 amd64 [upgradable from: 13.0.1-1]
cuda-cudart-13-0/unknown 13.0.96-1 amd64 [upgradable from: 13.0.88-1]
cuda-cudart-dev-13-0/unknown 13.0.96-1 amd64 [upgradable from: 13.0.88-1]
cuda-driver-dev-13-0/unknown 13.0.96-1 amd64 [upgradable from: 13.0.88-1]
cuda-libraries-13-0/unknown 13.0.2-1 amd64 [upgradable from: 13.0.1-1]
cuda-libraries-dev-13-0/unknown 13.0.2-1 amd64 [upgradable from: 13.0.1-1]
cuda-nsight-compute-13-0/unknown 13.0.2-1 amd64 [upgradable from: 13.0.1-1]
cuda-nsight-systems-13-0/unknown 13.0.2-1 amd64 [upgradable from: 13.0.1-1]
cuda-runtime-13-0/unknown 13.0.2-1 amd64 [upgradable from: 13.0.1-1]
cuda-toolkit-13-0-config-common/unknown 13.0.96-1 all [upgradable from: 13.0.88-1]
cuda-toolkit-13-0/unknown 13.0.2-1 amd64 [upgradable from: 13.0.1-1]
cuda-toolkit-13-config-common/unknown 13.0.96-1 all [upgradable from: 13.0.88-1]
cuda-toolkit-config-common/unknown 13.0.96-1 all [upgradable from: 13.0.88-1]
cuda-tools-13-0/unknown 13.0.2-1 amd64 [upgradable from: 13.0.1-1]
cuda-visual-tools-13-0/unknown 13.0.2-1 amd64 [upgradable from: 13.0.1-1]
cuda/unknown 13.0.2-1 amd64 [upgradable from: 13.0.1-1]
firmware-sof-signed/noble-updates 2023.12.1-1ubuntu1.10 all [upgradable from: 2023.12.1-1ubuntu1.9]
libcublas-13-0/unknown 13.1.0.3-1 amd64 [upgradable from: 13.0.2.14-1]
libcublas-dev-13-0/unknown 13.1.0.3-1 amd64 [upgradable from: 13.0.2.14-1]
libnss-systemd/noble-updates 255.4-1ubuntu8.11 amd64 [upgradable from: 255.4-1ubuntu8.10]
libpam-systemd/noble-updates 255.4-1ubuntu8.11 amd64 [upgradable from: 255.4-1ubuntu8.10]
libsystemd-shared/noble-updates 255.4-1ubuntu8.11 amd64 [upgradable from: 255.4-1ubuntu8.10]
libsystemd0/noble-updates 255.4-1ubuntu8.11 amd64 [upgradable from: 255.4-1ubuntu8.10]
libudev1/noble-updates 255.4-1ubuntu8.11 amd64 [upgradable from: 255.4-1ubuntu8.10]
snapd/noble-updates 2.71+ubuntu24.04 amd64 [upgradable from: 2.68.5+ubuntu24.04.1]
systemd-dev/noble-updates 255.4-1ubuntu8.11 all [upgradable from: 255.4-1ubuntu8.10]
systemd-hwe-hwdb/noble-updates 255.1.6 all [upgradable from: 255.1.5]
systemd-resolved/noble-updates 255.4-1ubuntu8.11 amd64 [upgradable from: 255.4-1ubuntu8.10]
systemd-sysv/noble-updates 255.4-1ubuntu8.11 amd64 [upgradable from: 255.4-1ubuntu8.10]
systemd-timesyncd/noble-updates 255.4-1ubuntu8.11 amd64 [upgradable from: 255.4-1ubuntu8.10]
systemd/noble-updates 255.4-1ubuntu8.11 amd64 [upgradable from: 255.4-1ubuntu8.10]
tcpdump/noble-updates 4.99.4-3ubuntu4.24.04.1 amd64 [upgradable from: 4.99.4-3ubuntu4]
udev/noble-updates 255.4-1ubuntu8.11 amd64 [upgradable from: 255.4-1ubuntu8.10]
```

~After `apt upgrade` and updating ik_llama.cpp from the main branch, the TG speed dropped. I was really tired, so I am not 100% sure, but I think that rollback of ik_llama.cpp update did not help, while rollback of `apt` upgrade did. This what I did exactly:~

1. ~`sudo apt upgrade`, pull of llama.cpp and ik_llama.cpp, recompiling, deploying~
2. ~noticed ik_llama.cpp is failing (OOM) on the first request to one of the models where it worked before~
3. ~rolled back the whole update (it is VM, so I used earlier snapshot), confirmed ik_llama.cpp worked well~
4. ~`sudo apt upgrade`, pull of llama.cpp and ik_llama.cpp, recompiling, redeploying **except of ik_llama.cpp**, to resolve later~
5. ~noticed the speed drop, so I rolled back everything again and went to bed, too tired to investigate further~

---

👤 **sousekd** commented on **2025-10-12** at **11:32:47**

@ikawrakow

Okay, I have a small update, contradicting my previous response. I've updated Ubuntu, holding back CUDA updates, and run benchmark on older ik_llama.cpp with about 17 t/s TG, I pulled ik_llama.cpp, recompiled and rerun the same bench. This time, it failed on OOM. I reduced the context size, which made it run successfully, but now giving me only 7 t/s TG.

So there is something spooky going on. I will do more tests...

---

👤 **sousekd** commented on **2025-10-12** at **12:38:45**

Okay, I was completely wrong — sorry for the confusion earlier.  
Blaming CUDA was baseless.

Below are the benchmark logs comparing the **older `ik_llama.cpp` (commit `3d4977cb`)** and the **current version (commit `0ad1d340`)**, both compiled with the latest CUDA.

Model is @ubergarm's Kimi-K2-Instruct-0905-smol-IQ5_KS, GPU is RTX 5090.

<details>
<summary>Older version: <code>ik_llama.cpp @ 3d4977cb</code></summary>

```bash
./llama-sweep-bench \
    --model "$MODEL_PATH" \
    --no-mmap \
    -mla 3 -fa -fmoe \
    -amb 512 -b 4096 -ub 4096 \
    -ctk f16 -c 65536 \
    -ngl 999 -ot exps=CPU \
    --threads 16 \
    --threads-batch 28 \
    --warmup-batch

ggml_cuda_init: GGML_CUDA_FORCE_MMQ:    no
ggml_cuda_init: GGML_CUDA_FORCE_CUBLAS: no
ggml_cuda_init: found 1 CUDA devices:
  Device 0: NVIDIA GeForce RTX 5090, compute capability 12.0, VMM: yes
llama_model_loader: additional 13 GGUFs metadata loaded.
llama_model_loader: loaded meta data with 52 key-value pairs and 1096 tensors from /mnt/hot/hfcache/ubergarm/Kimi-K2-Instruct-0905-GGUF/Kimi-K2-Instruct-0905-smol-IQ5_KS-00001-of-00014.gguf (version GGUF V3 (latest))
llama_model_loader: Dumping metadata keys/values. Note: KV overrides do not apply in this output.
llama_model_loader: - kv   0:                       general.architecture str              = deepseek2
llama_model_loader: - kv   1:                               general.type str              = model
llama_model_loader: - kv   2:                            general.version str              = 0905
llama_model_loader: - kv   3:                           general.finetune str              = Instruct-safetensors
llama_model_loader: - kv   4:                           general.basename str              = Kimi-K2
llama_model_loader: - kv   5:                         general.size_label str              = 384x14B
llama_model_loader: - kv   6:                      deepseek2.block_count u32              = 61
llama_model_loader: - kv   7:                   deepseek2.context_length u32              = 262144
llama_model_loader: - kv   8:                 deepseek2.embedding_length u32              = 7168
llama_model_loader: - kv   9:              deepseek2.feed_forward_length u32              = 18432
llama_model_loader: - kv  10:             deepseek2.attention.head_count u32              = 64
llama_model_loader: - kv  11:          deepseek2.attention.head_count_kv u32              = 1
llama_model_loader: - kv  12:                   deepseek2.rope.freq_base f32              = 50000.000000
llama_model_loader: - kv  13: deepseek2.attention.layer_norm_rms_epsilon f32              = 0.000010
llama_model_loader: - kv  14:                deepseek2.expert_used_count u32              = 8
llama_model_loader: - kv  15:                          general.file_type u32              = 150
llama_model_loader: - kv  16:        deepseek2.leading_dense_block_count u32              = 1
llama_model_loader: - kv  17:                       deepseek2.vocab_size u32              = 163840
llama_model_loader: - kv  18:            deepseek2.attention.q_lora_rank u32              = 1536
llama_model_loader: - kv  19:           deepseek2.attention.kv_lora_rank u32              = 512
llama_model_loader: - kv  20:             deepseek2.attention.key_length u32              = 576
llama_model_loader: - kv  21:           deepseek2.attention.value_length u32              = 512
llama_model_loader: - kv  22:         deepseek2.attention.key_length_mla u32              = 192
llama_model_loader: - kv  23:       deepseek2.attention.value_length_mla u32              = 128
llama_model_loader: - kv  24:       deepseek2.expert_feed_forward_length u32              = 2048
llama_model_loader: - kv  25:                     deepseek2.expert_count u32              = 384
llama_model_loader: - kv  26:              deepseek2.expert_shared_count u32              = 1
llama_model_loader: - kv  27:             deepseek2.expert_weights_scale f32              = 2.827000
llama_model_loader: - kv  28:              deepseek2.expert_weights_norm bool             = true
llama_model_loader: - kv  29:               deepseek2.expert_gating_func u32              = 2
llama_model_loader: - kv  30:             deepseek2.rope.dimension_count u32              = 64
llama_model_loader: - kv  31:                deepseek2.rope.scaling.type str              = yarn
llama_model_loader: - kv  32:              deepseek2.rope.scaling.factor f32              = 64.000000
llama_model_loader: - kv  33: deepseek2.rope.scaling.original_context_length u32              = 4096
llama_model_loader: - kv  34: deepseek2.rope.scaling.yarn_log_multiplier f32              = 0.100000
llama_model_loader: - kv  35:               general.quantization_version u32              = 2
llama_model_loader: - kv  36:                       tokenizer.ggml.model str              = gpt2
llama_model_loader: - kv  37:                         tokenizer.ggml.pre str              = kimi-k2
llama_model_loader: - kv  38:                      tokenizer.ggml.tokens arr[str,163840]  = ["!", "\"", "#", "$", "%", "&", "'", ...
llama_model_loader: - kv  39:                  tokenizer.ggml.token_type arr[i32,163840]  = [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, ...
llama_model_loader: - kv  40:                      tokenizer.ggml.merges arr[str,163328]  = ["Ġ Ġ", "ĠĠ ĠĠ", "Ġ t", "i n",...
llama_model_loader: - kv  41:                tokenizer.ggml.bos_token_id u32              = 163584
llama_model_loader: - kv  42:                tokenizer.ggml.eos_token_id u32              = 163585
llama_model_loader: - kv  43:            tokenizer.ggml.padding_token_id u32              = 163839
llama_model_loader: - kv  44:                    tokenizer.chat_template str              = {%- if tools -%}\n  <|im_system|>tool_...
llama_model_loader: - kv  45:                      quantize.imatrix.file str              = /mnt/data/models/ubergarm/Kimi-K2-Ins...
llama_model_loader: - kv  46:                   quantize.imatrix.dataset str              = ubergarm-imatrix-calibration-corpus-v...
llama_model_loader: - kv  47:             quantize.imatrix.entries_count i32              = 667
llama_model_loader: - kv  48:              quantize.imatrix.chunks_count i32              = 826
llama_model_loader: - kv  49:                                   split.no u16              = 0
llama_model_loader: - kv  50:                                split.count u16              = 14
llama_model_loader: - kv  51:                        split.tensors.count i32              = 1096
llama_model_loader: - type  f32:  365 tensors
llama_model_loader: - type q8_0:  549 tensors
llama_model_loader: - type iq6_k:    2 tensors
llama_model_loader: - type iq5_ks:  180 tensors
init_tokenizer: initializing tokenizer for type 2
load: special_eos_id is not in special_eog_ids - the tokenizer config may be incorrect
load: printing all EOG tokens:
load:   - 163585 ('[EOS]')
load:   - 163586 ('<|im_end|>')
load: special tokens cache size = 256
load: token to piece cache size = 1.0607 MB
llm_load_print_meta: format           = GGUF V3 (latest)
llm_load_print_meta: arch             = deepseek2
llm_load_print_meta: n_ctx_train      = 262144
llm_load_print_meta: n_embd           = 7168
llm_load_print_meta: n_layer          = 61
llm_load_print_meta: n_head           = 64
llm_load_print_meta: n_head_kv        = 64
llm_load_print_meta: n_rot            = 64
llm_load_print_meta: n_swa            = 0
llm_load_print_meta: n_swa_pattern    = 1
llm_load_print_meta: n_embd_head_k    = 192
llm_load_print_meta: n_embd_head_v    = 128
llm_load_print_meta: n_gqa            = 1
llm_load_print_meta: n_embd_k_gqa     = 12288
llm_load_print_meta: n_embd_v_gqa     = 8192
llm_load_print_meta: f_norm_eps       = 0.0e+00
llm_load_print_meta: f_norm_rms_eps   = 1.0e-05
llm_load_print_meta: f_clamp_kqv      = 0.0e+00
llm_load_print_meta: f_max_alibi_bias = 0.0e+00
llm_load_print_meta: f_logit_scale    = 0.0e+00
llm_load_print_meta: n_ff             = 18432
llm_load_print_meta: n_expert         = 384
llm_load_print_meta: n_expert_used    = 8
llm_load_print_meta: causal attn      = 1
llm_load_print_meta: pooling type     = 0
llm_load_print_meta: rope type        = 0
llm_load_print_meta: rope scaling     = yarn
llm_load_print_meta: freq_base_train  = 50000.0
llm_load_print_meta: freq_scale_train = 0.015625
llm_load_print_meta: n_ctx_orig_yarn  = 4096
llm_load_print_meta: rope_finetuned   = unknown
llm_load_print_meta: ssm_d_conv       = 0
llm_load_print_meta: ssm_d_inner      = 0
llm_load_print_meta: ssm_d_state      = 0
llm_load_print_meta: ssm_dt_rank      = 0
llm_load_print_meta: model type       = 671B
llm_load_print_meta: model ftype      = IQ5_KS - 5.25 bpw
llm_load_print_meta: model params     = 1.026 T
llm_load_print_meta: model size       = 632.664 GiB (5.295 BPW) 
llm_load_print_meta: repeating layers = 630.853 GiB (5.292 BPW, 1024.059 B parameters)
llm_load_print_meta: general.name     = n/a
llm_load_print_meta: n_layer_dense_lead   = 1
llm_load_print_meta: n_lora_q             = 1536
llm_load_print_meta: n_lora_kv            = 512
llm_load_print_meta: n_ff_exp             = 2048
llm_load_print_meta: n_expert_shared      = 1
llm_load_print_meta: expert_weights_scale = 2.8
llm_load_print_meta: expert_weights_norm  = 1
llm_load_print_meta: expert_gating_func   = sigmoid
llm_load_print_meta: rope_yarn_log_mul    = 0.1000
print_info: vocab type       = BPE
print_info: n_vocab          = 163840
print_info: n_merges         = 163328
print_info: BOS token        = 163584 '[BOS]'
print_info: EOS token        = 163585 '[EOS]'
print_info: EOT token        = 163586 '<|im_end|>'
print_info: PAD token        = 163839 '[PAD]'
print_info: LF token         = 198 'Ċ'
print_info: EOG token        = 163585 '[EOS]'
print_info: EOG token        = 163586 '<|im_end|>'
print_info: max token length = 512
llm_load_tensors: ggml ctx size =    0.90 MiB
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
llm_load_tensors: offloading 61 repeating layers to GPU
llm_load_tensors: offloading non-repeating layers to GPU
llm_load_tensors: offloaded 62/62 layers to GPU
llm_load_tensors:        CPU buffer size = 636030.00 MiB
llm_load_tensors:  CUDA_Host buffer size =   927.50 MiB
llm_load_tensors:      CUDA0 buffer size = 10890.94 MiB
....................................................................................................
============ llm_prepare_mla: need to compute 61 wkv_b tensors
================= Adjusted mainline llama.cpp MLA tensors to ik_llama.cpp
Computed blk.0.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.1.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.2.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.3.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.4.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.5.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.6.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.7.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.8.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.9.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.10.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.11.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.12.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.13.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.14.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.15.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.16.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.17.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.18.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.19.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.20.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.21.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.22.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.23.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.24.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.25.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.26.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.27.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.28.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.29.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.30.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.31.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.32.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.33.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.34.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.35.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.36.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.37.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.38.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.39.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.40.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.41.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.42.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.43.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.44.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.45.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.46.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.47.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.48.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.49.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.50.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.51.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.52.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.53.attn_kv_b.weight as 512 x 16384 and storellama_new_context_with_model: n_ctx      = 65536
llama_new_context_with_model: n_batch    = 4096
llama_new_context_with_model: n_ubatch   = 4096
llama_new_context_with_model: flash_attn = 1
llama_new_context_with_model: mla_attn   = 3
llama_new_context_with_model: attn_max_b = 512
llama_new_context_with_model: fused_moe  = 1
llama_new_context_with_model: fused_up_gate = 1
llama_new_context_with_model: ser        = -1, 0
llama_new_context_with_model: freq_base  = 50000.0
llama_new_context_with_model: freq_scale = 0.015625
llama_kv_cache_init:      CUDA0 KV buffer size =  4392.00 MiB
llama_new_context_with_model: KV self size  = 4392.00 MiB, c^KV (f16): 4392.00 MiB, kv^T: not used
llama_new_context_with_model:  CUDA_Host  output buffer size =     0.62 MiB
llama_new_context_with_model:      CUDA0 compute buffer size =  8474.52 MiB
llama_new_context_with_model:  CUDA_Host compute buffer size =  1136.05 MiB
llama_new_context_with_model: graph nodes  = 8100
llama_new_context_with_model: graph splits = 122

main: n_kv_max = 65536, n_batch = 4096, n_ubatch = 4096, flash_attn = 1, n_gpu_layers = 999, n_threads = 16, n_threads_batch = 28

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  4096 |   1024 |      0 |   18.926 |   216.42 |   57.108 |    17.93 |
```

</details>


<details>
<summary>Current version: <code>ik_llama.cpp @ 0ad1d340</code></summary>

```bash
./llama-sweep-bench \
    --model "$MODEL_PATH" \
    --no-mmap \
    -mla 3 -fa -fmoe \
    -amb 512 -b 4096 -ub 4096 \
    -ctk f16 -c 65536 \
    -ngl 999 -ot exps=CPU \
    --threads 16 \
    --threads-batch 28 \
    --warmup-batch

ggml_cuda_init: GGML_CUDA_FORCE_MMQ:    no
ggml_cuda_init: GGML_CUDA_FORCE_CUBLAS: no
ggml_cuda_init: found 1 CUDA devices:
  Device 0: NVIDIA GeForce RTX 5090, compute capability 12.0, VMM: yes
llama_model_loader: additional 13 GGUFs metadata loaded.
llama_model_loader: loaded meta data with 52 key-value pairs and 1096 tensors from /mnt/hot/hfcache/ubergarm/Kimi-K2-Instruct-0905-GGUF/Kimi-K2-Instruct-0905-smol-IQ5_KS-00001-of-00014.gguf (version GGUF V3 (latest))
llama_model_loader: Dumping metadata keys/values. Note: KV overrides do not apply in this output.
llama_model_loader: - kv   0:                       general.architecture str              = deepseek2
llama_model_loader: - kv   1:                               general.type str              = model
llama_model_loader: - kv   2:                            general.version str              = 0905
llama_model_loader: - kv   3:                           general.finetune str              = Instruct-safetensors
llama_model_loader: - kv   4:                           general.basename str              = Kimi-K2
llama_model_loader: - kv   5:                         general.size_label str              = 384x14B
llama_model_loader: - kv   6:                      deepseek2.block_count u32              = 61
llama_model_loader: - kv   7:                   deepseek2.context_length u32              = 262144
llama_model_loader: - kv   8:                 deepseek2.embedding_length u32              = 7168
llama_model_loader: - kv   9:              deepseek2.feed_forward_length u32              = 18432
llama_model_loader: - kv  10:             deepseek2.attention.head_count u32              = 64
llama_model_loader: - kv  11:          deepseek2.attention.head_count_kv u32              = 1
llama_model_loader: - kv  12:                   deepseek2.rope.freq_base f32              = 50000.000000
llama_model_loader: - kv  13: deepseek2.attention.layer_norm_rms_epsilon f32              = 0.000010
llama_model_loader: - kv  14:                deepseek2.expert_used_count u32              = 8
llama_model_loader: - kv  15:                          general.file_type u32              = 150
llama_model_loader: - kv  16:        deepseek2.leading_dense_block_count u32              = 1
llama_model_loader: - kv  17:                       deepseek2.vocab_size u32              = 163840
llama_model_loader: - kv  18:            deepseek2.attention.q_lora_rank u32              = 1536
llama_model_loader: - kv  19:           deepseek2.attention.kv_lora_rank u32              = 512
llama_model_loader: - kv  20:             deepseek2.attention.key_length u32              = 576
llama_model_loader: - kv  21:           deepseek2.attention.value_length u32              = 512
llama_model_loader: - kv  22:         deepseek2.attention.key_length_mla u32              = 192
llama_model_loader: - kv  23:       deepseek2.attention.value_length_mla u32              = 128
llama_model_loader: - kv  24:       deepseek2.expert_feed_forward_length u32              = 2048
llama_model_loader: - kv  25:                     deepseek2.expert_count u32              = 384
llama_model_loader: - kv  26:              deepseek2.expert_shared_count u32              = 1
llama_model_loader: - kv  27:             deepseek2.expert_weights_scale f32              = 2.827000
llama_model_loader: - kv  28:              deepseek2.expert_weights_norm bool             = true
llama_model_loader: - kv  29:               deepseek2.expert_gating_func u32              = 2
llama_model_loader: - kv  30:             deepseek2.rope.dimension_count u32              = 64
llama_model_loader: - kv  31:                deepseek2.rope.scaling.type str              = yarn
llama_model_loader: - kv  32:              deepseek2.rope.scaling.factor f32              = 64.000000
llama_model_loader: - kv  33: deepseek2.rope.scaling.original_context_length u32              = 4096
llama_model_loader: - kv  34: deepseek2.rope.scaling.yarn_log_multiplier f32              = 0.100000
llama_model_loader: - kv  35:               general.quantization_version u32              = 2
llama_model_loader: - kv  36:                       tokenizer.ggml.model str              = gpt2
llama_model_loader: - kv  37:                         tokenizer.ggml.pre str              = kimi-k2
llama_model_loader: - kv  38:                      tokenizer.ggml.tokens arr[str,163840]  = ["!", "\"", "#", "$", "%", "&", "'", ...
llama_model_loader: - kv  39:                  tokenizer.ggml.token_type arr[i32,163840]  = [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, ...
llama_model_loader: - kv  40:                      tokenizer.ggml.merges arr[str,163328]  = ["Ġ Ġ", "ĠĠ ĠĠ", "Ġ t", "i n",...
llama_model_loader: - kv  41:                tokenizer.ggml.bos_token_id u32              = 163584
llama_model_loader: - kv  42:                tokenizer.ggml.eos_token_id u32              = 163585
llama_model_loader: - kv  43:            tokenizer.ggml.padding_token_id u32              = 163839
llama_model_loader: - kv  44:                    tokenizer.chat_template str              = {%- if tools -%}\n  <|im_system|>tool_...
llama_model_loader: - kv  45:                      quantize.imatrix.file str              = /mnt/data/models/ubergarm/Kimi-K2-Ins...
llama_model_loader: - kv  46:                   quantize.imatrix.dataset str              = ubergarm-imatrix-calibration-corpus-v...
llama_model_loader: - kv  47:             quantize.imatrix.entries_count i32              = 667
llama_model_loader: - kv  48:              quantize.imatrix.chunks_count i32              = 826
llama_model_loader: - kv  49:                                   split.no u16              = 0
llama_model_loader: - kv  50:                                split.count u16              = 14
llama_model_loader: - kv  51:                        split.tensors.count i32              = 1096
llama_model_loader: - type  f32:  365 tensors
llama_model_loader: - type q8_0:  549 tensors
llama_model_loader: - type iq6_k:    2 tensors
llama_model_loader: - type iq5_ks:  180 tensors
init_tokenizer: initializing tokenizer for type 2
load: special_eos_id is not in special_eog_ids - the tokenizer config may be incorrect
load: printing all EOG tokens:
load:   - 163585 ('[EOS]')
load:   - 163586 ('<|im_end|>')
load: special tokens cache size = 256
load: token to piece cache size = 1.0607 MB
llm_load_print_meta: format           = GGUF V3 (latest)
llm_load_print_meta: arch             = deepseek2
llm_load_print_meta: n_ctx_train      = 262144
llm_load_print_meta: n_embd           = 7168
llm_load_print_meta: n_layer          = 61
llm_load_print_meta: n_head           = 64
llm_load_print_meta: n_head_kv        = 64
llm_load_print_meta: n_rot            = 64
llm_load_print_meta: n_swa            = 0
llm_load_print_meta: n_swa_pattern    = 1
llm_load_print_meta: n_embd_head_k    = 192
llm_load_print_meta: n_embd_head_v    = 128
llm_load_print_meta: n_gqa            = 1
llm_load_print_meta: n_embd_k_gqa     = 12288
llm_load_print_meta: n_embd_v_gqa     = 8192
llm_load_print_meta: f_norm_eps       = 0.0e+00
llm_load_print_meta: f_norm_rms_eps   = 1.0e-05
llm_load_print_meta: f_clamp_kqv      = 0.0e+00
llm_load_print_meta: f_max_alibi_bias = 0.0e+00
llm_load_print_meta: f_logit_scale    = 0.0e+00
llm_load_print_meta: n_ff             = 18432
llm_load_print_meta: n_expert         = 384
llm_load_print_meta: n_expert_used    = 8
llm_load_print_meta: causal attn      = 1
llm_load_print_meta: pooling type     = 0
llm_load_print_meta: rope type        = 0
llm_load_print_meta: rope scaling     = yarn
llm_load_print_meta: freq_base_train  = 50000.0
llm_load_print_meta: freq_scale_train = 0.015625
llm_load_print_meta: n_ctx_orig_yarn  = 4096
llm_load_print_meta: rope_finetuned   = unknown
llm_load_print_meta: ssm_d_conv       = 0
llm_load_print_meta: ssm_d_inner      = 0
llm_load_print_meta: ssm_d_state      = 0
llm_load_print_meta: ssm_dt_rank      = 0
llm_load_print_meta: model type       = 671B
llm_load_print_meta: model ftype      = IQ5_KS - 5.25 bpw
llm_load_print_meta: model params     = 1.026 T
llm_load_print_meta: model size       = 632.664 GiB (5.295 BPW) 
llm_load_print_meta: repeating layers = 630.853 GiB (5.292 BPW, 1024.059 B parameters)
llm_load_print_meta: general.name     = n/a
llm_load_print_meta: n_layer_dense_lead   = 1
llm_load_print_meta: n_lora_q             = 1536
llm_load_print_meta: n_lora_kv            = 512
llm_load_print_meta: n_ff_exp             = 2048
llm_load_print_meta: n_expert_shared      = 1
llm_load_print_meta: expert_weights_scale = 2.8
llm_load_print_meta: expert_weights_norm  = 1
llm_load_print_meta: expert_gating_func   = sigmoid
llm_load_print_meta: rope_yarn_log_mul    = 0.1000
print_info: vocab type       = BPE
print_info: n_vocab          = 163840
print_info: n_merges         = 163328
print_info: BOS token        = 163584 '[BOS]'
print_info: EOS token        = 163585 '[EOS]'
print_info: EOT token        = 163586 '<|im_end|>'
print_info: PAD token        = 163839 '[PAD]'
print_info: LF token         = 198 'Ċ'
print_info: EOG token        = 163585 '[EOS]'
print_info: EOG token        = 163586 '<|im_end|>'
print_info: max token length = 512
llm_load_tensors: ggml ctx size =    0.90 MiB
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
llm_load_tensors: offloading 61 repeating layers to GPU
llm_load_tensors: offloading non-repeating layers to GPU
llm_load_tensors: offloaded 62/62 layers to GPU
llm_load_tensors:        CPU buffer size = 632499.00 MiB
llm_load_tensors:  CUDA_Host buffer size =   927.50 MiB
llm_load_tensors:      CUDA0 buffer size = 14421.94 MiB
....................................................................................................
============ llm_prepare_mla: need to compute 61 wkv_b tensors
================= Adjusted mainline llama.cpp MLA tensors to ik_llama.cpp
Computed blk.0.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.1.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.2.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.3.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.4.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.5.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.6.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.7.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.8.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.9.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.10.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.11.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.12.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.13.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.14.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.15.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.16.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.17.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.18.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.19.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.20.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.21.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.22.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.23.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.24.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.25.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.26.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.27.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.28.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.29.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.30.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.31.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.32.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.33.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.34.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.35.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.36.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.37.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.38.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.39.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.40.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.41.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.42.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.43.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.44.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.45.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.46.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.47.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.48.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.49.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.50.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.51.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.52.attn_kv_b.weight as 512 x 16384 and stored in buffer CUDA0
Computed blk.53.attn_kv_b.weight as 512 x 16384 and storellama_new_context_with_model: n_ctx      = 65536
llama_new_context_with_model: n_batch    = 4096
llama_new_context_with_model: n_ubatch   = 4096
llama_new_context_with_model: flash_attn = 1
llama_new_context_with_model: mla_attn   = 3
llama_new_context_with_model: attn_max_b = 512
llama_new_context_with_model: fused_moe  = 1
llama_new_context_with_model: fused_up_gate = 1
llama_new_context_with_model: ser        = -1, 0
llama_new_context_with_model: freq_base  = 50000.0
llama_new_context_with_model: freq_scale = 0.015625
llama_kv_cache_init:      CUDA0 KV buffer size =  4392.00 MiB
llama_new_context_with_model: KV self size  = 4392.00 MiB, c^KV (f16): 4392.00 MiB, kv^T: not used
llama_new_context_with_model:  CUDA_Host  output buffer size =     0.62 MiB
llama_new_context_with_model:      CUDA0 compute buffer size =  8474.52 MiB
llama_new_context_with_model:  CUDA_Host compute buffer size =  1136.05 MiB
llama_new_context_with_model: graph nodes  = 8100
llama_new_context_with_model: graph splits = 122

main: n_kv_max = 65536, n_batch = 4096, n_ubatch = 4096, flash_attn = 1, n_gpu_layers = 999, n_threads = 16, n_threads_batch = 28

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  4096 |   1024 |      0 |   18.472 |   221.74 |  126.497 |     8.10 |
|  4096 |   1024 |   4096 |   19.064 |   214.85 |  127.391 |     8.04 |
```

</details>

In addition to the significant speed drop, I was able to run the same command with a 96K context on the older version, while the current version fails with an OOM error.

---

👤 **ikawrakow** commented on **2025-10-12** at **15:38:47**

@sousekd Hmm, I cannot reproduce (and I wouldn't know why this PR would change TG performance). But, just in case, can you try [#825](https://github.com/ikawrakow/ik_llama.cpp/issues/825)?

---

👤 **sousekd** commented on **2025-10-12** at **16:00:12**

@ikawrakow I will try it. For now, I can confirm up to f649e36 is fine, so the issue seems to be localized to this PR.

---

👤 **usrlocalben** commented on **2025-10-12** at **16:30:24**

I tested again to be sure.

f649e36a
```
prompt eval time     =    4876.58 ms /   625 tokens (    7.80 ms per token,   128.16 tokens per second)
generation eval time =   23352.28 ms /   400 runs (   58.38 ms per token,    17.13 tokens per second)
```
cpu during TG: ~3200% 

335a1f9b
```
prompt eval time     =    5001.73 ms /   625 tokens (    8.00 ms per token,   124.96d tokens per second)
generation eval time =   56118.58 ms /   401 runs (  139.95 ms per token,     7.15 tokens per second)
```
cpu during TG: ~1400% (1 thread at 100%)

This system uses hybrid inference, Blackwell attention & EPYC Experts.
Model is Kimi-K2-0905.

I still wonder the same: Did something that previously had internal-linkage (i.e. inlined / optimized away) now have external-linkage?
If so, it is likely something at the graph/execution level since workers appear to be waiting on the main thread.

---

👤 **ikawrakow** commented on **2025-10-12** at **16:43:30**

OK, I'm seeing a TG performance drop for GTP-OSS-120B with `-ot exps=CPU`. Not as dramatic as yours, but still significant (~30%). That's good as now I can investigate.

---

👤 **usrlocalben** commented on **2025-10-12** at **16:50:50**

@ikawrakow I ran each commit above and the problem occurs at 4b71c16a .

edit: This is very curious since at a glance none of this looks like hot-path.