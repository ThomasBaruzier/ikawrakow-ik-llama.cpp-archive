## 🔀 [Pull Request #838](https://github.com/ikawrakow/ik_llama.cpp/pull/838) - Grouped expert routing (CUDA)

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/cuda_grouped_topk` |
| **Target Branch** | `main` |
| **Created** | 2025-10-17 |
| **Updated** | 2025-10-21 |
| **Merged** | 2025-10-18 |

---

## 📄 Description

This PR adds CUDA implementation of grouped experts routing as used by the BailingMoeV2 arch (Ling/Ring models).

I did try initially to get a single kernel implementation as on the CPU (PR [#836](https://github.com/ikawrakow/ik_llama.cpp/issues/836)), but that wasn't working, so the op is composed of several separate kernel launches. Still, performance is not too bad with just a few percent degradation compared to standard routing (see tables below), and definitely much better compared to having the grouped expert top_k op done on the CPU.

The mainline BalingMoeV2 PR has not been merged yet, so will not give performance comparisons to `llama.cpp`. 

### Ling-mini-2.0-Q4_K_M, standard expert routing

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  2048 |    512 |      0 |    0.151 | 13527.88 |    1.290 |   396.98 |
|  2048 |    512 |   2048 |    0.123 | 16711.82 |    1.350 |   379.26 |
|  2048 |    512 |   4096 |    0.128 | 15997.00 |    1.425 |   359.21 |
|  2048 |    512 |   6144 |    0.134 | 15267.29 |    1.505 |   340.13 |
|  2048 |    512 |   8192 |    0.142 | 14449.09 |    1.571 |   325.89 |
|  2048 |    512 |  10240 |    0.149 | 13773.90 |    1.656 |   309.12 |
|  2048 |    512 |  12288 |    0.155 | 13199.11 |    1.712 |   299.15 |
|  2048 |    512 |  14336 |    0.162 | 12652.52 |    1.791 |   285.90 |
|  2048 |    512 |  16384 |    0.168 | 12186.85 |    1.849 |   276.85 |
|  2048 |    512 |  18432 |    0.175 | 11691.37 |    1.915 |   267.40 |
|  2048 |    512 |  20480 |    0.181 | 11322.67 |    1.996 |   256.49 |
|  2048 |    512 |  22528 |    0.188 | 10872.74 |    2.055 |   249.15 |
|  2048 |    512 |  24576 |    0.195 | 10494.17 |    2.145 |   238.66 |
|  2048 |    512 |  26624 |    0.202 | 10141.32 |    2.196 |   233.10 |
|  2048 |    512 |  28672 |    0.209 |  9810.83 |    2.270 |   225.59 |
|  2048 |    512 |  30720 |    0.214 |  9552.51 |    2.344 |   218.43 |

### Ling-mini-2.0-Q4_K_M, grouped expert routing

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  2048 |    512 |      0 |    0.154 | 13270.44 |    1.358 |   376.95 |
|  2048 |    512 |   2048 |    0.126 | 16242.49 |    1.415 |   361.73 |
|  2048 |    512 |   4096 |    0.132 | 15561.48 |    1.495 |   342.58 |
|  2048 |    512 |   6144 |    0.138 | 14797.69 |    1.571 |   325.89 |
|  2048 |    512 |   8192 |    0.144 | 14198.95 |    1.631 |   313.83 |
|  2048 |    512 |  10240 |    0.153 | 13428.54 |    1.720 |   297.66 |
|  2048 |    512 |  12288 |    0.159 | 12918.85 |    1.773 |   288.79 |
|  2048 |    512 |  14336 |    0.164 | 12485.45 |    1.858 |   275.63 |
|  2048 |    512 |  16384 |    0.172 | 11899.92 |    1.915 |   267.37 |
|  2048 |    512 |  18432 |    0.178 | 11517.00 |    1.983 |   258.18 |
|  2048 |    512 |  20480 |    0.185 | 11055.45 |    2.062 |   248.27 |
|  2048 |    512 |  22528 |    0.192 | 10668.56 |    2.123 |   241.19 |
|  2048 |    512 |  24576 |    0.198 | 10355.52 |    2.214 |   231.24 |
|  2048 |    512 |  26624 |    0.205 |  9967.83 |    2.267 |   225.82 |
|  2048 |    512 |  28672 |    0.213 |  9617.87 |    2.338 |   218.97 |
|  2048 |    512 |  30720 |    0.217 |  9448.63 |    2.416 |   211.91 |

---

## 💬 Conversation

👤 **ubergarm** commented on **2025-10-18** at **17:40:40**

Thanks for putting together CUDA `-ger` and merging so quickly! 

I've seen at least a couple positive reports suggesting `-ger` is working okay hybrid CPU+GPU e.g. https://huggingface.co/ubergarm2/Ling-1T-GGUF/discussions/3

---

👤 **ubergarm** commented on **2025-10-20** at **15:24:00**

Had a recent report on hf suggesting that `-ger` didn't work for them in hybrid CPU + 2x 3090 GPUs and asked for clarification and more details.

> I couldn't get -ger to run as of today without crashing

https://huggingface.co/ubergarm/Ling-1T-GGUF/discussions/6

Hopefully they respond either here or there.

---

👤 **ikawrakow** commented on **2025-10-20** at **19:42:38**

Well, they should crate an issue with detailed description. Grouped expert routing is working just fine for me, and I was even thinking to make it the default and change the flag to `-no-ger`.

---

👤 **magikRUKKOLA** commented on **2025-10-20** at **23:46:07**

@ubergarm 
> Had a recent report on hf suggesting that `-ger` didn't work for them in hybrid CPU + 2x 3090 GPUs and asked for clarification and more details.

I have the same config and getting a segfault with -ger enabled.

It runs fine without -ger:
```
/opt/ik_llama.cpp/ik_llama.cpp/build/bin/llama-server \
    --model /opt/ubergarm/Ling-1T-GGUF/smol-IQ4-KSS/Ling-1T-smol-IQ4_KSS-00001-of-00011.gguf \
    --alias ubergarm/Ling-1T-smol-IQ4_KSS \
    --ctx-size $((64 * 1024)) \
    -b $((16 * 512)) -ub $((16 * 512)) \
    --mlock \
    --temp 0.7 --top-k 0 --top-p 0.95 --min-p 0.1 --repeat-penalty 1.1 \
    -ctk q8_0 -ctv q8_0 \
    -fa \
    -fmoe \
    -amb 512 \
    --split-mode layer \
    --tensor-split 1,1 \
    --main-gpu 1 \
    --override-tensor exps=CPU \
    -no-ooae \
    --n-gpu-layers 99 \
    --threads $(grep ^cpu\\scores /proc/cpuinfo | uniq | awk '{print $4}' | xargs -I{} echo "{}-0" | bc) \
    --host 0.0.0.0 \
    --port 8080 \
    --log-enable \
    --logdir /var/log/ \
    --jinja \
    --special \
    --verbosity 2 \
    --verbose-prompt \
    --reasoning-format auto \
    --prompt-cache "$HOME/.cache/ik_llama.cpp/prompt-cache.bin" --prompt-cache-all \
    --slot-save-path "$HOME/.cache/ik_llama.cpp/slot.bin" \
    --lookup-cache-dynamic "$HOME/.cache/ik_llama.cpp/slot.bin" \
    --keep -1 \
    --slot-prompt-similarity 0.35 \
    --metrics
