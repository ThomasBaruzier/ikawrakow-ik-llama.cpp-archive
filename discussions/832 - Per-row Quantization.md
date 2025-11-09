## 🗣️ [Discussion #832](https://github.com/ikawrakow/ik_llama.cpp/discussions/832) - Per-row Quantization

| **Author** | `kimjoohyungsd` |
| :--- | :--- |
| **State** | ✅ **Open** |
| **Created** | 2025-10-14 |
| **Updated** | 2025-10-16 |

---

## 📄 Description

Hi, I'm curious about the quantization on llama.cpp. In structure data types for quantization in llama.cpp such as q4_0, this project have limited the scope of quantization block size to at most 256. In additon, I've found out that Int8 SIMD instructions are utilized to accelerate inference.
Given this information, I would like to ask if this project have tried to add vector-wise quantization blocks as leveraged by most of quantization methods for NVIDIA GPU? If so, I would like to ask was there any speed overhead due to fine-grained group size in CPU SIMD inference?

---

## 💬 Discussion

👤 **ikawrakow** commented on **2025-10-14** at **11:36:29**

As this is not an issue, I took the liberty to convert the question to a discussion.

>  I would like to ask if this project have tried to add vector-wise quantization blocks as leveraged by most of quantization methods for NVIDIA GPU?

Can you be more specific?

---

👤 **kimjoohyungsd** commented on **2025-10-15** at **00:47:18**

_Given this information, I would like to ask if this project have tried to add vector-wise quantization blocks as leveraged by most of quantization methods for NVIDIA GPU? If so, I would like to ask was there any speed overhead due to fine-grained group size in CPU SIMD inference?_

 Most of quantization papers that quantizes both weights and activations to reduce memory demand and increasing parallel Computing ability such as QuaRot, SpinQuant, Qserve in NVIDIA GPU hardware. As far as I know, these methods  have implemented per-channel for weight, per-token for activation. Issue that arise when weight and activations are quantized group-wise as in llama.cpp in NVIDIA GPU is that dequantizing partial sums need to be executed on CUDA cores which is much slower than Tensor cores, underutilizing maximum computing performace.

However, llama.cpp quantizes both weight and activations group-wise. So, I was wondering whether inference overhead that comes from dequantizing each partial sum in SIMD instructions in CPU is less severe than that of NVIDIA GPU. So I asked ChatGpt for the reason of group-wise quantization in CPU. What they replied is cache access alignment matters in CPU SIMD instructions and implementing per-channel (per-token) quantization might break Cache access misalignment. 

The reason, I've asked you specifically is that on open discussion in llama.cpp github repository january, 2024, you said that you have implemented per-row quantization in your modified version of llama.cpp. So I would like to ask whether there was actual speed improvements when quantized per-row compared to preexisting group_wise quantization? Thanks in advance

---

👤 **ikawrakow** commented on **2025-10-15** at **15:45:44**

So, there are several quantization types in `llama.cpp`:
* Legacy quants (`Q4_0, Q4_1, Q5_0, Q5_1, Q8_0`): these use blocks of 32 weights. Each block has one or two `fp16` scales
* k-quants (`Q2_K, Q3_K, Q4_K, Q5_K, Q6_K`): These use super-blocks of 256 weights. Each super-block has one or two `fp16` scales. Super-blocks are divided into blocks of 16 or 32 weights. Each block uses one or two quantized integer scales
* i-quants (`IQ2_XXS, IQ2_XS, IQ2_S, IQ3_XXS, IQ3_S, IQ4_XS`). Same blocks in super-blocks arrangement.

In `ik_llama.cpp` there are many additional quantization types. All use super-blocks of 256 and blocks of 16 or 32 as k- and i-quants. Some don't have the super-block `fp16` scale(s), but instead have a single float scale per tensor row (`IQ1_KT, IQ2_KT, IQ3_KT, IQ4_KT, IQ2_KS, IQ3_KS, IQ4_KS, IQ5_KS). This is what I meant with my comment in January of 2024, and not that we will suddenly not have any quantization blocks.

CUDA token generation performance is heavily memory bound, so performance is mainly determined by the number of bits per weight, and is not really dependent on block sizes.

CUDA prompt processing performance (a.k.a. prefill) is mostly determined by block size. Quants using blocks of 32 tend to be 10-30% faster than quants using blocks of 16. This is easily understandable, given the type of SIMD instructions we have available.

CPU token generation speed is even more heavily memory bound, so performance is determined by the number of bits per weight. But because the CPU does not much as much computing power as a GPU, there is also some dependence on the complexity of the bit packing.

CPU prompt processing speed is, to first order, independent of the quantization type. At least this is case in `ik_llama.cpp`. `llama.cpp` uses very inefficient CPU matrix multiplication implementations for most quantization types, so there you can find nearly an order of magnitude prefill performance difference between quantization types. 

Usage of per-row float scale has a negligible impact on performance compared to super-blocks. The main advantage is that we can save 0.0625 or 0.125 bits per model weight, thus having a slightly smaller quantized model size.

Does this answer your question?

---

👤 **kimjoohyungsd** commented on **2025-10-16** at **01:51:03**

_Usage of per-row float scale has a negligible impact on performance compared to super-blocks. The main advantage is that we can save 0.0625 or 0.125 bits per model weight, thus having a slightly smaller quantized model size._ Yes, this answered my question. If possible, may I ask underlying reason behind this issue?