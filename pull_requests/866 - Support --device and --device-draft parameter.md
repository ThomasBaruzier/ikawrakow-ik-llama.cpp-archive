## 🔀 [Pull Request #866](https://github.com/ikawrakow/ik_llama.cpp/pull/866) - Support --device and --device-draft parameter

| **Author** | `firecoperana` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `fcp/device` |
| **Target Branch** | `main` |
| **Created** | 2025-10-25 |
| **Updated** | 2025-10-29 |
| **Merged** | 2025-10-27 |
| **Assignees** | `firecoperana` |

---

## 📄 Description

This PR adds support for --device and --device-draft, which can accept string like none, CUDA0,CUDA1..., Vulkan0, Vulkan1,... and RPC[ip:port]. Using none is same as setting gpu-layer to 0. The order of GPU used in this parameter is used for tensor split. The way I implemented it is different from mainline, so it could have potential bugs. 

Minor changes:
1. add --n-gpu-layers-draft. Earlier it only accepts -ngld and --gpu-layers-draft.
2. Change some messages in speculative decoding to only print in debug build
3. Change LLAMA_LOG_DEBUG to not print message in release build. It's kind of a hack, and mainline's approach seems better, but needs more changes.

---

## 💬 Conversation

👤 **ikawrakow** started a conversation on `src/llama.cpp` on **2025-10-25** at **16:00:13**

```c++
static bool item_in_list(const std::vector<std::string> & devices, const char* name) {
    for (auto & device : devices) {
        if (striequals(device.c_str(), name)) {
            return true;
        }
    }
    return false;
}
```

---

👤 **ikawrakow** started a conversation on `src/llama.cpp` on **2025-10-25** at **16:08:58**

`const std::string &`

---

👤 **ikawrakow** started a conversation on `common/common.cpp` on **2025-10-25** at **16:14:52**

Are we getting a memory leak here? Not that it is serious, but still. Or maybe I just don't see where `mparams.devices` is freed.

---

👤 **ikawrakow** started a conversation on `common/common.cpp` on **2025-10-25** at **16:22:25**

This is just a general comment on command line arguments for draft models. Currently we can control a very limited subset of parameters for draft models via command line arguments. Hence I was wondering if it wouldn't be better to take an approach similar to the way compilers pass arguments to the linker. Something along the lines
```
--draft,arg,[value]
```
One would then first split the command line arguments into two sets, one for the main model and one for the draft (detected via the `--draft`), and then process the two lists creating two sets of parameters, one for the main model and one for the draft. In that way one can set everything for the draft that can be set for the main model. 

But I guess, maybe better to leave it for a future PR.

---

👤 **ikawrakow** reviewed this pull request 💬 on **2025-10-25** at **16:24:05**

Let's have some other people test this before merging.

---

👤 **MRGRD56** commented on **2025-10-27** at **13:28:14**

It seems the `--device` parameter isn't really working as expected. Or I might be doing something wrong.

---

#### My hardware:

- 256 GB RAM
- CUDA0: RTX 5090 (32 GB)
- CUDA1: RTX 4070 Ti Super (16 GB)

---

When I set `CUDA_VISIBLE_DEVICES=0`, the model loads fine:

