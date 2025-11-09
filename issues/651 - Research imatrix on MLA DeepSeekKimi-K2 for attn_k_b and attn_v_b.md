## 📌 [Issue #651](https://github.com/ikawrakow/ik_llama.cpp/issues/651) - Research: imatrix on MLA DeepSeek/Kimi-K2 for `attn_k_b` and `attn_v_b`

| **Author** | `ubergarm` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-07-26 |
| **Updated** | 2025-08-22 |

---

## 📄 Description

### Previous existing literature and research

We've had some discussions on how to use imatrix on DeepSeek/Kimi-K2 models with MLA to make sure it applies to `attn_k_b` and `attn_v_b` tensors.

When using an imatrix to quantize these messages are expected:
```
====== llama_model_quantize_internal: did not find weights for token_embd.weight
====== llama_model_quantize_internal: did not find weights for blk.0.attn_kv_b.weight
```

This is fine as the token_embd does not use imatrix data, and the strategy is to set `attn_kv_b` always to Q8_0 as it is only used for PP assuming end user mode of `-mla 3`.

However the following messages *not* okay and mean there maybe was an issue with how imatrix data was collected:
```bash
====== llama_model_quantize_internal: did not find weights for blk.0.attn_k_b.weight
====== llama_model_quantize_internal: did not find weights for blk.0.attn_v_b.weight
```

Given these tensors are good to quantize to speed up TG, ideally there should be imatrix data there.

In the past I had mistakenly left off `-mla 1` while creating imatrix and so was missing the data for `attn_k_b` and `attn_v_b` oops!

More recently with Kimi-K2 I *did* use `-mla 1` but still was missing data for both tensors. One tensors seems because it is named `attn_k_b.weight (reshaped)`. And `attn_v_b` does not appear in the verbose logs. For now I am just quantizing `attn_kv_b` as designed, but now also `attn_k_b`, and `attn_v_b` to Q8_0 on Kimi-K2-Instruct models to preserve perplexity at the cost of some TG speed.

See attached log files, too long to add into details fold:

