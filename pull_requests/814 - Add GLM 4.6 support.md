## 🔀 [Pull Request #814](https://github.com/ikawrakow/ik_llama.cpp/pull/814) - Add GLM 4.6 support

| **Author** | `Downtown-Case` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `main` |
| **Target Branch** | `main` |
| **Created** | 2025-10-01 |
| **Updated** | 2025-10-07 |
| **Merged** | 2025-10-01 |

---

## 📄 Description

Fixes: https://github.com/ikawrakow/ik_llama.cpp/issues/812

By marking a few tensors as optional. GLM 4.6 loads and inferences for me, if you wish to test: https://huggingface.co/Downtown-Case/GLM-4.6-128GB-RAM-IK-GGUF

Plucked from: https://github.com/ggml-org/llama.cpp/pull/16359

***

Random aside, but this might be a good opportunity to make sure the MTP (multi-token prediction) tensors are never loaded into RAM, since they are not used anyway? Their size is on the order of ~2GB quantized, I think, which is signficant when squeezing in a >3bpw quant.



- [x] I have read the [contributing guidelines](https://github.com/ggerganov/llama.cpp/blob/master/CONTRIBUTING.md)

---

## 💬 Conversation

👤 **ubergarm** commented on **2025-10-01** at **14:47:21**

@Downtown-Case 

Ahh thanks for opening this as otherwise I was gonna do it. I'll test it shortly, downloading new GLM-4.6 safetensors now and about to convert. i'll report back here within a few hours

---

👤 **Downtown-Case** commented on **2025-10-01** at **15:00:50**

@ubergarm 

See the workaround here, otherwise it will fail to convert from .safetensors!

https://github.com/ggml-org/llama.cpp/issues/16361#issuecomment-3353848502

***

Also, per a discussion here:

https://huggingface.co/steampunque/GLM-4.5-Air-Hybrid-GGUF/discussions/1#68dd13c2744d0e48b5723179

I think we can convert layer 92 to IQ1_S and shave off a gigabyte or two, since it's unused anyway.

---

👤 **ubergarm** commented on **2025-10-01** at **15:35:53**

> I think we can convert layer 92 to IQ1_S and shave off a gigabyte or two, since it's unused anyway.

So those nextn layers for MTP are marked as `TENSOR_SKIP` and ignored (which prints out in llama-server debug on startup) so they are not actually taking up any RAM or VRAM but just a little bit of disk space. The idea is to leave them something reasonable like iq5_ks or whatever you like so that in the future possibly if MTP is added then u don't have to re-cook all the quants. https://github.com/ikawrakow/ik_llama.cpp/pull/668#issuecomment-3155053188

If you want to quantize them very small like that anyway be careful as it might throw an error depending on the imatrix data so u might have to use `--ignore-imatrix-rules` or it could bail at the end. https://github.com/ikawrakow/ik_llama.cpp/pull/668#issuecomment-3156525915 

I'm converting safetensors to GGUF now using mainline lcpp, so will be able to test your PR soon! :crossed_fingers:

---

👤 **Downtown-Case** commented on **2025-10-01** at **16:07:23**

I see. Makes sense. Since they're not loaded, leaving them at a reasonable quant seems like a good idea.

For what it's worth, they didn't throw an error quantizing to IQ1_S.

---

👤 **ubergarm** commented on **2025-10-01** at **16:15:49**

So far so good, using ur PR to calculate imatrix from the full bf16 currently:

```bash
model=/mnt/data/models/ubergarm/GLM-4.6-GGUF/GLM-160x19B-4.6-BF16-00001-of-00015.gguf
numactl -N 1 -m 1 \
./build/bin/llama-imatrix \
    -m "$model" \
    -fa \
    --no-fused-up-gate \
    -f ubergarm-imatrix-calibration-corpus-v02.txt \
    -o /mnt/data/models/ubergarm/GLM-4.6-GGUF/imatrix-GLM-4.6-BF16.dat \
    --verbosity 1 \
    --layer-similarity \
    --seed 1337 \
    --ctx-size 512 \
    -ub 4096 -b 4096 \
    --numa numactl \
    --threads 128 \
    --threads-batch 192 \
    --no-mmap

system_info: n_threads = 128 (n_threads_batch = 192) / 768 | AVX = 1 | AVX_VNNI = 1 | AVX2 = 1 | AVX512 = 1 | AVX512_VBMI = 1 | AVX512_VNNI = 1 | AVX512_BF16 = 1 | FMA = 1 | NEON = 0 | SVE = 0 | ARM_FMA = 0 | F16C = 1 | FP16_VA = 0 | WASM_SIMD = 0 | BLAS = 0 | SSE3 = 1 | SSSE3 = 1 | VSX = 0 | MATMUL_INT8 = 0 | LLAMAFILE = 1 |
compute_imatrix: tokenizing the input ..
compute_imatrix: tokenization took 1050.35 ms
compute_imatrix: computing over 814 chunks with batch_size 512
compute_imatrix: 13.13 seconds per pass - ETA 2 hours 58.10 minutes
======================================= HAVE_FANCY_SIMD is defined
[1]17.7095,[2]6.8624,[3]4.4259,[4]3.1997,[5]2.5997,[6]2.2235,[7]2.0004,[8]1.8473,[9]1.8407,
save_imatrix: entry '             blk.48.ffn_gate_exps.weight' has partial data (99.38%) 1 out of 160 experts are missing data Storing **but be aware**
save_imatrix: entry '             blk.48.ffn_down_exps.weight' has partial data (99.38%) 1 out of 160 experts are missing data Storing **but be aware**
save_imatrix: entry '               blk.48.ffn_up_exps.weight' has partial data (99.38%) 1 out of 160 experts are missing data Storing **but be aware**

save_imatrix: stored collected data after 10 chunks in /mnt/data/models/ubergarm/GLM-4.6-GGUF/imatrix-GLM-4.6-BF16.dat
[10]1.7624,[11]1.8774,[12]1.9641,[13]2.0361,[14]2.0966,[15]1.9987,[16]1.9195,[17]1.8662,[18]1.8114,[19]1.7560,
```

Seems like a well-behaved model in terms of routed experts having enough imatrix data so that is good.

I'll confirm the imatrix has all the tensors (except mtp / layer 92 of course) and upload it then continue quantizing, testing, and releasing.

So far so good! Thanks!

---

👤 **ikawrakow** approved this pull request ✅ on **2025-10-01** at **18:37:19**

---

👤 **ubergarm** commented on **2025-10-01** at **18:55:07**

@Downtown-Case 

imatrix is available run off the bf16 here: https://huggingface.co/ubergarm/GLM-4.6-GGUF/blob/main/imatrix-GLM-4.6-BF16.dat

Seems to have everything and am using it to quantize now:

```bash
load_imatrix: imatrix dataset='ubergarm-imatrix-calibration-corpus-v02.txt'
load_imatrix: loaded 1001 importance matrix entries from /mnt/data/models/ubergarm/GLM-4.6-GGUF/imatrix-GLM-4.6-BF16.dat computed on 814 chunks
prepare_imatrix: have 1001 importance matrix entries
```

---

👤 **Ph0rk0z** commented on **2025-10-02** at **12:32:34**

Still get a curious message when offloading regarding those tensors.


```
model has unused tensor blk.92.attn_norm.weight (size = 20480 bytes) -- ignoring
model has unused tensor blk.92.attn_q.weight (size = 35389440 bytes) -- ignoring
model has unused tensor blk.92.attn_k.weight (size = 2949120 bytes) -- ignoring
model has unused tensor blk.92.attn_v.weight (size = 2949120 bytes) -- ignoring
model has unused tensor blk.92.attn_q.bias (size = 49152 bytes) -- ignoring
model has unused tensor blk.92.attn_k.bias (size = 4096 bytes) -- ignoring
model has unused tensor blk.92.attn_v.bias (size = 4096 bytes) -- ignoring
model has unused tensor blk.92.attn_output.weight (size = 35389440 bytes) -- ignoring
model has unused tensor blk.92.attn_q_norm.weight (size = 512 bytes) -- ignoring
model has unused tensor blk.92.attn_k_norm.weight (size = 512 bytes) -- ignoring
model has unused tensor blk.92.post_attention_norm.weight (size = 20480 bytes) -- ignoring
model has unused tensor blk.92.ffn_gate_inp.weight (size = 3276800 bytes) -- ignoring
model has unused tensor blk.92.exp_probs_b.bias (size = 640 bytes) -- ignoring
Tensor blk.92.ffn_gate_exps.weight buffer type overriden to CPU
model has unused tensor blk.92.ffn_gate_exps.weight (size = 540672000 bytes) -- ignoring
Tensor blk.92.ffn_down_exps.weight buffer type overriden to CPU
model has unused tensor blk.92.ffn_down_exps.weight (size = 540672000 bytes) -- ignoring
Tensor blk.92.ffn_up_exps.weight buffer type overriden to CPU
model has unused tensor blk.92.ffn_up_exps.weight (size = 540672000 bytes) -- ignoring
model has unused tensor blk.92.ffn_gate_shexp.weight (size = 4423680 bytes) -- ignoring
model has unused tensor blk.92.ffn_down_shexp.weight (size = 5406720 bytes) -- ignoring
model has unused tensor blk.92.ffn_up_shexp.weight (size = 4423680 bytes) -- ignoring
model has unused tensor blk.92.nextn.eh_proj.weight (size = 22528000 bytes) -- ignoring
model has unused tensor blk.92.nextn.enorm.weight (size = 20480 bytes) -- ignoring
model has unused tensor blk.92.nextn.hnorm.weight (size = 20480 bytes) -- ignoring
model has unused tensor blk.92.nextn.shared_head_norm.weight (size = 20480 bytes) -- ignoring
```

Model works though.

---

👤 **ubergarm** commented on **2025-10-02** at **17:09:17**

@Ph0rk0z 

> Still get a curious message when offloading regarding those tensors.

When GLM-4.5 supported was added, a new concept `TENSOR_SKIP` was introduced. This is because GLM-4.5 safetensors has MTP "nextn" blk.92 tensors which are used *only* for multi token predictions stuff which is *not* implemented in any llamas afaik. 

So the strategy was to go ahead and quantize them without any imatrix data, include them in the gguf files, but mark them as `TENSOR_SKIP` which will print out that message on startup e.g. `model has unused tensor ...  -- ignoring`

So the tensors only take up space on disk, but are *not* loaded into RAM/VRAM on inference time. 

In the future, if MTP support is added, these quants will already have some data available there without needing to re-download the entire model.

---

👤 **Ph0rk0z** commented on **2025-10-02** at **17:28:07**

I remember that part, but there were new tensors for 4.6

```
Tensor blk.92.ffn_gate_exps.weight buffer type overriden to CPU
model has unused tensor blk.92.ffn_gate_exps.weight (size = 540672000 bytes) -- ignoring
Tensor blk.92.ffn_down_exps.weight buffer type overriden to CPU
model has unused tensor blk.92.ffn_down_exps.weight (size = 540672000 bytes) -- ignoring
Tensor blk.92.ffn_up_exps.weight buffer type overriden to CPU
```

None of the 4.5 tensors have the second `Tensor blk.92.ffn_up_exps.weight buffer type overriden to CPU` message.

---

👤 **ubergarm** commented on **2025-10-02** at **20:14:36**

@Ph0rk0z 

I just tested GLM-4.6-smol-IQ2_KS on my local rig and see similar messages e.g.

```bash
Tensor blk.92.ffn_gate_exps.weight buffer type overriden to CPU
model has unused tensor blk.92.ffn_gate_exps.weight (size = 344555520 bytes) -- ignoring
Tensor blk.92.ffn_down_exps.weight buffer type overriden to CPU
model has unused tensor blk.92.ffn_down_exps.weight (size = 345702400 bytes) -- ignoring
Tensor blk.92.ffn_up_exps.weight buffer type overriden to CPU
model has unused tensor blk.92.ffn_up_exps.weight (size = 344555520 bytes) -- ignoring
```

Going back and looking at GLM-4.5 it does have routed exps layers on blk 92 e.g. `blk.92.ffn_(gate|down|up)_exps.weight`.

My impression is the logging is due to `-ot exps=CPU` which I am using here too. So it matches the override, and then it says `ignoring` which is what I would expect.

Perhaps you were not running `-ot exps=CPU` on GLM-4.5 as I would expect it to look similar (but don't have it on a rig with GPU to try).

Anyway, it seems okay to me, but if you think there is an issue let me know. thanks!

---

👤 **Ph0rk0z** commented on **2025-10-02** at **20:21:51**

I am using my exact same OT for 4.6 as I had for 4.5. It's definitely new. Not sure why it tries to override these layers but not the other unused ones. I only bring it up in case there is some place the ignore is skipped in the code leading to a possible bug, i.e they get loaded into sysram and then unused.

---

👤 **Thireus** commented on **2025-10-06** at **12:46:56**

Has anyone figured out how to enable thinking when using the openai compatible API? Thinking works fine on the web UI, but not via the chat API where I always get `<think></think>` as first tokens of the model's response.

---

👤 **Ph0rk0z** commented on **2025-10-06** at **13:35:54**

That's strange. I had to set reasoning budget to 0 otherwise it would think in chat completions. On text completions it's a matter of using the correct preset.

---

👤 **Thireus** commented on **2025-10-06** at **13:55:27**

Thanks @Ph0rk0z, you are right! It is enabled by default, for some reason my system prompt disables it 😳, so now I'm trying to figure out which part of my system prompt triggers this! I don't have any /no_think or similar...

Edit: Looks like the model decide on its own that it shouldn't think... especially when the system prompt is quite long and detailed about answer formatting and how to process information.

---

👤 **Downtown-Case** commented on **2025-10-07** at **14:44:57**

Not sure if related, but I've heard the default (GGUF) chat template is bugged, but unsloth fixed it in their upload: https://huggingface.co/unsloth/GLM-4.6-GGUF