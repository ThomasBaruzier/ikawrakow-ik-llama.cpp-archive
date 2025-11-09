## 🔀 [Pull Request #795](https://github.com/ikawrakow/ik_llama.cpp/pull/795) - Fix dequantization when requantizing

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/fix_dequantize_when_requantizing` |
| **Target Branch** | `main` |
| **Created** | 2025-09-24 |
| **Updated** | 2025-09-24 |
| **Merged** | 2025-09-24 |

---

## 📄 Description

Fixes [#793](https://github.com/ikawrakow/ik_llama.cpp/issues/793) 

When quantizing with `--allow-requantize` and the tensor being converted is quantized, the implementation inherited from mainline makes the by now legendary assumption of contiguous quantized blocks when converting the quantized data to `f32`. This fails for quantization types that use per tensor row scales (trellis quants, `IQX_KS`, `IQX_KSS`). This PR fixes the issue.