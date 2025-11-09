## 🔀 [Pull Request #896](https://github.com/ikawrakow/ik_llama.cpp/pull/896) - Allow quantization of ffn_gate_inp

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/quantize_ffn_gate_inp` |
| **Target Branch** | `main` |
| **Created** | 2025-11-04 |
| **Updated** | 2025-11-05 |
| **Merged** | 2025-11-05 |

---

## 📄 Description

Recent MoE models have a lot of experts. This means that the `ffn_gate_inp` tensors, which multiplies the output of self attention to create experts routing probabilities, is not really small (size of embedding times number of experts). On the main branch (and also in mainline `llama.cpp`) this tensor is left as `f32` and there is no provision to quantize it. The result is that the `ffn_gate_inp` matrix multiplication represents a non-negligible fraction of overall computation and data that needs to be fetched from memory during token generation (TG).

This PR adds the ability to quantize the `ffn_gate_inp` tensors by adding
```
--ffn-gate-inp-type type
```
to the `llama-quantize` command.

Here is a benchmark example of Qwen3-30B-A3B running on a Ryzen-7950X CPU with `ffn_gate_inp` left as `f32` or quantized to `Q8_0`

| model            |       size |      test |    t/s (f32)     |     t/s (Q8_0)    |  Speedup |
| ---------------- | ---------: | --------: | ---------------: | ---------------: | -------- |
| qwen3moe IQ4_KS  |  15.83 GiB |    pp2048 |    517.21 ± 2.81 |    521.49 ± 2.89 |  1.008   |   
| qwen3moe IQ4_KS  |  15.83 GiB |     tg128 |     26.36 ± 0.02 |     27.02 ± 0.10 |  1.025   |   
| qwen3moe IQ2_XXS |   8.05 GiB |    pp2048 |    517.82 ± 3.36 |    527.56 ± 4.61 |  1.019   |   
| qwen3moe IQ2_XXS |   8.05 GiB |     tg128 |     41.07 ± 0.43 |     43.62 ± 0.07 |  1.062   |

We observe non-negligible TG performance gain. In terms of model quality, the `f32` and `Q8_0` models are indistinguishable based on `PPL` and quick evaluation of responses to a set of prompts. Obviously, anyone who wants to take advantage of this option should make their own evaluation of potential quality degradation due to `ffn_gate_inp` quantization.

On CUDA the performance gain is less, about 1% in TG performance for the `IQ2_XXS` version of Qwen3-30B-A3B (which I can run fully offloaded on my RTX-4080 GPU).