```

I am creating an issue then?

---

👤 **hunterx2591** commented on **2025-10-21** at **00:03:14**

Hi - 

> Had a recent report on hf suggesting that `-ger` didn't work for them in hybrid CPU + 2x 3090 GPUs and asked for clarification and more details.
> 
> > I couldn't get -ger to run as of today without crashing
> 
> https://huggingface.co/ubergarm/Ling-1T-GGUF/discussions/6
> 
> Hopefully they respond either here or there.

Just to clarify it works when I remove -ngl 99 \ and i can run fully CPU but can't load to the GPUs

version: 3919 (5ae87f6c)
built with cc (Ubuntu 13.3.0-6ubuntu2~24.04) 13.3.0 for x86_64-linux-gnu



Here is the log : xeon@xeon-System-Product-Name:~/ik_llama.cpp/models$ numactl -N 0 -m 0  \
../build/bin/llama-server \
    --model "Ling-1T-IQ2_K-00001-of-00008.gguf"  \
    --alias ubergarm/Ling-1T-GGUF \
     -ctk q8_0 -ctv q8_0 \
    --ctx-size 4096 \
    -fa -fmoe -ger \
       -ub 4096 -b 4096 \
      -ngl 99 \
        -ot exps=CPU \
    --parallel 1 \
    --threads 48 \
     --threads-batch 56 \
     --host 127.0.0.1 \
    --port 8080 \
     --mirostat 2 --mirostat-ent 5 \
    --mirostat-lr 0.1 \
       --no-display-prompt \
       --verbosity 1 \
       --metrics \
       --log-test 
    
ggml_backend_register: registered backend CPU
ggml_cuda_init: GGML_CUDA_FORCE_MMQ:    no
ggml_cuda_init: GGML_CUDA_FORCE_CUBLAS: no
ggml_cuda_init: found 2 CUDA devices:
  Device 0: NVIDIA GeForce RTX 3090, compute capability 8.6, VMM: yes
  Device 1: NVIDIA GeForce RTX 3090, compute capability 8.6, VMM: yes
ggml_backend_register: registered backend CUDA0
ggml_backend_register: registered backend CUDA1
03 Hello World to **both** default output and stderr!
[1761007161] 04 Hello World to stderr!
[1761007161] 05 Hello World TEE with double printing to stderr prevented!
[1761007161] 07 Hello World to stdout!
INFO [                    main] build info | tid="128123511042048" timestamp=1761007162 build=3919 commit="5ae87f6c"
INFO [                    main] system info | tid="128123511042048" timestamp=1761007162 n_threads=48 n_threads_batch=56 total_threads=112 system_info="AVX = 1 | AVX_VNNI = 1 | AVX2 = 1 | AVX512 = 1 | AVX512_VBMI = 1 | AVX512_VNNI = 1 | AVX512_BF16 = 1 | FMA = 1 | NEON = 0 | SVE = 0 | ARM_FMA = 0 | F16C = 1 | FP16_VA = 0 | WASM_SIMD = 0 | BLAS = 1 | SSE3 = 1 | SSSE3 = 1 | VSX = 0 | MATMUL_INT8 = 0 | LLAMAFILE = 1 | "
llama_model_loader: additional 7 GGUFs metadata loaded.
llama_model_loader: loaded meta data with 50 key-value pairs and 1103 tensors from Ling-1T-IQ2_K-00001-of-00008.gguf (version GGUF V3 (latest))
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
llama_model_loader: - kv  16:                          general.file_type u32              = 138
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
llama_model_loader: - kv  48:                                split.count u16              = 8
llama_model_loader: - kv  49:                        split.tensors.count i32              = 1103
llama_model_loader: - type  f32:  473 tensors
llama_model_loader: - type q8_0:  400 tensors
llama_model_loader: - type iq2_k:  152 tensors
llama_model_loader: - type iq3_k:   76 tensors
llama_model_loader: - type iq4_k:    1 tensors
llama_model_loader: - type iq6_k:    1 tensors
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
llm_load_print_meta: model ftype      = IQ2_K - 2.375 bpw
llm_load_print_meta: model params     = 999.705 B
llm_load_print_meta: model size       = 330.923 GiB (2.843 BPW) 
llm_load_print_meta: repeating layers = 329.255 GiB (2.836 BPW, 997.130 B parameters)
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
llm_load_tensors: ggml ctx size =    1.42 MiB
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
llm_load_tensors: offloading 80 repeating layers to GPU
llm_load_tensors: offloading non-repeating layers to GPU
llm_load_tensors: offloaded 81/81 layers to GPU
llm_load_tensors:        CPU buffer size = 40496.61 MiB
llm_load_tensors:        CPU buffer size = 42790.64 MiB
llm_load_tensors:        CPU buffer size = 42824.64 MiB
llm_load_tensors:        CPU buffer size = 42280.64 MiB
llm_load_tensors:        CPU buffer size = 42824.64 MiB
llm_load_tensors:        CPU buffer size = 42824.64 MiB
llm_load_tensors:        CPU buffer size = 42246.64 MiB
llm_load_tensors:        CPU buffer size = 38669.32 MiB
llm_load_tensors:        CPU buffer size =   690.75 MiB
llm_load_tensors:      CUDA0 buffer size = 10082.57 MiB
llm_load_tensors:      CUDA1 buffer size =  9499.55 MiB
....................................................................................................
llama_new_context_with_model: n_ctx      = 4096
llama_new_context_with_model: n_batch    = 4096
llama_new_context_with_model: n_ubatch   = 4096
llama_new_context_with_model: flash_attn = 1
llama_new_context_with_model: mla_attn   = 0
llama_new_context_with_model: attn_max_b = 0
llama_new_context_with_model: fused_moe  = 1
llama_new_context_with_model: grouped er = 1
llama_new_context_with_model: fused_up_gate = 1
llama_new_context_with_model: ser        = -1, 0
llama_new_context_with_model: freq_base  = 600000.0
llama_new_context_with_model: freq_scale = 1
llama_kv_cache_init:      CUDA0 KV buffer size =   340.02 MiB
llama_kv_cache_init:      CUDA1 KV buffer size =   340.02 MiB
llama_new_context_with_model: KV self size  =  680.00 MiB, K (q8_0):  340.00 MiB, V (q8_0):  340.00 MiB
llama_new_context_with_model:  CUDA_Host  output buffer size =     1.20 MiB
llama_new_context_with_model: pipeline parallelism enabled (n_copies=1)
ggml_gallocr_reserve_n: reallocating CUDA0 buffer from size 0.00 MiB to 4008.14 MiB
ggml_gallocr_reserve_n: reallocating CUDA1 buffer from size 0.00 MiB to 2584.00 MiB
ggml_gallocr_reserve_n: reallocating CUDA_Host buffer from size 0.00 MiB to 160.05 MiB
llama_new_context_with_model:      CUDA0 compute buffer size =  4008.14 MiB
llama_new_context_with_model:      CUDA1 compute buffer size =  2584.00 MiB
llama_new_context_with_model:  CUDA_Host compute buffer size =   160.05 MiB
llama_new_context_with_model: graph nodes  = 3369
llama_new_context_with_model: graph splits = 195
XXXXXXXXXXXXXXXXXXXXX Setting only active experts offload
ggml_backend_sched_alloc_splits: failed to allocate graph, reserving (backend_ids_changed = 1)
Segmentation fault (core dumped)
xeon@xeon-System-Product-Name:~/ik_llama.cpp/models$

---

👤 **magikRUKKOLA** commented on **2025-10-21** at **01:30:27**

Test results for 12C CPU + 2xRTX3090 for ubergarm/Ling-1T-GGUF/smol-IQ4-KSS with -ger disabled:

```
llm_load_tensors: offloading 80 repeating layers to GPU
llm_load_tensors: offloading non-repeating layers to GPU
llm_load_tensors: offloaded 81/81 layers to GPU
llm_load_tensors:        CPU buffer size = 42986.73 MiB
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
llm_load_tensors:        CPU buffer size =   690.75 MiB
llm_load_tensors:      CUDA0 buffer size =  6695.50 MiB
llm_load_tensors:      CUDA1 buffer size =  8006.87 MiB
....................................................................................................
llama_new_context_with_model: n_ctx      = 65536
llama_new_context_with_model: n_batch    = 8192
llama_new_context_with_model: n_ubatch   = 8192
llama_new_context_with_model: flash_attn = 1
llama_new_context_with_model: mla_attn   = 0
llama_new_context_with_model: attn_max_b = 512
llama_new_context_with_model: fused_moe  = 1
llama_new_context_with_model: grouped er = 0
llama_new_context_with_model: fused_up_gate = 1
llama_new_context_with_model: ser        = -1, 0
llama_new_context_with_model: freq_base  = 600000.0
llama_new_context_with_model: freq_scale = 1
llama_kv_cache_init:      CUDA0 KV buffer size =  4896.02 MiB
llama_kv_cache_init:      CUDA1 KV buffer size =  5984.02 MiB
llama_new_context_with_model: KV self size  = 10880.00 MiB, K (q8_0): 5440.00 MiB, V (q8_0): 5440.00 MiB
llama_new_context_with_model:  CUDA_Host  output buffer size =     0.60 MiB
llama_new_context_with_model: pipeline parallelism enabled (n_copies=1)
llama_new_context_with_model:      CUDA0 compute buffer size =  8220.03 MiB
llama_new_context_with_model:      CUDA1 compute buffer size =  5168.00 MiB
llama_new_context_with_model:  CUDA_Host compute buffer size =  1280.09 MiB
llama_new_context_with_model: graph nodes  = 3445
llama_new_context_with_model: graph splits = 199

