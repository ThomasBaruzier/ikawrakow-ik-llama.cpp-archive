## 🔀 [Pull Request #829](https://github.com/ikawrakow/ik_llama.cpp/pull/829) - gpt-oss: duplicate experts biases when necessary

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/dup_experts_bias` |
| **Target Branch** | `main` |
| **Created** | 2025-10-13 |
| **Updated** | 2025-10-28 |
| **Merged** | 2025-10-14 |

---

## 📄 Description

While experimenting with hybrid GPU/CPU inference with GPT-OSS-120B on RTX-4080 in a Ryzen-5975WX rig, I noticed a significant performance difference depending on which tensors are overridden to the CPU:

Using
```
-ngl 100 -ot exps=CPU or -n-cpu-moe 100
```
which results in all `ffn_(up|gate|down)_exps.weight` and `ffn_(up|gate|down)_exps.bias` tensors to be stored on the CPU, we get these `llama-sweep-bench` results

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  4096 |   1024 |      0 |    3.903 |  1049.37 |   41.022 |    24.96 |
|  4096 |   1024 |   4096 |    4.012 |  1020.91 |   41.252 |    24.82 |
|  4096 |   1024 |   8192 |    3.994 |  1025.61 |   41.569 |    24.63 |
|  4096 |   1024 |  12288 |    4.042 |  1013.45 |   41.877 |    24.45 |
|  4096 |   1024 |  16384 |    4.112 |   996.11 |   42.164 |    24.29 |
|  4096 |   1024 |  20480 |    4.173 |   981.62 |   42.464 |    24.11 |
|  4096 |   1024 |  24576 |    4.208 |   973.43 |   42.751 |    23.95 |
|  4096 |   1024 |  28672 |    4.263 |   960.94 |   43.046 |    23.79 |

Using instead
```
-ngl 100 -ot "exps_(up|gate|down)_exps\.weight"
```
so that the `ffn_(up|gate|down)_exps.bias` tensors are stored on the GPU, we get these `llama-sweep-bench` results

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  4096 |   1024 |      0 |    3.091 |  1325.28 |   45.181 |    22.66 |
|  4096 |   1024 |   4096 |    3.180 |  1288.24 |   45.632 |    22.44 |
|  4096 |   1024 |   8192 |    3.204 |  1278.46 |   45.846 |    22.34 |
|  4096 |   1024 |  12288 |    3.252 |  1259.72 |   46.154 |    22.19 |
|  4096 |   1024 |  16384 |    3.320 |  1233.67 |   46.413 |    22.06 |
|  4096 |   1024 |  20480 |    3.384 |  1210.36 |   46.704 |    21.93 |
|  4096 |   1024 |  24576 |    3.423 |  1196.59 |   46.862 |    21.85 |
|  4096 |   1024 |  28672 |    3.468 |  1180.93 |   47.159 |    21.71 |

I.e., in the latter case we get 20-25% better PP performance but ~10% lower TG generation speed.

It is understandable that there will be some performance difference:
* When the experts are stored on the CPU, for TG the `ffn` portions of the compute graph are evaluated on the CPU, which requires copying the `ffn_(up|gate|down)_exps.bias` tensors from the GPU, so we get a lower TG performance in the latter case
*  For PP it is the other way around. When `ffn_(up|gate|down)_exps.bias` are not stored on the GPU, they need to be copied, so we get a lower PP performance in the former case.

But the PP performance difference is surprisingly large, considering that the `ffn_(up|gate|down)_exps.bias`  tensors are quite small (about 152 MiB for GPT-OSS-120B). On a 30 GB/s PCE-E it takes in the range of 5 ms to copy these tensors. The above results are for batch and u-batch size of 4096 tokens. At ~1300 tokens/second computing one u-batch takes 3+ seconds, so the 5 ms added for copying the experts biases are completely negligible. The difference in TG performance is more in line with the expectation: at 25 t/s a token generation requires ~40 ms, so the added 5 ms for copying the biases from the GPU to the CPU should lead to a ~10% drop in performance.

As I wasn't able to figure out why PP performance is impacted so much, this PR duplicates the  `ffn_(up|gate|down)_exps.bias` tensors when the corresponding ``ffn_(up|gate|down)_exps.weight` tensor is on a different device. When building the compute graph, and the duplicated tensors are present, they are used when batch size is less than 32 tokens. With this change we get the following `llama-sweep-bench` results

