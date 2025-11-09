## 🔀 [Pull Request #879](https://github.com/ikawrakow/ik_llama.cpp/pull/879) - Disable pipeline parallel for tensor override or allocation failed

| **Author** | `firecoperana` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `fcp/parallel` |
| **Target Branch** | `main` |
| **Created** | 2025-10-30 |
| **Updated** | 2025-10-31 |
| **Merged** | 2025-10-31 |

---

## 📄 Description

This PR attempts to solve abnormal vram issue when people use -DGGML_SCHED_MAX_COPIES=4 with tensor override. This values is linked to pipeline parallel. When GGML_SCHED_MAX_COPIES>1, it enables pipeline parallel in the code. Disable pipeline parallel will make vram return to normal.  
It will also disable pipeline parallel if allocation of compute buffer failed and reallocate with pipeline parallel disabled.

---

## 💬 Conversation

👤 **firecoperana** commented on **2025-10-30** at **01:14:46**

When I was testing with two gpu setup, enabling pipeline parallel leads to an extra 5400MiB vram usage in CUDA0_host, which is not present in mainline. Disabling pipeline parallel will get rid of the 5400MiB vram usage. This size is about 4 times of context size. Also some error in debug when doing inferencing, which is also not present in mainline. 

```
ggml_backend_sched_alloc_splits: failed to allocate graph, reserving (backend_ids_changed = 1)
ggml_backend_sched_alloc_splits: failed to allocate graph, reserving (backend_ids_changed = 1)
ggml_gallocr_needs_realloc: src 0 (inp_out_ids) of node inp_out_ids (view) is not valid
ggml_gallocr_alloc_graph: cannot reallocate multi buffer graph automatically, call reserve
ggml_backend_sched_alloc_splits: failed to allocate graph, reserving (backend_ids_changed = 0)
```


```
llm_load_tensors: ggml ctx size =    0.48 MiB
llm_load_tensors: offloading 27 repeating layers to GPU
llm_load_tensors: offloading non-repeating layers to GPU
llm_load_tensors: offloaded 28/28 layers to GPU
llm_load_tensors:  CUDA_Host buffer size =   106.25 MiB
llm_load_tensors:      CUDA0 buffer size =  7972.11 MiB
llm_load_tensors:      CUDA1 buffer size =   164.07 MiB
..........................................................................................
llama_new_context_with_model: n_ctx         = 5120
llama_new_context_with_model: n_batch       = 4096
llama_new_context_with_model: n_ubatch      = 4
llama_kv_cache_init:      CUDA0 KV buffer size =  1350.00 MiB
llama_new_context_with_model: KV self size  = 1350.00 MiB, K (f16):  810.00 MiB, V (f16):  540.00 MiB
llama_new_context_with_model:  CUDA_Host  output buffer size =     0.78 MiB
llama_new_context_with_model: pipeline parallelism enabled (n_copies=4)
ggml_gallocr_reserve_n: reallocating CUDA0 buffer from size 0.00 MiB to 3.88 MiB
ggml_gallocr_reserve_n: reallocating CUDA1 buffer from size 0.00 MiB to 1.72 MiB
ggml_gallocr_reserve_n: reallocating CUDA_Host buffer from size 0.00 MiB to 5405.72 MiB
llama_new_context_with_model:      CUDA0 compute buffer size =     3.88 MiB
llama_new_context_with_model:      CUDA1 compute buffer size =     1.72 MiB
llama_new_context_with_model:  CUDA_Host compute buffer size =  5405.72 MiB
```

---

👤 **firecoperana** commented on **2025-10-30** at **13:28:31**

The above issue happens when I use DeepSeek-V2-Lite-Chat.IQ4_XS.gguf. Since most people use tensor override for deepseek, it won't surface after this PR is merged. Qwen2.5-7B does not have vram issue, but I get `GGGGG ` in the output, but it should be from other commit.

---

👤 **ikawrakow** commented on **2025-10-30** at **16:17:39**

> Qwen2.5-7B does not have vram issue, but I get `GGGGG `