main: n_kv_max = 65536, n_batch = 8192, n_ubatch = 8192, flash_attn = 1, n_gpu_layers = 99, n_threads = 12, n_threads_batch = 12

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  8192 |   2048 |      0 |   40.398 |   202.78 |  477.660 |     4.29 |
|  8192 |   2048 |   8192 |   41.648 |   196.70 |  497.290 |     4.12 |
|  8192 |   2048 |  16384 |   42.807 |   191.37 |  513.179 |     3.99 |
|  8192 |   2048 |  24576 |   44.050 |   185.97 |  528.828 |     3.87 |

```

[EDIT]: GPU usage: 23.8GB each.

---

👤 **magikRUKKOLA** commented on **2025-10-21** at **01:58:54**

As related to the segfault.

```
Thread 1 "llama-server" received signal SIGSEGV, Segmentation fault.
0x00007fffe11b3d36 in cuda_bailingmoev2_experts(ggml_backend_cuda_context&, ggml_tensor*, ggml_tensor*) ()
   from /opt/ik_llama.cpp/ik_llama.cpp/build/ggml/src/libggml.so
```

```
(gdb) thread apply all bt

Thread 6 (Thread 0x7f87e7fff000 (LWP 2402202) "cuda-EvtHandlr"):
#0  0x00007fffe0aa49ee in ?? () from /lib/x86_64-linux-gnu/libc.so.6
#1  0x00007fffe0a99668 in ?? () from /lib/x86_64-linux-gnu/libc.so.6
#2  0x00007fffe0a996ad in ?? () from /lib/x86_64-linux-gnu/libc.so.6
#3  0x00007fffe0b0d9c6 in poll () from /lib/x86_64-linux-gnu/libc.so.6
#4  0x00007fffd731cb37 in ?? () from /lib/x86_64-linux-gnu/libcuda.so.1
#5  0x00007fffd740bd77 in ?? () from /lib/x86_64-linux-gnu/libcuda.so.1
#6  0x00007fffd7308883 in ?? () from /lib/x86_64-linux-gnu/libcuda.so.1
#7  0x00007fffe0a9cb7b in ?? () from /lib/x86_64-linux-gnu/libc.so.6
#8  0x00007fffe0b1a7b8 in ?? () from /lib/x86_64-linux-gnu/libc.so.6

