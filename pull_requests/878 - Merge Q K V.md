## 🔀 [Pull Request #878](https://github.com/ikawrakow/ik_llama.cpp/pull/878) - Merge Q, K, V

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/merge_qkv` |
| **Target Branch** | `main` |
| **Created** | 2025-10-29 |
| **Updated** | 2025-10-30 |
| **Merged** | 2025-10-30 |

---

## 📄 Description

This PR adds the option to merge `Q, K, V` tensors into a single, contiguous `QKV` tensor while loading the model. If the model has biases for  `Q, K, V` (e.g., GLM-4.5-MoE), these are also merged.

> [!IMPORTANT]
> Merging can only be done if the quantization types of `Q, K` and `V` are the same

The upside of doing this is better TG GPU performance as this reduces the number of kernel launches.

The downside is that `mmap` cannot be used because the merged tensor cannot be memory mapped to the model file (as there are gaps between the Q, K, V tensors).

Hence, for now, there is a command line option to enable `Q, K, V` merge, which is disabled by default:
```
-mqkv  or --merge-qkv
```
When specified, and the merging dis indeed happen, `mmap` is automatically disabled.

I have added the necessary changes to model loading and graph building to the following architectures for now:
* `LLM_ARCH_LLAMA, LLM_ARCH_LLAMA4` - covers all LlaMA models  
* `LLM_ARCH_QWEN3` - Qwen3 dense models
* `LLM_ARCH_QWEN3MOE` - Qwen3-MoE models
* `LLM_ARCH_GLM4_MOE` - GLM-4.5/6-MoE models
* `LLM_ARCH_OPENAI_MOE` - GPT-OSS models

Given interest and willingness to test, the approach can easily be extended to basically everything else.

I'm observing a **~4% speedup** for TG with Qwen3-30B-A3B fully loaded on the GPU.

The performance gain is less for the other tested arches, typically in the **1-2%** range (including dense models!).

Interestingly enough, I'm observing some performance gain even for hybrid inference with GLM-4.5-Air - about 4% at zero context with `--n-cpu-moe 41` (which is the maximum I can do on my GPU with `-b 4096 -ub 4096` and context of 32k). It is interesting because with hybrid inference, during generation the bulk of the time is spent on computing the MoE operations on the CPU, so effects related to a faster self attention are greatly reduced. Hence, I would be curios to see results for GLM-4.5/6-Air on one of the Blackwell Pro cards with full GPU offload.

---

## 💬 Conversation

👤 **ubergarm** commented on **2025-10-29** at **16:12:56**

While I don't have access to a Blackwell GPU for full offload, I did some hybrid CPU+CUDA testing with this PR.

My first observation is that this new `--merge-qkv` feature seems to require the `attn_(q|k|v)` tensors to be the same quantization type as otherwise it prints out:

```
merge_qkv: did not merge Q, K, V in layer 0 because 0, 0, 1
merge_qkv: did not merge Q, K, V in layer 1 because 0, 0, 1

...

# src/llama-load-tensors.cpp line ~2483
printf("%s: did not merge Q, K, V in layer %d because %d, %d, %d\n", __func__, i,
    wq->type == wk->type, wq->type == wv->type, hparams.f_attention_scale == 0.0f);
```

So unfortunately it won't work with ~~most~~ many layers for my published GLM-4.5-Air and GLM-4.6 quants.

I was able to test it with gpt-oss-120b however which showed very slight improvement (+~0.7%) in TG over main despite only ~9 additional dense layers offloaded onto GPU with remaining on CPU.

<img width="2262" height="1110" alt="sweep-bench-PR878-gpt-oss-120b" src="https://github.com/user-attachments/assets/7f78bb64-db13-4db0-b194-fea33a42acc9" />

<details>

<summary>👈 Details</summary>

## main@c33f39d5
```bash
model=/mnt/ai/models/ggml-org/gpt-oss-120b-GGUF/gpt-oss-120b-mxfp4-00001-of-00003.gguf
./build/bin/llama-sweep-bench \
  --model "$model" \
  -c 20480 \
  -ub 4096 -b 4096 \
  -ngl 99 \
  -ot "blk\.(0|1|2|3|4|5|6|7|8)\.ffn.*=CUDA0" \
  -ot "ffn_(up|gate|down)_exps\.weight=CPU" \
  --warmup-batch \
  --threads 16 \
  --no-mmap

