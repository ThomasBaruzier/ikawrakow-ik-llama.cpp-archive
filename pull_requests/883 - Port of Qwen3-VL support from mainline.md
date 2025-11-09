## 🔀 [Pull Request #883](https://github.com/ikawrakow/ik_llama.cpp/pull/883) - Port of Qwen3-VL support from mainline

| **Author** | `Thireus` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `tr/qwen3-vl-clean` |
| **Target Branch** | `main` |
| **Created** | 2025-10-30 |
| **Updated** | 2025-11-05 |
| **Merged** | 2025-11-04 |

---

## 📄 Description

- convert_hf_to_gguf.py - Not touched, use llama.cpp to convert model instead
- sycl and metal support for imrope not added - See ggml-metal and ggml-sycl folders of https://github.com/ggml-org/llama.cpp/pull/16780/files
- Vulkan support for imrope not tested
- Code not tested against any GGUF (looking for testers...)

Source: https://github.com/ggml-org/llama.cpp/pull/16780

- [x] I have read the [contributing guidelines](https://github.com/ggerganov/llama.cpp/blob/master/CONTRIBUTING.md)
- Self-reported review complexity:
  - [x] Low
  - [ ] Medium
  - [ ] High

---

## 💬 Conversation

👤 **ranilongxi** commented on **2025-10-31** at **03:28:46**

I tried to build this branch with cpu-only and CUDA but it kept failing:

CUDA
`cmake -B build -G Ninja -DCMAKE_BUILD_TYPE=Release -DGGML_LTO=ON -DGGML_NATIVE=ON -DGGML_CUDA=ON -DGGML_CUDA_F16=ON -DGGML_CUDA_FA_ALL_QUANTS=ON -DGGML_CUDA_DMMV_F16=ON -DGGML_CUDA_FORCE_MMQ=ON -DCMAKE_CUDA_ARCHITECTURES=89 -DLLAMA_CURL=ON -DGGML_BLAS=OFF -DGGML_SCHED_MAX_COPIES=1
`

`FAILED: src/CMakeFiles/llama.dir/llama-load-tensors.cpp.o
.../llama-load-tensors.cpp:1006:13: error: conflicting declaration 'int64_t n_embd'
     int64_t n_embd = hparams.n_embd / (hparams.n_deepstack_layers + 1);
             ^~~~~~
.../llama-load-tensors.cpp:227:40: note: previous declaration as 'const int64_t n_embd'
         [[maybe_unused]] const int64_t n_embd = hparams.n_embd; \
... in expansion of macro 'LOADING_PRELUDE'
.../llama-load-tensors.cpp:1045:13: error: conflicting declaration 'int64_t n_embd'
`
CPU only
`cmake -B build -G Ninja -DCMAKE_BUILD_TYPE=Release
ninja -C build -j22`

`FAILED: src/CMakeFiles/llama.dir/llama-load-tensors.cpp.o
.../llama-load-tensors.cpp:1006:13: error: conflicting declaration 'int64_t n_embd'
.../llama-load-tensors.cpp:227:40: note: previous declaration as 'const int64_t n_embd'
... in expansion of macro 'LOADING_PRELUDE'
`

---

👤 **Thireus** commented on **2025-10-31** at **06:29:54**

@ranilongxi, thank you; should be fixed now.

---

👤 **ikawrakow** started a conversation on `src/llama-load-tensors.cpp` on **2025-10-31** at **07:33:19**

I think this needs to have `llama_model_loader::TENSOR_NOT_REQUIRED`. Else, if the output tensor is missing, it will raise an error, so the change does nothing.

---

👤 **ikawrakow** started a conversation on `examples/mtmd/clip.cpp` on **2025-10-31** at **07:37:47**

Can we have `k_w, q_w, v_w`, and then also have `qkv_w`? Or not have any of those? I think, most likely what it needs to happen here is that one first tries to create `qkv_w` and, if that fails, only then one creates `k_w, q_w, v_w` (and they must be there in that case, so there is no `false` in the `get_tensor` argument list).

---

👤 **ikawrakow** started a conversation on `examples/mtmd/clip.cpp` on **2025-10-31** at **07:39:13**

Same comment as above, but now for `qkv_b` and `q_b, k_b, v_b`

---

👤 **ikawrakow** reviewed this pull request 💬 on **2025-10-31** at **07:41:10**

Apart from the comments, LGTM.

But we need to get this tested by someone before merging.

---

👤 **Thireus** commented on **2025-10-31** at **09:20:49**

Thank you @ikawrakow, I'll do my best to resolve your comments. There are still some compilation issues which I'll resolve as well.

---

👤 **Thireus** commented on **2025-10-31** at **10:12:58**

@ikawrakow please let me know if you are happy with how I've addressed your comments:
https://github.com/ikawrakow/ik_llama.cpp/pull/883/commits/e552942129f28fac8a39bfee5897691d74f4c431
and
https://github.com/ikawrakow/ik_llama.cpp/pull/883/commits/6a24dec8b02ad3b107cd0dbab1be7d5f240467e5

---

👤 **ikawrakow** started a conversation on `src/llama-load-tensors.cpp` on **2025-10-31** at **12:19:04**

I missed this one. This will create the `attn_q, attn_k, attn_v` tensors that all depend on `n_embd`. But for a vision model we have to use `_n_embd`, so the `_n_embd` change needs to be also done in the `merge_qkv()` function.

> 👤 **Thireus** replied on **2025-10-31** at **12:59:24**
> 
> Done, since it's not passed to the function I believe it makes more sense then to also have that division in the LOADING_PRELUDE, which avoids carrying a _var around.

---

👤 **Thireus** commented on **2025-11-01** at **07:06:33**

Windows test builds: https://github.com/Thireus/ik_llama.cpp/releases look for the `tr-qwen3-vl-merged` tags.

---

👤 **sayap** commented on **2025-11-01** at **13:42:30**

Made an IQ5_KS quant and f32/f16/bf16 mmproj from the PR branch (using mainline convert_hf_to_gguf.py). Works fine so far. Thanks!

---

👤 **MrHills-2** commented on **2025-11-01** at **17:19:42**

With llama-server and openai API text works, but sending images returns error "Failed to parse messages: Unsupported content part type: \\\"image_url\\\"\".

---

👤 **Thireus** commented on **2025-11-01** at **20:44:49**

> With llama-server and openai API text works, but sending images returns error "Failed to parse messages: Unsupported content part type: \"image_url\"".

It's a separate issue and has been mentioned by @ikawrakow - https://github.com/ikawrakow/ik_llama.cpp/issues/792#issuecomment-3341342894

---

👤 **arichiardi** commented on **2025-11-03** at **16:27:09**

I can test this branch, do you folks have a gguf to try out on a single 3090 by any chance?

---

👤 **MrHills-2** commented on **2025-11-03** at **17:28:25**

> I can test this branch, do you folks have a gguf to try out on a single 3090 by any chance?

https://huggingface.co/mradermacher/Qwen3-VL-30B-A3B-Instruct-i1-GGUF

Any quants should work as you still use og llama.cpp to quantize, no?

---

👤 **arichiardi** commented on **2025-11-03** at **22:42:03**

Ok, I have the `mradermacher/Qwen3-VL-30B-A3B-Instruct-i1-GGUF/Qwen3-VL-30B-A3B-Instruct.i1-IQ4_XS.gguf` up and running.

<img width="1534" height="714" alt="qwen3-VL-running" src="https://github.com/user-attachments/assets/6e92a1ea-acdb-46f9-ace5-ac0d469b08ae" />

---

👤 **ikawrakow** approved this pull request ✅ on **2025-11-04** at **17:20:43**

---

👤 **firecoperana** commented on **2025-11-05** at **04:19:19**

https://github.com/ikawrakow/ik_llama.cpp/pull/901 Support of vision model in llama-server