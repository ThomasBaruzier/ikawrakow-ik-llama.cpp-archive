## 🔀 [Pull Request #861](https://github.com/ikawrakow/ik_llama.cpp/pull/861) - CUDA: fuse ffn_up*unary_op(ffn_gate) for MMVQ 

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Source Branch** | `ik/mmvq_args` |
| **Target Branch** | `main` |
| **Created** | 2025-10-24 |
| **Updated** | 2025-10-26 |

---

## 📄 Description

This PR fuses `ffn_up*unary_op(ffn_gate)` into a single kernel for token generation (batch size = 1). We get in the range of 3% speedup for TG for MoE models, and ~1% for dense models.

Note that this is already done on the CPU back-end, so no changes there.

The downside of the PR is that now the compilation of `mmvq.cu` and `iqk_mmvq.cu` takes ~2.5 times longer.

---

## 💬 Conversation

👤 **Ph0rk0z** commented on **2025-10-24** at **18:32:10**

Gentlemen.. I regret to inform you that:
```

llama_kv_cache_init:      CUDA0 KV buffer size =  1632.01 MiB
llama_kv_cache_init:      CUDA1 KV buffer size =  1564.01 MiB
llama_kv_cache_init:      CUDA2 KV buffer size =  1564.01 MiB
llama_kv_cache_init:      CUDA3 KV buffer size =  1496.01 MiB
llama_new_context_with_model: KV self size  = 6256.00 MiB, K (q8_0): 3128.00 MiB, V (q8_0): 3128.00 MiB
llama_new_context_with_model:  CUDA_Host  output buffer size =     0.58 MiB
llama_new_context_with_model: pipeline parallelism enabled (n_copies=1)
llama_new_context_with_model:      CUDA0 compute buffer size =   360.00 MiB
llama_new_context_with_model:      CUDA1 compute buffer size =   360.00 MiB
llama_new_context_with_model:      CUDA2 compute buffer size =   284.50 MiB
llama_new_context_with_model:      CUDA3 compute buffer size =   612.00 MiB
llama_new_context_with_model:  CUDA_Host compute buffer size =   292.48 MiB
llama_new_context_with_model: graph nodes  = 4094
llama_new_context_with_model: graph splits = 158
XXXXXXXXXXXXXXXXXXXXX Setting only active experts offload

main: n_kv_max = 32768, n_batch = 2048, n_ubatch = 1024, flash_attn = 1, n_gpu_layers = 94, n_threads = 48, n_threads_batch = 48

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
/home/supermicro/ai/ik_llama.cpp/ggml/src/ggml-cuda/mmvq.cu:741: GGML_ASSERT(!ids || ids->ne[0] == dst->ne[2]) failed
```


I notice that graph splits have halved and graph nodes have doubled.

---

👤 **ikawrakow** commented on **2025-10-25** at **06:06:44**

@Ph0rk0z Can you rerun with the latest commit? I need the added diagnostic message because I'm unable to trigger this assert no matter how hard I try, so I don't really understand why you are getting it. Thanks.

---

👤 **Ph0rk0z** commented on **2025-10-25** at **11:25:57**

Here you go:

```
llama_new_context_with_model: graph nodes  = 4094
llama_new_context_with_model: graph splits = 158
XXXXXXXXXXXXXXXXXXXXX Setting only active experts offload

main: n_kv_max = 32768, n_batch = 2048, n_ubatch = 1024, flash_attn = 1, n_gpu_layers = 94, n_threads = 48, n_threads_batch = 48

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
ggml_cuda_op_fused_mul_mat_vec_q_id(blk.49.ffn_up_exps.weight->ffn_moe_gate_par-49): unexpected situation
  src0 = 5120 x 1536 x 120 x 1
  src1 = 5120 x 1 x 1 x 1
   ids = 120 x 1 x 1 x 1
   dst = 1536 x 120 x 1 x 1
/home/supermicro/ai/ik_llama.cpp/ggml/src/ggml-cuda/mmvq.cu:747: Fatal error
```

---

👤 **ikawrakow** commented on **2025-10-25** at **12:22:09**

Thanks! 

