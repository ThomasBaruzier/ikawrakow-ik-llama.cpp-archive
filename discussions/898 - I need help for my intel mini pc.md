## 🗣️ [Discussion #898](https://github.com/ikawrakow/ik_llama.cpp/discussions/898) - I need help for my intel mini pc

| **Author** | `lannashelton` |
| :--- | :--- |
| **State** | ✅ **Answered** |
| **Created** | 2025-11-04 |
| **Updated** | 2025-11-07 |

---

## 📄 Description

I have a mini pc with intel 125h cpu. It has 96 gb DDR5 ram so I can load big MOE models like GLM 4.5 Air on it. I run local services on it 24/7 and I like loading local AI on it to have access to all the time with low power consumption. I get about 4 t/s with GLM-Air in Q4_K_S quant using Kobold.cpp which is like a fork of llama.cpp. I can live with this token generation speed but prompt processing is also about 4-5 t/s and that is abysmally slow. I feel like I should be getting a lot more than that with 125h and DDR5 ram so I don't know what I am doing wrong. I have some questions.

1. If I use ik_llama.cpp, would it speed things up for me?
2. Which quant should I use? One of those R4 ones? iq4_NL? Q4_0? Which one is best for cpu only inference?
3. Is there any settings or flags I should enable/disable?

---

## 💬 Discussion

👤 **ikawrakow** commented on **2025-11-04** at **11:44:21**

For CPU-only inference my expectation is that you should get much better PP performance with `ik_llama.cpp` than with mainline. TG is mostly memory bound, so there performance gains, if any, will be small (and somewhat quantization type dependent).

I can run GLM-4.5-Air on my Ryzen-5975WX CPU. I get about 300 t/s PP. If what I find on cpubenchmark.net is representative for LLM inference performance, my CPU is about 3.75X faster than yours, so from that I would estimate about 80 t/s PP.  Although, if a significant portion of the cpubenchmark.net multi-threaded score comes from the efficiency cores, then perhaps you will get significantly less, perhaps more in the 20-40 t/s range (see also the last bullet point below). 

It is easiest to start with the same model(s) that you have been using. `ik_llama.cpp` supports all quantization type supported by `llama.cpp` (or `Kobold.cpp`). You can worry about quantization types later, after confirming that you get better performance.

For a MoE model such as GLM-4.5-Air, the main parameters that may influence performance are
* `-ub` or `--ubatch-size`. This can have quite a big impact on PP performance, but the best value tends to be model dependent (mostly dependent on the ratio of total to active experts). The default is `512`, so you should definitely try 1024 and 2048 to see where PP performance is best
* On the CPU using `Q8_0`-quantized K-cache can improve PP and TG performance for long context quite a bi compared to the default `f16` KV cache type. Quantized K-cache is more important for performance than quantized V cache. So, unless you believe that KV cache quantization reduces model output quality as some people around the Internet do, always add `-ctk q8_0` to your command line.
* For CPUs with big and little cores such as the 125h, you may need to explicitly set the number of threads to be the same as the number of performance cores (4 for the 125h?) using `-t`. Is this what you do with `Kobold.cpp`?

Another argument you may want to try is `-mqkv`. This will merge the Q, K, and V attention tensors into a single tensor if possible, which may result in a slightly better performance. I say "may" instead of "will", because on the CPU things often depend on how the stars fall (i.e., where tensor data ends up in RAM). If Q, K and V use a different quantization type there will be no difference.

Other command line options are set by default to the best possible value, so it is unlikely to gain performance by changing them.

