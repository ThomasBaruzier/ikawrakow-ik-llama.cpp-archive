## 🗣️ [Discussion #839](https://github.com/ikawrakow/ik_llama.cpp/discussions/839) - speculative decoding via network?

| **Author** | `magikRUKKOLA` |
| :--- | :--- |
| **State** | ✅ **Open** |
| **Created** | 2025-10-17 |
| **Updated** | 2025-10-26 |

---

## 📄 Description

Is it possible to run the inference with speculative decoding for MoE models such as DeepSeek-V3.1-Terminus via two machines inside LAN?  That is, say I have a THIREUS-5.4498bpw-R4 at one machine and say, IQ2_KS-2.472bpw at the draft-machine.  Since the second quant is somewhat sane but more than two time smaller (and the decode speed is limited to ram bw), the boost supposed to be quite significant?  like 60-70%?  Will it work?

REFS:

https://github.com/ggml-org/llama.cpp/discussions/6853
```
Here's a small illustration i made: https://github.com/ggml-org/llama.cpp/commit/c8d446df78664726ef1d2e70aa787b235813780e

    It runs main model (Llama3-70-Q8) on M2 Ultra GPU
    It runs draft model (Llama3-8-Q5) on 16 cpu perf cores of the same M2 Ultra
...
```

https://github.com/ggml-org/llama.cpp/discussions/6853#discussioncomment-12431262
```
you might find our [distributed speculative decoding](https://openreview.net/forum?id=cJd1BgZ9CS) algorithm useful for this CPU + GPU hardware setup. Our ICLR'25 publication includes theory and simulations demonstrating speedups over both vanilla speculative decoding (non-distributed) and autoregressive decoding—for any drafter model:
```

[EDIT]:
https://arxiv.org/abs/2509.04576

```
Communication-Efficient Collaborative LLM Inference via Distributed Speculative Decoding
[Ce Zheng](https://arxiv.org/search/eess?searchtype=author&query=Zheng,+C), [Tingting Yang](https://arxiv.org/search/eess?searchtype=author&query=Yang,+T)

    Speculative decoding is an emerging technique that accelerates large language model (LLM) inference by allowing a smaller draft model to predict multiple tokens in advance, which are then verified or corrected by a larger target model. In AI-native radio access networks (AI-RAN), this paradigm is well-suited for collaborative inference between resource-constrained end devices and more capable edge servers or base stations (BSs). However, existing distributed speculative decoding requires transmitting the full vocabulary probability distribution from the draft model on the device to the target model at the BS, which leads to prohibitive uplink communication overhead. To address this issue, we propose a ``Top-K Sparse Logits Transmission (TK-SLT)`` scheme, where the draft model transmits only the top-K token raw probabilities and the corresponding token indices instead of the entire distribution. This approach significantly reduces bandwidth consumption while maintaining inference performance. We further derive an analytical expression for the optimal draft length that maximizes inference throughput, and provide a theoretical analysis of the achievable speedup ratio under TK-SLT. Experimental results validate both the efficiency and effectiveness of the proposed method.
```

---

## 💬 Discussion

👤 **magikRUKKOLA** commented on **2025-10-17** at **22:50:30**

If that works, it means that it makes sense to build one machine with 512GB RAM and another, a draft-machine with only 256GB RAM (to run a smaller quant).  The price difference between 8x64GB 2666MT/s ECC and simple 8x32GB 3200MT/s non-ECC is quite significant.  Right?

---

👤 **saood06** commented on **2025-10-17** at **23:12:32**

