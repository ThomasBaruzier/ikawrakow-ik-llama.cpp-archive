## 📌 [Issue #793](https://github.com/ikawrakow/ik_llama.cpp/issues/793) - Bug: trellis dequantization fails because of INF and NaN values

| **Author** | `binghanc` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-09-24 |
| **Updated** | 2025-09-24 |

---

## 📄 Description

### What happened?

Here are my commands:
```
# if I use F32 and IQ4_KT,  I will also get INF and NaN values
QUANTIZED_GGUF_FILE="/path/to/Qwen3-30B-A3B-Thinking-2507-IQ1_KT.gguf"
DEQUANTIZE_TYPE="BF16" 
DEQUANTIZED_GGUF_FILE="/path/to/Qwen3-30B-A3B-Thinking-2507-IQ1_KT-${DEQUANTIZE_TYPE}.gguf"

./build/bin/llama-quantize \
    --allow-requantize \
    $QUANTIZED_GGUF_FILE \
    $DEQUANTIZED_GGUF_FILE \
    $DEQUANTIZE_TYPE
```
It's expected to dequantize the IQ1_KT model and save the dequantized IQ1_KT model, but the output logs show   errors about INF and NaN values, thus this dequantization process fails.


### Name and Version

main: build = 3884 (6d2e7ca4)
main: built with cc (Ubuntu 13.3.0-6ubuntu2~24.04) 13.3.0 for x86_64-linux-gnu


### What operating system are you seeing the problem on?

Linux

### Relevant log output