> 👤 **lannashelton** replied on **2025-11-04** at **12:19:36**
> 
> Thanks for the quick reply 🙏
> 
> Kobold.cpp automatically sets my thread amount to 8 and I was just running it like that. I tested threads 12 blasthreads 18 but I didn't notice difference in speed despite cpu cores becoming 100%
> 
> I downloaded and tested iq4_NL in kobold.cpp with same settings and I got same speed as before.
> 
> Now I installed and built ik_llama.cpp with blas settings and ran iq4_NL with threads 8 and -ctk q8_0 -ctv q8_0. This made prompt processing 13.29 t/s. Interesting!
> 
> I tested it with 4 threads like you said and prompt processing became 12.68 t/s. This is interesting because with 4 threads cpu temps hardly ever changes so feels like it's very light load pc despite getting like 2.5x faster prompt processing than before.
> 
> Is -ctv q8_0 unnecessary?

> 👤 **ikawrakow** replied on **2025-11-04** at **12:39:15**
> 
> > Is -ctv q8_0 unnecessary?
> 
> It tends to not improve performance (but will reduce the cache size).
> 
> How are you measuring PP performance?

> 👤 **lannashelton** replied on **2025-11-04** at **12:46:31**
> 
> > How are you measuring PP performance?
> 
> Terminal output
> 
> INFO [           print_timings] prompt eval time     =   77438.64 ms /  1045 tokens (   74.10 ms per token,    13.49 tokens per second) | tid="123319104358720" timestamp=1762260274 id_slot=0 id_task=0 t_prompt_processing=77438.642 n_prompt_tokens_processed=1045 t_token=74.10396363636364 n_tokens_second=13.494554824450562

> 👤 **ikawrakow** replied on **2025-11-04** at **12:49:18**
> 
> But are the 1045 tokens all in one prompt? If so, try adding `-ub 1024` to see what happens.

> 👤 **lannashelton** replied on **2025-11-04** at **12:58:25**
> 
> > But are the 1045 tokens all in one prompt? If so, try adding `-ub 1024` to see what happens.
> 
> Yes, same context. I've added -ub 1024 and it also helped:
> 
> INFO [           print_timings] prompt eval time     =   68045.46 ms /  1045 tokens (   65.12 ms per token,    15.36 tokens per second) | tid="124880837593408" timestamp=1762261055 id_slot=0 id_task=0 t_prompt_processing=68045.458 n_prompt_tokens_processed=1045 t_token=65.11527081339713 n_tokens_second=15.357380649859099
> 
> I have another question. When I load model, my ram usage doesn't increase, only cache increases. It won't let me mlock. Would that affect performance?

> 👤 **lannashelton** replied on **2025-11-04** at **13:04:16**
> 
> --thread 12 =
> 
> INFO [           print_timings] prompt eval time     =   60790.66 ms /  1045 tokens (   58.17 ms per token,    17.19 tokens per second) | tid="133771086178624" timestamp=1762261381 id_slot=0 id_task=0 t_prompt_processing=60790.664 n_prompt_tokens_processed=1045 t_token=58.17288421052631 n_tokens_second=17.190139591171437
> INFO [           print_timings] generation eval time =   59843.02 ms /   224 runs   (  267.16 ms per token,     3.74 tokens per second) | tid="133771086178624" timestamp=1762261381 id_slot=0 id_task=0 t_token_generation=59843.016 n_decoded=224 t_token=267.15632142857146 n_tokens_second=3.7431268504247845

> 👤 **ikawrakow** replied on **2025-11-04** at **13:16:34**
> 
> > Now I installed and built ik_llama.cpp with blas settings and ran iq4_NL
> 
> You mean you used `cmake -DGGML_BLAS=ON` ? If so, try building with `cmake -DGGML_BLAS=OFF`. This is normally much better.

