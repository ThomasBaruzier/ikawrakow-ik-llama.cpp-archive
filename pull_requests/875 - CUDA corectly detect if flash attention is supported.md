## 🔀 [Pull Request #875](https://github.com/ikawrakow/ik_llama.cpp/pull/875) - CUDA: corectly detect if flash attention is supported 

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/fattn_is_supported` |
| **Target Branch** | `main` |
| **Created** | 2025-10-28 |
| **Updated** | 2025-10-29 |
| **Merged** | 2025-10-29 |

---

## 📄 Description

This PR does two things:
* Allows more cases whee quantized KV cache can be used
* Correctly determines if FA is supported on CUDA for the given situation

With `VOLTA` or better Nvidia GPU basically any common attention head size with any quantization type is supported, except for the DeepSeek-V3/R1 case, which requires `TURING` or better.

---

## 💬 Conversation

👤 **Nexesenex** commented on **2025-10-28** at **15:45:42**

A few questions and observations:
-> IQ4_NL and Q6_0 do not support head size 64 for V cache? (I think you wrote it in the past, just want to be sure as of today).
-> Is KV F16 / Q8_0 (and Q8_0 / F16 for the sake of curiosity) supported for head size 256? --- Edit: What about for head size 192? (K F16 / V Q8_0 is a fair compromise, quality wise.)
-> Your conditionality for FA ALL QUANTS for head-size 128 seems to be a bit restrictive, because it excludes Q6_0 and IQ4_NL from the scheme, while Q6_0 is the best sub Q8_0 K cache quant, and IQ4_NL the best bang for our bucks for V.
-> Also, I think that the KV quants where V is a higher bpw than K can likely be skipped once and for all, because there's no point using them, due to the higher sensitivity of K cache to quantization.

---

👤 **ikawrakow** commented on **2025-10-28** at **16:58:40**

@Nexesenex 

IIRC, you have 3090s, correct?

In that case, the vector FA kernels will only get used if the KV cache is not quantized and it is estimated that the vector kernel is faster than the MMA kernel. If it is quantized, it will always go to the MMA kernel. The MMA kernel supports any combination of K and V cache (for quantized cache, it simply converts to `F16` first, and then uses the `F16` implementation). So, basically, the combinations that you see in `fattn-vec-f16.cu` and `fattn-vec-f32.cu` are only there so that old GPUs that do not support MMA can use them. In that case, yes, only the combinations written in these files are supported. 

There are of course also the restrictions imposed by the `llama.cpp` code, which limits the quantization types to `Q8_0, Q6_0, Q4_0, Q4_1, Q5_0, Q5_1, IQ4_NL`.

To see what happens, simply use a small model that loads quickly, and just run commands with whatever `-ctk/-ctv` arguments you want to try. If it doesn't work as described, let me know.

---

👤 **Nexesenex** commented on **2025-10-28** at **18:29:53**

Thanks you for all those precious informations, @Ikawrakow, now I know what to NOT look for for my own needs. Yes, I do run 2x3090, 1x RTX A4000 and I might be able to add my old 3060 to the mix when I get my new mobo this week.

A point intrigues me, though. Are those KV Quants scope limitations overwhelming or relatively superficial? An KV cache in IQ4_KS might come close to IQ4_NL in terms of precision, and allow a size gain of approx. 5% for the context if we account for the compute cache. A KV cache in IQ5_K/IQ4_KS might also be a banger at 4.9bpw.

---

👤 **ikawrakow** commented on **2025-10-29** at **05:28:44**

All quants that use blocks of 256 are not really suitable for KV cache, where attention head sizes are less than 256 for most models. The other thing is that quantization must be really fast, else there is a significant performance hit as the conversion from `f32` to the quantized data must be done for every processed token. This requirement kills the advantage of types where quantization is more computationally demanding. For instance, the quantization methods used for `IQ4_NL` and `Q6_0` when storing into the cache is much simpler (and hence less accurate) than the quantization method used during offline model weight quantization.