It is not currently supported here, there is an open request ([#785](https://github.com/ikawrakow/ik_llama.cpp/issues/785)) for what would add support for it.

---

👤 **magikRUKKOLA** commented on **2025-10-18** at **13:28:08**

ha!  I was just experimenting with speculative decoding and found out that I can speed up the decode by ~~about 15%~~ **up to 100%** for Qwen3-Coder-480B-A35B-Instruct-5.1546bpw.

I took the Qwen3-Coder-30B-A3B IQ1_KT as draft model for Qwen3-Coder-480B-A35B-Instruct-5.1546bpw.  To fit everything onto the GPU with a decent context size if had to use -ctv q8_0 and -ctkd q4_0 -ctvd q4_0 plus I had to reduce the batch sizes to 4k.

```
/opt/ik_llama.cpp/ik_llama.cpp/build/bin/llama-server \                                                                     
    --model /opt/THIREUS/Qwen3-Coder-480B-A35B-Instruct-5.1546bpw/Qwen3-Coder-480B-A35B-Instruct-THIREUS-BF16-SPECIAL_TENSOR-00001-of-00748.gguf \                                                                                                      
    --model-draft /opt/ubergarm/Qwen3-Coder-30B-A3B-Instruct-GGUF/Qwen3-Coder-30B-A3B-Instruct-IQ1_KT.gguf \                    --draft-max 16 \                                                                                                        
    --alias THIREUS/Qwen3-Coder-480B-A35B-Instruct-5.1546bpw \                                                              
    --ctx-size $((96 * 1024)) \                                                                                             
    --ctx-size-draft $((96 * 1024)) \                                                                                       
    -b 4096 -ub 4096 \                                                                                                      
    --mlock \                                                                                                               
    --temp 0.7 --top-k 20 --top-p 0.8 --min-p 0.1 --repeat-penalty 1.05 \                                                   
    -ctk q8_0 -ctv q8_0 \                                                                                                   
    -ctkd q4_0 -ctvd q4_0 \                                                                                                 
    -fa \                                                                                                                   
    -fmoe \                                                                                                                 
    -amb 512 \                                                                                                              
    --seed 3407 \                                                                                                           
    --split-mode layer \                                                                                                    
    --tensor-split 32,40 \                                                                                                  
    --main-gpu 1 \                                                                                                          
    --override-tensor exps=CPU \                                                                                            
    --gpu-layers 99 \                                                                                                       
    --gpu-layers-draft 99 \                                                                                                 
    --threads $(grep ^cpu\\scores /proc/cpuinfo | uniq | awk '{print $4}' | xargs -I{} echo "{}-0" | bc) \                  
    --threads-draft $(grep ^cpu\\scores /proc/cpuinfo | uniq | awk '{print $4}' | xargs -I{} echo "{}-0" | bc) \            
    --host 0.0.0.0 \                                                                                                        
    --port 8080 \                                                                                                           
    --log-enable \                                                                                                          
    --logdir /var/log/ \                                                                                                    
    --jinja \                                                                                                               
    --special \                                                                                                             
    --verbose-prompt --verbosity 2 \                                                                                        
    --prompt-cache "$HOME/.cache/ik_llama.cpp/prompt-cache.bin" --prompt-cache-all \                                        
    --slot-save-path "$HOME/.cache/ik_llama.cpp/slot.bin" \                                                                 
    --lookup-cache-dynamic "$HOME/.cache/ik_llama.cpp/slot.bin" \                                                           
    --keep -1 \                                                                                                             
    --slot-prompt-similarity 0.35 \                                                                                         
    --metrics  
```

so it does seem to be working.  Since llama-bench doesn't support the speculative decoding, I had to test it manually.  Like:

```
pre /opt/GGUF-Tool-Suite/GGUF-Tool-Suite/quant_assign.py | mods -m qwen3 "explain the code"
```

that resulted in a TG performance boost from 6.6tps to 7.81tps.

w/o spec. decoding:
```
INFO [           print_timings] prompt eval time     =   71046.52 ms / 19146 tokens (    3.71 ms per token,   269.49 tokens per second) | tid="140589686403072" timestamp=1760764217 id_slot=0 id_task=0 t_prompt_processing=71046.523 n_prompt_tokens_processed=19146 t_token=3.710776297921237 n_tokens_second=269.4853905799162
INFO [           print_timings] generation eval time =  223645.05 ms /  1477 runs   (  151.42 ms per token,     6.60 tokens per second) | tid="140589686403072" timestamp=1760764217 id_slot=0 id_task=0 t_token_generation=223645.049 n_decoded=1477 t_token=151.41844888287068 n_tokens_second=6.604215056868976
```

with spec. decoding:
```
INFO [           print_timings] prompt eval time     =  105156.30 ms / 19146 tokens (    5.49 ms per token,   182.07 tokens 
per second) | tid="140399929704448" timestamp=1760793199 id_slot=0 id_task=0 t_prompt_processing=105156.302 n_prompt_tokens_
processed=19146 t_token=5.492337929593648 n_tokens_second=182.0718267555662
INFO [           print_timings] generation eval time =  209608.00 ms /  1638 runs   (  127.97 ms per token,     7.81 tokens 
per second) | tid="140399929704448" timestamp=1760793199 id_slot=0 id_task=0 t_token_generation=209607.998 n_decoded=1638 t_
token=127.96581074481074 n_tokens_second=7.814587304058884
INFO [           print_timings]           total time =  314764.30 ms | tid="140399929704448" timestamp=1760793199 id_slot=0 

VERB [              operator()] data stream | tid="140371829452800" timestamp=1760793199 to_send="data: {\"choices\":[],\"cr
eated\":1760793199,\"id\":\"chatcmpl-5fbLFTeNRksT7Z21DoJvR5Dx0Ht7ZTNh\",\"model\":\"Qwen3-Coder-480B-A35B-Instruct-5.1546bpw
\",\"object\":\"chat.completion.chunk\",\"usage\":{\"completion_tokens\":1638,\"prompt_tokens\":19146,\"total_tokens\":20784
},\"timings\":{\"prompt_n\":19146,\"prompt_ms\":105156.302,\"prompt_per_token_ms\":5.492337929593648,\"prompt_per_second\":1
82.0718267555662,\"predicted_n\":1638,\"predicted_ms\":209607.998,\"predicted_per_token_ms\":127.96581074481074,\"predicted_
per_second\":7.814587304058884,\"draft_n\":1093,\"draft_n_accepted\":810}}\n\n"
```

So yeah, its working.  It would be nice to have the draft model run at a different network-connected device for sure.

BTW the --seed parameter doesn't seem to be working for a draft model?

[EDIT]:

Kek the actual decode speed is higher on a real data I am using (I am asking the LLM to rewrite the code so there is alot of repetitive segments so the draft model does the job of copy/pasting and the acceptance rate is quite high).  Its 10.74 tps!  Ha!

lol what a funny technique!  I wonder the same can be applied to other models.

```
 VERB [              operator()] data stream | tid="139636347760640" timestamp=1760797564 to_send="data: {\"choices\":[],\"created\":1760797564,\"id\":\"chatcmpl-HyjiL9mQnREv72nGIV3Q8tIBLMB4H9zX\",\"model\":\"Qwen3-Coder-480B-A35B-Instruct-5.1546bpw\",\"object\":\"chat.completion.chunk\",\"usage\":{\"completion_tokens\":1750,\"prompt_tokens\":18477,\"total_tokens\":20227},\"timings\":{\"prompt_n\":18477,\"prompt_ms\":101315.749,\"prompt_per_token_ms\":5.483344103480002,\"prompt_per_second\":182.37046246383667,\"predicted_n\":1750,\"predicted_ms\":162961.907,\"predicted_per_token_ms\":93.12108971428572,\"predicted_per_second\":10.738705948010292,\"draft_n\":1492,\"draft_n_accepted\":1340}}\n\n"
```

[EDIT2]:

Kek2 lol 12.77 tps in decode:

```
VERB [              operator()] data stream | tid="139636330975232" timestamp=1760799119 to_send="data: {\"choices\":[],\"created\":1760799119,\"id\":\"chatcmpl-vlWrtENX5pXzFNAg8zBO667OcqiaVuGV\",\"model\":\"Qwen3-Coder-480B-A35B-Instruct-5.1546bpw\",\"object\":\"chat.completion.chunk\",\"usage\":{\"completion_tokens\":18759,\"prompt_tokens\":20262,\"total_tokens\":39021},\"timings\":{\"prompt_n\":36,\"prompt_ms\":976.89,\"prompt_per_token_ms\":27.135833333333334,\"prompt_per_second\":36.851641433528854,\"predicted_n\":18759,\"predicted_ms\":1468782.208,\"predicted_per_token_ms\":78.29746830854523,\"predicted_per_second\":12.771805035372543,\"draft_n\":16706,\"draft_n_accepted\":16375}}\n\n"
```

lol its basically like 100% improvement in decode.

---

👤 **ikawrakow** commented on **2025-10-18** at **14:17:16**

> BTW the --seed parameter doesn't seem to be working for a draft model?

Yes, there is very little one can set for the draft model, and the seed is not one of the things that one can adjust.

> 👤 **magikRUKKOLA** replied on **2025-10-18** at **15:25:01**
> 
> @ikawrakow 
> > > BTW the --seed parameter doesn't seem to be working for a draft model?
> > 
> > Yes, there is very little one can set for the draft model, and the seed is not one of the things that one can adjust.
> 
> So right now there is no way to do the reproducible results via the speculative decoding?  (that is, each time it responds slightly differenly even though the --seed parameter is specified).
> 
> There is no support for speculative decoding both in llama-perplexily and llama-sweep-bench as well, right?

> 👤 **ikawrakow** replied on **2025-10-18** at **16:15:50**
> 
> There is no token generation involved when evaluating perplexity, so no point in having a draft model.
> 
> There is no support for a draft model in `llama-bench` or `llama-sweep-bench`. But these tools use random data, so even if one would make the effort to support a draft model, results would not be representative for real world usage (where we know that the success rate of the draft model and corresponding speedup (or slowdown) is highly context dependent). So, then, to have a proper benchmark one would need to start adding the ability to load and use specific contexts, which would make  things way more complicated.
> 
> My concept was that most people use greedy sampling in the draft model, so the seed does not play a role. If one wanted to use an actual sampling sequence for the draft model, then ideally it would be the same as the one used in the main model, and then  one would also implement correlated sampling (i.e., use the same random number sequence for sampling in the draft and in the main model) to increase the chance of accepting the draft sequence.
> 
> So, in short, there is a lot one could improve around draft model usage.

---

👤 **magikRUKKOLA** commented on **2025-10-21** at **06:56:28**

Tried to do the similar with Ling-1T and its working.  10% (+/-) the boost in decode with Q2_K official quant from Ling-Flash-2.0.

```
/opt/ik_llama.cpp/ik_llama.cpp/build/bin/llama-server \                                                                     
    --ctx-size $((32 * 1024)) \                                                                                             
    --ctx-size-draft $((32 * 1024)) \                                                                                       
    --model-draft /opt/inclusionAI/Ling-flash-2.0-GGUF/Q2_K/Ling-flash-2.0-Q2_K.gguf \                                      
    --draft-max 16 \                                                                                                        
    -ctkd q8_0 -ctvd q8_0 \                                                                                                 
    --gpu-layers-draft 99 \                                                                                                 
    --model /opt/ubergarm/Ling-1T-GGUF/smol-IQ4-KSS/Ling-1T-smol-IQ4_KSS-00001-of-00011.gguf \                              
    --alias ubergarm/Ling-1T-smol-IQ4_KSS-test \                                                                            
    -b $((8 * 512)) -ub $((8 * 512)) \                                                                                      
    --mlock \                                                                                                               
    --temp 0.7 --top-k 0 --top-p 0.95 --min-p 0.1 --repeat-penalty 1.1 \                                                    
    -ctk q8_0 -ctv q8_0 \                                                                                                   
    -fa \                                                                                                                   
    -fmoe \                                                                                                                 
    -amb 512 \                                                                                                              
    --split-mode layer \                                                                                                    
    --tensor-split 1,2,50 \                                                                                                 
    --main-gpu 1 \                                                                                                          
    --override-tensor exps=CPU \                                                                                            
    -no-ooae \                                                                                                              
    --n-gpu-layers 99 \                                                                                                     
    --threads $(grep ^cpu\\scores /proc/cpuinfo | uniq | awk '{print $4}' | xargs -I{} echo "{}-0" | bc) \                  
    --host 0.0.0.0 \                                                                                                        
    --port 8080 \                                                                                                           
    --log-enable \                                                                                                          
    --logdir /var/log/ \                                                                                                    
    --jinja \                                                                                                               
    --special \                                                                                                             
    --verbosity 2 \                                                                                                         
    --verbose-prompt \                                                                                                      
    --reasoning-format auto \                                                                                               
    --prompt-cache "$HOME/.cache/ik_llama.cpp/prompt-cache.bin" --prompt-cache-all \                                        
    --slot-save-path "$HOME/.cache/ik_llama.cpp/slot.bin" \                                                                 
    --lookup-cache-dynamic "$HOME/.cache/ik_llama.cpp/slot.bin" \                                                           
    --keep -1 \                                                                                                             
    --slot-prompt-similarity 0.35 \                                                                                         
    --metrics  
```

> 👤 **saood06** replied on **2025-10-21** at **08:32:56**
> 
> > Tried to do the similar with Ling-1T and its working. 10% (+/-) the boost in decode with Q2_K official quant from Ling-Flash-2.0.
> 
> If you are willing to make your own you could try and increase acceptance rate with similar size using newer quant types (see https://github.com/ikawrakow/ik_llama.cpp/commit/931bc412aef063037a6b2080f71dd844817176c8#commitcomment-161965520 ).

> 👤 **magikRUKKOLA** replied on **2025-10-21** at **21:13:15**
> 
> @saood06 
> > If you are willing to make your own
> 
> Yeah, there is no other option for me.  ling is the exact model I was looking for but its quite slow for my AMD Threadripper 3995wx system (~7 tps in decode).  So only option for me is to use the spec. decoding and really fast and sane quant.  Is there any 101 tutorial how to cook the inclusionAI/Ling-flash-2.0 quant with imatrix?
> 
> @Thireus , @ubergarm 
> 
> Hey, bro!  Are you planning to cook any Ling-flash-2.0 ?  Do you have any info how I can cook those?

> 👤 **ubergarm** replied on **2025-10-21** at **22:02:24**
> 
> @magikRUKKOLA  
> 
> If I understand what you posted above, it sounds like you're seeing 10% boost in TG speeds when using Ling-flash-2.0-Q2_K as the speculative decoding model for Ling-1T-smol-IQ4_KSS?
> 
> Ling-flash-2.0 Q2_K is like almost 38GB?! My impression is folks use *very small* like maybe 500MB or under 1GB models for speculative decoding so this is pretty wild. 
> 
> Before trying to cook your own IQ1_KT Ling-flash-2.0 I'd suggest trying out one of the existing smaller Ling-mini-2.0 version of the model like https://huggingface.co/inclusionAI/Ling-mini-2.0-GGUF?show_file_info=Ling-mini-2.0-Q4_K_M.gguf or the smaller Q2 assuming they are compatible.
> 
> I see team mradermacher is moving now on some of the bigger Ling quants so expect more mainline quants to become available soon now that their PR is merged. You can find some other experimental 0.6B draft models in this collection https://huggingface.co/collections/jukofyork/draft-models-67d349b9dddd282a74057d5d though I don't see anything for Ling yet.
> 
> The basic quant cookers guide is still fairly applicable though out of date at this point: https://github.com/ikawrakow/ik_llama.cpp/discussions/434
> 
> I don't have time this week to cook these models unfortunately.
> 
> Keep us posted how you make out!

> 👤 **magikRUKKOLA** replied on **2025-10-21** at **22:07:56**
> 
> @ubergarm 
> > If I understand what you posted above, it sounds like you're seeing 10% boost in TG speeds when using Ling-flash-2.0-Q2_K as the speculative decoding model for Ling-1T-smol-IQ4_KSS?
> 
> Yes, exactly!  10%+ boost.
> 
> > Ling-flash-2.0 Q2_K is like almost 38GB?! 
> 
> Well, yeah.  I am using three GPU system where the draft model takes about two GPUs.  I was thinking initially that I should do the inference of the draft model at a different machine but it fits okay as of now for the 32k ctx.
> 
> I also noticed that if I try to use lesser than q8_0 KV cache quantisation the performance drops for some reason.  That was not the case for Qwen3-Coder-480 spec. decoding with Qwen3-Coder-30B as a draft though.
> 
> > My impression is folks use very small like maybe 500MB or under 1GB models for speculative decoding so this is pretty wild.
> 
> Yeah, I tried those initially and they suck.  At least that was the case for Qwen3-Coder.  When the acceptance is lower than about 60% there is no point of using the draft model at all.  So for all those 0.6B-0.75B quants sucked.
> 
> [EDIT]:
> > The basic quant cookers guide is still fairly applicable though out of date at this point: https://github.com/ikawrakow/ik_llama.cpp/discussions/434
> 
> Okay, great!  Let me try to do that.

> 👤 **ubergarm** replied on **2025-10-21** at **22:26:16**
> 
> @magikRUKKOLA   
> 
> You're embarking down a fun path haha...
> 
> One tip I can give you is that the llama-imatrix commands changed a bit for these MoEs in order to exercise all the tensors to collect data. This is what I used for big Ling-1T which you can modify for Ling-mini-2.0 or whichever one you choose to try:
> 
> ```bash
> ./build/bin/llama-imatrix \
>     -m "$model" \
>     -fa \
>     --no-fused-up-gate \
>     -ger \
>     -f ubergarm-imatrix-calibration-corpus-v02.txt \
>     -o /mnt/data/models/ubergarm/Ling-1T-GGUF/imatrix-Ling-1T-Q8_0.dat \
>     --verbosity 1 \
>     --layer-similarity \
>     --seed 1337 \
>     --ctx-size 512 \
>     -ub 4096 -b 4096 \
>     --numa distribute \
>     --threads 128 \
>     --threads-batch 192 \
>     --no-mmap
> ```
> 
> You can adjust for your GPUs as desired. the `--seed 1337 \` does nothing, it is just for fun and as a "fingerprint" to see where these commands end up being copy pasted. Keep context size at 512, but you can adjust batches if you prefer for your hardware.
> 
> Also I don't know what dimensions the tensors are for the smaller Ling models, so you may be restricted to like `iq4_nl` or `q4_0` if column sizes not divisible by 256 (super block size of many of the newer quants).

> 👤 **magikRUKKOLA** replied on **2025-10-21** at **22:32:57**
> 
> @ubergarm 
> 
> One additional question regarding the ctx.
> 
> You wrote the following:
> 
> https://huggingface.co/ubergarm/Ling-1T-GGUF
> ```
> --ctx-size 65536 \
> ```
> 
> Does ik_llama.cpp actually supports the YaRN ctx extension?  In my case it just stops outputting after 32k ctx even if I run with 64k ctx.
> 
> Moreover, if I try to run the current master [with exactly 64ctx] I will encounter the:
> 
> ```
> warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context                                    
> warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context                                    
> warning: Cuda Driver error detected: Grid Dimension Y of 65536 exceeds maximum value of 65535                               
> warning: Cuda Driver error detected: Returning 1 (CUDA_ERROR_INVALID_VALUE) from cuLaunchKernel                             
> warning: Cuda Runtime API error detected: cudaLaunchKernel returned cudaErrorInvalidValue(CUresult=1): invalid argument                                                                                                                                 
> warning: Cuda Runtime API error detected: cudaGetLastError returned cudaErrorInvalidValue(CUresult=1): invalid argument     
>                                                                                                                             
> CUDA error: invalid argument                                                                                                
>   current device: 0, in function cuda_bailingmoev2_experts at /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml-cuda/argsort.cu:5
> 03                                                                                                                          
>   cudaGetLastError()                                                                                                        
> /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml-cuda.cu:121: CUDA error                                                        
> [Detaching after fork from child process 2947360]                                                                           
> warning: process 2932099 is already traced by process 2931995                                                               
> ptrace: Operation not permitted.                                                                                            
> No stack.                                                                                                                   
> The program is not being run.                                                                                               
>                                                                                                                             
> Thread 1 "llama-server" received signal SIGABRT, Aborted.                                                                   
> 0x00007fffe069e95c in ?? () from /lib/x86_64-linux-gnu/libc.so.6                            
> ```
> 
> So that error ( 'warning: Cuda Driver error detected: Grid Dimension Y of 65536 exceeds maximum value of 65535  ' )
>  is related to the ctx size, right?
> 
> 
> You never encountered such problems?
> 
> [1] https://github.com/ikawrakow/ik_llama.cpp/issues/854#issuecomment-3427453585
> [2] https://github.com/ikawrakow/ik_llama.cpp/issues/854#issuecomment-3429669481

> 👤 **ubergarm** replied on **2025-10-21** at **23:10:39**
> 
> > --ctx-size 65536 \
> 
> Thanks for catching my mistake, I just updated the model card to be 32k there now.
> 
> I saw a recent issue where you were discussing yarn with ik.  Thanks for pointing out that Ling-1T has 32k context, I just confirmed looking at the output:
> 
> ```bash
> llm_load_print_meta: n_ctx_train      = 32768
> ...
> llm_load_print_meta: rope type        = 2
> llm_load_print_meta: rope scaling     = none
> llm_load_print_meta: freq_base_train  = 600000.0
> llm_load_print_meta: freq_scale_train = 1
> llm_load_print_meta: n_ctx_orig_yarn  = 32768
> ```
> 
> A few thoughts:
> 
> 1. I have only run all my Ling-1T quants CPU-only
> 2. I haven't seen that issue, but pretty sure I pushed it out past 32k tokens in one test.
> 3. I generally avoid extending rope/yarn/context stuff as imo most models struggle under 32k context anway
> 4. psure ik did some of the og mainline lcpp rope/yarn stuff like two years ago
> 
> You could fiddle with it I suppose if you really feel you need more context and see how it goes. Look at `llama-server --help` under the header `context hacking:`
> 
> ```bash
> ./build/bin/llama-server --help
> context hacking:
> 
>          --rope-scaling {none,linear,yarn}
>                                   RoPE frequency scaling method, defaults to linear unless specified by the model
>          --rope-scale N           RoPE context scaling factor, expands context by a factor of N
>          --rope-freq-base N       RoPE base frequency, used by NTK-aware scaling (default: loaded from model)
>          --rope-freq-scale N      RoPE frequency scaling factor, expands context by a factor of 1/N
>          --yarn-orig-ctx N        YaRN: original context size of model (default: 0 = model training context size)
>          --yarn-ext-factor N      YaRN: extrapolation mix factor (default: -1.0, 0.0 = full interpolation)
>          --yarn-attn-factor N     YaRN: scale sqrt(t) or attention magnitude (default: -1.0)
>          --yarn-beta-slow N       YaRN: high correction dim or alpha (default: -1.0)
>          --yarn-beta-fast N       YaRN: low correction dim or beta (default: -1.0)
> 
> ```
> 
> play around with some of this stuff e.g. if I recall with qwen3-30b-a3b you could extend it 4 times with `--rope-scaling yarn --rope-scale 4 --yarn-orig-ctx 32768` to get 128k context. however, i recall testing some of the UD "long context" models which had worse perplexity than the default qwen versions. the qwen model card even said don't extend it unless you really actually need to use it. also no idea if it will work with Ling-1T as I just try to keep my prompts and code chunks small.
> 
> You could also possibly try changing the default rope frequency with `--rope-freq-base` given the default value is `600000` or possibly `--rope-freq-scale`, but i really don't know. 
> 
> assuming the bug is related to this, extending context tends to degrade quality of output in general. 
> 
> Anyway, have fun hacking and keep us posted what you find!

> 👤 **saood06** replied on **2025-10-21** at **23:33:36**
> 
> > I also noticed that if I try to use lesser than q8_0 KV cache quantisation the performance drops for some reason. That was not the case for Qwen3-Coder-480 spec. decoding with Qwen3-Coder-30B as a draft though.
> 
> Different models degrade differently with different cache quantization schemes. You were observing lower performance since your acceptance rate dropped as a result of the cache quantization causing more severe degradation with that draft model.

> 👤 **magikRUKKOLA** replied on **2025-10-22** at **01:45:37**
> 
> @ubergarm 
> 
> ```
>                                                                              
> system_info: n_threads = 12 / 24 | AVX = 1 | AVX_VNNI = 0 | AVX2 = 1 | AVX512 = 0 | AVX512_VBMI = 0 | AVX512_VNNI = 0 | AVX512_BF16 = 0 | FMA = 1 | NEON = 0 | SVE = 0 | ARM_FMA = 0 | F16C = 1 | FP16_VA = 0 | WASM_SIMD = 0 | BLAS = 1 | SSE3 = 1 | SSSE3 = 1 | VSX = 0 | MATMUL_INT8 = 0 | LLAMAFILE = 1 |                                                                       compute_imatrix: tokenizing the input ..                     
> compute_imatrix: tokenization took 218.745 ms                                                                               compute_imatrix: computing over 230 chunks with batch_size 512 
> compute_imatrix: 7.15 seconds per pass - ETA 27.40 minutes                                                                  [1]6.0992,[2]6.7395,[3]4.7859,[4]3.7236,[5]3.4693,[6]2.8952,[7]2.5397,[8]2.2916,[9]2.1170,
> save_imatrix: entry '             blk.29.ffn_down_exps.weight' has partial data (89.84%) 26 out of 256 experts are missing data - skipping                                                
> save_imatrix: entry '             blk.28.ffn_down_exps.weight' has partial data (89.06%) 28 out of 256 experts are missing data - skipping                                                                                                              save_imatrix: entry '             blk.27.ffn_down_exps.weight' has partial data (92.19%) 20 out of 256 experts are missing data - skipping                                                                                                              save_imatrix: entry '             blk.25.ffn_down_exps.weight' has partial data (87.89%) 31 out of 256 experts are missing data - skipping                                                
> save_imatrix: entry '             blk.24.ffn_down_exps.weight' has partial data (88.67%) 29 out of 256 experts are missing d
> ata - skipping                                                                                                              save_imatrix: entry '             blk.23.ffn_down_exps.weight' has partial data (92.58%) 19 out of 256 experts are missing data - skipping                                                                                                              save_imatrix: entry '             blk.26.ffn_down_exps.weight' has partial data (90.62%) 24 out of 256 experts are missing data - skipping                                                
> save_imatrix: entry '             blk.19.ffn_down_exps.weight' has partial data (94.53%) 14 out of 256 experts are missing data - skipping                                                                                                              save_imatrix: entry '             blk.18.ffn_down_exps.weight' has partial data (92.97%) 18 out of 256 experts are missing d
> ata - skipping                                                
> save_imatrix: entry '             blk.17.ffn_down_exps.weight' has partial data (90.62%) 24 out of 256 experts are missing data - skipping                                                
> save_imatrix: entry '              blk.6.ffn_down_exps.weight' has partial data (97.66%) 6 out of 256 experts are missing data Storing **but be aware**                                   
> save_imatrix: entry '              blk.4.ffn_down_exps.weight' has partial data (97.66%) 6 out of 256 experts are missing data Storing **but be aware**             
> ```
> 
> Is that okay?

> 👤 **ubergarm** replied on **2025-10-22** at **01:49:18**
> 
> @magikRUKKOLA 
> 
> Yes, it is quite common for sparse MoEs to be missing data in the early imatrix chunks for the routed experts. It should clear up by chunk 50 probably... I put my full run logs in the closed PR for Ling-1T here somewhere psure if you wanted to look at those.
> 
> You could use a longer imatrix corpus which may activate more of the routed experts, but I don't worry too terribly much about it. For example qwen3-30b-a3b folks had a lot of issues with this and were having problems making the very small quant types, but you'll likely be fine.

> 👤 **magikRUKKOLA** replied on **2025-10-22** at **02:59:18**
> 
> @ubergarm 
> 
> something along the lines?
> 
> *edited
> 
> File: /opt/cooking/make-quant.sh
> ```bash
> #!/usr/bin/env bash
> 
> #cat /opt/inclusionAI/Ling-flash-2.0-GGUF/Q2_K/Ling-flash-2.0-Q2_K.gguf.dump.mini | sed 's/blk.//' | sed -E 's/\| [0-9\.]+/ /g' | tr '|' ' ' | awk '{print $2, $1}' | sort | uniq -c        
> #     32 attn_k_norm.weight F32
> #     32 attn_norm.weight F32
> #     32 attn_output.weight Q3_K
> #     32 attn_qkv.weight Q2_K  
> #     32 attn_q_norm.weight F32
> #     31 exp_probs_b.bias F32  
> #     31 ffn_down_exps.weight Q3_K
> #     31 ffn_down_shexp.weight Q3_K            
> #      1 ffn_down.weight Q3_K  
> #     31 ffn_gate_exps.weight Q2_K
> #     31 ffn_gate_inp.weight F32         
> #     31 ffn_gate_shexp.weight Q2_K       
> #      1 ffn_gate.weight Q2_K  
> #     32 ffn_norm.weight F32   
> #     31 ffn_up_exps.weight Q2_K         
> #     31 ffn_up_shexp.weight Q2_K         
> #      1 ffn_up.weight Q2_K    
> #      1 output_norm.weight F32
> #      1 output.weight Q6_K    
> #      1 token_embd.weight Q2_K
> 
> custom="
> # Convert ALL Q2_K weights to IQ2_KL (token_embd + FFN/attention gates)
> (token_embd|.*ffn_gate_(exps|shexp)|.*attn_qkv|.*ffn_up_(exps|shexp)|.*ffn_up)\.weight=iq2_kl
> 
> # Convert Q3_K weights to IQ3_KS (output projection + FFN down exps/shexp)
> (output|.*ffn_down_(exps|shexp)|blk\.[0-9]+\.attn_output)\.weight=iq3_ks
> 
> # Keep these as-is (F32 norms/biases or special Q6_K output)
> .*\.weight=f32
> output\.weight=iq6_k
> "
> 
> custom=$(
>   echo "$custom" | grep -v '^#' | \
>   sed -Ez 's:\n+:,:g;s:,$::;s:^,::'
> )
> 
> ./ik_llama.cpp/build/bin/llama-quantize \
>     --imatrix /opt/cooking/ubergarm/Ling-flash-2.0/Ling-flash-256x3.4B-2.0-BF16-imatrix.dat \
>     --custom-q "$custom" \
>     /opt/cooking/ubergarm/Ling-flash-2.0/Ling-flash-256x3.4B-2.0-BF16-00001-of-00005.gguf \
>     ./ubergarm/Ling-flash-2.0/IQ2_KL/Ling-flash-256x3.4B-2.0-IQ2_KL.gguf \
>     IQ2_KL \
>     $(($(nproc) / 2))
> ```

> 👤 **ubergarm** replied on **2025-10-22** at **03:09:07**
> 
> @magikRUKKOLA  
> 
> You're making progress! I'd recommend looking at the recipes i publish on all my model cards especially: https://huggingface.co/ubergarm/Ling-1T-GGUF#smol-iq2_ks-264984-gib-2277-bpw
> 
> Open the "secret recipe" details by clicking on it and you can probably just copy paste the `custom="` section and change the iq2_ks i used to the iq2_kl which seems like you want to use.
> 
> you def want token_embd at at least iq4_k and the output at iq6_k and the typical strat with MoEs is jack up the attn/shexp/first N dense layers and crunch down the routed experts. if you look at the relative size the routed experts tend to make up over 95% of the total size of these big MoEs with the other layers very small and fitting on a single GPU often.
> 
> ignore any vectors as they are always f32 no need to specify them

> 👤 **magikRUKKOLA** replied on **2025-10-22** at **03:35:23**
> 
> @ubergarm 
> > I'd recommend looking at the recipes i publish on all my model cards especially: https://huggingface.co/ubergarm/Ling-1T-GGUF#smol-iq2_ks-264984-gib-2277-bpw
> 
> Lol!  I forgot that these models have the same architecture so that the recipes are compatible.  I instead simply made a dump of the official Q2_K quant and replaced the quants by their I-type quant equivalents. :D

> 👤 **magikRUKKOLA** replied on **2025-10-22** at **04:25:08**
> 
> @ubergarm 
> 
> lol that seemed to be working.
> 
> original Q2_K quant.
> 
> ```
> Final estimate: PPL = 8.6021 +/- 0.06947
> 
> llama_print_timings:        load time =    2593.69 ms
> llama_print_timings:      sample time =       0.00 ms /     1 runs   (    0.00 ms per token,      inf tokens per second)
> llama_print_timings: prompt eval time =  106220.83 ms / 304128 tokens (    0.35 ms per token,  2863.17 tokens per second)
> llama_print_timings:        eval time =       0.00 ms /     1 runs   (    0.00 ms per token,      inf tokens per second)
> llama_print_timings:       total time =  112702.61 ms / 304129 tokens
> ```
> 
> upgraded to I-type quants one:
> 
> ```
> Final estimate: PPL = 6.4181 +/- 0.04625
> 
> llama_print_timings:        load time =    2423.03 ms
> llama_print_timings:      sample time =       0.00 ms /     1 runs   (    0.00 ms per token,      inf tokens per second)
> llama_print_timings: prompt eval time =   87525.31 ms / 304128 tokens (    0.29 ms per token,  3474.74 tokens per second)
> llama_print_timings:        eval time =       0.00 ms /     1 runs   (    0.00 ms per token,      inf tokens per second)
> llama_print_timings:       total time =   94004.58 ms / 304129 tokens
> ```
> 
> command to test ppl:
> ```
> /opt/ik_llama.cpp/ik_llama.cpp/build/bin/llama-perplexity \
>     -f /opt/ik_llama.cpp/wiki.test.raw \
>     --ctx-size $((512)) \
>     --model /opt/cooking/ubergarm/Ling-flash-2.0/IQ2_KL/Ling-flash-256x3.4B-2.0-IQ2_KL.gguf \
>     --alias ubergarm/Ling-flash-256x3.4B-2.0-IQ2_KL \
>     -b $((8 * 512)) -ub $((8 * 512)) \
>     --mlock \
>     --temp 0.7 --top-k 0 --top-p 0.95 --min-p 0.1 --repeat-penalty 1.1 \
>     -ctk f16 -ctv f16 \
>     -fa \
>     -fmoe \
>     -ger \
>     -amb 512 \
>     --main-gpu 1 \
>     -no-ooae \
>     --n-gpu-layers 99 \
>     --threads $(grep ^cpu\\scores /proc/cpuinfo | uniq | awk '{print $4}' | xargs -I{} echo "{}-0" | bc) \
>     --host 0.0.0.0 \
>     --port 8080 \
>     --log-enable \
>     --logdir /var/log/ \
>     --jinja \
>     --special \
>     --verbosity 2 \
>     --verbose-prompt \
>     --reasoning-format auto \
>     --prompt-cache "$HOME/.cache/ik_llama.cpp/prompt-cache.bin" --prompt-cache-all \
>     --slot-save-path "$HOME/.cache/ik_llama.cpp/slot.bin" \
>     --lookup-cache-dynamic "$HOME/.cache/ik_llama.cpp/slot.bin" \
>     --keep -1 \
>     --slot-prompt-similarity 0.35 \
>     --metrics
> 
> ```
> 
> 
> bench:
> 
> 
> original quant Q2_K:
> 
> ```
> ggml_cuda_init: GGML_CUDA_FORCE_MMQ:    no
> ggml_cuda_init: GGML_CUDA_FORCE_CUBLAS: no
> ggml_cuda_init: found 2 CUDA devices:
>   Device 0: NVIDIA GeForce RTX 3090, compute capability 8.6, VMM: yes
>   Device 1: NVIDIA GeForce RTX 3090, compute capability 8.6, VMM: yes
> llm_load_tensors: ggml ctx size =    0.57 MiB
> llm_load_tensors: offloading 32 repeating layers to GPU
> llm_load_tensors: offloading non-repeating layers to GPU
> llm_load_tensors: offloaded 33/33 layers to GPU
> llm_load_tensors:        CPU buffer size =   201.47 MiB
> llm_load_tensors:      CUDA0 buffer size = 18193.59 MiB
> llm_load_tensors:      CUDA1 buffer size = 17510.90 MiB
> ............................................................................................
> llama_new_context_with_model: n_ctx      = 32768
> llama_new_context_with_model: n_batch    = 4096
> llama_new_context_with_model: n_ubatch   = 4096
> llama_new_context_with_model: flash_attn = 1
> llama_new_context_with_model: mla_attn   = 0
> llama_new_context_with_model: attn_max_b = 512
> llama_new_context_with_model: fused_moe  = 1
> llama_new_context_with_model: grouped er = 1
> llama_new_context_with_model: fused_up_gate = 1
> llama_new_context_with_model: ser        = -1, 0
> llama_new_context_with_model: freq_base  = 600000.0
> llama_new_context_with_model: freq_scale = 1
> llama_kv_cache_init:      CUDA0 KV buffer size =   578.01 MiB
> llama_kv_cache_init:      CUDA1 KV buffer size =   510.01 MiB
> llama_new_context_with_model: KV self size  = 1088.00 MiB, K (q8_0):  544.00 MiB, V (q8_0):  544.00 MiB
> llama_new_context_with_model:  CUDA_Host  output buffer size =     0.60 MiB
> llama_new_context_with_model: pipeline parallelism enabled (n_copies=1)
> llama_new_context_with_model:      CUDA0 compute buffer size =  1032.14 MiB
> llama_new_context_with_model:      CUDA1 compute buffer size =  2520.00 MiB
> llama_new_context_with_model:  CUDA_Host compute buffer size =   320.05 MiB
> llama_new_context_with_model: graph nodes  = 1362
> llama_new_context_with_model: graph splits = 3
> 
> main: n_kv_max = 32768, n_batch = 4096, n_ubatch = 4096, flash_attn = 1, n_gpu_layers = 99, n_threads = 12, n_threads_batch = 12
> 
> |    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
> |-------|--------|--------|----------|----------|----------|----------|
> |  4096 |   1024 |      0 |    1.432 |  2860.56 |    7.956 |   128.70 |
> |  4096 |   1024 |   4096 |    1.570 |  2608.81 |    8.680 |   117.97 |
> |  4096 |   1024 |   8192 |    1.711 |  2394.01 |    9.605 |   106.61 |
> |  4096 |   1024 |  12288 |    1.855 |  2208.31 |   10.454 |    97.95 |
> |  4096 |   1024 |  16384 |    1.997 |  2051.40 |   11.277 |    90.80 |
> |  4096 |   1024 |  20480 |    2.139 |  1915.29 |   12.048 |    84.99 |
> |  4096 |   1024 |  24576 |    2.281 |  1795.98 |   12.771 |    80.18 |
> |  4096 |   1024 |  28672 |    2.419 |  1693.03 |   13.551 |    75.57 |
> ```
> 
> 
> IQ2_KL:
> ```
>   Device 0: NVIDIA GeForce RTX 3090, compute capability 8.6, VMM: yes                                  
>   Device 1: NVIDIA GeForce RTX 3090, compute capability 8.6, VMM: yes                         
> llm_load_tensors: ggml ctx size =    0.57 MiB                                                                               
> llm_load_tensors: offloading 32 repeating layers to GPU                                                                     
> llm_load_tensors: offloading non-repeating layers to GPU                                                                    
> llm_load_tensors: offloaded 33/33 layers to GPU                                                                             
> llm_load_tensors:        CPU buffer size =   206.57 MiB                                                                     
> llm_load_tensors:      CUDA0 buffer size = 18240.25 MiB                                                                     
> llm_load_tensors:      CUDA1 buffer size = 17051.52 MiB                                                                     
> ...............................................................................................
> llama_new_context_with_model: n_ctx      = 32768                                                                            
> llama_new_context_with_model: n_batch    = 4096                                                                             
> llama_new_context_with_model: n_ubatch   = 4096                                                                             
> llama_new_context_with_model: flash_attn = 1                                                                                
> llama_new_context_with_model: mla_attn   = 0                                                                                
> llama_new_context_with_model: attn_max_b = 512                                                                              
> llama_new_context_with_model: fused_moe  = 1                                                                                
> llama_new_context_with_model: grouped er = 1                                                                                
> llama_new_context_with_model: fused_up_gate = 1                                                                             
> llama_new_context_with_model: ser        = -1, 0                                                                            
> llama_new_context_with_model: freq_base  = 600000.0                                                                         
> llama_new_context_with_model: freq_scale = 1                                                                                
> llama_kv_cache_init:      CUDA0 KV buffer size =   578.01 MiB                                                               
> llama_kv_cache_init:      CUDA1 KV buffer size =   510.01 MiB                                                               
> llama_new_context_with_model: KV self size  = 1088.00 MiB, K (q8_0):  544.00 MiB, V (q8_0):  544.00 MiB                     
> llama_new_context_with_model:  CUDA_Host  output buffer size =     0.60 MiB                                                 
> llama_new_context_with_model: pipeline parallelism enabled (n_copies=1)                                                     
> llama_new_context_with_model:      CUDA0 compute buffer size =  1032.14 MiB                                                 
> llama_new_context_with_model:      CUDA1 compute buffer size =  2520.00 MiB
> llama_new_context_with_model:  CUDA_Host compute buffer size =   320.05 MiB                                                 
> llama_new_context_with_model: graph nodes  = 1364             
> llama_new_context_with_model: graph splits = 3                                                                                                                                                                                                          
> main: n_kv_max = 32768, n_batch = 4096, n_ubatch = 4096, flash_attn = 1, n_gpu_layers = 99, n_threads = 12, n_threads_batch 
> = 12                                                                                                                        
>                                                                                                                             
> |    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |                                                     
> |-------|--------|--------|----------|----------|----------|----------|                                                     
> |  4096 |   1024 |      0 |    1.127 |  3634.70 |    8.485 |   120.69 |                                                     
> |  4096 |   1024 |   4096 |    1.261 |  3248.24 |    9.211 |   111.18 |                                                     
> |  4096 |   1024 |   8192 |    1.393 |  2939.54 |   10.107 |   101.32 |                                                     
> |  4096 |   1024 |  12288 |    1.527 |  2682.36 |   10.975 |    93.31 |                                                     
> |  4096 |   1024 |  16384 |    1.675 |  2444.73 |   11.795 |    86.82 |                                                     
> |  4096 |   1024 |  20480 |    1.814 |  2258.33 |   12.574 |    81.44 |                                                     
> |  4096 |   1024 |  24576 |    1.958 |  2092.17 |   13.283 |    77.09 |                                                     
> |  4096 |   1024 |  28672 |    2.092 |  1957.67 |   14.050 |    72.88 |
> ```
> 
> Q2_K:
> 
> ```                                 
> llm_load_print_meta: model size       = 35.064 GiB (2.927 BPW)                                                              
> llm_load_print_meta: repeating layers = 34.376 GiB (2.906 BPW, 101.602 B parameters)     
> ```
> 
> IQ2_KL:
> 
> ```
> llm_load_print_meta: model size       = 34.666 GiB (2.894 BPW)  
> llm_load_print_meta: repeating layers = 34.225 GiB (2.894 BPW, 101.602 B parameters)
> ```

> 👤 **magikRUKKOLA** replied on **2025-10-22** at **05:04:55**
> 
> @Thireus 
> 
> Hey, bro!  Can you tell how to calculate the ppl_results.csv for a specific model?

> 👤 **magikRUKKOLA** replied on **2025-10-22** at **07:37:21**
> 
> @ubergarm 
> 
> Ling-flash-2.0/IQ1_KT ppl:
> 
> ```
> Final estimate: PPL = 7.6694 +/- 0.05758
> ```
> 
> sweep-bench:
> 
> ```
> main: n_kv_max = 32768, n_batch = 4096, n_ubatch = 4096, flash_attn = 1, n_gpu_layers = 99, n_threads = 64, n_threads_batch = 64
> 
> |    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
> |-------|--------|--------|----------|----------|----------|----------|
> |  4096 |   1024 |      0 |    1.128 |  3630.15 |    8.387 |   122.09 |
> |  4096 |   1024 |   4096 |    1.253 |  3268.66 |    9.130 |   112.15 |
> |  4096 |   1024 |   8192 |    1.390 |  2947.34 |   10.003 |   102.37 |
> |  4096 |   1024 |  12288 |    1.514 |  2705.99 |   10.879 |    94.12 |
> |  4096 |   1024 |  16384 |    1.648 |  2485.63 |   11.722 |    87.36 |
> |  4096 |   1024 |  20480 |    1.798 |  2277.86 |   12.509 |    81.86 |
> |  4096 |   1024 |  24576 |    1.931 |  2121.21 |   13.232 |    77.39 |
> |  4096 |   1024 |  28672 |    2.075 |  1973.97 |   14.004 |    73.12 |
> 
> ```
> 
> sadly, IQ1_KT is not much faster than IQ2_KL.  But much smaller, and still with better ppl than Q2_K.  So that supposed to allow me to increase the batch sizes for faster prefill in spec. decoding.
> 
> Any way to speedup the decode in my situaltion?

> 👤 **Thireus** replied on **2025-10-22** at **08:51:42**
> 
> > @Thireus
> > 
> > Hey, bro! Can you tell how to calculate the ppl_results.csv for a specific model?
> 
> Fastest way:
> 1. Create or obtain the BF16 model, and split it with `--no-tensor-first-split --split-max-tensors 1`
> 2. Create a base recipe, like this: https://github.com/Thireus/GGUF-Tool-Suite/blob/main/models/GLM-4.6/iq6_k.recipe
> 3. Quantize the BF16 model with that base recipe, make sure to use `llama-quantize --keep-split`
> 4. Create a target recipe, like iq1_kt
> 5. Quantize the BF16 model with that target recipe, make sure to use `llama-quantize --keep-split`
> 6. Use download.conf with COPY (so it knows which folder to fetch the base and target recipe shards from)
> 7. Use the benchmark_each_tensor.sh script onto the quantized base recipe (make sure to edit USER_REGEX, BASELINE_QTYPE, PPL_COMMAND_TEMPLATE of the script for that model): `../benchmark_each_tensor.sh --qtypes iq1_kt` (ensure the qtype matches the target recipe).
> 
> My way:
> 1. Produce the BF16 model with `--no-tensor-first-split --split-max-tensors 1`
> 2. Compute or obtain @ubergarm's imatrix, and produce the quants of my base recipe with it with such script (making sure to adapt custom to the model): https://github.com/Thireus/GGUF-Tool-Suite/blob/main/models/GLM-4.6/GLM-4.6-THIREUS-ANY-SPECIAL.sh
> 3. Upload all the things on HuggingFace
> 4. Configure download.conf to point to HuggingFace
> 5. Create a base recipe, like this: https://github.com/Thireus/GGUF-Tool-Suite/blob/main/models/GLM-4.6/iq6_k.recipe
> 6. Use quant_downloader.sh to obtain the shard combination of the base recipe
> 7. Use the benchmark_each_tensor.sh script onto the quantized base recipe (make sure to edit USER_REGEX, BASELINE_QTYPE, PPL_COMMAND_TEMPLATE of the script for that model): `../benchmark_each_tensor.sh --qtypes iq1_kt` (ensure the qtype matches the target recipe).
> 
> Tips:
> - You can use https://github.com/Thireus/GGUF-Tool-Suite/blob/main/gguf_info.py to list the tensors
> - Then, `cut -d: -f 3,5 | sed 's/:dtype//g' | sed 's/\./\\\./g' | sed 's/=/$=/g' | sed 's/^/^/g'` <- to produce the USER_REGEX
> - Then, you can pass it to https://github.com/Thireus/GGUF-Tool-Suite/blob/main/quants_regex_merger.sh to merge the layers. Which is what you'll need for custom, USER_REGEX, and base recipes.

> 👤 **magikRUKKOLA** replied on **2025-10-22** at **14:34:54**
> 
> @ubergarm @Thireus 
> 
>  **[Discussion] Quantization & Speculative Decoding: Ling-flash-2.0-IQ1_KT-PUFFED Performance Report**
> 
> ### **Model Specification**
> **Quantization Scheme**: Hybrid `IQ1_KT` with selective layer targeting  
> **Base Model**: `Ling-flash-256x3.4B-2.0-BF16` → Quantized to `IQ1_KT-PUFFED`  
> **Recipe Highlights**:
> - **Attention + Dense**: `q8_0`
> - **Shared Experts**: `q8_0`
> - **Routed Experts**: `iq1_kt` (primary compression target)
> - **Embeddings/Output**: `q8_0`
> 
> ```bash
> # Key quantization pattern
> blk\..*\.ffn_(down|gate|up)_exps\.weight=iq1_kt  # Only routed MoE params → IQ1_KT
> ```
> 
> ### Methodology
> 
> ```bash
> pre /opt/GGUF-Tool-Suite/GGUF-Tool-Suite/quant_assign.py | mods -m ling32k "explain the code" 
> ```
> 
> ---
> 
> ### **Performance Benchmarks**
> **Hardware**: 64C CPU, split across 3 GPUs (layers partitioned via `--tensor-split 0,1,1`, main GPU = 2)  
> **Context**: `n_batch=4096`, `n_ubatch=4096`, flash attention enabled.
> 
> | PP  | TG   | N_KV    | T_PP (s) | S_PP (t/s) | T_TG (s) | S_TG (t/s) |
> |-----|------|---------|----------|------------|----------|------------|
> | 4096| 1024 | 0       | 1.058    | 3873.16    | 8.203    | 124.83     |
> | ... | ...  | ...     | ...      | ...        | ...      | ...        |
> | 4096| 1024 | 28672   | 2.020    | 2028.17    | 10.516   | 97.38      |
> 
> **Perplexity (wikitext)**:
> ```
> Final estimate: PPL = 7.5208 ± 0.0562
> ```
> → Within expected range for aggressive expert quantization, no severe degradation observed.
> 
> ---
> 
> ### **Speculative Decoding Setup**
> - **Draft Model**: `Ling-flash-2.0-IQ1_KT-PUFFED` (99 GPU layers)
> - **Target Model**: `Ling-1T-smol-IQ4_KSS`
> - **Config**:
>   - `--draft-min 1`, `--draft-max 16`
>   - `-ctkd f16 -ctvd f16`
>   - `--gpu-layers-draft 99`
>   - `--tensor-split 0,1,1`
> 
> **Observed Acceptance Rates (3 runs)**:
> ```
> Run 1: 1123 / 1799 → 62.42%
> Run 2: 905  / 1500 → 60.33%
> Run 3: 947  / 1381 → 68.57%
> → Weighted Avg Acceptance Rate ≈ 63.8%
> ```
> 
> ---
> 
> ### **Throughput Comparison**
> 
> | Mode               | Run 1 (t/s) | Run 2 (t/s) | Run 3 (t/s) | Avg (t/s)   |
> |--------------------|-------------|-------------|-------------|-------------|
> | With Spec. Decoding| 6.785       | 6.834       | 7.468       | **7.029**   |
> | Baseline (vanilla) | 6.267       | 6.224       | 6.269       | **6.268**   |
> 
> → **Speedup**: `(7.029 - 6.268)/6.268 ≈ +12.15%`
> 
> ### **P.S.:**
> 
> ~~The interesting thing is that the decode speed of IQ1_KT-PUFFED quant doesn't degrade too much with the larger context.  This allows to speedup the decode of the target model with large context (22k prefill).~~  Ha.  Its rather the effect of -fmoe and -fa enabled.
> 
> Another cool thing about the spec. decode is that the decode speed is correlated to the acceptance rate and if the task in hand involves alot of copy/pasting from one place of context to the decoding sequence then the decode speed will be higher still.  For example:
> 
> ```
> pre /opt/GGUF-Tool-Suite/GGUF-Tool-Suite/quant_assign.py | mods -m ling32k "add some really thoughtful verbose but concise comments and output the full code"
> ```
> 
> The decode tps in this case will be higher still.  ~~Rough measurements shows up to 9 tps (+40%).~~
> 
> ```bash
> journalctl -u ik_llama.cpp-ling-32k-ctx.service --since '3000 minutes ago' | grep --colour -Po '(?<=draft_n\\":)[0-9]+.*n_accepted\\":[0-9]+' | sed 's/,\\\"draft_n_accepted\\\":/\//' | awk -F/ '{print $2/$1}' | xargs -I{} echo "scale=0;{}*100" | bc | xargs -I{} echo "{}%"
> 
> VERB [              operator()] data stream | tid="139599038885888" timestamp=1761145619 to_send="data: {\"choices\":[],\"created\":1761145619,\"id\":\"chatcmpl-0OnjoYRcjt2m6A8T2AgXYlNr2sWJOZwa\",\"model\":\"Ling-1T-smol-IQ4_KSS-32k-ctx\",\"object\":\"chat.completion.chunk\",\"usage\":{\"completion_tokens\":10257,\"prompt_tokens\":22511,\"total_tokens\":32768},\"timings\":{\"prompt_n\":22499,\"prompt_ms\":198191.68,\"prompt_per_token_ms\":8.8089106182497,\"prompt_per_second\":113.5214152279248,\"predicted_n\":10257,\"predicted_ms\":1244671.413,\"predicted_per_token_ms\":121.34848522959929,\"predicted_per_second\":8.240729153791532,\"draft_n\":9825,\"draft_n_accepted\":7384}}\n\n"
> 
> decode tps: 8.24 tps
> acceptance rate: 75.155200%
> ```
> 
> ### Additional info
> 
> full command to run the inference:
> 
> ```
>                           
> export MALLOC_CONF="background_thread:true,percpu_arena:phycpu,metadata_thp:auto,dirty_decay_ms:10000,muzzy_decay_ms:60000"                                                      
> export LD_PRELOAD=/usr/local/lib/libjemalloc.so                                         
>                                                                       
>                                                                                                                             
> ulimit -n 9999                                                                                                                                             
> ulimit -l unlimited            
>                                
> export CUDA_VISIBLE_DEVICES="0,1,2"                                                                                                           
>                                                               
> # --verbose-prompt                                            
>                                                                                                                                                   
> #cuda-gdb --args \
> /opt/ik_llama.cpp/ik_llama.cpp/build/bin/llama-server \                                 
>     --ctx-size $((32 * 1024)) \                                                         
>     --ctx-size-draft $((32 * 1024)) \                                        
>     --model-draft /opt/ubergarm/Ling-flash-2.0/IQ1_KT-PUFFED/Ling-flash-256x3.4B-2.0-IQ1_KT-PUFFED.gguf \                                                                        
>     --draft-max 16 \   
>     --draft-min 1 \     
>     -ctkd f16 -ctvd f16 \    
>     --gpu-layers-draft 99 \                                                                                                 
>     --model /opt/ubergarm/Ling-1T-GGUF/smol-IQ4-KSS/Ling-1T-smol-IQ4_KSS-00001-of-00011.gguf \                                                             
>     --alias ubergarm/Ling-1T-smol-IQ4_KSS-32k-ctx \                                                                         
>     -b $((8 * 512)) -ub $((8 * 512)) \                                                  
>     --mlock \                                                 
>     --temp 0.7 --top-k 0 --top-p 0.95 --min-p 0.1 --repeat-penalty 1.1 \                                                                                                         
>     -ctk f16 -ctv f16 \                                                                                                     
>     -fa \                             
>     -fmoe \                           
>     -ger \                            
>     -amb 512 \                        
>     --override-tensor exps=CPU \                                                                                                                           
>     -no-ooae \                                                               
>     --n-gpu-layers 99 \                                                                                                                                    
>     --split-mode layer \              
>     --tensor-split 0,1,1 \                                                   
>     --main-gpu 2 \                    
>     --threads $(grep ^cpu\\scores /proc/cpuinfo | uniq | awk '{print $4}' | xargs -I{} echo "{}-0" | bc) \                                                 
>     --host 0.0.0.0 \                        
>     --port 8080 \                           
>     --log-enable \                          
>     --logdir /var/log/ \                    
>     --jinja \                               
>     --special \                             
>     --verbosity 2 \                         
>     --verbose-prompt \                      
>     --reasoning-format auto \               
>     --prompt-cache "$HOME/.cache/ik_llama.cpp/prompt-cache.bin" --prompt-cache-all \                                                                                             
>     --slot-save-path "$HOME/.cache/ik_llama.cpp/slot.bin" \                             
>     --lookup-cache-dynamic "$HOME/.cache/ik_llama.cpp/slot.bin" \                                                                                                                
>     --keep -1 \                             
>     --slot-prompt-similarity 0.35 \                                                     
>     --metrics                               
> ```

> 👤 **magikRUKKOLA** replied on **2025-10-22** at **15:05:13**
> 
> @ubergarm @Thireus 
> 
> The above likely means the following.  **Tensor parallelism for the draft model** is the way to go.  If its possible to boost the speed of the draft model ... BTW how to calculate the machine time taken for the prefill and decode of the draft model during the spec. decoding?  It should be possible to estimate the target tps by specification of the decode tps of the draft model, right?

> 👤 **saood06** replied on **2025-10-23** at **06:00:08**
> 
> >Another cool thing about the spec. decode is that the decode speed is correlated to the acceptance rate
> 
> For this reason lowering temperature (and any other changes to make sampling more deterministic) will also increase acceptance rate. Obviously this only works for some user/workload combinations as not every user/workload likes the output that comes with lowered temperature.
> 
> Also if you'd be willing to run the same three tests with the IQ2_KL, I'd be curious how much acceptance rate (and speed) improves over the IQ1_KT-PUFFED given the PPL differences between the two.

> 👤 **magikRUKKOLA** replied on **2025-10-23** at **15:06:45**
> 
> @saood06 
> 
> > Also if you'd be willing to run the same three tests with the IQ2_KL, I'd be curious how much acceptance rate (and speed) improves over the IQ1_KT-PUFFED given the PPL differences between the two.
> 
> Its pretty hard to run both models with f16 KV cache with only three 24GB GPUs.
> ```
> --ctx-size $((24 * 1024)) \
> --ctx-size-draft $((24 * 1024)) \
> -b $((4 * 512)) -ub $((4 * 512)) \
> -ctk f16 -ctv f16 \
> -ctkd f16 -ctvd f16 \
> --split-mode layer \
> --tensor-split 21,21,500 \
> ```
> 
> GPU1: 23.6GB, GPU2: 22.9GB, GPU3: 23.7GB
> 
> Let me see what the results are.
> 
> [EDIT]:
> 
> ```
> Run 1:  960   / 1559 → 61.58%
> Run 2:  987   / 1523 → 64.81%
> Run 3:  1138 / 1802 → 63.15%
> → Weighted Avg Acceptance Rate ≈ 63.17%
> ```
> 
> Lol not much difference at all.  But the acceptance rates are more consistent, jumping in range from 61% to 65%, while for the PUFFED quant it was from 60% to 69%.
> 
> The decode speed for IQ2_KL:
> 
> ```
> echo "scale=8;(6.547646147942585+7.087404545082322+6.94695118960171)/3"|bc
> 6.86066729
> ```

> 👤 **magikRUKKOLA** replied on **2025-10-23** at **15:42:22**
> 
> @saood06 
> 
> Let me do about 100 runs for each config to make sure.
> 
> in progress ...
> <details>
> <img width="1241" height="1241" alt="nvok" src="https://github.com/user-attachments/assets/95a201da-4f2c-4eec-8447-503eb751ee2c" />
> 
> </details>
> 
> Results for iq2_kl quant:
> 
> ![journal_20251023_233934_data_plot](https://github.com/user-attachments/assets/2bd8ed25-270d-4b1a-a933-5210efe2bab1)
> 
> 
> Results for iq1_kt-PUFFED quant:
> 
> ![journal_20251024_083056_data_plot](https://github.com/user-attachments/assets/cb772c81-a5b3-4f8c-9d09-2f55d95a8ad5)
> 
> Results for iq1_kt quant:
> 
> ![journal_20251024_191118_data_plot](https://github.com/user-attachments/assets/4d05673e-3be5-48d7-8422-d2b6c78fba51)
> 
> 
> Code:
> 
> File: /root/analyze_journal.sh
> ```bash
> #!/bin/bash
> 
> # Function to display usage
> usage() {
>     echo "Usage: $0 <service_name>"
>     echo "  service_name: Name of the systemd service to analyze"
>     echo ""
>     echo "Example: $0 ik_llama.cpp-ling-32k-ctx.service"
>     exit 1
> }
> 
> # Check arguments
> if [ $# -lt 1 ]; then
>     usage
> fi
> 
> SERVICE_NAME="$1"
> 
> # Get service start time
> echo "Getting service start time..."
> START_TIME=$(systemctl show -p ActiveEnterTimestamp "$SERVICE_NAME" | cut -d= -f2)
> 
> if [ -z "$START_TIME" ] || [ "$START_TIME" = "" ]; then
>     echo "Error: Service '$SERVICE_NAME' not found or not running"
>     exit 1
> fi
> 
> echo "Service started at: $START_TIME"
> 
> # Generate timestamp-based prefix
> TIMESTAMP=$(date +%Y%m%d_%H%M%S)
> PREFIX="journal_${TIMESTAMP}"
> 
> echo "Output prefix: $PREFIX"
> echo ""
> 
> # Create journal dump from service start time
> echo "Dumping journal from service start time..."
> journalctl -u "$SERVICE_NAME" --since "$START_TIME" > "${PREFIX}.log"
> 
> if [ ! -s "${PREFIX}.log" ]; then
>     echo "Error: Journal dump is empty or failed to create"
>     exit 1
> fi
> 
> # Extract acceptance rates
> echo "Extracting acceptance rates..."
> grep --colour -Po '(?<=draft_n\\":)[0-9]+.*n_accepted\\":[0-9]+' "${PREFIX}.log" | \
>     sed 's/,\\\"draft_n_accepted\\\":/\//' | \
>     awk -F/ '{print $2/$1}' | \
>     xargs -I{} echo "scale=0;{}*100" | \
>     bc > "${PREFIX}-acceptance.log"
> 
> # Extract speeds
> echo "Extracting speeds..."
> grep -Po '(?<=predicted_per_second\\":)[0-9.]+' "${PREFIX}.log" | \
>     xargs -I{} echo "{}" | \
>     bc > "${PREFIX}-speed.log"
> 
> # Count entries
> ENTRY_COUNT=$(wc -l < "${PREFIX}-speed.log")
> echo "Found $ENTRY_COUNT entries"
> echo ""
> 
> if [ "$ENTRY_COUNT" -eq 0 ]; then
>     echo "Error: No data found in journal"
>     exit 1
> fi
> 
> # Calculate overall acceptance rate
> echo "Calculating overall acceptance rate..."
> ACCEPTANCE_RATE=$(grep --colour -Po '(?<=draft_n\\":)[0-9]+.*n_accepted\\":[0-9]+' "${PREFIX}.log" | \
>     sed 's/,\\\"draft_n_accepted\\\":/\//' | \
>     awk -F/ '{print $2/$1}' | \
>     xargs -I{} echo "scale=0;{}*100" | \
>     bc | \
>     xargs -I{} echo "{}" | \
>     tr '\n' '+' | \
>     rev | \
>     cut -c2- | \
>     rev | \
>     xargs -I{} echo "scale=8;({})/$ENTRY_COUNT" | \
>     bc)
> 
> # Calculate average speed
> echo "Calculating average speed..."
> AVERAGE_SPEED=$(grep -Po '(?<=predicted_per_second\\":)[0-9.]+' "${PREFIX}.log" | \
>     xargs -I{} echo "{}" | \
>     tr '\n' '+' | \
>     rev | \
>     cut -c2- | \
>     rev | \
>     xargs -I{} echo "scale=8;({})/$ENTRY_COUNT" | \
>     bc)
> 
> # Display results
> echo ""
> echo "=== RESULTS ==="
> echo "Acceptance rate (%) for $ENTRY_COUNT runs: $ACCEPTANCE_RATE"
> echo "Average speed (tokens/sec): $AVERAGE_SPEED"
> echo ""
> 
> # Create sorted data files
> echo "Creating sorted data files..."
> paste "${PREFIX}-acceptance.log" "${PREFIX}-speed.log" | sort -k1,1n > "${PREFIX}-sorted-by-acceptance.txt"
> paste "${PREFIX}-acceptance.log" "${PREFIX}-speed.log" | sort -k2,2n > "${PREFIX}-sorted-by-speed.txt"
> 
> ## Display sample of sorted data
> #echo "Sample sorted by acceptance rate (first 10):"
> #head -10 "${PREFIX}-sorted-by-acceptance.txt"
> #echo ""
> #
> #echo "Sample sorted by speed (first 10):"
> #head -10 "${PREFIX}-sorted-by-speed.txt"
> #echo ""
> 
> # Summary file
> cat > "${PREFIX}-summary.txt" << EOF
> Analysis Summary
> ================
> Service: $SERVICE_NAME
> Start time: $START_TIME
> Timestamp: $TIMESTAMP
> Entries analyzed: $ENTRY_COUNT
> 
> Overall acceptance rate (%): $ACCEPTANCE_RATE
> Average speed (tokens/sec): $AVERAGE_SPEED
> 
> Files generated:
> - ${PREFIX}.log              : Raw journal data
> - ${PREFIX}-acceptance.log   : Individual acceptance rates
> - ${PREFIX}-speed.log        : Individual speeds
> - ${PREFIX}-sorted-by-acceptance.txt : Data sorted by acceptance rate
> - ${PREFIX}-sorted-by-speed.txt      : Data sorted by speed
> - ${PREFIX}-summary.txt      : This summary file
> EOF
> 
> echo "Summary written to ${PREFIX}-summary.txt"
> echo "All files created with prefix: $PREFIX"
> 
> # Optional: Call plot script if it exists
> if [ -f "./plot_data.sh" ]; then
>     echo ""
>     echo "Plotting data..."
>     ./plot_data.sh "${PREFIX}"
> fi
> ```
> 
> File: /root/plot_data.sh
> ```bash
> #!/bin/bash
> 
> # Check if prefix argument is provided
> if [ -z "$1" ]; then
>     echo "Usage: $0 <prefix> [title]"
>     echo "Example: $0 mydata 'My Custom Title'"
>     exit 1
> fi
> 
> # Define input files using prefix
> PREFIX="$1"
> ACCEPTANCE_LOG="${PREFIX}-sorted-by-acceptance.txt"
> SPEED_LOG="${PREFIX}-sorted-by-speed.txt"
> 
> # Optional title (second argument)
> TITLE="${2:-Acceptance vs Speed Data Scatter Plot}"
> 
> # Output files using prefix
> COMBINED_DATA="${PREFIX}_combined_data.dat"
> GNUPLOT_SCRIPT="${PREFIX}_plot.gp"
> OUTPUT_SVG="${PREFIX}_data_plot.svg"
> 
> # Check if input files exist
> if [ ! -f "$ACCEPTANCE_LOG" ] || [ ! -f "$SPEED_LOG" ]; then
>     echo "Error: Input files not found!"
>     echo "Looking for:"
>     echo "  $ACCEPTANCE_LOG"
>     echo "  $SPEED_LOG"
>     exit 1
> fi
> 
> # Combine the data (assuming single column files)
> paste "$ACCEPTANCE_LOG" "$SPEED_LOG" > "$COMBINED_DATA"
> 
> # Calculate mode function - with proper rounding strategy
> calculate_mode() {
>     local data="$1"
>     local total_count=$(echo "$data" | wc -l)
>     local target_count=10
> 
>     # Try different rounding levels to find meaningful modes
>     local decimals=8
>     local rounded_data=""
>     local best_mode="N/A"
>     local best_count=1
> 
>     while [ $decimals -ge 0 ]; do
>         # Round to specified decimal places
>         if [ $decimals -eq 0 ]; then
>             rounded_data=$(echo "$data" | awk '{printf "%.0f\n", $1}')
>         else
>             rounded_data=$(echo "$data" | awk -v d="$decimals" '{printf "%.*f\n", d, $1}')
>         fi
> 
>         # Find the most frequent value at this precision
>         local mode_result=$(echo "$rounded_data" | sort | uniq -c | sort -nr | head -n 1)
>         local max_count=$(echo "$mode_result" | awk '{print $1}')
> 
>         # If we found a value that appears more than once and more than previous best
>         if [ "$max_count" -gt "$best_count" ]; then
>             best_count=$max_count
>             if [ $decimals -eq 0 ]; then
>                 best_mode=$(echo "$mode_result" | awk '{printf "%.0f", $2}')
>             else
>                 best_mode=$(echo "$mode_result" | awk -v d="$decimals" '{printf "%.*f", d, $2}')
>             fi
>         fi
> 
>         unique_count=$(echo "$rounded_data" | sort | uniq | wc -l)
>         if [ $unique_count -le $target_count ]; then
>             break
>         fi
>         decimals=$((decimals - 1))
>     done
> 
>     echo "$best_mode"
> }
> 
> # Calculate comprehensive statistics function
> calculate_stats() {
>     local data="$1"
>     local total_points=$(echo "$data" | wc -l)
>     
>     # Mean
>     local mean=$(echo "$data" | awk '{sum += $1; count++} END {if(count>0) print sum/count; else print 0}')
>     
>     # Sort data for median and other calculations
>     local sorted_data=$(echo "$data" | sort -n)
>     
>     # Median
>     local median=$(echo "$sorted_data" | awk '{
>         data[NR] = $1
>     } 
>     END {
>         n = NR
>         if (n == 0) print 0
>         else if (n % 2 == 1) print data[(n+1)/2]
>         else print (data[n/2] + data[n/2+1]) / 2
>     }')
>     
>     # Mode (most frequent value)
>     local mode=$(calculate_mode "$data")
>     
>     # Min and Max
>     local min=$(echo "$sorted_data" | head -n 1)
>     local max=$(echo "$sorted_data" | tail -n 1)
>     
>     # Standard deviation
>     local std_dev=$(echo "$data" | awk -v mean="$mean" '{
>         sum += ($1 - mean)^2
>         count++
>     } 
>     END {
>         if (count > 1) print sqrt(sum/(count-1))
>         else print 0
>     }')
>     
>     # Output all statistics
>     echo "$total_points:$mean:$median:$mode:$min:$max:$std_dev"
> }
> 
> # Extract data columns
> ACCEPTANCE_DATA=$(awk '{print $1}' "$COMBINED_DATA")
> SPEED_DATA=$(awk '{print $2}' "$COMBINED_DATA")
> 
> # Calculate statistics for both columns
> ACCEPTANCE_STATS=$(calculate_stats "$ACCEPTANCE_DATA")
> SPEED_STATS=$(calculate_stats "$SPEED_DATA")
> 
> # Parse statistics
> TOTAL_POINTS=$(echo "$ACCEPTANCE_STATS" | cut -d: -f1)
> ACCEPTANCE_MEAN=$(echo "$ACCEPTANCE_STATS" | cut -d: -f2)
> ACCEPTANCE_MEDIAN=$(echo "$ACCEPTANCE_STATS" | cut -d: -f3)
> ACCEPTANCE_MODE=$(echo "$ACCEPTANCE_STATS" | cut -d: -f4)
> ACCEPTANCE_MIN=$(echo "$ACCEPTANCE_STATS" | cut -d: -f5)
> ACCEPTANCE_MAX=$(echo "$ACCEPTANCE_STATS" | cut -d: -f6)
> ACCEPTANCE_STD=$(echo "$ACCEPTANCE_STATS" | cut -d: -f7)
> 
> SPEED_MEAN=$(echo "$SPEED_STATS" | cut -d: -f2)
> SPEED_MEDIAN=$(echo "$SPEED_STATS" | cut -d: -f3)
> SPEED_MODE=$(echo "$SPEED_STATS" | cut -d: -f4)
> SPEED_MIN=$(echo "$SPEED_STATS" | cut -d: -f5)
> SPEED_MAX=$(echo "$SPEED_STATS" | cut -d: -f6)
> SPEED_STD=$(echo "$SPEED_STATS" | cut -d: -f7)
> 
> # Format statistics for display
> ACCEPTANCE_MEAN_FMT=$(printf "%.4f" "$ACCEPTANCE_MEAN" 2>/dev/null || echo "0.0000")
> ACCEPTANCE_MEDIAN_FMT=$(printf "%.4f" "$ACCEPTANCE_MEDIAN" 2>/dev/null || echo "0.0000")
> if [ "$ACCEPTANCE_MODE" = "N/A" ]; then
>     ACCEPTANCE_MODE_FMT="N/A"
> else
>     ACCEPTANCE_MODE_FMT=$(printf "%.4f" "$ACCEPTANCE_MODE" 2>/dev/null || "N/A")
> fi
> 
> ACCEPTANCE_MIN_FMT=$(printf "%.4f" "$ACCEPTANCE_MIN" 2>/dev/null || echo "0.0000")
> ACCEPTANCE_MAX_FMT=$(printf "%.4f" "$ACCEPTANCE_MAX" 2>/dev/null || echo "0.0000")
> ACCEPTANCE_STD_FMT=$(printf "%.4f" "$ACCEPTANCE_STD" 2>/dev/null || echo "0.0000")
> 
> SPEED_MEAN_FMT=$(printf "%.2f" "$SPEED_MEAN" 2>/dev/null || echo "0.00")
> SPEED_MEDIAN_FMT=$(printf "%.2f" "$SPEED_MEDIAN" 2>/dev/null || echo "0.00")
> if [ "$SPEED_MODE" = "N/A" ]; then
>     SPEED_MODE_FMT="N/A"
> else
>     SPEED_MODE_FMT=$(printf "%.2f" "$SPEED_MODE" 2>/dev/null || "N/A")
> fi
> 
> SPEED_MIN_FMT=$(printf "%.2f" "$SPEED_MIN" 2>/dev/null || echo "0.00")
> SPEED_MAX_FMT=$(printf "%.2f" "$SPEED_MAX" 2>/dev/null || echo "0.00")
> SPEED_STD_FMT=$(printf "%.2f" "$SPEED_STD" 2>/dev/null || echo "0.00")
> 
> # Create gnuplot script with enhanced features
> cat <<EOF > "$GNUPLOT_SCRIPT"
> set terminal svg size 1000,600 enhanced background rgb "white"
> set output '$OUTPUT_SVG'
> 
> set title "$TITLE"
> set xlabel "Acceptance Value"
> set ylabel "Speed Value"
> 
> set grid y
> #set key on outside bottom center horizontal box
> set key off
> 
> # Add comprehensive statistics as text labels in footer area
> set label 1 "N=$TOTAL_POINTS" at screen 0.5, 0.02 center font ",10"
> set label 2 "Acceptance: μ=$ACCEPTANCE_MEAN_FMT, Md=$ACCEPTANCE_MEDIAN_FMT, Mo=$ACCEPTANCE_MODE_FMT, σ=$ACCEPTANCE_STD_FMT, Range=[$ACCEPTANCE_MIN_FMT,$ACCEPTANCE_MAX_FMT]" at screen 0.5, 0.05 center font ",10"
> set label 3 "Speed: μ=$SPEED_MEAN_FMT, Md=$SPEED_MEDIAN_FMT, Mo=$SPEED_MODE_FMT, σ=$SPEED_STD_FMT, Range=[$SPEED_MIN_FMT,$SPEED_MAX_FMT]" at screen 0.5, 0.08 center font ",10"
> 
> # Adjust margins to make room for footer text
> set bmargin at screen 0.2
> 
> # Plot with legend below the plot
> plot "$COMBINED_DATA" using 1:2 with points pt 7 ps 2 lc rgb "#BB000080" title "Data Points"
> EOF
> 
> # Run gnuplot
> gnuplot "$GNUPLOT_SCRIPT"
> 
> echo "SVG plot saved to $OUTPUT_SVG"
> echo ""
> echo "Statistics Summary:"
> echo "=================="
> echo "Total Data Points: $TOTAL_POINTS"
> echo ""
> echo "Acceptance Statistics:"
> echo "  Mean:    $ACCEPTANCE_MEAN_FMT"
> echo "  Median:  $ACCEPTANCE_MEDIAN_FMT"
> echo "  Mode:    $ACCEPTANCE_MODE_FMT"
> echo "  Std Dev: $ACCEPTANCE_STD_FMT"
> echo "  Range:   [$ACCEPTANCE_MIN_FMT, $ACCEPTANCE_MAX_FMT]"
> echo ""
> echo "Speed Statistics:"
> echo "  Mean:    $SPEED_MEAN_FMT"
> echo "  Median:  $SPEED_MEDIAN_FMT"
> echo "  Mode:    $SPEED_MODE_FMT"
> echo "  Std Dev: $SPEED_STD_FMT"
> echo "  Range:   [$SPEED_MIN_FMT, $SPEED_MAX_FMT]"
> ```

> 👤 **magikRUKKOLA** replied on **2025-10-24** at **00:04:07**
> 
> @saood06 
> 
> The data for 100 runs for IQ2_KL is ready above.
> 
> I was thinking the following -- is it possible to extract the data from LLM such as that ...
> For example we got the inference result for a certain data. Then we dumping the statistics data such that which experts were activated, and which sections/portions of the LLM carried the most influence upon the inference result.  Then we are using this data to do a certain optimisations such as replacing the most frequently used or the most influential sections the most precision.  As far as I am understanding the @Thireus framework works quite differently, the influence of each tensor is calculated by incrementally dropping its precision and testing how it influenced the PPL outcome, right?
> 
> For example if we offloaded the LLM to the RAM, then it should be collect the data regarding the memory pages that CPU has prefetched into the cache, right?  If so, then one can see which pages are responsible for a certain LLM tensors (roughly).

> 👤 **ubergarm** replied on **2025-10-24** at **05:04:24**
> 
> > For example we got the inference result for a certain data. Then we dumping the statistics data such that which experts were activated, and which sections/portions of the LLM carried the most influence upon the inference result. Then we are using this data to do a certain optimisations such as replacing the most frequently used or the most influential sections the most precision
> 
> This sounds kinda like imatrix itself to me which running inference on "certain data" (the imatrix input corpus) it saves out a set of "statistics data" importances. 
> 
> Then we use this imatrix.dat data to `optimize weighted RMSE minimization when quantizing the tensor` with llama-quantize.
> 
> Reference PR: https://github.com/ggml-org/llama.cpp/pull/4861
> 
> >  If so, then one can see which pages are responsible for a certain LLM tensors (roughly).
> 
> This sounds a bit like the new REAP https://github.com/CerebrasResearch/reap paper/gitrepo
> 
> > selects experts to prune which contribute minimally to the layer output by considering both the router gate-values and average activation norm of the experts;
> 
> The claim:
> 
> > our method achieves near-lossless compression on code generation and tool-calling tasks with Qwen3-Coder-480B and Kimi-K2, even after pruning 50% of experts. 
> 
> some folks on beaver ai discord got the REAP code running on one of the big GLM models, but might be dealing with some issue with the ntp layers and the gguf metadata lining up right. but seems promising and worth testing!

> 👤 **magikRUKKOLA** replied on **2025-10-24** at **06:29:12**
> 
> @ubergarm 
> > our method achieves near-lossless compression on code generation and tool-calling tasks with Qwen3-Coder-480B and Kimi-K2, even after pruning 50% of experts.
> 
> bartowski/cerebras_Qwen3-Coder-REAP-25B-A3B-GGUF
> IQ2_XXS: Final estimate: PPL = 16.9463 +/- 0.13748 
> Q2_K_L: Final estimate: PPL = 14.2139 +/- 0.11650

> 👤 **ikawrakow** replied on **2025-10-24** at **06:38:49**
> 
> What are the PPL values for the corresponding not REAP'ed Qwen3-Coder-30B-A3B quants?

> 👤 **magikRUKKOLA** replied on **2025-10-24** at **07:29:36**
> 
> @ikawrakow 
> 
> Qwen3-Coder-30B-A3B-Instruct-UD-IQ2_XXS.gguf
> ```
> Final estimate: PPL = 10.3252 +/- 0.08182
> ```
> 
> bartowski/cerebras_Qwen3-Coder-REAP-25B-A3B-GGUF
> ```
> IQ2_XXS: Final estimate: PPL = 16.9463 +/- 0.13748
> ```
> 
> Verdict: **absolutely unusable**

> 👤 **ikawrakow** replied on **2025-10-24** at **07:34:13**
> 
> Haha, yes. Claiming "nearly lossless" is very common on arXiv.

> 👤 **magikRUKKOLA** replied on **2025-10-24** at **08:50:18**
> 
> @saood06 
> 
> Here are the final results after 100 runs of spec. decoding for each quant:
> 
> ```
> IQ1_KT-PUFFED;  acceptance: 62.49%, speed: 6.78 tps
> IQ2_KL;  acceptance: 65.78%, speed: 6.9 tps
> 
> PPL difference: 1.1027
> 
> IQ1_KT-PUFFED quant decode speed at about 20k ctx: 102.61 tps
> IQ2_KL quant decode speed at about 20k ctx: 81.44 tps
> ```
> 
> The detailed graphs are above.  https://github.com/ikawrakow/ik_llama.cpp/discussions/839#discussioncomment-14763959

> 👤 **magikRUKKOLA** replied on **2025-10-24** at **16:08:43**
> 
> @saood06 
> 
> ~~Uh oh.  I am getting a problem unfortunately.  The prelim results for the IQ1_KT shows that the acceptance rate is about the same as in PUFFED version but the speed influence of IQ1_KT (as a draft for spec. decoding) seems to be more positive (its faster by 0.1 tps on average right now).  But how?  Why?  I ran the tests (bench-sweep) for both models (the PUFFED version on a 3945wx though -- should not be matter, right, because the LLM in question is at GPU, right?).~~
> 
> <details>
> File: /opt/ubergarm/Ling-flash-2.0/IQ1_KT/bench.log
> ```
> 
> main: n_kv_max = 32768, n_batch = 4096, n_ubatch = 4096, flash_attn = 1, n_gpu_layers = 99, n_threads = 64, n_threads_batch = 64
> 
> |    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
> |-------|--------|--------|----------|----------|----------|----------|
> |  4096 |   1024 |      0 |    1.128 |  3630.15 |    8.387 |   122.09 |
> |  4096 |   1024 |   4096 |    1.253 |  3268.66 |    9.130 |   112.15 |
> |  4096 |   1024 |   8192 |    1.390 |  2947.34 |   10.003 |   102.37 |
> |  4096 |   1024 |  12288 |    1.514 |  2705.99 |   10.879 |    94.12 |
> |  4096 |   1024 |  16384 |    1.648 |  2485.63 |   11.722 |    87.36 |
> |  4096 |   1024 |  20480 |    1.798 |  2277.86 |   12.509 |    81.86 |
> |  4096 |   1024 |  24576 |    1.931 |  2121.21 |   13.232 |    77.39 |
> |  4096 |   1024 |  28672 |    2.075 |  1973.97 |   14.004 |    73.12 |
> ```
> 
> File: /opt/ubergarm/Ling-flash-2.0/IQ1_KT-PUFFED/bench.log
> ```
> 
> main: n_kv_max = 32768, n_batch = 4096, n_ubatch = 4096, flash_attn = 1, n_gpu_layers = 99, n_threads = 12, n_threads_batch = 12
> 
> |    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
> |-------|--------|--------|----------|----------|----------|----------|
> |  4096 |   1024 |      0 |    1.058 |  3873.16 |    8.203 |   124.83 |
> |  4096 |   1024 |   4096 |    1.183 |  3462.45 |    8.580 |   119.35 |
> |  4096 |   1024 |   8192 |    1.314 |  3116.23 |    8.933 |   114.63 |
> |  4096 |   1024 |  12288 |    1.462 |  2800.70 |    9.339 |   109.65 |
> |  4096 |   1024 |  16384 |    1.604 |  2553.19 |    9.661 |   105.99 |
> |  4096 |   1024 |  20480 |    1.736 |  2358.96 |    9.980 |   102.61 |
> |  4096 |   1024 |  24576 |    1.875 |  2184.94 |   10.219 |   100.21 |
> |  4096 |   1024 |  28672 |    2.020 |  2028.17 |   10.516 |    97.38 |
> ```
> 
> The sweep-bench clearly shows an advantage of PUFFED version vs a regular one.  At 20k ctx its 103 tps vs 82 tps.  Its 20% faster in decode!    The prefill is faster too: 2359 tps, 2278 tps.  The PPL of PUFFED version is slightly better too.
> 
> But why the hell its performing worse in decode?
> 
> Did I messed up the testing?  The enabled flash attention does have any influence when -ot with CPU tensor override ( --override-tensor exps=CPU ) is **not** specified?  (because I noticed that FA is not enabled by the draft model in the ik_llama.cpp log).
> </details>

> 👤 **ikawrakow** replied on **2025-10-24** at **16:17:02**
> 
> What is the difference between PUFFED and not PUFFED?

> 👤 **magikRUKKOLA** replied on **2025-10-24** at **19:10:33**
> 
> @ikawrakow 
> 
> diff
> ```diff
>  diff -u make-quant-IQ1_KT.sh  ./make-quant-IQ1_KT-PUFFED.sh 
>                                                                                   
> --- make-quant-IQ1_KT.sh        2025-10-24 19:09:11.885386107 +0000
> +++ ./make-quant-IQ1_KT-PUFFED.sh       2025-10-22 13:59:57.433981023 +0000
> @@ -5,24 +5,24 @@
>  # 80 Repeating Layers [0-79]
>  
>  # Attention
> -blk\..*\.attn_qkv.*=iq6_k
> -blk\..*\.attn_output.*=iq6_k
> +blk\..*\.attn_qkv.*=q8_0
> +blk\..*\.attn_output.*=q8_0
>  
>  # First 4 Dense Layers [0-3]
> -blk\..*\.ffn_down\.weight=iq5_ks
> -blk\..*\.ffn_(gate|up)\.weight=iq5_ks
> +blk\..*\.ffn_down\.weight=q8_0
> +blk\..*\.ffn_(gate|up)\.weight=q8_0
>  
>  # Shared Expert Layers [3-79]
> -blk\..*\.ffn_down_shexp\.weight=iq5_ks
> -blk\..*\.ffn_(gate|up)_shexp\.weight=iq5_ks
> +blk\..*\.ffn_down_shexp\.weight=q8_0
> +blk\..*\.ffn_(gate|up)_shexp\.weight=q8_0
>  
>  # Routed Experts Layers [3-79]
>  blk\..*\.ffn_down_exps\.weight=iq1_kt
>  blk\..*\.ffn_(gate|up)_exps\.weight=iq1_kt
>  
>  # Non-Repeating Layers
> -token_embd\.weight=iq4_k
> -output\.weight=iq6_k
> +token_embd\.weight=q8_0
> +output\.weight=q8_0
>  "
>  
>  custom=$(
> @@ -34,6 +34,6 @@
>      --imatrix ./ubergarm/Ling-flash-2.0/Ling-flash-256x3.4B-2.0-BF16-imatrix.dat \
>      --custom-q "$custom" \
>      ./ubergarm/Ling-flash-2.0/Ling-flash-256x3.4B-2.0-BF16-00001-of-00005.gguf \
> -    ./ubergarm/Ling-flash-2.0/IQ1_KT/Ling-flash-256x3.4B-2.0-IQ1_KT.gguf \
> +    ./ubergarm/Ling-flash-2.0/IQ1_KT-PUFFED/Ling-flash-256x3.4B-2.0-IQ1_KT-PUFFED.gguf \
>      IQ1_KT \
>      $(($(nproc) / 2))
> 
> ```
> 
> File: /root/make-quant-IQ1_KT-PUFFED.sh
> ```bash
> #!/usr/bin/env bash
> 
> 
> custom="
> # 80 Repeating Layers [0-79]
> 
> # Attention
> blk\..*\.attn_qkv.*=q8_0
> blk\..*\.attn_output.*=q8_0
> 
> # First 4 Dense Layers [0-3]
> blk\..*\.ffn_down\.weight=q8_0
> blk\..*\.ffn_(gate|up)\.weight=q8_0
> 
> # Shared Expert Layers [3-79]
> blk\..*\.ffn_down_shexp\.weight=q8_0
> blk\..*\.ffn_(gate|up)_shexp\.weight=q8_0
> 
> # Routed Experts Layers [3-79]
> blk\..*\.ffn_down_exps\.weight=iq1_kt
> blk\..*\.ffn_(gate|up)_exps\.weight=iq1_kt
> 
> # Non-Repeating Layers
> token_embd\.weight=q8_0
> output\.weight=q8_0
> "
> 
> custom=$(
>   echo "$custom" | grep -v '^#' | \
>   sed -Ez 's:\n+:,:g;s:,$::;s:^,::'
> )
> 
> ./ik_llama.cpp/build/bin/llama-quantize \
>     --imatrix ./ubergarm/Ling-flash-2.0/Ling-flash-256x3.4B-2.0-BF16-imatrix.dat \
>     --custom-q "$custom" \
>     ./ubergarm/Ling-flash-2.0/Ling-flash-256x3.4B-2.0-BF16-00001-of-00005.gguf \
>     ./ubergarm/Ling-flash-2.0/IQ1_KT-PUFFED/Ling-flash-256x3.4B-2.0-IQ1_KT-PUFFED.gguf \
>     IQ1_KT \
>     $(($(nproc) / 2))
> ```
> 
> File: /root/make-quant-IQ1_KT.sh
> ```bash
> #!/usr/bin/env bash
> 
> 
> custom="
> # 80 Repeating Layers [0-79]
> 
> # Attention
> blk\..*\.attn_qkv.*=iq6_k
> blk\..*\.attn_output.*=iq6_k
> 
> # First 4 Dense Layers [0-3]
> blk\..*\.ffn_down\.weight=iq5_ks
> blk\..*\.ffn_(gate|up)\.weight=iq5_ks
> 
> # Shared Expert Layers [3-79]
> blk\..*\.ffn_down_shexp\.weight=iq5_ks
> blk\..*\.ffn_(gate|up)_shexp\.weight=iq5_ks
> 
> # Routed Experts Layers [3-79]
> blk\..*\.ffn_down_exps\.weight=iq1_kt
> blk\..*\.ffn_(gate|up)_exps\.weight=iq1_kt
> 
> # Non-Repeating Layers
> token_embd\.weight=iq4_k
> output\.weight=iq6_k
> "
> 
> custom=$(
>   echo "$custom" | grep -v '^#' | \
>   sed -Ez 's:\n+:,:g;s:,$::;s:^,::'
> )
> 
> ./ik_llama.cpp/build/bin/llama-quantize \
>     --imatrix ./ubergarm/Ling-flash-2.0/Ling-flash-256x3.4B-2.0-BF16-imatrix.dat \
>     --custom-q "$custom" \
>     ./ubergarm/Ling-flash-2.0/Ling-flash-256x3.4B-2.0-BF16-00001-of-00005.gguf \
>     ./ubergarm/Ling-flash-2.0/IQ1_KT/Ling-flash-256x3.4B-2.0-IQ1_KT.gguf \
>     IQ1_KT \
>     $(($(nproc) / 2))
> ```

> 👤 **magikRUKKOLA** replied on **2025-10-24** at **19:18:35**
> 
> @saood06 
> 
> The full data regarding the iq1_kt of Ling-flash-2.0 in spec. decoding for Ling-T1 has been collected.
> ~~The results are more than strange.  It seems like the way the model performing in the llama-sweep-bench and in the draft model inside llama-server are two different things.~~
> 
> ~~Is there any way to see the timestamps of the execution stages (main model prefill/decode/verification and draft model prefill/decode) during spec. decoding?~~

> 👤 **magikRUKKOLA** replied on **2025-10-24** at **19:34:13**
> 
> @ikawrakow 
> retested at 64C CPU machine (where the inference test are actually running).
> 
> iq1_kt:
> ```
>   Device 0: NVIDIA GeForce RTX 3090, compute capability 8.6, VMM: yes
>   Device 1: NVIDIA GeForce RTX 3090, compute capability 8.6, VMM: yes
>   Device 2: NVIDIA GeForce RTX 3090, compute capability 8.6, VMM: yes
> llm_load_tensors: ggml ctx size =    0.57 MiB
> llm_load_tensors: offloading 32 repeating layers to GPU
> llm_load_tensors: offloading non-repeating layers to GPU
> llm_load_tensors: offloaded 33/33 layers to GPU
> llm_load_tensors:        CPU buffer size =   345.38 MiB
> llm_load_tensors:      CUDA0 buffer size = 11616.71 MiB
> llm_load_tensors:      CUDA1 buffer size = 11304.65 MiB
> ...................................................................................................
> llama_new_context_with_model: n_ctx      = 32768
> llama_new_context_with_model: n_batch    = 4096
> llama_new_context_with_model: n_ubatch   = 4096
> llama_new_context_with_model: flash_attn = 1
> llama_new_context_with_model: mla_attn   = 0
> llama_new_context_with_model: attn_max_b = 512
> llama_new_context_with_model: fused_moe  = 0
> llama_new_context_with_model: grouped er = 0
> llama_new_context_with_model: fused_up_gate = 1
> llama_new_context_with_model: ser        = -1, 0
> llama_new_context_with_model: freq_base  = 600000.0
> llama_new_context_with_model: freq_scale = 1
> llama_kv_cache_init:      CUDA0 KV buffer size =  1088.00 MiB
> llama_kv_cache_init:      CUDA1 KV buffer size =   960.00 MiB
> llama_new_context_with_model: KV self size  = 2048.00 MiB, K (f16): 1024.00 MiB, V (f16): 1024.00 MiB
> llama_new_context_with_model:  CUDA_Host  output buffer size =     0.60 MiB
> llama_new_context_with_model: pipeline parallelism enabled (n_copies=1)
> llama_new_context_with_model:      CUDA0 compute buffer size =  1292.02 MiB
> llama_new_context_with_model:      CUDA1 compute buffer size =  2520.00 MiB
> llama_new_context_with_model:  CUDA_Host compute buffer size =   320.05 MiB
> llama_new_context_with_model: graph nodes  = 1455
> llama_new_context_with_model: graph splits = 3
> 
> main: n_kv_max = 32768, n_batch = 4096, n_ubatch = 4096, flash_attn = 1, n_gpu_layers = 99, n_threads = 64, n_threads_batch = 64
> 
> |    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
> |-------|--------|--------|----------|----------|----------|----------|
> |  4096 |   1024 |      0 |    1.199 |  3416.70 |   10.591 |    96.69 |
> |  4096 |   1024 |   4096 |    1.248 |  3281.09 |   10.969 |    93.35 |
> |  4096 |   1024 |   8192 |    1.373 |  2983.84 |   11.328 |    90.40 |
> |  4096 |   1024 |  12288 |    1.504 |  2723.88 |   11.724 |    87.34 |
> |  4096 |   1024 |  16384 |    1.632 |  2509.63 |   12.038 |    85.06 |
> |  4096 |   1024 |  20480 |    1.757 |  2330.61 |   12.367 |    82.80 |
> |  4096 |   1024 |  24576 |    1.895 |  2160.97 |   12.604 |    81.25 |
> |  4096 |   1024 |  28672 |    2.024 |  2024.21 |   12.913 |    79.30 |
> 
> ```
> 
> iq1_kt-PUFFED:
> 
> ```
> ggml_cuda_init: found 3 CUDA devices:
>   Device 0: NVIDIA GeForce RTX 3090, compute capability 8.6, VMM: yes
>   Device 1: NVIDIA GeForce RTX 3090, compute capability 8.6, VMM: yes
>   Device 2: NVIDIA GeForce RTX 3090, compute capability 8.6, VMM: yes
> llm_load_tensors: ggml ctx size =    0.57 MiB
> llm_load_tensors: offloading 32 repeating layers to GPU
> llm_load_tensors: offloading non-repeating layers to GPU
> llm_load_tensors: offloaded 33/33 layers to GPU
> llm_load_tensors:        CPU buffer size =   652.38 MiB
> llm_load_tensors:      CUDA0 buffer size = 11881.56 MiB
> llm_load_tensors:      CUDA1 buffer size = 11647.89 MiB
> .................................................................................................
> llama_new_context_with_model: n_ctx      = 32768
> llama_new_context_with_model: n_batch    = 4096
> llama_new_context_with_model: n_ubatch   = 4096
> llama_new_context_with_model: flash_attn = 1
> llama_new_context_with_model: mla_attn   = 0
> llama_new_context_with_model: attn_max_b = 512
> llama_new_context_with_model: fused_moe  = 0
> llama_new_context_with_model: grouped er = 0
> llama_new_context_with_model: fused_up_gate = 1
> llama_new_context_with_model: ser        = -1, 0
> llama_new_context_with_model: freq_base  = 600000.0
> llama_new_context_with_model: freq_scale = 1
> llama_kv_cache_init:      CUDA0 KV buffer size =  1088.00 MiB
> llama_kv_cache_init:      CUDA1 KV buffer size =   960.00 MiB
> llama_new_context_with_model: KV self size  = 2048.00 MiB, K (f16): 1024.00 MiB, V (f16): 1024.00 MiB
> llama_new_context_with_model:  CUDA_Host  output buffer size =     0.60 MiB
> llama_new_context_with_model: pipeline parallelism enabled (n_copies=1)
> llama_new_context_with_model:      CUDA0 compute buffer size =  1292.02 MiB
> llama_new_context_with_model:      CUDA1 compute buffer size =  2520.00 MiB
> llama_new_context_with_model:  CUDA_Host compute buffer size =   320.05 MiB
> llama_new_context_with_model: graph nodes  = 1455
> llama_new_context_with_model: graph splits = 3
> 
> main: n_kv_max = 32768, n_batch = 4096, n_ubatch = 4096, flash_attn = 1, n_gpu_layers = 99, n_threads = 64, n_threads_batch = 64
> 
> |    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
> |-------|--------|--------|----------|----------|----------|----------|
> |  4096 |   1024 |      0 |    1.090 |  3758.64 |   10.919 |    93.78 |
> |  4096 |   1024 |   4096 |    1.199 |  3415.01 |   11.288 |    90.72 |
> |  4096 |   1024 |   8192 |    1.321 |  3100.04 |   11.629 |    88.05 |
> |  4096 |   1024 |  12288 |    1.456 |  2813.35 |   12.041 |    85.04 |
> |  4096 |   1024 |  16384 |    1.582 |  2589.07 |   12.379 |    82.72 |
> |  4096 |   1024 |  20480 |    1.712 |  2392.91 |   12.689 |    80.70 |
> |  4096 |   1024 |  24576 |    1.836 |  2231.48 |   12.933 |    79.18 |
> |  4096 |   1024 |  28672 |    1.985 |  2063.72 |   13.248 |    77.29 |
> ```
> 
> Somehow the PUFFED quant indeed slower.  Ha.  That explains everything.
> I only need to figure out why it was faster at 12C machine.  Let me check.
> 
> [EDIT]:
> 
> Ah.  I found the bug.  While doing the llama-sweep-bench I forgot to remove -fmoe for the IQ1_KT-PUFFED.  That gave it 20% boost in decode.
> 
> [EDIT2]:
> 
> The draft model doesn't use the fmoe, right?
> ```
>  llama_new_context_with_model: fused_moe  = 0
> ```

> 👤 **magikRUKKOLA** replied on **2025-10-24** at **20:01:27**
> 
> Okay cool.  So at least now we know that the drop in decode speed for the draft model from 82.8 to 80.7 in decode translates to about 0.1 tps difference in decode for the target model.
> 
> Thanks  everyone for the help!  I will continue by figuring out how to use @Thireus framework to improve the PPL.

> 👤 **ikawrakow** replied on **2025-10-25** at **05:39:01**
> 
> Oh, the draft is running without `fmoe` and without `fa`? This means that TG performance is quite a bit lower than it could be.
> 
> I guess, I need to put more effort into draft models. Or perhaps just make `fmoe` and `fa` to be the default.
> 
> Concerning the PUFFED vs not PUFFED performance: there is no way the PUFFED version can have a better TG performance. TG is memory bound, and the size of the PUFFED **active parameters** is quite a bit larger. PP of the PUFFED is better because the not PUFFED uses `IQ6_K`, which has a lower PP performance than `Q8_0` due to the block size of 16.

> 👤 **ikawrakow** replied on **2025-10-25** at **06:38:53**
> 
> OK, `fmoe` and flash attention are now on by default on the main branch. I'm curious to see if this will improve the speculative performance.

> 👤 **magikRUKKOLA** replied on **2025-10-25** at **11:48:44**
> 
> @saood06 
> 
> Make an additional run with Q8_0 KV for the draft model.
> 
> ![journal_20251025_113244_data_plot](https://github.com/user-attachments/assets/74f7d2ff-bcc3-48b4-9abd-5faaa6cad437)
> 
> If compared to the same config there is a performance degradation (~0.15 tps and about %1 in acceptance rate).
> 
> ![journal_20251024_191118_data_plot](https://github.com/user-attachments/assets/bce04a4f-0d0e-451d-9efb-be8c2045db3c)

> 👤 **magikRUKKOLA** replied on **2025-10-25** at **14:01:24**
> 
> @ikawrakow 
> > . I'm curious to see if this will improve the speculative performance.
> 
> Yes, it does.  Its about +0.16 tps for now (6.88 tps vs 7.04 tps).  Here are the prelim results:
> ![journal_20251025_201058_data_plot](https://github.com/user-attachments/assets/cb4bf3db-c649-4c12-83b5-bff42c310b8f)
> 
> 
> [EDIT]:
> 
> the only strange thing is that despite the improvement in speed (both of the draft LLM, and, consequently, the target LLM) the acceptance rate got reduced.
> Does that mean that the draft LLM produces lesser quality results with such config?  Or, what possibly could affect that?

> 👤 **magikRUKKOLA** replied on **2025-10-25** at **19:36:09**
> 
> I will be (re-)testing the IQ2_KL with the current master both in f16 and q8_0 with various configs.  In case if anyone got some more ideas let me know.  As of now it seems that I need to add a fourth GPU.

> 👤 **magikRUKKOLA** replied on **2025-10-26** at **06:36:21**
> 
> IQ2_KL draft performance improved too.  PR-863 and the old graph for comparison:
> 
> ![journal_20251026_062504_data_plot](https://github.com/user-attachments/assets/a3fc0dac-8d0b-4de3-9c66-429c39065d37)
> 
> 
> ![journal_20251025_201058_data_plot](https://github.com/user-attachments/assets/cb4bf3db-c649-4c12-83b5-bff42c310b8f)
> 
> 
> Interestingly, the performance of IQ2_KL is almost the same as IQ1_KT even though the acceptance rate is much higher for the IQ2_KL (+4%).