> 👤 **lannashelton** replied on **2025-11-04** at **13:44:03**
> 
> > You mean you used `cmake -DGGML_BLAS=ON` ? If so, try building with `cmake -DGGML_BLAS=OFF`. This is normally much better.
> 
> Yes. I built with BLAS=ON
> 
> Now I built again with BLAS=OFF and there doesn't seem to be difference.
> 
> INFO [           print_timings] prompt eval time     =   60470.60 ms /  1045 tokens (   57.87 ms per token,    17.28 tokens per second) | tid="137376428448064" timestamp=1762263578 id_slot=0 id_task=0 t_prompt_processing=60470.598 n_prompt_tokens_processed=1045 t_token=57.866600956937795 n_tokens_second=17.281125614137306
> INFO [           print_timings] generation eval time =   55814.36 ms /   205 runs   (  272.27 ms per token,     3.67 tokens per second) | tid="137376428448064" timestamp=1762263578 id_slot=0 id_task=0 t_token_generation=55814.355 n_decoded=205 t_token=272.26514634146343 n_tokens_second=3.672890244812468
> 
> Hmmm

> 👤 **ikawrakow** replied on **2025-11-04** at **14:03:18**
> 
> OK, sorry, I had forgotten that I threw out the `BLAS` parts exactly because people were building `ik_llama.cpp` with BLAS enabled and getting lower performance than they could be. So, yes, the cmake BLAS setting does not make a difference.
> 
> So, I guess, 17 t/s it is on that system.
> 
> In case you feel like downloading `ik_llama.cpp` specific quants, you can try for instance this one: https://huggingface.co/ubergarm/GLM-4.6-GGUF/tree/main/IQ4_KS. In terms of quantization quality it should be roughly equivalent or perhaps even slightly better than the `Q4_K_S/IQ4_NL` quants you are currently using. But as I don't have much experience with this class of CPU, I cannot really predict if this model will perform better on you system.

> 👤 **ikawrakow** replied on **2025-11-04** at **14:13:56**
> 
> Sorry, wrong link. I see @ubergarm doesn't have `IQ4_KS` for GLM-Air. https://huggingface.co/ubergarm/GLM-4.5-Air-GGUF/tree/main/IQ4_K is the one he has in this range of quantized model size.

> 👤 **lannashelton** replied on **2025-11-04** at **14:19:14**
> 
> > OK, sorry, I had forgotten that I threw out the `BLAS` parts exactly because people were building `ik_llama.cpp` with BLAS enabled and getting lower performance than they could be. So, yes, the cmake BLAS setting does not make a difference.
> > 
> > So, I guess, 17 t/s it is on that system.
> > 
> > In case you feel like downloading `ik_llama.cpp` specific quants, you can try for instance this one: https://huggingface.co/ubergarm/GLM-4.6-GGUF/tree/main/IQ4_KS. In terms of quantization quality it should be roughly equivalent or perhaps even slightly better than the `Q4_K_S/IQ4_NL` quants you are currently using. But as I don't have much experience with this class of CPU, I cannot really predict if this model will perform better on you system.
> 
> Yeah, this might be the limit for my poor lil pc. Thanks a lot. Even now it feels much better than how it was at start!

> 👤 **lannashelton** replied on **2025-11-04** at **14:56:21**
> 
> > Sorry, wrong link. I see @ubergarm doesn't have `IQ4_KS` for GLM-Air. https://huggingface.co/ubergarm/GLM-4.5-Air-GGUF/tree/main/IQ4_K is the one he has in this range of quantized model size.
> 
> I was going to get the KSS one, is that not good?

> 👤 **ikawrakow** replied on **2025-11-04** at **15:13:56**
> 
> That one works too

---

👤 **Kotali-2019** commented on **2025-11-06** at **14:37:48**

Hi, I’ve been experimenting with both ik_llama and the mainline llama.cpp for a while. Since you mentioned PP speed @ikawrakow , I’d like to share my test results on a Dual Xeon E5-2696 v3 system (not fancy, but each CPU has 18 cores, so 36 physical cores and 72 threads in total).
I ran the same llama-bench command on three different builds under Debian 12:
***root@LocalAI:~/llama-bin# ./llama-bench -m /root/models/Qwen/Qwen3-Embedding-0.6B-Q8_0.gguf -t 72 -ub 4906 -n 0***


