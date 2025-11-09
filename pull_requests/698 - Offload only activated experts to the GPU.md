## 🔀 [Pull Request #698](https://github.com/ikawrakow/ik_llama.cpp/pull/698) - Offload only activated experts to the GPU 

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/sched_copy_experts` |
| **Target Branch** | `main` |
| **Created** | 2025-08-16 |
| **Updated** | 2025-09-04 |
| **Merged** | 2025-09-04 |

---

## 📄 Description

This PR is based on [PR 15346](https://github.com/ggml-org/llama.cpp/pull/15346) in mainline `llama.cpp`. Due to the divergence of the code bases cherry-picking did not work. I also needed to implement the logic for the fused `ffn_up+ffn_gate` op, which is not present in mainline, so it isn't just a copy/paste.

For hybrid CPU/GPU inference for MoE models, when experts are stored in RAM only the activated experts are copied to the GPU. The change only affects prompt processing (experts stored in RAM are never copied to the GPU for TG), and only activates if the batch size is large enough to trigger GPU offload (which, unlike mainline where experts are copied for batch sizes >= 32, is model dependent and given by `32 * total_experts / active_experts`). The idea is that if a significant fraction of the experts are rarely activated, this could result in a non-negligible performance gains. However, for many models and a batch size large enough to trigger GPU offload, basically all experts are active, so there is no performance gain (and one may observe even a slight performance degradation as the added steps to synchronize with the GPU(s) and copy over the expert IDs do not pay off to reduce the amount of data being copied).

Here are some performance comparison examples. Most benchmarks are run on a Ryzen-7950X CPU + RTX-4080 GPU using `-ot exps=CPU` (so all experts are on the CPU). The GPT-OSS-120B case is run on a Ryzen-5975WX + RTX-4080 system (the  Ryzen-7950X rig does not have enough RAM for GPT-OSS-120B). In all cases flash attention is on and `fmoe = 1`. Only u-batch sizes large enough to trigger GPU offload are included in the tables.

### GPT-OSS-20B, MXFP4

| model              | n_batch | n_ubatch |n    test |    t/s (main)    |   t/s (PR)       |  Speedup |
| ------------------ | ------: | -------: | -------: | ---------------: | ---------------: | --------: |
| gpt-oss 20B MXFP4  |    4096 |      256 |   pp4096 |    469.59 ± 0.90 |    611.18 ± 4.24 |  1.302   |
| gpt-oss 20B MXFP4  |    4096 |      512 |   pp4096 |    856.50 ± 1.67 |   1036.59 ± 1.51 |  1.210   |
| gpt-oss 20B MXFP4  |    4096 |     1024 |   pp4096 |   1454.42 ± 5.67 |   1673.11 ± 4.94 |  1.150   |
| gpt-oss 20B MXFP4  |    4096 |     2048 |   pp4096 |   2236.24 ± 7.75 |  2422.16 ± 23.31 |  1.083   |
| gpt-oss 20B MXFP4  |    4096 |     4096 |   pp4096 |   2943.42 ± 5.39 |  3054.72 ± 19.96 |  1.038   |

### GPT-OSS-120B, MXFP4

| model               | n_batch | n_ubatch |      test |     t/s (main)   |     t/s (PR)     |  Speedup |
| ------------------- | ------: | -------: | --------: | ---------------: | ---------------: | -------: |
| gpt-oss 120B MXFP4  |    4096 |     1024 |    pp4096 |    244.54 ± 0.14 |    411.58 ± 2.42 | 1.683    |
| gpt-oss 120B MXFP4  |    4096 |     2048 |    pp4096 |    447.75 ± 1.01 |    667.45 ± 4.99 | 1.491    |
| gpt-oss 120B MXFP4  |    4096 |     4096 |    pp4096 |    761.12 ± 0.57 |   1015.11 ± 1.22 | 1.334    |

### DeepSeek-Lite, Q4_0, mla = 3

| model         | n_batch | n_ubatch |      test |     t/s (main)   |     t/s (PR)     |  Speedup |
| ------------- | ------: | -------: | --------: | ---------------: | ---------------: | -------: |
| deepseek2 16B |    4096 |      512 |    pp4096 |   1057.15 ± 4.55 |   1053.43 ± 3.96 | 0.996    |
| deepseek2 16B |    4096 |     1024 |    pp4096 |  1920.41 ± 33.44 |  1923.08 ± 32.75 | 1.001    |
| deepseek2 16B |    4096 |     2048 |    pp4096 |   3218.65 ± 8.49 |   3209.62 ± 7.41 | 0.997    |
| deepseek2 16B |    4096 |     4096 |    pp4096 |   4865.61 ± 9.75 |   4859.61 ± 6.14 | 0.999    |


 ## Update

So, this seems to be really only useful for GPT-OSS. There are also issues with multi-GPU setups. Hence, I have now added a command line parameter to turn this feature on:
```
--offload-only-active-experts  or  -ooae
```

Note that I have not bothered to add these flags to `llama-bench`, so one can only test with `llama-sweep-bench` (or `llama-cli`, `llama-server`, etc.).

---

## 💬 Conversation

👤 **Ph0rk0z** commented on **2025-08-16** at **14:39:48**

For qwen 235b:

```

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
/home/supermicro/ai/ik_llama.cpp/ggml/src/ggml-backend.cpp:203: GGML_ASSERT(offset + size <= ggml_nbytes(tensor) && "tensor write out of bounds") failed
Could not attach to process.  If your uid matches the uid of the target
process, check the setting of /proc/sys/kernel/yama/ptrace_scope, or try
again as the root user.  For more details, see /etc/sysctl.d/10-ptrace.conf
ptrace: Operation not permitted.
No stack.
The program is not being run.
Aborted (core dumped)
```


```
        CUDA_VISIBLE_DEVICES=0,1,2,3 numactl --interleave=all ./bin/llama-sweep-bench \
    -m Qwen3-235B-A22B-Instruct-2507-IQ4_KS/Qwen3-235B-A22B-Instruct-pure-IQ4_KS-00001-of-00003.gguf \
    -t 48 \
    -c 32768 \
    --numa distribute \
    -ngl 95 \
    -ctk q8_0 \
    -ctv q8_0 \
    -fa \
    -rtr \
    -fmoe \
    -ub 1024 \
    -ot "blk\.(0|1|2|3|4|5|6|7|8|9|10|11|12|13|14|15)\.ffn_.*.=CUDA0" \
    -ot "blk\.(16|17|18|19|20|21|22|23|24|25|26|27|28|29|30|31|32)\.ffn_.*.=CUDA1" \
    -ot "blk\.(33|34|35|36|37|38|39|40|41|42|43|44|45|46|47|48|49)\.ffn_.*.=CUDA2" \
    -ot "blk\.(50|51|52|53|54|55|56|57|58|59|60|61|62|63|64|65)\.ffn_.*.=CUDA3" \
    -ot "\.ffn_.*_exps.=CPU"
