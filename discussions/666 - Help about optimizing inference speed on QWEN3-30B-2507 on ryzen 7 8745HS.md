## 🗣️ [Discussion #666](https://github.com/ikawrakow/ik_llama.cpp/discussions/666) - Help about optimizing inference speed on QWEN3-30B-2507 on ryzen 7 8745HS

| **Author** | `delta-whiplash` |
| :--- | :--- |
| **State** | ✅ **Open** |
| **Created** | 2025-08-01 |
| **Updated** | 2025-08-07 |

---

## 📄 Description

Hello, here's my current setup:

```yaml
services:
  # 1. Qwen3-30B-A3B – CPU-only optimized configuration + Cache Level 1+3
  qwen30b:
    image: ghcr.io/ggml-org/llama.cpp:server
    container_name: qwen30b-server
    environment:
      - LLAMA_ARG_MODEL=/models/Qwen3-30B-A3B-Instruct-2507-UD-Q4_K_XL.gguf
      - LLAMA_ARG_JINJA=1
      - LLAMA_ARG_N_GPU_LAYERS=99           # Maximum possible, full GPU offload if available
      - LLAMA_ARG_THREADS=-1                # Use all available threads (auto-optimized)
      - LLAMA_ARG_CTX_SIZE=32768            # Optimal performance/memory trade-off
      - LLAMA_ARG_TEMP=0.7                  # Official Qwen best practice
      - LLAMA_ARG_TOP_P=0.8                 # Same
      - LLAMA_ARG_TOP_K=20                  # Same
      - LLAMA_ARG_MIN_P=0.0                 # Override default llama.cpp value
      - LLAMA_ARG_PRESENCE_PENALTY=1.5      # Reduce repetition (set to 0 if unsupported)
      - LLAMA_ARG_BATCH_SIZE=1024
      - LLAMA_ARG_HOST=0.0.0.0
      - LLAMA_ARG_PORT=8080
      - LLAMA_ARG_CACHE_PROMPT=1
    ports:
      - "8080:8080"
    volumes:
      - /home/delta/models:/models
      - /home/delta/cache/qwen30b:/cache
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/v1/models"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 60s
    restart: unless-stopped
    deploy:
      resources:
        limits:
          memory: 24G
        reservations:
          memory: 22G
```

I'm aiming to optimize both prompt processing speed and generation speed.

So far, my tests with `llama-optimus` have shown better CPU performance than GPU — though at the cost of higher latency.

I've offloaded 16 GB to the iGPU, and I have 64 GB of RAM available.

The `ik_llama.cpp` project seems promising. Do you think I can achieve better performance with my setup?

Currently, I'm getting:
- **61 tokens/sec** for prompt processing  
- **18 tokens/sec** for generation  

I haven’t found an official Docker image for `ik_llama.cpp` — is one planned?

Also, is the project compatible with **Vulkan** or **ROCm**?

---

## 💬 Discussion

👤 **ikawrakow** commented on **2025-08-01** at **10:47:42**

I have a Ryzen 7950X. Running this model CPU-only, I get in the range of 500 t/s for prompt processing (PP), and 30 t/s for token generation (TG). My guess is that your CPU should be able to do at least 200 t/s PP. TG is memory bound, so more difficult to estimate.

Vulkan is supported, but the important optimisations have not been ported to Vulkan yet, so performance should be very similar to llama.cpp.

There are no docker images, and no near future plans to provide.

> 👤 **delta-whiplash** replied on **2025-08-01** at **11:01:58**
> 
> I think if I understand correctly by switching to `ik_llama.cpp` running on CPU only, I could achieve better performance than my current setup.   
> 
> My system has two 32 GB RAM modules (total 64 GB) running at 5600 MHz.
> Currently, my Docker setup uses only CPU (no GPU) 
> 
> Should I consider changing my GGUF model quantization from Q4_K_XL to IQ4_NL, or perhaps another quantization type ?
> I’m not entirely clear on the differences between the various quantizations. 
> Could you please explain them to me, or point me to a reliable resource that covers this topic? 
> 
> Thanks in advance!

> 👤 **ikawrakow** replied on **2025-08-01** at **11:17:20**
> 
> I'm on vacation and using my phone, so difficult to give more detailed answers. If you want to try ik_llama.cpp, you can start with the model you are currently using and worry about better alternatives later.

> 👤 **delta-whiplash** replied on **2025-08-01** at **11:40:18**
> 
> ok am gonna give a try

> 👤 **sousekd** replied on **2025-08-01** at **12:16:01**
> 
> > ok am gonna give a try
> 
> Just an extra tip: Try experimenting with batch size. _Some_ models or quants benefit a lot from larger `-b` and `-ub`. I am not sure whether it applies to CPU-only configuration too, but with GPU, I was able to get 2x or 3x prompt processing speed only by increasing batch size to 8K or even 16K.