### **1. llama.cpp result**
build: 3cfa9c3f1 (6840)
load_backend: loaded RPC backend from /root/llama-bin/libggml-rpc.so
load_backend: loaded CPU backend from /root/llama-bin/libggml-cpu-haswell.so

| model                | size       | params     | backend | threads | n_ubatch | test   | t/s                |
|----------------------|-----------|-----------|---------|---------|----------|--------|---------------------|
| qwen3 0.6B Q8_0      | 603.87 MiB| 595.78 M  | CPU     | 72      | 4906     | pp512  | 627.86 ± 20.64     |

system_info: n_threads = 72 (n_threads_batch = 72) / 72 | CPU: SSE3=1 | SSSE3=1 | AVX=1 | AVX2=1 | F16C=1 | FMA=1 | BMI2=1 | LLAMAFILE=1 | OPENMP=1 | REPACK=1


### **2. ik_llama with BLAS**
build: 575e2c28 (3958)
| model                | size       | params     | backend | threads | n_ubatch | test   | t/s                |
|----------------------|-----------|-----------|---------|---------|----------|--------|---------------------|
| qwen3 0.6B Q8_0      | 761.24 MiB| 751.09 M  | BLAS    | 72      | 4906     | pp512  | 1143.57 ± 81.61    |

system_info: n_threads = 72 / 72 | AVX=1 | AVX2=1 | AVX512=0 | FMA=1 | F16C=1 | BLAS=1 | SSE3=1 | SSSE3=1 | LLAMAFILE=1


### **3. ik_llama without BLAS**
build: 575e2c28 (3958)
| model                | size       | params     | backend | threads | n_ubatch | test   | t/s                |
|----------------------|-----------|-----------|---------|---------|----------|--------|---------------------|
| qwen3 0.6B Q8_0      | 761.24 MiB| 751.09 M  | CPU     | 72      | 4906     | pp512  | 1169.02 ± 54.91    |

system_info: n_threads = 72 / 72 | AVX=1 | AVX2=1 | AVX512=0 | FMA=1 | F16C=1 | BLAS=0 | SSE3=1 | SSSE3=1 | LLAMAFILE=1


My questions:

Why does the model size differ between the llama.cpp and ik_llama builds (603.87 MiB vs 761.24 MiB)? Could this be due to repack compression?
Why do the parameter counts differ?
Why is there such a big difference in the ± values for t/s (81.61 vs 54.91)?

From my perspective, for embedding models, PP speed seems more critical than TG speed. Does that sound right?

---

👤 **ikawrakow** commented on **2025-11-06** at **15:15:53**

> Why does the model size differ between the llama.cpp and ik_llama builds (603.87 MiB vs 761.24 MiB)? Could this be due to repack compression?

Repack does not compress. Most likely the model does not have a dedicated output tensor but uses the token embedding tensor for that. In this situation `ik_llama.cpp` duplicates the token embedding tensor, so it gets counted twice. This used to be what `llama.cpp` does as well, but probably they have changed to avoid the duplication (or simply do not report the size of the duplicated tensor).

> Why do the parameter counts differ?

Same reason as above

> Why is there such a big difference in the ± values for t/s (81.61 vs 54.91)?

`llama-bench` runs by default 5 repetitions and uses the standard deviation of this small sample for the `+/-`. These runs are pretty short, so one gets a lot of fluctuation, so standard deviation is high and depends somewhat how the stars fell in that moment. You can get a better estimate by increasing the repetitions with, e.g., `-r 20`.

But apart from this, if batches of 512 tokens represents your use case, then `-p 512` is a good benchmark. But if your batches are much bigger, it is better to benchmark with a higher `-p` that is more in line with your use case.

Also, `-ub 4096` does not work, it will get truncated to the default batch size of 2048. If you want micro batches of 4096, you need to use `-b 4096 -ub 4096`. But in this benchmark this will do nothing as the prompt is only 512 tokens, so batch/u-batch of 512 will get used. To see how things depend on the u-batch size, you can use for instance
```
llama-bench -p 4096 -n 0 -b 4096 -ub 256,512,1024,2048,4096 $other_args
```

