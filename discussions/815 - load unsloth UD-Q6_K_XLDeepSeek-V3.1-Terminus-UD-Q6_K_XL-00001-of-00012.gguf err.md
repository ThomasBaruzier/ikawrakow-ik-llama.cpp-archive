## 🗣️ [Discussion #815](https://github.com/ikawrakow/ik_llama.cpp/discussions/815) - load unsloth UD-Q6_K_XL/DeepSeek-V3.1-Terminus-UD-Q6_K_XL-00001-of-00012.gguf error:

| **Author** | `calvin2021y` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-10-02 |
| **Updated** | 2025-10-02 |

---

## 📄 Description

I verify the SHA256 and it matched: 6194c58854fa8b56dd031130ae6677e5a477db3844d302f8c9cd9238ed76b400

error:

```sh
NFO [                    main] build info | tid="139830970436928" timestamp=1759376817 build=3901 commit="6051ba25"
INFO [                    main] system info | tid="139830970436928" timestamp=1759376817 n_threads=45 n_threads_batch=90 total_threads=96 system_info="AVX = 1 | AVX_VNNI = 0 | AVX2 = 1 | AVX512 = 1 | AVX512_VBMI = 1 | AVX512_VNNI = 1 | AVX512_BF16 = 1 | FMA = 1 | NEON = 0 | SVE = 0 | ARM_FMA = 0 | F16C = 1 | FP16_VA = 0 | WASM_SIMD = 0 | BLAS = 0 | SSE3 = 1 | SSSE3 = 1 | VSX = 0 | MATMUL_INT8 = 0 | LLAMAFILE = 0 | "
llama_model_load: error loading model: tensor 'blk.54.ffn_down_exps.weight' data is not within the file bounds, model is corrupted or incomplete
llama_load_model_from_file: failed to load model
llama_init_from_gpt_params: error: failed to load model 'unsloth/DeepSeek-V3.1-Terminus-UD-Q6_K_XL-00001-of-00012.gguf'
 ERR [              load_model] unable to load model | tid="139830970436928" timestamp=1759376817 model="unsloth/DeepSeek-V3.1-Terminus-UD-Q6_K_XL-00001-of-00012.gguf"
Segmentation fault (core dumped)
```

---

## 💬 Discussion

👤 **calvin2021y** commented on **2025-10-02** at **03:55:03**

the error come from other file, not the model path in the message : 'unsloth/DeepSeek-V3.1-Terminus-UD-Q6_K_XL-00001-of-00012.gguf'