```shell
Here are the errror logs:


main: build = 3884 (6d2e7ca4)
main: built with cc (Ubuntu 13.3.0-6ubuntu2~24.04) 13.3.0 for x86_64-linux-gnu
main: quantizing '/path/to/Qwen3-30B-A3B-Thinking-2507-IQ1_KT.gguf' to '/path/to/Qwen3-30B-A3B-Thinking-2507-IQ1_KT-BF16.gguf' as BF16
llama_model_loader: loaded meta data with 39 key-value pairs and 579 tensors from /path/to/Qwen3-30B-A3B-Thinking-2507-IQ1_KT.gguf (version GGUF V3 (latest))
llama_model_loader: Dumping metadata keys/values. Note: KV overrides do not apply in this output.
llama_model_loader: - kv   0:                       general.architecture str              = qwen3moe
llama_model_loader: - kv   1:                               general.type str              = model
llama_model_loader: - kv   2:                               general.name str              = Qwen3 30B A3B Thinking 2507
llama_model_loader: - kv   3:                            general.version str              = 2507
llama_model_loader: - kv   4:                           general.finetune str              = Thinking
llama_model_loader: - kv   5:                           general.basename str              = Qwen3
llama_model_loader: - kv   6:                         general.size_label str              = 30B-A3B
llama_model_loader: - kv   7:                            general.license str              = apache-2.0
llama_model_loader: - kv   8:                       general.license.link str              = https://huggingface.co/Qwen/Qwen3-30B...
llama_model_loader: - kv   9:                               general.tags arr[str,1]       = ["text-generation"]
llama_model_loader: - kv  10:                       qwen3moe.block_count u32              = 48
llama_model_loader: - kv  11:                    qwen3moe.context_length u32              = 262144
llama_model_loader: - kv  12:                  qwen3moe.embedding_length u32              = 2048
llama_model_loader: - kv  13:               qwen3moe.feed_forward_length u32              = 6144
llama_model_loader: - kv  14:              qwen3moe.attention.head_count u32              = 32
llama_model_loader: - kv  15:           qwen3moe.attention.head_count_kv u32              = 4
llama_model_loader: - kv  16:                    qwen3moe.rope.freq_base f32              = 10000000.000000
llama_model_loader: - kv  17:  qwen3moe.attention.layer_norm_rms_epsilon f32              = 0.000001
llama_model_loader: - kv  18:                 qwen3moe.expert_used_count u32              = 8
llama_model_loader: - kv  19:              qwen3moe.attention.key_length u32              = 128
llama_model_loader: - kv  20:            qwen3moe.attention.value_length u32              = 128
llama_model_loader: - kv  21:                          general.file_type u32              = 156
llama_model_loader: - kv  22:                      qwen3moe.expert_count u32              = 128
llama_model_loader: - kv  23:        qwen3moe.expert_feed_forward_length u32              = 768
llama_model_loader: - kv  24:               general.quantization_version u32              = 2
llama_model_loader: - kv  25:                       tokenizer.ggml.model str              = gpt2
llama_model_loader: - kv  26:                         tokenizer.ggml.pre str              = qwen2
llama_model_loader: - kv  27:                      tokenizer.ggml.tokens arr[str,151936]  = ["!", "\"", "#", "$", "%", "&", "'", ...
llama_model_loader: - kv  28:                  tokenizer.ggml.token_type arr[i32,151936]  = [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, ...
llama_model_loader: - kv  29:                      tokenizer.ggml.merges arr[str,151387]  = ["Ġ Ġ", "ĠĠ ĠĠ", "i n", "Ġ t",...
llama_model_loader: - kv  30:                tokenizer.ggml.eos_token_id u32              = 151645
llama_model_loader: - kv  31:            tokenizer.ggml.padding_token_id u32              = 151643
llama_model_loader: - kv  32:                tokenizer.ggml.bos_token_id u32              = 151643
llama_model_loader: - kv  33:               tokenizer.ggml.add_bos_token bool             = false
llama_model_loader: - kv  34:                    tokenizer.chat_template str              = {%- if tools %}\n    {{- '<|im_start|>...
llama_model_loader: - kv  35:                      quantize.imatrix.file str              = /mnt/raid/models/ubergarm/Qwen3-30B-A...
llama_model_loader: - kv  36:                   quantize.imatrix.dataset str              = eaddario-imatrix-corpus-combined-all-...
llama_model_loader: - kv  37:             quantize.imatrix.entries_count i32              = 385
llama_model_loader: - kv  38:              quantize.imatrix.chunks_count i32              = 5110
llama_model_loader: - type  f32:  241 tensors
llama_model_loader: - type iq6_k:    3 tensors
llama_model_loader: - type iq5_ks:   96 tensors
llama_model_loader: - type iq2_kt:   46 tensors
llama_model_loader: - type iq4_kt:  101 tensors
llama_model_loader: - type iq1_kt:   92 tensors
[   1/ 579]                    token_embd.weight - [ 2048, 151936,     1,     1], type = iq4_kt, converting to bf16 .. ggml_validate_row_data: found 12 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 13 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 21 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 13 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 13 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 12 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 8 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 7 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 6 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 17 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 9 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 8 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 16 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 21 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 5 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 8 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 17 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 7 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 18 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 6 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 24 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 11 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 12 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 17 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 11 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 27 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 9 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 10 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 15 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 17 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 13 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 11 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 21 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 13 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 19 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 22 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 8 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 22 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 20 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 18 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 13 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 13 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 11 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 13 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 11 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 11 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 12 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 22 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 13 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 21 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 12 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 9 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 12 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 18 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 29 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 19 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 13 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 10 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 21 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 11 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 10 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 17 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 11 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 8 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 20 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 18 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 12 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 16 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 19 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 22 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 19 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 23 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 12 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 20 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 15 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14322 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 22 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14344 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14638 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 15465 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14681 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14883 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14471 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14561 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14815 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14413 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 15233 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14393 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 15395 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14683 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 15507 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14447 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14521 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14540 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 9237 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14531 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14228 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 15426 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 15303 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 13856 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14748 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14399 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 15431 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14585 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14494 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14423 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14472 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14657 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14275 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14148 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14288 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14456 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 15408 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14878 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14817 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14302 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14496 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14344 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14569 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14406 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 15339 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14449 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14341 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14811 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14456 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 15355 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14447 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14333 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 15438 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14712 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 15342 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14911 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14255 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 18 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14629 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14163 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14587 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14646 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 15419 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14464 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14738 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 15343 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 15434 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14948 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 15278 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 15252 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14849 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14665 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14501 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 15065 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14749 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14362 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14344 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 15391 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14768 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14419 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14566 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 15154 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14804 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14642 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 14421 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 15333 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 2928 infinities in row of 16384 BF16 values
ggml_validate_row_data: found 65 NaNs in row of 16384 BF16 values
ggml_validate_row_data: found 93 NaNs in row of 16384 BF16 values
ggml_validate_row_data: found 46 NaNs in row of 16384 BF16 values
ggml_validate_row_data: found 91 NaNs in row of 16384 BF16 values
ggml_validate_row_data: found 79 NaNs in row of 16384 BF16 values
ggml_validate_row_data: found 65 NaNs in row of 16384 BF16 values
ggml_validate_row_data: found 68 NaNs in row of 16384 BF16 values
ggml_validate_row_data: found 84 NaNs in row of 16384 BF16 values
ggml_validate_row_data: found 106 NaNs in row of 16384 BF16 values
ggml_validate_row_data: found 83 NaNs in row of 16384 BF16 values
ggml_validate_row_data: found 63 NaNs in row of 16384 BF16 values
ggml_validate_row_data: found 100 NaNs in row of 16384 BF16 values
ggml_validate_row_data: found 78 NaNs in row of 16384 BF16 values
ggml_validate_row_data: found 73 NaNs in row of 16384 BF16 values
ggml_validate_row_data: found 119 NaNs in row of 16384 BF16 values
ggml_validate_row_data: found 65 NaNs in row of 16384 BF16 values
ggml_validate_row_data: found 80 NaNs in row of 16384 BF16 values
ggml_validate_row_data: found 74 NaNs in row of 16384 BF16 values
ggml_validate_row_data: found 78 NaNs in row of 16384 BF16 values
ggml_validate_row_data: found 132 NaNs in row of 16384 BF16 values
ggml_validate_row_data: found 82 NaNs in row of 16384 BF16 values
ggml_validate_row_data: found 129 NaNs in row of 16384 BF16 values
ggml_validate_row_data: found 67 NaNs in row of 16384 BF16 values
ggml_validate_row_data: found 75 NaNs in row of 16384 BF16 values
ggml_validate_row_data: found 121 NaNs in row of 16384 BF16 values
ggml_validate_row_data: found 75 NaNs in row of 16384 BF16 values
ggml_validate_row_data: found 103 NaNs in row of 16384 BF16 values
ggml_validate_row_data: found 154 NaNs in row of 16384 BF16 values
ggml_validate_row_data: found 66 NaNs in row of 16384 BF16 values
ggml_validate_row_data: found 113 NaNs in row of 16384 BF16 values
ggml_validate_row_data: found 67 NaNs in row of 16384 BF16 values
ggml_validate_row_data: found 72 NaNs in row of 16384 BF16 values
ggml_validate_row_data: found 86 NaNs in row of 16384 BF16 values
ggml_validate_row_data: found 57 NaNs in row of 16384 BF16 values
ggml_validate_row_data: found 125 NaNs in row of 16384 BF16 values
ggml_validate_row_data: found 76 NaNs in row of 16384 BF16 values
ggml_validate_row_data: found 77 NaNs in row of 16384 BF16 values
ggml_validate_row_data: found 83 NaNs in row of 16384 BF16 values
ggml_validate_row_data: found 92 NaNs in row of 16384 BF16 values
ggml_validate_row_data: found 116 NaNs in row of 16384 BF16 values
ggml_validate_row_data: found 69 NaNs in row of 16384 BF16 values
ggml_validate_row_data: found 117 NaNs in row of 16384 BF16 values
ggml_validate_row_data: found 80 NaNs in row of 16384 BF16 values
ggml_validate_row_data: found 74 NaNs in row of 16384 BF16 values
ggml_validate_row_data: found 103 NaNs in row of 16384 BF16 values
ggml_validate_row_data: found 108 NaNs in row of 16384 BF16 values
ggml_validate_row_data: found 62 NaNs in row of 16384 BF16 values
ggml_validate_row_data: found 136 NaNs in row of 16384 BF16 values
ggml_validate_row_data: found 67 NaNs in row of 16384 BF16 values
ggml_validate_row_data: found 118 NaNs in row of 16384 BF16 values
ggml_validate_row_data: found 122 NaNs in row of 16384 BF16 values
ggml_validate_row_data: found 49 NaNs in row of 16384 BF16 values
ggml_validate_row_data: found 68 NaNs in row of 16384 BF16 values
llama_model_quantize: failed to quantize: quantized data validation failed
```