> 👤 **ikawrakow** replied on **2025-11-06** at **16:24:09**
> 
> Another comment: using hyper-threading (almost) never helps performance. 1100-1200 t/s for a 0.6B parameter model seemed quite low, so I downloaded and tested the same model on my Ryzen-7950X CPU (16 cores, 32 threads). I get 3100 t/s for `pp512` using 16 threads, and 2980 t/s using 32 threads. `llama.cpp` manages 1260 t/s on the Ryzen-7950X.

> 👤 **Kotali-2019** replied on **2025-11-07** at **01:44:28**
> 
> Hi, thank you so much for the explanations and advice — much appreciated! @ikawrakow 
> I found this suggestion on Hugging Face (https://huggingface.co/Qwen/Qwen3-Embedding-0.6B-GGUF) to run Qwen embedding with **-ub 8192** using the following command:
> ***./build/bin/llama-server -m model.gguf --embedding --pooling last -ub 8192 --verbose-prompt***
> I tried this because, during my RAG process, I occasionally encountered an error suggesting that I should increase the physical batch size (unfortunately, I haven’t been able to reproduce that error yet to show you). Using -ub 8192 seemed to help, especially since I was running with --parallel 2. Later, I tested with -ub 4096 during benchmarking.
> ***Maybe I’m missing something regarding this optimization? For context, I’ve been embedding with a dimension of 1024 (dim = 1024).***

> 👤 **ikawrakow** replied on **2025-11-07** at **05:32:46**
> 
> Qwen3-0.6B is a dense model. The wisdom to use large batch sizes floating around the Internet only apply to MoE models running on the GPU, but even then a batch size of 8192 is pushing it, unless one is using hybrid inference (MoE tensors stored in RAM but copied to the GPU for computation). For Qwen3-0.6B running on the CPU you will get the best performance by leaving batch and u-batch size at their default values.
> 
> Anyway, computing embeddings is not a use case I have been focusing on, so let me know if you run into issues.

> 👤 **Kotali-2019** replied on **2025-11-07** at **14:15:16**
> 
> Hi @ikawrakow ,
> Sorry to bother you again. I’m eager to benchmark and test the performance of the ik-llama build. While testing with the curl command, I noticed that the embedding result seems completely incorrect.
> Could you please advise whether I should open a new issue for this, or if a quick suggestion from you here would be sufficient?
> Here’s what I ran:
> **Server command:**
> ```bash
> ./llama-server -m /root/models/Draft/Embedding-Qwen3-0.6B-Q8_0.gguf \
> -t 72 --host 0.0.0.0 --port 10002 --parallel 2 --embedding \
> --pooling last --embd-output-format json --alias Embedding-Qwen3-0.6B-Q8_0
> ```
> **Curl command:**
> ```bash
> curl --location 'http://192.168.11.101:10002/v1/embeddings' \
> --header 'Content-Type: application/json' \
> --data '{
>     "model": "Embedding-Qwen3-0.6B-Q8_0",
>     "input": "Test"
> }'
> ```
> **Expected result:**
> ```json
> {"model":"Embedding-Qwen3-0.6B-Q8_0","object":"list","usage":{"prompt_tokens":2,"total_tokens":2},"data":[{"embedding":[-0.012152200564742088,-0.059889741241931915,...0.01874898374080658,0.004860232584178448],"index":0,"object":"embedding"}]}
> ```
> **ik-llama build result:**
> ```json
> {"model":"Embedding-Qwen3-0.6B-Q8_0","object":"list","usage":{"prompt_tokens":0,"total_tokens":0},"data":[{"embedding":[],"index":0,"object":"embedding"}]}
> ```

> 👤 **ikawrakow** replied on **2025-11-07** at **14:44:24**
> 
> Yes, enter an issue. Personally I have never used the embedding endpoint, so it is possible it is broken.