```
./bin/llama-sweep-bench -m $model -t 32 -ngl 100 -c 32768 -b 4096 -ub 4096 -ot "ffn_(up|gate|down)_exps\.weight=CPU" -fa -fmoe -ooae
```

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  4096 |   1024 |      0 |    3.172 |  1291.49 |   40.803 |    25.10 |
|  4096 |   1024 |   4096 |    3.264 |  1254.77 |   41.165 |    24.88 |
|  4096 |   1024 |   8192 |    3.283 |  1247.61 |   41.389 |    24.74 |
|  4096 |   1024 |  12288 |    3.332 |  1229.26 |   41.692 |    24.56 |
|  4096 |   1024 |  16384 |    3.399 |  1205.15 |   41.952 |    24.41 |
|  4096 |   1024 |  20480 |    3.467 |  1181.36 |   42.262 |    24.23 |
|  4096 |   1024 |  24576 |    3.495 |  1171.92 |   42.509 |    24.09 |
|  4096 |   1024 |  28672 |    3.548 |  1154.57 |   42.789 |    23.93 |

I.e., we now get the better PP performance and the better TG performance.
 
If you are using GPT-OSS-120B with `ik_llama.cpp` and hybrid inference, `-ot "ffn_(up|gate|down)_exps\.weight=CPU"` is the recommended tensor override option after this PR has been merged.

**Note**: the PR only affects the GPT-OSS models as other common MoE models to not use experts biases.

---

## 💬 Conversation

👤 **ubergarm** commented on **2025-10-16** at **16:00:20**

I just did a 3 way comparison ik_llama.cpp with this PR, and mainline and a mainline PR in the linked PR above.

Just for comparison, it is a *huge* benefit to PP here on ik to use `-ooae` and also helps some to use `-ot "ffn_(up|gate|down)_exps\.weight=CPU"` instead of the usual `-ot exps=CPU`. Just these two changes give over +50% PP from ~1296 tok/sec PP up to ~2027 tok/sec!

Amusingly this seems to give better PP on a <$2500 gaming rig with old 3090TI FE than the new nvidia dgx spark: https://github.com/ggml-org/llama.cpp/discussions/16578

---

👤 **ikawrakow** commented on **2025-10-16** at **16:20:56**

Btw, is the linked post the reason for the recent `llama.cpp` star surge? I didn't see anything noteworthy being done there, so was wondering why daily `llama.cpp` stars suddenly tripled. Or is there something else going around social networks?

I was reading yesterday about the dgx spark, and my impression was that there wasn't too much excitement about it. It cannot run large frontier models, is limited to models such as gpt-oss-120B, glm4.5-air, and such. But for those one can build a system with comparable performance for quite a bit less than the $4k the spark costs.

Or am I wrong and should consider getting one?

---

👤 **ubergarm** commented on **2025-10-16** at **17:07:47**

> Btw, is the linked post the reason for the recent llama.cpp star surge? I didn't see anything noteworthy being done there, so was wondering why daily llama.cpp stars suddenly tripled. Or is there something else going around social networks?

honestly I'm not sure and wasn't aware of the uptick in stars. there has been some buzz with reviews and benchmarks coming out out YT for the nvidia dgx spark. while I don't twitter, I did hear about this social media post suggesting llama.cpp is better than ollama (duh...):

> Important info. The issue in that benchmark seems to be ollama. Native llama.cpp works much better.
> https://twitter-thread.com/t/1978372050239516989

Yeah folks are asking the same questions on reddit and discords I'm in now:

> Or am I wrong and should consider getting one?

tl;dr; personally i'd pass on it for now...

My impression is that an AMD Strix Halo machine with 128GB lpddr5 will give similar memory bandwidth APU for much cheaper. The main advantages of dgx spark imo are that it offers CUDA blackwell sm_121 arch, and in addition to usual 10GbE ports, `two QSFP ports driven by NVIDIA ConnectX-7 NIC capable of up to 200 Gbps` just in case you want to spend $8k and network two of them together haha...

Wendell has created some "unholy" combinations using a Framework Desktop AI Max+ shown running an ik_llama.cpp ~1.75BPW DeepSeek 671B quant with a CUDA GPU attached via USB4/Thunderbolt/Occulink enclosure for hybrid CPU+GPU inference: https://youtu.be/L-xgMQ-7lW0?t=1235 . Though someone on u/LocalLLaMA reddit sub tried it and seems like ik_llama.cpp is bottle-necking on the slower PCIe bandwidth for PP: https://www.reddit.com/r/LocalLLaMA/comments/1o7ewc5/fast_pcie_speed_is_needed_for_good_pp/

The idea there is when accessing the lpddr5 via the CPU path it gives about half the bandwidth, but still a good zen5 CPU with 128GB RAM @~120GB/s plus a discrete CUDA GPU is possibly compelling in the smallish form factor.

