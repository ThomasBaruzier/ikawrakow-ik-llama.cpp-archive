## 🔀 [Pull Request #722](https://github.com/ikawrakow/ik_llama.cpp/pull/722) - Log for debugging [#721](https://github.com/ikawrakow/ik_llama.cpp/issues/721)

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/debug_issue_721` |
| **Target Branch** | `main` |
| **Created** | 2025-08-23 |
| **Updated** | 2025-08-23 |
| **Merged** | 2025-08-23 |

---

## 📄 Description

Closes [#721](https://github.com/ikawrakow/ik_llama.cpp/issues/721)

---

## 💬 Conversation

👤 **ubergarm** commented on **2025-08-23** at **16:47:44**

This specific commit seems to be causing `llama-perplexity` to throw:

```bash
llama_new_context_with_model: KV self size  =  274.50 MiB, c^KV (f16):  274.50 MiB, kv^T: not used
llama_new_context_with_model:        CPU  output buffer size =     3.95 MiB
llama_new_context_with_model:        CPU compute buffer size =  2132.00 MiB
llama_new_context_with_model: graph nodes  = 3487
llama_new_context_with_model: graph splits = 1
======================================= HAVE_FANCY_SIMD is defined

system_info: n_threads = 128 (n_threads_batch = 192) / 768 | AVX = 1 | AVX_VNNI = 1 | AVX2 = 1 | AVX512 = 1 | AVX512_VBMI = 1 | AVX512_VNNI = 1 | AVX512_BF16 = 1 | FMA = 1 | NEON = 0 | SVE = 0 | ARM_FMA = 0 | F16C = 1 | FP16_VA = 0 | WASM_SIMD = 0 | BLAS = 0 | SSE3 = 1 | SSSE3 = 1 | VSX = 0 | MATMUL_INT8 = 0 | LLAMAFILE = 1 |
perplexity: tokenizing the input ..
perplexity: tokenization took 527.573 ms
perplexity: calculating perplexity over 561 chunks, n_ctx=512, batch_size=4096, n_seq=8
/home/w/projects/ik_llama.cpp/ggml/src/iqk/iqk_gemm_legacy_quants.cpp:1774: GGML_ASSERT(nrc_x%8 == 0) failed
/home/w/projects/ik_llama.cpp/ggml/src/iqk/iqk_gemm_legacy_quants.cpp:1774: GGML_ASSERT(nrc_x%8 == 0) failed
/home/w/projects/ik_llama.cpp/ggml/src/iqk/iqk_gemm_legacy_quants.cpp:1774: GGML_ASSERT(nrc_x%8 == 0) failed
```

as described better here in the second half of this comment: https://github.com/ikawrakow/ik_llama.cpp/issues/649#issuecomment-3217149956