## 📌 [Issue #831](https://github.com/ikawrakow/ik_llama.cpp/issues/831) - Per-row Quantization

| **Author** | `kimjoohyungsd` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-10-14 |
| **Updated** | 2025-10-14 |

---

## 📄 Description

Hi, I'm curious about the quantization on llama.cpp. In structure data types for quantization in llama.cpp such as q4_0, this project have limited the scope of quantization block size to at most 256. In additon, I've found out that Int8 SIMD instructions are utilized to accelerate inference.
Given this information, I would like to ask if this project have tried to add vector-wise quantization blocks as leveraged by most of quantization methods for NVIDIA GPU? If so, I would like to ask was there any speed overhead due to fine-grained group size in CPU SIMD inference?