Personally if I were starting from scratch today to build a biggish MoE inference rig, I'd probably spend $4k USD on the best am5 gaming rig similar to [Wendell's build here](https://www.youtube.com/watch?v=pA-R1FabTDY):

* $2k MSRP 5090 GPU
* $1k on 4x64GB DDR5@6000MT/s
* $1k+ Zen5 CPU, PSU, Case, Storage, etc...

There are some interesting looking RISC-V PCIe NPU "ai accelerator" cards becoming available too, but I'm still learning about those and hope to get access for benchmarking at least one soon.

---

👤 **mpetruc** commented on **2025-10-27** at **14:46:02**

@ubergarm i was intrigued by you comment " huge benefit to PP here on ik to use -ooae". gpt-oss-120b is my main model these days, so being able to get faster then my current ~20t/s pp would be amazing, so i immediately went to test it.
```
./llama-server --version
version: 3928 (16f30fcf)
built with cc (Ubuntu 11.4.0-1ubuntu1~22.04.2) 11.4.0 for x86_64-linux-gnu
```

running llama-server like so:
```
./llama-server -m ~/models/gpt-oss-120b-F16.gguf  --ctx-size 16384 -ngl 100 --n-cpu-moe 24 -ub 4096 -b 4096 --host 0.0.0.0 --port 5050  --jinja -amb 1024 --threads 8 --no-mmap -ooa
```

i'm getting an error:
```
error: unknown argument: -ooae
```
Is there a specific ik_llama.cpp version implementing this flag? How should i test it? 
thanks a lot

---

👤 **ikawrakow** commented on **2025-10-27** at **14:49:44**

> Is there a specific ik_llama.cpp version implementing this flag? How should i test it?

I changed `ooae` to be ON by default in [#863](https://github.com/ikawrakow/ik_llama.cpp/issues/863). Now this is turned on by default and you can turn it off with `-no-ooae`.

---

👤 **mpetruc** commented on **2025-10-27** at **19:49:18**

thank you.
i've been playing all morning with different flags and settings but can not get better PP and TG performance with ik_llama.cpp vs. llama.cpp with gpt-odd-120b. With a 4090, 128 DDR5 (at 4Ghz, unfortunately) and I9-13900k with 8 cores (HT disabled) , the best i'm able to get with ik_lcpp is 
```
./llama-server -m ~/models_test/gguf/gpt-oss-120b-F16.gguf  --ctx-size 32768 -ngl 999 -ub 4096 -b 4096 --host 0.0.0.0 --port 5050 --jinja --threads 8 --n-cpu-moe 25

INFO [           print_timings] prompt eval time     =    3116.96 ms /   310 tokens (   10.05 ms per token,    99.46 tokens per second) | tid="129108960858112" timestamp=1761593445 id_slot=0 id_task=0 t_prompt_processing=3116.962 n_prompt_tokens_processed=310 t_token=10.054716129032258 n_tokens_second=99.45581627238317
INFO [           print_timings] generation eval time =  113517.67 ms /  1841 runs   (   61.66 ms per token,    16.22 tokens per second) | tid="129108960858112" timestamp=1761593445 id_slot=0 id_task=0 t_token_generation=113517.671 n_decoded=1841 t_token=61.660875067897884 n_tokens_second=16.217739350906875
```

with lcpp i get
```
/llama-server -m ~/models_test/gguf/gpt-oss-120b-F16.gguf  --ctx-size 32768 -ngl 999 -ub 4096 -b 4096 --host 0.0.0.0 --port 5050 --jinja --threads 8 --n-cpu-moe 25 -fa on

prompt eval time =    2653.73 ms /   310 tokens (    8.56 ms per token,   116.82 tokens per second)
       eval time =   99151.71 ms /  1761 tokens (   56.30 ms per token,    17.76 tokens per second)
      total time =  101805.44 ms /  2071 tokens
 ```

Using the built-in web client, same system prompt, same sampler settings.

Are there any other settings i should use? Or maybe a specific/optimized quant?
Thanks a lot.

---

👤 **ikawrakow** commented on **2025-10-27** at **20:40:07**

What does the F16 in the model name mean? Which tensors are F16?

---

👤 **mpetruc** commented on **2025-10-27** at **23:57:28**

it's Unsloth's unquantized gguf version of gpt-oss-120b: https://huggingface.co/unsloth/gpt-oss-120b-GGUF/blob/main/gpt-oss-120b-F16.gguf. 
But to answer your question - i don't know for sure why they felt it necessary to add the F16. My understating is that it's just the original model from OpenAI but in gguf format. And with a "fixed" chat template.

---

👤 **ikawrakow** commented on **2025-10-28** at **05:22:32**

I would try the one from the `ggml` org instead.

https://huggingface.co/ggml-org/gpt-oss-120b-GGUF

The main difference is that it uses `Q8_0` for the tensors that are `F16` in Unsloth's model. I expect you will see a non-negligible uplift in TG performance.