```ps1
PowerShell 7.5.4
PS C:\Users\SU> $env:CUDA_VISIBLE_DEVICES=0; C:\apps\ik_llama.cpp\build\bin\Release\llama-server --port 15922 `
>>       --model "E:\AI\LLM\gguf\unsloth\Qwen3-235B-A22B-Instruct-2507-GGUF\Qwen3-235B-A22B-Instruct-2507-UD-Q4_K_XL-00001-of-00003.gguf" `
>>       -rtr `
>>       --ctx-size 16384 `
>>       -ctk q8_0 -ctv q8_0 `
>>       -amb 512 `
>>       --n-gpu-layers 99 `
>>       -mg 0 `
>>       -ot "blk\.(0?[0-9]|1[0-6])\.ffn_.*_exps.=CUDA0" `
>>       -ot ".ffn_.*_exps.=CPU" `
>>       --parallel 1 `
>>       -tb 30 -t 15 `
>>       --jinja --reasoning-budget 0 --no-mmap
ggml_cuda_init: GGML_CUDA_FORCE_MMQ:    no
ggml_cuda_init: GGML_CUDA_FORCE_CUBLAS: no
ggml_cuda_init: found 1 CUDA devices:
  Device 0: NVIDIA GeForce RTX 5090, compute capability 12.0, VMM: yes
INFO [                    main] build info | tid="39812" timestamp=1761570830 build=3933 commit="7a921655"
INFO [                    main] system info | tid="39812" timestamp=1761570830 n_threads=15 n_threads_batch=30 total_threads=32 system_info="AVX = 1 | AVX_VNNI = 0 | AVX2 = 1 | AVX512 = 1 | AVX512_VBMI = 0 | AVX512_VNNI = 0 | AVX512_BF16 = 0 | FMA = 1 | NEON = 0 | SVE = 0 | ARM_FMA = 0 | F16C = 1 | FP16_VA = 0 | WASM_SIMD = 0 | BLAS = 1 | SSE3 = 1 | SSSE3 = 1 | VSX = 0 | MATMUL_INT8 = 0 | LLAMAFILE = 1 | "
CUDA0: using device CUDA0 - 30841 MiB free
llama_model_loader: additional 2 GGUFs metadata loaded.
llama_model_loader: loaded meta data with 48 key-value pairs and 1131 tensors from E:\AI\LLM\gguf\unsloth\Qwen3-235B-A22B-Instruct-2507-GGUF\Qwen3-235B-A22B-Instruct-2507-UD-Q4_K_XL-00001-of-00003.gguf (version GGUF V3 (latest))
llama_model_loader: Dumping metadata keys/values. Note: KV overrides do not apply in this output.
llama_model_loader: - kv   0:                       general.architecture str              = qwen3moe
llama_model_loader: - kv   1:                               general.type str              = model
llama_model_loader: - kv   2:                               general.name str              = Qwen3-235B-A22B-Instruct-2507
llama_model_loader: - kv   3:                            general.version str              = 2507
llama_model_loader: - kv   4:                           general.finetune str              = Instruct
llama_model_loader: - kv   5:                           general.basename str              = Qwen3-235B-A22B-Instruct-2507
llama_model_loader: - kv   6:                       general.quantized_by str              = Unsloth
llama_model_loader: - kv   7:                         general.size_label str              = 235B-A22B
llama_model_loader: - kv   8:                            general.license str              = apache-2.0
llama_model_loader: - kv   9:                       general.license.link str              = https://huggingface.co/Qwen/Qwen3-235...
llama_model_loader: - kv  10:                           general.repo_url str              = https://huggingface.co/unsloth
llama_model_loader: - kv  11:                   general.base_model.count u32              = 1
llama_model_loader: - kv  12:                  general.base_model.0.name str              = Qwen3 235B A22B Instruct 2507
llama_model_loader: - kv  13:               general.base_model.0.version str              = 2507
llama_model_loader: - kv  14:          general.base_model.0.organization str              = Qwen
llama_model_loader: - kv  15:              general.base_model.0.repo_url str              = https://huggingface.co/Qwen/Qwen3-235...
llama_model_loader: - kv  16:                               general.tags arr[str,2]       = ["unsloth", "text-generation"]
llama_model_loader: - kv  17:                       qwen3moe.block_count u32              = 94
llama_model_loader: - kv  18:                    qwen3moe.context_length u32              = 262144
llama_model_loader: - kv  19:                  qwen3moe.embedding_length u32              = 4096
llama_model_loader: - kv  20:               qwen3moe.feed_forward_length u32              = 12288
llama_model_loader: - kv  21:              qwen3moe.attention.head_count u32              = 64
llama_model_loader: - kv  22:           qwen3moe.attention.head_count_kv u32              = 4
llama_model_loader: - kv  23:                    qwen3moe.rope.freq_base f32              = 5000000.000000
llama_model_loader: - kv  24:  qwen3moe.attention.layer_norm_rms_epsilon f32              = 0.000001
llama_model_loader: - kv  25:                 qwen3moe.expert_used_count u32              = 8
llama_model_loader: - kv  26:              qwen3moe.attention.key_length u32              = 128
llama_model_loader: - kv  27:            qwen3moe.attention.value_length u32              = 128
llama_model_loader: - kv  28:                      qwen3moe.expert_count u32              = 128
llama_model_loader: - kv  29:        qwen3moe.expert_feed_forward_length u32              = 1536
llama_model_loader: - kv  30:                       tokenizer.ggml.model str              = gpt2
llama_model_loader: - kv  31:                         tokenizer.ggml.pre str              = qwen2
llama_model_loader: - kv  32:                      tokenizer.ggml.tokens arr[str,151936]  = ["!", "\"", "#", "$", "%", "&", "'", ...
llama_model_loader: - kv  33:                  tokenizer.ggml.token_type arr[i32,151936]  = [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, ...
llama_model_loader: - kv  34:                      tokenizer.ggml.merges arr[str,151387]  = ["─а ─а", "─а─а ─а─а", "i n", "─а t",...
llama_model_loader: - kv  35:                tokenizer.ggml.eos_token_id u32              = 151645
llama_model_loader: - kv  36:            tokenizer.ggml.padding_token_id u32              = 151654
llama_model_loader: - kv  37:               tokenizer.ggml.add_bos_token bool             = false
llama_model_loader: - kv  38:                    tokenizer.chat_template str              = {%- if tools %}\n    {{- '<|im_start|>...
llama_model_loader: - kv  39:               general.quantization_version u32              = 2
llama_model_loader: - kv  40:                          general.file_type u32              = 15
llama_model_loader: - kv  41:                      quantize.imatrix.file str              = Qwen3-235B-A22B-Instruct-2507-GGUF/im...
llama_model_loader: - kv  42:                   quantize.imatrix.dataset str              = unsloth_calibration_Qwen3-235B-A22B-I...
llama_model_loader: - kv  43:             quantize.imatrix.entries_count u32              = 745
llama_model_loader: - kv  44:              quantize.imatrix.chunks_count u32              = 693
llama_model_loader: - kv  45:                                   split.no u16              = 0
llama_model_loader: - kv  46:                        split.tensors.count i32              = 1131
llama_model_loader: - kv  47:                                split.count u16              = 3
llama_model_loader: - type  f32:  471 tensors
llama_model_loader: - type q4_K:  609 tensors
llama_model_loader: - type q5_K:   40 tensors
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
llm_load_print_meta: model ftype      = Q4_K - Medium
llm_load_print_meta: model params     = 235.094 B
llm_load_print_meta: model size       = 125.098 GiB (4.571 BPW)
llm_load_print_meta: repeating layers = 124.297 GiB (4.566 BPW, 233.849 B parameters)
llm_load_print_meta: general.name     = Qwen3-235B-A22B-Instruct-2507
llm_load_print_meta: n_ff_exp         = 1536
print_info: vocab type       = BPE
print_info: n_vocab          = 151936
print_info: n_merges         = 151387
print_info: BOS token        = 11 ','
print_info: EOS token        = 151645 '<|im_end|>'
print_info: EOT token        = 151645 '<|im_end|>'
print_info: PAD token        = 151654 '<|vision_pad|>'
print_info: LF token         = 198 '─К'
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
Tensor blk.0.ffn_gate_exps.weight buffer type overriden to CUDA0
Tensor blk.0.ffn_down_exps.weight buffer type overriden to CUDA0
Tensor blk.0.ffn_up_exps.weight buffer type overriden to CUDA0
Tensor blk.1.ffn_gate_exps.weight buffer type overriden to CUDA0
Tensor blk.1.ffn_down_exps.weight buffer type overriden to CUDA0
Tensor blk.1.ffn_up_exps.weight buffer type overriden to CUDA0
Tensor blk.2.ffn_gate_exps.weight buffer type overriden to CUDA0
Tensor blk.2.ffn_down_exps.weight buffer type overriden to CUDA0
Tensor blk.2.ffn_up_exps.weight buffer type overriden to CUDA0
Tensor blk.3.ffn_gate_exps.weight buffer type overriden to CUDA0
Tensor blk.3.ffn_down_exps.weight buffer type overriden to CUDA0
Tensor blk.3.ffn_up_exps.weight buffer type overriden to CUDA0
Tensor blk.4.ffn_gate_exps.weight buffer type overriden to CUDA0
Tensor blk.4.ffn_down_exps.weight buffer type overriden to CUDA0
Tensor blk.4.ffn_up_exps.weight buffer type overriden to CUDA0
Tensor blk.5.ffn_gate_exps.weight buffer type overriden to CUDA0
Tensor blk.5.ffn_down_exps.weight buffer type overriden to CUDA0
Tensor blk.5.ffn_up_exps.weight buffer type overriden to CUDA0
Tensor blk.6.ffn_gate_exps.weight buffer type overriden to CUDA0
Tensor blk.6.ffn_down_exps.weight buffer type overriden to CUDA0
Tensor blk.6.ffn_up_exps.weight buffer type overriden to CUDA0
Tensor blk.7.ffn_gate_exps.weight buffer type overriden to CUDA0
Tensor blk.7.ffn_down_exps.weight buffer type overriden to CUDA0
Tensor blk.7.ffn_up_exps.weight buffer type overriden to CUDA0
Tensor blk.8.ffn_gate_exps.weight buffer type overriden to CUDA0
Tensor blk.8.ffn_down_exps.weight buffer type overriden to CUDA0
Tensor blk.8.ffn_up_exps.weight buffer type overriden to CUDA0
Tensor blk.9.ffn_gate_exps.weight buffer type overriden to CUDA0
Tensor blk.9.ffn_down_exps.weight buffer type overriden to CUDA0
Tensor blk.9.ffn_up_exps.weight buffer type overriden to CUDA0
Tensor blk.10.ffn_gate_exps.weight buffer type overriden to CUDA0
Tensor blk.10.ffn_down_exps.weight buffer type overriden to CUDA0
Tensor blk.10.ffn_up_exps.weight buffer type overriden to CUDA0
Tensor blk.11.ffn_gate_exps.weight buffer type overriden to CUDA0
Tensor blk.11.ffn_down_exps.weight buffer type overriden to CUDA0
Tensor blk.11.ffn_up_exps.weight buffer type overriden to CUDA0
Tensor blk.12.ffn_gate_exps.weight buffer type overriden to CUDA0
Tensor blk.12.ffn_down_exps.weight buffer type overriden to CUDA0
Tensor blk.12.ffn_up_exps.weight buffer type overriden to CUDA0
Tensor blk.13.ffn_gate_exps.weight buffer type overriden to CUDA0
Tensor blk.13.ffn_down_exps.weight buffer type overriden to CUDA0
Tensor blk.13.ffn_up_exps.weight buffer type overriden to CUDA0
Tensor blk.14.ffn_gate_exps.weight buffer type overriden to CUDA0
Tensor blk.14.ffn_down_exps.weight buffer type overriden to CUDA0
Tensor blk.14.ffn_up_exps.weight buffer type overriden to CUDA0
Tensor blk.15.ffn_gate_exps.weight buffer type overriden to CUDA0
Tensor blk.15.ffn_down_exps.weight buffer type overriden to CUDA0
Tensor blk.15.ffn_up_exps.weight buffer type overriden to CUDA0
Tensor blk.16.ffn_gate_exps.weight buffer type overriden to CUDA0
Tensor blk.16.ffn_down_exps.weight buffer type overriden to CUDA0
Tensor blk.16.ffn_up_exps.weight buffer type overriden to CUDA0
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
llm_load_tensors:  CUDA_Host buffer size =   333.84 MiB
llm_load_tensors:        CPU buffer size = 100752.00 MiB
llm_load_tensors:      CUDA0 buffer size = 27014.90 MiB
....................................................................................................
============ Repacked 231 tensors
llama_new_context_with_model: n_ctx         = 16384
llama_new_context_with_model: n_batch       = 2048
llama_new_context_with_model: n_ubatch      = 512
llama_new_context_with_model: flash_attn    = 1
llama_new_context_with_model: mla_attn      = 0
llama_new_context_with_model: attn_max_b    = 512
llama_new_context_with_model: fused_moe     = 1
llama_new_context_with_model: grouped er    = 0
llama_new_context_with_model: fused_up_gate = 1
llama_new_context_with_model: fused_mmad    = 1
llama_new_context_with_model: ser           = -1, 0
llama_new_context_with_model: freq_base     = 5000000.0
llama_new_context_with_model: freq_scale    = 1
llama_kv_cache_init:      CUDA0 KV buffer size =  1598.05 MiB
llama_new_context_with_model: KV self size  = 1598.00 MiB, K (q8_0):  799.00 MiB, V (q8_0):  799.00 MiB
llama_new_context_with_model:  CUDA_Host  output buffer size =     1.16 MiB
llama_new_context_with_model:      CUDA0 compute buffer size =   312.75 MiB
llama_new_context_with_model:  CUDA_Host compute buffer size =   104.01 MiB
llama_new_context_with_model: graph nodes  = 3483
llama_new_context_with_model: graph splits = 156
XXXXXXXXXXXXXXXXXXXXX Setting only active experts offload
INFO [                    init] initializing slots | tid="39812" timestamp=1761570935 n_slots=1
INFO [                    init] new slot | tid="39812" timestamp=1761570935 id_slot=0 n_ctx_slot=16384
INFO [                    main] model loaded | tid="39812" timestamp=1761570935
INFO [                    main] chat template | tid="39812" timestamp=1761570935 chat_template="{%- if tools %}\n    {{- '<|im_start|>system\\n' }}\n    {%- if messages[0].role == 'system' %}\n        {{- messages[0].content + '\\n\\n' }}\n    {%- endif %}\n    {{- \"# Tools\\n\\nYou may call one or more functions to assist with the user query.\\n\\nYou are provided with function signatures within <tools></tools> XML tags:\\n<tools>\" }}\n    {%- for tool in tools %}\n        {{- \"\\n\" }}\n        {{- tool | tojson }}\n    {%- endfor %}\n    {{- \"\\n</tools>\\n\\nFor each function call, return a json object with function name and arguments within <tool_call></tool_call> XML tags:\\n<tool_call>\\n{\\\"name\\\": <function-name>, \\\"arguments\\\": <args-json-object>}\\n</tool_call><|im_end|>\\n\" }}\n{%- else %}\n    {%- if messages[0].role == 'system' %}\n        {{- '<|im_start|>system\\n' + messages[0].content + '<|im_end|>\\n' }}\n    {%- endif %}\n{%- endif %}\n{%- set ns = namespace(multi_step_tool=true, last_query_index=messages|length - 1) %}\n{%- for message in messages[::-1] %}\n    {%- set index = (messages|length - 1) - loop.index0 %}\n    {%- if ns.multi_step_tool and message.role == \"user\" and message.content is string and not(message.content.startswith('<tool_response>') and message.content.endswith('</tool_response>')) %}\n        {%- set ns.multi_step_tool = false %}\n        {%- set ns.last_query_index = index %}\n    {%- endif %}\n{%- endfor %}\n{%- for message in messages %}\n    {%- if message.content is string %}\n        {%- set content = message.content %}\n    {%- else %}\n        {%- set content = '' %}\n    {%- endif %}\n    {%- if (message.role == \"user\") or (message.role == \"system\" and not loop.first) %}\n        {{- '<|im_start|>' + message.role + '\\n' + content + '<|im_end|>' + '\\n' }}\n    {%- elif message.role == \"assistant\" %}\n        {%- set reasoning_content = '' %}\n        {%- if message.reasoning_content is string %}\n            {%- set reasoning_content = message.reasoning_content %}\n        {%- else %}\n            {%- if '</think>' in content %}\n                {%- set reasoning_content = content.split('</think>')[0].rstrip('\\n').split('<think>')[-1].lstrip('\\n') %}\n                {%- set content = content.split('</think>')[-1].lstrip('\\n') %}\n            {%- endif %}\n        {%- endif %}\n        {%- if loop.index0 > ns.last_query_index %}\n            {%- if loop.last or (not loop.last and reasoning_content) %}\n                {{- '<|im_start|>' + message.role + '\\n<think>\\n' + reasoning_content.strip('\\n') + '\\n</think>\\n\\n' + content.lstrip('\\n') }}\n            {%- else %}\n                {{- '<|im_start|>' + message.role + '\\n' + content }}\n            {%- endif %}\n        {%- else %}\n            {{- '<|im_start|>' + message.role + '\\n' + content }}\n        {%- endif %}\n        {%- if message.tool_calls %}\n            {%- for tool_call in message.tool_calls %}\n                {%- if (loop.first and content) or (not loop.first) %}\n                    {{- '\\n' }}\n                {%- endif %}\n                {%- if tool_call.function %}\n                    {%- set tool_call = tool_call.function %}\n                {%- endif %}\n                {{- '<tool_call>\\n{\"name\": \"' }}\n                {{- tool_call.name }}\n                {{- '\", \"arguments\": ' }}\n                {%- if tool_call.arguments is string %}\n                    {{- tool_call.arguments }}\n                {%- else %}\n                    {{- tool_call.arguments | tojson }}\n                {%- endif %}\n                {{- '}\\n</tool_call>' }}\n            {%- endfor %}\n        {%- endif %}\n        {{- '<|im_end|>\\n' }}\n    {%- elif message.role == \"tool\" %}\n        {%- if loop.first or (messages[loop.index0 - 1].role != \"tool\") %}\n            {{- '<|im_start|>user' }}\n        {%- endif %}\n        {{- '\\n<tool_response>\\n' }}\n        {{- content }}\n        {{- '\\n</tool_response>' }}\n        {%- if loop.last or (messages[loop.index0 + 1].role != \"tool\") %}\n            {{- '<|im_end|>\\n' }}\n        {%- endif %}\n    {%- endif %}\n{%- endfor %}\n{%- if add_generation_prompt %}\n    {{- '<|im_start|>assistant\\n' }}\n{%- endif %}"
INFO [                    main] chat template | tid="39812" timestamp=1761570935 chat_example="<|im_start|>system\nYou are a helpful assistant<|im_end|>\n<|im_start|>user\nHello<|im_end|>\n<|im_start|>assistant\nHi there<|im_end|>\n<|im_start|>user\nHow are you?<|im_end|>\n<|im_start|>assistant\n" built_in=true
INFO [                    main] HTTP server listening | tid="39812" timestamp=1761570935 hostname="127.0.0.1" port="15922" n_threads_http="31"
INFO [            update_slots] all slots are idle | tid="39812" timestamp=1761570935
```

