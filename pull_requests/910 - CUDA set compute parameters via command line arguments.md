## 🔀 [Pull Request #910](https://github.com/ikawrakow/ik_llama.cpp/pull/910) - CUDA: set compute parameters via command line arguments

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/cuda_params` |
| **Target Branch** | `main` |
| **Created** | 2025-11-06 |
| **Updated** | 2025-11-07 |
| **Merged** | 2025-11-07 |

---

## 📄 Description

This PR adds the ability to control the following CUDA inference parameters via command line arguments:
* **Fusion**. This is currently compile-time enabled or disabled via `-DGGML_CUDA_FUSION`, but I find it highly annoying that I need to recompile each time I want to test the difference between fusion and no fusion. Fusion improves TG speed by quite a bit (especially when fully offloaded), so ideally it should be enabled. But there are issues [#893](https://github.com/ikawrakow/ik_llama.cpp/issues/893) and [#895](https://github.com/ikawrakow/ik_llama.cpp/issues/895) reporting problems, hence it is useful to enable or disable fusion via a command line argument.
*  **GPU offload threshold**. This is currently controlled via another not widely known `cmake` argument (`GGML_CUDA_MIN_BATCH_OFFLOAD`, see [#520](https://github.com/ikawrakow/ik_llama.cpp/issues/520)). But in reality the threshold for which it is better to offload tensors stored in RAM to the GPU for matrix multiplications is dependent no only on the particular hardware (GPU vs CPU speed, speed of PCI-E bus), but also on the model and quantization type. Hence, it is handy to be able to set this threshold on the command line
* **MMQ-ID threshold**. This threshold determines if the `mmq_id` matrix multiplication kernels introduced in [#728](https://github.com/ikawrakow/ik_llama.cpp/issues/728) get used, or if MoE matrix multiplication happen with the pre-728 approach. The reason I have added this is that `mmq_id` leads to a non-negligible increase in perplexity, see [comment](https://github.com/ikawrakow/ik_llama.cpp/pull/728#issuecomment-3494522254) and [comment](https://github.com/ikawrakow/ik_llama.cpp/pull/728#issuecomment-3496574276) in [#728](https://github.com/ikawrakow/ik_llama.cpp/issues/728). 

To set these parameters on the command line after this PR has been merged, one uses a comma separated list of `key=value` pairs like this
```
-cuda | --cuda-params fusion=int_value,offload-batch-size=int_value,mmq-id-size=int_value
```
Not all parameters need to be present, only those that one wants to change from their default value. I.e.
```
-cuda fusion=1
```
or
```
-cuda  offload-batch-size=64,mmq-id-size=0
```

is perfectly fine.

### Fusion

This should not need further explanation. One just enables (value=1) or disables it (value=0). The default is currently disabled because of [#893](https://github.com/ikawrakow/ik_llama.cpp/issues/893), [#895](https://github.com/ikawrakow/ik_llama.cpp/issues/895). Depending on how the story around fusion evolves, I may add a finer-grained control of which fused operations to enable or disable later. 

### GPU offload threshold

This is relevant for hybrid inference. The GPU offload threshold controls above what batch size tensors stored in RAM will get offloaded to the GPU for computations. Let's call it `min_batch_size`. It is set to GGML_CUDA_MIN_BATCH_OFFLOAD, which has a default value of 32 if not specified as `cmake` definition when building. For regular matrix multiplications `min_batch_size` is the threshold. But for MoE tensors, `ik_llama.cpp` uses as threshold
```
min_batch_size * number_of_total_experts / number_of_active_experts
```
So, for instance, for GLM-4.5/5-MoE, which has 128 total and 8 active experts, MoE tensors stored in RAM will only get offloaded for batch sizes `>= 32 * 128 / 8 = 512`. The new parameter allows to control that. As an example, I have run `llama-bench` like this
```
./bin/llama-bench -m Qwen3-30B-A3B-IQ2_XXS.gguf -t 16 -ngl 100 -ub 2048 -n 0 \
     -ot "ffn_(up|gate|down)_exps\.weight=CPU" \
     -p 32,64,128,256,512,1024,2048 -cuda offload-batch-size=0