[imat-kimi-no-mla.log](https://github.com/user-attachments/files/21441623/imat-kimi-no-mla.log)

[imat-kimi-mla-1.log](https://github.com/user-attachments/files/21441622/imat-kimi-mla-1.log)

A few things I can check would be:

### 1. Try it on DeepSeek-V2-Lite
So I just tried llama-imatrix on this model and it seems to use that `attn_k_b.weight (reshaped)` name and I don't ever see `attn_v_b`, though when actually quantizing it doesn't throw a message about missing `attn_v_b`.

```
# llama-quantize
[  11/ 431]                blk.0.attn_k_b.weight - [  128,  8192,     1,     1], type =   bf16, Using custom type q4_0 for tensor blk.0.attn_k_b.weight
====== llama_model_quantize_internal: did not find weights for blk.0.attn_k_b.weight
converting to q4_0 .. size =     2.00 MiB ->     0.56 MiB
[  12/ 431]                blk.0.attn_v_b.weight - [  512,  2048,     1,     1], type =   bf16, Using custom type q4_0 for tensor blk.0.attn_v_b.weight
converting to q4_0 .. size =     2.00 MiB ->     0.56 MiB
```

```
CUDA_VISIBLE_DEVICES="0" \
./build/bin/llama-imatrix \
    -mla 1 \
    --verbosity 2 \
    --layer-similarity \
    -m /mnt/raid/models/ubergarm/DeepSeek-V2-Lite-GGUF/DeepSeek-V2-Lite-64x1.6B-BF16.gguf \
    -f ubergarm-imatrix-calibration-corpus-v02.txt \
    -o /tmp/imatrix-tmp.dat \
    -ngl 99 \
    --ctx-size 512 \
    --threads 1

...

collect_imatrix[1]:     blk.20.ffn_down_shexp.weight, MUL_MAT,  2816 x   512, 0
collect_imatrix[1]:      blk.21.attn_kv_a_mqa.weight, MUL_MAT,  2048 x   512, 0
collect_imatrix[1]:             blk.21.attn_q.weight, MUL_MAT,  2048 x   512, 0
collect_imatrix[1]: blk.21.attn_k_b.weight (reshaped), MUL_MAT,   128 x   512, 0
collect_imatrix[1]:        blk.21.attn_output.weight, MUL_MAT,  2048 x   512, 0
collect_imatrix[1]:       blk.21.ffn_gate_inp.weight, MUL_MAT,  2048 x   512, 0
collect_imatrix[1]:      blk.21.ffn_gate_exps.weight, MUL_MAT_ID,  2048 x   512, 0
collect_imatrix[1]:        blk.21.ffn_up_exps.weight, MUL_MAT_ID,  2048 x   512, 0
collect_imatrix[1]:      blk.21.ffn_down_exps.weight, MUL_MAT_ID,  1408 x   512, 0
collect_imatrix[1]:     blk.21.ffn_gate_shexp.weight, MUL_MAT,  2048 x   512, 0
collect_imatrix[1]:       blk.21.ffn_up_shexp.weight, MUL_MAT,  2048 x   512, 0
collect_imatrix[1]:     blk.21.ffn_down_shexp.weight, MUL_MAT,  2816 x   512, 0
collect_imatrix[1]:      blk.22.attn_kv_a_mqa.weight, MUL_MAT,  2048 x   512, 0
```

---

We've discussed this topic across a number of discussions and PRs hah, here is most recent relevent comments:

## References
* https://github.com/ikawrakow/ik_llama.cpp/pull/642#issuecomment-3109818995
* https://github.com/ikawrakow/ik_llama.cpp/issues/601#issuecomment-3070185792

---

## 💬 Conversation

👤 **Lissanro** commented on **2025-07-30** at **20:57:38**

I recently built imatrix for Kimi K2 and this is the full log of the process:
https://dragon.studio/2025/07/kimi-k2-fp8-to-iq4_xs-conversion-log.txt 

Relevant to this issue:

```
 cat /var/tmp/kimi-k2-fp8-to-iq4_xs-conversion-log.txt | grep blk.0.attn_kv_b.weight
INFO:hf-to-gguf:blk.0.attn_kv_b.weight,       torch.bfloat16 --> BF16, shape = {512, 16384}
[   9/1157]               blk.0.attn_kv_b.weight - [  512, 16384,     1,     1], type =   bf16, converting to q8_0 .. size =    16.00 MiB ->     8.50 MiB
[   9/1157]               blk.0.attn_kv_b.weight - [  512, 16384,     1,     1], type =   bf16, converting to q8_0 .. size =    16.00 MiB ->     8.50 MiB
```

If I do `grep attn_kv_b.weight` output is much longer, but no "did not find weights" errors. However, I do have the error for `blk.0.attn_k_b.weight`:

```
> cat /var/tmp/kimi-k2-fp8-to-iq4_xs-conversion-log.txt | grep blk.0.attn_k_b.weight
INFO:hf-to-gguf:blk.0.attn_k_b.weight,        torch.bfloat16 --> BF16, shape = {128, 32768}
[  10/1157]                blk.0.attn_k_b.weight - [  128, 32768,     1,     1], type =   bf16, converting to q8_0 .. size =     8.00 MiB ->     4.25 MiB
[  10/1157]                blk.0.attn_k_b.weight - [  128, 32768,     1,     1], type =   bf16, 
====== llama_model_quantize_internal: did not find weights for blk.0.attn_k_b.weight
```

But not for `blk.0.attn_v_b.weight`:

```
> cat /var/tmp/kimi-k2-fp8-to-iq4_xs-conversion-log.txt | grep blk.0.attn_v_b.weight
INFO:hf-to-gguf:blk.0.attn_v_b.weight,        torch.bfloat16 --> BF16, shape = {512, 8192}
[  11/1157]                blk.0.attn_v_b.weight - [  512,  8192,     1,     1], type =   bf16, converting to q6_K .. size =     8.00 MiB ->     3.28 MiB
[  11/1157]                blk.0.attn_v_b.weight - [  512,  8192,     1,     1], type =   bf16, converting to iq4_xs .. size =     8.00 MiB ->     2.12 MiB
```

All not found weights in my case were:

```
> cat /var/tmp/kimi-k2-fp8-to-iq4_xs-conversion-log.txt | grep "did not find weights for" 
====== llama_model_quantize_internal: did not find weights for token_embd.weight
====== llama_model_quantize_internal: did not find weights for blk.0.attn_k_b.weight
====== llama_model_quantize_internal: did not find weights for blk.9.attn_k_b.weight
====== llama_model_quantize_internal: did not find weights for blk.10.attn_k_b.weight
====== llama_model_quantize_internal: did not find weights for blk.11.attn_k_b.weight
====== llama_model_quantize_internal: did not find weights for blk.12.attn_k_b.weight
====== llama_model_quantize_internal: did not find weights for blk.13.attn_k_b.weight
====== llama_model_quantize_internal: did not find weights for blk.14.attn_k_b.weight
====== llama_model_quantize_internal: did not find weights for blk.15.attn_k_b.weight
====== llama_model_quantize_internal: did not find weights for blk.16.attn_k_b.weight
====== llama_model_quantize_internal: did not find weights for blk.17.attn_k_b.weight
====== llama_model_quantize_internal: did not find weights for blk.18.attn_k_b.weight
====== llama_model_quantize_internal: did not find weights for blk.1.attn_k_b.weight
====== llama_model_quantize_internal: did not find weights for blk.19.attn_k_b.weight
====== llama_model_quantize_internal: did not find weights for blk.20.attn_k_b.weight
====== llama_model_quantize_internal: did not find weights for blk.21.attn_k_b.weight
====== llama_model_quantize_internal: did not find weights for blk.22.attn_k_b.weight
====== llama_model_quantize_internal: did not find weights for blk.23.attn_k_b.weight
====== llama_model_quantize_internal: did not find weights for blk.24.attn_k_b.weight
====== llama_model_quantize_internal: did not find weights for blk.25.attn_k_b.weight
====== llama_model_quantize_internal: did not find weights for blk.26.attn_k_b.weight
====== llama_model_quantize_internal: did not find weights for blk.27.attn_k_b.weight
====== llama_model_quantize_internal: did not find weights for blk.28.attn_k_b.weight
====== llama_model_quantize_internal: did not find weights for blk.2.attn_k_b.weight
====== llama_model_quantize_internal: did not find weights for blk.29.attn_k_b.weight
====== llama_model_quantize_internal: did not find weights for blk.30.attn_k_b.weight
====== llama_model_quantize_internal: did not find weights for blk.31.attn_k_b.weight
====== llama_model_quantize_internal: did not find weights for blk.32.attn_k_b.weight
====== llama_model_quantize_internal: did not find weights for blk.33.attn_k_b.weight
====== llama_model_quantize_internal: did not find weights for blk.34.attn_k_b.weight
====== llama_model_quantize_internal: did not find weights for blk.35.attn_k_b.weight
====== llama_model_quantize_internal: did not find weights for blk.36.attn_k_b.weight
====== llama_model_quantize_internal: did not find weights for blk.37.attn_k_b.weight
====== llama_model_quantize_internal: did not find weights for blk.38.attn_k_b.weight
====== llama_model_quantize_internal: did not find weights for blk.3.attn_k_b.weight
====== llama_model_quantize_internal: did not find weights for blk.39.attn_k_b.weight
====== llama_model_quantize_internal: did not find weights for blk.40.attn_k_b.weight
====== llama_model_quantize_internal: did not find weights for blk.41.attn_k_b.weight
====== llama_model_quantize_internal: did not find weights for blk.42.attn_k_b.weight
====== llama_model_quantize_internal: did not find weights for blk.43.attn_k_b.weight
====== llama_model_quantize_internal: did not find weights for blk.44.attn_k_b.weight
====== llama_model_quantize_internal: did not find weights for blk.45.attn_k_b.weight
====== llama_model_quantize_internal: did not find weights for blk.46.attn_k_b.weight
====== llama_model_quantize_internal: did not find weights for blk.47.attn_k_b.weight
====== llama_model_quantize_internal: did not find weights for blk.48.attn_k_b.weight
====== llama_model_quantize_internal: did not find weights for blk.4.attn_k_b.weight
====== llama_model_quantize_internal: did not find weights for blk.49.attn_k_b.weight
====== llama_model_quantize_internal: did not find weights for blk.50.attn_k_b.weight
====== llama_model_quantize_internal: did not find weights for blk.51.attn_k_b.weight
====== llama_model_quantize_internal: did not find weights for blk.52.attn_k_b.weight
====== llama_model_quantize_internal: did not find weights for blk.53.attn_k_b.weight
====== llama_model_quantize_internal: did not find weights for blk.54.attn_k_b.weight
====== llama_model_quantize_internal: did not find weights for blk.55.attn_k_b.weight
====== llama_model_quantize_internal: did not find weights for blk.56.attn_k_b.weight
====== llama_model_quantize_internal: did not find weights for blk.57.attn_k_b.weight
====== llama_model_quantize_internal: did not find weights for blk.58.attn_k_b.weight
====== llama_model_quantize_internal: did not find weights for blk.5.attn_k_b.weight
====== llama_model_quantize_internal: did not find weights for blk.59.attn_k_b.weight
====== llama_model_quantize_internal: did not find weights for output.weight
====== llama_model_quantize_internal: did not find weights for blk.60.attn_k_b.weight
====== llama_model_quantize_internal: did not find weights for blk.6.attn_k_b.weight
====== llama_model_quantize_internal: did not find weights for blk.7.attn_k_b.weight
====== llama_model_quantize_internal: did not find weights for blk.8.attn_k_b.weight
```

Since you mentioned `token_embd.weight` is not used in imatrix data, I am assuming that is fine, and perhaps it is normal for `output.weight` too. So if we do not count those, only `attn_k_b.weight` seems to be missing in my case.

I have modified fp8_cast_bf16.py script to run CPU, not only replacing "cuda" with "cpu", but also replacing CUDA-specific garbage collection, since otherwise the script would crash on CPU. Here is CPU-enabled version of the script: https://dragon.studio/2025/07/fp8_cast_bf16.py

Then, I run the following commands to convert FP8 to BF16, then to Q6_K for imatrix calibration, and based on it the final IQ4_XS quant:

```
python3 ~/pkgs/DeepSeek-V3/inference/fp8_cast_bf16.py \
--input-fp8-hf-path /mnt/neuro/models/Kimi-K2-Instruct \
--output-bf16-hf-path /mnt/neuro/models/Kimi-K2-Instruct-BF16/ \
&& \
cp -an /mnt/neuro/models/Kimi-K2-Instruct/* /mnt/neuro/models/Kimi-K2-Instruct-BF16/ \
&& \
cd /mnt/neuro/models/Kimi-K2-Instruct/ \
&& \
python3 /home/lissanro/pkgs/ik_llama.cpp/convert_hf_to_gguf.py \
--outtype bf16 \
--outfile /mnt/neuro/models/Kimi-K2-Instruct/Kimi-K2-Instruct-BF16.gguf \
/mnt/neuro/models/Kimi-K2-Instruct-BF16 \
&& \
rm -rf /mnt/neuro/models/Kimi-K2-Instruct-BF16 \
&& \
~/pkgs/ik_llama.cpp/build/bin/llama-quantize \
/mnt/neuro/models/Kimi-K2-Instruct/Kimi-K2-Instruct-BF16.gguf \
/mnt/neuro/models/Kimi-K2-Instruct/Kimi-K2-Instruct-Q6_K.gguf \
Q6_K \
&& \
numactl --cpunodebind=0 --interleave=all ~/pkgs/ik_llama.cpp/build/bin/llama-imatrix \
-m /mnt/neuro/models/Kimi-K2-Instruct/Kimi-K2-Instruct-Q6_K.gguf \
-f ~/pkgs/imatrix/compact-imatrix.txt \
--n-gpu-layers 62 \
--tensor-split 25,23,26,26 \
-ot "ffn_down_exps=CPU, ffn_up_exps=CPU, gate_exps=CPU" \
--threads 64 -b 4096 -ub 4096 \
&& \
~/pkgs/ik_llama.cpp/build/bin/llama-quantize \
--imatrix imatrix.dat \
/mnt/neuro/models/Kimi-K2-Instruct/Kimi-K2-Instruct-BF16.gguf \
/mnt/neuro/models/Kimi-K2-Instruct/Kimi-K2-Instruct-IQ4_XS.gguf \
IQ4_XS
```

* * *

To make things simpler for myself in the future, I wrote a Python script:
https://dragon.studio/2025/07/fp8-to-gguf
Which does all of the above and more, and also includes fp8-to-bf16 code that can run either on CPU or CUDA, and has many other options (running without options will print available arguments). But this script is currently work in progress and not fully tested yet, so if someone decides to give it a try, please keep that in mind. Some example usage:

### Usage Scenarios:

1. **Direct Quantization (No imatrix)**:
```bash
fp8_to_gguf --input-fp8-dir /path/to/model --final-quant-type Q4_K_M
```

2. **With imatrix Generation**:
```bash
fp8_to_gguf --input-fp8-dir /path/to/model \
  --imatrix-text calibration.txt \
  --calib-quant-type Q6_K \
  --final-quant-type IQ4_XS
```

3. **With Existing imatrix**:
```bash
fp8_to_gguf --input-fp8-dir /path/to/model \
  --imatrix existing_imatrix.dat \
  --final-quant-type IQ3_XXS
```

In case the script does not work as expected, manually doing the commands I shared above should yield the same result. It seems these steps resulted in less tensors having the "did not find weights" error, but I could not figure out how to solve it for `attn_k_b.weight` tensors yet.

---

👤 **ubergarm** commented on **2025-08-01** at **04:19:11**

Well here we are again, I'm trying to quantize https://huggingface.co/ubergarm/cogito-v2-preview-deepseek-671B-MoE-GGUF/. It is basically a deepseek 671B moe architecture and other than an issue in the original config.json after removing this line which is not in the original deepseek models:

```bash
$ cat config.json | grep head_
  "head_dim": 64,
```

Anyway here is the imatrix command and logs which seem to have that renamed tensor again:


<details>

<summary>👈 Details</summary>

```bash
#!/usr/bin/env bash

# Notes:
# https://github.com/ikawrakow/ik_llama.cpp/issues/651

# echo 0 | sudo tee /proc/sys/kernel/numa_balancing
# sudo sync; echo 3 | sudo tee /proc/sys/vm/drop_caches

model=/mnt/raid/models/ubergarm/cogito-v2-preview-deepseek-671B-MoE-GGUF/cogito-v2-preview-deepseek-671B-MoE-Q8_0.gguf

#numactl --interleave=all \
numactl -N 1 -m 1 \
./build/bin/llama-imatrix \
    -mla 1 \
    -m "$model" \
    -f ubergarm-imatrix-calibration-corpus-v02.txt \
    -o /mnt/raid/models/ubergarm/cogito-v2-preview-deepseek-671B-MoE-GGUF/imatrix-cogito-v2-preview-deepseek-671B-MoE-Q8_0.dat \
    --verbosity 2 \
    --ctx-size 512 \
    --layer-similarity \
    --numa numactl \
    -ub 4096 -b 4096 \
    --threads 128 \
    --threads-batch 192

collect_imatrix[1]:      blk.7.ffn_down_shexp.weight, MUL_MAT,  2048 x   512, 0
collect_imatrix[1]:       blk.8.attn_kv_a_mqa.weight, MUL_MAT,  7168 x   512, 0
collect_imatrix[1]:            blk.8.attn_q_a.weight, MUL_MAT,  7168 x   512, 0
collect_imatrix[1]:            blk.8.attn_q_b.weight, MUL_MAT,  1536 x   512, 0
collect_imatrix[1]: blk.8.attn_k_b.weight (reshaped), MUL_MAT,   128 x   512, 0
collect_imatrix[1]:         blk.8.attn_output.weight, MUL_MAT, 16384 x   512, 0
collect_imatrix[1]:        blk.8.ffn_gate_inp.weight, MUL_MAT,  7168 x   512, 0
collect_imatrix[1]:       blk.8.ffn_gate_exps.weight, MUL_MAT_ID,  7168 x   512, 0
collect_imatrix[1]:         blk.8.ffn_up_exps.weight, MUL_MAT_ID,  7168 x   512, 0
collect_imatrix[1]:       blk.8.ffn_down_exps.weight, MUL_MAT_ID,  2048 x   512, 0
collect_imatrix[1]:      blk.8.ffn_gate_shexp.weight, MUL_MAT,  7168 x   512, 0
collect_imatrix[1]:        blk.8.ffn_up_shexp.weight, MUL_MAT,  7168 x   512, 0
collect_imatrix[1]:      blk.8.ffn_down_shexp.weight, MUL_MAT,  2048 x   512, 0
collect_imatrix[1]:       blk.9.attn_kv_a_mqa.weight, MUL_MAT,  7168 x   512, 0
```

</details>

I'll try some more combinations of `mla` maybe lol... I used the `convert_hf_to_gguf.py` from this repo on the original models released bf16 safetensors.

---

👤 **ikawrakow** commented on **2025-08-01** at **05:21:50**

What happens if you add -fa ?

---

👤 **ubergarm** commented on **2025-08-01** at **05:40:19**

<details>

<summary>👈 -fa -mla 1</summary>

```bash
model=/mnt/raid/models/ubergarm/cogito-v2-preview-deepseek-671B-MoE-GGUF/cogito-v2-preview-deepseek-671B-MoE-Q8_0.gguf

    #-mla 1 \
#numactl --interleave=all \
numactl -N 1 -m 1 \
./build/bin/llama-imatrix \
    -m "$model" \
    -fa \
    -mla 1 \
    -f ubergarm-imatrix-calibration-corpus-v02.txt \
    -o /mnt/raid/models/ubergarm/cogito-v2-preview-deepseek-671B-MoE-GGUF/imatrix-cogito-v2-preview-deepseek-671B-MoE-Q8_0.dat \
    --verbosity 2 \
    --ctx-size 512 \
    --layer-similarity \
    --numa numactl \
    -ub 4096 -b 4096 \
    --threads 128 \
    --threads-batch 192

collect_imatrix[1]:      blk.6.ffn_down_shexp.weight, MUL_MAT,  2048 x   512, 0
collect_imatrix[1]:       blk.7.attn_kv_a_mqa.weight, MUL_MAT,  7168 x   512, 0
collect_imatrix[1]:            blk.7.attn_q_a.weight, MUL_MAT,  7168 x   512, 0
collect_imatrix[1]:            blk.7.attn_q_b.weight, MUL_MAT,  1536 x   512, 0
collect_imatrix[1]: blk.7.attn_k_b.weight (reshaped), MUL_MAT,   128 x   512, 0
collect_imatrix[1]:         blk.7.attn_output.weight, MUL_MAT, 16384 x   512, 0
collect_imatrix[1]:        blk.7.ffn_gate_inp.weight, MUL_MAT,  7168 x   512, 0
collect_imatrix[1]:       blk.7.ffn_gate_exps.weight, MUL_MAT_ID,  7168 x   512, 0
collect_imatrix[1]:         blk.7.ffn_up_exps.weight, MUL_MAT_ID,  7168 x   512, 0
collect_imatrix[1]:       blk.7.ffn_down_exps.weight, MUL_MAT_ID,  2048 x   512, 0
collect_imatrix[1]:      blk.7.ffn_gate_shexp.weight, MUL_MAT,  7168 x   512, 0
collect_imatrix[1]:        blk.7.ffn_up_shexp.weight, MUL_MAT,  7168 x   512, 0
collect_imatrix[1]:      blk.7.ffn_down_shexp.weight, MUL_MAT,  2048 x   512, 0
collect_imatrix[1]:       blk.8.attn_kv_a_mqa.weight, MUL_MAT,  7168 x   512, 0
```

</details>


<details>

<summary>👈 no fa no mla</summary>

```bash
numactl -N 1 -m 1 \
./build/bin/llama-imatrix \
    -m "$model" \
    -f ubergarm-imatrix-calibration-corpus-v02.txt \
    -o /mnt/raid/models/ubergarm/cogito-v2-preview-deepseek-671B-MoE-GGUF/imatrix-cogito-v2-preview-deepseek-671B-MoE-Q8_0.dat \
    --verbosity 2 \
    --ctx-size 512 \
    --layer-similarity \
    --numa numactl \
    -ub 4096 -b 4096 \
    --threads 128 \
    --threads-batch 192

collect_imatrix[1]:     blk.10.ffn_down_shexp.weight, MUL_MAT,  2048 x   512, 0
collect_imatrix[1]:           blk.11.attn_q_a.weight, MUL_MAT,  7168 x   512, 0
collect_imatrix[1]:           blk.11.attn_q_b.weight, MUL_MAT,  1536 x   512, 0
collect_imatrix[1]:      blk.11.attn_kv_a_mqa.weight, MUL_MAT,  7168 x   512, 0
collect_imatrix[1]:          blk.11.attn_kv_b.weight, MUL_MAT,   512 x   512, 0
collect_imatrix[1]:        blk.11.attn_output.weight, MUL_MAT, 16384 x   512, 0
collect_imatrix[1]:       blk.11.ffn_gate_inp.weight, MUL_MAT,  7168 x   512, 0
collect_imatrix[1]:      blk.11.ffn_gate_exps.weight, MUL_MAT_ID,  7168 x   512, 0
collect_imatrix[1]:        blk.11.ffn_up_exps.weight, MUL_MAT_ID,  7168 x   512, 0
collect_imatrix[1]:      blk.11.ffn_down_exps.weight, MUL_MAT_ID,  2048 x   512, 0
collect_imatrix[1]:     blk.11.ffn_gate_shexp.weight, MUL_MAT,  7168 x   512, 0
collect_imatrix[1]:       blk.11.ffn_up_shexp.weight, MUL_MAT,  7168 x   512, 0
collect_imatrix[1]:     blk.11.ffn_down_shexp.weight, MUL_MAT,  2048 x   512, 0
collect_imatrix[1]:           blk.12.attn_q_a.weight, MUL_MAT,  7168 x   512, 0
```

</details>

<details>

<summary>👈 yes fa no mla</summary>

```bash
numactl -N 1 -m 1 \
./build/bin/llama-imatrix \
    -m "$model" \
    -fa \
    -f ubergarm-imatrix-calibration-corpus-v02.txt \
    -o /mnt/raid/models/ubergarm/cogito-v2-preview-deepseek-671B-MoE-GGUF/imatrix-cogito-v2-preview-deepseek-671B-MoE-Q8_0.dat \
    --verbosity 2 \
    --ctx-size 512 \
    --layer-similarity \
    --numa numactl \
    -ub 4096 -b 4096 \
    --threads 128 \
    --threads-batch 192

collect_imatrix[1]:     blk.10.ffn_down_shexp.weight, MUL_MAT,  2048 x   512, 0
collect_imatrix[1]:           blk.11.attn_q_a.weight, MUL_MAT,  7168 x   512, 0
collect_imatrix[1]:           blk.11.attn_q_b.weight, MUL_MAT,  1536 x   512, 0
collect_imatrix[1]:      blk.11.attn_kv_a_mqa.weight, MUL_MAT,  7168 x   512, 0
collect_imatrix[1]:          blk.11.attn_kv_b.weight, MUL_MAT,   512 x   512, 0
collect_imatrix[1]:        blk.11.attn_output.weight, MUL_MAT, 16384 x   512, 0
collect_imatrix[1]:       blk.11.ffn_gate_inp.weight, MUL_MAT,  7168 x   512, 0
collect_imatrix[1]:      blk.11.ffn_gate_exps.weight, MUL_MAT_ID,  7168 x   512, 0
collect_imatrix[1]:        blk.11.ffn_up_exps.weight, MUL_MAT_ID,  7168 x   512, 0
collect_imatrix[1]:      blk.11.ffn_down_exps.weight, MUL_MAT_ID,  2048 x   512, 0
collect_imatrix[1]:     blk.11.ffn_gate_shexp.weight, MUL_MAT,  7168 x   512, 0
collect_imatrix[1]:       blk.11.ffn_up_shexp.weight, MUL_MAT,  7168 x   512, 0
collect_imatrix[1]:     blk.11.ffn_down_shexp.weight, MUL_MAT,  2048 x   512, 0
collect_imatrix[1]:           blk.12.attn_q_a.weight, MUL_MAT,  7168 x   512, 0
collect_imatrix[1]:           blk.12.attn_q_b.weight, MUL_MAT,  1536 x   512, 0
collect_imatrix[1]:      blk.12.attn_kv_a_mqa.weight, MUL_MAT,  7168 x   512, 0
```

</details>

Hrmm, when i went back to `--verbosity 1` and just trying to run it for any imatrix data now seeing an assert:
```bash
collect_imatrix[1]:       blk.60.ffn_gate_inp.weight, MUL_MAT,  7168 x   512, 0
collect_imatrix[1]:      blk.60.ffn_gate_exps.weight, MUL_MAT_ID,  7168 x   512, 0
collect_imatrix[1]:        blk.60.ffn_up_exps.weight, MUL_MAT_ID,  7168 x   512, 0
collect_imatrix[1]:      blk.60.ffn_down_exps.weight, MUL_MAT_ID,  2048 x   512, 0
collect_imatrix[1]:     blk.60.ffn_gate_shexp.weight, MUL_MAT,  7168 x   512, 0
collect_imatrix[1]:       blk.60.ffn_up_shexp.weight, MUL_MAT,  7168 x   512, 0
collect_imatrix[1]:     blk.60.ffn_down_shexp.weight, MUL_MAT,  2048 x   512, 0
/home/w/projects/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:505: /home/w/projects/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:505: GGML_ASSERT(Nx%num_row
s == 0) failedGGML_ASSERT(Nx%num_rows == 0) failed
/home/w/projects/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:505: GGML_ASSERT(Nx%num_rows == 0) failed
```

The model seems to be inferencing okay but did require explicitly setting `--chat-template deepseek3`. I'll keep fussing with it.

just throwing some random notes here in different in my gguf and the published unsloth one:

```bash
# ubegarm
llama_model_loader: - kv  25:             deepseek2.attention.key_length u32              = 192
llama_model_loader: - kv  26:           deepseek2.attention.value_length u32              = 128
# unsloth
deepseek2.attention.key_length     576
deepseek2.attention.value_length     512
deepseek2.attention.key_length_mla     192
deepseek2.attention.value_length_mla     128
```

Amusingly it quantizes using the imatrix from my old DeepSeek-V3-0324 release as apparently this model is fine-tuned from that... *probably* not a great idea ... lol

---

👤 **ubergarm** commented on **2025-08-22** at **02:57:57**

Still no dice tonight after making imatrix for [ubergarm/DeepSeek-V3.1-GGUF](https://huggingface.co/ubergarm/DeepSeek-V3.1-GGUF/blob/main/imatrix-DeepSeek-V3.1-Q8_0.dat). I'll have to make `attn_(v|k)_b = q8_0` as no imatrix data seems to come through for those two.

I'm not sure if I'm doing something wrong after converting the bf16 safetensors to bf16 GGUF. I used the convert_hf_to_gguf.py script from this ik_llama.cpp repo. I then copy over all the tokenizer.json and .py files from the deepseek-ai original safetensors model being careful not to overwrite the new `model.safetensors.index.json` file.

Here is the imatrix first run with `--verbosity 2` showing it with that `attn_k_b.weight (reshaped)` business. Where does that `(reshaped)` name come from, maybe I need to upgrade or downgrade pytorch or transformers? hah...

<details>

<summary>👈 llama-imatrix and partial logs</summary>

```bash
#!/usr/bin/env bash

model=/mnt/raid/models/ubergarm/DeepSeek-V3.1-GGUF/DeepSeek-V3.1-Q8_0.gguf

numactl -N 1 -m 1 \
./build/bin/llama-imatrix \
    -m "$model" \
    -fa -mla 1 \
    -f ubergarm-imatrix-calibration-corpus-v02.txt \
    -o /mnt/raid/models/ubergarm/DeepSeek-V3.1-GGUF/imatrix-DeepSeek-V3.1-Q8_0.dat \
    --verbosity 2 \
    --layer-similarity \
    --ctx-size 512 \
    -ub 4096 -b 4096 \
    --numa numactl \
    --threads 128 \
    --threads-batch 192 \
    --no-mmap

llama_model_loader: loaded meta data with 45 key-value pairs and 1147 tensors from /mnt/raid/models/ubergarm/DeepSeek-V3.1-GGUF/DeepSeek-V3.1-Q8_0.gguf (version GGUF V3 (latest))
llama_model_loader: Dumping metadata keys/values. Note: KV overrides do not apply in this output.
llama_model_loader: - kv   0:                       general.architecture str              = deepseek2
llama_model_loader: - kv   1:                               general.type str              = model
llama_model_loader: - kv   2:                               general.name str              = DeepSeek V3.1 Bf16 Safetensors
llama_model_loader: - kv   3:                           general.finetune str              = safetensors
llama_model_loader: - kv   4:                           general.basename str              = DeepSeek-V3.1
llama_model_loader: - kv   5:                         general.size_label str              = 256x21B
llama_model_loader: - kv   6:                      deepseek2.block_count u32              = 61
llama_model_loader: - kv   7:                   deepseek2.context_length u32              = 163840
llama_model_loader: - kv   8:                 deepseek2.embedding_length u32              = 7168
llama_model_loader: - kv   9:              deepseek2.feed_forward_length u32              = 18432
llama_model_loader: - kv  10:             deepseek2.attention.head_count u32              = 128
llama_model_loader: - kv  11:          deepseek2.attention.head_count_kv u32              = 128
llama_model_loader: - kv  12:                   deepseek2.rope.freq_base f32              = 10000.000000
llama_model_loader: - kv  13: deepseek2.attention.layer_norm_rms_epsilon f32              = 0.000001
llama_model_loader: - kv  14:                deepseek2.expert_used_count u32              = 8
llama_model_loader: - kv  15:                          general.file_type u32              = 7

llama_model_loader: - kv  42:               tokenizer.ggml.add_eos_token bool             = false
llama_model_loader: - kv  43:                    tokenizer.chat_template str              = {% if not add_generation_prompt is de...
llama_model_loader: - kv  44:               general.quantization_version u32              = 2
llama_model_loader: - type  f32:  361 tensors
llama_model_loader: - type q8_0:  786 tensors
init_tokenizer: initializing tokenizer for type 2
load: special_eos_id is not in special_eog_ids - the tokenizer config may be incorrect
load: printing all EOG tokens:

load: special tokens cache size = 818
load: token to piece cache size = 0.8223 MB
llm_load_print_meta: format           = GGUF V3 (latest)
llm_load_print_meta: arch             = deepseek2
llm_load_print_meta: n_ctx_train      = 163840
llm_load_print_meta: n_embd           = 7168
llm_load_print_meta: n_layer          = 61
llm_load_print_meta: n_head           = 128
llm_load_print_meta: n_head_kv        = 128
llm_load_print_meta: n_rot            = 64
llm_load_print_meta: n_swa            = 0
llm_load_print_meta: n_swa_pattern    = 1
llm_load_print_meta: n_embd_head_k    = 192
llm_load_print_meta: n_embd_head_v    = 128
llm_load_print_meta: n_gqa            = 1
llm_load_print_meta: n_embd_k_gqa     = 24576
llm_load_print_meta: n_embd_v_gqa     = 16384
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
llm_load_print_meta: rope type        = 0
llm_load_print_meta: rope scaling     = yarn
llm_load_print_meta: freq_base_train  = 10000.0
llm_load_print_meta: freq_scale_train = 0.025
llm_load_print_meta: n_ctx_orig_yarn  = 4096
llm_load_print_meta: rope_finetuned   = unknown
llm_load_print_meta: ssm_d_conv       = 0
llm_load_print_meta: ssm_d_inner      = 0
llm_load_print_meta: ssm_d_state      = 0
llm_load_print_meta: ssm_dt_rank      = 0
llm_load_print_meta: model type       = 671B
llm_load_print_meta: model ftype      = Q8_0
llm_load_print_meta: model params     = 672.050 B
llm_load_print_meta: model size       = 665.308 GiB (8.504 BPW) 
llm_load_print_meta: repeating layers = 663.474 GiB (8.504 BPW, 670.196 B parameters)
llm_load_print_meta: general.name     = DeepSeek V3.1 Bf16 Safetensors
llm_load_print_meta: n_layer_dense_lead   = 3
llm_load_print_meta: n_lora_q             = 1536
llm_load_print_meta: n_lora_kv            = 512
llm_load_print_meta: n_ff_exp             = 2048
llm_load_print_meta: n_expert_shared      = 1
llm_load_print_meta: expert_weights_scale = 2.5
llm_load_print_meta: expert_weights_norm  = 1
llm_load_print_meta: expert_gating_func   = sigmoid
llm_load_print_meta: rope_yarn_log_mul    = 0.1000
print_info: vocab type       = BPE
print_info: n_vocab          = 129280
print_info: n_merges         = 127741

print_info: max token length = 256
llm_load_tensors: ggml ctx size =    0.47 MiB
llm_load_tensors:        CPU buffer size = 681274.97 MiB
....................................................................................................
llama_new_context_with_model: n_ctx      = 512
llama_new_context_with_model: n_batch    = 512
llama_new_context_with_model: n_ubatch   = 512
llama_new_context_with_model: flash_attn = 1
llama_new_context_with_model: mla_attn   = 1
llama_new_context_with_model: attn_max_b = 0
llama_new_context_with_model: fused_moe  = 0
llama_new_context_with_model: ser        = -1, 0
llama_new_context_with_model: freq_base  = 10000.0
llama_new_context_with_model: freq_scale = 0.025
llama_kv_cache_init:        CPU KV buffer size =    34.31 MiB
llama_new_context_with_model: KV self size  =   34.31 MiB, c^KV (f16):   34.31 MiB, kv^T: not used
llama_new_context_with_model:        CPU  output buffer size =     0.49 MiB
llama_new_context_with_model:        CPU compute buffer size =   379.51 MiB
llama_new_context_with_model: graph nodes  = 3542
llama_new_context_with_model: graph splits = 1

system_info: n_threads = 128 (n_threads_batch = 192) / 768 | AVX = 1 | AVX_VNNI = 1 | AVX2 = 1 | AVX512 = 1 | AVX512_VBMI = 1 | AVX512_VNNI = 1 | AVX512_BF16 = 1 | FMA = 1 | NEON = 0 | SVE = 0 | ARM_FMA = 0 | F16C = 1 | FP16_VA = 0 | WASM_SIMD = 0 | BLAS = 0 | SSE3 = 1 | SSSE3 = 1 | VSX = 0 | MATMUL_INT8 = 0 | LLAMAFILE = 1 | 
compute_imatrix: tokenizing the input ..
compute_imatrix: tokenization took 657.005 ms
compute_imatrix: computing over 812 chunks with batch_size 512


collect_imatrix[0]:       blk.0.attn_kv_a_mqa.weight, MUL_MAT,  7168 x   512, 0
collect_imatrix[1]:            blk.0.attn_q_a.weight, MUL_MAT,  7168 x   512, 0
collect_imatrix[1]:            blk.0.attn_q_b.weight, MUL_MAT,  1536 x   512, 0
collect_imatrix[1]: blk.0.attn_k_b.weight (reshaped), MUL_MAT,   128 x   512, 0
collect_imatrix[1]:         blk.0.attn_output.weight, MUL_MAT, 16384 x   512, 0
collect_imatrix[1]:            blk.0.ffn_gate.weight, MUL_MAT,  7168 x   512, 0
collect_imatrix[1]:              blk.0.ffn_up.weight, MUL_MAT,  7168 x   512, 0
collect_imatrix[1]:            blk.0.ffn_down.weight, MUL_MAT, 18432 x   512, 0
collect_imatrix[1]:       blk.1.attn_kv_a_mqa.weight, MUL_MAT,  7168 x   512, 0
collect_imatrix[1]:            blk.1.attn_q_a.weight, MUL_MAT,  7168 x   512, 0
collect_imatrix[1]:            blk.1.attn_q_b.weight, MUL_MAT,  1536 x   512, 0
collect_imatrix[1]: blk.1.attn_k_b.weight (reshaped), MUL_MAT,   128 x   512, 0
collect_imatrix[1]:         blk.1.attn_output.weight, MUL_MAT, 16384 x   512, 0
collect_imatrix[1]:            blk.1.ffn_gate.weight, MUL_MAT,  7168 x   512, 0
collect_imatrix[1]:              blk.1.ffn_up.weight, MUL_MAT,  7168 x   512, 0
collect_imatrix[1]:            blk.1.ffn_down.weight, MUL_MAT, 18432 x   512, 0
collect_imatrix[1]:       blk.2.attn_kv_a_mqa.weight, MUL_MAT,  7168 x   512, 0
collect_imatrix[1]:            blk.2.attn_q_a.weight, MUL_MAT,  7168 x   512, 0
collect_imatrix[1]:            blk.2.attn_q_b.weight, MUL_MAT,  1536 x   512, 0
collect_imatrix[1]: blk.2.attn_k_b.weight (reshaped), MUL_MAT,   128 x   512, 0
collect_imatrix[1]:         blk.2.attn_output.weight, MUL_MAT, 16384 x   512, 0
collect_imatrix[1]:            blk.2.ffn_gate.weight, MUL_MAT,  7168 x   512, 0
collect_imatrix[1]:              blk.2.ffn_up.weight, MUL_MAT,  7168 x   512, 0
collect_imatrix[1]:            blk.2.ffn_down.weight, MUL_MAT, 18432 x   512, 0
collect_imatrix[1]:       blk.3.attn_kv_a_mqa.weight, MUL_MAT,  7168 x   512, 0
collect_imatrix[1]:            blk.3.attn_q_a.weight, MUL_MAT,  7168 x   512, 0
collect_imatrix[1]:            blk.3.attn_q_b.weight, MUL_MAT,  1536 x   512, 0
collect_imatrix[1]: blk.3.attn_k_b.weight (reshaped), MUL_MAT,   128 x   512, 0
collect_imatrix[1]:         blk.3.attn_output.weight, MUL_MAT, 16384 x   512, 0
collect_imatrix[1]:        blk.3.ffn_gate_inp.weight, MUL_MAT,  7168 x   512, 0
collect_imatrix[1]:       blk.3.ffn_gate_exps.weight, MUL_MAT_ID,  7168 x   512, 0
collect_imatrix[1]:         blk.3.ffn_up_exps.weight, MUL_MAT_ID,  7168 x   512, 0
collect_imatrix[1]:       blk.3.ffn_down_exps.weight, MUL_MAT_ID,  2048 x   512, 0
collect_imatrix[1]:      blk.3.ffn_gate_shexp.weight, MUL_MAT,  7168 x   512, 0
collect_imatrix[1]:        blk.3.ffn_up_shexp.weight, MUL_MAT,  7168 x   512, 0
collect_imatrix[1]:      blk.3.ffn_down_shexp.weight, MUL_MAT,  2048 x   512, 0
collect_imatrix[1]:       blk.4.attn_kv_a_mqa.weight, MUL_MAT,  7168 x   512, 0
collect_imatrix[1]:            blk.4.attn_q_a.weight, MUL_MAT,  7168 x   512, 0
collect_imatrix[1]:            blk.4.attn_q_b.weight, MUL_MAT,  1536 x   512, 0
collect_imatrix[1]: blk.4.attn_k_b.weight (reshaped), MUL_MAT,   128 x   512, 0
collect_imatrix[1]:         blk.4.attn_output.weight, MUL_MAT, 16384 x   512, 0
collect_imatrix[1]:        blk.4.ffn_gate_inp.weight, MUL_MAT,  7168 x   512, 0
collect_imatrix[1]:       blk.4.ffn_gate_exps.weight, MUL_MAT_ID,  7168 x   512, 0
collect_imatrix[1]:         blk.4.ffn_up_exps.weight, MUL_MAT_ID,  7168 x   512, 0
collect_imatrix[1]:       blk.4.ffn_down_exps.weight, MUL_MAT_ID,  2048 x   512, 0
collect_imatrix[1]:      blk.4.ffn_gate_shexp.weight, MUL_MAT,  7168 x   512, 0
collect_imatrix[1]:        blk.4.ffn_up_shexp.weight, MUL_MAT,  7168 x   512, 0
collect_imatrix[1]:      blk.4.ffn_down_shexp.weight, MUL_MAT,  2048 x   512, 0
collect_imatrix[1]:       blk.5.attn_kv_a_mqa.weight, MUL_MAT,  7168 x   512, 0
```

</details>

Likewise here is the quantize command using the above imatrix showing it missing imatrix data for those tensors unfortunately.

<details>

<summary>👈 llama-quantize and partial logs</summary>

```bash
#!/usr/bin/env bash

#          token_embd.weight - [ 7168, 129280,     1,     1], type =   bf16, converting to q8_0 .. size =  1767.50 MiB ->   938.98
#
#blk.3.ffn_down_shexp.weight - [ 2048,  7168,     1,     1], type =   bf16, converting to q8_0 .. size =    28.00 MiB ->    14.88 MiB
#blk.3.ffn_gate_shexp.weight - [ 7168,  2048,     1,     1], type =   bf16, converting to q8_0 .. size =    28.00 MiB ->    14.88
#  blk.3.ffn_up_shexp.weight - [ 7168,  2048,     1,     1], type =   bf16, converting to q8_0 .. size =    28.00 MiB ->    14.88
# blk.3.ffn_down_exps.weight - [ 2048,  7168,   256,     1], type =   bf16, converting to q8_0 .. size =  7168.00 MiB ->  3808.00
# blk.3.ffn_gate_exps.weight - [ 7168,  2048,   256,     1], type =   bf16, converting to q8_0 .. size =  7168.00 MiB ->  3808.00
#   blk.3.ffn_up_exps.weight - [ 7168,  2048,   256,     1], type =   bf16, converting to q8_0 .. size =  7168.00 MiB ->  3808.00 MiB
#
# blk.3.attn_kv_a_mqa.weight - [ 7168,   576,     1,     1], type =   bf16, converting to q8_0 .. size =     7.88 MiB ->     4.18
#     blk.3.attn_kv_b.weight - [  512, 32768,     1,     1], type =   bf16, converting to q8_0 .. size =    32.00 MiB ->    17.00
#      blk.3.attn_k_b.weight - [  128, 65536,     1,     1], type =   bf16, converting to q8_0 .. size =    16.00 MiB ->     8.50
#      blk.3.attn_v_b.weight - [  512, 16384,     1,     1], type =   bf16, converting to q8_0 .. size =    16.00 MiB ->     8.50 MiB
#   blk.3.attn_output.weight - [16384,  7168,     1,     1], type =   bf16, converting to q8_0 .. size =   224.00 MiB ->   119.00
#      blk.3.attn_q_a.weight - [ 7168,  1536,     1,     1], type =   bf16, converting to q8_0 .. size =    21.00 MiB ->    11.16
#      blk.3.attn_q_b.weight - [ 1536, 24576,     1,     1], type =   bf16, converting to q8_0 .. size =    72.00 MiB ->    38.25 MiB
#
#              output.weight - [ 7168, 129280,     1,     1], type =   bf16, converting to q8_0 .. size =  1767.50 MiB ->   938.98 MiB

custom="
## Attention [0-60] (GPU)
# attn_kv_b is only used for PP so keep it q8_0 for best speed and accuracy
blk\..*\.attn_kv_b\.weight=q8_0

# ideally k_b and v_b are smaller than q8_0 as they are is used for TG with -mla 3
# https://github.com/ikawrakow/ik_llama.cpp/issues/651
# blk.*.attn_k_b.weight is not divisible by 256 so only supports iq4_nl or legacy qN_0
blk\..*\.attn_k_b\.weight=q8_0
blk\..*\.attn_v_b\.weight=q8_0

# Balance of attn tensors
blk\..*\.attn_kv_a_mqa\.weight=q8_0
blk\..*\.attn_q_a\.weight=q8_0
blk\..*\.attn_q_b\.weight=q8_0
blk\..*\.attn_output\.weight=q8_0

## First Three Dense Layers [0-2] (GPU)
blk\..*\.ffn_down\.weight=q8_0
blk\..*\.ffn_(gate|up)\.weight=q8_0

## Shared Expert (1-60) (GPU)
blk\..*\.ffn_down_shexp\.weight=q8_0
blk\..*\.ffn_(gate|up)_shexp\.weight=q8_0

## Routed Experts (1-60) (CPU)
blk\..*\.ffn_down_exps\.weight=iq6_k
blk\..*\.ffn_(gate|up)_exps\.weight=iq5_k

## Token embedding and output tensors (GPU)
token_embd\.weight=iq6_k
output\.weight=iq6_k
"

custom=$(
  echo "$custom" | grep -v '^#' | \
  sed -Ez 's:\n+:,:g;s:,$::;s:^,::'
)

numactl -N 1 -m 1 \
./build/bin/llama-quantize \
    --custom-q "$custom" \
    --imatrix /mnt/raid/models/ubergarm/DeepSeek-V3.1-GGUF/imatrix-DeepSeek-V3.1-Q8_0.dat \
    /mnt/raid/models/ubergarm/DeepSeek-V3.1-GGUF/DeepSeek-V3.1-256x21B-safetensors-BF16-00001-of-00030.gguf \
    /mnt/raid/models/ubergarm/DeepSeek-V3.1-GGUF/DeepSeek-V3.1-IQ5_K.gguf \
    IQ5_K \
    192

llama_model_loader: - kv  44:               general.quantization_version u32              = 2
llama_model_loader: - kv  45:                                   split.no u16              = 0
llama_model_loader: - kv  46:                                split.count u16              = 30
llama_model_loader: - kv  47:                        split.tensors.count i32              = 1147
llama_model_loader: - type  f32:  361 tensors
llama_model_loader: - type bf16:  786 tensors
================================ Have weights data with 721 entries
[   1/1147]                    token_embd.weight - [ 7168, 129280,     1,     1], type =   bf16, Using custom type iq6_k for tensor token_embd.weight

====== llama_model_quantize_internal: did not find weights for token_embd.weight
converting to iq6_k .. Adding custom rule blk\..*\.attn_kv_b\.weight -> q8_0
Adding custom rule blk\..*\.attn_k_b\.weight -> q8_0
Adding custom rule blk\..*\.attn_v_b\.weight -> q8_0
Adding custom rule blk\..*\.attn_kv_a_mqa\.weight -> q8_0
Adding custom rule blk\..*\.attn_q_a\.weight -> q8_0
Adding custom rule blk\..*\.attn_q_b\.weight -> q8_0
Adding custom rule blk\..*\.attn_output\.weight -> q8_0
Adding custom rule blk\..*\.ffn_down\.weight -> q8_0
Adding custom rule blk\..*\.ffn_(gate|up)\.weight -> q8_0
Adding custom rule blk\..*\.ffn_down_shexp\.weight -> q8_0
Adding custom rule blk\..*\.ffn_(gate|up)_shexp\.weight -> q8_0
Adding custom rule blk\..*\.ffn_down_exps\.weight -> iq6_k
Adding custom rule blk\..*\.ffn_(gate|up)_exps\.weight -> iq5_k
Adding custom rule token_embd\.weight -> iq6_k
Adding custom rule output\.weight -> iq6_k
load_imatrix: imatrix dataset='ubergarm-imatrix-calibration-corpus-v02.txt'
load_imatrix: loaded 721 importance matrix entries from /mnt/raid/models/ubergarm/DeepSeek-V3.1-GGUF/imatrix-DeepSeek-V3.1-Q8_0.dat computed on 812 chunks
prepare_imatrix: have 721 importance matrix entries
size =  1767.50 MiB ->   731.86 MiB
[   2/1147]               blk.0.attn_norm.weight - [ 7168,     1,     1,     1], type =    f32, size =    0.027 MB
[   3/1147]                blk.0.ffn_down.weight - [18432,  7168,     1,     1], type =   bf16, Using custom type q8_0 for tensor blk.0.ffn_down.weight
converting to q8_0 .. size =   252.00 MiB ->   133.88 MiB
[   4/1147]                blk.0.ffn_gate.weight - [ 7168, 18432,     1,     1], type =   bf16, Using custom type q8_0 for tensor blk.0.ffn_gate.weight
converting to q8_0 .. size =   252.00 MiB ->   133.88 MiB
[   5/1147]                  blk.0.ffn_up.weight - [ 7168, 18432,     1,     1], type =   bf16, Using custom type q8_0 for tensor blk.0.ffn_up.weight
converting to q8_0 .. size =   252.00 MiB ->   133.88 MiB
[   6/1147]                blk.0.ffn_norm.weight - [ 7168,     1,     1,     1], type =    f32, size =    0.027 MB
[   7/1147]          blk.0.attn_kv_a_norm.weight - [  512,     1,     1,     1], type =    f32, size =    0.002 MB
[   8/1147]           blk.0.attn_kv_a_mqa.weight - [ 7168,   576,     1,     1], type =   bf16, Using custom type q8_0 for tensor blk.0.attn_kv_a_mqa.weight
converting to q8_0 .. size =     7.88 MiB ->     4.18 MiB
[   9/1147]               blk.0.attn_kv_b.weight - [  512, 32768,     1,     1], type =   bf16, Using custom type q8_0 for tensor blk.0.attn_kv_b.weight

====== llama_model_quantize_internal: did not find weights for blk.0.attn_kv_b.weight
converting to q8_0 .. size =    32.00 MiB ->    17.00 MiB
[  10/1147]                blk.0.attn_k_b.weight - [  128, 65536,     1,     1], type =   bf16, Using custom type q8_0 for tensor blk.0.attn_k_b.weight

====== llama_model_quantize_internal: did not find weights for blk.0.attn_k_b.weight
converting to q8_0 .. size =    16.00 MiB ->     8.50 MiB
[  11/1147]                blk.0.attn_v_b.weight - [  512, 16384,     1,     1], type =   bf16, Using custom type q8_0 for tensor blk.0.attn_v_b.weight

====== llama_model_quantize_internal: did not find weights for blk.0.attn_v_b.weight
converting to q8_0 .. size =    16.00 MiB ->     8.50 MiB
[  12/1147]             blk.0.attn_output.weight - [16384,  7168,     1,     1], type =   bf16, Using custom type q8_0 for tensor blk.0.attn_output.weight
converting to q8_0 .. size =   224.00 MiB ->   119.00 MiB
[  13/1147]           blk.0.attn_q_a_norm.weight - [ 1536,     1,     1,     1], type =    f32, size =    0.006 MB
[  14/1147]                blk.0.attn_q_a.weight - [ 7168,  1536,     1,     1], type =   bf16, Using custom type q8_0 for tensor blk.0.attn_q_a.weight
converting to q8_0 .. size =    21.00 MiB ->    11.16 MiB
[  15/1147]                blk.0.attn_q_b.weight - [ 1536, 24576,     1,     1], type =   bf16, Using custom type q8_0 for tensor blk.0.attn_q_b.weight
converting to q8_0 .. size =    72.00 MiB ->    38.25 MiB
[  16/1147]               blk.1.attn_norm.weight - [ 7168,     1,     1,     1], type =    f32, size =    0.027 MB
[  17/1147]                blk.1.ffn_down.weight - [18432,  7168,     1,     1], type =   bf16, Using custom type q8_0 for tensor blk.1.ffn_down.weight
```

</details>

---

👤 **ikawrakow** commented on **2025-08-22** at **03:11:17**

@ubergarm Try using the `convert_hf_to_gguf.py` script from mainline. The `attn_k_b` tensor does not have the right shape, and `attn_v_b` is missing.

---

👤 **ubergarm** commented on **2025-08-22** at **03:15:09**

> Try using the convert_hf_to_gguf.py script from mainline.

I'll give that a try soon, but if I recall, it doesn't have the `attn_kv_b` tensor and will always print that warning `missing wkv_b tensor(s) changing MLA from %d to 1\n Prompt processing performance will be crippled *` ? 

Thanks, will report back what I find!

---

👤 **ikawrakow** commented on **2025-08-22** at **03:16:56**

I guess, I need to remove this warning as the performance issues have been sorted out.

---

👤 **ubergarm** commented on **2025-08-22** at **14:39:34**

Okay, so converting hf bf16 safetensors to bf16 gguf using mainline lcpp then using imatrix here on ik seems to be working now and no longer printing out missing imatrix data warnings etc.

Some observations:

1. The imatrix has data for both `attn_(k|v)_b` tensors so those can be quantized now more safely instead of leaving q8_0.
2. The dimensions of `attn_(k|v)_b` are different now, and "3 dimensional" it seems.
3. The `attn_kv_b` tensor is gone, and this fork will reconstruct it in a q8_0 container for PP for some MLA paths (-mla 2 -mla 3 i think?) at runtime from the quantized `attn_(k|v)_b` tensors.
4. There are still warnings in both llama-quantize and llama-server when no `attn_kv_b` tensor is present e.g.

```
=============================
Detected incompatible DeepSeek model.
Will try to fix, but there are no guarantees

*** Your prompt processing speed will be crippled ***

Consider making your own ik_llama.cpp compatible model or
ask the model provider to make one for you,
==========================================================================
```

<details>

<summary>👈 Tensor Differences</summary>

```bash
          token_embd.weight - [ 7168, 129280,     1,     1], type =   bf16, converting to q8_0 .. size =  1767.50 MiB ->   938.98

# This section is if converted with ik
blk.3.ffn_down_shexp.weight - [ 2048,  7168,     1,     1], type =   bf16, converting to q8_0 .. size =    28.00 MiB ->    14.88 MiB
blk.3.ffn_gate_shexp.weight - [ 7168,  2048,     1,     1], type =   bf16, converting to q8_0 .. size =    28.00 MiB ->    14.88
  blk.3.ffn_up_shexp.weight - [ 7168,  2048,     1,     1], type =   bf16, converting to q8_0 .. size =    28.00 MiB ->    14.88
 blk.3.ffn_down_exps.weight - [ 2048,  7168,   256,     1], type =   bf16, converting to q8_0 .. size =  7168.00 MiB ->  3808.00
 blk.3.ffn_gate_exps.weight - [ 7168,  2048,   256,     1], type =   bf16, converting to q8_0 .. size =  7168.00 MiB ->  3808.00
   blk.3.ffn_up_exps.weight - [ 7168,  2048,   256,     1], type =   bf16, converting to q8_0 .. size =  7168.00 MiB ->  3808.00 MiB

 blk.3.attn_kv_a_mqa.weight - [ 7168,   576,     1,     1], type =   bf16, converting to q8_0 .. size =     7.88 MiB ->     4.18
     blk.3.attn_kv_b.weight - [  512, 32768,     1,     1], type =   bf16, converting to q8_0 .. size =    32.00 MiB ->    17.00
      blk.3.attn_k_b.weight - [  128, 65536,     1,     1], type =   bf16, converting to q8_0 .. size =    16.00 MiB ->     8.50
      blk.3.attn_v_b.weight - [  512, 16384,     1,     1], type =   bf16, converting to q8_0 .. size =    16.00 MiB ->     8.50 MiB
   blk.3.attn_output.weight - [16384,  7168,     1,     1], type =   bf16, converting to q8_0 .. size =   224.00 MiB ->   119.00
      blk.3.attn_q_a.weight - [ 7168,  1536,     1,     1], type =   bf16, converting to q8_0 .. size =    21.00 MiB ->    11.16
      blk.3.attn_q_b.weight - [ 1536, 24576,     1,     1], type =   bf16, converting to q8_0 .. size =    72.00 MiB ->    38.25 MiB

# This section is if converted with mainline
blk.3.ffn_down_shexp.weight - [ 2048,  7168,     1,     1], type =   bf16, converting to q8_0 .. size =    28.00 MiB ->    14.88 MiB
blk.3.ffn_gate_shexp.weight - [ 7168,  2048,     1,     1], type =   bf16, converting to q8_0 .. size =    28.00 MiB ->    14.88 MiB
  blk.3.ffn_up_shexp.weight - [ 7168,  2048,     1,     1], type =   bf16, converting to q8_0 .. size =    28.00 MiB ->    14.88 MiB
 blk.3.ffn_down_exps.weight - [ 2048,  7168,   256,     1], type =   bf16, converting to q8_0 .. size =  7168.00 MiB ->  3808.00 MiB
 blk.3.ffn_gate_exps.weight - [ 7168,  2048,   256,     1], type =   bf16, converting to q8_0 .. size =  7168.00 MiB ->  3808.00 MiB
   blk.3.ffn_up_exps.weight - [ 7168,  2048,   256,     1], type =   bf16, converting to q8_0 .. size =  7168.00 MiB ->  3808.00 MiB

 blk.3.attn_kv_a_mqa.weight - [ 7168,   576,     1,     1], type =   bf16, converting to q8_0 .. size =     7.88 MiB ->     4.18 MiB
      blk.3.attn_k_b.weight - [  128,   512,   128,     1], type =   bf16, converting to q8_0 .. size =    16.00 MiB ->     8.50 MiB
      blk.3.attn_v_b.weight - [  512,   128,   128,     1], type =   bf16, converting to q8_0 .. size =    16.00 MiB ->     8.50 MiB
   blk.3.attn_output.weight - [16384,  7168,     1,     1], type =   bf16, converting to q8_0 .. size =   224.00 MiB ->   119.00 MiB
      blk.3.attn_q_a.weight - [ 7168,  1536,     1,     1], type =   bf16, converting to q8_0 .. size =    21.00 MiB ->    11.16 MiB
      blk.3.attn_q_b.weight - [ 1536, 24576,     1,     1], type =   bf16, converting to q8_0 .. size =    72.00 MiB ->    38.25 MiB

              output.weight - [ 7168, 129280,     1,     1], type =   bf16, converting to q8_0 .. size =  1767.50 MiB ->   938.98 MiB
```

</details>

Given this seems to be the new way to go about it, I've [released the new imatrix here](https://huggingface.co/ubergarm/DeepSeek-V3.1-GGUF/blob/main/imatrix-DeepSeek-V3.1-Q8_0.dat) and will continue quantizing. Thanks!

---

👤 **ikawrakow** commented on **2025-08-22** at **15:41:32**

I have removed the scary warning in [#717](https://github.com/ikawrakow/ik_llama.cpp/issues/717) 

The mainline tensor dimensions are exactly as they should be.

The `ik_llama.cpp` tensor dimensions are also good. The `128 x 65536` `attn_k_b` tensor is viewed as a 3D tensor of `128 x 512 x 128` when used at runtime, so exactly as the mainline `attn_k_b` tensor. The `512 x 16384` `attn_v_b` tensor is also viewed as a 3D tensor of `512 x 128 x 128` at runtime, so exactly like the mainline `attn_v_b`. Really not sure why the `imatrix` stuff did not work.

> The attn_kv_b tensor is gone, and this fork will reconstruct it in a q8_0 container for PP for some MLA paths (-mla 2 -mla 3 i think?) at runtime from the quantized attn_(k|v)_b tensors.

Yes, `attn_kv_b` gets constructed when loading the model as `Q8_0`. This is OK because it only gets used for prompt processing (with batch size > 128), where the calculation is not memory bound. `attn_k_b` and `attn_v_b` get used for TG, so using fewer than 8 bits for those should be beneficial for TG performance.

---

👤 **ikawrakow** commented on **2025-08-22** at **16:00:14**

I had forgotten the DeepSeek tensor dimensions. Here is a list of the non-MoE tensors per repeating layer with number of parameters:

| tensor name | parameters |
| ---: | ---: |
| attn_kv_a_mqa |     4,128,768 |
| attn_k_b     |     8,388,608 |
| attn_v_b     |    8,388,608 |
| attn_output  |   117,440,512 |
| attn_q_a     |    11,010,048 |
| attn_q_b     |    37,748,736 |
| ffn_down_shexp |   14,680,064 |
| ffn_gate_shexp |   14,680,064 |
| ffn_up_shexp  |   14,680,064 |

So, `attn_output` is by far the most important target when looking for reducing the number of bits spent on non-MoE tensors. 6.7 billion out of the 37 billion active parameters are in that one tensor type! I guess, bpw reduction for `attn_k_b` and `attn_v_b` will have negligible impact on TG performance.

---

👤 **ubergarm** commented on **2025-08-22** at **16:07:45**

> So, attn_output is by far the most important target when looking for reducing the number of bits spent on non-MoE tensors.

Great! that was also my observation and have adjusted my latest recipes to leave some of the smaller ones at larger sizes then quantize that `attn_output` a bit to improve TG a bit.

Looking at attn tensors for a single layer:

| tensor | one layer size MiB @ Q8_0| Percent of attn tensors |
| --- | --- | --- |
| attn_kv_a_mqa | 4.18 MiB | 2.2% |
| attn_k_b | 8.50 MiB | 4.5% |
| attn_v_b | 8.50 MiB | 4.5% |
| attn_output | 119.00 MiB | 62.8% |
| attn_q_a | 11.16 MiB |  5.9% |
| attn_q_b | 38.25 MiB | 20.2% |

For the smallest quant, I went with the non `_r4` IQ1_S 133.610 GiB (1.710 BPW) for now too. Hopefully will have more perplexity data and graph coming in the following couple days! Thanks for all the recent updates!

---

👤 **ubergarm** commented on **2025-08-22** at **16:17:58**

I'll close this now that https://github.com/ikawrakow/ik_llama.cpp/pull/717 is merged. Cheers!

---

👤 **ikawrakow** commented on **2025-08-22** at **16:31:29**

For regular attention, the ordered list of tensor importance from highest to lowest is `attn_v, attn_output, attn_k, attn_q`.  If this is similar for MLA, the `attn_q_a` and `attn_q_b` tensors could be another candidate for reducing bpw of active model weights, and I wouldn't go below 5 bits for `attn_output`.

If these models weren't so giant so that everything takes forever, one could properly study the impact of reducing bpw for the various tensors involved.