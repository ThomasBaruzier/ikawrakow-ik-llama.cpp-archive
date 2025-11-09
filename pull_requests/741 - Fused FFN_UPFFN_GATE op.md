## 🔀 [Pull Request #741](https://github.com/ikawrakow/ik_llama.cpp/pull/741) - Fused FFN_UP+FFN_GATE op

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/fused_ffn_up_gate` |
| **Target Branch** | `main` |
| **Created** | 2025-08-30 |
| **Updated** | 2025-09-24 |
| **Merged** | 2025-08-31 |

---

## 📄 Description

Similar to the fused `ffn_up/ffn_gate` op for MoE models, but applicable to regular FFN networks. As such, more relevant for dense models, where I observe a ~2.5% performance improvement for prompt processing, and ~0.6% for token generation for LlaMA-3-8B.

For MoE models there is no gain when there are no shared experts and no dense-lead layers, and a very minor performance gain (sub-1%) for models with shared experts (e.g., DeepSeek).

Only implemented for the CPU and CUDA back-ends for now. But given that `ik_llama.cpp` is predominantly used on these platforms, I decided to turn the option on by default. It can be turned off using `-no-fug` or `--no-fused-up-gate`.

---

## 💬 Conversation

👤 **ubergarm** commented on **2025-09-23** at **20:03:40**

Heads up @Thireus and other quant cookers, if you make an imatrix for a big MoE e.g. Kimi-K2 or DeepSeek-V3.1-Terminus you will *need* to add `--no-fused-up-gate` to explicitly disable this feature or your resulting imatrix.dat will be missing the first N dense layer and shared expert (gate|up) tensor importance data.

I ran into this on the updated recent Kimi-K2, but only when I hit it again on Terminus did I notice this was explicitly enabled and so have now updated my imatrix scripts. Also if you don't use this `-no-fug` and try to use `--layer-similarity` it will exit saying this: `Oops, inconsistent ffn vs last_input size` which is a good clue to remember to add `-no-fug` hah...

```bash
# used triton-cpu method cast fp8 safetensors to bf16 safetensors
# used mainline lcpp to hf convert bf16 safetensors to bf16 GGUFs
# you can adjust batch sizes as desired for your target hardware
numactl -N 1 -m 1 \
./build/bin/llama-imatrix \
    -m "$model" \
    -fa -mla 1 \
    --no-fused-up-gate \
    -f ubergarm-imatrix-calibration-corpus-v02.txt \
    -o /mnt/data/models/ubergarm/DeepSeek-V3.1-Terminus-GGUF/imatrix-DeepSeek-V3.1-Terminus-Q8_0.dat \
    --verbosity 1 \
    --layer-similarity \
    --ctx-size 512 \
    --numa numactl \
    --threads 128 \
    --threads-batch 192 \
    -ub 4096 -b 4096 \
    --no-mmap
```

---

👤 **ikawrakow** commented on **2025-09-24** at **05:49:56**

@ubergarm Thanks for pointing out that one needs to disable the fused up+gate op for imatrix calculations. In my usual style I forgot to mention that in the PR.