```
to measure performance as a function of prompt length (i.e., u-batch size) when tensors get always offloaded. Then the same benchmark but with `-cuda offload-batch-size=1000` to measure the performance with tensors never being offloaded. The graph shows the result

<img width="792" height="612" alt="xx1" src="https://github.com/user-attachments/assets/78a84107-65ee-423c-abb6-4534f7afa454" />

We see that for this particular model, GPU offload becomes faster somewhere around u-batch size of 150-160 tokens. Hence, it would be better to change the default via `-cuda offload-batch-size=10` (10 because Qwen3-30B-A3B has 128 total and 8 active experts, so per above equation MoE GPU offload is done for batches greater than `128/8 * 10 = 160`)

### **MMQ-ID threshold**

For MoE matrix multiplications `ik_llama.cpp` uses the `mmq_id` approach added in PR [#728](https://github.com/ikawrakow/ik_llama.cpp/issues/728), if 
```
batch_size <= 32 * total_number_of_experts
```
This magic threshold has been determined experimentally. For larger batch sizes the original `ik_llama.cpp` MoE implementation tends to be faster.  One can now verify by running, e.g.
```
./bin/llama-bench -m $model -n 0 -t 1 -ngl 100 -b 8192 -ub 8192 -p 256,512,1024,2048,4096,8192 -cuda  mmq-id-size=0
```
to have the `mmq_id` approach be never used, and then the same command but using `-cuda  mmq-id-size=1000` to always  use `mmq_id`.

But apart from speed, @Nexesenex has expressed [concern](https://github.com/ikawrakow/ik_llama.cpp/pull/728#issuecomment-3494522254) that `mmq_id` increases perplexity. If you are worried that this is significant, you can now disable it via `-cuda  mmq-id-size=0`. It is worth noting that mainline `llama.cpp` suffers from the exact same PPL increase (see https://github.com/ikawrakow/ik_llama.cpp/pull/728#issuecomment-3496574276), and there is no way to disable it.

---

## 💬 Conversation

👤 **Nexesenex** commented on **2025-11-06** at **19:40:14**

Thank you very much, IK, those params will help very much.
Compiling and testing.

Edit: @ikawrakow , it works.

Q:\LLAMA_IK_SERVER_TEST>llama-perplexity -m X:\text-generation-webui\models\zai-org_GLM-4.5-Air-bf16-iMat-IQ4_XS_L-00001-of-00005.gguf -f wiki.test.raw --parallel 1 -ngl 150 -b 512 -mg 0 -ts 18,18,12 --no-mmap -no-fmoe -no-fug -no-mmad -cuda fusion=1,offload-batch-size=1,mmq-id-size=0 -c 512 --override-kv glm4moe.expert_used_count=int:7

-> Final estimate: PPL = 4.6508 +/- 0.02854 (Like before PR 728). Nice!

From this first command line, with modifs:

With FUG activated : Final estimate: PPL = 4.6508 +/- 0.02854

With FMOE activated : Final estimate: PPL = 4.6511 +/- 0.02854

With FUG and FMOE activated : Final estimate: PPL = 4.6511 +/- 0.02854

With FUG, FMOE and MMAD activated : Final estimate: PPL over 565 chunks for n_ctx=512 = 4.6513 +/- 0.02854 (and +/- 0.003 - 0.01 pts PPL of variance over individual scores during the PPL test before it realigns at the end).

With Rope cache (w/ or w/o MMQ_ID) : Perplexity 512 jumps by 150% approximatively. That's severe. ^^

That practice to allow to enable / disable CUDA improvements by CLI is a very good one, passed the hassle of coding it, and makes the testing very easy!