---

## 💬 Conversation

👤 **ikawrakow** commented on **2025-09-24** at **05:24:07**

Where did the quantized model come from? Did you create it yourself or did you download it from somewhere? If you created yourself, what was the `ik_llama.cpp` version used to quantize?

---

👤 **binghanc** commented on **2025-09-24** at **05:38:27**

Thanks for your reply!
The quantized model is from https://huggingface.co/ubergarm/Qwen3-30B-A3B-Thinking-2507-GGUF
The `ik_llama.cpp` version is main branch, below is the commit info: https://github.com/ikawrakow/ik_llama.cpp/pull/771
```
commit 6d2e7ca42d72719f5d5829d9fd4d6614ddbc57f6 (HEAD -> main, origin/main, origin/HEAD)
Author: firecoperana <xuqiaowei1124@gmail.com>
Date:   Sat Sep 13 00:51:40 2025 -0500

    Deepseek V3.1 native tool calling support (OpenAI Style) (#771)
```

---

👤 **ikawrakow** commented on **2025-09-24** at **06:48:58**

This model seems to have been created before the fixes in PR [#735](https://github.com/ikawrakow/ik_llama.cpp/issues/735), so it needs to be recreated to avoid NaN/infinite values.

Also the conversion stops after processing the token embedding tensor, so we don't know if it contains NaN/infinite values in other tensors. Quantization of the token embedding tensor for models with large vocabularies is tricky because many of the embeddings contain a lot of zeros, so I wouldn't recommend using a trellis quant for the token embedding tensor.

cc: @ubergarm

In practice the model may still work just fine if none of the tokens containing invalid embeddings appear in the context. 

What is the purpose of converting the quantized model to `bf16` ?

---

👤 **binghanc** commented on **2025-09-24** at **08:03:33**

# Brief Summary 
I followed your sugggestions but still got INF and NaN values in other weight tensors.


# My Purpose
My purpose of converting the quantized model to bf16 is to **evaluate the quantized_model's quality in other frameworks** (for example, pytorch, tensorrt-llm), especially to evaluate the trellis quantization format. Overall, I actually want to **get a fake-quantized model in huggingface format, then I can load it in many frameworks**.

Now I am trying to do like this:
- Step 1: `quantized_model.gguf`->`dequantized_model.gguf`, now I am using `--allow-requantize` to do this step
- Step 2: `dequantized_model.gguf` -> `dequantized_model.safetensors`, now I wrote a script by myself to do this step.

Thank you for your outstanding work, and if you have a better way to achieve my purpose, it will be of great help to me.

# Quantization Step

With the below commands, I followed your suggestions:  ①quantize the model by myself, ②do not quantize token embeddings, and the result is just as expected.

```

python convert_hf_to_gguf.py \
    --outfile /path/to/Qwen3-0.6B-bf16.gguf \ 
    --outtype bf16 \
    /path/to/Qwen3-0.6B # this model is from https://huggingface.co/Qwen/Qwen3-0.6B



./build/bin/llama-quantize \
    --leave-output-tensor \
    --custom-q "token_embd.*=bf16,output.*=bf16" \
    /path/to/Qwen3-0.6B-bf16.gguf \
    /path/to/Qwen3-0.6B-IQ4_KT.gguf \
    IQ4_KT

```


# Dequantization Step and the Error Logs

```
./build/bin/llama-quantize \
    --allow-requantize \
    /path/to/Qwen3-0.6B-IQ4_KT.gguf`\
    /path/to/Qwen3-0.6B-IQ4_KT-bf16.gguf \
    bf16

```

Then I try to dequantize the trellis_quantized_model.gguf to bf16 with the above commands and got error logs(INF and NaN) as below.

```
main: build = 3884 (6d2e7ca4)
main: built with cc (Ubuntu 13.3.0-6ubuntu2~24.04) 13.3.0 for x86_64-linux-gnu
main: quantizing '/path/to/Qwen3-0.6B-IQ4_KT.gguf' to '/path/to/Qwen3-0.6B-IQ4_KT-bf16.gguf' as BF16
llama_model_loader: loaded meta data with 34 key-value pairs and 311 tensors from /path/to/Qwen3-0.6B-IQ4_KT.gguf (version GGUF V3 (latest))
llama_model_loader: Dumping metadata keys/values. Note: KV overrides do not apply in this output.
llama_model_loader: - kv   0:                       general.architecture str              = qwen3
llama_model_loader: - kv   1:                               general.type str              = model
llama_model_loader: - kv   2:                               general.name str              = Qwen3 0.6B
llama_model_loader: - kv   3:                           general.basename str              = Qwen3
llama_model_loader: - kv   4:                         general.size_label str              = 0.6B
llama_model_loader: - kv   5:                            general.license str              = apache-2.0
llama_model_loader: - kv   6:                       general.license.link str              = https://huggingface.co/Qwen/Qwen3-0.6...
llama_model_loader: - kv   7:                   general.base_model.count u32              = 1
llama_model_loader: - kv   8:                  general.base_model.0.name str              = Qwen3 0.6B Base
llama_model_loader: - kv   9:          general.base_model.0.organization str              = Qwen
llama_model_loader: - kv  10:              general.base_model.0.repo_url str              = https://huggingface.co/Qwen/Qwen3-0.6...
llama_model_loader: - kv  11:                               general.tags arr[str,1]       = ["text-generation"]
llama_model_loader: - kv  12:                          qwen3.block_count u32              = 28
llama_model_loader: - kv  13:                       qwen3.context_length u32              = 40960
llama_model_loader: - kv  14:                     qwen3.embedding_length u32              = 1024
llama_model_loader: - kv  15:                  qwen3.feed_forward_length u32              = 3072
llama_model_loader: - kv  16:                 qwen3.attention.head_count u32              = 16
llama_model_loader: - kv  17:              qwen3.attention.head_count_kv u32              = 8
llama_model_loader: - kv  18:                       qwen3.rope.freq_base f32              = 1000000.000000
llama_model_loader: - kv  19:     qwen3.attention.layer_norm_rms_epsilon f32              = 0.000001
llama_model_loader: - kv  20:                 qwen3.attention.key_length u32              = 128
llama_model_loader: - kv  21:               qwen3.attention.value_length u32              = 128
llama_model_loader: - kv  22:                          general.file_type u32              = 153
llama_model_loader: - kv  23:                       tokenizer.ggml.model str              = gpt2
llama_model_loader: - kv  24:                         tokenizer.ggml.pre str              = qwen2
llama_model_loader: - kv  25:                      tokenizer.ggml.tokens arr[str,151936]  = ["!", "\"", "#", "$", "%", "&", "'", ...
llama_model_loader: - kv  26:                  tokenizer.ggml.token_type arr[i32,151936]  = [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, ...
llama_model_loader: - kv  27:                      tokenizer.ggml.merges arr[str,151387]  = ["Ġ Ġ", "ĠĠ ĠĠ", "i n", "Ġ t",...
llama_model_loader: - kv  28:                tokenizer.ggml.eos_token_id u32              = 151645
llama_model_loader: - kv  29:            tokenizer.ggml.padding_token_id u32              = 151643
llama_model_loader: - kv  30:                tokenizer.ggml.bos_token_id u32              = 151643
llama_model_loader: - kv  31:               tokenizer.ggml.add_bos_token bool             = false
llama_model_loader: - kv  32:                    tokenizer.chat_template str              = {%- if tools %}\n    {{- '<|im_start|>...
llama_model_loader: - kv  33:               general.quantization_version u32              = 2
llama_model_loader: - type  f32:  113 tensors
llama_model_loader: - type bf16:   30 tensors
llama_model_loader: - type iq4_k:   28 tensors
llama_model_loader: - type iq4_kt:  140 tensors
[   1/ 311]                        **output.weight - [ 1024, 151936,     1,     1], type =   bf16**, size =  296.750 MB
[   2/ 311]                    **token_embd.weight - [ 1024, 151936,     1,     1], type =   bf16**, size =  296.750 MB
[   3/ 311]               blk.0.attn_norm.weight - [ 1024,     1,     1,     1], type =    f32, size =    0.004 MB
[   4/ 311]                blk.0.ffn_down.weight - [ 3072,  1024,     1,     1], type = iq4_kt, converting to bf16 .. ggml_validate_row_data: found 5577 infinities in row of 18432 BF16 values
ggml_validate_row_data: found 2756 infinities in row of 18432 BF16 values
ggml_validate_row_data: found 4608 NaNs in row of 18432 BF16 values
ggml_validate_row_data: found 9216 NaNs in row of 18432 BF16 values
ggml_validate_row_data: found 8376 infinities in row of 18432 BF16 values
ggml_validate_row_data: found 97 NaNs in row of 18432 BF16 values
ggml_validate_row_data: found 55 NaNs in row of 18432 BF16 values
ggml_validate_row_data: found 58 NaNs in row of 18432 BF16 values
ggml_validate_row_data: found 98 NaNs in row of 18432 BF16 values
ggml_validate_row_data: found 149 NaNs in row of 18432 BF16 values
ggml_validate_row_data: found 11905 infinities in row of 18432 BF16 values
ggml_validate_row_data: found 140 NaNs in row of 18432 BF16 values
ggml_validate_row_data: found 135 NaNs in row of 18432 BF16 values
ggml_validate_row_data: found 11571 infinities in row of 18432 BF16 values
ggml_validate_row_data: found 6537 infinities in row of 18432 BF16 values
ggml_validate_row_data: found 3063 infinities in row of 18432 BF16 values
ggml_validate_row_data: found 5536 infinities in row of 18432 BF16 values
ggml_validate_row_data: found 8660 infinities in row of 18432 BF16 values
ggml_validate_row_data: found 3773 infinities in row of 18432 BF16 values
ggml_validate_row_data: found 7601 infinities in row of 18432 BF16 values
ggml_validate_row_data: found 10899 infinities in row of 18432 BF16 values
llama_model_quantize: failed to quantize: quantized data validation failed
main: failed to quantize model from '/path/to/Qwen3-0.6B-IQ4_KT.gguf'

```

---

👤 **ikawrakow** commented on **2025-09-24** at **08:32:53**

Oh, I see. It looks like the issue is in the `iq4_kt -> bf16` conversion. This part of the code assumes contiguous quantization blocks, which is not true for the trellis quants (and several other `ik_llama.cpp` quantization types), which contain per tensor row scales.

Let me see if I can fix it.

---

👤 **ikawrakow** commented on **2025-09-24** at **08:54:08**

Try PR [#795](https://github.com/ikawrakow/ik_llama.cpp/issues/795) and let me know if it works.

Hopefully it now also works for `Qwen3-30B-A3B-Thinking-2507-IQ1_KT.gguf`.

---

👤 **binghanc** commented on **2025-09-24** at **10:00:56**

# Brief Summary
I think that PR [[#795](https://github.com/ikawrakow/ik_llama.cpp/issues/795)](https://github.com/ikawrakow/ik_llama.cpp/pull/795) fixs the errors about INF and NaN values in trellis dequantization!  
Thank you for your quick response and great help!

# Validation Results

```
./build/bin/llama-perplexity \
    -m $GGUF_MODEL_PATH \
    -f /path/to/wikitext-2-raw/wiki.test.raw \
    -ngl 999

```

Eval ppl using the above commands, and I got the below results 
-  Qwen3-0.6B
    - Qwen3-0.6B-bf16.gguf: Final estimate: PPL = 21.8983 +/- 0.19159
    - Qwen3-0.6B-IQ4_KT.gguf: Final estimate: PPL = 28.9232 +/- 0.26099
    - Qwen3-0.6B-IQ4_KT-bf16.gguf (dequantized): Final estimate: PPL = 28.8638 +/- 0.26026

 
-  Qwen3-30B-A3B-Thinking-2507
    - [Qwen3-30B-A3B-Thinking-2507-IQ4_KT.gguf](https://huggingface.co/ubergarm/Qwen3-30B-A3B-Thinking-2507-GGUF/blob/main/Qwen3-30B-A3B-Thinking-2507-IQ4_KT.gguf)：Final estimate: PPL = 7.5776 +/- 0.05289
    - Qwen3-30B-A3B-Thinking-2507-IQ4_KT-bf16.gguf (dequantized)：Final estimate: PPL = 7.4890 +/- 0.05220

---

👤 **ikawrakow** commented on **2025-09-24** at **10:45:49**

@ubergarm  False alarm, it was the `iqX_kt -> f32` conversion when re-quantizing causing the Inf/Nan's.

---

👤 **ubergarm** commented on **2025-09-24** at **13:55:12**

Interesting concept to "dequantize" back to bf16 for comparing in different ecosystems e.g. pytorch, vllm...

> I wouldn't recommend using a trellis quant for the token embedding tensor.

Yeah I keep it simple with iq4_k or iq6_k mostly for token embedding and final output lately even on the trellis quants.

Thanks for this new fix/feature!