But when I use `-dev CUDA0` instead, it won't load:

```ps1
PowerShell 7.5.4
PS C:\Users\SU> C:\apps\ik_llama.cpp\build\bin\Release\llama-server --port 15922 `
>>       --model "E:\AI\LLM\gguf\unsloth\Qwen3-235B-A22B-Instruct-2507-GGUF\Qwen3-235B-A22B-Instruct-2507-UD-Q4_K_XL-00001-of-00003.gguf" `
>>       -rtr `
>>       --ctx-size 16384 `
>>       -ctk q8_0 -ctv q8_0 `
>>       -amb 512 `
>>       -dev CUDA0 `
>>       --n-gpu-layers 99 `
>>       -mg 0 `
>>       -ot "blk\.(0?[0-9]|1[0-6])\.ffn_.*_exps.=CUDA0" `
>>       -ot ".ffn_.*_exps.=CPU" `
>>       --parallel 1 `
>>       -tb 30 -t 15 `
>>       --jinja --reasoning-budget 0 --no-mmap
ggml_cuda_init: GGML_CUDA_FORCE_MMQ:    no
ggml_cuda_init: GGML_CUDA_FORCE_CUBLAS: no
ggml_cuda_init: found 2 CUDA devices:
  Device 0: NVIDIA GeForce RTX 5090, compute capability 12.0, VMM: yes
  Device 1: NVIDIA GeForce RTX 4070 Ti SUPER, compute capability 8.9, VMM: yes