llm_load_tensors: offloaded 37/37 layers to GPU
llm_load_tensors:        CPU buffer size = 43683.05 MiB
llm_load_tensors:  CUDA_Host buffer size =   586.82 MiB
llm_load_tensors:      CUDA0 buffer size = 16282.53 MiB
....................................................................................................
llama_new_context_with_model: n_ctx         = 20480
llama_new_context_with_model: n_batch       = 4096
llama_new_context_with_model: n_ubatch      = 4096
llama_new_context_with_model: flash_attn    = 1
llama_new_context_with_model: mla_attn      = 0
llama_new_context_with_model: attn_max_b    = 0
llama_new_context_with_model: fused_moe     = 1
llama_new_context_with_model: grouped er    = 0
llama_new_context_with_model: fused_up_gate = 1
llama_new_context_with_model: fused_mmad    = 1
llama_new_context_with_model: ser           = -1, 0
llama_new_context_with_model: freq_base     = 150000.0
llama_new_context_with_model: freq_scale    = 0.03125
llama_kv_cache_init:      CUDA0 KV buffer size =  1440.00 MiB
llama_new_context_with_model: KV self size  = 1440.00 MiB, K (f16):  720.00 MiB, V (f16):  720.00 MiB
llama_new_context_with_model:  CUDA_Host  output buffer size =     0.77 MiB
llama_new_context_with_model:      CUDA0 compute buffer size =  3232.00 MiB
llama_new_context_with_model:  CUDA_Host compute buffer size =   365.05 MiB
llama_new_context_with_model: graph nodes  = 1409
llama_new_context_with_model: graph splits = 56
XXXXXXXXXXXXXXXXXXXXX Setting only active experts offload

main: n_kv_max = 20480, n_batch = 4096, n_ubatch = 4096, flash_attn = 1, n_gpu_layers = 99, n_threads = 16, n_threads_batch = 16
```

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  4096 |   1024 |      0 |    1.991 |  2057.56 |   23.494 |    43.59 |
|  4096 |   1024 |   4096 |    2.065 |  1983.29 |   23.725 |    43.16 |
|  4096 |   1024 |   8192 |    2.129 |  1923.54 |   24.025 |    42.62 |
|  4096 |   1024 |  12288 |    2.203 |  1858.91 |   24.191 |    42.33 |
|  4096 |   1024 |  16384 |    2.298 |  1782.59 |   24.468 |    41.85 |

## PR878 ik/merge_qkv@a2f3b08f --merge-qkv
```bash
model=/mnt/ai/models/ggml-org/gpt-oss-120b-GGUF/gpt-oss-120b-mxfp4-00001-of-00003.gguf
./build/bin/llama-sweep-bench \
  --model "$model" \
 --merge-qkv \
  -c 20480 \
  -ub 4096 -b 4096 \
  -ngl 99 \
  -ot "blk\.(0|1|2|3|4|5|6|7|8)\.ffn.*=CUDA0" \
  -ot "ffn_(up|gate|down)_exps\.weight=CPU" \
  --warmup-batch \
  --threads 16 \
  --no-mmap

llm_load_tensors: offloaded 37/37 layers to GPU
llm_load_tensors:        CPU buffer size = 43683.05 MiB
llm_load_tensors:  CUDA_Host buffer size =   586.82 MiB
llm_load_tensors:      CUDA0 buffer size = 16282.51 MiB
....................................................................................................
llama_new_context_with_model: n_ctx         = 20480
llama_new_context_with_model: n_batch       = 4096
llama_new_context_with_model: n_ubatch      = 4096
llama_new_context_with_model: flash_attn    = 1
llama_new_context_with_model: mla_attn      = 0
llama_new_context_with_model: attn_max_b    = 0
llama_new_context_with_model: fused_moe     = 1
llama_new_context_with_model: grouped er    = 0
llama_new_context_with_model: fused_up_gate = 1
llama_new_context_with_model: fused_mmad    = 1
llama_new_context_with_model: ser           = -1, 0
llama_new_context_with_model: freq_base     = 150000.0
llama_new_context_with_model: freq_scale    = 0.03125
llama_kv_cache_init:      CUDA0 KV buffer size =  1440.00 MiB
llama_new_context_with_model: KV self size  = 1440.00 MiB, K (f16):  720.00 MiB, V (f16):  720.00 MiB
llama_new_context_with_model:  CUDA_Host  output buffer size =     0.77 MiB
llama_new_context_with_model:      CUDA0 compute buffer size =  3232.00 MiB
llama_new_context_with_model:  CUDA_Host compute buffer size =   365.05 MiB
llama_new_context_with_model: graph nodes  = 1301
llama_new_context_with_model: graph splits = 56
XXXXXXXXXXXXXXXXXXXXX Setting only active experts offload

