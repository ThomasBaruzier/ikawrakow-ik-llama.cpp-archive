## 🔀 [Pull Request #907](https://github.com/ikawrakow/ik_llama.cpp/pull/907) - model : Port Minimax M2 from mainline

| **Author** | `firecoperana` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `fcp/minimax` |
| **Target Branch** | `main` |
| **Created** | 2025-11-06 |
| **Updated** | 2025-11-06 |
| **Merged** | 2025-11-06 |

---

## 📄 Description

Add support for Minimax M2 https://github.com/ggml-org/llama.cpp/pull/16831
Closes https://github.com/ikawrakow/ik_llama.cpp/issues/877
Only add model support. Tool calling support is not yet merged.

---

## 💬 Conversation

👤 **ikawrakow** started a conversation on `src/llama-build-context.cpp` on **2025-11-06** at **06:16:13**

The code from here to line 8416 can now be replaced with
```c++
            auto [Qcur, Kcur, Vcur] = llm_build_mul_mat_qkv(gf, cur, 
                    model.layers[il].wqkv, nullptr, model.layers[il].wqk, nullptr,
                    model.layers[il].wq, nullptr, model.layers[il].wk, nullptr, model.layers[il].wv, nullptr,
                    model.layers[il].attn_q_norm, model.layers[il].attn_k_norm, 0, il); 
```
In that way we don't need to worry about how Q, K, V should be reshaped, we automatically get Q,K,V fusion for this model if the user requested via `-mqkv`, and we use a standardized way to compute attention (when attention is standard as it is here).

It would be useful to do that so it can get tested as part of this PR.

> 👤 **firecoperana** replied on **2025-11-06** at **13:39:51**
> 
> I get error `ik_llama.cpp\ggml\src\ggml.c:6358: GGML_ASSERT(ggml_can_repeat(b, a)) failed` when loading the model using cpu only backend after incorporating both changes. I think it's that the buffer of a is null that caused the crash. 
> Call Stack:
> ```
>  	ggml.dll!ggml_abort(const char * file, int line, const char * fmt, ...) Line 272	C
>  	ggml.dll!ggml_mul_impl(ggml_context * ctx, ggml_tensor * a, ggml_tensor * b, bool inplace) Line 6360	C
> >	ggml.dll!ggml_fused_rms_norm_impl(ggml_context * ctx, ggml_tensor * a, ggml_tensor * b, float eps, bool inplace) Line 7277	C
>  	ggml.dll!ggml_fused_rms_norm(ggml_context * ctx, ggml_tensor * a, ggml_tensor * b, float eps) Line 7305	C
>  	llama.dll!llm_build_context::llm_build_norm(ggml_context * ctx, ggml_tensor * cur, const llama_hparams & hparams, ggml_tensor * mw, ggml_tensor * mb, llm_norm_type type, const std::function<void __cdecl(ggml_tensor *,char const *,int)> & cb, int il, float scale_eps) Line 582	C++
>  	llama.dll!llm_build_context::llm_build_mul_mat_qkv(ggml_cgraph * gf, ggml_tensor * cur, ggml_tensor * wqkv, ggml_tensor * bqkv, ggml_tensor * wqk, ggml_tensor * bqk, ggml_tensor * wq, ggml_tensor * bq, ggml_tensor * wk, ggml_tensor * bk, ggml_tensor * wv, ggml_tensor * bv, ggml_tensor * q_norm, ggml_tensor * k_norm, float attention_scale, int il) Line 1348	C++
>  	llama.dll!llm_build_context::build_minimaxm2() Line 8418	C++
> ```

> 👤 **ikawrakow** replied on **2025-11-06** at **14:07:26**
> 
> Oh, I see. The `attn_q_norm` and `attn_k_norm` tensors are not just 1d with the size of one attention head, but rather `head_size x head_count`, so the usual trick of fusing the `RMS_NORM`, does not work. Sorry. Ignore the two comments then.

---

👤 **ikawrakow** started a conversation on `src/llama-load-tensors.cpp` on **2025-11-06** at **06:18:28**

Here you can use
```
use_mmap_buffer &= !merge_qkv(tn, i, 0);
```
to create `wk, wq` and `wv`.

---

👤 **ikawrakow** approved this pull request ✅ on **2025-11-06** at **06:18:53**