Can you try [#863](https://github.com/ikawrakow/ik_llama.cpp/issues/863) ?

---

👤 **Ph0rk0z** commented on **2025-10-25** at **13:44:32**

You mean https://github.com/ikawrakow/ik_llama.cpp/pull/864 ? That one fails exact same way.

Removing the FA parameters just made me edit my command lines.

---

👤 **ikawrakow** commented on **2025-10-25** at **14:29:48**

> You mean https://github.com/ikawrakow/ik_llama.cpp/pull/864 ? That one fails exact same way.

Yes, I mean [#864](https://github.com/ikawrakow/ik_llama.cpp/issues/864). It is not possible for it to fail with the same assert because the assert is no longer there. You need to properly rebuild.

---

👤 **Ph0rk0z** commented on **2025-10-25** at **15:02:44**

I built it when https://github.com/ikawrakow/ik_llama.cpp/pull/864/commits/5bd9cd349051c7dd741e91edcc93f258e62080de was the latest commit.

I will try with your latest and see where it moves to and if it does.

---

👤 **Ph0rk0z** commented on **2025-10-25** at **15:09:18**

Here's what happened next:


```
llama_new_context_with_model:      CUDA0 compute buffer size =   360.00 MiB
llama_new_context_with_model:      CUDA1 compute buffer size =   360.00 MiB
llama_new_context_with_model:      CUDA2 compute buffer size =   284.50 MiB
llama_new_context_with_model:      CUDA3 compute buffer size =   612.00 MiB
llama_new_context_with_model:  CUDA_Host compute buffer size =   292.48 MiB
llama_new_context_with_model: graph nodes  = 4094
llama_new_context_with_model: graph splits = 158
XXXXXXXXXXXXXXXXXXXXX Setting only active experts offload

main: n_kv_max = 32768, n_batch = 2048, n_ubatch = 1024, flash_attn = 1, n_gpu_layers = 94, n_threads = 48, n_threads_batch = 48

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
CUDA error: an illegal memory access was encountered
  current device: 3, in function ggml_backend_cuda_synchronize at /home/supermicro/ai/ik_llama.cpp/ggml/src/ggml-cuda.cu:3450
  cudaStreamSynchronize(cuda_ctx->stream())
/home/supermicro/ai/ik_llama.cpp/ggml/src/ggml-cuda.cu:122: CUDA error
```

---

👤 **ikawrakow** commented on **2025-10-25** at **15:14:37**

OK, then, I need your command line. Especially given your other comment about having to modify scripts because defaults for `fmoe/fa` changed.  

Just to make sure, this is with [#864](https://github.com/ikawrakow/ik_llama.cpp/issues/864)?

---

👤 **ikawrakow** commented on **2025-10-25** at **15:17:59**

Also, in case you are on Linux, running with
```
cuda-gdb --args your_command_goes_here
```
and posting the debugger output when it crashes would be very helpful. Thanks!

---

👤 **Ph0rk0z** commented on **2025-10-25** at **16:03:12**

Commandline is pretty simple.


```
            CUDA_VISIBLE_DEVICES=0,1,2,3 numactl --interleave=all ./bin/llama-sweep-bench \
    -m /models/GLM-4.6-REAP-218B-A32B-MXFP4_MOE/GLM-4.6-REAP-218B-A32B-MXFP4_MOE-00001-of-00004.gguf \
    -t 48 \
    -c 32768 \
    --numa distribute \
    -ngl 94 \
    -ctk q8_0 \
    -ctv q8_0 \
    -rtr \
    -ub 1024 \
    -ot "blk\.(0|1|2|3|4|5|6|7|8|9|10|11|12|13|14)\.ffn_.*_exps.=CUDA0" \
    -ot "blk\.(15|16|17|18|19|20|21|22|23|24|25|26|27)\.ffn_.*_exps.=CUDA1" \
    -ot "blk\.(28|29|30|31|32|33|34|35|36|37|38|39|40)\.ffn_.*_exps.=CUDA2" \
    -ot "blk\.(41|42|43|44|45|46|47|48|49|50|51|52|53)\.ffn_.*_exps.=CUDA3" \
    -ot "\.ffn_.*_exps.=CPU"
```


```
CUDA_VISIBLE_DEVICES=0,1,2,3 numactl --interleave=all ./bin/llama-sweep-bench \
    -m /GLM-4.6-REAP-266B-A32B-Q4_K/GLM-4.6-REAP-266B-A32B-Q4_K-00001-of-00004.gguf \
    -t 48 \
    -c 32768 \
    --numa distribute \
    -ngl 94 \
    -ctk q8_0 \
    -ctv q8_0 \
    -rtr \
    -ub 1024 \
    -ot "blk\.(0|1|2|3|4|5|6|7|8|9|10|11|12|13)\.ffn_.*_exps.=CUDA0" \
    -ot "blk\.(14|15|16|17|18|19|20|21|22|23|24|25)\.ffn_.*_exps.=CUDA1" \
    -ot "blk\.(26|27|28|29|30|31|32|33|34|35|36|37)\.ffn_.*_exps.=CUDA2" \
    -ot "blk\.(38|39|40|41|42|43|44|45|46|47|48)\.ffn_.*_exps.=CUDA3" \
    -ot "blk\.(49)\.ffn_(up|gate)_exps\.weight=CUDA3" \
    -ot "\.ffn_.*_exps.=CPU"
```

I removed -fa and -fmoe because those no longer exist after the main commit.

Will have to try gdb and see what it shows.

---

👤 **ikawrakow** commented on **2025-10-25** at **16:08:10**

> Will have to try gdb and see what it shows.

`cuda-gdb`, not `gdb`. `cuda-gdb` will tells in which kernel we get the illegal memory access. `gdb` will tell us nothing.

---

👤 **Ph0rk0z** commented on **2025-10-25** at **17:45:00**

So I figured out the cause of the bug because I was switching models.

`    -ot "blk\.(49)\.ffn_(up|gate)_exps\.weight=CUDA3" \`


You fuse ops and this causes the up/gate to be on GPU but then down is on CPU. Makes for an illegal memory access and explains why cuda3 was the culprit. I can't tell what happened speed-wise because my disk cache is full with like 300gb so speeds at their worst.

Do the same on one of your models and I bet it's now reproducible.

---

👤 **ikawrakow** commented on **2025-10-26** at **06:42:56**

Thanks!

The issue should be fixed on the latest [#864](https://github.com/ikawrakow/ik_llama.cpp/issues/864)

---

👤 **ikawrakow** commented on **2025-10-26** at **17:02:33**

Closing in favor of [#864](https://github.com/ikawrakow/ik_llama.cpp/issues/864)