main: n_kv_max = 20480, n_batch = 4096, n_ubatch = 4096, flash_attn = 1, n_gpu_layers = 99, n_threads = 16, n_threads_batch = 16
```

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  4096 |   1024 |      0 |    2.001 |  2047.16 |   23.308 |    43.93 |
|  4096 |   1024 |   4096 |    2.064 |  1984.87 |   23.561 |    43.46 |
|  4096 |   1024 |   8192 |    2.144 |  1910.62 |   23.859 |    42.92 |
|  4096 |   1024 |  12288 |    2.210 |  1853.32 |   24.013 |    42.64 |
|  4096 |   1024 |  16384 |    2.308 |  1774.96 |   24.292 |    42.15 |

</details>

---

👤 **ikawrakow** commented on **2025-10-29** at **16:18:32**

@ubergarm Thanks for pointing out that it only works when all 3 tensors are of the same type. I have added that to the description.

> So unfortunately it won't work with most layers for my published GLM-4.5-Air and GLM-4.6 quants.

That's too bad. Somehow I thought most of your quants were using `Q8_0` attention tensors.

---

👤 **ubergarm** commented on **2025-10-29** at **16:31:12**

> That's too bad. Somehow I thought most of your quants were using Q8_0 attention tensors.

Ahh yes, good point, my recipes lately do use q8_0 for all attn tensors especially the `IQN_K` quants. The `_KS` and smaller variants tend to use q8_0 for `attn_(k|v)` but `attn_(q|o)` tend to be around ~6bpw.

So this will still work for those full q8_0 attn mixes and I'll keep it in mind to continue to release at least some full q8_0 attn mixes like this.

---

👤 **Ph0rk0z** commented on **2025-10-29** at **16:33:03**

I take it that it can't work for deepseek? I will try it when I load GLM 4.6 again.

---

👤 **ikawrakow** commented on **2025-10-29** at **16:41:25**

> I take it that it can't work for deepseek? I will try it when I load GLM 4.6 again.

DeepSeek's attention is completely different, so this trick does not apply.

---

👤 **ikawrakow** commented on **2025-10-29** at **16:43:02**

> and I'll keep it in mind to continue to release at least some full q8_0 attn mixes like this.

They don't need to be `q8_0`, just need to be of the same type to be able to merge. `attn_output` does not need to be the same as `attn_q, attn_k, attn_v`.

---

👤 **Nexesenex** commented on **2025-10-29** at **17:48:53**

@Ikawrakow, would it be possible to have options for merging only:

- Q & K.
- K & V.

Many quants have the same type for Q and K, and some others, like mine and Ubergarm's can use a similar K and V indeed.

- As for the couple Q & V, it would usually never occur with K not being similar to V. But if it's not hassle, it could be added as well for the sake of exhaustivity.

---

👤 **sousekd** commented on **2025-10-29** at **20:18:42**

> Hence, I would be curios to see results for GLM-4.5/6-Air on one of the Blackwell Pro cards with full GPU offload.

I see very small difference on RTX PRO 6000 @ 450 W running [anikifoss/GLM-4.5-Air-HQ4_K](https://huggingface.co/anikifoss/GLM-4.5-Air-HQ4_K):

### main

```
    ./llama-sweep-bench \
        --model "$MODEL_PATH" \
        --no-mmap \
        -b 2048 -ub 1024 \
        -c 98304 \
        -ngl 999 \
        --warmup-batch
```

### mqkv

```
    ./llama-sweep-bench \
        --model "$MODEL_PATH" \
        --merge-qkv \
        -b 2048 -ub 1024 \
        -c 98304 \
        -ngl 999 \
        --warmup-batch