> 👤 **delta-whiplash** replied on **2025-08-02** at **16:21:28**
> 
> Hi @sousekd  @ikawrakow ,
> 
> Thank you for your responses and the tips on batch size experimentation
> it's been incredibly helpful! I'm the original poster (delta-whiplash), and I've spent the last  days testing ik_llama.cpp on my 
> I'm using now the Qwen3-30B-A3B-Instruct-2507-IQ4_XS.gguf model with a context size of 32768, and I've focused on optimizing prompt processing (PP) and token generation (TG) speeds without changing quantization to preserve response quality (IQ4_XS is already an acceptable balance for me).
> 
> I've compiled from the `main` branch (commit ae0ba31f) using Clang with Zen4-specific flags:  
> ```
> cmake .. -DCMAKE_BUILD_TYPE=Release -DGGML_AVX2=ON -DGGML_F16C=ON -DGGML_FMA=ON -DGGML_NATIVE=ON -DGGML_VULKAN=OFF -DGGML_CCACHE=ON -DGGML_OPENMP=ON -DCMAKE_C_COMPILER=clang -DCMAKE_CXX_COMPILER=clang++ -DCMAKE_C_FLAGS="-march=znver4 -mtune=znver4 -O3 -fno-finite-math-only -mavx512f -mavx512vl -mavx512bw -mavx512dq -fopenmp -fuse-ld=lld" -DCMAKE_CXX_FLAGS="-march=znver4 -mtune=znver4 -O3 -fno-finite-math-only -mavx512f -mavx512vl -mavx512bw -mavx512dq -fopenmp -fuse-ld=lld"  
> 
> make -j$(nproc)
> ```
> 
> All tests used numactl for CPU/memory affinity (`numactl --cpunodebind=0 --membind=0`), and I ran consistent prompts (e.g., ~1096 tokens for session 1, ~380 for session 2, ~140 for session 3) to measure PP and TG across 3-4 sessions per run. Metrics are averages from logs (prompt eval time, generation eval time, total time, tokens/s). No NaNs or crashes observed, and response quality remained stable (coherent, no degradation).
> 
> What Worked Well :
> - **Switching to ik_llama.cpp from vanilla llama.cpp (via Docker)**: Massive gains overall PP sped up from 63 tok/s to 225 tok/s, TG from 22 tok/s to 25 tok/s. This was the biggest win, aligning with your benchmarks (I hit ~240 tok/s PP on long prompts vs. ~60-70 in Docker).
> - **Numactl affinity**: Consistent boost in PP from ~200 tok/s to ~225 tok/s and TG from ~23 tok/s to ~25 tok/s by binding to node 0.
> - **Flash Attention (--flash-attn)**: Activated successfully; added to TG debit from 24 tok/s to 25-26 tok/s.
> - **Larger batch sizes**: As per your tip, increasing -b (with -ub) helped PP a lot on CPU (e.g., from 225 tok/s at 2048 to 242 tok/s at 8192), though less dramatic than GPU (no 2-3x, but still worthwhile). No quality impact.
> 
> What Worked Less Well :
> - **MLA (--mla-use 1)**: Turned off in logs ("MLA is only available for LLM_ARCH_DEEPSEEK2"), so no gains Qwen3 isn't supported yet? Would love if this could be extended.
> - **Very large batches (16K)**: PP improved (e.g., from 225 tok/s to 244 tok/s), but total time spiked on long generations (e.g., from 8.7k ms to 20.8k ms in one session due to variability); better for short prompts only.
> - **CPU-only limits**: TG stayed ~24-26 tok/s max (memory-bound, as you mentioned); didn't hit your 7950X's 30 tok/s, likely due to my weaker CPU.
> 
> Detailed Test Results :
> Here are the key runs with commands, average metrics across sessions, and observations (dates: August 1-2, 2025). All used the same model/parameters (--temp 0.7, --top-p 0.8, etc.).
> 
> 1. **Baseline: Vanilla llama.cpp (Docker, batch 1024, no numactl/flash-attn)**  
>    Command: (From initial history, equivalent to Docker config).  
>    - Avg PP: 63 tok/s (15 ms/tok), TG: 22 tok/s (45 ms/tok), Total time: ~15k ms/session.  
>    - Observation: Slow PP; baseline for comparisons. Worked as expected but unoptimized.
> 
> 2. **ik_llama.cpp optimized build (batch 2048, numactl, --flash-attn --mla-use 1)**  
>    Command: `numactl --cpunodebind=0 --membind=0 ./bin/llama-server ... --batch-size 2048 --ubatch-size 512 --flash-attn --mla-use 1`  
>    - Avg PP: 225 tok/s (4.5 ms/tok, from 63 tok/s), TG: 25 tok/s (40 ms/tok, from 22 tok/s), Total time: ~8k ms/session (from ~15k ms).  
>    - Observation: Huge PP boost; MLA off, but flash-attn helped TG. Worked very well for all sessions.
> 
> 3. **Batch 8192 (with ub 512, numactl, --flash-attn --mla-use 1)**  
>    Command: As above, but `--batch-size 8192 --ubatch-size 512`.  
>    - Avg PP: 216 tok/s (4.9 ms/tok, from 225 tok/s), TG: 26 tok/s (38 ms/tok, from 25 tok/s), Total time: ~8.7k ms/session (from ~8k ms).  
>    - Observation: Mixed—PP slightly slower on short prompts, but TG improved. Less effective than expected on CPU; no big jump like GPU.
> 
> 4. **Batch 16384 (with ub 1024, numactl, --flash-attn --mla-use 1)**  
>    Command: As above, but `--batch-size 16384 --ubatch-size 1024`.  
>    - Avg PP: 212 tok/s (5.1 ms/tok, from 225 tok/s), TG: 25 tok/s (39 ms/tok, from 25 tok/s), Total time: ~20.8k ms/session (from ~8k ms due to one long generation).  
>    - Observation: PP gains on very long prompts, but overall unstable (high variance in total time). Not ideal for mixed workloads; risked higher latency.
> 
> Overall ik_llama.cpp is a gamechanger for my setupgot me to ~240 tok/s PP and 25-26 tok/s TG consistently with batch 2048 + flash-attn. Batch experimentation helped PP (best at 8K for balance), but gains were modest on CPU (~5-10%, not 2-3x). Would love MLA support for Qwen3 to push further. Any advice on enabling fused MoE or other CPU tweaks? 
> 
> Thanks again!