```


I copied the PR into a branch and make clean/ccmake ./make

---

👤 **ikawrakow** commented on **2025-08-16** at **14:51:36**

@Ph0rk0z

Thanks for the bug report. If you are willing to test, can you do these two things:
* Pull the latest and rerun as above. When it fails, there will be an additional log message before the assert that I need to see
* Try running on a single GPU and let me know if that works.

---

👤 **Ph0rk0z** commented on **2025-08-16** at **15:49:36**

Multi gpu crashes like this:


```
ggml_backend_tensor_set_async(CUDA0#blk.66.ffn_up_exps.weight#0): offset = 18442617028498948096, size = 3348992, nbytes = 428605440
/home/supermicro/ai/ik_llama.cpp/ggml/src/ggml-backend.cpp:204: GGML_ASSERT(offset + size <= ggml_nbytes(tensor) && "tensor write out of bounds") failed
Could not attach to process.  If your uid matches the uid of the target
process, check the setting of /proc/sys/kernel/yama/ptrace_scope, or try
again as the root user.  For more details, see /etc/sysctl.d/10-ptrace.conf
ptrace: Operation not permitted.
No stack.
The program is not being 
```

Single GPU runs:

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  1024 |    256 |      0 |   15.013 |    68.21 |   21.552 |    11.88 |
|  1024 |    256 |   1024 |   14.166 |    72.29 |   21.880 |    11.70 |

---

👤 **ikawrakow** commented on **2025-08-16** at **15:58:17**

Thanks for testing!

So, the issue is with multi-GPU, which I cannot test myself.

Btw, I did test Qwen3-30B-A3B and observe zero effect.

Just in the process of testing with `GLM-4.5-Air-UD-IQ2_XXS.gguf` and it seems there are no gains there either.

So, it looks like having rarely activated experts is specific to GPT-OSS. Although I can also remember people having trouble to get imatrix data for the large DeepSeek models, which indicates that at least some experts are rarely activated, so possibly one may gain something there.

---

👤 **Ph0rk0z** commented on **2025-08-16** at **16:18:04**

Thought some experts in qwen didn't get used as well: https://huggingface.co/kalomaze/Qwen3-16B-A3B

Maybe it just doesn't help here. Not a big fan of OSS. Tried it on huggingface chat and decided to skip it when it came out.

---

👤 **saood06** commented on **2025-08-17** at **01:17:56**

>Thought some experts in qwen didn't get used as well: https://huggingface.co/kalomaze/Qwen3-16B-A3B

From that repo discussion [here](https://huggingface.co/kalomaze/Qwen3-16B-A3B/discussions/14) it lost basically all Chinese capability.

>Although I can also remember people having trouble to get imatrix data for the large DeepSeek models, which indicates that at least some experts are rarely activated, so possibly one may gain something there.

It was a single expert on a single layer I think, it was Arctic where less diverse datasets got less experts but even with relatively diverse ones there was still a few missing I think.

The performance gain was still decreasing with increasing n_ubatch for `gpt-oss-120b` so I'm assuming at sufficiently high batch sizes it would disappear (I don't think OpenAI left unused experts), it would just take a lot more for the larger model as they share the same number of active experts, the larger one just has a much bigger pool of experts.

---

👤 **Ph0rk0z** commented on **2025-08-17** at **11:50:45**

>From that repo discussion [here](https://huggingface.co/kalomaze/Qwen3-16B-A3B/discussions/14) it lost basically all Chinese capability.

Yep, but if you're not using chinese it should avoid those experts and never activate them. Seems it didn't work out that way in this PR tho.