```

### results

|   N_KV |    S_PP t/s main   |   S_PP t/s -mqkv   |   S_TG t/s main   |  S_TG t/s -mqkv   |
|--------|----------|----------|----------|----------|
|      0 |  4080.48 |  4070.36 |    93.12 |    94.72 |
|   1024 |  3916.30 |  3842.24 |    92.25 |    93.77 |
|   2048 |  3719.20 |  3616.56 |    90.17 |    91.64 |
|   3072 |  3576.62 |  3408.28 |    88.87 |    90.24 |
|   4096 |  3377.50 |  3249.75 |    87.11 |    88.50 |
|  ... |  ... |   ... |   ... |    ... |
|  93184 |   665.68 |   664.76 |    23.77 |    23.89 |
|  94208 |   660.19 |   659.60 |    23.44 |    23.55 |
|  95232 |   653.36 |   653.19 |    23.09 |    23.21 |
|  96256 |   647.81 |   647.73 |    22.58 |    22.69 |
|  97280 |   641.53 |   641.78 |    22.45 |    22.56 |

---

EDIT:

And here is [Q4_K_M](https://huggingface.co/lmstudio-community/GLM-4.5-Air-GGUF) from **bartowski**:

### main

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  1024 |    256 |      0 |    0.269 |  3803.44 |    2.033 |   125.90 |
|  1024 |    256 |   1024 |    0.284 |  3600.60 |    2.060 |   124.30 |
|  1024 |    256 |   2048 |    0.297 |  3451.05 |    2.128 |   120.31 |
|  1024 |    256 |   3072 |    0.310 |  3298.36 |    2.174 |   117.73 |

### mqkv

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  1024 |    256 |      0 |    0.262 |  3908.56 |    2.038 |   125.62 |
|  1024 |    256 |   1024 |    0.280 |  3662.00 |    2.064 |   124.02 |
|  1024 |    256 |   2048 |    0.293 |  3499.47 |    2.134 |   119.97 |
|  1024 |    256 |   3072 |    0.307 |  3340.37 |    2.179 |   117.46 |

Happy to test more if needed.

---

👤 **ubergarm** commented on **2025-10-30** at **04:31:17**

One more data point running CPU-only on a single socket AMD EPYC 9975 suggesting PR with `--merge-qkv` improving TG around 1% (testing with 96 threads out of 128 physical cores):

This quant has the same quantization for `attn_(q|k|v)` tensors (q8_0 in this case).

<img width="2087" height="1109" alt="sweep-bench-PR878-GLM-4 6-IQ4_K" src="https://github.com/user-attachments/assets/7cae264e-8e95-455d-b621-dfe69ac618dc" />

<details>

<summary>👈 Details</summary>

```bash
model=/mnt/raid/hf/GLM-4.6-GGUF/IQ4_K/GLM-4.6-IQ4_K-00001-of-00005.gguf

numactl -N 1 -m 1 \
./build/bin/llama-sweep-bench \
    --model "$model"\
    --merge-qkv \ # <--- test exactly the same with/without this new argument
    --ctx-size 20480 \
    -ub 4096 -b 4096 \
    --threads 96 \
    --threads-batch 128 \
    --numa numactl \
    --no-mmap \
    --warmup-batch

llama_model_loader: - type  f32:  835 tensors
llama_model_loader: - type q8_0:  652 tensors
llama_model_loader: - type iq4_k:  181 tensors
llama_model_loader: - type iq5_k:   90 tensors
llama_model_loader: - type iq6_k:    1 tensors

...