> 👤 **ikawrakow** replied on **2025-08-02** at **16:43:09**
> 
> MLA is the DeepSeek self attention mechanism. Qwen models don't use that, so the mla command line argument does nothing.

> 👤 **delta-whiplash** replied on **2025-08-03** at **10:12:07**
> 
> Ok I understand 
> Is there any command line or build option I forgot to enhance my actual setup or am I already getting SOTA speed on my hardware?
> 
> Do you have any option or recommandation for K_V system prompt caching (I have 3 or 4 app that use different long system prompt Soo I wanted to enhance pp)
> 
> Am using litellm in front of ik_llama.cpp for better api token management

---

👤 **YurkoHoshko** commented on **2025-08-07** at **06:59:35**

Can confirm similar on my MiniPC with the same CPU (8945HS(8C/16T, up to 5.2GHz) / 64GB DDR5)

Additionally, experimenting with GPU offloading (RTX 3060, 12GB). Some amount of improvement on generation, but prompt processing jumped almost 2x - great performance compared to `llama.cpp` - huge kudos to maintainers!

```bash
/ik_llama.cpp/build/bin/llama-bench  -m /models/Qwen3-Coder-30B-A3B-Instruct-UD-Q4_K_XL.gguf -ngl 99 -fa -fmoe -ub 768 -ot  'blk.(1[8-9]|[2-4][0-9]).ffn_.*._exps=CPU' -rtr  1
ggml_cuda_init: GGML_CUDA_FORCE_MMQ:    no
ggml_cuda_init: GGML_CUDA_FORCE_CUBLAS: no
ggml_cuda_init: found 1 CUDA devices:
  Device 0: NVIDIA GeForce RTX 3060, compute capability 8.6, VMM: yes
| model                          |       size |     params | backend    | ngl | n_ubatch | rtr |          test |              t/s |
| ------------------------------ | ---------: | ---------: | ---------- | --: | -------: | --: | ------------: | ---------------: |
============ Repacked 90 tensors
| qwen3moe ?B Q4_K - Medium      |  16.45 GiB |    30.53 B | CUDA       |  99 |      768 |   1 |         pp512 |    445.85 ± 2.40 |
| qwen3moe ?B Q4_K - Medium      |  16.45 GiB |    30.53 B | CUDA       |  99 |      768 |   1 |         tg128 |     36.28 ± 0.22 |
```

Without re-packing my results drop quite a bit:

```bash
/ik_llama.cpp/build/bin/llama-bench  -m /models/Qwen3-Coder-30B-A3B-Instruct-UD-Q4_K_XL.gguf -ngl 99 -fa -fmoe -ot  'blk.(1[8-9]|[2-4][0-9]).ffn_.*._exps=CPU' -rtr  0 -ub 768
ggml_cuda_init: GGML_CUDA_FORCE_MMQ:    no
ggml_cuda_init: GGML_CUDA_FORCE_CUBLAS: no
ggml_cuda_init: found 1 CUDA devices:
  Device 0: NVIDIA GeForce RTX 3060, compute capability 8.6, VMM: yes
| model                          |       size |     params | backend    | ngl | n_ubatch |          test |              t/s |
| ------------------------------ | ---------: | ---------: | ---------- | --: | -------: | ------------: | ---------------: |
| qwen3moe ?B Q4_K - Medium      |  16.45 GiB |    30.53 B | CUDA       |  99 |      768 |         pp512 |    251.61 ± 0.80 |
| qwen3moe ?B Q4_K - Medium      |  16.45 GiB |    30.53 B | CUDA       |  99 |      768 |         tg128 |     36.23 ± 0.18 |
```