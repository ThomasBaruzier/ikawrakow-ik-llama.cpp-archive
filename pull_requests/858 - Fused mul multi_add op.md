## 🔀 [Pull Request #858](https://github.com/ikawrakow/ik_llama.cpp/pull/858) - Fused mul + multi_add op

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/fused_mul_multiadd` |
| **Target Branch** | `main` |
| **Created** | 2025-10-23 |
| **Updated** | 2025-10-24 |
| **Merged** | 2025-10-24 |

---

## 📄 Description

This PR adds a new op, `GGML_FUSED_MUL_MUTI_ADD`, that fuses the multiplication of the experts with their respective weight, followed by the addition of the individual experts results into the output of the MoE FFN compute graph. It is implemented for the CPU and on CUDA. It is `ON` by default, but can be disabled via `-no-mmad` or `-no-fused-mul-multiadd`.

We get solid 4-6% PP and 1-2% TG performance gains on CUDA with full offload, and also when running CPU only. For hybrid inference I'm seeing ~2% PP performance gains for GLM-4.5-AIR-IQ2_XXS and GPT-OSS-120B-MXFP4 with all experts left on the CPU and batch/u-batch size of 4096.