================================== Created merged qkv blk.0.attn_qkv.weight
================================== Created merged qkv blk.1.attn_qkv.weight
.
.
.
================================== Created merged qkv blk.90.attn_qkv.weight
================================== Created merged qkv blk.91.attn_qkv.weight
```

## without --merge-qkv
|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  4096 |   1024 |      0 |   21.760 |   188.24 |   73.941 |    13.85 |
|  4096 |   1024 |   4096 |   31.503 |   130.02 |   84.554 |    12.11 |
|  4096 |   1024 |   8192 |   36.866 |   111.11 |   94.128 |    10.88 |
|  4096 |   1024 |  12288 |   45.560 |    89.90 |  106.851 |     9.58 |
|  4096 |   1024 |  16384 |   59.197 |    69.19 |  114.232 |     8.96 |

## --merge-qkv (Created merged qkv for all layers)
|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  4096 |   1024 |      0 |   21.014 |   194.92 |   73.064 |    14.02 |
|  4096 |   1024 |   4096 |   31.499 |   130.04 |   83.766 |    12.22 |
|  4096 |   1024 |   8192 |   38.997 |   105.03 |   93.651 |    10.93 |
|  4096 |   1024 |  12288 |   47.621 |    86.01 |  105.914 |     9.67 |
|  4096 |   1024 |  16384 |   57.464 |    71.28 |  112.807 |     9.08 |

</details>

---

👤 **ikawrakow** commented on **2025-10-30** at **05:46:03**

@sousekd

Thanks for testing!

The Bartowski quants use `Q8_0` for `K`, `Q4_K` for `Q` and `Q6_K` for `V`, so no merging was done. Short of downloading the anikifoss model, I don't see a way to check the tensor types in their model (and I was curious to see the tensor types there, given the 30% difference in TG performance for what it appears to be a very similar size model).

---

👤 **ikawrakow** commented on **2025-10-30** at **06:03:48**

@Nexesenex 

> would it be possible to have options for merging only Q & K, K & V

This is of course possible, and I was even considering it while working on the PR. But is it worth it? Performance gains, if any, will be a fraction of what we get with the full QKV merge, which is already very small. But we will triple the number of possible paths to take.

---

👤 **sousekd** commented on **2025-10-30** at **07:35:10**

> Short of downloading the anikifoss model, I don't see a way to check the tensor types in their model (and I was curious to see the tensor types there, given the 30% difference in TG performance for what it appears to be a very similar size model).
> 

@ikawrakow I see @anikifoss has their recipe described on HF, would it provide the details you need?

Otherwise, if you point me at another specific quants, I'll test them.

---

👤 **ikawrakow** commented on **2025-10-30** at **07:47:59**

> @ikawrakow I see @anikifoss has their recipe described on HF, would it provide the details you need?

Oh, sorry, I see it now. I was looking for one of these buttons where you can see the GGUF metadata and the tensor types.

Yes, this model gets the QKV merge. But it uses `bf16` for the shared experts and the output tensor, so that explains the 30% lower TG performance compared to Bartowski `Q4_K_M`.

---

👤 **ikawrakow** commented on **2025-10-30** at **08:15:49**

@sousekd If you are actively using the @anikifoss model, you may want to try the script below (just replace  `$anikifoss_original` and `$anikifoss_new` with corresponding file names. This should give you a very significant boost in TG performance. 

```bash
#!/usr/bin/env bash

custom="
blk\.0\.ffn_down\.weight=q6_0
blk\.0\.ffn_gate\.weight=q6_0
blk\.0\.ffn_up\.weight=q6_0
blk\.46\.nextn\.eh_proj\.weight=q6_0
blk\.46\.nextn\.embed_tokens\.weight=q6_0
blk\.46\.nextn\.shared_head_head\.weight=q6_0

blk\.[0-9]\.attn_k\.weight=q6_0
blk\.[0-9]\.attn_output\.weight=q6_0
blk\.[0-9]\.attn_q\.weight=q6_0
blk\.[0-9]\.attn_v\.weight=q6_0
blk\.[1-3][0-9]\.attn_k\.weight=q6_0
blk\.[1-3][0-9]\.attn_output\.weight=q6_0
blk\.[1-3][0-9]\.attn_q\.weight=q6_0
blk\.[1-3][0-9]\.attn_v\.weight=q6_0
blk\.4[0-6]\.attn_k\.weight=q6_0
blk\.4[0-6]\.attn_output\.weight=q6_0
blk\.4[0-6]\.attn_q\.weight=q6_0
blk\.4[0-6]\.attn_v\.weight=q6_0

