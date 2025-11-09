## 📌 [Issue #779](https://github.com/ikawrakow/ik_llama.cpp/issues/779) - Use of both VRAM and RAM in context.

| **Author** | `iiinferus-ai` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-09-14 |
| **Updated** | 2025-09-26 |

---

## 📄 Description

Good afternoon!
I have a system with 512gb RAM and 32gb VRAM (Dual Xeon and 5090) for the Qwen3-Coder-480B-A35B-Instruct-UD-Q6_K_XL model (408gb). How can I use both VRAM and RAM in the context?

My current command is (the context is loaded only in VRAM):
`numactl --interleave=all ./build/bin/llama-sweep-bench \
    --alias unsloth/Qwen3-Coder-480B-A35B-Instruct-UD-Q6_K_XL \
    --model /home/user/Qwen3-Coder-480B-A35B-Instruct-UD-Q6_K_XL/Qwen3-Coder-480B-A35B-Instruct-UD-Q6_K_XL-00001-of-00009.gguf \
    -rtr \
    --ctx-size 64000 \
    --temp 0.6 --top-k 0 --top-p 1.0 --min-p 0.1 --repeat-penalty 1.0 \
    -fa \
    -mla 3 -fmoe -amb 512 \
    -ctk f16 \
    -b $((4 * 1024)) -ub $((4 * 1024)) \
    --n-gpu-layers 99 \
    --numa numactl \
    --override-tensor exps=CPU,attn_kv_b=CPU \
    --override-tensor down_exps=CPU,gate_exps=CPU,up_exps=CPU \
    --threads 96 \
    --no-mmap
    --warmup-batch`

`cmake -B build -DCMAKE_BUILD_TYPE=Release -DGGML_CUDA=ON -DGGML_BLAS=OFF -DGGML_AVX=ON -DGGML_AVX512=ON -DGGML_AVX512_BF16=ON -DGGML_AVX512_VBMI=ON -DGGML_AVX512_VNNI=ON -DGGML_CUDA_FA_ALL_QUANTS=1 -DGGML_SCHED_MAX_COPIES=1 -DGGML_CUDA_IQK_FORCE_BF16=1`


When using `-ctx-size 64000 \ --gpu-layers-draft 32000 \` the context (64000) is still loaded only in VRAM.

---

## 💬 Conversation

👤 **ikawrakow** commented on **2025-09-23** at **08:15:48**

The context (a.k.a. KV cache) is stored on the device where the associated attention tensors are. With your command, which puts all attention tensors on the GPU, the KV cache is stored in VRAM. You have the following options:
* To have only part of the KV cache stored in VRAM, use `--n-gpu-layers N`, where `N` is less than the number of of layers in the model.
* Use `-nkvo`, which will result in all KV cache being stored in RAM

Both options will reduce inference speed.

Another way to reduce VRAM usage (and so have the ability to have more context in VRAM) is to use a smaller `-ub` (and also `-b`). This will lead to the same TG performance but a lower prompt processing speed.