I just tested with `Qwen2.5-7B-VL` and I don't see any issues. Are you sure this is not due to the change in this PR?

> ggml_backend_sched_alloc_splits: failed to allocate graph, reserving (backend_ids_changed = 1)

The absence of these messages is due to mainline changes in the allocator (`ggml-alloc.c`). They used to be there. Mainline only very recently had problems with VRAM usage exploding due to these changes that got fixed in [this PR](https://github.com/ggml-org/llama.cpp/pull/16679). I did actually make a synced version of `ggml-alloc.c`, but it looked like there is a slight performance drop, while the purpose of trying to avoid reallocating the graph from scratch is to save time. Hence, it is fine to see those messages in debug mode.

> When I was testing with two gpu setup, enabling pipeline parallel leads to an extra 5400MiB vram usage in CUDA0_host

`CUDA_Host` buffers are allocated on the host (so, RAM, not VRAM). As such not quite as dramatic as losing 5.4 GB VRAM. Still, it would be useful to also find the reason for this leak. Perhaps it is somehow related to the mainline leak fixed by the recent PR there mentioned above?

---

👤 **firecoperana** commented on **2025-10-30** at **19:22:00**

All this PR does is enable or disable pipeline parallel. I tested with it disabled, and `gggg` still happened.  
The last good commit for me is https://github.com/ikawrakow/ik_llama.cpp/pull/866, so it's probably caused by https://github.com/ikawrakow/ik_llama.cpp/pull/868. Only happens with Qwen 2.5 7B and 14B models that I use to test, but not other models. 

My cmake command is ```cmake -B build  -DGGML_CUDA=ON -DGGML_SCHED_MAX_COPIES=1 -DGGML_CUDA_FA_ALL_QUANTS=ON -DGGML_IQK_FA_ALL_QUANTS=1 -DGGML_IQK_FLASH_ATTENTION=1 -DGGML_CUDA_USE_GRAPHS=OFF```

Server command:
``` --n-gpu-layers 99 --split-mode layer --tensor-split 99 --main-gpu 0  --ubatch-size 4 --batch-size 32 --threads 14 --no-mmap --no-warmup  --ctx-size 5000 ```

I will do some digging for the ram usage issue.

---

👤 **firecoperana** commented on **2025-10-31** at **03:47:41**

For testing purpose, I synced the ggml-alloc.c from mainline, but still see 5.4G in ram. Then I tried different mla, With `-mla 1` and `-mla 3`, ram usage is 1.23G, but with `-mla 2`, there is only 0.66 MiB in ram. At least there is a way to fix it when needed. Do you want these new commits in this PR?

---

👤 **ikawrakow** commented on **2025-10-31** at **06:33:19**

@firecoperana Thank you for investigating this!

I think, if syncing with mainline's `ggml-alloc.c` does not resolve the extra allocation, it is better to not add it to the PR. Adding it would require way more testing to make sure there are no bad interactions with the other parts of the back-end that are subtly different.

Concerning the gibberish with Qwen2.5: which model are you using? I tested with Qwen2.5-7B vision model that I had lying around, assuming the text portion of it is the same as pure text Qwen2.5.

---

👤 **firecoperana** commented on **2025-10-31** at **12:05:38**

I was using Qwen2.5-7B.i1-Q4_K_S.gguf, but I tested Qwen2.5-VL-7B-Instruct-Q8_0.gguf and it also produce gibberish. I also see this with GLM 4.5 air using CUDA. Maybe I need to play around with my build command.

---

👤 **ikawrakow** approved this pull request ✅ on **2025-10-31** at **12:20:42**

---

👤 **ikawrakow** commented on **2025-10-31** at **12:53:59**

> I also see this with GLM 4.5

GLM-4.5-Air, being so popular, is one of the models I test with. It works fine for me. Both, CPU-only and hybrid GPU/CPU. I cannot do CUDA-only with my setup.

Perhaps pull the latest main branch and make a fresh build? If the failure persist, please post full logs including command line.

---

👤 **firecoperana** commented on **2025-10-31** at **13:54:26**

After I did a fresh build, it works now. Thanks!