blk\.[1-9]\.ffn_down_exps\.weight=q6_0
blk\.[1-9]\.ffn_down_shexp\.weight=q6_0
blk\.[1-9]\.ffn_gate_exps\.weight=q4_K
blk\.[1-9]\.ffn_gate_shexp\.weight=q6_0
blk\.[1-9]\.ffn_up_exps\.weight=q4_K
blk\.[1-9]\.ffn_up_shexp\.weight=q6_0
blk\.[1-3][0-9]\.ffn_down_exps\.weight=q6_0
blk\.[1-3][0-9]\.ffn_down_shexp\.weight=q6_0
blk\.[1-3][0-9]\.ffn_gate_exps\.weight=q4_K
blk\.[1-3][0-9]\.ffn_gate_shexp\.weight=q6_0
blk\.[1-3][0-9]\.ffn_up_exps\.weight=q4_K
blk\.[1-3][0-9]\.ffn_up_shexp\.weight=q6_0
blk\.4[0-6]\.ffn_down_exps\.weight=q6_0
blk\.4[0-6]\.ffn_down_shexp\.weight=q6_0
blk\.4[0-6]\.ffn_gate_exps\.weight=q4_K
blk\.4[0-6]\.ffn_gate_shexp\.weight=q6_0
blk\.4[0-6]\.ffn_up_exps\.weight=q4_K
blk\.4[0-6]\.ffn_up_shexp\.weight=q6_0

output\.weight=q6_0
token_embd\.weight=q6_0
"

custom=$(
  echo "$custom" | grep -v '^#' | \
  sed -Ez 's:\n+:,:g;s:,$::;s:^,::'
)

echo "Running with: -custom-q $custom"

./build/bin/llama-quantize --allow-requantize --custom-q "$custom" $anikifoss_original $anikifoss_new Q4_K
```

---

👤 **Nexesenex** commented on **2025-10-30** at **12:37:27**

> @Nexesenex
> 
> > would it be possible to have options for merging only Q & K, K & V
> 
> This is of course possible, and I was even considering it while working on the PR. But is it worth it? Performance gains, if any, will be a fraction of what we get with the full QKV merge, which is already very small. But we will triple the number of possible paths to take.

Balancing that with your valuable time, it doesn't seem that important indeed.
But, from my comfortable standpoint, any couple of tenth of percent of performance gained multiplies with the other tenths of percent and percents gained with other commits, and on the long run, all those little optimisations which could be dismissed one by one multiply themselves, and give a serious one.
The gain for 2 merged tensors should be around half the gain with 3 merged tensors, if I understand correctly the principle.
So, I would say that you should do it still, if you feel generous.

---

👤 **magikRUKKOLA** commented on **2025-10-30** at **14:26:38**

troll-rig, 12C CPU:

main, GLM-4.6:
```
main: n_kv_max = 73728, n_batch = 8192, n_ubatch = 8192, flash_attn = 1, n_gpu_layers = 99, n_threads = 12, n_threads_batch = 12

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  8192 |   2048 |      0 |   22.750 |   360.09 |  371.612 |     5.51 |
|  8192 |   2048 |   8192 |   27.226 |   300.89 |  398.527 |     5.14 |
|  8192 |   2048 |  16384 |   32.145 |   254.85 |  424.979 |     4.82 |
```

pr-878 (fused_mmad    = 1):
```
main: n_kv_max = 73728, n_batch = 8192, n_ubatch = 8192, flash_attn = 1, n_gpu_layers = 99, n_threads = 12, n_threads_batch = 12

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  8192 |   2048 |      0 |   25.863 |   316.75 |  362.069 |     5.66 |
|  8192 |   2048 |   8192 |   30.704 |   266.80 |  388.849 |     5.27 |
|  8192 |   2048 |  16384 |   35.479 |   230.90 |  414.753 |     4.94 |
```

+~2.5% better decode, yep.

---

👤 **ikawrakow** commented on **2025-10-30** at **16:36:42**

> But, from my comfortable standpoint, any couple of tenth of percent of performance gained multiplies with the other tenths of percent and percents gained with other commits, and on the long run, all those little optimisations which could be dismissed one by one multiply themselves, and give a serious one.

I agree with that, and that's why there are all these recent PRs, each bringing only a minor performance improvement. But for now I think there are better 1% possibilities than merging `K & V` or `Q & K`. I just made [#882](https://github.com/ikawrakow/ik_llama.cpp/issues/882), and the next thing to do is to fuse the `Q` and `K` RoPE operations.