Thread 5 (Thread 0x7f89a4ddc000 (LWP 2402201) "llama-server"):
#0  0x00007fffe0aa49ee in ?? () from /lib/x86_64-linux-gnu/libc.so.6
#1  0x00007fffe0a99668 in ?? () from /lib/x86_64-linux-gnu/libc.so.6
#2  0x00007fffe0a99c9c in ?? () from /lib/x86_64-linux-gnu/libc.so.6
#3  0x00007fffe0a9c31d in pthread_cond_timedwait () from /lib/x86_64-linux-gnu/libc.so.6
#4  0x00007fffd7269b5a in ?? () from /lib/x86_64-linux-gnu/libcuda.so.1
#5  0x00007fffd7308883 in ?? () from /lib/x86_64-linux-gnu/libcuda.so.1
#6  0x00007fffe0a9cb7b in ?? () from /lib/x86_64-linux-gnu/libc.so.6
#7  0x00007fffe0b1a7b8 in ?? () from /lib/x86_64-linux-gnu/libc.so.6

Thread 4 (Thread 0x7f89a67fe000 (LWP 2402200) "cuda-EvtHandlr"):
#0  0x00007fffe0aa49ee in ?? () from /lib/x86_64-linux-gnu/libc.so.6
#1  0x00007fffe0a99668 in ?? () from /lib/x86_64-linux-gnu/libc.so.6
#2  0x00007fffe0a996ad in ?? () from /lib/x86_64-linux-gnu/libc.so.6
#3  0x00007fffe0b0d9c6 in poll () from /lib/x86_64-linux-gnu/libc.so.6
#4  0x00007fffd731cb37 in ?? () from /lib/x86_64-linux-gnu/libcuda.so.1
#5  0x00007fffd740bd77 in ?? () from /lib/x86_64-linux-gnu/libcuda.so.1
#6  0x00007fffd7308883 in ?? () from /lib/x86_64-linux-gnu/libcuda.so.1
#7  0x00007fffe0a9cb7b in ?? () from /lib/x86_64-linux-gnu/libc.so.6
#8  0x00007fffe0b1a7b8 in ?? () from /lib/x86_64-linux-gnu/libc.so.6

