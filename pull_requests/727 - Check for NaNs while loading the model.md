## 🔀 [Pull Request #727](https://github.com/ikawrakow/ik_llama.cpp/pull/727) - Check for NaNs while loading the model.

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/validate_quants_on_load` |
| **Target Branch** | `main` |
| **Created** | 2025-08-25 |
| **Updated** | 2025-08-27 |
| **Merged** | 2025-08-27 |

---

## 📄 Description

Apparently there are quantized models out there that contain NaNs in the model weights. This leads to various asserts at run time. To remedy the situation this PR adds the ability to check for NaNs while loading the model. To avoid unnecessary checks and associated loading delays for models that are known to be good, the check is behind a command line option `-vq` or `--validate-quants`.

There are checks for many quantization types, but the implementation is not yet exhaustive. Here is the list of the currently covered quantization types:
```
GGML_TYPE_IQ2_K
GGML_TYPE_IQ3_K
GGML_TYPE_IQ4_K
GGML_TYPE_IQ5_K
GGML_TYPE_IQ6_K
GGML_TYPE_IQ2_XXS
GGML_TYPE_IQ2_XS
GGML_TYPE_IQ2_S
GGML_TYPE_IQ3_XXS
GGML_TYPE_IQ3_S
GGML_TYPE_IQ4_XS
GGML_TYPE_IQ2_K_R4
GGML_TYPE_IQ3_K_R4
GGML_TYPE_IQ4_K_R4
GGML_TYPE_IQ5_K_R4
GGML_TYPE_IQ2_XXS_R4
GGML_TYPE_IQ2_XS_R4
GGML_TYPE_IQ2_S_R4
GGML_TYPE_IQ3_XXS_R4
GGML_TYPE_IQ3_S_R4
GGML_TYPE_IQ4_XS_R8
GGML_TYPE_IQ2_BN
GGML_TYPE_IQ4_KSS
GGML_TYPE_IQ4_KS
GGML_TYPE_IQ5_KS
GGML_TYPE_IQ2_BN_R4
GGML_TYPE_IQ4_KS_R4
GGML_TYPE_IQ5_KS_R4
GGML_TYPE_IQ1_BN
GGML_TYPE_IQ2_KS
GGML_TYPE_IQ2_KL
GGML_TYPE_IQ3_KS
GGML_TYPE_IQ1_S_R4
GGML_TYPE_IQ1_M_R4
```

### Original PR description

Add a check for NaNs while loading a model.

This is just a quick hack to check if there are NaNs in models where people are reporting issues.

The check is done unconditionally for all tensors stored on the CPU.
It would be better to have an option to turn on/off this check as it does trigger actual loading into RAM in a single thread, which may not really be what we want.

---

## 💬 Conversation

👤 **ubergarm** commented on **2025-08-27** at **17:34:44**

Thanks for this QA feature, I can confirm that it catches the nan blocks in the broken quant `Qwen3-Coder-30B-A3B-Instruct-IQ3_K` we've been discussing. I will use this feature as an additional validation step before releasing quants in the future!

```
llm_load_tensors:        CPU buffer size = 14857.44 MiB
................................................................................................
check_tensor_for_blocks_256_fp16: found 3200 NaN block scales out of 786432 blocks in tensor blk.1.ffn_gate_exps.weight
    there are 3200 NaN block scales for expert 27
check_tensor_for_blocks_256_fp16: found 3200 NaN block scales out of 786432 blocks in tensor blk.1.ffn_up_exps.weight
    there are 3200 NaN block scales for expert 27
Found 2 bad tensors in model
llama_model_load: error loading model: Bad tensors in model
llama_load_model_from_file: failed to load model
llama_init_from_gpt_params: error: failed to load model '/mnt/raid/models/ubergarm/Qwen3-Coder-30B-A3B-Instruct-GGUF/Qwen3-Coder-30B-A3B-Instruct-IQ3_K-OG-bad.gguf'
```