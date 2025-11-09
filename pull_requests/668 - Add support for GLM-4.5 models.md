## 🔀 [Pull Request #668](https://github.com/ikawrakow/ik_llama.cpp/pull/668) - Add support for GLM-4.5 models

| **Author** | `Thireus` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `glm-4.5-clean` |
| **Target Branch** | `main` |
| **Created** | 2025-08-01 |
| **Updated** | 2025-08-13 |
| **Merged** | 2025-08-07 |

---

## 📄 Description

- [x] I have read the [contributing guidelines](https://github.com/ggerganov/llama.cpp/blob/master/CONTRIBUTING.md)
- Self-reported review complexity:
  - [ ] Low
  - [x] Medium
  - [ ] High

Port from llama.cpp - See https://github.com/ggml-org/llama.cpp/pull/14939
Ported from https://github.com/sammcj/llama.cpp/tree/dcbbd2cb057a6c6e907e0195395a74201ef19e1b
Old conversation - https://github.com/ikawrakow/ik_llama.cpp/pull/662

Still some cleanup work to do, such as the GGML_ASSERT which needs to be restored. Contributors are welcomed.

Want to test it out?
```
# Clone patched llama.cpp for glm-4.5
git clone https://github.com/ikawrakow/ik_llama.cpp
cd ik_llama.cpp
git remote add Thireus https://github.com/Thireus/ik_llama.cpp.git
git fetch Thireus
git checkout glm-4.5-clean
git pull
git rev-parse --short HEAD
```

Instructions to convert HF to GGUF:
<details>

→ Alternatively you can download [GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT](https://huggingface.co/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT) - must be used with `ulimit -n 9999`


```
# Install dependencies
apt-get install python3-dev python3-pip python3-venv python3-wheel python3-setuptools git cmake pipx ccache
apt-get install --no-install-recommends zlib1g-dev libxml2-dev libssl-dev libgmp-dev libmpfr-dev

# Install uv via pipx
pipx install uv

# Prepare env
mkdir -p ~/AI/hf-to-bf16
cd ~/AI/hf-to-bf16
uv venv ./venv --python 3.12 --python-preference=only-managed

# Activate env
source venv/bin/activate

# Clone patched ik_llama.cpp for glm-4.5, if not already available
git clone https://github.com/ikawrakow/ik_llama.cpp
cd ik_llama.cpp
git remote add Thireus https://github.com/Thireus/ik_llama.cpp.git
git fetch Thireus
git checkout glm-4.5-clean
git pull
git rev-parse --short HEAD

# Install ik_llama.cpp dependencies
cd ~/AI/hf-to-bf16/ik_llama.cpp
uv pip install -r requirements/requirements-convert_hf_to_gguf.txt --prerelease=allow --index-strategy unsafe-best-match
# Build ik_llama.cpp (optional)
cmake -B build -DGGML_AVX=ON -DGGML_AVX2=ON -DLLAMA_CURL=OFF
cmake --build build --config Release -j16
cd ..

# Build triton-cpu
cd ~/AI/hf-to-bf16/
git clone https://github.com/triton-lang/triton-cpu --recursive
cd triton-cpu
uv pip install ninja cmake wheel setuptools pybind11

# Apply this patch - https://github.com/ikawrakow/ik_llama.cpp/issues/383#issuecomment-2865306085
nano -w CMakeLists.txt
---
#  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Werror -Wno-covered-switch-default -fvisibility=hidden")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-covered-switch-default -fvisibility=hidden")
---
nano -w third_party/cpu/CMakeLists.txt
---
#find_package(dnnl CONFIG)
#if (dnnl_FOUND)
#... comment all the lines up until the endif
#endif()
---

# Install dependencies
uv pip install -r python/requirements.txt

# Compile
MAX_JOBS=16 uv pip install -e python --no-build-isolation

# Be patient, "Preparing Packages" downloads a lot of stuff before build begins...

# Convert HF to BF16 for GLM-4.5
# It is assumed that GLM-4.5 was downloaded from https://huggingface.co/zai-org/GLM-4.5 in ~/AI/huggingface/GLM-4.5
cd ~/AI/hf-to-bf16/ik_llama.cpp
mkdir -p ~/AI/GLM-4.5/GLM-4_5-BF16
python convert_hf_to_gguf.py \
     --outtype bf16 \
     --outfile ~/AI/GLM-4.5/GLM-4_5-BF16/GLM-4_5-BF16\
     --no-tensor-first-split \
     ~/AI/huggingface/GLM-4.5
```

</details>

Basic instructions to quantize from BF16 to q4_K quant:
<details>

→ Alternatively you can download [GLM-4.5-THIREUS-Q4_K-SPECIAL_SPLIT](https://huggingface.co/Thireus/GLM-4.5-THIREUS-Q4_K-SPECIAL_SPLIT) - must be used with `ulimit -n 9999` and ik_llama built with `-DGGML_MAX_CONTEXTS=2048`
**IMPORTANT: This is not how these SPECIAL_SPLIT quants are supposed to be used, as they must be used in a recipe (read more at https://gguf.thireus.com), so expect quite bad perplexity. However still usable this way for testing purpose.**


```
# Example with q4_K
mkdir ~/AI/GLM-4.5/GLM-4_5-Q4_K
llama-quantize ~/AI/GLM-4.5/GLM-4_5-BF16/GLM-4_5-BF16-00001-of-01762.gguf ~/AI/GLM-4.5/GLM-4_5-Q4_K/GLM-4_5-Q4_K.gguf q4_K 8
```

</details>

Basic instructions to lauch llama-server:
<details>

```
llama-server -m ~/AI/GLM-4.5/GLM-4_5-Q4_K/GLM-4_5-Q4_K.gguf -fa \
  -amb 1024 \
  -fmoe \
  -ctk f16 \
  -c 4096 \
  -ngl 99 \
  -ot "blk\.([0-9]|1[0-5])\.ffn_.*=CUDA0" \
  -ot exps=CPU \
  -b 2048 -ub 1024 \
  --warmup-batch \
  --no-mmap \
  --threads 8 \
  --main-gpu 0
```

</details>

Quant mix recipe for the adventurous:
<details>

- 3.9070bpw (PPL: 4.6903 +/- 0.01835) for GLM-4.5-Air: https://github.com/Thireus/GGUF-Tool-Suite/blob/main/recipe_examples/GLM-4.5-Air.ROOT-3.9070bpw-4.6903ppl.50GB-GGUF_5GB-GPU_45GB-CPU.a02563d_270bea7.recipe

</details>

Windows builds available at: https://github.com/Thireus/ik_llama.cpp/releases

---

## 💬 Conversation

👤 **ubergarm** started a conversation on `src/llama.cpp` on **2025-08-01** at **20:15:40**

Did you test perplexity without this to see if it runs clean? Usually only add a model in here if perplexity is throwing nans without it.

> 👤 **Thireus** replied on **2025-08-01** at **20:35:56**
> 
> At that time the model was still not working (or I had not tried llama-server); so I was trying my best to find any possible change I could make to make the model work, without putting much thoughts into it. I had noticed you made those changes in one of your old PR, so I followed your steps. It is entirely possible that this change may not be required after all.

---

👤 **ubergarm** started a conversation on `src/llama.cpp` on **2025-08-01** at **20:21:59**

This if is slightly different than mainline: https://github.com/ggml-org/llama.cpp/pull/14939/files#diff-36e262e316ec1404e29880eb8b8ce4660ac584f0d0434710efc48a66497bdb59R13557

On mainline it is:
> if (il == n_layer - 1 && inp_out_ids) {

I haven't followed the mainline PR closely enough to see if this changed recently nor looked in depth yet at what inp_out_ids even is supposed to be over there.

> 👤 **Thireus** replied on **2025-08-01** at **20:34:14**
> 
> Yes, I've noticed this diff, it was originally there: https://github.com/ggml-org/llama.cpp/pull/14939/files/fae4df8ee05c57bf2e2d5e3f0b094010f34dc86f#diff-36e262e316ec1404e29880eb8b8ce4660ac584f0d0434710efc48a66497bdb59R13554

---

👤 **ubergarm** reviewed this pull request 💬 on **2025-08-01** at **20:23:30**

Just two quick observations and trying to better understand how you ported the mainline PR as it is a bit different (did you make some changes, or was this port from a couple days ago etc).

Anyway, thanks for opening a clean PR to make it easier to look at the differences!

---

👤 **Thireus** commented on **2025-08-01** at **20:29:42**

> Just two quick observations and trying to better understand how you ported the mainline PR as it is a bit different (did you make some changes, or was this port from a couple days ago etc).
> 
> Anyway, thanks for opening a clean PR to make it easier to look at the differences!

This is a port from a few days ago indeed - to the best of my knowledge it was https://github.com/ggml-org/llama.cpp/pull/14939/files/fae4df8ee05c57bf2e2d5e3f0b094010f34dc86f. With slight variations as I was trying to best match existing implementations such as DeepSeek to make the port compatible with ik_llama.

---

👤 **Thireus** commented on **2025-08-01** at **20:43:04**

@ubergarm - /nothink seems to be working just fine. I'm not witnessing the issues they are reporting on the llama.cpp implementation.

---

👤 **ubergarm** commented on **2025-08-01** at **20:43:54**

My other thought about chat template stuff is after running gguf dump i see:
```
$ python gguf-py/scripts/gguf_dump.py /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/GLM-4.5-Thireus-Q8_0.gguf
...
     44: STRING     |        1 | tokenizer.chat_template = '[gMASK]<sop>\n{%- if tools -%}\n<|system|>\n# Tools\n\nYou may ca'
```

This looks right with what the official chat template is here: https://huggingface.co/zai-org/GLM-4.5?chat_template=default

When starting the model with llama-server it says:
```
INFO [                    main] chat template | tid="123750572615872" timestamp=1754072872 chat_example="<|system|>\nYou are a helpful assistant\n<|user|>\nHello\n<|assistant|>\nHi there\n<|user|>\nHow are you?\n<|assistant|>\n" built_in=true
```

So for `/chat/completions/` API endpoint it seems like it will be properly applying the basic template, however I haven't run their jinja through the decoder script (i gotta find my notes hah) and made sure it matches up closely enough. But in limited testing llama-server seems to be working okay.

I'll upload your imatrix soon after it finishes to here: https://huggingface.co/ubergarm/GLM-4.5-GGUF 

Once again though, I'm hesitant to release quants just yet until things are more tested, but excited to see this coming along! Thanks for your efforts!

---

👤 **ubergarm** commented on **2025-08-01** at **20:46:30**

I'll already have GLM-4.5-Air bf16 safetensors downloaded, so will try this PR going through the whole process and see how it comes out for testing.

---

👤 **Thireus** commented on **2025-08-01** at **20:47:09**

> I'll already have GLM-4.5-Air bf16 safetensors downloaded, so will try this PR going through the whole process and see how it comes out for testing.

I was about to that that this is only tested on GLM-4.5, not the others. Good thinking!

---

👤 **ddh0** commented on **2025-08-01** at **20:52:31**

Also cc @AesSedai

---

👤 **ubergarm** commented on **2025-08-01** at **21:07:25**

@Thireus 

Your imatrix is uploaded here: https://huggingface.co/ubergarm/GLM-4.5-GGUF/blob/main/imatrix-GLM-4.5-BF16.dat

Here are the logs of the run including `--layer-similarity` if you are interested:

<details>

<summary>👈 imatrix logs and command</summary>

```bash
#!/usr/bin/env bash

ulimit -n 9999

# echo 0 | sudo tee /proc/sys/kernel/numa_balancing
# sudo sync; echo 3 | sudo tee /proc/sys/vm/drop_caches

model=/mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/GLM-4.5-THIREUS-BF16-SPECIAL_TENSOR-00001-of-01762.gguf

numactl -N 1 -m 1 \
./build/bin/llama-imatrix \
    -m "$model" \
    -fa \
    -f ubergarm-imatrix-calibration-corpus-v02.txt \
    -o /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat \
    --verbosity 1 \
    --layer-similarity \
    --seed 1337 \
    --ctx-size 512 \
    -ub 4096 -b 4096 \
    --numa numactl \
    --threads 128 \
    --threads-batch 192 \
    --no-mmap

llama_model_loader: loaded meta data with 44 key-value pairs and 1761 tensors from /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/GLM-4.5-THIREUS-BF16-SPECIAL_TENSOR-00001-of-01762.gguf (version GGUF V3 (latest))
llama_model_loader: Dumping metadata keys/values. Note: KV overrides do not apply in this output.
llama_model_loader: - kv   0:                       general.architecture str              = glm4moe
llama_model_loader: - kv   1:                               general.type str              = model
llama_model_loader: - kv   2:                               general.name str              = GLM 4.5
llama_model_loader: - kv   3:                            general.version str              = 4.5
llama_model_loader: - kv   4:                           general.basename str              = GLM
llama_model_loader: - kv   5:                         general.size_label str              = 160x21B
llama_model_loader: - kv   6:                            general.license str              = mit
llama_model_loader: - kv   7:                               general.tags arr[str,1]       = ["text-generation"]
llama_model_loader: - kv   8:                          general.languages arr[str,2]       = ["en", "zh"]
llama_model_loader: - kv   9:                        glm4moe.block_count u32              = 93
llama_model_loader: - kv  10:                     glm4moe.context_length u32              = 131072
llama_model_loader: - kv  11:                   glm4moe.embedding_length u32              = 5120
llama_model_loader: - kv  12:                glm4moe.feed_forward_length u32              = 12288
llama_model_loader: - kv  13:               glm4moe.attention.head_count u32              = 96
llama_model_loader: - kv  14:            glm4moe.attention.head_count_kv u32              = 8
llama_model_loader: - kv  15:                     glm4moe.rope.freq_base f32              = 1000000.000000
llama_model_loader: - kv  16:   glm4moe.attention.layer_norm_rms_epsilon f32              = 0.000010
llama_model_loader: - kv  17:                  glm4moe.expert_used_count u32              = 8
llama_model_loader: - kv  18:               glm4moe.attention.key_length u32              = 128
llama_model_loader: - kv  19:             glm4moe.attention.value_length u32              = 128
llama_model_loader: - kv  20:                          general.file_type u32              = 32
llama_model_loader: - kv  21:               glm4moe.rope.dimension_count u32              = 64
llama_model_loader: - kv  22:                       glm4moe.expert_count u32              = 160
llama_model_loader: - kv  23:         glm4moe.expert_feed_forward_length u32              = 1536
llama_model_loader: - kv  24:                glm4moe.expert_shared_count u32              = 1
llama_model_loader: - kv  25:          glm4moe.leading_dense_block_count u32              = 3
llama_model_loader: - kv  26:                 glm4moe.expert_gating_func u32              = 2
llama_model_loader: - kv  27:               glm4moe.expert_weights_scale f32              = 2.500000
llama_model_loader: - kv  28:                glm4moe.expert_weights_norm bool             = true
llama_model_loader: - kv  29:               general.quantization_version u32              = 2
llama_model_loader: - kv  30:                       tokenizer.ggml.model str              = gpt2
llama_model_loader: - kv  31:                         tokenizer.ggml.pre str              = glm4
llama_model_loader: - kv  32:                      tokenizer.ggml.tokens arr[str,151552]  = ["!", "\"", "#", "$", "%", "&", "'", ...
llama_model_loader: - kv  33:                  tokenizer.ggml.token_type arr[i32,151552]  = [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, ...
llama_model_loader: - kv  34:                      tokenizer.ggml.merges arr[str,318088]  = ["Ġ Ġ", "Ġ ĠĠĠ", "ĠĠ ĠĠ", "...
llama_model_loader: - kv  35:                tokenizer.ggml.eos_token_id u32              = 151329
llama_model_loader: - kv  36:            tokenizer.ggml.padding_token_id u32              = 151329
llama_model_loader: - kv  37:                tokenizer.ggml.eot_token_id u32              = 151336
llama_model_loader: - kv  38:            tokenizer.ggml.unknown_token_id u32              = 151329
llama_model_loader: - kv  39:                tokenizer.ggml.bos_token_id u32              = 151329
llama_model_loader: - kv  40:                    tokenizer.chat_template str              = [gMASK]<sop>\n{%- if tools -%}\n<|syste...
llama_model_loader: - kv  41:                                   split.no u16              = 0
llama_model_loader: - kv  42:                                split.count u16              = 1762
llama_model_loader: - kv  43:                        split.tensors.count i32              = 1761
llama_model_loader: - type  f32:  838 tensors
llama_model_loader: - type bf16:  923 tensors
llm_load_vocab: special tokens cache size = 36
llm_load_vocab: token to piece cache size = 0.9713 MB
llm_load_print_meta: format           = GGUF V3 (latest)
llm_load_print_meta: arch             = glm4moe
llm_load_print_meta: vocab type       = BPE
llm_load_print_meta: n_vocab          = 151552
llm_load_print_meta: n_merges         = 318088
llm_load_print_meta: vocab_only       = 0
llm_load_print_meta: n_ctx_train      = 131072
llm_load_print_meta: n_embd           = 5120
llm_load_print_meta: n_layer          = 93
llm_load_print_meta: n_head           = 96
llm_load_print_meta: n_head_kv        = 8
llm_load_print_meta: n_rot            = 64
llm_load_print_meta: n_swa            = 0
llm_load_print_meta: n_swa_pattern    = 1
llm_load_print_meta: n_embd_head_k    = 128
llm_load_print_meta: n_embd_head_v    = 128
llm_load_print_meta: n_gqa            = 12
llm_load_print_meta: n_embd_k_gqa     = 1024
llm_load_print_meta: n_embd_v_gqa     = 1024
llm_load_print_meta: f_norm_eps       = 0.0e+00
llm_load_print_meta: f_norm_rms_eps   = 1.0e-05
llm_load_print_meta: f_clamp_kqv      = 0.0e+00
llm_load_print_meta: f_max_alibi_bias = 0.0e+00
llm_load_print_meta: f_logit_scale    = 0.0e+00
llm_load_print_meta: n_ff             = 12288
llm_load_print_meta: n_expert         = 160
llm_load_print_meta: n_expert_used    = 8
llm_load_print_meta: causal attn      = 1
llm_load_print_meta: pooling type     = 0
llm_load_print_meta: rope type        = 2
llm_load_print_meta: rope scaling     = linear
llm_load_print_meta: freq_base_train  = 1000000.0
llm_load_print_meta: freq_scale_train = 1
llm_load_print_meta: n_ctx_orig_yarn  = 131072
llm_load_print_meta: rope_finetuned   = unknown
llm_load_print_meta: ssm_d_conv       = 0
llm_load_print_meta: ssm_d_inner      = 0
llm_load_print_meta: ssm_d_state      = 0
llm_load_print_meta: ssm_dt_rank      = 0
llm_load_print_meta: model type       = 355B.A32B
llm_load_print_meta: model ftype      = BF16
llm_load_print_meta: model params     = 358.338 B
llm_load_print_meta: model size       = 670.586 GiB (16.075 BPW) 
llm_load_print_meta: repeating layers = 667.696 GiB (16.075 BPW, 356.786 B parameters)
llm_load_print_meta: general.name     = GLM 4.5
llm_load_print_meta: BOS token        = 151329 '<|endoftext|>'
llm_load_print_meta: EOS token        = 151329 '<|endoftext|>'
llm_load_print_meta: UNK token        = 151329 '<|endoftext|>'
llm_load_print_meta: PAD token        = 151329 '<|endoftext|>'
llm_load_print_meta: LF token         = 128 'Ä'
llm_load_print_meta: EOT token        = 151336 '<|user|>'
llm_load_print_meta: max token length = 1024
llm_load_tensors: ggml ctx size =    0.72 MiB
llm_load_tensors:        CPU buffer size = 686680.19 MiB
....................................................................................................
llama_new_context_with_model: n_ctx      = 512
llama_new_context_with_model: n_batch    = 512
llama_new_context_with_model: n_ubatch   = 512
llama_new_context_with_model: flash_attn = 1
llama_new_context_with_model: mla_attn   = 0
llama_new_context_with_model: attn_max_b = 0
llama_new_context_with_model: fused_moe  = 0
llama_new_context_with_model: ser        = -1, 0
llama_new_context_with_model: freq_base  = 1000000.0
llama_new_context_with_model: freq_scale = 1
llama_kv_cache_init:        CPU KV buffer size =   186.00 MiB
llama_new_context_with_model: KV self size  =  186.00 MiB, K (f16):   93.00 MiB, V (f16):   93.00 MiB
llama_new_context_with_model:        CPU  output buffer size =     0.58 MiB
llama_new_context_with_model:        CPU compute buffer size =   306.00 MiB
llama_new_context_with_model: graph nodes  = 4779
llama_new_context_with_model: graph splits = 1

system_info: n_threads = 128 (n_threads_batch = 192) / 768 | AVX = 1 | AVX_VNNI = 1 | AVX2 = 1 | AVX512 = 1 | AVX512_VBMI = 1 | AVX512_VNNI = 1 | AVX512_BF16 = 1 | FMA = 1 | NEON = 0 | SVE = 0 | ARM_FMA = 0 | F16C = 1 | FP16_VA = 0 | WASM_SIMD = 0 | BLAS = 0 | SSE3 = 1 | SSSE3 = 1 | VSX = 0 | MATMUL_INT8 = 0 | LLAMAFILE = 1 | 
compute_imatrix: tokenizing the input ..
compute_imatrix: tokenization took 901.12 ms
compute_imatrix: computing over 814 chunks with batch_size 512
compute_imatrix: 10.08 seconds per pass - ETA 2 hours 16.72 minutes
[1]16.8092,[2]6.7403,[3]4.3630,[4]3.1866,[5]2.5865,[6]2.2122,[7]1.9898,[8]1.8430,[9]1.8314,
save_imatrix: entry '             blk.92.ffn_gate_exps.weight' has partial data (98.75%) 2 out of 160 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.51.ffn_down_exps.weight' has partial data (99.38%) 1 out of 160 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.48.ffn_gate_exps.weight' has partial data (99.38%) 1 out of 160 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.51.ffn_up_exps.weight' has partial data (99.38%) 1 out of 160 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.30.ffn_up_exps.weight' has partial data (99.38%) 1 out of 160 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.29.ffn_down_exps.weight' has partial data (99.38%) 1 out of 160 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.29.ffn_gate_exps.weight' has partial data (99.38%) 1 out of 160 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.29.ffn_up_exps.weight' has partial data (99.38%) 1 out of 160 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.48.ffn_down_exps.weight' has partial data (99.38%) 1 out of 160 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.92.ffn_up_exps.weight' has partial data (98.75%) 2 out of 160 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.30.ffn_gate_exps.weight' has partial data (99.38%) 1 out of 160 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.30.ffn_down_exps.weight' has partial data (99.38%) 1 out of 160 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.51.ffn_gate_exps.weight' has partial data (99.38%) 1 out of 160 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.92.ffn_down_exps.weight' has partial data (98.75%) 2 out of 160 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.48.ffn_up_exps.weight' has partial data (99.38%) 1 out of 160 experts are missing data Storing **but be aware**

save_imatrix: stored collected data after 10 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[10]1.7538,[11]1.8628,[12]1.9497,[13]2.0128,[14]2.0758,[15]1.9792,[16]1.9022,[17]1.8481,[18]1.7962,[19]1.7420,
save_imatrix: stored collected data after 20 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[20]1.7069,[21]1.6655,[22]1.6390,[23]1.6079,[24]1.5804,[25]1.5523,[26]1.6324,[27]1.7262,[28]1.8429,[29]1.8173,
save_imatrix: stored collected data after 30 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[30]1.8001,[31]1.8139,[32]1.8058,[33]1.8766,[34]1.8526,[35]1.8476,[36]1.8334,[37]1.8276,[38]1.8529,[39]1.8663,
save_imatrix: stored collected data after 40 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[40]1.8546,[41]1.8837,[42]1.8914,[43]1.9060,[44]1.9156,[45]1.9178,[46]1.9054,[47]1.9153,[48]1.9143,[49]1.9182,
save_imatrix: stored collected data after 50 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[50]1.9079,[51]1.9235,[52]1.9356,[53]1.9249,[54]1.9343,[55]1.9362,[56]1.9417,[57]1.9357,[58]1.9853,[59]2.0340,
save_imatrix: stored collected data after 60 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[60]2.0820,[61]2.0979,[62]2.1400,[63]2.1680,[64]2.1632,[65]2.1640,[66]2.1671,[67]2.1524,[68]2.1657,[69]2.2064,
save_imatrix: stored collected data after 70 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[70]2.2582,[71]2.2854,[72]2.3223,[73]2.3531,[74]2.3716,[75]2.3998,[76]2.4148,[77]2.4411,[78]2.4389,[79]2.4221,
save_imatrix: stored collected data after 80 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[80]2.4199,[81]2.4225,[82]2.4477,[83]2.4895,[84]2.5059,[85]2.5095,[86]2.5137,[87]2.5037,[88]2.5068,[89]2.4945,
save_imatrix: stored collected data after 90 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[90]2.4849,[91]2.4808,[92]2.4657,[93]2.4469,[94]2.4741,[95]2.5218,[96]2.5412,[97]2.5434,[98]2.5515,[99]2.5706,
save_imatrix: stored collected data after 100 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[100]2.5892,[101]2.5974,[102]2.5991,[103]2.6306,[104]2.6540,[105]2.6476,[106]2.6845,[107]2.7273,[108]2.7577,[109]2.7980,
save_imatrix: stored collected data after 110 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[110]2.8276,[111]2.8644,[112]2.8963,[113]2.8919,[114]2.9083,[115]2.9220,[116]2.9303,[117]2.9364,[118]2.9665,[119]3.0045,
save_imatrix: stored collected data after 120 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[120]3.0421,[121]3.0367,[122]3.0118,[123]2.9970,[124]3.0177,[125]3.0060,[126]2.9822,[127]2.9828,[128]2.9806,[129]2.9875,
save_imatrix: stored collected data after 130 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[130]2.9961,[131]3.0125,[132]3.0298,[133]3.0353,[134]3.0729,[135]3.0906,[136]3.0662,[137]3.0415,[138]3.0195,[139]2.9963,
save_imatrix: stored collected data after 140 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[140]3.0054,[141]3.0137,[142]3.0481,[143]3.0755,[144]3.0806,[145]3.0999,[146]3.1236,[147]3.1461,[148]3.1797,[149]3.2059,
save_imatrix: stored collected data after 150 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[150]3.2329,[151]3.2507,[152]3.2723,[153]3.2842,[154]3.2903,[155]3.2860,[156]3.3026,[157]3.3125,[158]3.3244,[159]3.3376,
save_imatrix: stored collected data after 160 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[160]3.3521,[161]3.3566,[162]3.3596,[163]3.3730,[164]3.3788,[165]3.3865,[166]3.3999,[167]3.4016,[168]3.4032,[169]3.4102,
save_imatrix: stored collected data after 170 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[170]3.4193,[171]3.4246,[172]3.4296,[173]3.4358,[174]3.4532,[175]3.4650,[176]3.4702,[177]3.4760,[178]3.4935,[179]3.4819,
save_imatrix: stored collected data after 180 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[180]3.4896,[181]3.5034,[182]3.5243,[183]3.5399,[184]3.5470,[185]3.5495,[186]3.5481,[187]3.5473,[188]3.5489,[189]3.5499,
save_imatrix: stored collected data after 190 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[190]3.5511,[191]3.5486,[192]3.5701,[193]3.5991,[194]3.6236,[195]3.6501,[196]3.6697,[197]3.7037,[198]3.7149,[199]3.7318,
save_imatrix: stored collected data after 200 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[200]3.7226,[201]3.7382,[202]3.7311,[203]3.7091,[204]3.6876,[205]3.7082,[206]3.7229,[207]3.7318,[208]3.7417,[209]3.7609,
save_imatrix: stored collected data after 210 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[210]3.7771,[211]3.7935,[212]3.8121,[213]3.8290,[214]3.8321,[215]3.8110,[216]3.7881,[217]3.7656,[218]3.7432,[219]3.7212,
save_imatrix: stored collected data after 220 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[220]3.7055,[221]3.7036,[222]3.6939,[223]3.6892,[224]3.6760,[225]3.6579,[226]3.6574,[227]3.6620,[228]3.6838,[229]3.7077,
save_imatrix: stored collected data after 230 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[230]3.7183,[231]3.7402,[232]3.7358,[233]3.7603,[234]3.7894,[235]3.8015,[236]3.8160,[237]3.8175,[238]3.8411,[239]3.8694,
save_imatrix: stored collected data after 240 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[240]3.8659,[241]3.8761,[242]3.8903,[243]3.9100,[244]3.9294,[245]3.9438,[246]3.9562,[247]3.9656,[248]3.9554,[249]3.9816,
save_imatrix: stored collected data after 250 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[250]3.9954,[251]4.0137,[252]4.0246,[253]4.0289,[254]4.0353,[255]4.0385,[256]4.0511,[257]4.0559,[258]4.0678,[259]4.0822,
save_imatrix: stored collected data after 260 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[260]4.0923,[261]4.1029,[262]4.1146,[263]4.1297,[264]4.1417,[265]4.1577,[266]4.1437,[267]4.1496,[268]4.1544,[269]4.1681,
save_imatrix: stored collected data after 270 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[270]4.1888,[271]4.2028,[272]4.2240,[273]4.2244,[274]4.2225,[275]4.2310,[276]4.2363,[277]4.2530,[278]4.2672,[279]4.2808,
save_imatrix: stored collected data after 280 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[280]4.2902,[281]4.2913,[282]4.3053,[283]4.3167,[284]4.3194,[285]4.3358,[286]4.3368,[287]4.3410,[288]4.3498,[289]4.3473,
save_imatrix: stored collected data after 290 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[290]4.3583,[291]4.3637,[292]4.3700,[293]4.3864,[294]4.3991,[295]4.4124,[296]4.4292,[297]4.4341,[298]4.4530,[299]4.4657,
save_imatrix: stored collected data after 300 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[300]4.4822,[301]4.4951,[302]4.5095,[303]4.5144,[304]4.5336,[305]4.5411,[306]4.5451,[307]4.5536,[308]4.5710,[309]4.5813,
save_imatrix: stored collected data after 310 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[310]4.5858,[311]4.5939,[312]4.6025,[313]4.6165,[314]4.6244,[315]4.6338,[316]4.6466,[317]4.6587,[318]4.6726,[319]4.6779,
save_imatrix: stored collected data after 320 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[320]4.6821,[321]4.6755,[322]4.6849,[323]4.6688,[324]4.6853,[325]4.6888,[326]4.6671,[327]4.6807,[328]4.6914,[329]4.6975,
save_imatrix: stored collected data after 330 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[330]4.7043,[331]4.7033,[332]4.7074,[333]4.7257,[334]4.7232,[335]4.7341,[336]4.7505,[337]4.7598,[338]4.7639,[339]4.7518,
save_imatrix: stored collected data after 340 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[340]4.7632,[341]4.7790,[342]4.7941,[343]4.8108,[344]4.8320,[345]4.8604,[346]4.8630,[347]4.8644,[348]4.8671,[349]4.8744,
save_imatrix: stored collected data after 350 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[350]4.8872,[351]4.9059,[352]4.9061,[353]4.9034,[354]4.9135,[355]4.9099,[356]4.9097,[357]4.9080,[358]4.9038,[359]4.9090,
save_imatrix: stored collected data after 360 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[360]4.9194,[361]4.9156,[362]4.9129,[363]4.8963,[364]4.8796,[365]4.8635,[366]4.8503,[367]4.8318,[368]4.8157,[369]4.7996,
save_imatrix: stored collected data after 370 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[370]4.7866,[371]4.7714,[372]4.7556,[373]4.7429,[374]4.7300,[375]4.7123,[376]4.6990,[377]4.6849,[378]4.6693,[379]4.6561,
save_imatrix: stored collected data after 380 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[380]4.6537,[381]4.6400,[382]4.6338,[383]4.6367,[384]4.6232,[385]4.6175,[386]4.6067,[387]4.5890,[388]4.5728,[389]4.5652,
save_imatrix: stored collected data after 390 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[390]4.5555,[391]4.5414,[392]4.5239,[393]4.5066,[394]4.5040,[395]4.5026,[396]4.4983,[397]4.4885,[398]4.4904,[399]4.4896,
save_imatrix: stored collected data after 400 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[400]4.4737,[401]4.4599,[402]4.4532,[403]4.4396,[404]4.4289,[405]4.4190,[406]4.4097,[407]4.3939,[408]4.3788,[409]4.3650,
save_imatrix: stored collected data after 410 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[410]4.3531,[411]4.3425,[412]4.3370,[413]4.3287,[414]4.3245,[415]4.3204,[416]4.3186,[417]4.3135,[418]4.3088,[419]4.2949,
save_imatrix: stored collected data after 420 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[420]4.2807,[421]4.2662,[422]4.2536,[423]4.2399,[424]4.2282,[425]4.2152,[426]4.2014,[427]4.1906,[428]4.1766,[429]4.1644,
save_imatrix: stored collected data after 430 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[430]4.1522,[431]4.1426,[432]4.1317,[433]4.1220,[434]4.1199,[435]4.1181,[436]4.1119,[437]4.1020,[438]4.0960,[439]4.0828,
save_imatrix: stored collected data after 440 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[440]4.0702,[441]4.0585,[442]4.0473,[443]4.0362,[444]4.0323,[445]4.0235,[446]4.0202,[447]4.0146,[448]4.0049,[449]4.0016,
save_imatrix: stored collected data after 450 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[450]3.9946,[451]3.9874,[452]3.9767,[453]3.9699,[454]3.9632,[455]3.9550,[456]3.9433,[457]3.9322,[458]3.9206,[459]3.9096,
save_imatrix: stored collected data after 460 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[460]3.8989,[461]3.8899,[462]3.8810,[463]3.8748,[464]3.8676,[465]3.8636,[466]3.8585,[467]3.8536,[468]3.8486,[469]3.8434,
save_imatrix: stored collected data after 470 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[470]3.8384,[471]3.8334,[472]3.8284,[473]3.8242,[474]3.8191,[475]3.8139,[476]3.8096,[477]3.8047,[478]3.7999,[479]3.7961,
save_imatrix: stored collected data after 480 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[480]3.7857,[481]3.7764,[482]3.7724,[483]3.7656,[484]3.7584,[485]3.7486,[486]3.7395,[487]3.7308,[488]3.7220,[489]3.7158,
save_imatrix: stored collected data after 490 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[490]3.7084,[491]3.7021,[492]3.6979,[493]3.6921,[494]3.6859,[495]3.6784,[496]3.6778,[497]3.6749,[498]3.6698,[499]3.6680,
save_imatrix: stored collected data after 500 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[500]3.6647,[501]3.6636,[502]3.6639,[503]3.6667,[504]3.6647,[505]3.6596,[506]3.6523,[507]3.6562,[508]3.6667,[509]3.6749,
save_imatrix: stored collected data after 510 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[510]3.6826,[511]3.6895,[512]3.6970,[513]3.7014,[514]3.7053,[515]3.7070,[516]3.7142,[517]3.7171,[518]3.7236,[519]3.7323,
save_imatrix: stored collected data after 520 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[520]3.7451,[521]3.7605,[522]3.7736,[523]3.7724,[524]3.7795,[525]3.7835,[526]3.7893,[527]3.7909,[528]3.7931,[529]3.8026,
save_imatrix: stored collected data after 530 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[530]3.8079,[531]3.8093,[532]3.8157,[533]3.8216,[534]3.8286,[535]3.8288,[536]3.8280,[537]3.8288,[538]3.8326,[539]3.8370,
save_imatrix: stored collected data after 540 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[540]3.8417,[541]3.8458,[542]3.8479,[543]3.8506,[544]3.8552,[545]3.8600,[546]3.8687,[547]3.8770,[548]3.8832,[549]3.8919,
save_imatrix: stored collected data after 550 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[550]3.8987,[551]3.9068,[552]3.9131,[553]3.9191,[554]3.9255,[555]3.9314,[556]3.9288,[557]3.9261,[558]3.9228,[559]3.9276,
save_imatrix: stored collected data after 560 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[560]3.9331,[561]3.9378,[562]3.9433,[563]3.9446,[564]3.9491,[565]3.9497,[566]3.9542,[567]3.9548,[568]3.9547,[569]3.9539,
save_imatrix: stored collected data after 570 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[570]3.9546,[571]3.9576,[572]3.9542,[573]3.9509,[574]3.9462,[575]3.9421,[576]3.9350,[577]3.9290,[578]3.9228,[579]3.9166,
save_imatrix: stored collected data after 580 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[580]3.9139,[581]3.9150,[582]3.9130,[583]3.9138,[584]3.9120,[585]3.9120,[586]3.9116,[587]3.9092,[588]3.9037,[589]3.9042,
save_imatrix: stored collected data after 590 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[590]3.9009,[591]3.8938,[592]3.8872,[593]3.8797,[594]3.8743,[595]3.8708,[596]3.8695,[597]3.8677,[598]3.8667,[599]3.8645,
save_imatrix: stored collected data after 600 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[600]3.8599,[601]3.8535,[602]3.8530,[603]3.8530,[604]3.8531,[605]3.8489,[606]3.8471,[607]3.8441,[608]3.8471,[609]3.8458,
save_imatrix: stored collected data after 610 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[610]3.8437,[611]3.8437,[612]3.8425,[613]3.8375,[614]3.8309,[615]3.8233,[616]3.8164,[617]3.8088,[618]3.8017,[619]3.7944,
save_imatrix: stored collected data after 620 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[620]3.7875,[621]3.7793,[622]3.7714,[623]3.7645,[624]3.7581,[625]3.7505,[626]3.7442,[627]3.7367,[628]3.7299,[629]3.7234,
save_imatrix: stored collected data after 630 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[630]3.7168,[631]3.7100,[632]3.7058,[633]3.6985,[634]3.6944,[635]3.6929,[636]3.6877,[637]3.6811,[638]3.6755,[639]3.6690,
save_imatrix: stored collected data after 640 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[640]3.6621,[641]3.6558,[642]3.6496,[643]3.6435,[644]3.6372,[645]3.6309,[646]3.6249,[647]3.6194,[648]3.6184,[649]3.6123,
save_imatrix: stored collected data after 650 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[650]3.6055,[651]3.5992,[652]3.5929,[653]3.5864,[654]3.5796,[655]3.5730,[656]3.5668,[657]3.5611,[658]3.5547,[659]3.5570,
save_imatrix: stored collected data after 660 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[660]3.5573,[661]3.5604,[662]3.5587,[663]3.5528,[664]3.5491,[665]3.5438,[666]3.5377,[667]3.5324,[668]3.5275,[669]3.5226,
save_imatrix: stored collected data after 670 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[670]3.5177,[671]3.5125,[672]3.5068,[673]3.5011,[674]3.4970,[675]3.4922,[676]3.4869,[677]3.4818,[678]3.4765,[679]3.4708,
save_imatrix: stored collected data after 680 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[680]3.4667,[681]3.4611,[682]3.4562,[683]3.4514,[684]3.4459,[685]3.4411,[686]3.4390,[687]3.4378,[688]3.4343,[689]3.4294,
save_imatrix: stored collected data after 690 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[690]3.4236,[691]3.4176,[692]3.4125,[693]3.4072,[694]3.4033,[695]3.4014,[696]3.3996,[697]3.3970,[698]3.3955,[699]3.3934,
save_imatrix: stored collected data after 700 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[700]3.3914,[701]3.3899,[702]3.3879,[703]3.3860,[704]3.3840,[705]3.3822,[706]3.3807,[707]3.3782,[708]3.3768,[709]3.3745,
save_imatrix: stored collected data after 710 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[710]3.3728,[711]3.3708,[712]3.3714,[713]3.3711,[714]3.3711,[715]3.3720,[716]3.3730,[717]3.3740,[718]3.3748,[719]3.3762,
save_imatrix: stored collected data after 720 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[720]3.3783,[721]3.3785,[722]3.3791,[723]3.3798,[724]3.3814,[725]3.3823,[726]3.3842,[727]3.3854,[728]3.3873,[729]3.3872,
save_imatrix: stored collected data after 730 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[730]3.3873,[731]3.3885,[732]3.3909,[733]3.3917,[734]3.3921,[735]3.3923,[736]3.3934,[737]3.3951,[738]3.3954,[739]3.3978,
save_imatrix: stored collected data after 740 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[740]3.3994,[741]3.4013,[742]3.4027,[743]3.4031,[744]3.4030,[745]3.4037,[746]3.4056,[747]3.4068,[748]3.4084,[749]3.4093,
save_imatrix: stored collected data after 750 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[750]3.4106,[751]3.4113,[752]3.4133,[753]3.4155,[754]3.4160,[755]3.4166,[756]3.4181,[757]3.4199,[758]3.4207,[759]3.4219,
save_imatrix: stored collected data after 760 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[760]3.4226,[761]3.4234,[762]3.4251,[763]3.4255,[764]3.4273,[765]3.4286,[766]3.4300,[767]3.4306,[768]3.4316,[769]3.4319,
save_imatrix: stored collected data after 770 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[770]3.4328,[771]3.4347,[772]3.4354,[773]3.4356,[774]3.4361,[775]3.4380,[776]3.4390,[777]3.4410,[778]3.4409,[779]3.4423,
save_imatrix: stored collected data after 780 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[780]3.4438,[781]3.4454,[782]3.4471,[783]3.4492,[784]3.4495,[785]3.4502,[786]3.4508,[787]3.4524,[788]3.4528,[789]3.4550,
save_imatrix: stored collected data after 790 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[790]3.4564,[791]3.4576,[792]3.4577,[793]3.4588,[794]3.4610,[795]3.4623,[796]3.4624,[797]3.4635,[798]3.4646,[799]3.4681,
save_imatrix: stored collected data after 800 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[800]3.4686,[801]3.4682,[802]3.4700,[803]3.4718,[804]3.4725,[805]3.4733,[806]3.4738,[807]3.4747,[808]3.4753,[809]3.4761,
save_imatrix: stored collected data after 810 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat
[810]3.4778,[811]3.4796,[812]3.4808,[813]3.4815,[814]3.4820,
save_imatrix: stored collected data after 814 chunks in /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/imatrix-GLM-4.5-BF16.dat

Final estimate: PPL = 3.4820 +/- 0.01710

llama_print_timings:        load time =  252304.21 ms
llama_print_timings:      sample time =       0.00 ms /     1 runs   (    0.00 ms per token,      inf tokens per second)
llama_print_timings: prompt eval time = 7778409.55 ms / 416768 tokens (   18.66 ms per token,    53.58 tokens per second)
llama_print_timings:        eval time =       0.00 ms /     1 runs   (    0.00 ms per token,      inf tokens per second)
llama_print_timings:       total time = 8047122.96 ms / 416769 tokens

======================== sorted layer importances
  0: Layer   0, <cos_sim> = 0.427685
  1: Layer   2, <cos_sim> = 0.754953
  2: Layer   1, <cos_sim> = 0.761973
  3: Layer   3, <cos_sim> = 0.859269
  4: Layer  91, <cos_sim> = 0.884954
  5: Layer   4, <cos_sim> = 0.903777
  6: Layer  32, <cos_sim> = 0.909284
  7: Layer   6, <cos_sim> = 0.912099
  8: Layer  39, <cos_sim> = 0.916231
  9: Layer  37, <cos_sim> = 0.916806
 10: Layer  23, <cos_sim> = 0.917485
 11: Layer  31, <cos_sim> = 0.917567
 12: Layer  41, <cos_sim> = 0.919794
 13: Layer  40, <cos_sim> = 0.921433
 14: Layer  33, <cos_sim> = 0.921658
 15: Layer  29, <cos_sim> = 0.921735
 16: Layer  30, <cos_sim> = 0.921975
 17: Layer  22, <cos_sim> = 0.923691
 18: Layer  14, <cos_sim> = 0.923877
 19: Layer  28, <cos_sim> = 0.923969
 20: Layer  38, <cos_sim> = 0.923987
 21: Layer  24, <cos_sim> = 0.924143
 22: Layer  34, <cos_sim> = 0.925237
 23: Layer  26, <cos_sim> = 0.925394
 24: Layer   7, <cos_sim> = 0.925584
 25: Layer  36, <cos_sim> = 0.927248
 26: Layer  13, <cos_sim> = 0.9273
 27: Layer  25, <cos_sim> = 0.928089
 28: Layer  21, <cos_sim> = 0.92832
 29: Layer  85, <cos_sim> = 0.928453
 30: Layer  84, <cos_sim> = 0.929674
 31: Layer  27, <cos_sim> = 0.930059
 32: Layer  35, <cos_sim> = 0.930223
 33: Layer  10, <cos_sim> = 0.930323
 34: Layer   8, <cos_sim> = 0.931374
 35: Layer   9, <cos_sim> = 0.931623
 36: Layer  11, <cos_sim> = 0.932746
 37: Layer  92, <cos_sim> = 0.93573
 38: Layer  42, <cos_sim> = 0.93659
 39: Layer  12, <cos_sim> = 0.938836
 40: Layer   5, <cos_sim> = 0.941925
 41: Layer  43, <cos_sim> = 0.944567
 42: Layer  86, <cos_sim> = 0.944959
 43: Layer  15, <cos_sim> = 0.947795
 44: Layer  20, <cos_sim> = 0.94999
 45: Layer  18, <cos_sim> = 0.952245
 46: Layer  83, <cos_sim> = 0.953255
 47: Layer  19, <cos_sim> = 0.953365
 48: Layer  44, <cos_sim> = 0.953818
 49: Layer  45, <cos_sim> = 0.953992
 50: Layer  17, <cos_sim> = 0.95693
 51: Layer  16, <cos_sim> = 0.957523
 52: Layer  46, <cos_sim> = 0.958581
 53: Layer  87, <cos_sim> = 0.958638
 54: Layer  80, <cos_sim> = 0.958765
 55: Layer  90, <cos_sim> = 0.959166
 56: Layer  81, <cos_sim> = 0.959806
 57: Layer  82, <cos_sim> = 0.961588
 58: Layer  47, <cos_sim> = 0.961593
 59: Layer  89, <cos_sim> = 0.961599
 60: Layer  88, <cos_sim> = 0.962237
 61: Layer  79, <cos_sim> = 0.9637
 62: Layer  48, <cos_sim> = 0.964249
 63: Layer  50, <cos_sim> = 0.965041
 64: Layer  49, <cos_sim> = 0.965927
 65: Layer  51, <cos_sim> = 0.966417
 66: Layer  54, <cos_sim> = 0.968401
 67: Layer  52, <cos_sim> = 0.968414
 68: Layer  76, <cos_sim> = 0.970006
 69: Layer  53, <cos_sim> = 0.970636
 70: Layer  55, <cos_sim> = 0.971746
 71: Layer  78, <cos_sim> = 0.971977
 72: Layer  75, <cos_sim> = 0.972703
 73: Layer  77, <cos_sim> = 0.976101
 74: Layer  58, <cos_sim> = 0.978261
 75: Layer  56, <cos_sim> = 0.978325
 76: Layer  57, <cos_sim> = 0.979194
 77: Layer  59, <cos_sim> = 0.979934
 78: Layer  73, <cos_sim> = 0.980678
 79: Layer  67, <cos_sim> = 0.981173
 80: Layer  66, <cos_sim> = 0.981491
 81: Layer  72, <cos_sim> = 0.981982
 82: Layer  68, <cos_sim> = 0.982146
 83: Layer  65, <cos_sim> = 0.982195
 84: Layer  61, <cos_sim> = 0.982205
 85: Layer  74, <cos_sim> = 0.982215
 86: Layer  60, <cos_sim> = 0.982433
 87: Layer  71, <cos_sim> = 0.982878
 88: Layer  63, <cos_sim> = 0.983266
 89: Layer  70, <cos_sim> = 0.98376
 90: Layer  64, <cos_sim> = 0.984047
 91: Layer  69, <cos_sim> = 0.984223
 92: Layer  62, <cos_sim> = 0.984883

======================== sorted attention importances
  0: Layer   0, <cos_sim> = 0.344938
  1: Layer   1, <cos_sim> = 0.591517
  2: Layer   2, <cos_sim> = 0.654333
  3: Layer   3, <cos_sim> = 0.823345
  4: Layer   7, <cos_sim> = 0.838734
  5: Layer  13, <cos_sim> = 0.848823
  6: Layer   4, <cos_sim> = 0.853067
  7: Layer   6, <cos_sim> = 0.85565
  8: Layer   9, <cos_sim> = 0.859293
  9: Layer   8, <cos_sim> = 0.865704
 10: Layer  15, <cos_sim> = 0.874006
 11: Layer  12, <cos_sim> = 0.876242
 12: Layer   5, <cos_sim> = 0.879342
 13: Layer  10, <cos_sim> = 0.882239
 14: Layer  11, <cos_sim> = 0.882394
 15: Layer  17, <cos_sim> = 0.892329
 16: Layer  16, <cos_sim> = 0.893135
 17: Layer  21, <cos_sim> = 0.897359
 18: Layer  14, <cos_sim> = 0.900934
 19: Layer  19, <cos_sim> = 0.90725
 20: Layer  20, <cos_sim> = 0.913042
 21: Layer  18, <cos_sim> = 0.915475
 22: Layer  23, <cos_sim> = 0.921054
 23: Layer  22, <cos_sim> = 0.928428
 24: Layer  24, <cos_sim> = 0.932289
 25: Layer  25, <cos_sim> = 0.937948
 26: Layer  32, <cos_sim> = 0.941675
 27: Layer  28, <cos_sim> = 0.942764
 28: Layer  27, <cos_sim> = 0.943415
 29: Layer  26, <cos_sim> = 0.944893
 30: Layer  33, <cos_sim> = 0.94565
 31: Layer  31, <cos_sim> = 0.947089
 32: Layer  30, <cos_sim> = 0.947855
 33: Layer  37, <cos_sim> = 0.948046
 34: Layer  38, <cos_sim> = 0.950758
 35: Layer  39, <cos_sim> = 0.951663
 36: Layer  35, <cos_sim> = 0.952977
 37: Layer  41, <cos_sim> = 0.953607
 38: Layer  29, <cos_sim> = 0.954196
 39: Layer  40, <cos_sim> = 0.954582
 40: Layer  34, <cos_sim> = 0.954729
 41: Layer  42, <cos_sim> = 0.954918
 42: Layer  36, <cos_sim> = 0.959325
 43: Layer  85, <cos_sim> = 0.962455
 44: Layer  43, <cos_sim> = 0.964714
 45: Layer  44, <cos_sim> = 0.965964
 46: Layer  45, <cos_sim> = 0.967585
 47: Layer  46, <cos_sim> = 0.970301
 48: Layer  86, <cos_sim> = 0.970694
 49: Layer  84, <cos_sim> = 0.971816
 50: Layer  51, <cos_sim> = 0.974334
 51: Layer  52, <cos_sim> = 0.974995
 52: Layer  83, <cos_sim> = 0.975456
 53: Layer  92, <cos_sim> = 0.977125
 54: Layer  50, <cos_sim> = 0.978028
 55: Layer  47, <cos_sim> = 0.978144
 56: Layer  81, <cos_sim> = 0.978666
 57: Layer  48, <cos_sim> = 0.978749
 58: Layer  49, <cos_sim> = 0.978877
 59: Layer  82, <cos_sim> = 0.978997
 60: Layer  53, <cos_sim> = 0.979392
 61: Layer  80, <cos_sim> = 0.979488
 62: Layer  91, <cos_sim> = 0.980023
 63: Layer  87, <cos_sim> = 0.981837
 64: Layer  58, <cos_sim> = 0.981972
 65: Layer  54, <cos_sim> = 0.982405
 66: Layer  79, <cos_sim> = 0.98302
 67: Layer  88, <cos_sim> = 0.983417
 68: Layer  78, <cos_sim> = 0.984292
 69: Layer  89, <cos_sim> = 0.986245
 70: Layer  90, <cos_sim> = 0.986536
 71: Layer  61, <cos_sim> = 0.986631
 72: Layer  68, <cos_sim> = 0.986747
 73: Layer  59, <cos_sim> = 0.986949
 74: Layer  71, <cos_sim> = 0.987292
 75: Layer  73, <cos_sim> = 0.987331
 76: Layer  55, <cos_sim> = 0.987493
 77: Layer  72, <cos_sim> = 0.987677
 78: Layer  76, <cos_sim> = 0.98837
 79: Layer  57, <cos_sim> = 0.988642
 80: Layer  56, <cos_sim> = 0.988706
 81: Layer  77, <cos_sim> = 0.988904
 82: Layer  67, <cos_sim> = 0.989027
 83: Layer  65, <cos_sim> = 0.989138
 84: Layer  70, <cos_sim> = 0.989192
 85: Layer  63, <cos_sim> = 0.989244
 86: Layer  74, <cos_sim> = 0.989437
 87: Layer  64, <cos_sim> = 0.989463
 88: Layer  66, <cos_sim> = 0.989895
 89: Layer  60, <cos_sim> = 0.989994
 90: Layer  69, <cos_sim> = 0.991278
 91: Layer  75, <cos_sim> = 0.991565
 92: Layer  62, <cos_sim> = 0.991979

======================== sorted ffn importances
  0: Layer   0, <cos_sim> = 0.624763
  1: Layer   1, <cos_sim> = 0.625883
  2: Layer   2, <cos_sim> = 0.758364
  3: Layer   6, <cos_sim> = 0.823684
  4: Layer   3, <cos_sim> = 0.82519
  5: Layer  14, <cos_sim> = 0.855503
  6: Layer   8, <cos_sim> = 0.857968
  7: Layer  11, <cos_sim> = 0.858688
  8: Layer   4, <cos_sim> = 0.860767
  9: Layer  12, <cos_sim> = 0.861034
 10: Layer   7, <cos_sim> = 0.866609
 11: Layer   5, <cos_sim> = 0.867507
 12: Layer  16, <cos_sim> = 0.880483
 13: Layer   9, <cos_sim> = 0.884367
 14: Layer  15, <cos_sim> = 0.884556
 15: Layer  10, <cos_sim> = 0.888465
 16: Layer  20, <cos_sim> = 0.889985
 17: Layer  91, <cos_sim> = 0.895348
 18: Layer  18, <cos_sim> = 0.897993
 19: Layer  19, <cos_sim> = 0.900205
 20: Layer  13, <cos_sim> = 0.900349
 21: Layer  24, <cos_sim> = 0.916468
 22: Layer  17, <cos_sim> = 0.916497
 23: Layer  22, <cos_sim> = 0.916572
 24: Layer  25, <cos_sim> = 0.919954
 25: Layer  26, <cos_sim> = 0.920522
 26: Layer  23, <cos_sim> = 0.921078
 27: Layer  27, <cos_sim> = 0.925478
 28: Layer  29, <cos_sim> = 0.930093
 29: Layer  21, <cos_sim> = 0.930119
 30: Layer  32, <cos_sim> = 0.933381
 31: Layer  31, <cos_sim> = 0.934207
 32: Layer  28, <cos_sim> = 0.935718
 33: Layer  30, <cos_sim> = 0.936237
 34: Layer  34, <cos_sim> = 0.938
 35: Layer  37, <cos_sim> = 0.940027
 36: Layer  36, <cos_sim> = 0.941134
 37: Layer  33, <cos_sim> = 0.943274
 38: Layer  39, <cos_sim> = 0.943999
 39: Layer  40, <cos_sim> = 0.94547
 40: Layer  35, <cos_sim> = 0.945489
 41: Layer  41, <cos_sim> = 0.94562
 42: Layer  38, <cos_sim> = 0.948466
 43: Layer  43, <cos_sim> = 0.952106
 44: Layer  42, <cos_sim> = 0.953932
 45: Layer  44, <cos_sim> = 0.954758
 46: Layer  45, <cos_sim> = 0.955495
 47: Layer  84, <cos_sim> = 0.957791
 48: Layer  46, <cos_sim> = 0.962356
 49: Layer  47, <cos_sim> = 0.963516
 50: Layer  85, <cos_sim> = 0.964269
 51: Layer  50, <cos_sim> = 0.964335
 52: Layer  90, <cos_sim> = 0.964703
 53: Layer  48, <cos_sim> = 0.96498
 54: Layer  51, <cos_sim> = 0.965182
 55: Layer  49, <cos_sim> = 0.965669
 56: Layer  52, <cos_sim> = 0.968839
 57: Layer  86, <cos_sim> = 0.968873
 58: Layer  89, <cos_sim> = 0.968923
 59: Layer  79, <cos_sim> = 0.971219
 60: Layer  83, <cos_sim> = 0.971772
 61: Layer  80, <cos_sim> = 0.972194
 62: Layer  87, <cos_sim> = 0.972445
 63: Layer  53, <cos_sim> = 0.972528
 64: Layer  81, <cos_sim> = 0.972955
 65: Layer  78, <cos_sim> = 0.973467
 66: Layer  57, <cos_sim> = 0.974059
 67: Layer  77, <cos_sim> = 0.974158
 68: Layer  82, <cos_sim> = 0.974329
 69: Layer  55, <cos_sim> = 0.974836
 70: Layer  88, <cos_sim> = 0.974857
 71: Layer  92, <cos_sim> = 0.975135
 72: Layer  54, <cos_sim> = 0.975202
 73: Layer  75, <cos_sim> = 0.975318
 74: Layer  76, <cos_sim> = 0.975601
 75: Layer  56, <cos_sim> = 0.976377
 76: Layer  58, <cos_sim> = 0.976683
 77: Layer  60, <cos_sim> = 0.977063
 78: Layer  73, <cos_sim> = 0.977821
 79: Layer  67, <cos_sim> = 0.97792
 80: Layer  72, <cos_sim> = 0.978052
 81: Layer  70, <cos_sim> = 0.978202
 82: Layer  59, <cos_sim> = 0.978237
 83: Layer  65, <cos_sim> = 0.978362
 84: Layer  71, <cos_sim> = 0.978628
 85: Layer  69, <cos_sim> = 0.978685
 86: Layer  66, <cos_sim> = 0.978833
 87: Layer  63, <cos_sim> = 0.979037
 88: Layer  64, <cos_sim> = 0.979078
 89: Layer  62, <cos_sim> = 0.979626
 90: Layer  74, <cos_sim> = 0.979673
 91: Layer  68, <cos_sim> = 0.979996
 92: Layer  61, <cos_sim> = 0.980596
```

</details>

---

👤 **Thireus** commented on **2025-08-01** at **21:16:28**

Thank you so much @ubergarm!

---

👤 **InfernalDread** commented on **2025-08-01** at **21:41:58**

I apologize if this is off topic, but is it possible for a GLM-4.5-Air 4 bit GGUF quant to test as well, as it is the much friendlier option for resource limited individuals (such as myself).

Thank you all for your amazing and quick work!

---

👤 **ubergarm** commented on **2025-08-01** at **21:49:50**

> I apologize if this is off topic, but is it possible for a GLM-4.5-Air 4 bit GGUF quant to test as well, as it is the much friendlier option for resource limited individuals (such as myself).

Having an issue converting the original bf16 safetensors using this PR to get my bf16 GGUF.
I changed two lines which gets me further:
```diff
@@ -3951,8 +3951,8 @@ class Dots1Model(Qwen2MoeModel):
             return [(self.map_tensor_name(name), data_torch)]
         return super().modify_tensors(data_torch, name, bid)

-@ModelBase.register("Glm4MoeForCausalLM")
-class Glm4MoeModel(TextModel):
+@Model.register("Glm4MoeForCausalLM")
+class Glm4MoeModel(Model):
     model_arch = gguf.MODEL_ARCH.GLM4_MOE

     def __init__(self, *args, **kwargs):
@@ -3960,7 +3960,7 @@ class Glm4MoeModel(TextModel):
         # GLM4_MOE has num_hidden_layers + 1 actual layers (including NextN layer)
         self.block_count = self.hparams["num_hidden_layers"] + 1
         self.tensor_map = gguf.get_tensor_name_map(self.model_arch, self.block_count)
```

But then it bombs out with this:
```bash
python \
    convert_hf_to_gguf.py \
    --outtype bf16 \
    --split-max-size 50G \
    --outfile /mnt/raid/models/ubergarm/GLM-4.5-Air-GGUF/ \
    /mnt/raid/models/zai-org/GLM-4.5-Air/

INFO:hf-to-gguf:gguf: experts used count = 8
INFO:hf-to-gguf:gguf: file type = 24
INFO:hf-to-gguf:Set model tokenizer
WARNING:hf-to-gguf:

WARNING:hf-to-gguf:**************************************************************************************
WARNING:hf-to-gguf:** WARNING: The BPE pre-tokenizer was not recognized!
WARNING:hf-to-gguf:**          There are 2 possible reasons for this:
WARNING:hf-to-gguf:**          - the model has not been added to convert_hf_to_gguf_update.py yet
WARNING:hf-to-gguf:**          - the pre-tokenization config has changed upstream
WARNING:hf-to-gguf:**          Check your model files and convert_hf_to_gguf_update.py and update them accordingly.
WARNING:hf-to-gguf:** ref:     https://github.com/ggerganov/llama.cpp/pull/6920
WARNING:hf-to-gguf:**
WARNING:hf-to-gguf:** chkhsh:  a1336059768a55c99a734006ffb02203cd450fed003e9a71886c88acf24fdbc2
WARNING:hf-to-gguf:**************************************************************************************
WARNING:hf-to-gguf:

Traceback (most recent call last):
  File "/home/w/projects/ik_llama.cpp/convert_hf_to_gguf.py", line 4578, in <module>
    main()
  File "/home/w/projects/ik_llama.cpp/convert_hf_to_gguf.py", line 4572, in main
    model_instance.write()
  File "/home/w/projects/ik_llama.cpp/convert_hf_to_gguf.py", line 431, in write
    self.prepare_metadata(vocab_only=False)
  File "/home/w/projects/ik_llama.cpp/convert_hf_to_gguf.py", line 417, in prepare_metadata
    self.set_vocab()
  File "/home/w/projects/ik_llama.cpp/convert_hf_to_gguf.py", line 3971, in set_vocab
    tokens, toktypes, tokpre = self.get_vocab_base()
                               ^^^^^^^^^^^^^^^^^^^^^
  File "/home/w/projects/ik_llama.cpp/convert_hf_to_gguf.py", line 512, in get_vocab_base
    tokpre = self.get_vocab_base_pre(tokenizer)
             ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/home/w/projects/ik_llama.cpp/convert_hf_to_gguf.py", line 662, in get_vocab_base_pre
    raise NotImplementedError("BPE pre-tokenizer was not recognized - update get_vocab_base_pre()")
NotImplementedError: BPE pre-tokenizer was not recognized - update get_vocab_base_pre()
```

Probably just a few name changes to go still? My other option is to use mainline lcpp's convert python script then try to pick back up here. Thoughts?

Oh I'll check for a missing commit here: `convert_hf_to_gguf_update.py`

---

👤 **Thireus** commented on **2025-08-01** at **21:57:27**

> Probably just a few name changes to go still? My other option is to use mainline lcpp's convert python script then try to pick back up here. Thoughts?
> 
> Oh I'll check for a missing commit here: `convert_hf_to_gguf_update.py`

I recommend using the llama.cpp script for this. Which is what I did. So not too sure if I had ported all the right changes. There are no changes to the _update.py version and I don't know what this version of the script is used for.

---

👤 **InfernalDread** commented on **2025-08-01** at **21:59:16**

<img width="612" height="497" alt="image" src="https://github.com/user-attachments/assets/1ec97531-d010-4cba-aa78-052de70ff4bc" />

I think it has something to do with the convert_hf_to_gguf_update.py script as well, as I do not see any reference of GLM 4.5 in the original convert_hf_to_gguf.py script

---

👤 **ubergarm** commented on **2025-08-01** at **22:05:27**

> I recommend using the llama.cpp script for this. Which is what I did. So not too sure if I had ported all the right changes. There are no changes to the _update.py version and I don't know what this version of the script is used for.

Got it, yes I am now using `sammcj	git@github.com:sammcj/llama.cpp.git glm-4-5@3d15c4a94` and converting Air currently. I have not added the convert logic for Hunyuan-A13B and previous THUDM dense model either, but only implemented the cpp code.

For this PR will likely want to either:
1. drop the convert script as it is broken currently and focus on the cpp code then follow up later with another PR for it
2. spend a little time now to fix it up and bring it along as it is already started

Just some thoughts. I'll keep going, and other good news this seems to be working now: GLM-4.5-IQ4_KSS.gguf 178.396 GiB (4.276 BPW)

---

👤 **ubergarm** commented on **2025-08-01** at **22:22:20**

@Thireus 

Sorry for spamming you, hah.. I think this patch is enough to get Air to convert here with this script. Interestingly needed a checksum from an older GLM model hah... should Be able to save it as `patch.diff` and run `git apply patch.diff` and push the change if u like.

```diff
diff --git a/convert_hf_to_gguf.py b/convert_hf_to_gguf.py
index c2748822..b1ca9e1c 100644
--- a/convert_hf_to_gguf.py
+++ b/convert_hf_to_gguf.py
@@ -618,6 +618,9 @@ class Model:
         if chkhsh == "b6e8e1518dc4305be2fe39c313ed643381c4da5db34a98f6a04c093f8afbe99b":
             # ref: https://huggingface.co/THUDM/glm-4-9b-chat
             res = "chatglm-bpe"
+        if chkhsh == "a1336059768a55c99a734006ffb02203cd450fed003e9a71886c88acf24fdbc2":
+            # ref: https://huggingface.co/THUDM/glm-4-9b-hf
+            res = "glm4"
         if chkhsh == "9ca2dd618e8afaf09731a7cf6e2105b373ba6a1821559f258b272fe83e6eb902":
             # ref: https://huggingface.co/zai-org/GLM-4.5-Air, https://huggingface.co/zai-org/GLM-4.5
             res = "gpt-2"
@@ -3951,8 +3954,8 @@ class Dots1Model(Qwen2MoeModel):
             return [(self.map_tensor_name(name), data_torch)]
         return super().modify_tensors(data_torch, name, bid)

-@ModelBase.register("Glm4MoeForCausalLM")
-class Glm4MoeModel(TextModel):
+@Model.register("Glm4MoeForCausalLM")
+class Glm4MoeModel(Model):
     model_arch = gguf.MODEL_ARCH.GLM4_MOE

     def __init__(self, *args, **kwargs):
```

Keep in mind I believe some things have changed in that draft PR and there could be a few other things. I'm gonna keep going with the convert Air from mainline for now as it is about 70% done.

---

👤 **InfernalDread** commented on **2025-08-01** at **22:26:18**

not sure if that "res" needs to be changed from "gpt-2" to "glm4"

---

👤 **ubergarm** commented on **2025-08-01** at **23:31:28**

I used mainline convert on Air and quantized a Q8_0 but got this trying to start up ik_llama.cpp llama-server:

```
llama_model_load: error loading model: done_getting_tensors: wrong number of tensors; expected 803, got 757
```

Unfortunately I'm out of time to play with this today. I may try using this PRs convert and look closer at the tensors when I have some more time.

---

👤 **TheLegendOfKitty** started a conversation on `src/llama.cpp` on **2025-08-02** at **03:17:13**

Is MTP on the roadmap for ik_llama?

---

👤 **TheLegendOfKitty** approved this pull request ✅ on **2025-08-02** at **03:18:16**

---

👤 **ubergarm** commented on **2025-08-02** at **05:40:37**

Okay, having some luck with GLM-4.5-Air now. Cooking imatrix off the bf16 and testing the Q8_0 on llama-server so far so good!

```bash
compute_imatrix: tokenizing the input ..
compute_imatrix: tokenization took 899.288 ms
compute_imatrix: computing over 814 chunks with batch_size 512
compute_imatrix: 7.09 seconds per pass - ETA 1 hours 36.25 minutes
[1]18.4611,[2]7.6556,[3]4.9280,[4]3.5268,[5]2.8022,[6]2.3688,[7]2.1118,[8]1.9437,[9]1.9381,
save_imatrix: entry '             blk.46.ffn_down_exps.weight' has partial data (96.88%) 4 out of 128 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.46.ffn_up_exps.weight' has partial data (96.88%) 4 out of 128 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.46.ffn_gate_exps.weight' has partial data (96.88%) 4 out of 128 experts are missing data Storing **but be aware**

save_imatrix: stored collected data after 10 chunks in /mnt/raid/models/ubergarm/GLM-4.5-Air-GGUF/imatrix-GLM-4.5-Air-BF16.dat
[10]1.8529,[11]1.9772,[12]2.1353,[13]2.2048,[14]2.2779,
```

```
llm_load_print_meta: model type       = 106B.A12B
llm_load_print_meta: model ftype      = Q8_0
llm_load_print_meta: model params     = 109.194 B
llm_load_print_meta: model size       = 108.119 GiB (8.505 BPW)
llm_load_print_meta: repeating layers = 106.891 GiB (8.505 BPW, 107.952 B parameters)
llm_load_print_meta: general.name     = GLM 4.5 Air
```

I was thinking about what @TheLegendOfKitty said and I was like "psure MTP is *not* on the roadmap" so I just removed those MTP tensors and seems to be happy now. The other clue was on startup before this next patch here it was complaining with: `llama_model_load: error loading model: done_getting_tensors: wrong number of tensors; expected 803, got 757`. So 803-757 is 46 which psure is the number of repeating layers. So it was as if there were one tensor per layer missing.  Thinking on it some more I was like "isn't there some need to set unused tensors with some kind of no-op?" but I don't recall so I just tried to remove adding those MTP tensors because I don't even know what MTP means.

So anyway I used this PR plus this patch on `convert_hf_to_gguf.py` and managed to convert GLM-4.5-Air and run it successfully now too. There might be a better way to go about it like is there a way to mark those extra tensors as noop so they don't cause a fuss? Also I added a comment because the mainline PR is using a different name ".ffn_gate_inp.bias" here is ".exp_probs_b" on mainline, so mainline couldn't run the Q8_0 throwing `llama_model_load: error loading model: missing tensor 'blk.3.exp_probs_b'` so I couldn't test it over there.

<details>

<summary>👈 Patch</summary>

```diff
diff --git a/convert_hf_to_gguf.py b/convert_hf_to_gguf.py
index c2748822..5da48e24 100644
--- a/convert_hf_to_gguf.py
+++ b/convert_hf_to_gguf.py
@@ -618,6 +618,9 @@ class Model:
         if chkhsh == "b6e8e1518dc4305be2fe39c313ed643381c4da5db34a98f6a04c093f8afbe99b":
             # ref: https://huggingface.co/THUDM/glm-4-9b-chat
             res = "chatglm-bpe"
+        if chkhsh == "a1336059768a55c99a734006ffb02203cd450fed003e9a71886c88acf24fdbc2":
+            # ref: https://huggingface.co/THUDM/glm-4-9b-hf
+            res = "glm4"
         if chkhsh == "9ca2dd618e8afaf09731a7cf6e2105b373ba6a1821559f258b272fe83e6eb902":
             # ref: https://huggingface.co/zai-org/GLM-4.5-Air, https://huggingface.co/zai-org/GLM-4.5
             res = "gpt-2"
@@ -3951,8 +3954,8 @@ class Dots1Model(Qwen2MoeModel):
             return [(self.map_tensor_name(name), data_torch)]
         return super().modify_tensors(data_torch, name, bid)

-@ModelBase.register("Glm4MoeForCausalLM")
-class Glm4MoeModel(TextModel):
+@Model.register("Glm4MoeForCausalLM")
+class Glm4MoeModel(Model):
     model_arch = gguf.MODEL_ARCH.GLM4_MOE

     def __init__(self, *args, **kwargs):
@@ -4093,7 +4096,7 @@ class Glm4MoeModel(TextModel):
         # Handle expert gating input (routing gate)
         if ".mlp.gate.e_score_correction_bias" in name:
             new_name = name.replace("model.layers.", "blk.").replace(
-                ".mlp.gate.e_score_correction_bias", ".ffn_gate_inp.bias"
+                ".mlp.gate.e_score_correction_bias", ".ffn_gate_inp.bias" # *NOTE* this is ".exp_probs_b" in mainline PR
             )
             return [(new_name, data_torch)]
         elif ".mlp.gate.weight" in name:
@@ -4132,6 +4135,7 @@ class Glm4MoeModel(TextModel):
             return [(self.map_tensor_name(new_name), data_torch)]

         # Handle special NextN tensors - preserve for future MTP support
+        # What are these MTP tensors? If we preserve them do they need to be "noop" or whatever?
         if (
             ".embed_tokens." in name
             or ".shared_head." in name
@@ -4140,7 +4144,9 @@ class Glm4MoeModel(TextModel):
             or ".hnorm." in name
         ):
             new_name = name.replace("model.layers.", "blk.").replace("model.", "").replace(".weight", "")
-            return [(new_name, data_torch)]
+            logger.debug(f"Skipping MTP tensor: {new_name}")
+            return []
+            #return [(new_name, data_torch)]

         # GLM tensor mapping - handle directly without map_tensor_name
         if ".input_layernorm." in name:
```

</details>

If this imatrix cooks I'll upload that at least and see about a ~4BPW quant or not depending on if the tensors are nice sizes etc.

---

👤 **Thireus** commented on **2025-08-02** at **05:50:17**

@ubergarm - Don't we want to preserve the MTP tensors for future MTP support? I see you've set `return[]`. Would the script work without this change?

---

👤 **Thireus** commented on **2025-08-02** at **05:53:33**

@ubergarm - More info about MTP can be found here: https://github.com/ggml-org/llama.cpp/pull/13236

Also discussed here: https://github.com/ggml-org/llama.cpp/pull/14939#discussion_r2243715997

---

👤 **ubergarm** commented on **2025-08-02** at **05:59:48**

@Thireus 

I removed them just to see if would get it working, which seems like it does. There is a commit floating around with some code to keep them but then skip them so it doesn't error but I haven't looked fully: https://github.com/ggml-org/llama.cpp/pull/14939/commits/dbfadb661e15e080df9962dca35f23eba9610955#diff-36e262e316ec1404e29880eb8b8ce4660ac584f0d0434710efc48a66497bdb59R4400-R4401

So If you want to keep them, probably have to make sure it doesn't cause an error on loading.

Also when starting up it autodetects the wrong template psure so you can use `--chat-template chatglm4`

```
numactl -N 0 -m 0 \
./build/bin/llama-server \
    --model "$model"\
    --alias ubergarm/GLM-4.5-Air-Q8_0 \
    --ctx-size 122880 \
    -fa -fmoe \
    -ctk q8_0 -ctv q8_0 \
    --chat-template chatglm4 \
    -ub 4096 -b 4096 \
    --parallel 3 \
    --threads 128 \
    --threads-batch 192 \
    --numa numactl \
    --host 127.0.0.1 \
    --port 8080 \
    --no-mmap
```

---

👤 **Thireus** started a conversation on `src/llama.cpp` on **2025-08-02** at **06:34:55**

Still need to figure out if this dirty patch to bypass the assert is still needed

---

👤 **ubergarm** commented on **2025-08-02** at **07:06:45**

Okay I gotta sleep and will be busy this weekend, but a couple parting gifts for you @Thireus 

> but is it possible for a GLM-4.5-Air 4 bit GGUF quant to test as well, as it is the much friendlier option for resource limited individuals (such as myself).

1. GLM-4.5-Air imatrix from the bf16: https://huggingface.co/ubergarm/GLM-4.5-Air-GGUF/blob/main/imatrix-GLM-4.5-Air-BF16.dat
2. https://huggingface.co/ubergarm/GLM-4.5-Air-GGUF/tree/main/IQ4_KSS

```bash
llm_load_print_meta: model type       = 106B.A12B
llm_load_print_meta: model ftype      = IQ4_KSS - 4.0 bpw
llm_load_print_meta: model params     = 109.194 B
llm_load_print_meta: model size       = 54.124 GiB (4.258 BPW)
llm_load_print_meta: repeating layers = 53.320 GiB (4.243 BPW, 107.952 B parameters)
llm_load_print_meta: general.name     = GLM 4.5 Air
```

I've tested a bit locally and seems to be working. I've included how to build this PR and an example command how to run llama-server as well for folks to test. Hope this helps and thanks a lot everyone for working together to figure this one out as the model seems pretty good for the size imo!

*UPDATE*:

Instructions how to test here: https://huggingface.co/ubergarm/GLM-4.5-Air-GGUF#quick-start

Seems to use about ~11GB VRAM with 32k context and q8_0 kv-cache with the rest on RAM. In testing on 2x RTX A6000 48GB VRAM (each, the older non blackwell non pro ones) it gets about 50 tok/sec TG. However, when I throw longer context prompt like 3.5k of python to refactor the streaming response seems to freeze up despite the GPU utilization remaining high. Maybe the other PR had some more things with bos tokens etc or mentioned streaming response getting hung up? Dunno, past my bed time :zzz: 

Okie, good night have fun!

---

👤 **createthis** commented on **2025-08-02** at **15:02:39**

I just tried this patch with Q8_0 generated by llama.cpp with the current patch there. Got this error:

> llama_model_load: error loading model: done_getting_tensors: wrong number of tensors; expected 1761, got 1671

So, it looks like this branch needs to be updated to skip the MTP tensors.

---

👤 **ubergarm** commented on **2025-08-02** at **15:05:30**

@createthis 

> So, it looks like this branch needs to be updated to skip the MTP tensors.

Correct, I linked the relevent PR above. My test quant was created *without* the MTP tensors as I hadn't seen the code to skip the MTP tensors at the time. 

I just got some perplexity values for Air-Q8_0 with and without `-fa` and looking reasonable here at around ~4.6. This is that comment on the mainline PR: https://github.com/ggml-org/llama.cpp/pull/14939#issuecomment-3146551088

---

👤 **createthis** commented on **2025-08-02** at **16:09:34**

I rebuilt the GGUF without MTP using this patch:

```diff
diff --git a/convert_hf_to_gguf.py b/convert_hf_to_gguf.py
index b386177c..dcf02770 100755
--- a/convert_hf_to_gguf.py
+++ b/convert_hf_to_gguf.py
@@ -6785,7 +6785,9 @@ class Glm4MoeModel(TextModel):
             or ".hnorm." in name
         ):
             new_name = name.replace("model.layers.", "blk.").replace("model.", "").replace(".weight", "")
-            return [(new_name, data_torch)]
+            logger.debug(f"Skipping MTP tensor: {new_name}")
+            return []
+            #return [(new_name, data_torch)]
 
         # GLM tensor mapping - handle directly without map_tensor_name
         if ".input_layernorm." in name:
```

I'll try patching ik_llama next, but reproducing what ubergarm has already done was the fastest path forward.

Startup command:

```bash
./build/bin/llama-server \
    --model /data/GLM-4.5-GGUF-no-MTP/q8_0/Main-160x21B-Q8_0-00001-of-00008.gguf \
    --alias GLM-4.5-GGUF:q8_0 \
    --numa numactl \
    --threads 32 \
    --ctx-size 131072 \
    --n-gpu-layers 94 \
    -ot "blk\.(3|4|5|6|7|8)\.ffn_.*=CUDA0" \
    -ot exps=CPU \
    -ub 4096 -b 4096 \
    --seed 3407 \
    --temp 0.3 \
    --min-p 0.0 \
    --flash-attn \
    --host 0.0.0.0 \
    -mla 3 \
    -fmoe \
    -rtr \
    --chat-template chatglm4 \
    --port 11434
```

I noticed this during startup:

```
 MLA is only available for LLM_ARCH_DEEPSEEK2 -> turning off MLA
```

EDIT: Actually, looking at architecture info, GLM-4.5 may not be MLA. nvm.

I can also confirm that `--chat-template chatglm4` was necessary.

I didn't realize `GLM-4.5` was a thinking model. Is there any way to turn that off?

---

👤 **ubergarm** commented on **2025-08-02** at **16:39:06**

> EDIT: Actually, looking at architecture info, GLM-4.5 may not be MLA. nvm.

Correct, `-mla` is only for DeepSeek/Kimi-K2 MLA architechture models. It does not apply here on any of the GLM models. Also remove `-rtr` when using `-ub 4096 -b 4096` in general for faster PP these days.  Also remove `--numa numactl` if you're not launching with `numactl ...`, only relevant on larger multi-numa servers.

> I didn't realize GLM-4.5 was a thinking model. Is there any way to turn that off?

Correct both GLM-4.5 and GLM-4.5-AIr seem to be "hybrid reasoning" meaning you can enable/disable thinking. Here is a note from the official github: https://github.com/zai-org/GLM-4.5?tab=readme-ov-file#request-parameter-instructions

I'm not 100% if that `enable_thinking` argument would work on (ik_)llama.cpp server or not or if you can simply pass in `/no_thinking` in the promptor something similar like other hybrid reasoning models.

I'm curious if your responses are stopping after a few hundred tokens generated on longish context prompts (e.g. 3-4k tokens prompt). I've seen this behavior before psure, but can't recall what the issue was right now.

---

👤 **Thireus** commented on **2025-08-02** at **19:03:42**

@createthis, @ubergarm - thanks for testing, I will try to find which exact llama.cpp commit I used to produce the BF16 version I have on huggingface, maybe this empty return is there for it. And will apply the change if that's indeed the case.

I am currently dealing with a faulty RAM stick, which will keep me busy for a few hours :(

---

👤 **createthis** commented on **2025-08-02** at **22:03:50**

I have to do a thing for a bit, but so far the only thing I've noticed is this apparent `Q8_0` vs `F32` discrepancy:

```bash
(/data/conda-envs/llama_cpp_cu128) jesse@larry:~/llama.cpp$ ./gguf-py/gguf/scripts/gguf_dump.py /data/GLM-4.5-GGUF-no-MTP/q8_0/Main-160x21B-Q8_0-00008-of-00008.gguf | grep shared_head
INFO:gguf-dump:* Loading: /data/GLM-4.5-GGUF-no-MTP/q8_0/Main-160x21B-Q8_0-00008-of-00008.gguf
    153:  775946240 |  5120, 151552,     1,     1 | Q8_0    | blk.92.shared_head.head
    177:       5120 |  5120,     1,     1,     1 | F32     | blk.92.shared_head.norm
(/data/conda-envs/llama_cpp_cu128) jesse@larry:~/llama.cpp$ ./gguf-py/gguf/scripts/gguf_dump.py /data/GLM-4.5-GGUF/q8_0/Main-160x21B-Q8_0-00008-of-00008.gguf | grep shared_head
INFO:gguf-dump:* Loading: /data/GLM-4.5-GGUF/q8_0/Main-160x21B-Q8_0-00008-of-00008.gguf
    170:  775946240 |  5120, 151552,     1,     1 | F32     | blk.92.shared_head.head
    194:       5120 |  5120,     1,     1,     1 | F32     | blk.92.shared_head.norm
(/data/conda-envs/llama_cpp_cu128) jesse@larry:~/llama.cpp$
```

I also noticed the first file is a bit different:

```bash
(/data/conda-envs/llama_cpp_cu128) jesse@larry:~/llama.cpp$ ./gguf-py/gguf/scripts/gguf_dump.py /data/GLM-4.5-GGUF/q8_0/Main-160x21B-Q8_0-00001-of-00008.gguf | tail
INFO:gguf-dump:* Loading: /data/GLM-4.5-GGUF/q8_0/Main-160x21B-Q8_0-00001-of-00008.gguf
    228:    5242880 |  5120,  1024,     1,     1 | Q8_0    | blk.12.attn_k.weight
    229:   62914560 | 12288,  5120,     1,     1 | Q8_0    | blk.12.attn_output.weight
    230:        128 |   128,     1,     1,     1 | F32     | blk.12.attn_q_norm.weight
    231:      12288 | 12288,     1,     1,     1 | F32     | blk.12.attn_q.bias
    232:   62914560 |  5120, 12288,     1,     1 | Q8_0    | blk.12.attn_q.weight
    233:       1024 |  1024,     1,     1,     1 | F32     | blk.12.attn_v.bias
    234:    5242880 |  5120,  1024,     1,     1 | Q8_0    | blk.12.attn_v.weight
    235:       5120 |  5120,     1,     1,     1 | F32     | blk.13.attn_norm.weight
    236: 1258291200 |  1536,  5120,   160,     1 | Q8_0    | blk.13.ffn_down_exps.weight
    237: 1258291200 |  5120,  1536,   160,     1 | Q8_0    | blk.13.ffn_gate_exps.weight
(/data/conda-envs/llama_cpp_cu128) jesse@larry:~/llama.cpp$
```

```bash
(/data/conda-envs/llama_cpp_cu128) jesse@larry:~/llama.cpp$ ./gguf-py/gguf/scripts/gguf_dump.py /data/GLM-4.5-GGUF-no-MTP/q8_0/Main-160x21B-Q8_0-00001-of-00008.gguf | tail
INFO:gguf-dump:* Loading: /data/GLM-4.5-GGUF-no-MTP/q8_0/Main-160x21B-Q8_0-00001-of-00008.gguf
    245:        128 |   128,     1,     1,     1 | F32     | blk.13.attn_k_norm.weight
    246:       1024 |  1024,     1,     1,     1 | F32     | blk.13.attn_k.bias
    247:    5242880 |  5120,  1024,     1,     1 | Q8_0    | blk.13.attn_k.weight
    248:   62914560 | 12288,  5120,     1,     1 | Q8_0    | blk.13.attn_output.weight
    249:        128 |   128,     1,     1,     1 | F32     | blk.13.attn_q_norm.weight
    250:      12288 | 12288,     1,     1,     1 | F32     | blk.13.attn_q.bias
    251:   62914560 |  5120, 12288,     1,     1 | Q8_0    | blk.13.attn_q.weight
    252:       1024 |  1024,     1,     1,     1 | F32     | blk.13.attn_v.bias
    253:    5242880 |  5120,  1024,     1,     1 | Q8_0    | blk.13.attn_v.weight
    254:       5120 |  5120,     1,     1,     1 | F32     | blk.14.attn_norm.weight
(/data/conda-envs/llama_cpp_cu128) jesse@larry:~/llama.cpp$
```

No idea if that's significant or not.

It seems odd to me that it complains of fewer tensors in the file that fails (the one WITH MTP):

```
llama_model_load: error loading model: done_getting_tensors: wrong number of tensors; expected 1761, got 1653
```

Why does it find fewer when the file has more?

---

👤 **espen96** commented on **2025-08-03** at **00:57:18**

Been testing the quant ubergarm made with this PR.

I got to around 7-8k tokens before the model appeared to be missing messages and responding to older ones. /nothink also broke down. I tested that at around 4k tokens? and it quickly got funny. responding normally inside a thinking block, thinking sometimes happening, sometimes not.

overall the model appears to be doing fine, besides the /nothink errors and this context issue.
used this to run the model with llama-server
`      ${ctx-32k}
      -fa -fmoe 
      -ctk q8_0 -ctv q8_0 
      --chat-template chatglm4 
      -ngl 999 
      -ot "blk\.(1[0-9]|[2-9][0-9])\.ffn_.*_exps=CPU"
      -amb 512 
      --parallel 1 
      --threads -1 
      --no-mmap
      -ts 100,0
      --main-gpu 0
      -ub 4096 -b 4096 
      `
      

<img width="1061" height="178" alt="image" src="https://github.com/user-attachments/assets/79cd7d30-489c-4216-ba41-671d9e606ceb" />

<img width="1065" height="219" alt="image" src="https://github.com/user-attachments/assets/f1c62ed6-2f25-4273-82ad-d05ce021f70b" />

<img width="1038" height="606" alt="image" src="https://github.com/user-attachments/assets/93286471-d0fc-43a3-b7d9-84ca39359a39" />

---

👤 **createthis** commented on **2025-08-03** at **03:47:35**

P.S. The performance honestly isn't terrible, considering I'm running Q8_0:

```
INFO [           print_timings] prompt eval time     =   45882.21 ms / 12542 tokens (    3.66 ms per token,   273.35 tokens per second) 
| tid="131708593647616" timestamp=1754191342 id_slot=0 id_task=1209 t_prompt_processing=45882.209 n_prompt_tokens_processed=12542 t_toke
n=3.658284882793813 n_tokens_second=273.3521396060072
INFO [           print_timings] generation eval time =   52582.74 ms /   655 runs   (   80.28 ms per token,    12.46 tokens per second) 
```

---

👤 **Thireus** commented on **2025-08-03** at **06:24:27**

This is the commit that I've used to produce BF16, which doesn't drop the MTP-related tensors. Please use this version. I'll go ahead and edit the ik_llama.cpp `convert_hf_to_gguf.py` script so that it uses the same logic.

```
# ggml-org/llama.cpp commit 7f026fbdbbe3289d0a4bf5f3a2850de1668c4600 with incorrect glm4moe.block_count - sha256sum GLM-4-00001-of-01762.gguf:
# 7de2e62d49ee81295ce8cad03bd72cd38bb0facb692016e8b5e4caf98c266ef as seen on https://huggingface.co/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/blob/b02b39c54e491b4cbef57cb7362055262ea4b049/GLM-4.5-THIREUS-BF16-SPECIAL_TENSOR-00001-of-01762.gguf

# ggml-org/llama.cpp commit 9a249d6fba9a736aab9ed3efba729e231aacf09d with correct glm4moe.block_count - sha256sum GLM-4-00001-of-01762.gguf:
# c5a7f219db1c4daa8830cc14bb5922edc330d656ce55c063eb7c065739fe77e0 as seen on https://huggingface.co/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/blob/f27f808d917cf614b7c85d4e4ed41fdf6bc0ae89/GLM-4.5-THIREUS-BF16-SPECIAL_TENSOR-00001-of-01762.gguf

# Clone llama.cpp repo and fetch commit
git clone https://github.com/ggml-org/llama.cpp --recursive
cd llama.cpp  # Or clone again if you haven't yet
git fetch --all --tags --prune
#git fetch origin 7f026fbdbbe3289d0a4bf5f3a2850de1668c4600
#git checkout 7f026fbdbbe3289d0a4bf5f3a2850de1668c4600
git fetch origin 9a249d6fba9a736aab9ed3efba729e231aacf09d
git checkout 9a249d6fba9a736aab9ed3efba729e231aacf09d

# Install required dependencies... the usual.
...

# Convert
cd ~/AI/fp8-to-bf16/llama.cpp
ulimit -n 99999
python convert_hf_to_gguf.py \
     --outtype bf16 \
     --outfile ~/AI/GLM-4.5/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/GLM-4.5-THIREUS-BF16-SPECIAL_TENSOR \
     --no-tensor-first-split --split-max-tensors 1 \
     ~/AI/huggingface/GLM-4.5

Evidence:
---
~/AI/GLM-4.5/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT$ sha256sum * # As seen on https://huggingface.co/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/tree/f27f808d917cf614b7c85d4e4ed41fdf6bc0ae89
c5a7f219db1c4daa8830cc14bb5922edc330d656ce55c063eb7c065739fe77e0  GLM-4-00001-of-01762.gguf
5c79829a85672017d549c1cdbeb198a73abe0b532a18c0b02ccff18622a54400  GLM-4-00002-of-01762.gguf
87075db98a79f76dd77806cada9cc95e732462a85bcf3646816513b7b95e1051  GLM-4-00003-of-01762.gguf
79c546ddc3aabf5943e41545a0b5a1add6359d21b40d79adde7fb6efd908e677  GLM-4-00004-of-01762.gguf
992ac053771e596dce461fd346909a050669f6636f756080c9ebe5f4f75e051a  GLM-4-00005-of-01762.gguf
20cd4b7f342b665e729c17d37ef55926e6d0214557766e650f6e9869e9fd5a25  GLM-4-00006-of-01762.gguf
e12e9883ffc61e8ecfbac1c8239e65b72922bd1a0bf7ce0ca4bab115edae15b2  GLM-4-00007-of-01762.gguf
74a1ca34076563290f2db8ea6a883034dcf5a7d9a9fa564585f82b179b40f703  GLM-4-00008-of-01762.gguf
379f306a61cda447622216c8f27cc8c76a2f022ae163351d85bed2100e082080  GLM-4-00009-of-01762.gguf
6ad58b88826d30df9088425f3a94171a32014a3a0b80e1c9814cca6f1153069a  GLM-4-00010-of-01762.gguf
6410498a7b54cf7b23f6d8e1b1a775916c373a9f75f980cdde433ba2f402cb32  GLM-4-00011-of-01762.gguf
35004982cbcaf56ec640decf6c919e712d38b2939cdc52380252f145d6ba6cc6  GLM-4-00012-of-01762.gguf
3918e605ac3c01a7724752b88f2cb4aed02acb6c15e4ab5bd38a33a01da222b4  GLM-4-00013-of-01762.gguf
...
---
```

---

👤 **Thireus** commented on **2025-08-03** at **06:46:37**

I can confirm the chat template isn't added to the gguf when using ik_llama.cpp's script... "special_vocab.chat_template" returns None. So I believe it's either not supported by ik_llama.cpp or there is another way to add the template.

```
python convert_hf_to_gguf.py  ~/AI/huggingface/GLM-4.5 --vocab-only
strings ~/AI/huggingface/GLM-4.5/GLM-4.5-vocab.gguf | grep tokenizer.chat_template # Returns nothing
```

---

👤 **Thireus** commented on **2025-08-03** at **07:39:39**

Ok, the issue with the chat_template not being added when using our ik_llama version of the `convert_hf_to_gguf.py` is that we use an old version of vocab.py which doesn't support template coming from .jinja files but only templates coming from .json files.

Relevant lines:
https://github.com/ggml-org/llama.cpp/blob/master/gguf-py/gguf/vocab.py#L269-L277

GLM-4.5 repos only provide .jinja templates: https://huggingface.co/zai-org/GLM-4.5/blob/main/chat_template.jinja

I'm just tempted to use the updated vocab.py file from llama.cpp project as part of this pull request...

---

👤 **Thireus** commented on **2025-08-03** at **07:51:43**

That's done, I've updated the vocab.py to support jinja chat templates in this pull request. You should now be able to use `convert_hf_to_gguf.py` from ik_llama to produce good BF16 ggufs that contain the chat template and the MTP-related tensors.

Note: You won't get the same sha256 hash of the header 00001 file since ik_llama reports a different general.quantization_version. But everything else should be the same as https://huggingface.co/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/blob/f27f808d917cf614b7c85d4e4ed41fdf6bc0ae89/GLM-4.5-THIREUS-BF16-SPECIAL_TENSOR-00001-of-01762.gguf.

```
$ diff strings1.txt strings3.txt 
37d36
< general.quantization_version
324515a324515
> general.quantization_version
```

Please report back if that works.

---

👤 **Thireus** commented on **2025-08-03** at **08:52:34**

This PR should be in a better state now, you should be able to:
- Convert GLM4.5 to BF16 using the `convert_hf_to_gguf.py` ik_llama script
- Quantize the BF16 model to any quant you like using ik_llama's `llama-quantize`
- Run the quantized GLM4.5 using  `llama-server`, other binaries such as `llama-perplexity` should work too, with and without flash-attention. No need to add the chat template since it's now already included in the ggufs produced.
- ~`/no_think`~ `/nothink` should work as expected, and the model should give good answers to prompts and not collapse (contrary to what is observed on the llama.cpp fork)

```
git clone https://github.com/ikawrakow/ik_llama.cpp
cd ik_llama.cpp
git remote add Thireus https://github.com/Thireus/ik_llama.cpp.git
git fetch Thireus
git checkout glm-4.5-clean
git pull
```

Can someone confirm this all works now?
For my part this all works with GLM4.5 - https://huggingface.co/zai-org/GLM-4.5

---

👤 **espen96** commented on **2025-08-03** at **11:51:55**

Currently downloading 4.5-Air off hf.
Could take a while. I'll report back after I've attempted to convert it... Probably in a few hours

---

👤 **createthis** commented on **2025-08-03** at **14:25:13**

FYI, llama.cpp GGUFs have diverged from this PR. If you try to load a llama.cpp GGUF with this PR you get:

```
llama_model_load: error loading model: check_tensor_dims: tensor 'blk.0.ffn_norm.weight' not found
```

---

👤 **Thireus** commented on **2025-08-03** at **14:36:03**

> FYI, llama.cpp GGUFs have diverged from this PR. If you try to load a llama.cpp GGUF with this PR you get:
> 
> ```
> llama_model_load: error loading model: check_tensor_dims: tensor 'blk.0.ffn_norm.weight' not found
> ```

Indeed, the llama.cpp codebase has undergone several patches and changes, some of which may not have been thoroughly documented or fully understood.
It might be worthwhile for them to revisit the known working state I have highlighted and incrementally reintroduce changes with clear rationale and validation at each step.

It's unfortunate that the previously working GGUF conversion script is getting ignored, and new testers are now working with a version that appears to be broken. Despite indications that a rollback to a validated version might be beneficial, there doesn't seem to be a clear effort to pursue that path.

---

👤 **espen96** commented on **2025-08-03** at **14:46:58**

update on my end: windows is not playing ball, had the script stop several times at various points during conversion from hf to gguf. Love linux, but it tends to choke up on this rig when I push it beyond the limits. I only got 64 gb of ram, so this is deep into paging or else I'd have switched over to that. 

70-80% into this run, hopefully it finishes. I was able to make some space on a faster ssd for this "project".
edit: it failed at 84%, darn it.


Hopefully we can get this model running properly. I think it is fair to say the progress so far across all the porting efforts I have seen can be summarized as "chaotic". The efforts put into this is obviously apprecianted, this is a very promising set of models. Bit of a shame we didn't get an official implementation this time

---

👤 **Thireus** commented on **2025-08-03** at **15:03:36**

> update on my end: windows is not playing ball, had the script stop several times at various points during conversion from hf to gguf. Love linux, but it tends to choke up on this rig when I push it beyond the limits. I only got 64 gb of ram, so this is deep into paging or else I'd have switched over to that.
> 
> 70-80% into this run, hopefully it finishes. I was able to make some space on a faster ssd for this "project". edit: it failed at 84%, darn it.
> 
> Hopefully we can get this model running properly. I think it is fair to say the progress so far across all the porting efforts I have seen can be summarized as "chaotic". The efforts put into this is obviously apprecianted, this is a very promising set of models. Bit of a shame we didn't get an official implementation this time

My notes for Linux:

```
# Install dependencies
apt install python3-dev python3-pip python3-venv python3-wheel python3-setuptools git acl netcat-openbsd cmake pipx

# Install uv via pipx
pipx install uv

# Prepare env
mkdir -p ~/AI/fp8-to-bf16
cd ~/AI/fp8-to-bf16
uv venv ./venv --python 3.12 --python-preference=only-managed

# Activate env
source venv/bin/activate

# Clone patched llama.cpp for glm-4.5
git clone https://github.com/ikawrakow/ik_llama.cpp
cd ik_llama.cpp
git remote add Thireus https://github.com/Thireus/ik_llama.cpp.git
git fetch Thireus
git checkout glm-4.5-clean
git pull
git rev-parse --short HEAD

# Install ik_llama.cpp dependencies
cd ~/AI/fp8-to-bf16/ik_llama.cpp
uv pip install -r requirements/requirements-convert_hf_to_gguf.txt --prerelease=allow --index-strategy unsafe-best-match
# Build ik_llama.cpp (optional)
cmake -B build -DGGML_AVX=ON -DGGML_AVX2=ON -DLLAMA_CURL=OFF
cmake --build build --config Release -j16
cd ..

# Build triton-cpu
git clone https://github.com/triton-lang/triton-cpu --recursive
cd triton-cpu
uv pip install ninja cmake wheel setuptools pybind11

# Apply this patch - https://github.com/ikawrakow/ik_llama.cpp/issues/383#issuecomment-2865306085
nano -w CMakeLists.txt
---
#  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Werror -Wno-covered-switch-default -fvisibility=hidden")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-covered-switch-default -fvisibility=hidden")
---
nano -w third_party/cpu/CMakeLists.txt
---
#find_package(dnnl CONFIG)
#if (dnnl_FOUND)
#... up until the endif
#endif()
---

# Install dependencies
uv pip install -r python/requirements.txt
apt-get update
apt-get install -y ccache
apt-get install -y --no-install-recommends \
    zlib1g-dev \
    libxml2-dev \
    libssl-dev \
    libgmp-dev \
    libmpfr-dev

# Compile
MAX_JOBS=16 uv pip install -e python --no-build-isolation

# Be patient, "Preparing Packages" downloads a lot of stuff before build begins...

# GLM-4.5
cd ~/AI/fp8-to-bf16/ik_llama.cpp
ulimit -n 99999
# commented out --split-max-tensors 1 to avoid confusing llama.cpp people
python convert_hf_to_gguf.py \
     --outtype bf16 \
     --outfile ~/AI/GLM-4.5/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/GLM-4.5-THIREUS-BF16-SPECIAL_TENSOR \
     --no-tensor-first-split \
     ~/AI/huggingface/GLM-4.5
```

This is CPU only; no GPU on the machine required. I don't know about RAM requirement but it's not that much.

---

👤 **createthis** commented on **2025-08-03** at **15:24:01**

Mine are similar for ubuntu 24.04.2:

<details>

```bash
conda create -p /data/conda-envs/llama_cpp_cu128 python=3.11

conda activate /data/conda-envs/llama_cpp_cu128
conda install -c conda-forge libstdcxx-ng

sudo apt install libtbb-dev libssl-dev libcurl4-openssl-dev libaio1t64 libaio-dev libfmt-dev libgflags-dev zlib1g-dev patchelf

pip3 install packaging ninja cpufeature numpy openai
pip3 install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu128

# build flash-attention
cd flash-attention
export CUDA_HOME=/usr/local/cuda
export TORCH_CUDA_ARCH_LIST="12.0"
TMPDIR=/data/tmp MAX_JOBS=32 python setup.py install

cd ~/ik_llama.cpp
cmake -B build -DGGML_CUDA=ON
cmake --build build --config Release
```

It assumes you have a blackwell GPU and that you've already installed nvidia drivers and cuda 12.8.1 though. I have to build flash-attention from source, which is surprisingly resource intensive. Some of those deps may be unnecessary as it's been cargo culted from ktransformers.

I put everything on `/data` because that's my multi-terabyte drive volume. The conda env is 7.8G and I have one for each AI project I use. 
</details>

---

👤 **espen96** commented on **2025-08-03** at **16:23:25**

awesome, all setup on Arch now.
it seems so far things are going well.  I am not sure it is using much of any ram at all actually. it seems to be relying entierly on the disk. 

It appears to have completed. 804 / 804 shards. no errors, no sudden unexpected stops.
I can't run this at this size, so quantization next.

I assume I can, and probably should, use ubergarms "Secret Recipe" and imatrix for this? unless something has changed that would invalidate that approach.

---

👤 **Nexesenex** commented on **2025-08-03** at **16:45:25**

Works like a charm on IKL server (Windows 11, exe) with https://huggingface.co/ubergarm/GLM-4.5-Air-GGUF/tree/main/IQ4_KSS
This, at 14k context.
Thanks, Thireus!

Edit : Well, I know it's not a review thread, so an edit and no additional comment, but GLM-4.5 Air is impressive. It competes with Command A 111b and even Mistral 123b with ease, despite being MOE and not dense. I'm delighted, supporting it is worth the effort.
And it works great with 7 experts, the perplexity being 0.005 lower than with 8.

---

👤 **espen96** commented on **2025-08-03** at **17:06:52**

ok, forgot the  -DGGML_MAX_CONTEXTS=2048 in the build parameters for IK, took me a hot minute to realize why it wouldn't quantize the 803 way split... ups.

in any case, everything appears to be going well right now.  at this rate I expect to have a fully quantized model in 30 minutes? 


since I did experience errors earlier, I will make sure to test this model up to 32k tokens and the same settings as before.
if it can keep a good track of the conversation, and doesn't spit out anything funny at such a high ctx, then it's probably fine. I'll also make sure to test the no_think by itself this time


good work so far!

---

👤 **createthis** commented on **2025-08-03** at **17:19:51**

It's inferring on my system too. I quantized to Q4_K_M. At first I ran it with `--chat-template chatglm4` and it stopped generating around 27k context, but then I tried it without that option and it went a little further but decohered around 29k context.

Snippet from the log around that time:

<details>

```
INFO [      log_server_request] request | tid="132614511681536" timestamp=1754241226 remote_addr="192.168.0.201" remote_port=62771 status=200 method="POST" path="/v1/chat/completions" params={}
INFO [            update_slots] all slots are idle | tid="132625045901312" timestamp=1754241226
INFO [      log_server_request] request | tid="132614471983104" timestamp=1754241270 remote_addr="192.168.0.201" remote_port=62780 status=200 method="GET" path="/v1/models" params={}
INFO [   launch_slot_with_task] slot is processing task | tid="132625045901312" timestamp=1754241270 id_slot=0 id_task=3532
INFO [            update_slots] kv cache rm [p0, end) | tid="132625045901312" timestamp=1754241270 id_slot=0 id_task=3532 p0=29522
INFO [           print_timings] prompt eval time     =    6802.79 ms /   711 tokens (    9.57 ms per token,   104.52 tokens per second) | tid="132625045901312" timestamp=1754241289 id_slot=0 id_task=3532 t_prompt_processing=6802.789 n_prompt_tokens_processed=711 t_token=9.567917018284106 n_tokens_second=104.51595661720509
INFO [           print_timings] generation eval time =   12428.17 ms /    87 runs   (  142.85 ms per token,     7.00 tokens per second) | tid="132625045901312" timestamp=1754241289 id_slot=0 id_task=3532 t_token_generation=12428.174 n_decoded=87 t_token=142.8525747126437 n_tokens_second=7.000223846238392
INFO [           print_timings]           total time =   19230.96 ms | tid="132625045901312" timestamp=1754241289 id_slot=0 id_task=3532 t_prompt_processing=6802.789 t_token_generation=12428.174 t_total=19230.963
INFO [            update_slots] slot released | tid="132625045901312" timestamp=1754241289 id_slot=0 id_task=3532 n_ctx=131072 n_past=30319 n_system_tokens=0 n_cache_tokens=30319 truncated=false
INFO [            update_slots] all slots are idle | tid="132625045901312" timestamp=1754241289
INFO [format_partial_response_oaicompat] DEBUG: Streaming finish_reason check | tid="132614463590400" timestamp=1754241289 generated_text="<think>We are looking at a diff snippet that shows the special tokens handling in the `set_vocab` method of the `Glm4MoeModel` class.\n The issue is that some of the token strings are empty or incomplete. Specifically, we see:\n   - `tokenizer.get_added_vocab()[\"\"]` (empty string) for \"bos\", \"unk\", and \"sop\"\n   - `tokenizer.get_added_vocab()[\"" model_name="GLM-4.5-GGUF:q4_k_m" tool_calls_count=0
```
</details>

Startup command:

```bash
./build/bin/llama-server \
    --model /data/GLM-4.5-GGUF/q4_k_m/GLM-4.5-Q4_K_M.gguf \
    --alias GLM-4.5-GGUF:q4_k_m \
    --numa numactl \
    --threads 32 \
    --ctx-size 131072 \
    --n-gpu-layers 94 \
    -ot "blk\.(3|4|5|6|7|8|9|10|11|12|13|14)\.ffn_.*=CUDA0" \
    -ot exps=CPU \
    -ub 4096 -b 4096 \
    --seed 3407 \
    --temp 0.3 \
    --min-p 0.0 \
    --flash-attn \
    --host 0.0.0.0 \
    -fmoe \
    --port 11434
```

Note: I run `numactl` on purpose because I have dual CPUs and it helps with generation speed on the main llama.cpp branch. I use NPS4 and run this first:

```bash
echo 0 | sudo tee /proc/sys/kernel/numa_balancing
echo 3 | sudo tee /proc/sys/vm/drop_caches
```

Here's what it looks like in Open WebUI when it stops generating randomly:
<img width="1244" height="756" alt="Screenshot 2025-08-03 at 1 14 58 PM" src="https://github.com/user-attachments/assets/151d9b6c-fa96-47e2-9743-c5a706e1b0bd" />

---

👤 **createthis** commented on **2025-08-03** at **17:41:00**

This prompt appears to insta-break it in Open WebUI:

````
I want to test something. Repeat this back to me as closely as you can:  
  
```diff  
+        # Special tokens
+        # Note: Using <|endoftext|> (151329) for eos and eot causes endless generation
+        special_vocab._set_special_token("bos", tokenizer.get_added_vocab()["[gMASK]"])  # 151331
+        special_vocab._set_special_token("eos", tokenizer.get_added_vocab()["<|user|>"])  # 151336 - end of
+        special_vocab._set_special_token("eot", tokenizer.get_added_vocab()["<|user|>"])  # 151336 - same as EOS
+        special_vocab._set_special_token("eog", tokenizer.get_added_vocab()["<|user|>"])  # 151336 - same as EOS
+        special_vocab._set_special_token("unk", tokenizer.get_added_vocab()["<|endoftext|>"]) # 151329
+        special_vocab._set_special_token("eom", tokenizer.get_added_vocab()["<|observation|>"])  # 151338
+
+        if "<sop>" in tokenizer.get_added_vocab():
+            special_vocab._set_special_token("sop", tokenizer.get_added_vocab()["<sop>"])  # 151333
+        if "<eop>" in tokenizer.get_added_vocab():
+            special_vocab._set_special_token("eop", tokenizer.get_added_vocab()["<eop>"])  # 151334
```
````

<img width="1129" height="707" alt="Screenshot 2025-08-03 at 1 39 22 PM" src="https://github.com/user-attachments/assets/4828cce7-dfc7-400c-bb0c-f09d8b447587" />

I can set up `mitmproxy` if you want a simple `curl` script to repro.

EDIT: `kimi-k2` successfully responds to this prompt

---

👤 **espen96** commented on **2025-08-03** at **17:54:01**

> This prompt appears to insta-break it in Open WebUI:
> 
> ```
> I want to test something. Repeat this back to me as closely as you can:  
>   
> ```diff  
> +        # Special tokens
> +        # Note: Using <|endoftext|> (151329) for eos and eot causes endless generation
> +        special_vocab._set_special_token("bos", tokenizer.get_added_vocab()["[gMASK]"])  # 151331
> +        special_vocab._set_special_token("eos", tokenizer.get_added_vocab()["<|user|>"])  # 151336 - end of
> +        special_vocab._set_special_token("eot", tokenizer.get_added_vocab()["<|user|>"])  # 151336 - same as EOS
> +        special_vocab._set_special_token("eog", tokenizer.get_added_vocab()["<|user|>"])  # 151336 - same as EOS
> +        special_vocab._set_special_token("unk", tokenizer.get_added_vocab()["<|endoftext|>"]) # 151329
> +        special_vocab._set_special_token("eom", tokenizer.get_added_vocab()["<|observation|>"])  # 151338
> +
> +        if "<sop>" in tokenizer.get_added_vocab():
> +            special_vocab._set_special_token("sop", tokenizer.get_added_vocab()["<sop>"])  # 151333
> +        if "<eop>" in tokenizer.get_added_vocab():
> +            special_vocab._set_special_token("eop", tokenizer.get_added_vocab()["<eop>"])  # 151334
> ```
> ```
> 
> <img alt="Screenshot 2025-08-03 at 1 39 22 PM" width="1129" height="707" src="https://private-user-images.githubusercontent.com/1695880/473804477-4828cce7-dfc7-400c-bb0c-f09d8b447587.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NTQyNDM4MTIsIm5iZiI6MTc1NDI0MzUxMiwicGF0aCI6Ii8xNjk1ODgwLzQ3MzgwNDQ3Ny00ODI4Y2NlNy1kZmM3LTQwMGMtYmIwYy1mMDlkOGI0NDc1ODcucG5nP1gtQW16LUFsZ29yaXRobT1BV1M0LUhNQUMtU0hBMjU2JlgtQW16LUNyZWRlbnRpYWw9QUtJQVZDT0RZTFNBNTNQUUs0WkElMkYyMDI1MDgwMyUyRnVzLWVhc3QtMSUyRnMzJTJGYXdzNF9yZXF1ZXN0JlgtQW16LURhdGU9MjAyNTA4MDNUMTc1MTUyWiZYLUFtei1FeHBpcmVzPTMwMCZYLUFtei1TaWduYXR1cmU9NGNiNjMzZTg5NTBiMmQzMzk3NWJhYmZjZWU5MjQ1YTYyN2E4ODYyMWMzOTM3ZmQ1MTBiMTU2ODVlY2ZmYzAyOCZYLUFtei1TaWduZWRIZWFkZXJzPWhvc3QifQ.hDamtyLDniY6SohKXAlANjtga7eQCJMtWH8CT5zL-KY">
> I can set up `mitmproxy` if you want a simple `curl` script to repro.
> 
> EDIT: `kimi-k2` successfully responds to this prompt

Ok, my assumption is that you just made the model output its eop token. which I assume is something like end of prompt or soemthing like that. in any case, I believe it read the token, and used the exact token rather than some safe variant, and it was cut off

---

👤 **createthis** commented on **2025-08-03** at **18:15:30**

> Ok, my assumption is that you just made the model output its eop token. which I assume is something like end of prompt or soemthing like that. in any case, I believe it read the token, and used the exact token rather than some safe variant, and it was cut off

That makes sense. So more of a fine tuning issue with the model itself.

---

👤 **Thireus** commented on **2025-08-03** at **18:18:46**

> ok, forgot the -DGGML_MAX_CONTEXTS=2048 in the build parameters for IK, took me a hot minute to realize why it wouldn't quantize the 803 way split... ups.
> 
> in any case, everything appears to be going well right now. at this rate I expect to have a fully quantized model in 30 minutes?
> 
> since I did experience errors earlier, I will make sure to test this model up to 32k tokens and the same settings as before. if it can keep a good track of the conversation, and doesn't spit out anything funny at such a high ctx, then it's probably fine. I'll also make sure to test the no_think by itself this time
> 
> good work so far!

Yes, alternatively you can also remove the `--split-max-tensors 1` option if you don't want to split the gguf.

---

👤 **espen96** commented on **2025-08-03** at **18:49:00**

<img width="1081" height="832" alt="image" src="https://github.com/user-attachments/assets/13124778-c917-486d-b445-a7b91192aace" /> not even 1k tokens in and it already messed up reasoning. I have not tried the toggle yet

---

👤 **tacos8me** commented on **2025-08-03** at **19:10:29**

Excellent work. I just tried big dad to Q4_K_M, and it's working fine initially.

./build/bin/llama-quantize /mnt/nvme-1/models/GLM-4.5/bf16-00001-of-01762.gguf /
 /mnt/nvme-1/models/GLM-4.5/q4_k_m/glm-q4_k_m.gguf /
 Q4_K_M

<img width="1529" height="886" alt="Screenshot 2025-08-03 150703" src="https://github.com/user-attachments/assets/5966b613-9cb0-4d3a-96b2-4082544ecc96" />

---

👤 **KonstantinGeist** commented on **2025-08-03** at **20:33:54**

Got the quants from https://huggingface.co/ubergarm/GLM-4.5-Air-GGUF and it seemed to work initially. In fact, it nailed my usual test question about my (relatively unknown) hometown - no open-weights model I tried before could do it!

However, after a few turns I got this:
```
Aug 03 23:27:21 aiworker01 llama-swap[2470794]: INFO [   launch_slot_with_task] slot is processing task | tid="138028751591680" timestamp=1754252841 id_slot=0 id_task=2
Aug 03 23:27:21 aiworker01 llama-swap[2470794]: INFO [            update_slots] kv cache rm [p0, end) | tid="138028751591680" timestamp=1754252841 id_slot=0 id_task=2 p0=0
Aug 03 23:27:21 aiworker01 llama-swap[2470794]: /home/konstantin.geyst/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1146: GGML_ASSERT(fms.S[j] > 0) failed
Aug 03 23:27:21 aiworker01 llama-swap[2470794]: /home/konstantin.geyst/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1146: GGML_ASSERT(fms.S[j] > 0) failed
Aug 03 23:27:21 aiworker01 llama-swap[2470794]: /home/konstantin.geyst/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1146: /home/konstantin.geyst/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1146: GGML_ASSERT(fms.S[j] > 0) failed
Aug 03 23:27:21 aiworker01 llama-swap[2470794]: /home/konstantin.geyst/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1146: /home/konstantin.geyst/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1146: /home/konstantin.geyst/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1146: GGML_ASSERT(fms.S[j] > 0) failed/home/konstantin.geyst/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1146: /home/konstantin.geyst/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1146: /home/konstantin.geyst/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1146: GGML_ASSERT(fms.S[j] > 0) failedGGML_ASSERT(fms.S[j] > 0) failedGGML_ASSERT(fms.S[j] > 0) failed
Aug 03 23:27:21 aiworker01 llama-swap[2470794]: /home/konstantin.geyst/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1146: GGML_ASSERT(fms.S[j] > 0) failed
Aug 03 23:27:21 aiworker01 llama-swap[2470794]: /home/konstantin.geyst/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1146: GGML_ASSERT(fms.S[j] > 0) failed
Aug 03 23:27:21 aiworker01 llama-swap[2470794]: GGML_ASSERT(fms.S[j] > 0) failed
Aug 03 23:27:21 aiworker01 llama-swap[2470794]: /home/konstantin.geyst/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1146: GGML_ASSERT(fms.S[j] > 0) failed
Aug 03 23:27:21 aiworker01 llama-swap[2470794]: GGML_ASSERT(fms.S[j] > 0) failed
Aug 03 23:27:21 aiworker01 llama-swap[2470794]: /home/konstantin.geyst/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1146: GGML_ASSERT(fms.S[j] > 0) failed
Aug 03 23:27:21 aiworker01 llama-swap[2470794]: /home/konstantin.geyst/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1146: GGML_ASSERT(fms.S[j] > 0) failed
Aug 03 23:27:21 aiworker01 llama-swap[2470794]: /home/konstantin.geyst/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1146: GGML_ASSERT(fms.S[j] > 0) failed
Aug 03 23:27:21 aiworker01 llama-swap[2470794]: GGML_ASSERT(fms.S[j] > 0) failed
Aug 03 23:27:22 aiworker01 llama-swap[2470794]: Could not attach to process.  If your uid matches the uid of the targe
```

I ran with the following command:
```
/home/konstantin.geyst/bin/ik_llama/llama-server --port 8600 --model /home/konstantin.geyst/models/WARNING-EXPERIMENTAL-IKLLAMACPP-ONLY-GLM-4.5-Air-IQ4_KSS-00001-of-00002.gguf -ub 4096 -b 4096 -ngl 0 --no-kv-offload --no-warmup --min-p 0.0 --top-k 20 --top-p 0.8 --temp 0.7 -np 1 --ctx-size 32768 --threads 16 -fa -fmoe -ctk q8_0 -ctv q8_0 --chat-template chatglm4
```
Recent commits from the glm-4.5-clean branch.

Maybe the quants are outdated now? Can someone share the up-to-date 4-bit quants? Or it's a bug somewhere?

---

👤 **ubergarm** commented on **2025-08-03** at **20:56:05**

@KonstantinGeist 

> Can someone share the up-to-date 4-bit quants? Or it's a bug somewhere?

From what I can tell both this PR and the PR on mainline are still in flux. Until support is stable and merged (and ideally similar enough that this fork can run mainline quants) there is no such thing as an "up-to-date" quant so to speak other then something you've created yourself.

Your assert issue is suggestive of some kind numerical issue psure, you can see some similar asserts I've been collecting over here: https://github.com/ikawrakow/ik_llama.cpp/issues/649

> /home/konstantin.geyst/bin/ik_llama/llama-server --port 8600 --model /home/konstantin.geyst/models/WARNING-EXPERIMENTAL-IKLLAMACPP-ONLY-GLM-4.5-Air-IQ4_KSS-00001-of-00002.gguf -ub 4096 -b 4096 -ngl 0 --no-kv-offload --no-warmup --min-p 0.0 --top-k 20 --top-p 0.8 --temp 0.7 -np 1 --ctx-size 32768 --threads 16 -fa -fmoe -ctk q8_0 -ctv q8_0 --chat-template chatglm4

Do you not have large enough GPU or why are you not offloading kv cache and no layers? Perhaps recompile with cpu-only backend disabling CUDA, or use the recommended model card example with -ngl 99 -ot exps=CPU style.

---

👤 **KonstantinGeist** commented on **2025-08-03** at **21:04:03**

>Until support is stable and merged (and ideally similar enough that this fork can run mainline quants) there is no such thing as an "up-to-date" quant so to speak other then something you've created yourself.

Yeah I know it's unstable, I hoped to help speed up the development by reporting crashes I encountered while testing, but I was not sure if the quants I downloaded were still "in sync" with the recent commits.

>Do you not have large enough GPU or why are you not offloading kv cache and no layers? Perhaps recompile with cpu-only backend disabling CUDA, or use the recommended model card example with -ngl 99 -ot exps=CPU style.

I run an internal server actively used by coworkers, and the main model there right now is Qwen3 which occupies the whole GPU. So being able to run new experimental LLMs on CPU in parallel to the GPU using ik_llama.cpp is very useful.

Sorry for derailing the thread.

---

👤 **ubergarm** commented on **2025-08-03** at **23:15:29**

@KonstantinGeist 

> Sorry for derailing the thread.

All good I appreciate the report, I'd suggest you compile a version of Ik_llama.cpp with CPU-only backend to run without using any VRAM and let us know if you still see those assserts just to be safe, but likely related to how you're running it I'm hoping and still useful information e.g.

```
# build CPU-only backend disabling CUDA/VULKAN GPU
cmake -B build -DCMAKE_BUILD_TYPE=Release -DGGML_CUDA=0 -DGGML_BLAS=0 -DGGML_VULKAN=0
cmake --build build --config Release -j $(nproc)

# adjust for your port and sampler settings etc
./build/bin/llama-server \
    --model "$model"\
    --alias ubergarm/GLM-4.5-Air-IQ4_KSS \
    --ctx-size 32768 \
    -fa -fmoe \
    -ctk q8_0 -ctv q8_0 \
    --chat-template chatglm4 \
    --override-kv tokenizer.ggml.eot_token_id=int:151336 \
    -ub 4096 -b 4096 \
    --parallel 1 \
    --threads 16 \
    --host 127.0.0.1 \
    --port 8080 \
    --no-mmap
```

---

👤 **KonstantinGeist** commented on **2025-08-03** at **23:57:33**

@ubergarm 
Same assert after recompiling and adjusting the parameters as you suggested.
Qwen3-235b works perfectly with ik_llama.cpp on the same machine (your IQ2_KL quants)
The assert mostly manifests around the 3rd turn (i.e. I type my 3rd message, hit ENTER, and it crashes), though not always - sometimes it happens on the 4th or 5th turn.
When the model is generating an answer, everything seems fine, it hasn't crashed mid sentence so far.
Let's see if others run into the same issue.

---

👤 **AesSedai** commented on **2025-08-04** at **08:08:07**

> This PR should be in a better state now, you should be able to:
> 
> * Convert GLM4.5 to BF16 using the `convert_hf_to_gguf.py` ik_llama script
> * Quantize the BF16 model to any quant you like using ik_llama's `llama-quantize`
> * Run the quantized GLM4.5 using  `llama-server`, other binaries such as `llama-perplexity` should work too, with and without flash-attention. No need to add the chat template since it's now already included in the ggufs produced.
> * `/no_think` should work as expected, and the model should give good answers to prompts and not collapse (contrary to what is observed on the llama.cpp fork)
> 
> ```
> git clone https://github.com/ikawrakow/ik_llama.cpp
> cd ik_llama.cpp
> git remote add Thireus https://github.com/Thireus/ik_llama.cpp.git
> git fetch Thireus
> git checkout glm-4.5-clean
> git pull
> ```
> 
> Can someone confirm this all works now? For my part this all works with GLM4.5 - https://huggingface.co/zai-org/GLM-4.5

@Thireus that all worked for me, under 32k tokens (will get back to that in a moment).

From `292300d2`, I compiled with
```
$ cmake -B build -DGGML_CUDA=ON -DGGML_BLAS=OFF -DGGML_SCHED_MAX_COPIES=1 -DGGML_CUDA_IQK_FORCE_BF16=1 -DCMAKE_BUILD_TYPE=Release
$ cmake --build build --config Release -j 48 --clean-first
```
Then converted the GLM4.5 HF to a BF16.gguf, then llama-quantized it to Q6_K and ran it with:
```
$ ./ik_llama.cpp/build/bin/llama-server \
    --n-gpu-layers 999 --threads 48 --batch-size 4096 --ubatch-size 4096 \
    --override-tensor "blk\..*_exps\.=CPU" --fused-moe \
    --ctx-size 65536 --flash-attn --cache-type-k q8_0 --cache-type-v q8_0 --port 10000 --host 0.0.0.0 \
    --model /mnt/srv/slush/gguf/GLM-4.5-GGUF/GLM-4.5-Q6_K.gguf
```

For prompts below 32k tokens, the output was coherent. But somewhere above 32k it breaks and outputs either gibberish or infinitely repeating Chinese characters. This was a result at `n_past=41006`:

> bandwidth,stylated, other, other, been, other, lower, not the verbal, other, other's available, Origins, other, tons, fmt, other,牢记, whatever, other, missing, other
,ughty, other, temp, reader, the, snaba, other, table, label

I've seen the same occur on the mainline llama.cpp PR. I thought it might have been a rope issue of some sort and tried adding
```
--rope-scaling yarn --rope-scale 4 --yarn-orig-ctx 32768
```
And ran into the same issue with a new swipe of the same prompt, also at `n_past=41006`:
>") a else's wait's同时在 at attire contraseña.
>
> is in question satisfying between атмосфер while waitpairs twins黎 still, enjoying the voices, so。 is connected. understanding, and celebrity, absolutely dot, and celebrity Junction moment.
>
>, so nobody's之余 and, you absolutely, but refampb and celeb

So I think that things are fine on short to medium length prompts, but above 32k it's unusable from my testing. Somehow @ubergarm had an API up briefly yesterday that worked on a 40k length prompt and I haven't figured out how yet.

---

👤 **espen96** commented on **2025-08-04** at **09:16:50**

I have been building up a turn based chat, it has been doing well. I am at 24k now in a 32k context.
switching to a 64k context, to see if I get the same issues. I am using a slightly custom quant now at 4.696 BPW

---

👤 **Thireus** commented on **2025-08-04** at **10:19:29**

That's a fair point @AesSedai, we have not yet tested large context sizes. Next on my list, but let us know if you spot something in the code - https://github.com/ikawrakow/ik_llama.cpp/pull/668/files#diff-57725fa4c99fac28d0283eaa8b59c5e030355af0c8fbaf33145a64e3f2aa3420R16273-R16279.

For the adventurous, I have generated a first quant mix recipe at 3.6910bpw (PPL: 3.2785 +/- 0.01835) for GLM-4.5: https://github.com/Thireus/GGUF-Tool-Suite/blob/main/recipe_examples/GLM-4.5.ROOT-3.6910bpw-3.2785ppl.153GB-GGUF_19GB-GPU_134GB-CPU.68f915c_9c7682b.recipe

---

👤 **Thireus** started a conversation on `src/llama.cpp` on **2025-08-04** at **10:51:04**

Is RoPE working properly? cc: @espen96, @AesSedai

---

👤 **espen96** commented on **2025-08-04** at **11:48:26**

had a few instances of thinking issues. I wonder if this is somehting the model is sensitive to rather than an implementation issue? might be a few layers that need extra care during quantization? 

I'll have to stick to Air for testing here.

I am beyond the 41k "barrier" been switching topics on it like crazy. while it is trying to make some sense of it all and making funny suggestions or comments, I am not seeing any signs of collapse. currently past 45k with only the expected issues models have beyond 32-64k.

I had it recall a story it wrote early on, and it did so at 15-16k without major issues, and again at 45k with no major issues.

at 16k I had it write a followup. I had this recalled at 46-48k with some errors but nothing major.
this second story was also redone as I said it was a bit chaotic and asked for a redo. its retelling matched what it claimed, it recalled the correct "fixed" version and not the flawed one.


No signs of token level issues or collapse yet. I am down to 4 tps so the final stretch to 64k might take a second

---

👤 **espen96** commented on **2025-08-04** at **14:32:00**

ok. I have taken my quant of GLM 4.5 air up to  the 64k threshold. No chinese characters, no catastrophic collapse.
seems something broke in the attempt  to shift the context though?

```
INFO [            update_slots] slot context shift | tid="140305446477824" timestamp=1754317336 id_slot=0 id_task=7 n_keep=0 n_left=65535 n_discard=32767 n_ctx=65536 n_past=65535 n_system_tokens=0 n_cache_tokens=65535
/home/lucis/AI/fp8-to-bf16/ik_llama.cpp/ggml/src/ggml-cuda/rope.cu:370: GGML_ASSERT(src0->type == GGML_TYPE_F32 || src0->type == GGML_TYPE_F16) failed
```

Not sure if that error is related to this model or what. 

In any case.
 I mentioned earlier that I had a few instances of the model messing up its reasoning tags. 
 I have not tested the toggle yet.
 it only happened a few times now, and overall the model appears to be coherent, keeping track of the conversation well. 
 It got most of the important facts it recalled right. I can't tell if any factual errors are from quantization, the model quality / training data, context degrading the quality etc. or something on our end. 
 
 It's stable enough from this test that I would assume, without having checked, that it's no dumber than the implementation they are hosting themselves. 

this is the recipie I used, a variant of ubergarms quant with only a few adjustments, based on the bf16 gguf guide Thireus gave.


```
custom="
# 47 Repeating Layers [0-46]

# Attention
#blk\.(0)\.attn_q.*=q8_0
#blk\.(0)\.attn_k.*=q8_0
#blk\.(0)\.attn_v.*=q8_0
#blk\.(0)\.attn_output.*=q8_0

blk\..*\.attn_q.*=iq5_ks
blk\..*\.attn_k.*=iq5_ks
blk\..*\.attn_v.*=iq5_ks
blk\..*\.attn_output.*=iq5_ks

# First 1 Dense Layers [0]
blk\..*\.ffn_down\.weight=q6_0
blk\..*\.ffn_(gate|up)\.weight=iq5_ks

# Shared Expert Layers [2-46]
blk\..*\.ffn_down_shexp\.weight=q6_0
blk\..*\.ffn_(gate|up)_shexp\.weight=iq5_ks

# Routed Experts Layers [2-46]
#blk\.(3|92)\.ffn_down_exps\.weight=q8_0
#blk\.(3|92)\.ffn_(gate|up)_exps\.weight=q8_0

blk\..*\.ffn_down_exps\.weight=iq4_nl
blk\..*\.ffn_(gate|up)_exps\.weight=iq4_k

# Non-Repeating Layers
token_embd\.weight=iq4_k
output\.weight=iq6_k
"
```

I'll check the "no think" command in a minute.

---

👤 **espen96** commented on **2025-08-04** at **15:12:36**

<img width="1085" height="243" alt="image" src="https://github.com/user-attachments/assets/a1198089-cce8-4a58-9624-380afc74085f" />
yea, seems to be a bit borked that command

---

👤 **Thireus** commented on **2025-08-04** at **15:25:49**

> <img width="1085" height="243" alt="image" src="https://github.com/user-attachments/assets/a1198089-cce8-4a58-9624-380afc74085f" />
> 
> yea, seems to be a bit borked that command

It's /no_think as far as I know that should be used.

---

👤 **espen96** commented on **2025-08-04** at **15:40:52**

/no_think is not doing anything. The model even comments on it, wondering what it might  mean. Meanwhile the /nothink command is recognized/invisible to the model but doesn't seem to be working reliably or at all. 

I cant check the code right now. 
If it's meant to be no_think but the command is registered as nothink or vice versa then that would make sense to me. Not an expert here

---

👤 **Thireus** commented on **2025-08-04** at **16:01:24**

That's weird, it's working for me on GLM-4.5

---

👤 **espen96** commented on **2025-08-04** at **17:16:44**

ok, I checked the template and the tokenizer for both models, it is supposed to be /nothink

looking at the mainline pr, it seems they pushed some changes to fix some issues with the chat template a few hours ago that seems related.

I dont think we have a way here to load an external jinja template? so I can't confirm right now if their change will fix this without redoing my quant

---

👤 **Thireus** commented on **2025-08-04** at **17:53:12**

> ok, I checked the template and the tokenizer for both models, it is supposed to be /nothink
> 
> looking at the mainline pr, it seems they pushed some changes to fix some issues with the chat template a few hours ago that seems related.

I've seen, working on it.

---

👤 **AesSedai** commented on **2025-08-04** at **18:15:26**

I had the same context issue on mainline and ik_llama, maybe I have an old version of the HF safetensors or something and that is causing my issue. Either way, it seems like people in mainline can't reproduce the same context issues that I'm having, so it seems specific to my setup. But thank you for testing!

---

👤 **Thireus** commented on **2025-08-04** at **18:55:04**

> > <img alt="image" width="1085" height="243" src="https://private-user-images.githubusercontent.com/722361/474100390-a1198089-cce8-4a58-9624-380afc74085f.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NTQzMzAyNjgsIm5iZiI6MTc1NDMyOTk2OCwicGF0aCI6Ii83MjIzNjEvNDc0MTAwMzkwLWExMTk4MDg5LWNjZTgtNGE1OC05NjI0LTM4MGFmYzc0MDg1Zi5wbmc_WC1BbXotQWxnb3JpdGhtPUFXUzQtSE1BQy1TSEEyNTYmWC1BbXotQ3JlZGVudGlhbD1BS0lBVkNPRFlMU0E1M1BRSzRaQSUyRjIwMjUwODA0JTJGdXMtZWFzdC0xJTJGczMlMkZhd3M0X3JlcXVlc3QmWC1BbXotRGF0ZT0yMDI1MDgwNFQxNzUyNDhaJlgtQW16LUV4cGlyZXM9MzAwJlgtQW16LVNpZ25hdHVyZT0xNjVhYmM1MjM5YjRhMTA0ZmNmZTRkZDdmZDkwYjkzNTU2MmZkNDQ1ZjQ3NjJmZDE0YjYyZjY2Nzk3ODY0ZmQ1JlgtQW16LVNpZ25lZEhlYWRlcnM9aG9zdCJ9.o_sgZ9g0ht4D-KeZlDhIxo2bHlDNV4_Y3bl6h64E9BM">
> > yea, seems to be a bit borked that command
> 
> It's /no_think as far as I know that should be used.

You're right, `/no_think` does not work, it's instead `/nothink` now. And it is working for me.

I'm still working on migrating and testing the changes. I can see they've merged the code to main.

---

👤 **Thireus** commented on **2025-08-04** at **19:31:53**

Please be warned that after porting the changes, previous GGUFs won't work because several tensors have been renamed. BF16 will also need to be re-done.

---

👤 **Thireus** commented on **2025-08-04** at **20:59:11**

> I had the same context issue on mainline and ik_llama, maybe I have an old version of the HF safetensors or something and that is causing my issue. Either way, it seems like people in mainline can't reproduce the same context issues that I'm having, so it seems specific to my setup. But thank you for testing!

I've spotted in the code that they made changes to the ROPE type used by GLM4_MOE, it was LLAMA_ROPE_TYPE_NORM, now it is LLAMA_ROPE_TYPE_NEOX. Which may have been your issue.

---

👤 **ubergarm** commented on **2025-08-04** at **21:47:35**

@Thireus 

Thanks for bringing this into line for compatibility with the merged mainline PR. I have all the safetensors downloaded and ready to test. Let me know if you hit any snags or decide to skip all the python and focus on the cpp. I know there were some recent changes adding TENSOR_SKIP flag for those NextN MTP tensors.

I'll operate in this order after you give the go-ahead, beginning with the smaller `Air` version then moving on to larger version:

- [x] convert_hf_to_gguf.py from this repo to get fresh bf16 GGUFs
- [x] make a pure q8_0 without imatrix and test compatibility with mainline llama.cpp
- [ ] calculate imatrix from the bf16 GGUF
- [ ] delete any old experimental imatrix/gguf from previous versions off of HF
- [ ] quantize, test, and release to HF

---

👤 **Thireus** commented on **2025-08-04** at **21:51:58**

@ubergarm, I believe I'm done porting the changes, I'm currently converting to BF16 and will then produce a q6_K for testing purpose. ETA ~ 1h. But for faster testing I'll push the changes to another branch on my fork if you don't mind (I'd like to avoid polluting this pull request with broken implementations).

---

👤 **Thireus** commented on **2025-08-04** at **22:02:34**

@ubergarm and others: https://github.com/Thireus/ik_llama.cpp/tree/glm-4.5-testing

Total leap of faith - Just finished porting the code a few minutes ago and to my surprise it compiles without errors. BF16 appears to be converted properly. But I have not ran llama-quantize nor tested quantized models on it. If you don't mind helping me test please.

---

My ongoing test using GLM-4.5 + https://github.com/Thireus/ik_llama.cpp/tree/glm-4.5-testing

- [x] HF to BF16 conversion produces same GGUF as llama.cpp (`convert_hf_to_gguf.py` script works as intended)
- [x] ik_llama.cpp compiles on Linux
- [x] ik_llama.cpp compiles on Windows - see https://github.com/Thireus/ik_llama.cpp/releases/tag/glm-4.5-b4030-d10c3c6
- [x] Quantizing the BF16 version works and no tensors get omitted (`llama-quantize` works as intended) - see https://huggingface.co/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT
- [ ] Loading the model in llama-server works
- [ ] Answers are coherent
- [ ] `/nothink` works
- [ ] Good PPL on high quants

---

👤 **ubergarm** commented on **2025-08-04** at **23:40:02**

So far converting both Air and full GLM-4.5 with Thireus/ik_llama.cpp glm-4.5-testing@a90aec1f going good so far.

Cool the Q8_0 made with your branch is running on mainline lcpp so off to a good start. I'll holler after getting imatrix uploaded with bf16 if anyone else wants to use it.
```bash
print_info: EOG token        = 151329 '<|endoftext|>'
print_info: EOG token        = 151336 '<|user|>'
print_info: EOG token        = 151338 '<|observation|>'
print_info: max token length = 1024
load_tensors: loading model tensors, this can take a while... (mmap = false)
model has unused tensor blk.46.attn_norm.weight (size = 16384 bytes) -- ignoring
model has unused tensor blk.46.attn_q.weight (size = 53477376 bytes) -- ignoring
model has unused tensor blk.46.attn_k.weight (size = 4456448 bytes) -- ignoring
model has unused tensor blk.46.attn_v.weight (size = 4456448 bytes) -- ignoring
model has unused tensor blk.46.attn_q.bias (size = 49152 bytes) -- ignoring
```

Also in a quick test adding `/nothink` at the end of the prompt *did* suppress thinking (the thinking tags are there, but empty).

imatrix for Air is looking good with all exps excercised with my usual corpus text:

```
system_info: n_threads = 128 (n_threads_batch = 192) / 768 | AVX = 1 | AVX_VNNI = 1 | AVX2 = 1 | AVX512 = 1 | AVX512_VBMI = 1 | AVX512_VNNI = 1 | AVX512_BF16 = 1 | FMA = 1 | NEON = 0 | SVE = 0 | ARM_FMA = 0 | F16C = 1 | FP16_VA = 0 | WASM_SIMD = 0 | BLAS = 0 | SSE3 = 1 | SSSE3 = 1 | VSX = 0 | MATMUL_INT8 = 0 | LLAMAFILE = 1 |
compute_imatrix: tokenizing the input ..
compute_imatrix: tokenization took 904.416 ms
compute_imatrix: computing over 814 chunks with batch_size 512
compute_imatrix: 5.63 seconds per pass - ETA 1 hours 16.38 minutes
[1]17.8300,[2]7.5284,[3]4.8735,[4]3.4901,[5]2.7791,[6]2.3527,[7]2.0990,[8]1.9333,[9]1.9293,
save_imatrix: stored collected data after 10 chunks in /mnt/raid/models/ubergarm/GLM-4.5-Air-GGUF/imatrix-GLM-4.5-Air-BF16.dat
```

---

👤 **Nexesenex** commented on **2025-08-05** at **00:01:10**

@Thireus : Thank you!
I'm converting bf16 then will requant my custom IQ4_KS. Results tomorrow morning.

---

👤 **Thireus** commented on **2025-08-05** at **00:06:28**

Amazing. I'll merge -testing into -clean in a bit so we can have the changes in this PR.

@ubergarm - yes please, I would like the new imatrix for BF16 of both GLM-4.5 and GLM-4.5-Air 🙏

---

👤 **AesSedai** commented on **2025-08-05** at **01:14:01**

> > I had the same context issue on mainline and ik_llama, maybe I have an old version of the HF safetensors or something and that is causing my issue. Either way, it seems like people in mainline can't reproduce the same context issues that I'm having, so it seems specific to my setup. But thank you for testing!
> 
> I've spotted in the code that they made changes to the ROPE type used by GLM4_MOE, it was LLAMA_ROPE_TYPE_NORM, now it is LLAMA_ROPE_TYPE_NEOX. Which may have been your issue.

I set it back to LLMAA_ROPE_TYPE_NORM, recompiled, and still have the same issue with an Air Q8_0 quant cooked up by @ddh0.
```
diff --git a/src/llama-model.cpp b/src/llama-model.cpp
index 44f89003b..ea0d7f70f 100644
--- a/src/llama-model.cpp
+++ b/src/llama-model.cpp
@@ -18462,6 +18462,7 @@ llama_rope_type llama_model_rope_type(const llama_model * model) {
         case LLM_ARCH_ARCEE:
         case LLM_ARCH_ERNIE4_5:
         case LLM_ARCH_ERNIE4_5_MOE:
+        case LLM_ARCH_GLM4_MOE:
             return LLAMA_ROPE_TYPE_NORM;
 
         // the pairs of head values are offset by n_rot/2
@@ -18505,7 +18506,6 @@ llama_rope_type llama_model_rope_type(const llama_model * model) {
         case LLM_ARCH_HUNYUAN_DENSE:
         case LLM_ARCH_LFM2:
         case LLM_ARCH_SMALLTHINKER:
-        case LLM_ARCH_GLM4_MOE:
             return LLAMA_ROPE_TYPE_NEOX;
 
         case LLM_ARCH_QWEN2VL:
```

Later tonight or tomorrow I'll try to get a different ~40k token prompt to test. This is driving me up a wall.

---

👤 **ddh0** commented on **2025-08-05** at **01:28:36**

(For the record it's working for me out to 36k tokens and counting.)

---

👤 **ddh0** commented on **2025-08-05** at **01:30:39**

Oh nevermind sorry I thought this was the mainline PR - I'm not using ik_llama.cpp at all for this model.

---

👤 **createthis** commented on **2025-08-05** at **01:34:51**

@AesSedai (love that handle btw) when was the last time you ran a memcheck, gpu memcheck, and made sure your deps all match up?

I think more than one of us has had a mem stick go bad randomly.

---

👤 **ubergarm** commented on **2025-08-05** at **01:42:13**

So just updated imatrix and IQ4_KSS here: https://huggingface.co/ubergarm/GLM-4.5-Air-GGUF

The good news is it seems to be working okay in limited testing, however it is not honoring `/nothink` added to the end of prompt. Here are the llama-server logs for both mainline and this incoming PR (glm-4.5-testing branch) and I'm guessing it is how we handle `/chat/completions` template here perhaps. Both running the same Air-Q8_0 converted and quantized here.

mainline:
[llama-server-mainline-GLM-4.5-Air.log](https://github.com/user-attachments/files/21587964/llama-server-mainline-GLM-4.5-Air.log)

ik_llama.cpp:
[llama-server-ik-GLM-4.5-Air.log](https://github.com/user-attachments/files/21588046/llama-server-ik-GLM-4.5-Air.log)

Guessing we are not processing the jinja from the chat template here for `/chat/completions` endpoint and relying on the simple `--chat-template glm4` code. I have not tried the text endpoint with my own chat template on the client.

Going to calculate the imatrix for the full size GLM-4.5 overnight. Hopefully tomorrow will have some time to upload that as well as possibly some more quants if things are looking good on the PR.

---

👤 **AesSedai** commented on **2025-08-05** at **01:54:10**

@createthis thanks :)

I had a friend run a 35k prompt against my API and he also got gibberish, so my prompt shouldn't be faulty. I don't think it's an issue with the llama.cpp or ik_llama.cpp code anymore, it's likely something with my setup. Thanks all for the double-checking, I'll take this debugging offline and stop clogging up the comments with it while I try and chase this thing down.

---

👤 **ddh0** commented on **2025-08-05** at **02:42:03**

> The good news is it seems to be working okay in limited testing, however it is not honoring /nothink added to the end of prompt.

It also has this problem sometimes on mainline, I think it's just inherent to the model. Sometimes it works sometimes it doesn't.

---

👤 **AesSedai** commented on **2025-08-05** at **04:35:54**

@Thireus @createthis narrowed down the gibberish issue a bit. It requires setting `--batch-size 4096 --ubatch-size 4096` and possibly having a long multi-turn chat going. When I removed the batch-size / ubatch-size, my 40k and 50k token chats began working again. Setting the sizes up to 2048 / 2048 also worked. Something about 4096 / 4096 combined with over 32k context across multiple turns leads to that gibberish edge case.

I also tried a needle in a haystack test with a 35k token prompt with a direction to answer a question from the text as a one-shot and that worked. So I don't have a reproducible smoking gun, but batch-size / ubatch-size is involved.

---

👤 **KonstantinGeist** commented on **2025-08-05** at **05:32:15**

The updated quants by ubergram still fail for me with `GGML_ASSERT(fms.S[j] > 0) failed` with the recent commits (CPU only) :(

---

👤 **espen96** commented on **2025-08-05** at **06:04:34**

> @Thireus @createthis narrowed down the gibberish issue a bit. It requires setting `--batch-size 4096 --ubatch-size 4096` and possibly having a long multi-turn chat going. When I removed the batch-size / ubatch-size, my 40k and 50k token chats began working again. Setting the sizes up to 2048 / 2048 also worked. Something about 4096 / 4096 combined with over 32k context across multiple turns leads to that gibberish edge case.
> 
> I also tried a needle in a haystack test with a 35k token prompt with a direction to answer a question from the text as a one-shot and that worked. So I don't have a reproducible smoking gun, but batch-size / ubatch-size is involved.

Ah, I was doing half of that at most in my runs. 

Batch/ubatch is supposed to speed up prompt processing right? Not generation? 

Anyways, I can test this on my rig when the quant is finished downloading

---

👤 **ikawrakow** commented on **2025-08-05** at **06:06:01**

> The updated quants by ubergram still fail for me with `GGML_ASSERT(fms.S[j] > 0) failed` with the recent commits (CPU only) :(

Does it work without flash attention? What is the CPU?

---

👤 **espen96** commented on **2025-08-05** at **07:05:20**

I can confirm that the /nothink command is still not working. Same as before, it appears to work at times, then reverts, or results in weirdness like the model thinking openly without the tags

---

👤 **Thireus** commented on **2025-08-05** at **07:23:51**

> I can confirm that the /nothink command is still not working. Same as before, it appears to work at times, then reverts, or results in weirdness like the model thinking openly without the tags

Does it also happen on llama.cpp?

---

👤 **ddh0** commented on **2025-08-05** at **07:37:10**

> > I can confirm that the /nothink command is still not working. Same as before, it appears to work at times, then reverts, or results in weirdness like the model thinking openly without the tags
> 
> Does it also happen on llama.cpp?

Yes, it does

---

👤 **espen96** commented on **2025-08-05** at **07:51:57**

downloading an unsloth quant so I have something to use with mainline as well.

however. I decided to test it on z.ai, the officially hosted variant.

<img width="1087" height="855" alt="image" src="https://github.com/user-attachments/assets/cfb1f0bf-f47d-408f-8d07-2d97291c9f26" />
<img width="1087" height="423" alt="image" src="https://github.com/user-attachments/assets/816a73de-b04e-45e7-a8fc-b4334b6c1dfe" />
<img width="1087" height="500" alt="image" src="https://github.com/user-attachments/assets/f2f59840-15d5-48b7-8ebf-55c2fa7dc515" />

it does seem as if the official implementation could be a bit odd? I am not sure if they are intending for the model to reason a bit once in a while even if /nothink was engaged? or if it should be set every turn? Perhaps it really wants to reason and is struggling to obey "no thinking"? I am not quite sure how this works for these fancy hybrid models. However, it is something to be aware of.

the larger model does the same in their chat

---

👤 **tacos8me** commented on **2025-08-05** at **09:43:27**

I'm also seeing gibberish beyond 40-50k context, even with -b and -ub as low as 2048. But only with the air model.

Big model is not doing this. Both Q8 quants I've created with latest ik_llama

Just tried same air quant in llama.cpp, and haven't hit the same issue yet fwiw.

---

👤 **Nexesenex** commented on **2025-08-05** at **10:37:15**

So, I reconverted with the final port the weights of GLM 4.5 Air into bf16 and Ubergarm's new iMatrix.
Then, I quantized :
And I got this :

``
[ 781/ 803] blk.46.nextn.shared_head_head.weight - [ 4096, 151552,     1,     1], type =   bf16,
====== llama_model_quantize_internal: did not find weights for blk.46.nextn.shared_head_head.weight
converts to iq4_ks .. size =  1184.00 MiB ->   315.08 MiB
[ 782/ 803]                   output_norm.weight - [ 4096,     1,     1,     1], type =    f32, size =    0.016 MB
[ 783/ 803]          blk.46.nextn.eh_proj.weight - [ 8192,  4096,     1,     1], type =   bf16,
====== llama_model_quantize_internal: did not find weights for blk.46.nextn.eh_proj.weight
converts to iq4_ks .. size =    64.00 MiB ->    17.02 MiB
[ 784/ 803]            blk.46.nextn.enorm.weight - [ 4096,     1,     1,     1], type =    f32, size =    0.016 MB
[ 785/ 803]            blk.46.nextn.hnorm.weight - [ 4096,     1,     1,     1], type =    f32, size =    0.016 MB
[ 786/ 803]              blk.46.attn_norm.weight - [ 4096,     1,     1,     1], type =    f32, size =    0.016 MB
[ 787/ 803]          blk.46.ffn_down_exps.weight - [ 1408,  4096,   128,     1], type =   bf16,

change_type_if_necessary : tensor cols 1408 x 4096 are not divisible by 256, required for iq4_ks - using fallback quantization iq4_nl

====== llama_model_quantize_internal: did not find weights for blk.46.ffn_down_exps.weight
converts to iq4_nl .. size =  1408.00 MiB ->   396.00 MiB
[ 788/ 803]          blk.46.ffn_gate_exps.weight - [ 4096,  1408,   128,     1], type =   bf16,
====== llama_model_quantize_internal: did not find weights for blk.46.ffn_gate_exps.weight
converts to iq4_ks .. size =  1408.00 MiB ->   374.69 MiB
[ 789/ 803]            blk.46.ffn_up_exps.weight - [ 4096,  1408,   128,     1], type =   bf16,
====== llama_model_quantize_internal: did not find weights for blk.46.ffn_up_exps.weight
converts to iq4_ks .. size =  1408.00 MiB ->   374.69 MiB
[ 790/ 803]              blk.46.exp_probs_b.bias - [  128,     1,     1,     1], type =    f32, size =    0.000 MB
[ 791/ 803]           blk.46.ffn_gate_inp.weight - [ 4096,   128,     1,     1], type =    f32, size =    2.000 MB
[ 792/ 803]         blk.46.ffn_down_shexp.weight - [ 1408,  4096,     1,     1], type =   bf16,
====== llama_model_quantize_internal: did not find weights for blk.46.ffn_down_shexp.weight
converts to q6_0 .. size =    11.00 MiB ->     4.47 MiB
[ 793/ 803]         blk.46.ffn_gate_shexp.weight - [ 4096,  1408,     1,     1], type =   bf16,
====== llama_model_quantize_internal: did not find weights for blk.46.ffn_gate_shexp.weight
converts to iq6_k .. size =    11.00 MiB ->     4.55 MiB
[ 794/ 803]           blk.46.ffn_up_shexp.weight - [ 4096,  1408,     1,     1], type =   bf16,
====== llama_model_quantize_internal: did not find weights for blk.46.ffn_up_shexp.weight
converts to iq6_k .. size =    11.00 MiB ->     4.55 MiB
[ 795/ 803]    blk.46.post_attention_norm.weight - [ 4096,     1,     1,     1], type =    f32, size =    0.016 MB
[ 796/ 803]                   blk.46.attn_k.bias - [ 1024,     1,     1,     1], type =    f32, size =    0.004 MB
[ 797/ 803]                 blk.46.attn_k.weight - [ 4096,  1024,     1,     1], type =   bf16,
====== llama_model_quantize_internal: did not find weights for blk.46.attn_k.weight
converts to iq6_k .. size =     8.00 MiB ->     3.31 MiB
[ 798/ 803]            blk.46.attn_output.weight - [12288,  4096,     1,     1], type =   bf16,
====== llama_model_quantize_internal: did not find weights for blk.46.attn_output.weight
converts to iq5_k .. size =    96.00 MiB ->    33.00 MiB
[ 799/ 803]                   blk.46.attn_q.bias - [12288,     1,     1,     1], type =    f32, size =    0.047 MB
[ 800/ 803]                 blk.46.attn_q.weight - [ 4096, 12288,     1,     1], type =   bf16,
====== llama_model_quantize_internal: did not find weights for blk.46.attn_q.weight
converts to iq4_k .. size =    96.00 MiB ->    27.00 MiB
[ 801/ 803]                   blk.46.attn_v.bias - [ 1024,     1,     1,     1], type =    f32, size =    0.004 MB
[ 802/ 803]                 blk.46.attn_v.weight - [ 4096,  1024,     1,     1], type =   bf16,
====== llama_model_quantize_internal: did not find weights for blk.46.attn_v.weight
converts to iq6_k .. size =     8.00 MiB ->     3.31 MiB
[ 803/ 803] blk.46.nextn.shared_head_norm.weight - [ 4096,     1,     1,     1], type =    f32, size =    0.016 MB
``

Before, I was getting that :

``
[ 781/ 803]              blk.46.shared_head.head - [ 4096, 151552,     1,     1], type =   bf16, size = 1184.000 MB
[ 782/ 803]                   output_norm.weight - [ 4096,     1,     1,     1], type =    f32, size =    0.016 MB
[ 783/ 803]                       blk.46.eh_proj - [ 8192,  4096,     1,     1], type =   bf16, size =   64.000 MB
[ 784/ 803]                         blk.46.enorm - [ 4096,     1,     1,     1], type =    f32, size =    0.016 MB
[ 785/ 803]                         blk.46.hnorm - [ 4096,     1,     1,     1], type =    f32, size =    0.016 MB
[ 786/ 803]              blk.46.attn_norm.weight - [ 4096,     1,     1,     1], type =    f32, size =    0.016 MB
[ 787/ 803]          blk.46.ffn_down_exps.weight - [ 1408,  4096,   128,     1], type =   bf16,

change_type_if_necessary : tensor cols 1408 x 4096 are not divisible by 256, required for iq5_k - using fallback quantization q6_0
converts to q6_0 .. size =  1408.00 MiB ->   572.00 MiB
[ 788/ 803]          blk.46.ffn_gate_exps.weight - [ 4096,  1408,   128,     1], type =   bf16, converts to iq4_k .. size =  1408.00 MiB ->   396.00 MiB
[ 789/ 803]            blk.46.ffn_up_exps.weight - [ 4096,  1408,   128,     1], type =   bf16, converts to iq4_k .. size =  1408.00 MiB ->   396.00 MiB
[ 790/ 803]             blk.46.ffn_gate_inp.bias - [  128,     1,     1,     1], type =    f32, size =    0.000 MB
[ 791/ 803]           blk.46.ffn_gate_inp.weight - [ 4096,   128,     1,     1], type =    f32, size =    2.000 MB
[ 792/ 803]         blk.46.ffn_down_shexp.weight - [ 1408,  4096,     1,     1], type =   bf16, converts to q6_0 .. size =    11.00 MiB ->     4.47 MiB
[ 793/ 803]         blk.46.ffn_gate_shexp.weight - [ 4096,  1408,     1,     1], type =   bf16, converts to q6_0 .. size =    11.00 MiB ->     4.47 MiB
[ 794/ 803]           blk.46.ffn_up_shexp.weight - [ 4096,  1408,     1,     1], type =   bf16, converts to q6_0 .. size =    11.00 MiB ->     4.47 MiB
[ 795/ 803]               blk.46.ffn_norm.weight - [ 4096,     1,     1,     1], type =    f32, size =    0.016 MB
[ 796/ 803]                   blk.46.attn_k.bias - [ 1024,     1,     1,     1], type =    f32, size =    0.004 MB
[ 797/ 803]                 blk.46.attn_k.weight - [ 4096,  1024,     1,     1], type =   bf16, converts to iq6_k .. size =     8.00 MiB ->     3.31 MiB
[ 798/ 803]            blk.46.attn_output.weight - [12288,  4096,     1,     1], type =   bf16, converts to iq5_k .. size =    96.00 MiB ->    33.00 MiB
[ 799/ 803]                   blk.46.attn_q.bias - [12288,     1,     1,     1], type =    f32, size =    0.047 MB
[ 800/ 803]                 blk.46.attn_q.weight - [ 4096, 12288,     1,     1], type =   bf16, converts to iq4_k .. size =    96.00 MiB ->    27.00 MiB
[ 801/ 803]                   blk.46.attn_v.bias - [ 1024,     1,     1,     1], type =    f32, size =    0.004 MB
[ 802/ 803]                 blk.46.attn_v.weight - [ 4096,  1024,     1,     1], type =   bf16, converts to iq6_k .. size =     8.00 MiB ->     3.31 MiB
[ 803/ 803]              blk.46.shared_head.norm - [ 4096,     1,     1,     1], type =    f32, size =    0.016 MB
``

Then, I run PPL 512.
Before the final port, I was using this command :

``
llama-perplexity -m X:\text-generation-webui\models\zai-org_GLM-4.5-Air-bf16-iMat-IQ4_KS_plus9-00001-of-00005.gguf -f wiki.test.raw --parallel 1 -ngl 150 -b 512 -mg 0 -ts 18,18,12 --no-mmap -fa -c 512 -ot "blk.46.embed_tokens=CPU" -ot "blk.46.shared_head.head=CPU" -ot "blk.46.eh_proj=CPU" -ot "blk.46.enorm=CPU" -ot "blk.46.hnorm=CPU" -ot "blk.46.shared_head.norm=CPU" --override-kv glm4moe.expert_used_count=int:7
``

With a quant made like this.

``
Adding custom rule blk.46.shared_head.head -> q6_0
Adding custom rule blk\.([0-1]|4[6])\.attn_q.* -> iq4_k
Adding custom rule blk\.([2-9]|1[0-7]|3[6-9]|4[0-5])\.attn_q.* -> iq4_ks
Adding custom rule blk\..*\.attn_q.* -> iq4_kss
Adding custom rule blk\.([0-1]|4[6])\.attn_output.* -> iq5_k
Adding custom rule blk\.([2-9]|1[0-7]|3[6-9]|4[0-5])\.attn_output.* -> iq5_k
Adding custom rule blk\..*\.attn_output.* -> iq5_ks
Adding custom rule blk\.([0-1]|4[6])\.attn_k.* -> iq6_k
Adding custom rule blk\..*\.attn_k.* -> iq5_k
Adding custom rule blk\..*\.attn_v.* -> iq6_k
Adding custom rule blk\.(0)\.ffn_(down)\.weight -> q8_0
Adding custom rule blk\.(0)\.ffn_(gate|up)\.weight -> q6_0
Adding custom rule blk\.(1)\.ffn_down_exps\.weight -> iq5_k
Adding custom rule blk\.(4[6])\.ffn_down_exps\.weight -> iq5_k
Adding custom rule blk\.(1|)\.ffn_(gate|up)_exps\.weight -> iq5_ks
Adding custom rule blk\.(4[6])\.ffn_(gate|up)_exps\.weight -> iq4_k
Adding custom rule blk\.(1[8-9]|2[0-9]|3[0-5])\.ffn_(down|gate|up)_exps\.weight -> iq4_kss
Adding custom rule blk\..*\.ffn_(gate|up)_exps\.weight -> iq4_ks
Adding custom rule blk\..*\.ffn_down_shexp\.weight -> q6_0
Adding custom rule blk\..*\.ffn_(gate|up)_shexp\.weight -> q6_0
Adding custom rule token_embd\.weight -> q6_0
Adding custom rule output.weight -> iq6_k
``

And obtained that:

``
Final estimate: PPL = 4.7051 +/- 0.02872

llama_print_timings:        load time =   31289.83 ms
llama_print_timings:      sample time =       0.00 ms /     1 runs   (    0.00 ms per token,      inf tokens per second)
llama_print_timings: prompt eval time =  699558.94 ms / 289280 tokens (    2.42 ms per token,   413.52 tokens per second)
llama_print_timings:        eval time =       0.00 ms /     1 runs   (    0.00 ms per token,      inf tokens per second)
llama_print_timings:       total time =  709233.04 ms / 289281 tokens
``

With my new quant made with the final port (slightly edited on a few tensors, I used this one:

``
llama-perplexity -m X:\text-generation-webui\models\zai-org_GLM-4.5-Air-bf16-iMat-IQ4_KS_plus9-00001-of-00005.gguf -f wiki.test.raw --parallel 1 -ngl 150 -b 512 -mg 0 -ts 18,18,12 --no-mmap -fa -c 512 -ot "blk.46.embed_tokens=CPU" -ot "blk.46.shared_head.head=CPU" -ot "blk.46.eh_proj=CPU" -ot "blk.46.enorm=CPU" -ot "blk.46.hnorm=CPU" -ot "blk.46.shared_head.norm=CPU" -ot "blk.46.ffn_down_exps.weight=CPU" -ot "blk.46.ffn_gate_exps.weight=CPU" -ot "blk.46.ffn_up_exps.weight=CPU" -ot "blk.46.ffn_down_shexp.weight=CPU" -ot "blk.46.ffn_gate_shexp.weight=CPU" -ot "blk.46.ffn_up_shexp.weight=CPU" -ot "blk.46.attn_q.weight=CPU" -ot "blk.46.attn_k.weight=CPU" -ot "blk.46.attn_v.weight=CPU" -ot "blk.46.attn_output.weight=CPU" --override-kv glm4moe.expert_used_count=int:7
``

The quant is made like this :

``
Adding custom rule blk.46.shared_head.head -> iq6_k
Adding custom rule blk.46.nextn.embed_tokens.weight -> q6_0
Adding custom rule blk\.([0-1]|4[6])\.attn_q.* -> iq4_k
Adding custom rule blk\.([2-9]|1[0-7]|2[4-6]|3[6-9]|4[0-5])\.attn_q.* -> iq4_ks
Adding custom rule blk\..*\.attn_q.* -> iq4_kss
Adding custom rule blk\.([0-1]|4[6])\.attn_output.* -> iq5_k
Adding custom rule blk\.([2-9]|1[0-7]|2[4-6]|3[6-9]|4[0-5])\.attn_output.* -> iq5_k
Adding custom rule blk\..*\.attn_output.* -> iq5_ks
Adding custom rule blk\.([0-1]|4[6])\.attn_k.* -> iq6_k
Adding custom rule blk\..*\.attn_k.* -> iq5_k
Adding custom rule blk\..*\.attn_v.* -> iq6_k
Adding custom rule blk\.(0)\.ffn_(down)\.weight -> q8_0
Adding custom rule blk\.(0)\.ffn_(gate|up)\.weight -> iq6_k
Adding custom rule blk\.(1)\.ffn_down_exps\.weight -> iq5_k
Adding custom rule blk\.(1)\.ffn_(gate|up)_exps\.weight -> iq5_ks
Adding custom rule blk\.(1[8-9]|2[0-3]|2[7-9]||3[0-5])\.ffn_(down|gate|up)_exps\.weight -> iq4_kss
Adding custom rule blk\..*\.ffn_(gate|up)_exps\.weight -> iq4_ks
Adding custom rule blk\..*\.ffn_down_shexp\.weight -> q6_0
Adding custom rule blk\..*\.ffn_(gate|up)_shexp\.weight -> iq6_k
Adding custom rule token_embd\.weight -> q6_0
Adding custom rule output.weight -> iq6_k
``

And I obtained this :

``
Final estimate: PPL = 4.6582 +/- 0.02860

llama_print_timings:        load time =   30266.96 ms
llama_print_timings:      sample time =       0.00 ms /     1 runs   (    0.00 ms per token,      inf tokens per second)
llama_print_timings: prompt eval time =  675949.53 ms / 289280 tokens (    2.34 ms per token,   427.96 tokens per second)
llama_print_timings:        eval time =       0.00 ms /     1 runs   (    0.00 ms per token,      inf tokens per second)
llama_print_timings:       total time =  685632.02 ms / 289281 tokens
``

A few observations of my own (no matter if they are already made/solved or not by others) :
- My recipe is weird because I use 2x24GB and one 16GB card, and I like to fully offload, so I tailored it for that precise need.
- The changes on layer 46 do not impact PPL negatively, and I even gained close to a one 1% drop. (the little diffs in recipes entitle to 0.005 max delta between the 2 quants with the former PR, certainly not 0.05!)
- The PPL got improved by around 1%, if anything.
- The non-offload on GPU for layer 46 doesn't affect badly prompt processing (to be tested on higher context, though).
- Thanks Thireus for the PR, and Ubergarm for the new iMatrix!

Forgive the lack of care in the redaction of this post!

---

👤 **ubergarm** commented on **2025-08-05** at **11:38:35**

@Nexesenex 

> So, I reconverted with the final port the weights of GLM 4.5 Air into bf16 and Ubergarm's new iMatrix.

Thanks, I'm looking more closely at my logs and the changes since our earlier attempt and yeah a few things of note:

### Include NextN MTP Tensors
Both GLM-4.5 and GLM-4.5-Air convert now include the `NextN MTP` tensors on the last layer e.g. for the big one:
```bash
INFO:hf-to-gguf:blk.92.nextn.embed_tokens.weight,     torch.bfloat16 --> BF16, shape = {5120, 151552}
INFO:hf-to-gguf:blk.92.nextn.shared_head_head.weight, torch.bfloat16 --> BF16, shape = {5120, 151552}
INFO:hf-to-gguf:blk.92.nextn.eh_proj.weight,          torch.bfloat16 --> BF16, shape = {10240, 5120}
INFO:hf-to-gguf:blk.92.nextn.enorm.weight,            torch.bfloat16 --> F32, shape = {5120}
INFO:hf-to-gguf:blk.92.nextn.hnorm.weight,            torch.bfloat16 --> F32, shape = {5120}
INFO:hf-to-gguf:blk.92.nextn.shared_head_norm.weight, torch.bfloat16 --> F32, shape = {5120}
```

On mainline the new solution introduced is a new `TENSOR_SKIP` flag to mark those specific tensors to not be loaded into VRAM for now, but still be available for the future if MTP were to be implemented (multi-token-prediction)

```
# mainline llama.cpp@41613437f
# https://github.com/ggml-org/llama.cpp/blob/master/src/llama-model.cpp#L2037-L2046
$ grep -r TENSOR_SKIP
llama-model-loader.h:    static const int TENSOR_SKIP         = 1 << 2;
llama-model.cpp:    const auto TENSOR_SKIP         = llama_model_loader::TENSOR_SKIP;
llama-model.cpp:            if (info.op == GGML_OP_NONE || flags & TENSOR_SKIP) {
llama-model.cpp:                            flags |= TENSOR_SKIP;
```

Oh wait a sec, I don't see this implemented on `Thireus/ik_llama.cpp glm-4.5-testing@a90aec1f`... Maybe @Thireus handled this differently?

Also right my new imatrix totally skips that last layer, and I see a note here on mainline suggesting because that last layer is totally skipped there too, to not quantize those tensors too small given no imatrix data now? https://github.com/ggml-org/llama.cpp/pull/14939#issuecomment-3153670235

I'm a little confused and want to look more closely at the differences around how skipping those NextN tesnors vs skipping the whole last layer is implemented in the graph before releasing more quants. Though the big GLM-4.5 imatrix is uploaded now (missing last layer data now) here: https://huggingface.co/ubergarm/GLM-4.5-GGUF

Also a few baseline perplexity values updated here: https://huggingface.co/ubergarm/GLM-4.5-Air-GGUF#quant-collection

Thanks.

---

👤 **espen96** commented on **2025-08-05** at **12:01:07**

> > I can confirm that the /nothink command is still not working. Same as before, it appears to work at times, then reverts, or results in weirdness like the model thinking openly without the tags
> 
> Does it also happen on llama.cpp?

seems /nothink is working with unloths `GLM-4.5-Air-UD-Q4_K_XL` on llama.cpp, using `--jinja`. multiple messages in a row, no thinking.

the same quant on ik_llama.cpp is not following the command. it continues to think / reason. 
edit: using `--chat-template chatglm4` appears to fix the issue on ik_llama.cpp
edit2: llama.cpp works properly without using `--jinja`

---

👤 **Thireus** commented on **2025-08-05** at **12:15:28**

> @Nexesenex
> 
> > So, I reconverted with the final port the weights of GLM 4.5 Air into bf16 and Ubergarm's new iMatrix.
> 
> Thanks, I'm looking more closely at my logs and the changes since our earlier attempt and yeah a few things of note:
> ### Include NextN MTP Tensors
> 
> Both GLM-4.5 and GLM-4.5-Air convert now include the `NextN MTP` tensors on the last layer e.g. for the big one:
> 
> ```shell
> INFO:hf-to-gguf:blk.92.nextn.embed_tokens.weight,     torch.bfloat16 --> BF16, shape = {5120, 151552}
> INFO:hf-to-gguf:blk.92.nextn.shared_head_head.weight, torch.bfloat16 --> BF16, shape = {5120, 151552}
> INFO:hf-to-gguf:blk.92.nextn.eh_proj.weight,          torch.bfloat16 --> BF16, shape = {10240, 5120}
> INFO:hf-to-gguf:blk.92.nextn.enorm.weight,            torch.bfloat16 --> F32, shape = {5120}
> INFO:hf-to-gguf:blk.92.nextn.hnorm.weight,            torch.bfloat16 --> F32, shape = {5120}
> INFO:hf-to-gguf:blk.92.nextn.shared_head_norm.weight, torch.bfloat16 --> F32, shape = {5120}
> ```
> 
> On mainline the new solution introduced is a new `TENSOR_SKIP` flag to mark those specific tensors to not be loaded into VRAM for now, but still be available for the future if MTP were to be implemented (multi-token-prediction)
> 
> ```
> # mainline llama.cpp@41613437f
> # https://github.com/ggml-org/llama.cpp/blob/master/src/llama-model.cpp#L2037-L2046
> $ grep -r TENSOR_SKIP
> llama-model-loader.h:    static const int TENSOR_SKIP         = 1 << 2;
> llama-model.cpp:    const auto TENSOR_SKIP         = llama_model_loader::TENSOR_SKIP;
> llama-model.cpp:            if (info.op == GGML_OP_NONE || flags & TENSOR_SKIP) {
> llama-model.cpp:                            flags |= TENSOR_SKIP;
> ```
> 
> Oh wait a sec, I don't see this implemented on `Thireus/ik_llama.cpp glm-4.5-testing@a90aec1f`... Maybe @Thireus handled this differently?
> 
> Also right my new imatrix totally skips that last layer, and I see a note here on mainline suggesting because that last layer is totally skipped there too, to not quantize those tensors too small given no imatrix data now? [ggml-org/llama.cpp[#14939](https://github.com/ikawrakow/ik_llama.cpp/issues/14939) (comment)](https://github.com/ggml-org/llama.cpp/pull/14939#issuecomment-3153670235)
> 
> I'm a little confused and want to look more closely at the differences around how skipping those NextN tesnors vs skipping the whole last layer is implemented in the graph before releasing more quants. Though the big GLM-4.5 imatrix is uploaded now (missing last layer data now) here: https://huggingface.co/ubergarm/GLM-4.5-GGUF
> 
> Also a few baseline perplexity values updated here: https://huggingface.co/ubergarm/GLM-4.5-Air-GGUF#quant-collection
> 
> Thanks.

That TENSOR_SKIP addition was not in the code I reviewed yesterday, which was https://github.com/sammcj/llama.cpp/tree/447f9337890d38e4702b1dc6204ff5009b828275. I'll reference which version was ported in the top post of this pull request. I will go ahead and implement the additional changes.

Thanks for the imatrix. I've also noticed that I cannot quantize to iq1_m_r4 due to your imatrix missing them. But as the comment suggests, it shouldn't be an issue and we need to ensure q8_0 or similar high quants are used for them, is that right?

---

👤 **ubergarm** commented on **2025-08-05** at **12:40:37**

@Thireus 

> I will go ahead and implement the additional changes.

Thanks, yeah it is kind of confusing to me. Despite adding `TENSOR_SKIP` flag in mainline, they still seem to skip the *entire* last layer including non `nextn` tensors like attn/exps/shexp hence why imatrix data is now missing. This is probably why while you didn't add `TENSOR_SKIP` here, it seems to be working anyway. (as you have the same logic in the graph `// Only process up to last layer (skip final NextN layer)`)

For example, here are the tensors on the last layer of Air which include both `nextn` tensors and the usual attn/exps/shexp tensors:

```
$ grep blk\.46 quantize-GLM-4.5-Air-Q8_0.log | grep bf16
[  14/ 803]     blk.46.nextn.embed_tokens.weight - [ 4096, 151552,     1,     1], type =   bf16, converting to q8_0 .. size =  1184.00 MiB ->   629.00 MiB
[ 781/ 803] blk.46.nextn.shared_head_head.weight - [ 4096, 151552,     1,     1], type =   bf16, converting to q8_0 .. size =  1184.00 MiB ->   629.00 MiB
[ 783/ 803]          blk.46.nextn.eh_proj.weight - [ 8192,  4096,     1,     1], type =   bf16, converting to q8_0 .. size =    64.00 MiB ->    34.00 MiB
[ 787/ 803]          blk.46.ffn_down_exps.weight - [ 1408,  4096,   128,     1], type =   bf16, converting to q8_0 .. size =  1408.00 MiB ->   748.00 MiB
[ 788/ 803]          blk.46.ffn_gate_exps.weight - [ 4096,  1408,   128,     1], type =   bf16, converting to q8_0 .. size =  1408.00 MiB ->   748.00 MiB
[ 789/ 803]            blk.46.ffn_up_exps.weight - [ 4096,  1408,   128,     1], type =   bf16, converting to q8_0 .. size =  1408.00 MiB ->   748.00 MiB
[ 792/ 803]         blk.46.ffn_down_shexp.weight - [ 1408,  4096,     1,     1], type =   bf16, converting to q8_0 .. size =    11.00 MiB ->     5.84 MiB
[ 793/ 803]         blk.46.ffn_gate_shexp.weight - [ 4096,  1408,     1,     1], type =   bf16, converting to q8_0 .. size =    11.00 MiB ->     5.84 MiB
[ 794/ 803]           blk.46.ffn_up_shexp.weight - [ 4096,  1408,     1,     1], type =   bf16, converting to q8_0 .. size =    11.00 MiB ->     5.84 MiB
[ 797/ 803]                 blk.46.attn_k.weight - [ 4096,  1024,     1,     1], type =   bf16, converting to q8_0 .. size =     8.00 MiB ->     4.25 MiB
[ 798/ 803]            blk.46.attn_output.weight - [12288,  4096,     1,     1], type =   bf16, converting to q8_0 .. size =    96.00 MiB ->    51.00 MiB
[ 800/ 803]                 blk.46.attn_q.weight - [ 4096, 12288,     1,     1], type =   bf16, converting to q8_0 .. size =    96.00 MiB ->    51.00 MiB
[ 802/ 803]                 blk.46.attn_v.weight - [ 4096,  1024,     1,     1], type =   bf16, converting to q8_0 .. size =     8.00 MiB ->     4.25 MiB
```

I haven't yet found a reference in zRzRzRzRzRs transformers PR or some kind of official GLM-4.5 paper with enough detail to explain if that **whole layer** is only for MTP, or just the nextn tensors which are stored in the same layer alongside the "normal" tensors?

> For both GLM-4.5 and GLM-4.5-Air, we add an MTP (Multi-Token Prediction) layer to support speculative decoding during inference. https://z.ai/blog/glm-4.5

Here is the code block for the glm4_moe graph, which is the same in your PR, where it skips that **entire** last layer hence why there is no imatrix data and hence why they are suggesting to use a larger quant for that last layer for the attn/exps/shexp despite it not being used anyway?? https://github.com/ggml-org/llama.cpp/blob/master/src/llama-model.cpp#L13715-L13718

I still have my old imatrix made from a convert with ripped out MTP tensors which *does* have imatrix data for the last attn/exps/shexp layer tensors. But if we're just skipping it anyway seems like just make it something middling just in case MTP is added in the future and they are no longer ignored.

You can see the debug log from mainline where they are considered ignored (with the TENSOR_SKIP flag presumably preventing them from loading into VRAM):

```
print_info: max token length = 1024
load_tensors: loading model tensors, this can take a while... (mmap = false)
model has unused tensor blk.46.attn_norm.weight (size = 16384 bytes) -- ignoring
model has unused tensor blk.46.attn_q.weight (size = 53477376 bytes) -- ignoring
model has unused tensor blk.46.attn_k.weight (size = 4456448 bytes) -- ignoring
model has unused tensor blk.46.attn_v.weight (size = 4456448 bytes) -- ignoring
model has unused tensor blk.46.attn_q.bias (size = 49152 bytes) -- ignoring
model has unused tensor blk.46.attn_k.bias (size = 4096 bytes) -- ignoring
model has unused tensor blk.46.attn_v.bias (size = 4096 bytes) -- ignoring
model has unused tensor blk.46.attn_output.weight (size = 53477376 bytes) -- ignoring
model has unused tensor blk.46.post_attention_norm.weight (size = 16384 bytes) -- ignoring
model has unused tensor blk.46.ffn_gate_inp.weight (size = 2097152 bytes) -- ignoring
model has unused tensor blk.46.exp_probs_b.bias (size = 512 bytes) -- ignoring
model has unused tensor blk.46.ffn_gate_exps.weight (size = 784334848 bytes) -- ignoring
model has unused tensor blk.46.ffn_down_exps.weight (size = 784334848 bytes) -- ignoring
model has unused tensor blk.46.ffn_up_exps.weight (size = 784334848 bytes) -- ignoring
model has unused tensor blk.46.ffn_gate_shexp.weight (size = 6127616 bytes) -- ignoring
model has unused tensor blk.46.ffn_down_shexp.weight (size = 6127616 bytes) -- ignoring
model has unused tensor blk.46.ffn_up_shexp.weight (size = 6127616 bytes) -- ignoring
model has unused tensor blk.46.nextn.eh_proj.weight (size = 35651584 bytes) -- ignoring
model has unused tensor blk.46.nextn.embed_tokens.weight (size = 659554304 bytes) -- ignoring
model has unused tensor blk.46.nextn.enorm.weight (size = 16384 bytes) -- ignoring
model has unused tensor blk.46.nextn.hnorm.weight (size = 16384 bytes) -- ignoring
model has unused tensor blk.46.nextn.shared_head_head.weight (size = 659554304 bytes) -- ignoring
model has unused tensor blk.46.nextn.shared_head_norm.weight (size = 16384 bytes) -- ignoring
load_tensors:          CPU model buffer size = 108340.16 MiB
....................................................................................................
```

In conclusion, while I don't know if it is correct to ignore/skip the **entire** final layer, it seems like that is what the mainline implementation is doing. The addition of the TENSOR_SKIP flag is only to save RAM/VRAM as those tensors won't be loaded, but the actual skipping logic is in the graph I guess.

---

👤 **createthis** commented on **2025-08-05** at **12:56:06**

> I haven't yet found a reference in zRzRzRzRzRs transformers PR or some kind of official GLM-4.5 paper with enough detail to explain if that **whole layer** is only for MTP, or just the nextn tensors which are stored in the same layer alongside the "normal" tensors?

I assume you're talking about this one? https://github.com/huggingface/transformers/pull/39805

Is that how everyone seemed to magically know how to implement the model without there being a tech document explaining the differences? Is everyone just porting their PR?

---

👤 **Thireus** commented on **2025-08-05** at **12:56:55**

@ubergarm - I have ported the changes, please check on https://github.com/Thireus/ik_llama.cpp/tree/glm-4.5-testing

https://github.com/Thireus/ik_llama.cpp/commit/3f3e384fabe25ac916e12aba8c0f7462ec3c6e84

---
Handle TENSOR_SKIP
Ported the changes from:

sammcj/llama.cpp@f129567
sammcj/llama.cpp@dcbbd2c

Except op info since ik_llama.cpp doesn't support this operation.

---

👤 **ubergarm** commented on **2025-08-05** at **13:17:12**

@Thireus  

Thanks I'll take a peek. One more small change just came in which might apply: https://github.com/ggml-org/llama.cpp/pull/15088

Regarding the question "does skipping the **entire** last layer and not just the `nextn` tensors have any noticible effects?". I had taken perplexity measurements of the Air-Q8_0 with and without that last layer involved:

Earlier attempt with ripped out nextn/mtp tensors that included the final attn/exps/shexp layer:
Final estimate: PPL = 4.6361 +/- 0.02821

Current implementation which skips the **entire** last layer:
Final estimate: PPL = 4.5798 +/- 0.02804

So barring any other differences for which I'm not accounting, this would seem to indicate empirically that skipping the **entire** final layer including not just nextn tensors but also the "normal" attn/exps/shexp is likely the "correct" implementation in that it gives a "better" perplexity.

So quantize them however you want as it will only effect the file size now that they are not being loaded due to TENSOR_SKIP and they are not involved at all in actual inferencing. In the future if MTP support is added, only then would it possibly make a difference.

> Except op info since ik_llama.cpp doesn't support this operation.

Hrm, I don't think it is actually ignoring/skipping loading quite yet as with or without the most recent patch the CPU buffer size for the Air-Q8_0 did not decrease like it does on  mainline:

```
# with TENSOR_SKIP on mainline
load_tensors:          CPU model buffer size = 108340.16 MiB

# with and without TENSOR_SKIP here
llm_load_tensors:        CPU buffer size = 112006.33 MiB
```

I'll poke at that a bit more.

@createthis 

> Is everyone just porting their PR?

More or less a new model comes out and generally someone from that organization will submit a PR to transformers or provide a reference implementation. When we're lucky that person/organization also submits PRs to vllm/sglang/mlx/llama.cpp as well. If they don't, then folks try to port it from one of those places to llama.cpp and ik_llama.cpp is how I understand it.

---

👤 **ubergarm** commented on **2025-08-05** at **13:47:18**

@Thireus 

This patch does the needful to actually skip loading if a tensor has the TENSOR_SKIP flag.

```diff
$ git diff -w
diff --git a/src/llama.cpp b/src/llama.cpp
index 7c27d3cf..f99927b4 100644
--- a/src/llama.cpp
+++ b/src/llama.cpp
@@ -4896,6 +4896,17 @@ struct llama_model_loader {
             return NULL;
         }

+        // skip unused tensors
+        if (flags & TENSOR_SKIP) {
+            const size_t nbytes = ggml_nbytes(cur);
+            LLAMA_LOG_WARN("model has unused tensor %s (size = %zu bytes) -- ignoring\n", name.c_str(), nbytes);
+
+            size_data -= nbytes;
+            n_created++;
+
+            return nullptr;
+        }
+
         return create_tensor_for(ctx, cur, flags & TENSOR_DUPLICATED);
     }
```

So now here we have the expected lower buffer size:

```
# skip loading all blk.46 due to TENSOR_SKIP flag
model has unused tensor blk.46.nextn.embed_tokens.weight (size = 659554304 bytes) -- ignoring
model has unused tensor blk.46.nextn.enorm.weight (size = 16384 bytes) -- ignoring
model has unused tensor blk.46.nextn.hnorm.weight (size = 16384 bytes) -- ignoring
model has unused tensor blk.46.nextn.shared_head_head.weight (size = 659554304 bytes) -- ignoring
model has unused tensor blk.46.nextn.shared_head_norm.weight (size = 16384 bytes) -- ignoring
llm_load_tensors:        CPU buffer size = 108340.16 MiB
```

---

👤 **Thireus** commented on **2025-08-05** at **13:47:54**

> @Thireus
> 
> This patch does the needful to actually skip loading if a tensor has the TENSOR_SKIP flag.
> 
> ```diff
> $ git diff -w
> diff --git a/src/llama.cpp b/src/llama.cpp
> index 7c27d3cf..f99927b4 100644
> --- a/src/llama.cpp
> +++ b/src/llama.cpp
> @@ -4896,6 +4896,17 @@ struct llama_model_loader {
>              return NULL;
>          }
> 
> +        // skip unused tensors
> +        if (flags & TENSOR_SKIP) {
> +            const size_t nbytes = ggml_nbytes(cur);
> +            LLAMA_LOG_WARN("model has unused tensor %s (size = %zu bytes) -- ignoring\n", name.c_str(), nbytes);
> +
> +            size_data -= nbytes;
> +            n_created++;
> +
> +            return nullptr;
> +        }
> +
>          return create_tensor_for(ctx, cur, flags & TENSOR_DUPLICATED);
>      }
> ```
> 
> So now here we have the expected lower buffer size:
> 
> ```
> # skip loading all blk.46 due to TENSOR_SKIP flag
> model has unused tensor blk.46.nextn.embed_tokens.weight (size = 659554304 bytes) -- ignoring
> model has unused tensor blk.46.nextn.enorm.weight (size = 16384 bytes) -- ignoring
> model has unused tensor blk.46.nextn.hnorm.weight (size = 16384 bytes) -- ignoring
> model has unused tensor blk.46.nextn.shared_head_head.weight (size = 659554304 bytes) -- ignoring
> model has unused tensor blk.46.nextn.shared_head_norm.weight (size = 16384 bytes) -- ignoring
> llm_load_tensors:        CPU buffer size = 108340.16 MiB
> ```

Thank you, I'll implement this on testing.

---

👤 **Thireus** commented on **2025-08-05** at **13:56:16**

@ubergarm, I've pushed the change on the testing branch, please let me know if that works and if you are happy with it.

Here: https://github.com/Thireus/ik_llama.cpp/commit/a3641e6646d77d3f0b36798ca313b50e41386f72

---
I don't think we are affected by this bug since we actually use the correct code mentioned in ggml-org/llama.cpp[#15088](https://github.com/ikawrakow/ik_llama.cpp/issues/15088) already.

---

👤 **ubergarm** commented on **2025-08-05** at **14:08:56**

>  I've pushed the change on the testing branch, please let me know if that works and if you are happy with it.

Yup, looks like everything is working now, those tensors are not being loaded so don't waste VRAM/RAM and are also skipped at inference time so I believe we line up with mainline lcpp. If anyone can try a mainline llama.cpp quant on Thireus/ik_llama.cpp glm-4.5-clean@46c3fa95 , that would be appreciated.

> I don't think we are affected by this bug

Correct, I went and checked too and you had already removed the offending lines.

Great job I think we're in good shape. The upshot of all this now is:

1. The imatrix has no data for any tensors on the last layer
2. As such quantize them large enough to not fail knowing they are not used for inference and not even loaded into RAM/VRAM. Some day in the future possibly they may be used, otherwise they just take up some disk space.
3. We seem to line up with the mainline implementation and *should* have compatibility (I'd like at least one test of a mainline quant running here to confirm). I've already checked the other-way around where a Q8_0 from here works on mainline.

I'm happy to continue now quantizing and releasing some models and really appreciate all your effort getting your testing commits into this PR for final review/approval! Go team!

*EDIT*

Final note as @espen96 pointed out we still need to explicitly pass `--chat-template chatglm4` on ik_llama.cpp which does improve the chances of adding `/nothink` at the end of the prompt actually being honored.

---

👤 **Thireus** started a conversation on `convert_hf_to_gguf.py` on **2025-08-05** at **14:19:08**

@ubergarm, @espen96, can we try without these lines to see if it solves the `/nothink` situation?

---

👤 **Thireus** commented on **2025-08-05** at **14:19:55**

I've merged testing to this pull request.

About @espen96, I'm wondering if it has anything to do with with these patched lines on `convert_hf_to_gguf.py` - https://github.com/ikawrakow/ik_llama.cpp/pull/668/files#diff-ec77d8003b92ff283179456d36b8b56abf635e7b1232e70daf16676e8920ccf1R3989-R3992

---

👤 **Thireus** started a conversation on `src/llama.cpp` on **2025-08-05** at **14:28:57**

Can someone have a look if bypassing this asset is still necessary please?

---

👤 **ubergarm** commented on **2025-08-05** at **14:29:57**

>  I'm wondering if it has anything to do with with these patched lines on convert_hf_to_gguf.py

I believe that is realated only to passing `enable_thinking=False` to the llama-server endpoint which will inject `/nothink` at the end of the prompt.

The issue here is without explicitly setting `--chat-template chatglm4` it auto-detects incorrectly at startup so uses the wrong chat template.

#### with `--chat-template gchatglm4`
> INFO [                    main] chat template | tid="137380572002496" timestamp=1754403608 chat_example="[gMASK]<sop><|system|>\nYou are
a helpful assistant<|user|>\nHello<|assistant|>\nHi there<|user|>\nHow are you?<|assistant|>" built_in=false
INFO [                    main] HTTP server listening | tid="137380572002496" timestamp=1754403608 n_threads_http="767" port="8080" hostn
ame="127.0.0.1"

#### without it
> INFO [                    main] chat template | tid="140508746688704" timestamp=1754404055 chat_example="<|system|>\nYou are a helpful assistant\n<|user|>\nHello\n<|assistant|>\nHi there\n<|user|>\nHow are you?\n<|assistant|>\n" built_in=true

So while it would be nice to support the `enable_thinking=False` I'm not worried about that and don't want to make new GGUFs lol. The actual issue is around here with the autodetect: https://github.com/ikawrakow/ik_llama.cpp/blob/main/src/llama.cpp#L23267-L23271 I believe. Possibly one of the earlier checks is returning true due to something in the chat template and so not falling through and returning chatglm4 properly. Wait is it GLM or GML ?? haha...

---

👤 **Thireus** commented on **2025-08-05** at **14:39:36**

Got it.

Not sure if related but do we want to have an entry for GLM 4.5 in this section? https://github.com/Thireus/ik_llama.cpp/blob/glm-4.5-clean/src/llama.cpp#L1740

@ubergarm, you probably know this but the chat template is found in the header of the gguf and can be separated from the rest of the gguf using the llama-quantize option `--no-tensor-first-split`. So in case the chat template changes, if you had converted/quantized with that option, only the small 00001 gguf file would change, and you can kill the conversion/quantization process as soon as this file is available.

---

👤 **createthis** commented on **2025-08-05** at **14:40:22**

mainline `q4_k_m` works great on the `glm-4.5-testing` branch.

Performance is good too:
<img width="1512" height="201" alt="Screenshot 2025-08-05 at 10 33 35 AM" src="https://github.com/user-attachments/assets/dde79005-2e29-436c-8293-f5e794ee5c81" />

then by 38k context:

<img width="1512" height="265" alt="Screenshot 2025-08-05 at 10 36 50 AM" src="https://github.com/user-attachments/assets/77d99566-9870-45c2-b8e7-13500e696177" />

---

👤 **createthis** commented on **2025-08-05** at **14:47:35**

It is perhaps worth noting that performance appears to degrade with context length less quickly in mainline (nearly same prompts, I just skipped a poem generation turn):

<img width="1241" height="182" alt="Screenshot 2025-08-05 at 10 44 33 AM" src="https://github.com/user-attachments/assets/6a1e84ce-5863-4b9f-b83d-6bf1105ad0e0" />

<img width="1112" height="171" alt="Screenshot 2025-08-05 at 10 45 25 AM" src="https://github.com/user-attachments/assets/def5aa64-81c2-4036-94bc-071e34c604f2" />

---

👤 **espen96** commented on **2025-08-05** at **14:53:03**

looking good so far here as well.  feeling quite happy with the model so far, not seeing any obvious issues.
although all these benchmarks i'm seeing is making my souped up once top of the line gaming / workstation build feel like a toy hah. 

But yea, I don't see any problems with the model itself now. the /nothink command appears to work fine. I personally never saw any context issues in my 64k run. 

I can never seem to get stable enough performance out of my weird rig, so I can't speak to any differences between mainline and IK in regards to the performance quirks noted above, but it seems to run fine in general.

---

👤 **Thireus** commented on **2025-08-05** at **15:15:08**

> It is perhaps worth noting that performance appears to degrade with context length less quickly in mainline (nearly same prompts, I just skipped a poem generation turn):
> <img alt="Screenshot 2025-08-05 at 10 44 33 AM" width="1241" height="182" src="https://private-user-images.githubusercontent.com/1695880/474589642-6a1e84ce-5863-4b9f-b83d-6bf1105ad0e0.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NTQ0MDcxNTUsIm5iZiI6MTc1NDQwNjg1NSwicGF0aCI6Ii8xNjk1ODgwLzQ3NDU4OTY0Mi02YTFlODRjZS01ODYzLTRiOWYtYjgzZC02YmYxMTA1YWQwZTAucG5nP1gtQW16LUFsZ29yaXRobT1BV1M0LUhNQUMtU0hBMjU2JlgtQW16LUNyZWRlbnRpYWw9QUtJQVZDT0RZTFNBNTNQUUs0WkElMkYyMDI1MDgwNSUyRnVzLWVhc3QtMSUyRnMzJTJGYXdzNF9yZXF1ZXN0JlgtQW16LURhdGU9MjAyNTA4MDVUMTUxNDE1WiZYLUFtei1FeHBpcmVzPTMwMCZYLUFtei1TaWduYXR1cmU9YmQzNTRlOGU3NzhiYzVjNWQ0OWVkYzcxZWZmNmZhMThlZWU2ZDAyNTUwZWMyZWM0YjRlZTljZjE0MDY3NzRiOCZYLUFtei1TaWduZWRIZWFkZXJzPWhvc3QifQ._yN4PEZZ-Bm1rIA5-mAaALQO776w3DRCsR74KQGYdo0"> <img alt="Screenshot 2025-08-05 at 10 45 25 AM" width="1112" height="171" src="https://private-user-images.githubusercontent.com/1695880/474589700-def5aa64-81c2-4036-94bc-071e34c604f2.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NTQ0MDcxNTUsIm5iZiI6MTc1NDQwNjg1NSwicGF0aCI6Ii8xNjk1ODgwLzQ3NDU4OTcwMC1kZWY1YWE2NC04MWMyLTQwMzYtOTRiYy0wNzFlMzRjNjA0ZjIucG5nP1gtQW16LUFsZ29yaXRobT1BV1M0LUhNQUMtU0hBMjU2JlgtQW16LUNyZWRlbnRpYWw9QUtJQVZDT0RZTFNBNTNQUUs0WkElMkYyMDI1MDgwNSUyRnVzLWVhc3QtMSUyRnMzJTJGYXdzNF9yZXF1ZXN0JlgtQW16LURhdGU9MjAyNTA4MDVUMTUxNDE1WiZYLUFtei1FeHBpcmVzPTMwMCZYLUFtei1TaWduYXR1cmU9YWRkMzQ0YTk0MzIwYTE5ZWI3YzhjYzRmYjgyODczN2E2ZTFhODBhYzJlYzZjNjdjY2NlYzBlOThkMzJiYzZlNCZYLUFtei1TaWduZWRIZWFkZXJzPWhvc3QifQ.rCQATMwhQLKK36QvfxXVdviYg1ISdcqDjbal13PAQt0">

Is this a GLM-4.5 specific observation or does this happen to other models of the same type as well?

---

👤 **ikawrakow** commented on **2025-08-05** at **15:37:13**

@createthis In what sense do the two screenshots show that performance degrades less quickly with context length in mainline?

---

👤 **usrlocalben** commented on **2025-08-05** at **16:44:41**

works for me.
trailing `/nothink` seems to behave as it should.

neat: -Air w/4bpw, all up/gate exps fit on 48GB card. (RTX 8000)

10K ctx input example below (e.g. "summarize this EULA <10k ctx blob>")

GLM-4.5-Air, [ubergarm IQ4_KSS](https://huggingface.co/ubergarm/GLM-4.5-Air-GGUF)

setup:
```
-fa -fmoe
-b 4096 -ub 4096
-ctk f16
--n-gpu-layers 99
-ot "ffn_up_exps=CUDA0,ffn_gate_exps=CUDA0"
-ot exps=CPU
-op 26,0,27,0,29,0
--temp 0.6 --top-p 1.0
-c 32000
--chat-template chatglm4
-m /path/to/GLM-4.5-Air/ubergarm/IQ4_KSS/GLM-4.5-Air-IQ4_KSS-00001-of-00002.gguf
```

perf:
```
prompt eval time     =   24621.27 ms / 10110 tokens (    2.44 ms per token,   410.62 tokens per second)
generation eval time =   13261.98 ms /   263 runs   (   50.43 ms per token,    19.83 tokens per second)
```


GLM-4.5, [unsloth Q6_K](https://huggingface.co/unsloth/GLM-4.5-GGUF)

setup, same but different offload:
```
...
-ot "blk\.(3|4|5|6|7|8|9|10|11)\.ffn_up_exps=CUDA0,blk\.(3|4|5|6|7|8|9|10|11)\.ffn_gate_exps=CUDA0"
...
-m /path/to/GLM-4.5/unsloth/Q6_K/GLM-4.5-Q6_K-00001-of-00006.gguf
```

perf:
```
prompt eval time     =   92889.47 ms / 10110 tokens (    9.19 ms per token,   108.84 tokens per second)
generation eval time =   58377.62 ms /   451 runs   (  129.44 ms per token,     7.73 tokens per second)
```

---

👤 **ubergarm** commented on **2025-08-05** at **17:03:30**

@Thireus 

Okay so looking into it with more detail seems like we're auto-detecting chat template incorrectly as `LLM_CHAT_TEMPLATE_FALCON_3` given the heuristic is to look in the template matching `<|assistant|>` and `<|user|>` which also appear in GLM-4 template. 

The fix here is twofold:

1. Rename GML to GLM to prevent future confusion. This wasn't effecting anything just confusing typo.
2. Move the auto-detecting of GLM-4 up higher above FALCON_3 as the `[gMASK]<sop>` seems more unique and less likely to trigger a false positive.

Here is that patch:

<details>

<summary>👈 patch auto-detection of GLM4 chat template</summary>

```diff
diff --git a/src/llama.cpp b/src/llama.cpp
index ca68be50..28757a86 100644
--- a/src/llama.cpp
+++ b/src/llama.cpp
@@ -1740,8 +1740,8 @@ enum llm_chat_template {
     LLM_CHAT_TEMPLATE_DEEPSEEK_3,
     LLM_CHAT_TEMPLATE_COMMAND_R,
     LLM_CHAT_TEMPLATE_LLAMA_3,
-    LLM_CHAT_TEMPLATE_CHATGML_3,
-    LLM_CHAT_TEMPLATE_CHATGML_4,
+    LLM_CHAT_TEMPLATE_CHATGLM_3,
+    LLM_CHAT_TEMPLATE_CHATGLM_4,
     LLM_CHAT_TEMPLATE_MINICPM,
     LLM_CHAT_TEMPLATE_EXAONE_3,
     LLM_CHAT_TEMPLATE_RWKV_WORLD,
@@ -1781,8 +1781,8 @@ static const std::map<std::string, llm_chat_template> LLM_CHAT_TEMPLATES = {
     { "deepseek3",         LLM_CHAT_TEMPLATE_DEEPSEEK_3        },
     { "command-r",         LLM_CHAT_TEMPLATE_COMMAND_R         },
     { "llama3",            LLM_CHAT_TEMPLATE_LLAMA_3           },
-    { "chatglm3",          LLM_CHAT_TEMPLATE_CHATGML_3         },
-    { "chatglm4",          LLM_CHAT_TEMPLATE_CHATGML_4         },
+    { "chatglm3",          LLM_CHAT_TEMPLATE_CHATGLM_3         },
+    { "chatglm4",          LLM_CHAT_TEMPLATE_CHATGLM_4         },
     { "minicpm",           LLM_CHAT_TEMPLATE_MINICPM           },
     { "exaone3",           LLM_CHAT_TEMPLATE_EXAONE_3          },
     { "rwkv-world",        LLM_CHAT_TEMPLATE_RWKV_WORLD        },
@@ -23851,6 +23851,11 @@ static llm_chat_template llama_chat_detect_template(const std::string & tmpl) {
                 return LLM_CHAT_TEMPLATE_LLAMA_2;
             }
         }
+    } else if (tmpl_contains("[gMASK]sop")) {
+        // chatglm3-6b
+        return LLM_CHAT_TEMPLATE_CHATGLM_3;
+    } else if (tmpl_contains("[gMASK]<sop>")) {
+        return LLM_CHAT_TEMPLATE_CHATGLM_4;
     } else if (tmpl_contains("<|assistant|>") && tmpl_contains("<|end|>")) {
         return LLM_CHAT_TEMPLATE_PHI_3;
     } else if (tmpl_contains("<|assistant|>") && tmpl_contains("<|user|>")) {
@@ -23883,11 +23888,6 @@ static llm_chat_template llama_chat_detect_template(const std::string & tmpl) {
         return LLM_CHAT_TEMPLATE_COMMAND_R;
     } else if (tmpl_contains("<|start_header_id|>") && tmpl_contains("<|end_header_id|>")) {
         return LLM_CHAT_TEMPLATE_LLAMA_3;
-    } else if (tmpl_contains("[gMASK]sop")) {
-        // chatglm3-6b
-        return LLM_CHAT_TEMPLATE_CHATGML_3;
-    } else if (tmpl_contains("[gMASK]<sop>")) {
-        return LLM_CHAT_TEMPLATE_CHATGML_4;
     } else if (tmpl_contains(LU8("<用户>"))) {
         // MiniCPM-3B-OpenHermes-2.5-v2-GGUF
         return LLM_CHAT_TEMPLATE_MINICPM;
@@ -24170,7 +24170,7 @@ static int32_t llama_chat_apply_template_internal(
         if (add_ass) {
             ss << "<|start_header_id|>assistant<|end_header_id|>\n\n";
         }
-    } else if (tmpl == LLM_CHAT_TEMPLATE_CHATGML_3) {
+    } else if (tmpl == LLM_CHAT_TEMPLATE_CHATGLM_3) {
         // chatglm3-6b
         ss << "[gMASK]" << "sop";
         for (auto message : chat) {
@@ -24180,7 +24180,7 @@ static int32_t llama_chat_apply_template_internal(
         if (add_ass) {
             ss << "<|assistant|>";
         }
-    } else if (tmpl == LLM_CHAT_TEMPLATE_CHATGML_4) {
+    } else if (tmpl == LLM_CHAT_TEMPLATE_CHATGLM_4) {
         ss << "[gMASK]" << "<sop>";
         for (auto message : chat) {
             std::string role(message->role);
```

</details>

---

> these patched lines on convert_hf_to_gguf.py - https://github.com/ikawrakow/ik_llama.cpp/pull/668/files#diff-ec77d8003b92ff283179456d36b8b56abf635e7b1232e70daf16676e8920ccf1R3989-R3992

If this is not already here (don't think it is?), then yes probably a "good thing" to bring it over. I'm not 100% sure setting `enable_thinking=False` as url parameter or json body field or whatever works here on llama-server API endpoint as I've not tested it myself.

> the chat template is found in the header of the gguf and can be separated from the rest of the gguf using the llama-quantize option --no-tensor-first-split. So in case the chat template changes, if you had converted/quantized with that option, only the small 00001 gguf file would change, and you can kill the conversion/quantization process as soon as this file is available.

Yes, I've only released one quant, I can make the fix re-writing the gguf metadata I suppose without needing to full re-convert re-quantize again. Probably best to squash this now before releasing a bunch of quants lol...

Thanks

---

@createthis 

> performance appears to degrade with context length

Are those two screenshots **both** from mainline llama-server? 

This is what output looks like from ik_llama.cpp llama-server

```bash
INFO [           print_timings] prompt eval time     =     326.14 ms /    25 tokens (   13.05 ms per token,    76.65 tokens per second) | tid="134702932592832" timestamp=1754413131 id_slot=0 id_task=0 t_prompt_processing=326.142 n_prompt_tokens_processed=25 t_token=13.045679999999999 n_tokens_second=76.65372751746172
INFO [           print_timings] generation eval time =    2921.37 ms /    62 runs   (   47.12 ms per token,    21.22 tokens per second) | tid="134702932592832" timestamp=1754413131 id_slot=0 id_task=0 t_token_generation=2921.367 n_decoded=62 t_token=47.118822580645165 n_tokens_second=21.22294117788008
INFO [           print_timings]           total time =    3247.51 ms | tid="134702932592832" timestamp=1754413131 id_slot=0 id_task=0 t_prompt_processing=326.142 t_token_generation=2921.367 t_total=3247.509
```

I'd suggest using `llama-sweep-bench` both here and on mainline (with my branch https://github.com/ubergarm/llama.cpp/tree/ug/port-sweep-bench) if you want to do a/b testing of speed between forks. I just rebased my branch to tip of mainline.

---

👤 **Thireus** commented on **2025-08-05** at **17:16:21**

@ubergarm, "it is" already there.

I've pushed your patch and restored the original GGLM_ASSERT on the -testing branch. Could you please try to launch the model with these changes? Thank you so much!

---

👤 **ubergarm** commented on **2025-08-05** at **17:36:17**

@Thireus 

Okay just confirmed things seem to be working now on latest [41a235b](https://github.com/Thireus/ik_llama.cpp/commit/41a235bbcad55e3fec7c07755793c52f2f2c61b5). No need to explicitly specify `--chat-template glm4` anymore.

>  "it is" already there.

Great! In fact it has been here since before I made my GGUFs psure, so no need for me to overwrite my chate template psure YAY!:

```
     43: STRING     |        1 | tokenizer.chat_template = '[gMASK]<sop>\n{%- if tools -%}\n<|system|>\n# Tools\n\nYou may call one or more functions to assist with the user query.\n\nYou are provided with function signatures within <tools></tools> XML tags:\n<tools>\n{% for tool in tools %}\n{{ tool | tojson(ensure_ascii=False) }}\n{% endfor %}\n</tools>\n\nFor each function call, output the function name and arguments within the following XML format:\n<tool_call>{function-name}\n<arg_key>{arg-key-1}</arg_key>\n<arg_value>{arg-value-1}</arg_value>\n<arg_key>{arg-key-2}</arg_key>\n<arg_value>{arg-value-2}</arg_value>\n...\n</tool_call>{%- endif -%}\n{%- macro visible_text(content) -%}\n    {%- if content is string -%}\n        {{- content }}\n    {%- elif content is iterable and content is not mapping -%}\n        {%- for item in content -%}\n            {%- if item is mapping and item.type == \'text\' -%}\n                {{- item.text }}\n            {%- elif item is string -%}\n                {{- item }}\n            {%- endif -%}\n        {%- endfor -%}\n    {%- else -%}\n        {{- content }}\n    {%- endif -%}\n{%- endmacro -%}\n{%- set ns = namespace(last_user_index=-1) %}\n{%- for m in messages %}\n    {%- if m.role == \'user\' %}\n        {% set ns.last_user_index = loop.index0 -%}\n    {%- endif %}\n{%- endfor %}\n{% for m in messages %}\n{%- if m.role == \'user\' -%}<|user|>\n{% set content = visible_text(m.content) %}{{ content }}\n{{- \'/nothink\' if (enable_thinking is defined and not enable_thinking and not content.endswith("/nothink")) else \'\' -}}\n{%- elif m.role == \'assistant\' -%}\n<|assistant|>\n{%- set reasoning_content = \'\' %}\n{%- set content = visible_text(m.content) %}\n{%- if m.reasoning_content is string %}\n    {%- set reasoning_content = m.reasoning_content %}\n{%- else %}\n    {%- if \'</think>\' in content %}\n        {%- set reasoning_content = content.split(\'</think>\')[0].rstrip(\'\\n\').split(\'<think>\')[-1].lstrip(\'\\n\') %}\n        {%- set content = content.split(\'</think>\')[-1].lstrip(\'\\n\') %}\n    {%- endif %}\n{%- endif %}\n{%- if loop.index0 > ns.last_user_index and reasoning_content -%}\n{{ \'\\n<think>\' + reasoning_content.strip() +  \'</think>\'}}\n{%- else -%}\n{{ \'\\n<think></think>\' }}\n{%- endif -%}\n{%- if content.strip() -%}\n{{ \'\\n\' + content.strip() }}\n{%- endif -%}\n{% if m.tool_calls %}\n{% for tc in m.tool_calls %}\n{%- if tc.function %}\n    {%- set tc = tc.function %}\n{%- endif %}\n{{ \'\\n<tool_call>\' + tc.name }}\n{% set _args = tc.arguments %}\n{% for k, v in _args.items() %}\n<arg_key>{{ k }}</arg_key>\n<arg_value>{{ v | tojson(ensure_ascii=False) if v is not string else v }}</arg_value>\n{% endfor %}\n</tool_call>{% endfor %}\n{% endif %}\n{%- elif m.role == \'tool\' -%}\n{%- if m.content is string -%}\n{%- if loop.first or (messages[loop.index0 - 1].role != "tool") %}\n    {{- \'<|observation|>\' }}\n{%- endif %}\n{{- \'\\n<tool_response>\\n\' }}\n{{- m.content }}\n{{- \'\\n</tool_response>\' }}\n{%- else -%}\n<|observation|>{% for tr in m.content %}\n\n<tool_response>\n{{ tr.output if tr.output is defined else tr }}\n</tool_response>{% endfor -%}\n{% endif -%}\n{%- elif m.role == \'system\' -%}\n<|system|>\n{{ visible_text(m.content) }}\n{%- endif -%}\n{%- endfor -%}\n{%- if add_generation_prompt -%}\n    <|assistant|>{{- \'\\n<think></think>\' if (enable_thinking is defined and not enable_thinking) else \'\' -}}\n{%- endif -%}'
```

fwiw I tried setting `enable_thinking=False` in my python client JSON Post and seems like it does *not* actually disable thinking. Here is my JSON POST body data to the llama-server /chat/completions/ API endpoint:

```json
{"messages": [{"role": "system", "content": "You are a helpful AI assistant."}, {"role": "user", "content": "Count form 1 to 10 in French."}], "stream": true, "model": "ubergarm/GLM-4.5-Air-IQ4_KSS", "temperature": 0.6, "top_p": 0.95, "n_predict": -1, "cache_prompt": true, "enable_thinking": false}
```

So I'll stick to adding `/nothink` at the end of the prompt haha...

---

👤 **Thireus** commented on **2025-08-05** at **17:47:50**

@ubergarm, sorry I just realised I also revert the GGML_ASSERT revert... so you actually didn't test it 🤦‍♂️

Would you mind giving it another go?

This is the change: https://github.com/Thireus/ik_llama.cpp/commit/323e7f3dd647d18940998fb152a8852d3faf0ca6

You'll just need to load a model, if it succeeds then it means the original GGLM_ASSERT is good and the patch I implemented is no longer needed. Thank you so much.

---

👤 **ubergarm** commented on **2025-08-05** at **17:57:43**

@Thireus 

All good, this one has been more of an adventure than anyone anticipated lol. 

Just tested Thireus/ik_llama.cpp glm-4.5-clean@cae058f1 

```
$ ./build/bin/llama-server --version
version: 3841 (cae058f1)
built with cc (Ubuntu 13.3.0-6ubuntu2~24.04) 13.3.0 for x86_64-linux-gnu
```

Working fine with my [ubergarm/GLM-4.5-IQ4_KSS](https://huggingface.co/ubergarm/GLM-4.5-Air-GGUF/tree/main/IQ4_KSS) quant! Wahooo! :partying_face:

---

👤 **Thireus** commented on **2025-08-05** at **18:02:26**

> Just tested Thireus/ik_llama.cpp glm-4.5-clean@cae058f1

Sorry, I meant on the glm-4.5-testing branch: https://github.com/Thireus/ik_llama.cpp/commit/323e7f3dd647d18940998fb152a8852d3faf0ca6

Edit: gpt-oss is out, all the focus is going to be there now haha

---

👤 **espen96** commented on **2025-08-05** at **18:22:43**

> > Just tested Thireus/ik_llama.cpp glm-4.5-clean@cae058f1
> 
> Sorry, I meant on the glm-4.5-testing branch: [Thireus@323e7f3](https://github.com/Thireus/ik_llama.cpp/commit/323e7f3dd647d18940998fb152a8852d3faf0ca6)
> 
> Edit: gpt-oss is out, all the focus is going to be there now haha


Haha, yea.

I've found basically all open weight (especially Moe) models to be extremely lacking when it comes to general knowledge usecases...
Even topics you'd naively assume to be safe. GPT-4o did well, Deepseek and Kimi K2 were highly problematic, Qwen as well. Llama 3.3 did fine.

 So, these OAI models is the perfect western countepart to the largely Chinese MoE models I've been using for my research. 

Expect to see me help everyone out in testing these two models to the end hah

---

👤 **Thireus** commented on **2025-08-05** at **18:29:02**

@ubergarm - that last layer missing in imatrix is giving me a major headache for small quants.

---

👤 **ubergarm** commented on **2025-08-05** at **18:35:10**

@Thireus 

I just tested this and it loads and runs in a couple short inferencing tests.

```bash
Thireus/ik_llama.cpp$ git branch
* glm-4.5-testing
  main
Thireus/ik_llama.cpp$ ./build/bin/llama-server --version
version: 3839 (323e7f3d)
built with cc (Ubuntu 13.3.0-6ubuntu2~24.04) 13.3.0 for x86_64-linux-gnu
```

---

> that last layer missing in imatrix is giving me a major headache for small quants.

Is this on Air or the big one or both? I haven't tried going too small yet. Does it error out even if you set to full Q8_0?

>  gpt-oss  

whaa never heard of it, guess i gotta hit refresh on r/LocalLLaMA :skull: :headstone: :sob:

---

👤 **createthis** commented on **2025-08-05** at **18:37:38**

> gpt-oss

I'm getting my breaking news from this PR too. lol.

---

👤 **Thireus** commented on **2025-08-05** at **18:58:59**

@ubergarm, thanks. It's on both. Try iq1_m for example.

---

👤 **ubergarm** commented on **2025-08-05** at **19:01:41**

> @ubergarm, thanks. It's on both. Try iq1_m for example.

Are you passing a custom-q override for the last blk to q8_0? 

I'll try soon, have to head out, thanks for doing the heavy lifting on this one!

---

👤 **Thireus** commented on **2025-08-05** at **19:10:11**

@ubergarm when quantizing:

```
[ 797/ 803]                 blk.46.attn_k.weight - [ 4096,  1024,     1,     1], type =   bf16, Using custom type iq1_m for tensor blk.46.attn_k.weight

====== llama_model_quantize_internal: did not find weights for blk.46.attn_k.weight


============================================================
Missing importance matrix for tensor blk.46.attn_k.weight in a very low-bit quantization
The result will be garbage, so bailing out
============================================================
```

Happens for all the tensors of layer 46 on -Air.

Edit: You did mention this actually...

> Also right my new imatrix totally skips that last layer, and I see a note here on mainline suggesting because that last layer is totally skipped there too, to not quantize those tensors too small given no imatrix data now? ggml-org/llama.cpp[#14939](https://github.com/ikawrakow/ik_llama.cpp/issues/14939) (comment)
> 
> I'm a little confused and want to look more closely at the differences around how skipping those NextN tesnors vs skipping the whole last layer is implemented in the graph before releasing more quants. Though the big GLM-4.5 imatrix is uploaded now (missing last layer data now) here: https://huggingface.co/ubergarm/GLM-4.5-GGUF

---

👤 **Nexesenex** commented on **2025-08-05** at **20:04:31**

The last changes look good on my side as well.
I now have a VRAM occupation as expected.

Oh, by the way, do we have this option in IK_Llama? https://github.com/ggml-org/llama.cpp/pull/15077

> > Just tested Thireus/ik_llama.cpp glm-4.5-clean@cae058f1
> 
> Sorry, I meant on the glm-4.5-testing branch: [Thireus@323e7f3](https://github.com/Thireus/ik_llama.cpp/commit/323e7f3dd647d18940998fb152a8852d3faf0ca6)
> 
> Edit: gpt-oss is out, all the focus is going to be there now haha

Insane days. One SOTA 100b from China, and one from the West, then? Downloading. :D

---

👤 **ikawrakow** commented on **2025-08-05** at **20:24:26**

@Thireus To quantize iq1_m (or other very low bow quants) add --ignore-imatrix-rules to your llama-quantize command.

---

👤 **SamuelOliveirads** commented on **2025-08-05** at **20:26:27**

> My other thought about chat template stuff is after running gguf dump i see:
> 
> ```
> $ python gguf-py/scripts/gguf_dump.py /mnt/data/models/Thireus/GLM-4.5-THIREUS-BF16-SPECIAL_SPLIT/GLM-4.5-Thireus-Q8_0.gguf
> ...
>      44: STRING     |        1 | tokenizer.chat_template = '[gMASK]<sop>\n{%- if tools -%}\n<|system|>\n# Tools\n\nYou may ca'
> ```
> 
> This looks right with what the official chat template is here: https://huggingface.co/zai-org/GLM-4.5?chat_template=default
> 
> When starting the model with llama-server it says:
> 
> ```
> INFO [                    main] chat template | tid="123750572615872" timestamp=1754072872 chat_example="<|system|>\nYou are a helpful assistant\n<|user|>\nHello\n<|assistant|>\nHi there\n<|user|>\nHow are you?\n<|assistant|>\n" built_in=true
> ```
> 
> So for `/chat/completions/` API endpoint it seems like it will be properly applying the basic template, however I haven't run their jinja through the decoder script (i gotta find my notes hah) and made sure it matches up closely enough. But in limited testing llama-server seems to be working okay.
> 
> I'll upload your imatrix soon after it finishes to here: https://huggingface.co/ubergarm/GLM-4.5-GGUF
> 
> Once again though, I'm hesitant to release quants just yet until things are more tested, but excited to see this coming along! Thanks for your efforts!


I was trying your IQ4 quant with --chat-template chatglm4, and it worked well until I tried to make changes like deleting a word and asking the model to continue, or hitting regenerate. After that, the output turned into an infinite stream of broken tokens like:
```
<|assistant|>
>")>")>")>")>")>")>")>")>")>")>")>")>")>")>")>")>")>")>")>")>")>")>")>")>")>")>")>")>")>")>")>")>")>")>")>")>")>")>")>")
```
Resetting the context, trying different chats, even using the direct endpoint didn’t fix it only restarting the server worked.

I downloaded the [Unsloth Q8 quant](https://huggingface.co/unsloth/GLM-4.5-Air-GGUF?show_file_info=UD-Q8_K_XL%2FGLM-4.5-Air-UD-Q8_K_XL-00001-of-00003.gguf) and ran the same tasks. The output was perfect and coherent, so this may be a problem specific to the previous quant.

This was my build:
```
version: 3838 (46c3fa95)
built with cc (Ubuntu 13.3.0-6ubuntu2~24.04) 13.3.0 for x86_64-linux-gnu
```

---

👤 **Thireus** commented on **2025-08-05** at **20:36:54**

> @Thireus To quantize iq1_m (or other very low bow quants) add --ignore-imatrix-rules to your llama-quantize command.

omg, thank you!

---

👤 **simple6502** commented on **2025-08-06** at **06:30:38**

Been watching this PR since the beginning and just recently got to try GLM-4.5-Air within it, and it has been working very well! Got up to 10K context with no issues, /nothink works well, and runs very fast when ingesting a lot of context at once!

Running Unsloth's UD-Q5_K_XL quant of GLM-4.5-Air currently. I plan to switch over to @ubergarm's IQ5_K quant after it finishes downloading.

My system is a 2x 3090 system with each card power limited, 128GBs of DDR4 RAM running at 2666MHz I believe, and a 16-core Xeon E5-2697A V4. 

My settings I am using to load the model (under llama-swap):

```
  "glm-4.5-air":
    cmd: ${ik_llama_cpp}
      --model /media/nix/Storage/Other/AI/GGUFmodels/glm-4.5-air/GLM-4.5-Air-UD-Q5_K_XL-00001-of-00002.gguf
      --ctx-size 32768
      --temp 0.6
      --top-p 0.8
      --top-k 20
      --min-p 0.0
      -fa
      -fmoe
      --chat-template chatglm4
      -ts 24,24
      -ot "blk.[0-9].ffn_up_exps=CUDA0,blk.[0-9].ffn_gate_exps=CUDA0,blk.[0-9].ffn_down_exps=CUDA0"
      -ot "blk.1[0-8].ffn_up_exps=CUDA1,blk.1[0-8].ffn_gate_exps=CUDA1,blk.1[0-8].ffn_down_exps=CUDA1"
      -ot ".ffn_.*_exps.=CPU"
      -ngl 99
      -ub 4096
      -b 4096
      --parallel 1
      --threads 14
```

I could not be happier! I plan to try a lower quant of GLM-4.5 soon to see how it performs on my system, but I just wanted to share my experience and thank you all for your hard work so far!

---

👤 **Thireus** reviewed this pull request 💬 on **2025-08-06** at **14:13:03**

I'm happy with this version.

---

👤 **Thireus** commented on **2025-08-06** at **14:16:25**

We still need at least 1 approving review to merge this pull request.

---

👤 **espen96** commented on **2025-08-06** at **15:06:22**

As I mentioned last time, everything seemed fine to me with normal use, up to 64k context. Unless anything has changed for the worse since then, I'm happy with the pr. 

I should be able to run the IQ5_K quant, I'll take it for a spin and report back to make sure everything is peachy.



@ubergarm could I request something in-between the Iq4_KSS and IQ5_K for size, for the Air variant? Perhaps IQ4_K would be a good option? 

The IQ5_K could end up being a bit tight for daily use. Gotta have some resources to use the computer as well, this is a workstation not a server sadly.

---

👤 **ubergarm** approved this pull request ✅ on **2025-08-06** at **15:57:14**

---

👤 **ikawrakow** commented on **2025-08-06** at **16:02:13**

I'll be back tonight and will check and merge the PR first thing tomorrow morning my time.

---

👤 **ChicoPinto70** commented on **2025-08-06** at **16:55:04**

I also wanna thank all of you for this PR. I just ran it with the Ubergarm GLM 4.5 IQ5_K, 64K ctx in the roo code. It completes sucessfully the watermelon prompt and the voxel pagoda garden tests.  :100:

---

👤 **ubergarm** commented on **2025-08-06** at **17:21:57**

> I also wanna thank all of you for this PR. I just ran it with the Ubergarm GLM 4.5 IQ5_K, 64K ctx in the roo code. It completes sucessfully the watermelon prompt and the voxel pagoda garden tests. 💯

Great to hear you got that big one working well! Gotta say the GLM-4.5-355B-A32B perplexities are *misbehaving* as the IQ5_K (which is almost exactly 6.0 BPW) is showing even *lower perplexity* than the Q8_0 and BF16...  I'll try another quant around IQ4_K size and see how it looks:

<img width="2069" height="1400" alt="image" src="https://github.com/user-attachments/assets/a2283f99-0ed1-490d-8e75-59c48a77fe88" />


The smaller GLM-4.5-Air-106B-A12B has a more standard looking perplexity trend:

<img width="2068" height="1400" alt="image" src="https://github.com/user-attachments/assets/d4bc6aed-d5c3-47eb-b1c5-174a141b5cb6" />

---

👤 **espen96** commented on **2025-08-06** at **17:36:35**

went up to and past the 32k context limit I set.
Still coherent around 32k, skipped ahead in the multi turn chat by injecting almost all of "Romeo and Juliet". it is still coherent, it did not get confused by that, nor did it start rambling sonnets at me,

the nothink command worked at 28K
In the next turn however it reverted to thinking. I am a bit unsure if that's what they intended. As mentioned I saw something similar in the offical hosted chat. in that impementation it would jump back and forth between thnking and not thinking though.

After hitting the token limit and going past it, the model  started to speak normally inside the reasoning, rather than cleanly separating them. 

nothink did turn off reasoning again, but as mentioned it instantly reverts to thinking on the next turn.

---

👤 **Thireus** commented on **2025-08-06** at **17:41:13**

Maybe a dumb question but don't you need to insert /nothink for each of the prompt submitted by the user? Isn't it the intended behaviour to revert back to thinking mode if new prompts don't mention /nothink? (Personally I'd like this behaviour)

---

👤 **ubergarm** commented on **2025-08-06** at **17:52:34**

> don't you need to insert /nothink for each of the prompt submitted by the user? Isn't it the intended behaviour to revert back to thinking mode if new prompts don't mention /nothink?

My impression is that hybrid thinking LLMs don't have any *logical* "thinking" and "nothinking" modes, but have been trained on two different instruct datasets (not 100% sure on this). 

In my experience with GLM specifically, if I turn off thinking in the first round of chat, it will often remain off on the second round. I assume this is because `/nothink` still appears in the text sent to the LLM from the first round chat thread.

Hybrid thinking LLMs while convenient having just a single model, can definitely feel a little wonky.

---

👤 **espen96** commented on **2025-08-06** at **17:55:00**

> Maybe a dumb question but don't you need to insert /nothink for each of the prompt submitted by the user? Isn't it the intended behaviour to revert back to thinking mode if new prompts don't mention /nothink? (Personally I'd like this behaviour)

reading the jinja template again, you might be right
Personally I think that is an odd choice, but if that's how it is supposed to work, then that is how it is supposed to work.

did a bit more testing with it. it is slightly wonky at times, but it appears to work for the most part at the very least.

I'd just ship it

---

👤 **createthis** commented on **2025-08-06** at **17:56:05**

I think this is the relevant line of the template:

> ```{{- '/nothink' if (enable_thinking is defined and not enable_thinking and not content.endswith("/nothink")) else '' -}}```

---

👤 **ubergarm** commented on **2025-08-06** at **18:00:33**

@createthis 

> I think this is the relevant line of the template:

Refer to above comment: https://github.com/ikawrakow/ik_llama.cpp/pull/668#issuecomment-3156025629

I believe that is supposed to just be a secondary path to inject `/nothink` text into the end of the prompt and not even sure it works here or if it is supposed to be used for example with transformers call passing kwargs `enable_thinking` or what... Or maybe you can use it with the `/text/completions` API endpoint if you're doing all the templating on the client side. I mostly just use the simple `/chat/completions` endpoint with the simple internal defined templates.

---

👤 **trilog-inc** commented on **2025-08-06** at **21:04:33**

Hi,

Love the model up until now!(GLM-4.5, not Air) It got quite far in Cline as an agent. However, it started to lose its marbles after about 50K tokens. Everything was very coherent until then and it doesn't matter if I was using ik_llama.cpp or llama.cpp. Using your IQ5_K quant. I guess I will try the IQ4_K to confirm if its the quant or the code ...

---

👤 **ikawrakow** approved this pull request ✅ on **2025-08-07** at **04:54:48**

---

👤 **Thireus** commented on **2025-08-07** at **05:01:35**

Thank you @ikawrakow for the review and the merge.

Thank you to our friends at llama.cpp for the original code and code improvements that followed!

Thank you all for the valuable help with bug fixing and testing the code! It was indeed quite an adventure like @ubergarm said.

🥳

---

👤 **Ph0rk0z** commented on **2025-08-07** at **17:09:32**

For some reason putting extra down exp layers on cuda0 causes it to ignore the context and spit gibberish. I will try some others like gate+up and see what happens. Extra tensors on other GPU are fine.

---

👤 **ubergarm** commented on **2025-08-08** at **15:25:11**

> For some reason putting extra down exp layers on cuda0 causes it to ignore the context and spit gibberish. I will try some others like gate+up and see what happens. Extra tensors on other GPU are fine.

In general I try not to bother with splitting gate/down/up tensors across GPUs especially when using `-fmoe`. I'm not sure why people do this, perhaps trying to eek out that last 500MB of VRAM on a given GPU?

---

👤 **Thireus** commented on **2025-08-08** at **18:54:07**

> For some reason putting extra down exp layers on cuda0 causes it to ignore the context and spit gibberish. I will try some others like gate+up and see what happens. Extra tensors on other GPU are fine.

Experiencing the same.

~Don't use `-ot exps=CPU`. Instead, explicitly specify the layers like so: `-ot "blk\.(8[0-9]|90|91|92)\.ffn_.*=CPU"`.~ (not quite sure what's going on, but looks like it's related to the context size)

Examples:

```
# Spitting junk
CUDA_DEVICE_ORDER=PCI_BUS_ID CUDA_VISIBLE_DEVICES=0,1,2 ~/ik_llama-main-b4065-a09bed8-bin-win-cuda-12.8-x64-avx512/llama-server -m GLM-4.5-THIREUS-BF16-SPECIAL_TENSOR-00001-of-01762.gguf -fa \
  -amb 1024 \
  -fmoe \
  -ctk f16 \
  -c 65536 \
  -ngl 99 \
  -ot "blk\.([0-9]|[1-2][0-9]|3[0-9])\.ffn_.*=CUDA0" -ot "blk\.([4-6][0-9]|7[0-9])\.ffn_.*=CUDA1" -ot "blk\.(8[0-6])\.ffn_.*=CUDA2" \
  -ot exps=CPU \
  -b 4096 -ub 2048 \
  --warmup-batch \
  --no-mmap \
  --threads 36 \
  --main-gpu 0
```
```
# Not spitting junk
CUDA_DEVICE_ORDER=PCI_BUS_ID CUDA_VISIBLE_DEVICES=0,1,2 ~/ik_llama-main-b4065-a09bed8-bin-win-cuda-12.8-x64-avx512/llama-server -m GLM-4.5-THIREUS-BF16-SPECIAL_TENSOR-00001-of-01762.gguf -fa \
  -amb 1024 \
  -fmoe \
  -ctk f16 \
  -c 4096 \
  -ngl 99 \
  -ot "blk\.([0-9]|[1-2][0-9]|3[0-6])\.ffn_.*=CUDA0" -ot "blk\.(37|38|[4-6][0-9]|7[0-2])\.ffn_.*=CUDA1" -ot "blk\.(7[3-9])\.ffn_.*=CUDA2" \
  -ot "blk\.(8[0-9]|90|91|92)\.ffn_.*=CPU" \
  -b 4096 -ub 2048 \
  --warmup-batch \
  --no-mmap \
  --threads 36 \
  --main-gpu 0
```

---

👤 **Ph0rk0z** commented on **2025-08-08** at **19:07:52**

Hmm.. that alleviated it? Also from looking at my GPU load, I wonder how the other layers are getting split. Perhaps attn or shexp are ending up on different GPU for a block. I shouldn't see one GPU twirling at 2% while another is at 5% and another at 50% (offloaded matrix multiplication).

Cuda0 and cuda3 get loaded up the heaviest for unknown reasons. Maybe there's some more speed to be gained here keeping things together. I could try doing ffn* like you instead of ffn*_exp but still no idea where the other pieces of the model end up. Will give it a go.

I notice this model isn't as sensitive as d/s and qwen to an extra tensor on gpu or not.

---

👤 **Ph0rk0z** commented on **2025-08-08** at **19:23:07**

Yea I'll see what I get changing the tensors around. Also, FYI, changing AMB didn't seem to change the size of the buffers. I get same sized ones with 1024,512,64, etc. 

here is what I mean about cuda0/3


```
    CUDA_VISIBLE_DEVICES=0,1,2,3 numactl --interleave=all ./bin/llama-sweep-bench \
    -m GLM-4.5-UD-Q3_K_XL-00001-of-00004.gguf \
    -t 48 \
    -c 34816 \
    --numa distribute \
    -ngl 94 \
    -ctk q8_0 \
    -ctv q8_0 \
    -fa \
    -fmoe \
    -b 4096 \
    -ub 4096 \
    -ot "blk\.(0|1|2|3|4|5|6|7|8|9|10|11)\.ffn_.*_exps.=CUDA0" \
    -ot "blk\.(12|13|14|15|16|17|18|19|20|21|22)\.ffn_.*_exps.=CUDA1" \
    -ot "blk\.(23|24|25|26|27|28|29|30|31|32|33)\.ffn_.*_exps.=CUDA2" \
    -ot "blk\.(34|35|36|37|38|39|40|41|42)\.ffn_.*_exps.=CUDA3" \
    -ot "blk\.(43|44)\.ffn_(down)_exps\.weight=CUDA0" \
    -ot "blk\.(45|46)\.ffn_(down)_exps\.weight=CUDA3" \
    -ot "\.ffn_.*_exps.=CPU"
    
    
llama_new_context_with_model:      CUDA0 compute buffer size =  2336.02 MiB
llama_new_context_with_model:      CUDA1 compute buffer size =  1008.02 MiB
llama_new_context_with_model:      CUDA2 compute buffer size =  1008.02 MiB
llama_new_context_with_model:      CUDA3 compute buffer size =  2448.00 MiB
llama_new_context_with_model:  CUDA_Host compute buffer size =   848.05 MiB
```

---

👤 **Thireus** commented on **2025-08-08** at **19:29:17**

There's definitely something up with the memory allocation of the context. When I increase the context, there is an invisible limit after which the model spits junk (example: "chiam chiam chiam chiam chiam chiam chiam chiam chiam chiam chiam chiam chiam chiam chiam chiam chiam chiam chiam chiam chiam chiam chiam chiam chiam").

Almost as if the context buffer couldn't be correctly allocated, but llama doesn't tell me about it and just proceeds.

---

👤 **AesSedai** commented on **2025-08-08** at **19:47:44**

@Ph0rk0z have you tried with `-b 2048 -ub 2048`? I was having the same issue with incoherent context over ~32k when I had mine set to 4096 / 4096. Changing it to 2048 / 2048, with no other changes, fixed the issue for me.

---

👤 **Ph0rk0z** commented on **2025-08-08** at **19:51:17**

I can really use only up to ~32k ctx and haven't tested much beyond a few turns with this model yet (only 200GB/s sysram). ffn* is about 3t/s slower than doing exp. 

Also finding that 4096 batches give you better total context processing speed but rtr and smaller batches give you better *latency*. For the same chat message, 4096 produces 50t/s while rtr and ub1024 produces 100t/s despite a slower overall bench speed.

Another funny thing.. at 4096 batch size, model feels more intelligent than rtr/1024.. but then rtr/512 it "feels" smarter again. I'll have to try 2048. It's tough to balance t/s vs pp t/s and latency and takes me many tries with layers.

---

👤 **trilog-inc** commented on **2025-08-08** at **20:33:11**

When i removed -b 4096 -ub 4096 ( defaults to 2048 then), i was able to process very large prompts. 112K tokens was the most I tried, and it was generating coherently but crawling. I can't say for anything related to it being smarter however. This context was built up during a Cline task where multiple turns were used during the plan phase and it was in the act phase where it was implementing the code changes, without errors i may add.

---

👤 **Ph0rk0z** commented on **2025-08-08** at **21:03:48**

Using 2048/2048 for me has the same PP speed as using RTR and ub 1024. Microbatch of 512 is maybe 10/s slower on PP and a few fractions off generation t/s. RTR does raise the output t/s by 1 or 2 though. 

Only 4096 without RTR doubles my prompt processing into the 180s, but as I found out, during actual chat it increases the latency. I played with ggml minimum batch to offload calculations too and could only get slightly better. Probably the best I'm going to get.

---

👤 **Thireus** commented on **2025-08-08** at **21:43:14**

Just to confirm, 128K context size isn't a problem as long as you have the VRAM for it.

```
# GLM-4.5
CUDA_DEVICE_ORDER=PCI_BUS_ID CUDA_VISIBLE_DEVICES=0,1,2 ~/ik_llama-main-b4065-a09bed8-bin-win-cuda-12.8-x64-avx512/llama-server -m GLM-4.5-THIREUS-BF16-SPECIAL_TENSOR-00001-of-01762.gguf -fa \
  -amb 1024 \
  -fmoe \
  -ctk f16 \
  -c 131072 \
  -ngl 99 \
  -ot "blk\.([0-9]|[1-3][0-9]|4[0-2])\.ffn_.*=CUDA0" -ot "blk\.(4[3-9]|[5-7][0-9])\.ffn_.*=CUDA1" -ot "blk\.(8[0-9]|90|91|92)\.ffn_.*=CUDA2" \
  -b 16384 -ub 4096 \
  --warmup-batch \
  --no-mmap \
  --threads 36 \
  --main-gpu 0
```

Getting 17.1t/s gen speed on small prompts.

On large prompt I get this:
```
Prompt
- Tokens: 103858
- Time: 277706.878 ms
- Speed: 374.0 t/s
Generation
- Tokens: 835
- Time: 306732.479 ms
- Speed: 2.7 t/s
```

Sad that the gen speed drops so much. :( But haven't tested to properly tweak the batch params yet.

llama-sweep on [GLM-4.5.ROOT-3.1602bpw-3.4340ppl.130GB-GGUF_10GB-GPU_120GB-CPU.a02563d_ea909f4.recipe](GLM-4.5.ROOT-3.1602bpw-3.4340ppl.130GB-GGUF_10GB-GPU_120GB-CPU.a02563d_ea909f4.recipe)
```
CUDA_DEVICE_ORDER=PCI_BUS_ID CUDA_VISIBLE_DEVICES=0,1,2 ~/ik_llama-main-b4065-a09bed8-bin-win-cuda-12.8-x64-avx512/llama-sweep-bench -m GLM-4.5-THIREUS-BF16-SPECIAL_TENSOR-00001-of-01762.gguf -fa \
  -amb 1024 \
  -fmoe \
  -ctk f16 \
  -c 131072 \
  -ngl 99 \
  -ot "blk\.([0-9]|[1-3][0-9]|4[0-2])\.ffn_.*=CUDA0" -ot "blk\.(4[3-9]|[5-7][0-9])\.ffn_.*=CUDA1" -ot "blk\.(8[0-9]|90|91|92)\.ffn_.*=CUDA2" \
  -b 16384 -ub 4096 \
  --warmup-batch \
  --no-mmap \
  --threads 36 \
  --main-gpu 0
  ...
|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  4096 |   1024 |      0 |    4.585 |   893.26 |   59.539 |    17.20 |
|  4096 |   1024 |   4096 |    4.922 |   832.23 |   63.364 |    16.16 |
|  4096 |   1024 |   8192 |    5.527 |   741.05 |   68.314 |    14.99 |
|  4096 |   1024 |  12288 |    5.900 |   694.27 |   72.943 |    14.04 |
|  4096 |   1024 |  16384 |    6.439 |   636.13 |   77.203 |    13.26 |
|  4096 |   1024 |  20480 |    6.924 |   591.58 |   81.941 |    12.50 |
|  4096 |   1024 |  24576 |    7.450 |   549.81 |   86.816 |    11.80 |
|  4096 |   1024 |  28672 |    8.102 |   505.56 |   93.576 |    10.94 |
|  4096 |   1024 |  32768 |    8.670 |   472.42 |  105.365 |     9.72 |
|  4096 |   1024 |  36864 |    9.314 |   439.78 |  121.616 |     8.42 |
|  4096 |   1024 |  40960 |    9.749 |   420.16 |  136.792 |     7.49 |
|  4096 |   1024 |  45056 |   10.312 |   397.21 |  164.048 |     6.24 |
|  4096 |   1024 |  49152 |   10.799 |   379.29 |  209.441 |     4.89 |
|  4096 |   1024 |  53248 |   11.441 |   358.01 |  222.971 |     4.59 |
|  4096 |   1024 |  57344 |   12.041 |   340.18 |  240.474 |     4.26 |
|  4096 |   1024 |  61440 |   12.609 |   324.84 |  255.593 |     4.01 |
|  4096 |   1024 |  65536 |   13.010 |   314.85 |  269.840 |     3.79 |
|  4096 |   1024 |  69632 |   13.702 |   298.94 |  283.598 |     3.61 |
|  4096 |   1024 |  73728 |   14.311 |   286.22 |  296.909 |     3.45 |
|  4096 |   1024 |  77824 |   14.825 |   276.29 |  310.117 |     3.30 |
|  4096 |   1024 |  81920 |   15.660 |   261.55 |  323.660 |     3.16 |
|  4096 |   1024 |  86016 |   16.116 |   254.15 |  330.415 |     3.10 |
|  4096 |   1024 |  90112 |   16.626 |   246.37 |  341.227 |     3.00 |
|  4096 |   1024 |  94208 |   17.405 |   235.33 |  351.375 |     2.91 |
|  4096 |   1024 |  98304 |   18.307 |   223.74 |  361.095 |     2.84 |
|  4096 |   1024 | 102400 |   18.758 |   218.36 |  375.655 |     2.73 |
|  4096 |   1024 | 106496 |   19.434 |   210.76 |  390.398 |     2.62 |
|  4096 |   1024 | 110592 |   19.938 |   205.44 |  408.206 |     2.51 |
|  4096 |   1024 | 114688 |   20.735 |   197.54 |  409.481 |     2.50 |
|  4096 |   1024 | 118784 |   21.345 |   191.89 |  414.118 |     2.47 |
|  4096 |   1024 | 122880 |   22.245 |   184.13 |  428.125 |     2.39 |
|  4096 |   1024 | 126976 |   22.811 |   179.57 |  441.632 |     2.32 |
```

---

👤 **TheLegendOfKitty** commented on **2025-08-08** at **22:11:33**

> Just to confirm, 128K context size isn't a problem as long as you have the VRAM for it.
> 
> 
> 
> ```
> 
> # GLM-4.5
> 
> CUDA_DEVICE_ORDER=PCI_BUS_ID CUDA_VISIBLE_DEVICES=0,1,2 ~/ik_llama-main-b4065-a09bed8-bin-win-cuda-12.8-x64-avx512/llama-server -m GLM-4.5-THIREUS-BF16-SPECIAL_TENSOR-00001-of-01762.gguf -fa \
> 
>   -amb 1024 \
> 
>   -fmoe \
> 
>   -ctk f16 \
> 
>   -c 131072 \
> 
>   -ngl 99 \
> 
>   -ot "blk\.([0-9]|[1-3][0-9]|4[0-2])\.ffn_.*=CUDA0" -ot "blk\.(4[3-9]|[5-7][0-9])\.ffn_.*=CUDA1" -ot "blk\.(8[0-9]|90|91|92)\.ffn_.*=CUDA2" \
> 
>   -b 16384 -ub 4096 \
> 
>   --warmup-batch \
> 
>   --no-mmap \
> 
>   --threads 36 \
> 
>   --main-gpu 0
> 
> ```
> 
> 
> 
> Getting 17.1t/s gen speed on small prompts.
> 
> 
> 
> On large prompt I get this:
> 
> ```
> 
> Prompt
> 
> - Tokens: 103858
> 
> - Time: 277706.878 ms
> 
> - Speed: 374.0 t/s
> 
> Generation
> 
> - Tokens: 835
> 
> - Time: 306732.479 ms
> 
> - Speed: 2.7 t/s
> 
> ```
> 
> 
> 
> Sad that the gen speed drops so much. :( But haven't tested to properly tweak the batch params yet.
> 
> 
> 
> llama-sweep on [GLM-4.5.ROOT-3.1602bpw-3.4340ppl.130GB-GGUF_10GB-GPU_120GB-CPU.a02563d_ea909f4.recipe](GLM-4.5.ROOT-3.1602bpw-3.4340ppl.130GB-GGUF_10GB-GPU_120GB-CPU.a02563d_ea909f4.recipe)
> 
> ```
> 
> CUDA_DEVICE_ORDER=PCI_BUS_ID CUDA_VISIBLE_DEVICES=0,1,2 ~/ik_llama-main-b4065-a09bed8-bin-win-cuda-12.8-x64-avx512/llama-sweep-bench -m GLM-4.5-THIREUS-BF16-SPECIAL_TENSOR-00001-of-01762.gguf -fa \
> 
>   -amb 1024 \
> 
>   -fmoe \
> 
>   -ctk f16 \
> 
>   -c 131072 \
> 
>   -ngl 99 \
> 
>   -ot "blk\.([0-9]|[1-3][0-9]|4[0-2])\.ffn_.*=CUDA0" -ot "blk\.(4[3-9]|[5-7][0-9])\.ffn_.*=CUDA1" -ot "blk\.(8[0-9]|90|91|92)\.ffn_.*=CUDA2" \
> 
>   -b 16384 -ub 4096 \
> 
>   --warmup-batch \
> 
>   --no-mmap \
> 
>   --threads 36 \
> 
>   --main-gpu 0
> 
>   ...
> 
> |    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
> 
> |-------|--------|--------|----------|----------|----------|----------|
> 
> |  4096 |   1024 |      0 |    4.585 |   893.26 |   59.539 |    17.20 |
> 
> |  4096 |   1024 |   4096 |    4.922 |   832.23 |   63.364 |    16.16 |
> 
> |  4096 |   1024 |   8192 |    5.527 |   741.05 |   68.314 |    14.99 |
> 
> |  4096 |   1024 |  12288 |    5.900 |   694.27 |   72.943 |    14.04 |
> 
> |  4096 |   1024 |  16384 |    6.439 |   636.13 |   77.203 |    13.26 |
> 
> |  4096 |   1024 |  20480 |    6.924 |   591.58 |   81.941 |    12.50 |
> 
> |  4096 |   1024 |  24576 |    7.450 |   549.81 |   86.816 |    11.80 |
> 
> |  4096 |   1024 |  28672 |    8.102 |   505.56 |   93.576 |    10.94 |
> 
> |  4096 |   1024 |  32768 |    8.670 |   472.42 |  105.365 |     9.72 |
> 
> |  4096 |   1024 |  36864 |    9.314 |   439.78 |  121.616 |     8.42 |
> 
> |  4096 |   1024 |  40960 |    9.749 |   420.16 |  136.792 |     7.49 |
> 
> ```

What are your specs?

I suspect speculative decoding would greatly benefit the performance of this model, but support is limited to SGLang.

---

👤 **Thireus** commented on **2025-08-08** at **22:15:44**

The worst possible build you can imagine:

- 2x RTX PRO 6000 Blackwell Workstation Edition + 1x 5090
- Running on Intel i9-7980xe with 32GB of DDR4 and PCIe-3 motherboard (x16, x16, x8) 😂

The frankenrig like @ubergarm would say.

---

👤 **espen96** commented on **2025-08-08** at **22:19:00**

> The worst possible build you can imagine:
> 
> * 2x RTX PRO 6000 Blackwell Workstation Edition + 1x 5090
> * Running on Intel i9-7980xe with 32GB of DDR4 and PCIe-3 motherboard (x16, x16, x8) 😂
> 
> The frankenrig like @ubergarm would say.

And I thought my rig was a bit funny. That's quite the unexpected rig you got there

---

👤 **Thireus** commented on **2025-08-08** at **22:19:51**

> And I thought my rig was a bit funny. That's quite the unexpected rig you got there

Bottlenecks all the way - the story of my life.

---

👤 **trilog-inc** commented on **2025-08-08** at **22:22:37**

> The worst possible build you can imagine:
> 
>     * 2x RTX PRO 6000 Blackwell Workstation Edition + 1x 5090
> 
>     * Running on Intel i9-7980xe with 32GB of DDR4 and PCIe-3 motherboard (x16, x16, x8) 😂
> 
> 
> The frankenrig like @ubergarm would say.

That's awesome. 
Im running 3x 3090 and 1x4090 with 512DDR5 on a Xeon workstation. Any tips on what parameters to tune to optimize PP?

---

👤 **Thireus** commented on **2025-08-08** at **22:27:22**

> That's awesome. Im running 3x 3090 and 1x4090 with 512DDR5 on a Xeon workstation. Any tips on what parameters to tune to optimize PP?

`-amb 1024` and `-b 16384 -ub 8192` used to work quite well for Intel on DeepSeek-R1, but for AMD people not so much apparently. But here on this model quite frankly I'm not too sure, as I haven't witnessed huge gains by cranking these parameters up. They also cost a lot of VRAM, so unless you have spare VRAM you shouldn't really bump them especially if the gain isn't noticeable. Best to try to fit as many layers into your GPUs (and use smaller quant) I would say.

---

👤 **TheLegendOfKitty** commented on **2025-08-08** at **22:28:53**

> The worst possible build you can imagine:
> 
> * 2x RTX PRO 6000 Blackwell Workstation Edition + 1x 5090
> * Running on Intel i9-7980xe with 32GB of DDR4 and PCIe-3 motherboard (x16, x16, x8) 😂
> 
> The frankenrig like @ubergarm would say.

Why are you splitting throughout all the GPUs? Can't you fit IQ4 on one GPU? Would be interesting to see that instead.

---

👤 **Thireus** commented on **2025-08-08** at **22:31:41**

> Why are you splitting throughout all the GPUs? Can't you fit IQ4 on one GPU? Would be interesting to see that instead.

This is GLM-4.5, not GLM-4.5-Air (which fits nicely on one GPU).
I wish I had 200+GB of VRAM on one GPU though.

---

👤 **TheLegendOfKitty** commented on **2025-08-08** at **22:33:37**

> > Why are you splitting throughout all the GPUs? Can't you fit IQ4 on one GPU? Would be interesting to see that instead.
> 
> This is GLM-4.5, not Air. But, I wish I had 200+GB of VRAM on one GPU.

What does your performance with Air on one GPU look like? Also, if you're doing full GPU inference exllamav3 would probably be faster. They merged support for GLM 4.5 but it's slower for me because I'm on 3x 3090 and it's not optimized for Ampere.

---

👤 **trilog-inc** commented on **2025-08-08** at **22:34:55**

> > That's awesome. Im running 3x 3090 and 1x4090 with 512DDR5 on a Xeon workstation. Any tips on what parameters to tune to optimize PP?
> 
> `-amb 1024` and `-b 16384 -ub 8192` used to work quite well for Intel on DeepSeek-R1, but for AMD people not so much apparently. But here on this model quite frankly I'm not too sure, as I haven't witnessed huge gains by cranking these parameters up. They also cost a lot of VRAM, so unless you have spare VRAM you shouldn't really bump them especially if the gain isn't noticeable. Best to try to fit as many layers into your GPUs (and use smaller quant) I would say.

Noted. Will give these a shot and report back 👍

---

👤 **ubergarm** commented on **2025-08-08** at **22:35:18**

i'm afk tonight but quick tip -amb only applies to MLA buffers psure so no
effect except on deepseek and kimi-k2

loving all the franken-rigs running franken-quants with franken-argument
commands 😆

---

👤 **Thireus** commented on **2025-08-08** at **22:38:44**

> What does your performance with Air on one GPU look like? Also, if you're doing full GPU inference exllamav3 would probably be faster. They merged support for GLM 4.5 but it's slower for me because I'm on 3x 3090 and it's not optimized for Ampere.

It was uber fast. But can't recall the numbers now.
I used to be a big fan of exllama-v1/v2 and decided to explore other projects after I discovered that RoPE wasn't working as expected (noticeable loss of context for 100k+ context sizes), also loading models was painfully slow. I would go back to visiting the project if things get resolved. I'm also thinking about exploring vllm (again).

---

👤 **Ph0rk0z** commented on **2025-08-08** at **23:03:57**

With 4x3090 I get about 29t/s on air exl3 4.0bpw. They have TP in the dev branch now. I'm sure the speed would go up for larger batches.

My GLM speeds not that great though.

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  1024 |    256 |      0 |    9.052 |   113.12 |   19.599 |    13.06 |
|  1024 |    256 |   1024 |    8.997 |   113.81 |   20.399 |    12.55 |
|  1024 |    256 |   2048 |    9.045 |   113.21 |   21.475 |    11.92 |
|  1024 |    256 |   3072 |    9.277 |   110.39 |   22.506 |    11.37 |
|  1024 |    256 |   4096 |    9.184 |   111.50 |   24.613 |    10.40 |
|  1024 |    256 |   5120 |    9.250 |   110.70 |   25.270 |    10.13 |
|  1024 |    256 |   6144 |    9.343 |   109.60 |   26.887 |     9.52 |



By 32k it drops off to like 5t/s

---

👤 **TheLegendOfKitty** commented on **2025-08-08** at **23:16:02**

> > What does your performance with Air on one GPU look like? Also, if you're doing full GPU inference exllamav3 would probably be faster. They merged support for GLM 4.5 but it's slower for me because I'm on 3x 3090 and it's not optimized for Ampere.
> 
> It was uber fast. But can't recall the numbers now. I used to be a big fan of exllama-v1/v2 and decided to explore other projects after I discovered that RoPE wasn't working as expected (noticeable loss of context for 100k+ context sizes), also loading models was painfully slow. I would go back to visiting the project if things get resolved. I'm also thinking about exploring vllm (again).

You can also try SGLang for functional multi-token prediction.

---

👤 **Thireus** commented on **2025-08-09** at **01:09:09**

llama-sweep on GLM-4.5-Air - [GLM-4.5-Air.ROOT-3.9070bpw-4.6903ppl.50GB-GGUF_5GB-GPU_45GB-CPU.a02563d_270bea7.recipe](https://github.com/Thireus/GGUF-Tool-Suite/blob/main/recipe_examples/GLM-4.5-Air.ROOT-3.9070bpw-4.6903ppl.50GB-GGUF_5GB-GPU_45GB-CPU.a02563d_270bea7.recipe)

Full 128K context fits in one RTX PRO 6000 Blackwell.

```
CUDA_DEVICE_ORDER=PCI_BUS_ID CUDA_VISIBLE_DEVICES=0 ~/ik_llama-main-b4065-a09bed8-bin-win-cuda-12.8-x64-avx512/llama-sweep-bench -m GLM-4.5-Air-THIREUS-BF16-SPECIAL_TENSOR-00001-of-00804.gguf -fa \
  -fmoe \
  -ctk f16 \
  -c 131072 \
  -ngl 99 \
  -b 16384 -ub 16384 \
  --warmup-batch \
  --no-mmap \
  --threads 36 \
  --main-gpu 0
  ...
|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
| 16384 |   4096 |      0 |    8.654 |  1893.28 |  101.798 |    40.24 |
| 16384 |   4096 |  16384 |   14.274 |  1147.80 |  134.539 |    30.44 |
| 16384 |   4096 |  32768 |   19.802 |   827.39 |  182.594 |    22.43 |
| 16384 |   4096 |  49152 |   25.308 |   647.38 |  384.922 |    10.64 |
| 16384 |   4096 |  65536 |   30.683 |   533.97 |  491.501 |     8.33 |
| 16384 |   4096 |  81920 |   36.679 |   446.68 |  592.425 |     6.91 |
| 16384 |   4096 |  98304 |   42.123 |   388.95 |  689.477 |     5.94 |
ggml_cuda_compute_forward: CPY failed
CUDA error: invalid configuration argument
  current device: 0, in function ggml_cuda_compute_forward at D:\a\ik_llama.cpp\ik_llama.cpp\ggml\src\ggml-cuda.cu:3007
  err
D:\a\ik_llama.cpp\ik_llama.cpp\ggml\src\ggml-cuda.cu:110: CUDA error
```

Ah! Looks like I need to reduce these batch numbers a little...

---

👤 **Thireus** commented on **2025-08-09** at **09:13:04**

Batch benchmarks for GLM-4.5-Air - [GLM-4.5-Air.ROOT-3.9070bpw-4.6903ppl.50GB-GGUF_5GB-GPU_45GB-CPU.a02563d_270bea7.recipe](https://github.com/Thireus/GGUF-Tool-Suite/blob/main/recipe_examples/GLM-4.5-Air.ROOT-3.9070bpw-4.6903ppl.50GB-GGUF_5GB-GPU_45GB-CPU.a02563d_270bea7.recipe):

![GLM-4.5-Air_spp.png](https://github.com/Thireus/GGUF-Tool-Suite/blob/main/ts_graphs//GLM-4.5-Air_spp.png?raw=true)
![GLM-4.5-Air_stg.png](https://github.com/Thireus/GGUF-Tool-Suite/blob/main/ts_graphs//GLM-4.5-Air_stg.png?raw=true)

```
Metric meanings:
  PP        Number of prompt tokens processed before generation
  TG        Number of tokens generated in generation phase
  N_KV      Size of the KV-cache in tokens (lookback window)
  T_PP s    Time taken for prompt processing (in seconds)
  S_PP t/s  Throughput for prompt processing (tokens per second)
  T_TG s    Time taken for generation phase (in seconds)
  S_TG t/s  Throughput for token generation (tokens per second)

Cutoff for means (common range used): 0 .. 61440 (shortest max(N_KV) among eligible series)
Interpolation grid: 300 points over 0..61440
Only series with max(N_KV) >= 60000 were considered.
Selection margins: S_PP margin = 2.5% ; S_TG margin = 0.5%

S_TG t/s means (best -> worst):
  -b 16384 -ub 1024 (model: GLM-4.5-Air) (b=16384, ub=1024) -> S_TG mean = 23.5769 over 300 grid points
  -b 1024 -ub 1024 (model: GLM-4.5-Air) (b=1024, ub=1024) -> S_TG mean = 23.4617 over 300 grid points <-- BEST
  -b 2048 -ub 1024 (model: GLM-4.5-Air) (b=2048, ub=1024) -> S_TG mean = 23.4591 over 300 grid points
  -b 4096 -ub 1024 (model: GLM-4.5-Air) (b=4096, ub=1024) -> S_TG mean = 23.4252 over 300 grid points
  -b 8192 -ub 1024 (model: GLM-4.5-Air) (b=8192, ub=1024) -> S_TG mean = 23.3973 over 300 grid points
  -b 8192 -ub 512 (model: GLM-4.5-Air) (b=8192, ub=512) -> S_TG mean = 23.3783 over 300 grid points
  -b 8192 -ub 4096 (model: GLM-4.5-Air) (b=8192, ub=4096) -> S_TG mean = 23.3715 over 300 grid points
  -b 2048 -ub 2048 (model: GLM-4.5-Air) (b=2048, ub=2048) -> S_TG mean = 23.3271 over 300 grid points
  -b 4096 -ub 4096 (model: GLM-4.5-Air) (b=4096, ub=4096) -> S_TG mean = 23.3263 over 300 grid points
  -b 2048 -ub 512 (model: GLM-4.5-Air) (b=2048, ub=512) -> S_TG mean = 23.3022 over 300 grid points
  -b 4096 -ub 2048 (model: GLM-4.5-Air) (b=4096, ub=2048) -> S_TG mean = 23.2854 over 300 grid points
  -b 4096 -ub 512 (model: GLM-4.5-Air) (b=4096, ub=512) -> S_TG mean = 23.206 over 300 grid points
  -b 1024 -ub 512 (model: GLM-4.5-Air) (b=1024, ub=512) -> S_TG mean = 23.1303 over 300 grid points
  -b 512 -ub 512 (model: GLM-4.5-Air) (b=512, ub=512) -> S_TG mean = 23.02 over 300 grid points

S_PP t/s means (best -> worst):
  -b 8192 -ub 4096 (model: GLM-4.5-Air) (b=8192, ub=4096) -> S_PP mean = 1224.72 over 300 grid points
  -b 4096 -ub 4096 (model: GLM-4.5-Air) (b=4096, ub=4096) -> S_PP mean = 1220.26 over 300 grid points <-- BEST
  -b 2048 -ub 2048 (model: GLM-4.5-Air) (b=2048, ub=2048) -> S_PP mean = 1042.54 over 300 grid points
  -b 4096 -ub 2048 (model: GLM-4.5-Air) (b=4096, ub=2048) -> S_PP mean = 1037.23 over 300 grid points
  -b 1024 -ub 1024 (model: GLM-4.5-Air) (b=1024, ub=1024) -> S_PP mean = 844.421 over 300 grid points
  -b 16384 -ub 1024 (model: GLM-4.5-Air) (b=16384, ub=1024) -> S_PP mean = 844.381 over 300 grid points
  -b 2048 -ub 1024 (model: GLM-4.5-Air) (b=2048, ub=1024) -> S_PP mean = 844.174 over 300 grid points
  -b 4096 -ub 1024 (model: GLM-4.5-Air) (b=4096, ub=1024) -> S_PP mean = 841.961 over 300 grid points
  -b 8192 -ub 1024 (model: GLM-4.5-Air) (b=8192, ub=1024) -> S_PP mean = 840.998 over 300 grid points
  -b 8192 -ub 512 (model: GLM-4.5-Air) (b=8192, ub=512) -> S_PP mean = 588.077 over 300 grid points
  -b 2048 -ub 512 (model: GLM-4.5-Air) (b=2048, ub=512) -> S_PP mean = 586.643 over 300 grid points
  -b 4096 -ub 512 (model: GLM-4.5-Air) (b=4096, ub=512) -> S_PP mean = 584.933 over 300 grid points
  -b 1024 -ub 512 (model: GLM-4.5-Air) (b=1024, ub=512) -> S_PP mean = 584.238 over 300 grid points
  -b 512 -ub 512 (model: GLM-4.5-Air) (b=512, ub=512) -> S_PP mean = 582.34 over 300 grid points
```

Batch benchmarks for GLM-4.5 - [GLM-4.5.ROOT-3.1602bpw-3.4340ppl.130GB-GGUF_10GB-GPU_120GB-CPU.a02563d_ea909f4.recipe](https://github.com/Thireus/GGUF-Tool-Suite/blob/main/recipe_examples/GLM-4.5.ROOT-3.1602bpw-3.4340ppl.130GB-GGUF_10GB-GPU_120GB-CPU.a02563d_ea909f4.recipe):

![GLM-4.5_spp.png](https://github.com/Thireus/GGUF-Tool-Suite/blob/main/ts_graphs//GLM-4.5_spp.png?raw=true)
![GLM-4.5_stg.png](https://github.com/Thireus/GGUF-Tool-Suite/blob/main/ts_graphs//GLM-4.5_stg.png?raw=true)

```
Metric meanings:
  PP        Number of prompt tokens processed before generation
  TG        Number of tokens generated in generation phase
  N_KV      Size of the KV-cache in tokens (lookback window)
  T_PP s    Time taken for prompt processing (in seconds)
  S_PP t/s  Throughput for prompt processing (tokens per second)
  T_TG s    Time taken for generation phase (in seconds)
  S_TG t/s  Throughput for token generation (tokens per second)

Cutoff for means (common range used): 0 .. 61440 (shortest max(N_KV) among eligible series)
Interpolation grid: 300 points over 0..61440
Only series with max(N_KV) >= 60000 were considered.
Selection margins: S_PP margin = 2.5% ; S_TG margin = 0.5%

S_TG t/s means (best -> worst):
  -b 512 -ub 512 (model: GLM-4.5) (b=512, ub=512) -> S_TG mean = 10.1677 over 300 grid points <-- BEST
  -b 1024 -ub 512 (model: GLM-4.5) (b=1024, ub=512) -> S_TG mean = 10.1597 over 300 grid points
  -b 16384 -ub 512 (model: GLM-4.5) (b=16384, ub=512) -> S_TG mean = 10.1569 over 300 grid points
  -b 2048 -ub 512 (model: GLM-4.5) (b=2048, ub=512) -> S_TG mean = 10.149 over 300 grid points
  -b 8192 -ub 512 (model: GLM-4.5) (b=8192, ub=512) -> S_TG mean = 10.1479 over 300 grid points
  -b 4096 -ub 512 (model: GLM-4.5) (b=4096, ub=512) -> S_TG mean = 10.1383 over 300 grid points
  -b 8192 -ub 1024 (model: GLM-4.5) (b=8192, ub=1024) -> S_TG mean = 10.1297 over 300 grid points
  -b 16384 -ub 1024 (model: GLM-4.5) (b=16384, ub=1024) -> S_TG mean = 10.1262 over 300 grid points
  -b 1024 -ub 1024 (model: GLM-4.5) (b=1024, ub=1024) -> S_TG mean = 10.1153 over 300 grid points
  -b 4096 -ub 1024 (model: GLM-4.5) (b=4096, ub=1024) -> S_TG mean = 10.1099 over 300 grid points
  -b 2048 -ub 1024 (model: GLM-4.5) (b=2048, ub=1024) -> S_TG mean = 10.1095 over 300 grid points
  -b 8192 -ub 2048 (model: GLM-4.5) (b=8192, ub=2048) -> S_TG mean = 10.089 over 300 grid points
  -b 16384 -ub 2048 (model: GLM-4.5) (b=16384, ub=2048) -> S_TG mean = 10.0784 over 300 grid points
  -b 2048 -ub 2048 (model: GLM-4.5) (b=2048, ub=2048) -> S_TG mean = 10.0752 over 300 grid points
  -b 4096 -ub 2048 (model: GLM-4.5) (b=4096, ub=2048) -> S_TG mean = 10.0667 over 300 grid points
  -b 8192 -ub 4096 (model: GLM-4.5) (b=8192, ub=4096) -> S_TG mean = 10.0171 over 300 grid points
  -b 16384 -ub 4096 (model: GLM-4.5) (b=16384, ub=4096) -> S_TG mean = 10.0095 over 300 grid points
  -b 4096 -ub 4096 (model: GLM-4.5) (b=4096, ub=4096) -> S_TG mean = 10.0003 over 300 grid points

S_PP t/s means (best -> worst):
  -b 16384 -ub 4096 (model: GLM-4.5) (b=16384, ub=4096) -> S_PP mean = 532.96 over 300 grid points
  -b 8192 -ub 4096 (model: GLM-4.5) (b=8192, ub=4096) -> S_PP mean = 531.683 over 300 grid points
  -b 4096 -ub 4096 (model: GLM-4.5) (b=4096, ub=4096) -> S_PP mean = 528.932 over 300 grid points <-- BEST
  -b 2048 -ub 2048 (model: GLM-4.5) (b=2048, ub=2048) -> S_PP mean = 452.073 over 300 grid points
  -b 8192 -ub 2048 (model: GLM-4.5) (b=8192, ub=2048) -> S_PP mean = 451.692 over 300 grid points
  -b 16384 -ub 2048 (model: GLM-4.5) (b=16384, ub=2048) -> S_PP mean = 451.139 over 300 grid points
  -b 4096 -ub 2048 (model: GLM-4.5) (b=4096, ub=2048) -> S_PP mean = 450.91 over 300 grid points
  -b 8192 -ub 1024 (model: GLM-4.5) (b=8192, ub=1024) -> S_PP mean = 339.944 over 300 grid points
  -b 16384 -ub 1024 (model: GLM-4.5) (b=16384, ub=1024) -> S_PP mean = 339.427 over 300 grid points
  -b 4096 -ub 1024 (model: GLM-4.5) (b=4096, ub=1024) -> S_PP mean = 339.425 over 300 grid points
  -b 2048 -ub 1024 (model: GLM-4.5) (b=2048, ub=1024) -> S_PP mean = 338.671 over 300 grid points
  -b 1024 -ub 1024 (model: GLM-4.5) (b=1024, ub=1024) -> S_PP mean = 338.496 over 300 grid points
  -b 16384 -ub 512 (model: GLM-4.5) (b=16384, ub=512) -> S_PP mean = 241.359 over 300 grid points
  -b 2048 -ub 512 (model: GLM-4.5) (b=2048, ub=512) -> S_PP mean = 241.344 over 300 grid points
  -b 8192 -ub 512 (model: GLM-4.5) (b=8192, ub=512) -> S_PP mean = 241.158 over 300 grid points
  -b 1024 -ub 512 (model: GLM-4.5) (b=1024, ub=512) -> S_PP mean = 240.458 over 300 grid points
  -b 4096 -ub 512 (model: GLM-4.5) (b=4096, ub=512) -> S_PP mean = 240.145 over 300 grid points
  -b 512 -ub 512 (model: GLM-4.5) (b=512, ub=512) -> S_PP mean = 239.895 over 300 grid points
```

I'll update the graphs as benchmarking progresses.

---

👤 **espen96** commented on **2025-08-09** at **11:30:08**

> I'm running batch benchmarks for GLM-4.5-Air - [GLM-4.5.ROOT-3.1602bpw-3.4340ppl.130GB-GGUF_10GB-GPU_120GB-CPU.a02563d_ea909f4.recipe](https://github.com/ikawrakow/ik_llama.cpp/pull/GLM-4.5.ROOT-3.1602bpw-3.4340ppl.130GB-GGUF_10GB-GPU_120GB-CPU.a02563d_ea909f4.recipe). This is what I have so far:
> 
> ![GLM-4.5-Air_spp.png](https://camo.githubusercontent.com/0c82632aad84702aeb825ead2233d6a0a3b3c7f14d58dfd3c45592a6559193cd/68747470733a2f2f746869726575732e636f6d2f4749544855422f474c4d2d342e352d4169725f7370702e706e67) ![GLM-4.5-Air_stg.png](https://camo.githubusercontent.com/4fafcdf36b27529a685604e279029b1535289cddf44bf643674660ff90a6ebdb/68747470733a2f2f746869726575732e636f6d2f4749544855422f474c4d2d342e352d4169725f7374672e706e67)
> 
> I'll update the graphs when I get into the 16k batch size. But all seem to indicate so far that 4096/4096 is the right balance.

looking good! 

it's a bit messy right now, but it seems we can infer some options for various setups.

---

👤 **Thireus** commented on **2025-08-09** at **18:22:45**

> it's a bit messy right now, but it seems we can infer some options for various setups.

[Graphs updated](https://github.com/ikawrakow/ik_llama.cpp/pull/668#issuecomment-3170522512) with more clarity and also provided the logs that show what's the best combination.

Bottom line (at least for my specs), -b 4096 -ub 4096 is the best option. Which is reassuring because that's the combination that allows for decent room into VRAM.

```
#!/usr/bin/env bash

BATCH_SIZES=(512 1024 2048 4096 8192 16384)

CUDA_DEVICE_ORDER=PCI_BUS_ID
CUDA_VISIBLE_DEVICES=0,1,2
MODEL="GLM-4.5-THIREUS-BF16-SPECIAL_TENSOR-00001-of-01762.gguf"
BENCH_PATH="$HOME/ik_llama-main-b4065-a09bed8-bin-win-cuda-12.8-x64-avx512/llama-sweep-bench"

for i in "${BATCH_SIZES[@]}"; do
    for j in "${BATCH_SIZES[@]}"; do
        # Skip cases where i < j
        if (( i < j )); then
            continue
        fi

        OUTPUT_FILE="GLM-4.5_${i}_${j}.txt"

        "$BENCH_PATH" \
            -m "$MODEL" \
            -fa \
            -amb 1024 \
            -fmoe \
            -ctk f16 \
            -c 65536 \
            -ngl 99 \
            -ot "blk\.([0-9]|[1-3][0-9]|4[0-2])\.ffn_.*=CUDA0" \
            -ot "blk\.(4[3-9]|[5-7][0-9])\.ffn_.*=CUDA1" \
            -ot "blk\.(8[0-9]|90|91|92)\.ffn_.*=CUDA2" \
            -b "$i" \
            -ub "$j" \
            --warmup-batch \
            --no-mmap \
            --threads 36 \
            --main-gpu 0 \
            > "$OUTPUT_FILE" 2>&1
    done
done
```

Script to collect and plot the results: https://github.com/Thireus/GGUF-Tool-Suite/blob/main/plot_llama_sweep.py

---

👤 **Thireus** commented on **2025-08-10** at **06:07:05**

Sadly the model completely fails the [Dipiloblop](https://thireus.com/GITHUB/Dipiloblop.txt) prompt. On chat.z.ai it believes I'm asking about Runescape lol, it's clearly been training on giving that answer for similar-looking prompt. On ik_llama it invents numbers and is unable to find the correct ones from the XP table.

Answer from https://chat.z.ai:

<details>

```
Based on the **Skills/Experience Table** provided in the DipiloBlop WikiBook, the maximum level in any skill is **100**, which requires **13,034,431 XP**. Half of this maximum XP is **6,517,215.5 XP**.  

To determine the level at which you reach half the XP of the max level:  
- **Level 91** requires **6,507,243 XP**.  
- **Level 92** requires **7,505,874 XP**.  

**Level 91 (6,507,243 XP)** is the closest to half the max XP (6,517,215.5 XP), differing by only **9,972.5 XP** (less than 0.15% of the total XP needed for level 100). This makes level 91 the practical answer for "half the XP of max level."  

### Key Details from the Experience Table:
| Level | XP Required       | Notes                                  |
|-------|-------------------|----------------------------------------|
| 90    | 5,379,721         | Below half max XP.                     |
| **91**| **6,507,243**     | **~99.85% of half max XP (6,517,215.5)** |
| 92    | 7,505,874         | Exceeds half max XP.                   |
| 100   | 13,034,431        | Max level XP.                          |

### Why Level 91?
- The XP curve in DipiloBlop grows exponentially, meaning levels near 100 require disproportionately more XP. Level 91 sits just below the halfway point in XP terms, making it the threshold for "half the XP journey" to max level.  
- This is consistent across all skills, as the experience table is universal.  

### Practical Implication:
- Reaching **level 91** means you've completed roughly half the XP grind needed for level 100, even though you're 9 levels away from the cap. This highlights the steep XP scaling in later levels.
```

</details>

Answer from ik_llama.cpp:

<details>

```
<think>We are given a large text dump from the "Next Gen DipiloBlop" WikiBook, which includes various guides, skill descriptions, quest information, and more. The user asks: "In dipiloblop, what level are you at when you get to half the xp of max level?"

First, we need to understand what "max level" means. In the context of the game, the maximum level for any skill is level 100 (as mentioned in the Skills/Experience Table section). The experience required for level 100 is 13,034,431 (from the table: level 100 requires 13,034,431 experience).

Half of that experience would be: 13,034,431 / 2 = 6,517,215.5

Now, we need to find out at which level the cumulative experience reaches 6,517,215.5. We can look at the experience table provided:

The table lists the experience required for each level from 1 to 101. We are looking for the level at which the cumulative experience is at least 6,517,215.5.

Looking at the table:

- Level 90: 12,385,868
- Level 91: 14,037,935
- Level 92: 19,448,6167
... (Note: The table in the provided text has some formatting issues and missing entries, but we can try to interpolate or use the available data.)

However, the table provided in the text only goes up to level 101, but we have level 90 and 91. We see:

Level 90: 12,385,868
Level 91: 14,037,935

But 12,385,868 is already more than 6,517,215.5, so we need a lower level.

Let's check lower levels:

Level 85: 8,162,089
Level 86: 9,177,944
Level 87: 10,401,620
Level 88: 11,785,480
Level 89: 13,283,427

But wait, level 85 is 8,162,089 which is more than 6,517,215.5? Actually, 8 million is more than 6.5 million, so we need even lower.

Let me check the table again. The table in the text is:

\`\`\`
Level 	Experience needed to reach level 	Experience difference from last level
1	0	0
2	266	266
3	553	287
4	888	335
5	1,264	376
6	1,697	433
7	2,202	505
8	2,761	559
9	3,421	660
10	4,162	741
11	5,009	847
12	5,987	978
13	7,090	1,103
14	8,339	1,249
15	9,792	1,453
16	11,423	1,631
17	13,273	1,850
18	15,397	2,124
19	17,798	2,401
20	20,537	2,739
21	23,635	3,098
22	27,183	3,548
23	31,160	3,977
24	35,720	4,560
25	40,895	5,175
26	46,750	5,855
27	53,376	6,626
28	60,943	7,567
29	69,493	8,550
30	79,195	9,702
31	90,198	11,003
32	102,691	12,493
33	116,838	14,147
34	132,892	16,054
35	151,085	18,193
36	171,731	20,646
37	195,093	23,362
38	221,634	26,541
39	251,703	30,069
40	285,768	34,065
41	324,379	38,611
42	368,189	43,810
43	417,795	49,606
44	474,049	56,254
45	537,848	63,799
46	610,073	72,225
47	691,990	81,917
48	784,837	92,847
49	890,059	105,222
50	1,009,301	119,242
51	1,144,480	135,179
52	1,297,644	153,164
53	1,471,284	173,640
54	1,667,999	196,715
55	1,891,034	223,035
56	2,143,788	252,754
57	2,430,212	286,424
58	2,754,802	324,590
59	3,122,772	367,970
60	3,539,721	416,949
61	4,012,280	472,559
62	4,547,846	535,566
63	5,154,809	606,963
64	5,842,739	687,930
65	6,622,340	779,601
66	7,505,874	883,534
67	8,507,246	1,001,372
68	9,642,117	1,134,871
69	10,928,297	1,286,180
70	12,385,868	1,457,571
71	14,037,935	1,652,067
72	15,910,153	1,872,218
73	18,031,971	2,121,818
74	20,436,698	2,404,727
75	23,162,089	2,725,391
76	26,250,771	3,088,682
77	29,751,292	3,500,521
78	33,718,506	3,967,214
79	38,214,716	4,496,210
80	43,310,285	5,095,569
81	49,085,275	5,774,990
82	55,630,259	6,544,984
83	63,047,786	7,417,527
84	71,454,370	8,406,584
85	80,981,751	9,527,381
86	91,779,442	10,797,691
87	104,016,620	12,237,178
88	117,885,480	13,868,860
89	133,603,420	15,717,940
90	151,417,175	17,813,755
91	171,605,798	20,188,623
92	194,486,167	22,880,369
93	220,417,391	25,931,224
94	249,805,802	29,388,411
95	283,112,599	33,306,797
96	320,860,372	37,747,773
97	363,640,890	42,780,518
98	412,125,351	48,484,461
99	467,074,482	54,949,131
100	507,038,931	48,364,282
101	556,387,432	49,348,501
\`\`\`

We are looking for the level at which the cumulative experience is at least 6,517,215.5.

Looking at the table:

Level 80: 43,310,285
Level 81: 49,085,275
Level 82: 55,630,259
Level 83: 63,047,786
Level 84: 71,454,370
Level 85: 80,981,751

But wait, these numbers are in the millions? Actually, looking at the table, the numbers are in thousands? Let me check the pattern:

Level 1: 0
Level 2: 266
...
Level 99: 467,074,482
Level 100: 507,038,931

But 507 million for level 100? That seems too high. Let me check the original RuneScape experience table for comparison. In RuneScape, level 99 requires 13,034,431 experience. However, in this text, level 100 is 507,038,931? That doesn't match.

Looking back at the text, I see:

"Level 100: 507,038,931"

But then level 101: 556,387,432

And the note at the bottom: "Note: The experience table is from the original RuneScape, but the numbers are scaled up in DipiloBlop? Or perhaps it's a typo."

But in the provided text, the experience table might be incorrect? Alternatively, the numbers might be in a different unit.

Looking at the Skills/Experience Table section again:

\`\`\`
Skills/Experience Table

This table shows players how much experience they need to reach to get to the next level, as well as the experience difference from the last level they were at.
Level 	Experience needed to reach level 	Experience difference from last level
1	0	0
...
99	467,074,482	54,949,131
100	507,038,931	48,364,282
101	556,387,432	49,348,501
\`\`\`

This table seems to have very large numbers. However, in the context of the game, it might be that the experience required for level 100 is 507,038,931? Then half of that would be 253,519,465.5.

Now, we look for the level that requires at least 253,519,465.5 cumulative experience.

From the table:

Level 85: 80,981,751
Level 86: 91,779,442
Level 87: 104,016,620
Level 88: 117,885,480
Level 89: 133,603,420
Level 90: 151,417,175
Level 91: 171,605,798
Level 92: 194,486,167
Level 93: 220,417,391
Level 94: 249,805,802
Level 95: 283,112,599

We see that level 94 requires 249,805,802 and level 95 requires 283,112,599. Since 253,519,465.5 is between level 94 and 95, we can say that at level 94 you have not yet reached half the max experience, and at level 95 you have exceeded it.

But note: the experience at level 94 is 249,805,802, which is less than 253,519,465.5, and level 95 is 283,112,599 which is more.

Therefore, when you reach level 95, you have more than half the experience required for level 100.

But the user asked: "what level are you at when you get to half the xp of max level?"

This is a bit ambiguous. It could mean:

1. At what level do you have exactly half the experience of the max level? (But levels are discrete, so we look for the level at which the cumulative experience is closest to half of max.)
2. Or, at what level do you reach half the experience of the max level? (Which would be the level at which the cumulative experience is at least half of max.)

Given that, we can say:

- At level 94, you have 249,805,802 XP, which is less than half of 507,038,931 (which is 253,519,465.5).
- At level 95, you have 283,112,599 XP, which is more than half.

So, you reach half the XP of max level (level 100) when you achieve level 95.

But note: the experience table in the provided text might be incorrect because in standard RuneScape, level 99 is the max and requires 13,034,431 XP. However, in this text, level 100 is the max and requires 507,038,931 XP. We must go by the provided text.

Therefore, the answer is level 95.

But let me double-check the half: 507,038,931 / 2 = 253,519,465.5

Level 94: 249,805,802 < 253,519,465.5
Level 95: 283,112,599 > 253,519,465.5

So, you are at level 95 when you exceed half the XP of max level.

Alternatively, if the question means the level at which you have exactly half, then there is no exact level because the experience doesn't land exactly at half. But the level at which you first reach or exceed half is level 95.

Thus, the answer is level 95.</think>Based on the provided information from the "Next Gen DipiloBlop" WikiBook, the maximum level in any skill is level 100, which requires **507,038,931 experience points** (as per the Skills/Experience Table section).

Half of the maximum experience is calculated as:  
**507,038,931 ÷ 2 = 253,519,465.5 XP**.

To determine the level at which a player reaches or exceeds this half-max XP, we refer to the cumulative experience table:

- **Level 94** requires **249,805,802 XP** (less than 253,519,465.5 XP).  
- **Level 95** requires **283,112,599 XP** (greater than 253,519,465.5 XP).  

Thus, **you reach half the XP of the maximum level (level 100) when you achieve level 95**. At this point, you have exceeded 253,519,465.5 XP, placing you beyond the halfway mark to level 100.

### Key Notes:
- The experience values are sourced directly from the provided Skills/Experience Table in the WikiBook.  
- Level 100 is explicitly stated as the maximum (e.g., "Level 100: 507,038,931 XP").  
- The calculation confirms that level 95 is the threshold where cumulative XP surpasses half of the max XP.
```

</details>

I'll be reverting back to using DeepSeek-R1-0825 (which answers correctly) as GLM-4.5 isn't suitable for my use-case which involves processing large context prompts... Unless we have incorrectly implemented RoPE, which I doubt...

Actually, I might give a try to Qwen3 Thinking, since it is able to solve it at really good speed:

```
Prompt
- Tokens: 107655
- Time: 242465.555 ms
- Speed: 444.0 t/s
Generation
- Tokens: 2793
- Time: 189865.006 ms
- Speed: 14.7 t/s
```

<details>

```
<think>
To answer this question, I need to find out at what level a player reaches half of the total XP needed to get to level 101 (which appears to be the maximum level).

First, let me find the total XP needed to reach level 101 from the experience table provided in the text.

Looking at the "Skills/Experience Table" section, I can see a table showing the experience needed to reach each level. I need to find the entry for level 101.

From the table:
Level 101: 556,387,432 XP

So the total XP needed to reach level 101 is 556,387,432.

Half of this would be: 556,387,432 / 2 = 278,193,716 XP

Now I need to find at which level the cumulative XP reaches approximately 278,193,716.

Let me scan through the experience table to find this:

Looking at the table:
Level 90: 151,417,175 XP
Level 91: 171,605,798 XP
Level 92: 194,486,167 XP
Level 93: 220,417,391 XP
Level 94: 249,805,802 XP
Level 95: 283,112,599 XP

I see that level 94 requires 249,805,802 XP and level 95 requires 283,112,599 XP.

Since 278,193,716 is between these two values, the player would reach half the XP of max level somewhere between levels 94 and 95. More specifically, it would be at level 94, with some additional XP toward level 95.

To be more precise, I could calculate the exact point:

278,193,716 - 249,805,802 = 28,387,914 XP into level 95

The XP difference between level 94 and level 95 is:
283,112,599 - 249,805,802 = 33,306,797

So the player would be at approximately:
94 + (28,387,914 / 33,306,797) = 94 + 0.852 = level 94.852

So the player would be at approximately level 94.85 when they've earned half the XP needed to reach level 101.

Let me double-check my calculation:
Total XP to max level: 556,387,432
Half of that: 278,193,716

XP at level 94: 249,805,802
XP needed for level 95: 283,112,599

Yes, 278,193,716 is between these values, confirming my calculation.

Actually, let me check one more time to make sure I'm interpreting the table correctly. The table shows "Experience needed to reach level" which means the cumulative XP needed to reach that level from level 1.

So level 101 requires 556,387,432 XP total.
Half of that is 278,193,716.

Looking at the table again:
Level 94: 249,805,802 XP
Level 95: 283,112,599 XP

So at level 94, the player has 249,805,802 XP, which is less than half.
At level 95, the player has 283,112,599 XP, which is more than half.

Therefore, the player reaches half the XP at some point while leveling up to level 95, specifically when they have earned 278,193,716 XP.

To express this as a level:
Level = 94 + (278,193,716 - 249,805,802) / (283,112,599 - 249,805,802)
Level = 94 + 28,387,914 / 33,306,797
Level = 94 + 0.852
Level = 94.852

So the player is at approximately level 94.85 when they've earned half the XP needed to reach level 101.

Actually, let me check the level 100 and 101 values again to be absolutely sure:

Level 100: 507,038,931 XP
Level 101: 556,387,432 XP

So the total XP to reach level 101 is 556,387,432.
Half of that is 278,193,716.

Let me check the levels around there again:
Level 94: 249,805,802 XP
Level 95: 283,112,599 XP

Yes, 278,193,716 is between these two values.

Actually, I just noticed something. The table shows "Experience needed to reach level" which means the cumulative XP needed to reach that level from level 1.

So to reach level 101, you need 556,387,432 XP total.
Half of that is 278,193,716.

Looking at the table:
Level 94: 249,805,802 XP
Level 95: 283,112,599 XP

So the player reaches half the XP total somewhere while leveling up to level 95.

To be precise:
Level = 94 + (278,193,716 - 249,805,802) / (283,112,599 - 249,805,802)
Level = 94 + 28,387,914 / 33,306,797
Level = 94 + 0.852
Level = 94.852

So the player is at approximately level 94.85 when they've earned half the XP needed to reach level 101.

Wait, I need to double-check the level 101 value. Let me look at the table again:

From the text:
"Level 101	556,387,432	49,348,501"

Yes, so 556,387,432 is the total XP needed to reach level 101.

Half of that is 278,193,716.

Now, let's check the level where this happens:
Level 94: 249,805,802 XP
Level 95: 283,112,599 XP

So 278,193,716 is between these two values.

Therefore, the player is at approximately level 94.85 when they've earned half the XP needed to reach level 101.

Actually, let me do one more check to be 100% sure.

Total XP to level 101: 556,387,432
Half of that: 278,193,716

XP at level 94: 249,805,802
XP needed for level 95: 283,112,599

So the player reaches half the XP total when they're at:
94 + (278,193,716 - 249,805,802) / (283,112,599 - 249,805,802)
= 94 + 28,387,914 / 33,306,797
= 94 + 0.852
= 94.852

So the answer is approximately level 94.85.

Actually, I just noticed something else. The table shows "Experience needed to reach level" which means the cumulative XP needed to reach that level from level 1.

So level 101 requires 556,387,432 XP total.
Half of that is 278,193,716.

Looking at the table:
Level 94: 249,805,802 XP
Level 95: 283,112,599 XP

So 278,193,716 is between these two values.

To find the exact level:
Level = 94 + (278,193,716 - 249,805,802) / (283,112,599 - 249,805,802)
Level = 94 + 28,387,914 / 33,306,797
Level = 94 + 0.852
Level = 94.852

So the player is at approximately level 94.85 when they've earned half the XP needed to reach level 101.

Let me check if there's any chance I misread the level 101 value:

From the text:
"Level 101	556,387,432	49,348,501"

Yes, that's correct.

So my final answer is level 94.85 (approximately).
</think>

To determine at what level a player reaches half the XP of max level in DipiloBlop, I need to analyze the experience table provided in the Skills/Experience Table section.

According to the table, the total XP needed to reach level 101 (the maximum level) is 556,387,432 XP.

Half of this total would be: 556,387,432 ÷ 2 = 278,193,716 XP

Now I need to find which level corresponds to approximately 278,193,716 XP by examining the experience table:

- Level 94 requires 249,805,802 XP
- Level 95 requires 283,112,599 XP

Since 278,193,716 falls between these two values, the player reaches half the max XP somewhere between levels 94 and 95.

To calculate the exact point:
- XP needed to go from level 94 to 95: 283,112,599 - 249,805,802 = 33,306,797
- XP into level 95 when reaching half the max: 278,193,716 - 249,805,802 = 28,387,914
- Fraction of level 95 completed: 28,387,914 ÷ 33,306,797 = 0.852

Therefore, a player reaches half the XP of max level at approximately level 94.85.

This means when a player has earned about 278 million XP (out of the 556 million needed for level 101), they'll be at roughly level 94.85.
```

</details>

---

👤 **usrlocalben** commented on **2025-08-10** at **15:29:29**

@Thireus (and at the risk of going off topic)

Kimi K2 [IQ4_KS](https://huggingface.co/ubergarm/Kimi-K2-Instruct-GGUF) would be answer-correct if it didn't show its work. 😓

> To determine the level at which you've reached half the experience required for the maximum level in DipiloBlop, let's break it down using the provided data.
> 
> 1. **Maximum Level**: The highest level mentioned in the experience table is level 101, which requires **507,038,931 XP**.
> 2. **Half of Max XP**: Half of 507,038,931 XP is approximately **253,519,465 XP**.
> 3. **Finding the Level**: Looking at the experience table, the closest level that matches or exceeds 253,519,465 XP is **level 94**, which requires **249,805,802 XP**. The next level (95) requires **283,112,599 XP**, which is beyond half of the max XP.
> 
> Thus, you are at **level 94** when you've accumulated roughly half the experience needed for the maximum level (101).

IQ2_KL (ubergarm)😰

> To determine the level at which you have half the XP required for the maximum level (level 99) in DipiloBlop, we can refer to the **Skills/Experience Table** provided in the book. 
> 
> From the table, the total XP required to reach level 99 is **467,074,482**. Half of this XP would be **233,537,241**.
> 
> Looking at the table, the closest level that matches this XP is **level 88**, where the cumulative XP required is **233,537,241**. 
> 
> Thus, you are at **level 88** when you have half the XP needed to reach the maximum level.

Intel AutoRound Q2

> To determine the level at which you reach half the experience required for the maximum level (level 100) in DipiloBlop, let's analyze the provided experience table:
> 
> 1. **Maximum Level**: The highest level mentioned is 100, with a total experience requirement of **507,038,931 XP**.
> 2. **Half of Max XP**: Half of 507,038,931 XP is **253,519,465.5 XP**.
> 
> Looking at the experience table:
> - **Level 85** requires **80,981,751 XP**.
> - **Level 90** requires **151,417,175 XP**.
> - **Level 95** requires **283,112,599 XP**.
> 
> Since **253,519,465.5 XP** falls between **Level 95 (283,112,599 XP)** and **Level 90 (151,417,175 XP)**, the closest level where you reach approximately half the max XP is **Level 95**.
> 
> Thus, the answer is **Level 95**.

---

👤 **ubergarm** commented on **2025-08-10** at **15:42:12**

@Thireus 

> Full 128K context fits in one RTX PRO 6000 Blackwell.

Thanks for posting your GLM-4.5-Air fully offloaded results. I had an interesting report come in on hugging face with similar results as yours, that is suggesting mainline is running faster for this sm120 arch CUDA fully offloaded cases currently:

https://huggingface.co/ubergarm/GLM-4.5-Air-GGUF/discussions/2#68980a066bc7e6a3fcef131c

Have you done comparisons with mainline using the [llama-sweep-bench port](https://github.com/ubergarm/llama.cpp/tree/ug/port-sweep-bench) lately? I was surprised to see mainline appears to be markedly faster at least in this hardware/model/quant configuration.

---

👤 **Thireus** commented on **2025-08-10** at **17:38:06**

> Have you done comparisons with mainline using the [llama-sweep-bench port](https://github.com/ubergarm/llama.cpp/tree/ug/port-sweep-bench) lately? I was surprised to see mainline appears to be markedly faster at least in this hardware/model/quant configuration.

Oh wow, I definitely need to check this out! Thanks for the tip. I'll provide feedback after I do.

@usrlocalben - Kimi is a beast! Sadly quite slow and very big, but it's on my list of options too!

---

👤 **saood06** commented on **2025-08-10** at **19:13:29**

>IQ2 😰

Which IQ2?

---

👤 **usrlocalben** commented on **2025-08-10** at **21:21:47**

> > IQ2 😰
> 
> Which IQ2?

[ubergarm IQ2_KL](https://huggingface.co/ubergarm/Kimi-K2-Instruct-GGUF)

added intel autoround Q2 above as well.

@Thireus K2 is much faster than R1. fewer active params and no-think. much worse vram rent though.

---

👤 **Thireus** commented on **2025-08-10** at **22:10:11**

> Have you done comparisons with mainline using the [llama-sweep-bench port](https://github.com/ubergarm/llama.cpp/tree/ug/port-sweep-bench) lately? I was surprised to see mainline appears to be markedly faster at least in this hardware/model/quant configuration.

I can confirm llama.cpp is more than 2x faster than ik_llama.cpp on the Blackwell card. :|

I might switch back to llama.cpp then. 😂

See details: https://huggingface.co/ubergarm/GLM-4.5-Air-GGUF/discussions/2#689916c972f9ea17ff768e4b

---

👤 **saood06** commented on **2025-08-10** at **23:10:19**

> > > IQ2 😰
> > 
> > 
> > Which IQ2?
> 
> [ubergarm IQ2_KL](https://huggingface.co/ubergarm/Kimi-K2-Instruct-GGUF)
> 
> added intel autoround Q2 above as well.

Thanks for clarifying and adding the extra data point, that looks like an impressive result for the autoround compared to the [ubergarm IQ2_KL](https://huggingface.co/ubergarm/Kimi-K2-Instruct-GGUF)

>@Thireus K2 is much faster than R1. fewer active params and no-think. much worse vram rent though.

I'm not @Thireus but I haven't tried K2 even though it sounds tempting for that reason because my R1/V3 quants leave me no spare room at ~4.5 BPW, and so either I go to a lower BPW or RPC and neither sounded compelling enough so far.

---

👤 **Thireus** commented on **2025-08-12** at **06:06:09**

Very interesting... GLM-4.5-Air was able to solve the [Dipiloblop](https://thireus.com/GITHUB/Dipiloblop.txt) prompt with mainline llama.cpp using the following recipe:

<details>

```
## Quant mix recipe created using Thireus' GGUF Tool Suite - https://gguf.thireus.com/
# Model name: GLM-4.5-Air
# Link to the original model: https://huggingface.co/zai-org/GLM-4.5-Air

## Model head & embeddings — qbits: 32 8 5 
output_norm\.weight=f32
token_embd\.weight=q5_K
output\.weight=q8_0

## Multi-headed attention parameters — qbits: 32 4 
blk\.([0-9]|[1-3][0-9]|4[0-6])\.attn_k\.bias=f32
blk\.([0-9]|[1-3][0-9]|4[0-6])\.attn_output\.weight=iq4_xs
blk\.([0-9]|[1-3][0-9]|4[0-6])\.attn_k\.weight=iq4_xs
blk\.([0-9]|[1-3][0-9]|4[0-6])\.attn_q\.weight=iq4_xs
blk\.([0-9]|[1-3][0-9]|4[0-6])\.attn_q\.bias=f32
blk\.([0-9]|[1-3][0-9]|4[0-6])\.attn_v\.bias=f32
blk\.([0-9]|[1-3][0-9]|4[0-6])\.attn_v\.weight=iq4_xs
blk\.([0-9]|[1-3][0-9]|4[0-6])\.attn_norm\.weight=f32

## Core FFN weights — qbits: 32 8 
blk\.0\.ffn_gate\.weight=q8_0
blk\.([1-9]|[1-3][0-9]|4[0-6])\.ffn_gate_inp\.weight=f32
blk\.0\.ffn_down\.weight=q8_0
blk\.0\.ffn_up\.weight=q8_0

## Other tensors — qbits: 32 4 
blk\.([0-9]|[1-3][0-9]|4[0-6])\.post_attention_norm\.weight=f32
blk\.46\.nextn\.shared_head_head\.weight=iq4_xs
blk\.46\.nextn\.embed_tokens\.weight=iq4_xs
blk\.46\.nextn\.shared_head_norm\.weight=f32
blk\.([1-9]|[1-3][0-9]|4[0-6])\.exp_probs_b\.bias=f32
blk\.46\.nextn\.enorm\.weight=f32
blk\.46\.nextn\.hnorm\.weight=f32
blk\.46\.nextn\.eh_proj\.weight=iq4_xs

## GPU-loaded ffn_*_shexp
# ffn_down_shexp (down-projection) — qbits: 4 
blk\.([1-9]|[1-3][0-9]|4[0-6])\.ffn_down_shexp\.weight=iq4_xs

# ffn_up_shexp (up-projection) — qbits: 8 6 5 
blk\.(1|[3-4]|[7-8]|17|19|30|32|38|2[3-5]|4[0-5]|2[8-9]|2[0-1]|1[2-3]|3[4-5])\.ffn_up_shexp\.weight=q8_0
blk\.([5-6]|22|31|33|39|46|[2-3][6-7]|1[5-6]|1[0-1])\.ffn_up_shexp\.weight=q6_K
blk\.(2|9|14|18)\.ffn_up_shexp\.weight=q5_K

# ffn_gate_shexp (gate-projection) — qbits: 8 6 5 
blk\.([1-3]|[6-9]|19|20|30|33|4[0-5]|1[0-2]|2[3-9])\.ffn_gate_shexp\.weight=q8_0
blk\.(4|13|18|46|[2-3][1-2]|3[4-9]|1[5-6])\.ffn_gate_shexp\.weight=q6_K
blk\.(5|14|17)\.ffn_gate_shexp\.weight=q5_K

## CPU-loaded ffn_*_exps
# ffn_down_exps (down-extraction) — qbits: 4 
blk\.([1-9]|[1-3][0-9]|4[0-6])\.ffn_down_exps\.weight=iq4_nl

# ffn_up_exps (up-extraction) — qbits: 5 4 3 2 
blk\.46\.ffn_up_exps\.weight=q5_K
blk\.(25|37|39|3[1-2]|2[8-9]|3[4-5])\.ffn_up_exps\.weight=iq4_xs
blk\.(3|[5-6]|10|12|15|18|30|33|36|38|4[0-5]|2[6-7]|2[0-4])\.ffn_up_exps\.weight=iq3_s
blk\.([1-2]|4|[7-9]|11|19|1[6-7]|1[3-4])\.ffn_up_exps\.weight=q2_K

# ffn_gate_exps (gate-extraction) — qbits: 5 4 3 2 
blk\.46\.ffn_gate_exps\.weight=q5_K
blk\.(10|2[8-9]|3[1-4])\.ffn_gate_exps\.weight=iq4_xs
blk\.(3|6|30|4[0-5]|2[0-7]|3[5-9]|1[8-9]|1[3-6])\.ffn_gate_exps\.weight=iq3_s
blk\.([1-2]|[4-5]|[7-9]|17|1[1-2])\.ffn_gate_exps\.weight=q2_K

## Summary of tensor sizes per class
# GPU Total: 5.015 GiB (94.9%) | 5.29 GiB max, if all were q8_0 | 4.64 GiB min, if all were q5_K
# CPU Total: 44.902 GiB (87.0%) | 51.61 GiB max, if all were iq4_xs | 39.04 GiB min, if all were q2_K
# GPU+CPU Total: 49.917 GiB (90.9%)

## Summary of tensor counts and bpw per qtype
#
# GPU-loaded quants:
# QTYPE		Count	BPW	Assigned GiB	% Assigned	Max GiB (all)
# +f32       	331	32.0  	  0.09 GiB	-		-
# +q8_0      	1  	8.5   	  0.04 GiB	-		-
# q8_0      	57 	8.5   	  1.01 GiB	54.9%		1.84
# q6_K      	31 	6     	  0.14 GiB	9.6%		1.42
# q5_K      	8  	5     	  0.42 GiB	35.5%		1.19
# +iq4_xs    	237	4.25  	  3.31 GiB	-		-
#
# CPU-loaded quants:
# QTYPE		Count	BPW	Assigned GiB	% Assigned	Max GiB (all)
# +q5_K      	2  	5     	  0.95 GiB	-		-
# +iq4_nl    	46 	4.5   	 17.79 GiB	-		-
# iq4_xs    	16 	4.25  	  5.84 GiB	17.8%		32.87
# iq3_s     	52 	3.4375	 15.36 GiB	57.8%		26.59
# q2_K      	22 	2     	  4.96 GiB	24.4%		20.30
#
# -Average BPW: 3.7031
#
# -Notes:
# - '+' means user-defined pre-assigned tensors, or tensor missing from csv data or f32 tensors
# - Recipe produced on the 2025-08-12 05:39:33 UTC+0000 using Thireus' GGUF tools (https://gguf.thireus.com/)
# - Script SHA-256: a02563df96ccec6c78ab7c716771153ae5f5ef4e9ee6a04d372f273eb1662e9c
# - Calibration dataset 'ppl_results.csv' SHA-256: c596235f01c582988d23f97e1e6809a83923ae3f5321e3cde00625c9c92952f3
# - tensors.bf16.map SHA-256: f440313db9b7ce593240c0b0acb723182ee3ae9570eca868dc6eb440112fdd67
# - tensors.bf16.map model name: GLM-4.5-Air-THIREUS-BF16-SPECIAL_TENSOR-00804-of-00804
# - tensors.iq4_xs.map SHA-256: 28c799175c45409d6f59d609e82c5f0ed2bba3240b7c5697afbdc76824b1b046
# - tensors.iq4_xs.map model name: GLM-4.5-Air-THIREUS-IQ4_XS-SPECIAL_TENSOR-00804-of-00804
# - tensors.iq3_s.map SHA-256: c728289eeab5a07292bbaec6db8949ba9ddcf63e6cc337b0f15565755769e115
# - tensors.iq3_s.map model name: GLM-4.5-Air-THIREUS-IQ3_S-SPECIAL_TENSOR-00804-of-00804
# - tensors.q2_K.map SHA-256: 35835d7d56a8161d07c31c73f42008cba7c7e2025d4bb60be2a82b254497f183
# - tensors.q2_K.map model name: GLM-4.5-Air-THIREUS-Q2_K-SPECIAL_TENSOR-00804-of-00804
# - tensors.q8_0.map SHA-256: c00093e70a6c32aab72b404457c12a7b238b0e030975267d93d2b09a30796151
# - tensors.q8_0.map model name: GLM-4.5-Air-THIREUS-Q8_0-SPECIAL_TENSOR-00804-of-00804
# - tensors.q5_K.map SHA-256: b60aadca788055846a572cad5121e1d93bfa9bbbd520ae6350c84f52319f945f
# - tensors.q5_K.map model name: GLM-4.5-Air-THIREUS-Q5_K-SPECIAL_TENSOR-00804-of-00804
# - tensors.q6_K.map SHA-256: 5165939ae192b9008b49432f574da6df0a8df9989faf337cc3a062d04f80aef2
# - tensors.q6_K.map model name: GLM-4.5-Air-THIREUS-Q6_K-SPECIAL_TENSOR-00804-of-00804
# - tensors.iq2_ks.map SHA-256: efed8f3d7d712a6ad99c5904f6e2f4b89387cc78e4008d9ca557bd04da1f2b31
# - tensors.iq2_ks.map model name: GLM-4.5-Air-THIREUS-IQ2_KS-SPECIAL_TENSOR-00804-of-00804
# - tensors.iq4_nl.map SHA-256: ed9419ab49fc319d033148d4241f28fcafb1d3089cb3172da9fa5f62c978a93d
# - tensors.iq4_nl.map model name: GLM-4.5-Air-THIREUS-IQ4_NL-SPECIAL_TENSOR-00804-of-00804
# - GPG signatures: PASSED
# - Command used:
# ../../quant_assign.py ppl_results.csv --tolerance 0.01 --cpu-irq-k 1.5 --gpu-irq-k 1.5 --gpu-assign-qtype iq4_xs \
# --cpu-tensors-max-size 45 --gpu-tensors-max-size 95% --exponential-factor 8 --cpu-tensors \
# 'blk\.([1-9]|[1-3][0-9]|4[0-5])\.ffn_up_exps\.weight' 'blk\.([1-9]|[1-3][0-9]|4[0-5])\.ffn_gate_exps\.weight' \
# --gpu-tensors '.*' --cpu-quants iq4_xs iq3_s q2_K --gpu-quants q8_0 q5_K q6_K --cpu-assign-tensors \
# 'blk\.(46)\.ffn_up_exps\.weight=q5_K' 'blk\.(46)\.ffn_gate_exps\.weight=q5_K' \
# 'blk\.([1-9]|[1-3][0-9]|4[0-6])\.ffn_down_exps\.weight=iq4_nl' --gpu-assign-tensors \
# 'blk\.(0)\.ffn_down\.weight=q8_0'

## THE END!
```

</details>

Model answer:

<details>

```
<think>Let's analyze the question: "In dipiloblop, what level are you at when you get to half the xp of max level?"

From the provided book, I need to look for information about experience and leveling. I should find the experience table or any information about how experience relates to levels.

Looking through the text, I found a "Skills/Experience Table" section that shows experience needed for each level. Let me extract this information:

\`\`\`
Level 	Experience needed to reach level 	Experience difference from last level
1	0	0
2	266	266
3	553	287
4	888	335
5	1,264	376
6	1,697	433
7	2,202	505
8	2,761	559
9	3,421	660
10	4,162	741
11	5,009	847
12	5,987	978
13	7,090	1,103
14	8,339	1,249
15	9,792	1,453
16	11,423	1,631
17	13,273	1,850
18	15,397	2,124
19	17,798	2,401
20	20,537	2,739
21	23,635	3,098
22	27,183	3,548
23	31,160	3,977
24	35,720	4,560
25	40,895	5,175
26	46,750	5,855
27	53,376	6,626
28	60,943	7,567
29	69,493	8,550
30	79,195	9,702
31	90,198	11,003
32	102,691	12,493
33	116,838	14,147
34	132,892	16,054
35	151,085	18,193
36	171,731	20,646
37	195,093	23,362
38	221,634	26,541
39	251,703	30,069
40	285,768	34,065
41	324,379	38,611
42	368,189	43,810
43	417,795	49,606
44	474,049	56,254
45	537,848	63,799
46	610,073	72,225
47	691,990	81,917
48	784,837	92,847
49	890,059	105,222
50	1,009,301	119,242
51	1,144,480	135,179
52	1,297,644	153,164
53	1,471,284	173,640
54	1,667,999	196,715
55	1,891,034	223,035
56	2,143,788	252,754
57	2,430,212	286,424
58	2,754,802	324,590
59	3,122,772	367,970
60	3,539,721	416,949
61	4,012,280	472,559
62	4,547,846	535,566
63	5,154,809	606,963
64	5,842,739	687,930
65	6,622,340	779,601
66	7,505,874	883,534
67	8,507,246	1,001,372
68	9,642,117	1,134,871
69	10,928,297	1,286,180
70	12,385,868	1,457,571
71	14,037,935	1,652,067
72	15,910,153	1,872,218
73	18,031,971	2,121,818
74	20,436,698	2,404,727
75	23,162,089	2,725,391
76	26,250,771	3,088,682
77	29,751,292	3,500,521
78	33,718,506	3,967,214
79	38,214,716	4,496,210
80	43,310,285	5,095,569
81	49,085,275	5,774,990
82	55,630,259	6,544,984
83	63,047,786	7,417,527
84	71,454,370	8,406,584
85	80,981,751	9,527,381
86	91,779,442	10,797,691
87	104,016,620	12,237,178
88	117,885,480	13,868,860
89	133,603,420	15,717,940
90	151,417,175	17,813,755
91	171,605,798	20,188,623
92	194,486,167	22,880,369
93	220,417,391	25,931,224
94	249,805,802	29,388,411
95	283,112,599	33,306,797
96	320,860,372	37,747,773
97	363,640,890	42,780,518
98	412,125,351	48,484,461
99	467,074,482	54,949,131
100	507,038,931	48,364,282
101	556,387,432	49,348,501
\`\`\`

To answer the question, I need to determine what level corresponds to having half the experience needed to reach the maximum level.

From the table, I can see that:
- Level 100 requires 507,038,931 experience points
- Level 101 requires 556,387,432 experience points

The maximum level shown is 101, which requires 556,387,432 experience points.

Half of the experience needed to reach level 101 is 556,387,432 ÷ 2 = 278,193,716 experience points.

Looking at the table, I need to find which level requires closest to 278,193,716 experience points.

From the table:
- Level 75 requires 23,162,089 experience points
- Level 76 requires 26,250,771 experience points
- Level 77 requires 29,751,292 experience points
- Level 78 requires 33,718,506 experience points
- Level 79 requires 38,214,716 experience points
- Level 80 requires 43,310,285 experience points
- Level 81 requires 49,085,275 experience points
- Level 82 requires 55,630,259 experience points
- Level 83 requires 63,047,786 experience points
- Level 84 requires 71,454,370 experience points
- Level 85 requires 80,981,751 experience points
- Level 86 requires 91,779,442 experience points
- Level 87 requires 104,016,620 experience points
- Level 88 requires 117,885,480 experience points
- Level 89 requires 133,603,420 experience points
- Level 90 requires 151,417,175 experience points
- Level 91 requires 171,605,798 experience points
- Level 92 requires 194,486,167 experience points
- Level 93 requires 220,417,391 experience points
- Level 94 requires 249,805,802 experience points
- Level 95 requires 283,112,599 experience points

Half of the max experience (278,193,716) falls between level 94 (249,805,802) and level 95 (283,112,599).

To be more precise, I can calculate how far we are from level 94 to level 95:
- Level 94 to 95 difference: 283,112,599 - 249,805,802 = 33,306,797 experience points
- How much we need beyond level 94: 278,193,716 - 249,805,802 = 28,387,914 experience points
- Percentage of the way: 28,387,914 ÷ 33,306,797 ≈ 0.852 or 85.2%

So, you would be at approximately 85.2% of the way from level 94 to level 95 when you have half the experience needed to reach level 101.

However, the question asks what level you're at, not what percentage. Since you don't reach level 95 until you have 283,112,599 experience points, and 278,193,716 is less than that, you would still be level 94 when you have half the experience needed to reach the maximum level of 101.

Actually, let me double-check this. The question is asking what level you're at when you have half the experience needed to reach the maximum level. The maximum level shown in the table is 101, which requires 556,387,432 experience points.

Half of that is 278,193,716 experience points.

Looking at the table:
- Level 94 requires 249,805,802 experience points
- Level 95 requires 283,112,599 experience points

Since 278,193,716 is greater than 249,805,802 but less than 283,112,599, you would be level 94 when you have half the experience needed to reach level 101.

Wait, I think I need to be more precise. In most RPGs like this, you're considered a certain level once you've accumulated enough experience to reach that level, not when you're partway through it. So, when you have 278,193,716 experience points, you've accumulated enough to reach level 94, but not enough to reach level 95. Therefore, you would be considered level 94.

So the answer is that you're at level 94 when you have half the experience needed to reach the maximum level of 101.</think>Based on the experience table provided in the book, I can determine what level you're at when you have half the experience needed to reach the maximum level.

The maximum level shown in the table is level 101, which requires 556,387,432 experience points to reach.

Half of this experience requirement is:
556,387,432 ÷ 2 = 278,193,716 experience points

Looking at the experience table:
- Level 94 requires 249,805,802 experience points
- Level 95 requires 283,112,599 experience points

Since 278,193,716 is greater than the requirement for level 94 but less than the requirement for level 95, you would be at level 94 when you have half the experience needed to reach the maximum level of 101.
```

</details>

```
Prompt
- Tokens: 104293
- Time: 115899.915 ms
- Speed: 899.9 t/s
Generation
- Tokens: 2762
- Time: 139554.456 ms
- Speed: 19.8 t/s
```

---

👤 **Thireus** commented on **2025-08-12** at **09:03:08**

I'm positive. Same model, different results: ik_llama.cpp is not able to solve it, while llama.cpp solves it. Something is up with our implementation.

---

👤 **ikawrakow** commented on **2025-08-12** at **09:08:47**

@Thireus

Unless you are using a zero-temperature setting, to make a definitive statement you need to run the task several times and count how many times the task was solved.

But given the performance results, I wouldn't be surprised if the implementation in `ik_llama.cpp` is not 100% correct. I'm in the middle of implementing GPT-OSS, so don't want to get distracted with other stuff, but when I'm done, I'll try to look more closely into GLM-4.5.

---

👤 **Thireus** commented on **2025-08-12** at **19:58:46**

@ikawrakow, you're right - need a more scientific testing methodology.  I have decided to set the temps and related settings the same between ik_llama.cpp and llama.cpp and do 10 rounds for each. Here are the results:

[ik_llama.cpp](https://github.com/Thireus/ik_llama.cpp/releases/tag/main-b4074-62ef02e):

```
CUDA_DEVICE_ORDER=PCI_BUS_ID CUDA_VISIBLE_DEVICES=0 ~/ik_llama-main-b4074-62ef02e-bin-win-cuda-12.8-x64-avx512/llama-server -m GLM-4.5-Air-THIREUS-BF16-SPECIAL_TENSOR-00001-of-00804.gguf -fa   -ctk f16   -ngl 99   -b 4096 -ub 4096   --threads 36   --main-gpu 0 --no-mmap --temp 0.6 --top-k 40 --top-p 0.95 --min-p 0.05
```

0. WRONG, BUT CORRECT... Thinks max level is 99.
1. WRONG. Numbers pulled from the XP table are mixed up.
2. ALMOST. Considered level 95 instead.
3. CORRECT!
4. WRONG, BUT CORRECT... Numbers pulled from the XP table are mixed up, but answer is correct.
5. WRONG, BUT CORRECT... Numbers pulled from the XP table are mixed up, but answer is correct.
6. CORRECT!
7. CORRECT!
8. WRONG, completely wrong, model is hallucinating. Numbers pulled from the XP table are invented.
9. WRONG. Numbers pulled from the XP table are very mixed up.

[llama.cpp](https://github.com/Thireus/llama.cpp/releases/tag/b6146):

```
$ CUDA_DEVICE_ORDER=PCI_BUS_ID CUDA_VISIBLE_DEVICES=1 ~/llama-b6146-bin-win-cuda-12.8-x64/llama-server -m GLM-4.5-Air-THIREUS-BF16-SPECIAL_TENSOR-00001-of-00804.gguf -fa   -ctk f16   -c 131072   -ngl 99   -b 4096 -ub 4096   --no-mmap   --threads 36   --main-gpu 0 --temp 0.6 --top-k 40 --top-p 0.95 --min-p 0.05 --port 8081
```

0. WRONG. Numbers pulled from the XP table are mixed up.
1. CORRECT!
2. WRONG. Numbers pulled from the XP table are mixed up.
3. WRONG, BUT CORRECT... Numbers pulled from the XP table are mixed up, but answer is correct.
4. CORRECT!
5. CORRECT!
6. CORRECT!
7. CORRECT!
8. CORRECT!
9. CORRECT!

So, it looks like ik_llama.cpp is somehow unable to pull the correct XP for level 101 most of the time.

The same model recipe was used for both:

<details>

```
## Model head & embeddings
output_norm\.weight=f32
token_embd\.weight=q5_K
output\.weight=q8_0

## Multi-headed attention parameters
blk\.([0-9]|[1-3][0-9]|4[0-6])\.attn_k\.bias=f32
blk\.([0-9]|[1-3][0-9]|4[0-6])\.attn_output\.weight=iq4_xs
blk\.([0-9]|[1-3][0-9]|4[0-6])\.attn_k\.weight=iq4_xs
blk\.([0-9]|[1-3][0-9]|4[0-6])\.attn_q\.weight=iq4_xs
blk\.([0-9]|[1-3][0-9]|4[0-6])\.attn_q\.bias=f32
blk\.([0-9]|[1-3][0-9]|4[0-6])\.attn_v\.bias=f32
blk\.([0-9]|[1-3][0-9]|4[0-6])\.attn_v\.weight=iq4_xs
blk\.([0-9]|[1-3][0-9]|4[0-6])\.attn_norm\.weight=f32

## Core FFN weights
blk\.0\.ffn_gate\.weight=iq3_xxs
blk\.([1-9]|[1-3][0-9]|4[0-6])\.ffn_gate_inp\.weight=f32
blk\.0\.ffn_down\.weight=iq3_xxs
blk\.0\.ffn_up\.weight=iq3_xxs

## Other tensors
blk\.([0-9]|[1-3][0-9]|4[0-6])\.post_attention_norm\.weight=f32
blk\.46\.nextn\.shared_head_head\.weight=iq4_xs
blk\.46\.nextn\.embed_tokens\.weight=iq4_xs
blk\.46\.nextn\.shared_head_norm\.weight=f32
blk\.([1-9]|[1-3][0-9]|4[0-6])\.exp_probs_b\.bias=f32
blk\.46\.nextn\.enorm\.weight=f32
blk\.46\.nextn\.hnorm\.weight=f32
blk\.46\.nextn\.eh_proj\.weight=iq4_xs

## GPU-loaded ffn_*_shexp
# ffn_down_shexp (down-projection) — qbits: 4 
blk\..*\.ffn_down_shexp\.weight=iq3_xxs

# ffn_up_shexp (up-projection)
blk\..*\.ffn_up_shexp\.weight=iq3_xxs

# ffn_gate_shexp (gate-projection)
blk\..*\.ffn_gate_shexp\.weight=iq3_xxs

## CPU-loaded ffn_*_exps
# ffn_down_exps (down-extraction)
blk\..*\.ffn_down_exps\.weight=iq3_xxs

# ffn_up_exps (up-extraction)
blk\..*\.ffn_up_exps\.weight=iq3_xxs

# ffn_gate_exps (gate-extraction)
blk\..*\.ffn_gate_exps\.weight=iq3_xxs
```

</details>

[ik_vs_llama.zip](https://github.com/user-attachments/files/21751568/ik_vs_llama.zip)

---

👤 **trilog-inc** commented on **2025-08-12** at **20:08:35**

One thing to note, for long context -b 4096 and -ub 4096 breaks the model. Same with mainline. Maybe you should use -b 2048 and -ub 2048 

 https://github.com/ggml-org/llama.cpp/pull/14939#issuecomment-3153285813

---

👤 **Thireus** commented on **2025-08-12** at **20:09:31**

> One thing to note, for long context -b 4096 and -ub 4096 breaks the model. Same with mainline. Maybe you should use -b 2048 and -ub 2048
> 
> [ggml-org/llama.cpp[#14939](https://github.com/ikawrakow/ik_llama.cpp/issues/14939) (comment)](https://github.com/ggml-org/llama.cpp/pull/14939#issuecomment-3153285813)

Ah! I'll give it a go thanks. Interestingly I have not noticed any issue with llama.cpp.

Edit: Nope that's not it, same issue.
Edit2: omg, `-n 512` was in the command line. Alright, giving it all another shot.
Edit3: Previous results updated. Outcome remains that something is up with the ik_llama.cpp implementation (or I'm doing something wrong, which is entirely possible).

---

👤 **saood06** commented on **2025-08-13** at **04:54:59**

> [ik_llama.cpp](https://github.com/Thireus/ik_llama.cpp/releases/tag/main-b4074-62ef02e):
> 
> ```
> CUDA_DEVICE_ORDER=PCI_BUS_ID CUDA_VISIBLE_DEVICES=0 ~/ik_llama-main-b4074-62ef02e-bin-win-cuda-12.8-x64-avx512/llama-server -m GLM-4.5-Air-THIREUS-BF16-SPECIAL_TENSOR-00001-of-00804.gguf -fa   -ctk f16   -ngl 99   -b 4096 -ub 4096   --threads 36   --main-gpu 0 -p 8192 --no-mmap --temp 0.6 --top-k 40 --top-p 0.95 --min-p 0.05
> ```
>[...]
> [llama.cpp](https://github.com/Thireus/llama.cpp/releases/tag/b6146):
> 
> ```
> $ CUDA_DEVICE_ORDER=PCI_BUS_ID CUDA_VISIBLE_DEVICES=1 ~/llama-b6146-bin-win-cuda-12.8-x64/llama-server -m GLM-4.5-Air-THIREUS-BF16-SPECIAL_TENSOR-00001-of-00804.gguf -fa   -ctk f16   -c 131072   -ngl 99   -b 4096 -ub 4096   --no-mmap   --threads 36   --main-gpu 0 --temp 0.6 --top-k 40 --top-p 0.95 --min-p 0.05 --port 8081
> ```

Why is `-p 8192` set for ik_llama? It is not set for llama.cpp

---

👤 **Thireus** commented on **2025-08-13** at **09:46:45**

> Why is `-p 8192` set for ik_llama? It is not set for llama.cpp

Lack of sleep. I ran everything again and updated my results in this post: https://github.com/ikawrakow/ik_llama.cpp/pull/668#issuecomment-3180833773

Same outcome as before, I believe this parameter didn't have any effect.

---

👤 **saood06** commented on **2025-08-13** at **09:53:09**

>I believe this parameter didn't have any effect

I thought it set a global system_prompt which would prefix the context which could affect results.

---

👤 **Thireus** commented on **2025-08-13** at **10:02:10**

> I thought it set a global system_prompt which would prefix the context which could affect results.

You were right to point this out, I think this is what it does indeed and probably didn't make the previous results any better. The updated results are without this -p option.

Would be good if someone could replicate this test without necessarily using my parameters but their own to validate if something is up with our implementation. Could be RoPE, could be something else... could also be extreme good luck I had with 10 good seeds on llama.cpp... but I doubt it.

---

👤 **ikawrakow** commented on **2025-08-13** at **10:11:46**

> WRONG, BUT CORRECT... Numbers pulled from the XP table are mixed up, but answer is correct.

So, if we count this as correct, then `llama.cpp` gets 9/10, `ik_llama.cpp` gets 6/10. Let's assume for a moment that there isn't a real difference between the two, and let's assume that the actual fraction of correct answers is 7.5/10 (average of `llama.cpp` and `ik_llama.cpp`). With these assumptions, a sample of 10 random draws has a statistical uncertainty of `7.5*sqrt(0.25*0.75/9) = 3.2`, and hence the difference is not really statistically significant.

But if there is a real difference, the fact that `llama.cpp` mixes up the table more often may indicate an issue with tokenization and/or handling of special tokens.

---

👤 **trilog-inc** commented on **2025-08-13** at **14:43:39**

Another update from mainline, @ajunca narrowed down some of the large context issues to a CUDA graph issue. Could be interesting to run your test with a compiled version with GGML_CUDA_USE_GRAPHS=OFF 

https://github.com/ggml-org/llama.cpp/pull/14939#issuecomment-3181307048

---

👤 **ikawrakow** commented on **2025-08-13** at **14:48:41**

CUDA graphs are turned off for MoE models in `ik_llama.cpp`.

Oh, and the huge PPL observed for GPT-OSS is there with or without  CUDA graphs. Here and in mainline.

---

👤 **ajunca** commented on **2025-08-13** at **15:27:57**

> Another update from mainline, @ajunca narrowed down some of the large context issues to a CUDA graph issue. Could be interesting to run your test with a compiled version with GGML_CUDA_USE_GRAPHS=OFF
> 
> [ggml-org/llama.cpp[#14939](https://github.com/ikawrakow/ik_llama.cpp/issues/14939) (comment)](https://github.com/ggml-org/llama.cpp/pull/14939#issuecomment-3181307048)

No, at the end was not a problem of cuda graphs... After more testing, the problem was still there. I think though, is a problem of CUDA device synchronization. And that there are two problems related:

- One is the gibberish problem (eg. GGGGGGGGG...), that at the end if you wait long enough (sometimes is very fast, some times it takes longer), you get a cuda crash like:
```
/app/ggml/src/ggml-cuda/ggml-cuda.cu:84: CUDA error
CUDA error: an illegal memory access was encountered
[0m  current device: 1, in function ggml_backend_cuda_synchronize at /app/ggml/src/ggml-cuda/ggml-cuda.cu:2612
[0m  cudaStreamSynchronize(cuda_ctx->stream())
[0mlibggml-base.so(+0x16d4b)[0x7fb29ab44d4b]
libggml-base.so(ggml_print_backtrace+0x21f)[0x7fb29ab451af]
libggml-base.so(ggml_abort+0x152)[0x7fb29ab45382]
/app/libggml-cuda.so(+0xea826)[0x7fb292aea826]
/app/libggml-cuda.so(+0xebaf3)[0x7fb292aebaf3]
libggml-base.so(ggml_backend_sched_synchronize+0x2e)[0x7fb29ab5aece]
libllama.so(_ZN13llama_context11synchronizeEv+0x14)[0x7fb29a897ac4]
libllama.so(llama_get_logits_ith+0x16)[0x7fb29a899de6]
/app/llama-server(+0x1f00fe)[0x55d97e53e0fe]
/app/llama-server(+0xdf547)[0x55d97e42d547]
/app/llama-server(+0x857bd)[0x55d97e3d37bd]
/app/llama-server(+0x4d6a5)[0x55d97e39b6a5]
/usr/lib/x86_64-linux-gnu/libc.so.6(+0x29d90)[0x7fb29a029d90]
/usr/lib/x86_64-linux-gnu/libc.so.6(__libc_start_main+0x80)[0x7fb29a029e40]
/app/llama-server(+0x4f0f5)[0x55d97e39d0f5]
```

- And second issue is that some times, during prompt processing it gets kind of stuck/freeze, and does not advance more, lets say it stucks at 0.84. If you look at GPU usage, you will see one idle, and the other 100%. It also smells a bit as a synchronization issue. This is why I think they are connected. Example when pp gets stuck:
<img width="1686" height="319" alt="Screenshot_20250813_143915" src="https://github.com/user-attachments/assets/4f37cff9-e4fe-4b02-a1db-c7bad38d969f" />

If you go to #https://github.com/ggml-org/llama.cpp/issues/15112 you will see things I tried. 

Probably I will open a new issue, with all the stuff.

---

👤 **ikawrakow** commented on **2025-08-13** at **15:30:34**

@ajunca The problems you are describing are about mainline `llama.cpp`, not `ik_llama.cpp`, correct?

---

👤 **trilog-inc** commented on **2025-08-13** at **15:35:20**

Apologies if i brought in a different issue. I can report that setting -b 4096 and -ub 4096 affects ik_llama the same way as ajunca has described in mainline. Thought it could be good to reference their investigation if they are related, maybe they arent

---

👤 **ajunca** commented on **2025-08-13** at **15:46:01**

> @ajunca The problems you are describing are about mainline `llama.cpp`, not `ik_llama.cpp`, correct?

Yes indead, are related to llama.cpp. I tried also ik_llama.cpp at some point, I remember I had problems, but don't remember which ones. Does ik_llama.cpp has a Vulkan backend? In llama.cpp, the Vulkan backend is workings (but much slower).

---

👤 **ikawrakow** commented on **2025-08-13** at **15:51:18**

> In llama.cpp, the Vulkan backend is workings (but much slower).

The much slower part is no longer true.

---

👤 **Thireus** commented on **2025-08-13** at **15:55:41**

> One is the gibberish problem (eg. GGGGGGGGG...), that at the end if you wait long enough (sometimes is very fast, some times it takes longer), you get a cuda crash like:

I can confirm the GGGGGGG problem occurred to me as well when using the IQ3_K quant for some reason - https://huggingface.co/Thireus/Qwen3-Coder-480B-A35B-Instruct-THIREUS-IQ3_K-SPECIAL_SPLIT/discussions/1#6898de3d70c4fed919a7e7e9 - I thought it had something to do with the quantization of the model for that qtype being wrong... but seems like it may not be it. I've avoided using this quant and haven't been able to reproduce the issue with other quants since.

---

👤 **ajunca** commented on **2025-08-13** at **15:55:56**

> > In llama.cpp, the Vulkan backend is workings (but much slower).
> 
> The much slower part is no longer true.

I mean that when running with the Vulkan backend is not producing the mentioned problem. So looks like a CUDA problem. 

Well just tried today. I get like 60% speed for generation (not bad at all), but only like 15% for prompt processing, so yea, quite slower on this regard.

---

👤 **ajunca** commented on **2025-08-13** at **15:56:53**

> > One is the gibberish problem (eg. GGGGGGGGG...), that at the end if you wait long enough (sometimes is very fast, some times it takes longer), you get a cuda crash like:
> 
> I can confirm the GGGGGGG problem occurred to me as well when using the IQ3_K quant for some reason - https://huggingface.co/Thireus/Qwen3-Coder-480B-A35B-Instruct-THIREUS-IQ3_K-SPECIAL_SPLIT/discussions/1#6898de3d70c4fed919a7e7e9 - I thought it had something to do with the quantization of the model for that qtype being wrong... but seems like it may not be it.

I tried many different, even MXFP4, and at some point and with enough testing, all seems to produce it.

---

👤 **ikawrakow** commented on **2025-08-13** at **16:02:33**

> Well just tried today. I get like 60% speed for generation (not bad at all), but only like 15% for prompt processing, so yea, quite slower on this regard.

15% of what? 15% of mainline `llama.cpp`, 15% of the speed on  the same system using the CUDA back-end? Something else?

---

👤 **ajunca** commented on **2025-08-13** at **16:05:29**

> > Well just tried today. I get like 60% speed for generation (not bad at all), but only like 15% for prompt processing, so yea, quite slower on this regard.
> 
> 15% of what? 15% of mainline `llama.cpp`, 15% of the speed on the same system using the CUDA back-end? Something else?

Yea sorry, I was comparing the Vulkan vs Cuda in llama.cpp. Saying that in Vulkan it seems to work correctly, the only problem is that is considerably slower than the CUDA backend. (percentage compared to CUDA toks/s).