Thread 3 (Thread 0x7f89a6fff000 (LWP 2402199) "llama-server"):
#0  0x00007fffe0aa49ee in ?? () from /lib/x86_64-linux-gnu/libc.so.6
#1  0x00007fffe0a99668 in ?? () from /lib/x86_64-linux-gnu/libc.so.6
#2  0x00007fffe0a99c9c in ?? () from /lib/x86_64-linux-gnu/libc.so.6
#3  0x00007fffe0a9c31d in pthread_cond_timedwait () from /lib/x86_64-linux-gnu/libc.so.6
#4  0x00007fffd7269b5a in ?? () from /lib/x86_64-linux-gnu/libcuda.so.1
#5  0x00007fffd7308883 in ?? () from /lib/x86_64-linux-gnu/libcuda.so.1
#6  0x00007fffe0a9cb7b in ?? () from /lib/x86_64-linux-gnu/libc.so.6
#7  0x00007fffe0b1a7b8 in ?? () from /lib/x86_64-linux-gnu/libc.so.6

Thread 2 (Thread 0x7fffb13ff000 (LWP 2402120) "cuda00001400006"):
#0  0x00007fffe0aa49ee in ?? () from /lib/x86_64-linux-gnu/libc.so.6
#1  0x00007fffe0a99668 in ?? () from /lib/x86_64-linux-gnu/libc.so.6
#2  0x00007fffe0a996ad in ?? () from /lib/x86_64-linux-gnu/libc.so.6
#3  0x00007fffe0b0d9c6 in poll () from /lib/x86_64-linux-gnu/libc.so.6
#4  0x00007fffd731cb37 in ?? () from /lib/x86_64-linux-gnu/libcuda.so.1
#5  0x00007fffd740bd77 in ?? () from /lib/x86_64-linux-gnu/libcuda.so.1
#6  0x00007fffd7308883 in ?? () from /lib/x86_64-linux-gnu/libcuda.so.1
#7  0x00007fffe0a9cb7b in ?? () from /lib/x86_64-linux-gnu/libc.so.6
#8  0x00007fffe0b1a7b8 in ?? () from /lib/x86_64-linux-gnu/libc.so.6