INFO [                    main] build info | tid="53412" timestamp=1761570698 build=3933 commit="7a921655"
INFO [                    main] system info | tid="53412" timestamp=1761570698 n_threads=15 n_threads_batch=30 total_threads=32 system_info="AVX = 1 | AVX_VNNI = 0 | AVX2 = 1 | AVX512 = 1 | AVX512_VBMI = 0 | AVX512_VNNI = 0 | AVX512_BF16 = 0 | FMA = 1 | NEON = 0 | SVE = 0 | ARM_FMA = 0 | F16C = 1 | FP16_VA = 0 | WASM_SIMD = 0 | BLAS = 1 | SSE3 = 1 | SSSE3 = 1 | VSX = 0 | MATMUL_INT8 = 0 | LLAMAFILE = 1 | "
CUDA0: using device CUDA0 - 30841 MiB free
llama_model_loader: additional 2 GGUFs metadata loaded.
llama_model_loader: loaded meta data with 48 key-value pairs and 1131 tensors from E:\AI\LLM\gguf\unsloth\Qwen3-235B-A22B-Instruct-2507-GGUF\Qwen3-235B-A22B-Instruct-2507-UD-Q4_K_XL-00001-of-00003.gguf (version GGUF V3 (latest))
llama_model_loader: Dumping metadata keys/values. Note: KV overrides do not apply in this output.
llama_model_loader: - kv   0:                       general.architecture str              = qwen3moe
llama_model_loader: - kv   1:                               general.type str              = model
llama_model_loader: - kv   2:                               general.name str              = Qwen3-235B-A22B-Instruct-2507
llama_model_loader: - kv   3:                            general.version str              = 2507
llama_model_loader: - kv   4:                           general.finetune str              = Instruct
llama_model_loader: - kv   5:                           general.basename str              = Qwen3-235B-A22B-Instruct-2507
llama_model_loader: - kv   6:                       general.quantized_by str              = Unsloth
llama_model_loader: - kv   7:                         general.size_label str              = 235B-A22B
llama_model_loader: - kv   8:                            general.license str              = apache-2.0
llama_model_loader: - kv   9:                       general.license.link str              = https://huggingface.co/Qwen/Qwen3-235...
llama_model_loader: - kv  10:                           general.repo_url str              = https://huggingface.co/unsloth
llama_model_loader: - kv  11:                   general.base_model.count u32              = 1
llama_model_loader: - kv  12:                  general.base_model.0.name str              = Qwen3 235B A22B Instruct 2507
llama_model_loader: - kv  13:               general.base_model.0.version str              = 2507
llama_model_loader: - kv  14:          general.base_model.0.organization str              = Qwen
llama_model_loader: - kv  15:              general.base_model.0.repo_url str              = https://huggingface.co/Qwen/Qwen3-235...
llama_model_loader: - kv  16:                               general.tags arr[str,2]       = ["unsloth", "text-generation"]
llama_model_loader: - kv  17:                       qwen3moe.block_count u32              = 94
llama_model_loader: - kv  18:                    qwen3moe.context_length u32              = 262144
llama_model_loader: - kv  19:                  qwen3moe.embedding_length u32              = 4096
llama_model_loader: - kv  20:               qwen3moe.feed_forward_length u32              = 12288
llama_model_loader: - kv  21:              qwen3moe.attention.head_count u32              = 64
llama_model_loader: - kv  22:           qwen3moe.attention.head_count_kv u32              = 4
llama_model_loader: - kv  23:                    qwen3moe.rope.freq_base f32              = 5000000.000000
llama_model_loader: - kv  24:  qwen3moe.attention.layer_norm_rms_epsilon f32              = 0.000001
llama_model_loader: - kv  25:                 qwen3moe.expert_used_count u32              = 8
llama_model_loader: - kv  26:              qwen3moe.attention.key_length u32              = 128
llama_model_loader: - kv  27:            qwen3moe.attention.value_length u32              = 128
llama_model_loader: - kv  28:                      qwen3moe.expert_count u32              = 128
llama_model_loader: - kv  29:        qwen3moe.expert_feed_forward_length u32              = 1536
llama_model_loader: - kv  30:                       tokenizer.ggml.model str              = gpt2
llama_model_loader: - kv  31:                         tokenizer.ggml.pre str              = qwen2
llama_model_loader: - kv  32:                      tokenizer.ggml.tokens arr[str,151936]  = ["!", "\"", "#", "$", "%", "&", "'", ...
llama_model_loader: - kv  33:                  tokenizer.ggml.token_type arr[i32,151936]  = [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, ...
llama_model_loader: - kv  34:                      tokenizer.ggml.merges arr[str,151387]  = ["─а ─а", "─а─а ─а─а", "i n", "─а t",...
llama_model_loader: - kv  35:                tokenizer.ggml.eos_token_id u32              = 151645
llama_model_loader: - kv  36:            tokenizer.ggml.padding_token_id u32              = 151654
llama_model_loader: - kv  37:               tokenizer.ggml.add_bos_token bool             = false
llama_model_loader: - kv  38:                    tokenizer.chat_template str              = {%- if tools %}\n    {{- '<|im_start|>...
llama_model_loader: - kv  39:               general.quantization_version u32              = 2
llama_model_loader: - kv  40:                          general.file_type u32              = 15
llama_model_loader: - kv  41:                      quantize.imatrix.file str              = Qwen3-235B-A22B-Instruct-2507-GGUF/im...
llama_model_loader: - kv  42:                   quantize.imatrix.dataset str              = unsloth_calibration_Qwen3-235B-A22B-I...
llama_model_loader: - kv  43:             quantize.imatrix.entries_count u32              = 745
llama_model_loader: - kv  44:              quantize.imatrix.chunks_count u32              = 693
llama_model_loader: - kv  45:                                   split.no u16              = 0
llama_model_loader: - kv  46:                        split.tensors.count i32              = 1131
llama_model_loader: - kv  47:                                split.count u16              = 3
llama_model_loader: - type  f32:  471 tensors
llama_model_loader: - type q4_K:  609 tensors
llama_model_loader: - type q5_K:   40 tensors
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
llm_load_print_meta: model ftype      = Q4_K - Medium
llm_load_print_meta: model params     = 235.094 B
llm_load_print_meta: model size       = 125.098 GiB (4.571 BPW)
llm_load_print_meta: repeating layers = 124.297 GiB (4.566 BPW, 233.849 B parameters)
llm_load_print_meta: general.name     = Qwen3-235B-A22B-Instruct-2507
llm_load_print_meta: n_ff_exp         = 1536
print_info: vocab type       = BPE
print_info: n_vocab          = 151936
print_info: n_merges         = 151387
print_info: BOS token        = 11 ','
print_info: EOS token        = 151645 '<|im_end|>'
print_info: EOT token        = 151645 '<|im_end|>'
print_info: PAD token        = 151654 '<|vision_pad|>'
print_info: LF token         = 198 '─К'
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
Tensor blk.0.ffn_gate_exps.weight buffer type overriden to CUDA0
Tensor blk.0.ffn_down_exps.weight buffer type overriden to CUDA0
Tensor blk.0.ffn_up_exps.weight buffer type overriden to CUDA0
Tensor blk.1.ffn_gate_exps.weight buffer type overriden to CUDA0
Tensor blk.1.ffn_down_exps.weight buffer type overriden to CUDA0
Tensor blk.1.ffn_up_exps.weight buffer type overriden to CUDA0
Tensor blk.2.ffn_gate_exps.weight buffer type overriden to CUDA0
Tensor blk.2.ffn_down_exps.weight buffer type overriden to CUDA0
Tensor blk.2.ffn_up_exps.weight buffer type overriden to CUDA0
Tensor blk.3.ffn_gate_exps.weight buffer type overriden to CUDA0
Tensor blk.3.ffn_down_exps.weight buffer type overriden to CUDA0
Tensor blk.3.ffn_up_exps.weight buffer type overriden to CUDA0
Tensor blk.4.ffn_gate_exps.weight buffer type overriden to CUDA0
Tensor blk.4.ffn_down_exps.weight buffer type overriden to CUDA0
Tensor blk.4.ffn_up_exps.weight buffer type overriden to CUDA0
Tensor blk.5.ffn_gate_exps.weight buffer type overriden to CUDA0
Tensor blk.5.ffn_down_exps.weight buffer type overriden to CUDA0
Tensor blk.5.ffn_up_exps.weight buffer type overriden to CUDA0
Tensor blk.6.ffn_gate_exps.weight buffer type overriden to CUDA0
Tensor blk.6.ffn_down_exps.weight buffer type overriden to CUDA0
Tensor blk.6.ffn_up_exps.weight buffer type overriden to CUDA0
Tensor blk.7.ffn_gate_exps.weight buffer type overriden to CUDA0
Tensor blk.7.ffn_down_exps.weight buffer type overriden to CUDA0
Tensor blk.7.ffn_up_exps.weight buffer type overriden to CUDA0
Tensor blk.8.ffn_gate_exps.weight buffer type overriden to CUDA0
Tensor blk.8.ffn_down_exps.weight buffer type overriden to CUDA0
Tensor blk.8.ffn_up_exps.weight buffer type overriden to CUDA0
Tensor blk.9.ffn_gate_exps.weight buffer type overriden to CUDA0
Tensor blk.9.ffn_down_exps.weight buffer type overriden to CUDA0
Tensor blk.9.ffn_up_exps.weight buffer type overriden to CUDA0
Tensor blk.10.ffn_gate_exps.weight buffer type overriden to CUDA0
Tensor blk.10.ffn_down_exps.weight buffer type overriden to CUDA0
Tensor blk.10.ffn_up_exps.weight buffer type overriden to CUDA0
Tensor blk.11.ffn_gate_exps.weight buffer type overriden to CUDA0
Tensor blk.11.ffn_down_exps.weight buffer type overriden to CUDA0
Tensor blk.11.ffn_up_exps.weight buffer type overriden to CUDA0
Tensor blk.12.ffn_gate_exps.weight buffer type overriden to CUDA0
Tensor blk.12.ffn_down_exps.weight buffer type overriden to CUDA0
Tensor blk.12.ffn_up_exps.weight buffer type overriden to CUDA0
Tensor blk.13.ffn_gate_exps.weight buffer type overriden to CUDA0
Tensor blk.13.ffn_down_exps.weight buffer type overriden to CUDA0
Tensor blk.13.ffn_up_exps.weight buffer type overriden to CUDA0
Tensor blk.14.ffn_gate_exps.weight buffer type overriden to CUDA0
Tensor blk.14.ffn_down_exps.weight buffer type overriden to CUDA0
Tensor blk.14.ffn_up_exps.weight buffer type overriden to CUDA0
Tensor blk.15.ffn_gate_exps.weight buffer type overriden to CUDA0
Tensor blk.15.ffn_down_exps.weight buffer type overriden to CUDA0
Tensor blk.15.ffn_up_exps.weight buffer type overriden to CUDA0
Tensor blk.16.ffn_gate_exps.weight buffer type overriden to CUDA0
Tensor blk.16.ffn_down_exps.weight buffer type overriden to CUDA0
Tensor blk.16.ffn_up_exps.weight buffer type overriden to CUDA0
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
llm_load_tensors:  CUDA_Host buffer size =   333.84 MiB
llm_load_tensors:        CPU buffer size = 100752.00 MiB
llm_load_tensors:      CUDA0 buffer size = 27014.90 MiB
....................................................................................................
============ Repacked 231 tensors
llama_new_context_with_model: n_ctx         = 16384
llama_new_context_with_model: n_batch       = 2048
llama_new_context_with_model: n_ubatch      = 512
llama_new_context_with_model: flash_attn    = 1
llama_new_context_with_model: mla_attn      = 0
llama_new_context_with_model: attn_max_b    = 512
llama_new_context_with_model: fused_moe     = 1
llama_new_context_with_model: grouped er    = 0
llama_new_context_with_model: fused_up_gate = 1
llama_new_context_with_model: fused_mmad    = 1
llama_new_context_with_model: ser           = -1, 0
llama_new_context_with_model: freq_base     = 5000000.0
llama_new_context_with_model: freq_scale    = 1
llama_kv_cache_init:      CUDA0 KV buffer size =  1598.05 MiB
llama_new_context_with_model: KV self size  = 1598.00 MiB, K (q8_0):  799.00 MiB, V (q8_0):  799.00 MiB
llama_new_context_with_model:  CUDA_Host  output buffer size =     1.16 MiB
llama_new_context_with_model: pipeline parallelism enabled (n_copies=4)
ggml_backend_cuda_buffer_type_alloc_buffer: allocating 20120.78 MiB on device 0: cudaMalloc failed: out of memory
ggml_gallocr_reserve_n: failed to allocate CUDA0 buffer of size 21098168320
llama_new_context_with_model: failed to allocate compute buffers
llama_init_from_gpt_params: error: failed to create context with model 'E:\AI\LLM\gguf\unsloth\Qwen3-235B-A22B-Instruct-2507-GGUF\Qwen3-235B-A22B-Instruct-2507-UD-Q4_K_XL-00001-of-00003.gguf'
 ERR [              load_model] unable to load model | tid="53412" timestamp=1761570806 model="E:\\AI\\LLM\\gguf\\unsloth\\Qwen3-235B-A22B-Instruct-2507-GGUF\\Qwen3-235B-A22B-Instruct-2507-UD-Q4_K_XL-00001-of-00003.gguf"
PS C:\Users\SU>
```

As you can see, it tries to allocate some extra 20 GB which it does not require when using `CUDA_VISIBLE_DEVICES=0`.

This also happens when I try to use my 2 GPUs for a single model with ik_llama.cpp, it needs much more VRAM for some reason so I can fit in way less layers in VRAM, so I only use the RTX 5090 with ik_llama.cpp.

Thus, I can't use it with a draft model either, it gives me the same error:

```ps1
C:\apps\ik_llama.cpp\build\bin\Release\llama-server --port 15922 `
      --model "E:\AI\LLM\gguf\unsloth\Qwen3-235B-A22B-Instruct-2507-GGUF\Qwen3-235B-A22B-Instruct-2507-UD-Q4_K_XL-00001-of-00003.gguf" `
      -rtr `
      --ctx-size 16384 `
      -ctk q8_0 -ctv q8_0 `
      -amb 512 `
      -dev CUDA0 `
      --n-gpu-layers 99 `
      -mg 0 `
      -ot "blk\.(0?[0-9]|1[0-6])\.ffn_.*_exps.=CUDA0" `
      -ot ".ffn_.*_exps.=CPU" `
      --parallel 1 `
      -tb 30 -t 15 `
      --jinja --reasoning-budget 0 --no-mmap `
      --model-draft "E:\AI\LLM\gguf\unsloth\Qwen3-4B-Instruct-2507-GGUF\Qwen3-4B-Instruct-2507-UD-Q6_K_XL.gguf" `
      -devd CUDA1 `
      -ngld 999 `
      --ctx-size-draft 16384 `
      -ctkd q8_0 -ctvd q8_0
```

---

```ps1
PS C:\Users\SU> C:\apps\ik_llama.cpp\build\bin\Release\llama-server.exe --version
version: 3933 (7a921655)
built with MSVC 19.41.34123.0 for x64
```

---

👤 **firecoperana** commented on **2025-10-27** at **14:01:15**

I see `llama_new_context_with_model: pipeline parallelism enabled (n_copies=4)` in your logs. Did you compile with GGML_SCHED_MAX_COPIES=1? Add DGGML_SCHED_MAX_COPIES=1 in cmake option and see if it resolves the abnormal vram usage.

---

👤 **ikawrakow** commented on **2025-10-27** at **14:05:42**

> I see llama_new_context_with_model: pipeline parallelism enabled (n_copies=4) in your logs. Did you compile with GGML_SCHED_MAX_COPIES=1?

I thought I had fixed this unfortunate thing. But every now and then we see someone running with `GGML_SCHED_MAX_COPIES=4`, so clearly I have missed something in my fix.

---

👤 **firecoperana** commented on **2025-10-27** at **14:42:29**

I don't recall seeing this issue in mainline and their default is also 4, so they probably have some fix for it.

---

👤 **MRGRD56** commented on **2025-10-27** at **14:47:41**

Actually, to build this new version, I just ran:

```ps1
cmake --build ./build --config Release -j 32
```

Now I ran this:

```ps1
cmake --build ./build --target clean
cmake -B ./build -DGGML_CUDA=ON -DGGML_BLAS=OFF -DGGML_SCHED_MAX_COPIES=1
cmake --build ./build --config Release -j 32
```

And, yeah, now it works, it uses normal amount of VRAM. And this PR works too, so far!

Thank you!

---

👤 **ikawrakow** approved this pull request ✅ on **2025-10-27** at **16:13:20**

---

👤 **firecoperana** commented on **2025-10-29** at **00:09:18**

@ikawrakow 
The VRAM issue seems be to related to 
```// enabling pipeline parallelism in the scheduler increases memory usage, so it is only done when necessary
bool pipeline_parallel =
    llama_get_device_count(*model) > 1 &&
    model->n_gpu_layers > (int)model->hparams.n_layer &&
    model->split_mode == LLAMA_SPLIT_MODE_LAYER &&
    params.offload_kqv;
```
mainline is 
```bool pipeline_parallel =
    model.n_devices() > 1 &&
    model.params.n_gpu_layers > (int) model.hparams.n_layer &&
    model.params.split_mode == LLAMA_SPLIT_MODE_LAYER &&
    cparams.offload_kqv &&
    !model.has_tensor_overrides();    
```

---

👤 **ikawrakow** commented on **2025-10-29** at **06:29:43**

Oh, they simply turn off pipeline parallel when there are tensor overrides? Do you want to submit a PR?

---

👤 **firecoperana** commented on **2025-10-29** at **15:24:05**

Sure