Thread 1 (Thread 0x7ffff7d3a000 (LWP 2402116) "llama-server"):
#0  0x00007fffe11b3d36 in cuda_bailingmoev2_experts(ggml_backend_cuda_context&, ggml_tensor*, ggml_tensor*) () from /opt/ik_llama.cpp/ik_llama.cpp/build/ggml/src/libggml.so
#1  0x00007fffe1376118 in ggml_backend_cuda_graph_compute(ggml_backend*, ggml_cgraph*) () from /opt/ik_llama.cpp/ik_llama.cpp/build/ggml/src/libggml.so
#2  0x00007fffe116ffa2 in ggml_backend_sched_graph_compute_async () from /opt/ik_llama.cpp/ik_llama.cpp/build/ggml/src/libggml.so
#3  0x00007ffff7e6d28c in llama_decode () from /opt/ik_llama.cpp/ik_llama.cpp/build/src/libllama.so
#4  0x00005555556fab08 in llama_init_from_gpt_params(gpt_params&) ()
#5  0x0000555555610d63 in server_context::load_model(gpt_params const&) ()
#6  0x000055555559fd70 in main ()

```

```
(gdb) bt full
#0  0x00007fffe11b3d36 in cuda_bailingmoev2_experts(ggml_backend_cuda_context&, ggml_tensor*, ggml_tensor*) ()
   from /opt/ik_llama.cpp/ik_llama.cpp/build/ggml/src/libggml.so
No symbol table info available.
#1  0x00007fffe1376118 in ggml_backend_cuda_graph_compute(ggml_backend*, ggml_cgraph*) ()
   from /opt/ik_llama.cpp/ik_llama.cpp/build/ggml/src/libggml.so
No symbol table info available.
#2  0x00007fffe116ffa2 in ggml_backend_sched_graph_compute_async ()
   from /opt/ik_llama.cpp/ik_llama.cpp/build/ggml/src/libggml.so
No symbol table info available.
#3  0x00007ffff7e6d28c in llama_decode () from /opt/ik_llama.cpp/ik_llama.cpp/build/src/libllama.so
No symbol table info available.
#4  0x00005555556fab08 in llama_init_from_gpt_params(gpt_params&) ()
No symbol table info available.
#5  0x0000555555610d63 in server_context::load_model(gpt_params const&) ()
No symbol table info available.
#6  0x000055555559fd70 in main ()
No symbol table info available.
```

```
(gdb) info registers
rax            0x1                 1
rbx            0x7f93c64580f0      140272663363824
rcx            0x1                 1
rdx            0x1                 1
rsi            0x100               256
rdi            0x0                 0
rbp            0x7fffffffa6d0      0x7fffffffa6d0
rsp            0x7fffffffa520      0x7fffffffa520
r8             0x555555ebe610      93825002104336
r9             0x0                 0
r10            0x7fffffffa6f0      140737488332528
r11            0x7fffe11b3c90      140736970046608
r12            0x0                 0
r13            0x7f93c64586b0      140272663365296
r14            0x7f93c6458820      140272663365664
r15            0x555556c7cc80      93825016515712
rip            0x7fffe11b3d36      0x7fffe11b3d36 <cuda_bailingmoev2_experts(ggml_backend_cuda_context&, ggml_tensor*, ggml_tensor*)+166>
eflags         0x10202             [ IF RF ]
cs             0x33                51
ss             0x2b                43
ds             0x0                 0
es             0x0                 0
fs             0x0                 0
gs             0x0                 0
fs_base        0x7ffff7d3a000      140737351229440
gs_base        0x0                 0
```

```
(gdb) x/10i $pc
=> 0x7fffe11b3d36 <_Z25cuda_bailingmoev2_expertsR25ggml_backend_cuda_contextP11ggml_tensorS2_+166>:
    cmpq   $0x1,0x18(%r12)
   0x7fffe11b3d3c <_Z25cuda_bailingmoev2_expertsR25ggml_backend_cuda_contextP11ggml_tensorS2_+172>:
    jne    0x7fffe11b45cf <_Z25cuda_bailingmoev2_expertsR25ggml_backend_cuda_contextP11ggml_tensorS2_+2367>
   0x7fffe11b3d42 <_Z25cuda_bailingmoev2_expertsR25ggml_backend_cuda_contextP11ggml_tensorS2_+178>:
    mov    -0x168(%rbp),%rax
   0x7fffe11b3d49 <_Z25cuda_bailingmoev2_expertsR25ggml_backend_cuda_contextP11ggml_tensorS2_+185>:
    mov    0x10(%rax),%rax
   0x7fffe11b3d4d <_Z25cuda_bailingmoev2_expertsR25ggml_backend_cuda_contextP11ggml_tensorS2_+189>:
    cmp    %rax,0x10(%r12)
   0x7fffe11b3d52 <_Z25cuda_bailingmoev2_expertsR25ggml_backend_cuda_contextP11ggml_tensorS2_+194>:
    jne    0x7fffe11b45ae <_Z25cuda_bailingmoev2_expertsR25ggml_backend_cuda_contextP11ggml_tensorS2_+2334>
   0x7fffe11b3d58 <_Z25cuda_bailingmoev2_expertsR25ggml_backend_cuda_contextP11ggml_tensorS2_+200>:
    mov    -0x190(%rbp),%rcx
   0x7fffe11b3d5f <_Z25cuda_bailingmoev2_expertsR25ggml_backend_cuda_contextP11ggml_tensorS2_+207>:
    movslq -0x180(%rbp),%rax
   0x7fffe11b3d66 <_Z25cuda_bailingmoev2_expertsR25ggml_backend_cuda_contextP11ggml_tensorS2_+214>:
    cmp    0x18(%rcx),%rax
   0x7fffe11b3d6a <_Z25cuda_bailingmoev2_expertsR25ggml_backend_cuda_contextP11ggml_tensorS2_+218>:
    jne    0x7fffe11b46bc <_Z25cuda_bailingmoev2_expertsR25ggml_backend_cuda_contextP11ggml_tensorS2_+2604>
```

[EDIT]:

so its null-pointer dereference.  the following patch proves (that is, no crash occurs but the data is a garbage) that something is wrong with topk.  the src is placed with 0x18 offset, right?

ggml/src/ggml-cuda/argsort.cu
```
476 void cuda_bailingmoev2_experts(ggml_backend_cuda_context & ctx, ggml_tensor * dst, ggml_tensor * topk) {  
477     if (!topk || !dst || !topk->src[0] || !topk->src[0]->src[0] ||                                                      
478         !topk->src[0]->src[0]->src[0] || !topk->src[0]->src[1]) {                                                       
479         return;                                                                                                         
480     } 
```