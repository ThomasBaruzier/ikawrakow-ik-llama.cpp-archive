## 🗣️ [Discussion #718](https://github.com/ikawrakow/ik_llama.cpp/discussions/718) - Help w/ Faster prefill on CPU-MoE?

| **Author** | `QuantumPlayDev` |
| :--- | :--- |
| **State** | ✅ **Answered** |
| **Created** | 2025-08-22 |
| **Updated** | 2025-11-05 |

---

## 📄 Description

**Problem**:

Our first long turn (prefill) is slow on CPU-MoE: both GPUs sit ~1–10% SM during prompt digestion, only rising after tokens start. Subsequent turns are fast (cache helps), but we’d like higher GPU utilization during prefill without OOMs.

**Goal**:

- Max prefill throughput (long prompt digestion) while staying OOM-safe and keeping 128k context stable.
- Keep this working on 2×24 GB RTX 4090 now; later we’ll have 2×96 GB RTX 6000-class GPUs and can move experts to VRAM.

⸻

What we’re asking:

- Best offload policy for CPU-MoE prefill (is -op 26,1,27,1,29,1 the right starting point?).

- Practical -ub / -amb ranges on 2×24 GB for 128k ctx (8-bit KV), and how to balance with --n-gpu-layers.

- Good -ot FFN pinning patterns for Qwen3-Coder-480B to keep both GPUs busy without prefill OOM.

- Preferred NUMA mode on EPYC for large prefill: --numa distribute vs --numa isolate.

- Any build-time flags (e.g., GGML_CUDA_MIN_BATCH_OFFLOAD) that help CPU-MoE prefill.

⸻

Minimal repro (single command)

This is the smallest command that’s stable for us on 2×4090 and shows the issue.

# in host$ = Pop!_OS terminal (single-GPU server)
MODEL_FIRST="$(ls -1v $HOME/models/Qwen3-Coder-480B-A35B-Instruct/Qwen3-480B-A35B-Instruct-IQ5_K-00001-of-*.gguf | head -n1)"

CUDA_VISIBLE_DEVICES=1,0 $HOME/ik_llama.cpp/build/bin/llama-server \
  --model "$MODEL_FIRST" \
  --alias openai/local \
  --host 127.0.0.1 --port 8080 \
  --ctx-size 131072 \
  -fa -fmoe --cpu-moe \
  --split-mode layer --n-gpu-layers 63 \
  -ctk q8_0 -ctv q8_0 \
  -b 2048 -ub 512 -amb 512 \        # 640/640 sometimes OOMs with wider -ot
  --threads 20 --threads-batch 20 \
  --prompt-cache "$HOME/.cache/ik-llama/openai_local_8080.promptcache" --prompt-cache-all \
  --slot-save-path "$HOME/llama_slots/openai_local_8080" \
  --keep -1 \
  --slot-prompt-similarity 0.35 \
  -op 26,1,27,1,29,1 \              # offload policy to push PP work to CUDA
  -ot 'blk\.(3|4)\.ffn_.*=CUDA0' \
  -ot 'blk\.(5|6)\.ffn_.*=CUDA1' \
  --metrics

**Behavior**:

- First long turn (128k possible; KV at q8_0) shows low SM% during prefill (often 1–10%), then ~20–30% as tokens start.

- Follow-up turns with the same prefix are near-instant (prompt/slot cache doing its job).

- Widening -ot (e.g., add blocks 2/7) helps a bit until VRAM pressure forces us back to 512/512 or narrower -ot.

⸻

**Context / what we’ve tried**

- Model: Qwen3-Coder-480B-A35B-Instruct (GGUF IQ5_K, 8 shards).

- Approach: Experts on CPU for stability/VRAM headroom (--cpu-moe, --override-tensor exps=CPU), dense layers to GPU (--split-mode layer, --n-gpu-layers ~56–63), 8-bit KV (-ctk/-ctv q8_0).

- Compute buffers: -ub/-amb 384/384 → 512/512 (stable). 640/640 sometimes OOMs when -ot is widened.

- Threads: --threads 20 --threads-batch 20 (EPYC sweet spot across runs).

- Prompt/slot caching: --prompt-cache … --prompt-cache-all --slot-save-path … --keep -1 + client cache_prompt:true → follow-ups are fast.

- NUMA: tried distribute and isolate; not a clear win yet.

- Build: CUDA on; experimenting with GGML_CUDA_MIN_BATCH_OFFLOAD=16, GGML_SCHED_MAX_COPIES=1, GGML_CUDA_FA_ALL_QUANTS=ON, GGML_IQK_FA_ALL_QUANTS=ON.

- Throughput: ~11.4–12.0 tok/s gen at 128k ctx (IQ5_K). Prefill is the bottleneck.

⸻

Hardware
- CPU: AMD EPYC 9225 • RAM: 768 GB DDR5-6000
- GPUs (now): 2× RTX 4090 (24 GB)
- GPUs (soon): 2× 96 GB RTX 6000-class (plan to drop --cpu-moe then)

⸻

Thanks! Any recommended -op policies, -ub/-amb ranges, -ot patterns, and NUMA/build tips for CPU-MoE prefill on 2×4090 would be hugely appreciated. Happy to run micro-sweeps and share CSVs if that helps.

---

## 💬 Discussion

👤 **magikRUKKOLA** commented on **2025-08-22** at **20:56:30**

I haven't tested the Qwen3-Coder.  Personally I would rather prefer DeepSeek (considering a very poor performance of Qwen3 previous releases ... newermind).  So regarding to DeepSeek specifically, I found out that 3 x RTX 3090 can handle 160k context perfectly so with the highest precision quant for DeepSeek for example (6.2bpw, from @Thireus ) I am having up to 6 tps.  https://github.com/ikawrakow/ik_llama.cpp/discussions/477#discussioncomment-13823794 .
An additional note is that one would probably have to install a custom NVIDIA driver to enable P2P PCIe between the GPUS.

> 👤 **Thireus** replied on **2025-08-22** at **21:39:02**
> 
> > I haven't tested the Qwen3-Coder. Personally I would rather prefer DeepSeek (considering a very poor performance of Qwen3 previous releases ... newermind). So regarding to DeepSeek specifically, I found out that 3 x RTX 3090 can handle 160k context perfectly so with the highest precision quant for DeepSeek for example (6.2bpw, from @Thireus ) I am having up to 6 tps. [[#477](https://github.com/ikawrakow/ik_llama.cpp/issues/477) (reply in thread)](https://github.com/ikawrakow/ik_llama.cpp/discussions/477#discussioncomment-13823794) . An additional note is that you would probably have to install a custom NVIDIA driver to enable P2P PCIe between the GPUS.
> 
> Since last time we spoke there's been some suggestions from @ikawrakow who pointed out faster speed is achieved when harmonizing ffn up and gate exps tensors (which allows them to be fused).
> 
> I recommend the following:
> 1. Send your recipe file to https://colab.research.google.com/github/Thireus/GGUF-Tool-Suite/blob/main/recipe_to_colab_params.ipynb
> 2. Then pass the pipeline parameters to https://colab.research.google.com/github/Thireus/GGUF-Tool-Suite/blob/main/quant_recipe_pipeline.ipynb
> 
> You will obtain a new recipe with harmonized ffn exps layers. That will give you some additional TG/s speed.
> 
> I can also do it for you if you'd like, but I can't recall which .recipe you use so you'll have to send it here please.
> 
> Edit: I believe I found it, was it this one? https://github.com/Thireus/GGUF-Tool-Suite/blob/main/recipe_examples/ik_llama.cpp_recipes/DeepSeek-R1-0528.ROOT-6.2478bpw-3.2240ppl.488GB-GGUF_11GB-GPU_477GB-CPU.3c88ec6_c3039f4.recipe
> 
> I'm attaching the harmonized recipe, give it a try. The PPL shouldn't be negatively affected. Please let me know how much of a speed boost that gives you.
> 
> [DeepSeek-R1-0528.ROOT-6.4296bpw-0.0000ppl.502GB-GGUF_11GB-GPU_491GB-CPU.6d32a73_633eef7.recipe.zip](https://github.com/user-attachments/files/21945437/DeepSeek-R1-0528.ROOT-6.4296bpw-0.0000ppl.502GB-GGUF_11GB-GPU_491GB-CPU.6d32a73_633eef7.recipe.zip)
> 
> Maybe give a try to this one as well (should give better PPL):
> 
> [DeepSeek-R1-0528.ROOT-6.5522bpw-0.0000ppl.511GB-GGUF_11GB-GPU_500GB-CPU.6d32a73_815e027.recipe.zip](https://github.com/user-attachments/files/21945484/DeepSeek-R1-0528.ROOT-6.5522bpw-0.0000ppl.511GB-GGUF_11GB-GPU_500GB-CPU.6d32a73_815e027.recipe.zip)

> 👤 **trilog-inc** replied on **2025-08-23** at **16:08:12**
> 
> Hey, are you able to run the full context during inference successfully? I keep getting the following error around 62K token PP using the Thireus prompt  ( https://thireus.com/GITHUB/Dipiloblop.txt )
> 
> > INFO [            update_slots] kv cache rm [p0, end) | tid="140371140771840" timestamp=1755874364 id_slot=0 id_task=0 p0=40960
> > INFO [            update_slots] kv cache rm [p0, end) | tid="140371140771840" timestamp=1755874437 id_slot=0 id_task=0 p0=49152
> > INFO [            update_slots] kv cache rm [p0, end) | tid="140371140771840" timestamp=1755874518 id_slot=0 id_task=0 p0=57344
> > /mnt/home_extend/llm/ik_llama/1708/ik_llama.cpp/ggml/src/ggml-cuda/cpy.cu:380: GGML_ASSERT(ggml_nbytes(src0) <= INT_MAX) failed
> 
> using this command ( maybe im doing something wrong here ): 
> ./build/bin/llama-server --model /mnt/home_extend/models/unsloth_DeepSeek-V3.1-GGUF/Q4_K_M/DeepSeek-V3.1-Q4_K_M-00001-of-00009.gguf --alias DeepSeek-V3.1 --threads 44 --ctx-size 112640 --n-gpu-layers 99 -ub 8192 -b 8192 -mla 3 -fa -amb 512  -ot exps=CPU --temp 0.6 --top-p 0.95 -fa -fmoe --host 0.0.0.0 --jinja --port 8099 --main-gpu 1 --lookup-cache-dynamic /mnt/home_extend/data/ik_llama.kv.dump --split-mode layer --tensor-split 10,21,21,21 --split-mode layer -ctk q8_0 -ctv q8_0

> 👤 **ikawrakow** replied on **2025-08-23** at **16:11:30**
> 
> This has been fixed in [#705](https://github.com/ikawrakow/ik_llama.cpp/issues/705)

> 👤 **trilog-inc** replied on **2025-08-23** at **16:13:57**
> 
> This is embarassing. I will update and try later. Thanks for all your hard work 🙌

> 👤 **trilog-inc** replied on **2025-08-25** at **14:57:23**
> 
> Works as expected

> 👤 **magikRUKKOLA** replied on **2025-08-26** at **14:10:46**
> 
> @Thireus 
> > 
> > I'm attaching the harmonized recipe, give it a try. The PPL shouldn't be negatively affected. Please let me know how much of a speed boost that gives you.
> > 
> > [DeepSeek-R1-0528.ROOT-6.4296bpw-0.0000ppl.502GB-GGUF_11GB-GPU_491GB-CPU.6d32a73_633eef7.recipe.zip](https://github.com/user-attachments/files/21945437/DeepSeek-R1-0528.ROOT-6.4296bpw-0.0000ppl.502GB-GGUF_11GB-GPU_491GB-CPU.6d32a73_633eef7.recipe.zip)
> > 
> 
> It seems like the bigger quant performing slightly slower actually (but have also slightly better ppl;  should we add it to the graphs?
> 
> ```
> /opt/ik_llama.cpp/ik_llama.cpp/build/bin/llama-server --version
> version: 3859 (50f7119d)
> built with cc (Debian 14.2.0-19) 14.2.0 for x86_64-linux-gnu
> ```
> 
> ```
> nvidia-smi
> Tue Aug 26 14:01:10 2025
> +-----------------------------------------------------------------------------------------+
> | NVIDIA-SMI 580.76.05              Driver Version: 580.76.05      CUDA Version: 13.0     |
> +-----------------------------------------+------------------------+----------------------+
> | GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
> | Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
> |                                         |                        |               MIG M. |
> |=========================================+========================+======================|
> |   0  NVIDIA GeForce RTX 3090        On  |   00000000:41:00.0 Off |                  N/A |
> | 30%   41C    P8             20W /  350W |       0MiB /  24576MiB |      0%      Default |
> |                                         |                        |                  N/A |
> +-----------------------------------------+------------------------+----------------------+
> |   1  NVIDIA GeForce RTX 3090        On  |   00000000:42:00.0 Off |                  N/A |
> |  0%   42C    P8             15W /  350W |       0MiB /  24576MiB |      0%      Default |
> |                                         |                        |                  N/A |
> +-----------------------------------------+------------------------+----------------------+
> |   2  NVIDIA GeForce RTX 3090        On  |   00000000:61:00.0 Off |                  N/A |
> | 30%   42C    P8             20W /  350W |       0MiB /  24576MiB |      0%      Default |
> |                                         |                        |                  N/A |
> +-----------------------------------------+------------------------+----------------------+
> 
> +-----------------------------------------------------------------------------------------+
> | Processes:                                                                              |
> |  GPU   GI   CI              PID   Type   Process name                        GPU Memory |
> |        ID   ID                                                               Usage      |
> |=========================================================================================|
> |  No running processes found                                                             |
> +-----------------------------------------------------------------------------------------+
> ```
> 
> THIREUS-6.2478bpw:
> 
> ```
> ....................................................................................................
> llama_new_context_with_model: n_ctx      = 163840
> llama_new_context_with_model: n_batch    = 8192
> llama_new_context_with_model: n_ubatch   = 4096
> llama_new_context_with_model: flash_attn = 1
> llama_new_context_with_model: mla_attn   = 3
> llama_new_context_with_model: attn_max_b = 512
> llama_new_context_with_model: fused_moe  = 1
> llama_new_context_with_model: ser        = -1, 0
> llama_new_context_with_model: freq_base  = 10000.0
> llama_new_context_with_model: freq_scale = 0.025
> llama_kv_cache_init:      CUDA0 KV buffer size =  1243.13 MiB
> llama_kv_cache_init:      CUDA1 KV buffer size =  2390.64 MiB
> llama_kv_cache_init:      CUDA2 KV buffer size =  2199.39 MiB
> llama_new_context_with_model: KV self size  = 5833.12 MiB, c^KV (q8_0): 5833.12 MiB, kv^T: not used
> llama_new_context_with_model:  CUDA_Host  output buffer size =     0.49 MiB
> llama_new_context_with_model: pipeline parallelism enabled (n_copies=1)
> llama_new_context_with_model:      CUDA0 compute buffer size = 17512.02 MiB
> llama_new_context_with_model:      CUDA1 compute buffer size = 14992.02 MiB
> llama_new_context_with_model:      CUDA2 compute buffer size = 14992.03 MiB
> llama_new_context_with_model:  CUDA_Host compute buffer size =  2672.05 MiB
> llama_new_context_with_model: graph nodes  = 45841
> llama_new_context_with_model: graph splits = 178
> 
> main: n_kv_max = 163840, n_batch = 8192, n_ubatch = 4096, flash_attn = 1, n_gpu_layers = 99, n_threads = 64, n_threads_batch = 64
> 
> |    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
> |-------|--------|--------|----------|----------|----------|----------|
> |  4096 |   1024 |      0 |   34.812 |   117.66 |  158.705 |     6.45 |
> |  4096 |   1024 |   4096 |   35.392 |   115.73 |  161.739 |     6.33 |
> |  4096 |   1024 |   8192 |   35.941 |   113.96 |  166.934 |     6.13 |
> |  4096 |   1024 |  12288 |   36.532 |   112.12 |  170.786 |     6.00 |
> |  4096 |   1024 |  16384 |   37.114 |   110.36 |  174.620 |     5.86 |
> ```
> 
> 
> **THIREUS-6.4296bpw**:
> 
> ```
> ....................................................................................................
> llama_new_context_with_model: n_ctx      = 163840
> llama_new_context_with_model: n_batch    = 8192
> llama_new_context_with_model: n_ubatch   = 4096
> llama_new_context_with_model: flash_attn = 1
> llama_new_context_with_model: mla_attn   = 3
> llama_new_context_with_model: attn_max_b = 512
> llama_new_context_with_model: fused_moe  = 1
> llama_new_context_with_model: ser        = -1, 0
> llama_new_context_with_model: freq_base  = 10000.0
> llama_new_context_with_model: freq_scale = 0.025
> llama_kv_cache_init:      CUDA0 KV buffer size =  1243.13 MiB
> llama_kv_cache_init:      CUDA1 KV buffer size =  2390.64 MiB
> llama_kv_cache_init:      CUDA2 KV buffer size =  2199.39 MiB
> llama_new_context_with_model: KV self size  = 5833.12 MiB, c^KV (q8_0): 5833.12 MiB, kv^T: not used
> llama_new_context_with_model:  CUDA_Host  output buffer size =     0.49 MiB
> llama_new_context_with_model: pipeline parallelism enabled (n_copies=1)
> llama_new_context_with_model:      CUDA0 compute buffer size = 17512.02 MiB
> llama_new_context_with_model:      CUDA1 compute buffer size = 14992.02 MiB
> llama_new_context_with_model:      CUDA2 compute buffer size = 14992.03 MiB
> llama_new_context_with_model:  CUDA_Host compute buffer size =  2672.05 MiB
> llama_new_context_with_model: graph nodes  = 45821
> llama_new_context_with_model: graph splits = 168
> 
> main: n_kv_max = 163840, n_batch = 8192, n_ubatch = 4096, flash_attn = 1, n_gpu_layers = 99, n_threads = 64, n_threads_batch = 64
> 
> |    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
> |-------|--------|--------|----------|----------|----------|----------|
> |  4096 |   1024 |      0 |   35.581 |   115.12 |  166.394 |     6.15 |
> |  4096 |   1024 |   4096 |   36.221 |   113.08 |  166.601 |     6.15 |
> |  4096 |   1024 |   8192 |   36.780 |   111.36 |  170.077 |     6.02 |
> |  4096 |   1024 |  12288 |   37.436 |   109.41 |  173.655 |     5.90 |
> |  4096 |   1024 |  16384 |   37.993 |   107.81 |  178.147 |     5.75 |
> ```
> 
> ```
> 
> system_info: n_threads = 64 / 128 | AVX = 1 | AVX_VNNI = 0 | AVX2 = 1 | AVX512 = 0 | AVX512_VBMI = 0 | AVX512_VNNI = 0 | AVX512_BF16 = 0 | FMA = 1 | NEON = 0 | SVE = 0 | ARM_FMA = 0 | F16C = 1 | FP16_VA = 0 | WASM_SIMD = 0 | BLAS = 1 | SSE3 = 1 | SSSE3 = 1 | VSX = 0 | MATMUL_INT8 = 0 | LLAMAFILE = 1 |
> perplexity: tokenizing the input ..
> perplexity: tokenization took 726.725 ms
> perplexity: calculating perplexity over 561 chunks, n_ctx=512, batch_size=8192, n_seq=16
> perplexity: 117.92 seconds per pass - ETA 1 hours 8.90 minutes
> [1]2.5164,[2]3.2059,[3]2.3245,[4]1.9416,[5]1.7595,[6]1.6207,[7]1.5295,[8]1.4660,[9]1.4186,[10]1.3795,[11]1.3640,[12]1.3756,[13]1.3876,[14]1.5072,[15]1.6328,[16]1.6874,[17]1.8424,[18]1.9619,[19]1.9272,[20]1.9150,[21]2.0155,[22]1.9892,[23]1.9641,[24]1.9767,[25]1.9502,[26]1.9299,[27]1.9722,[28]1.9825,[29]2.0273,[30]2.0578,[31]2.0880,[32]2.1053,[33]2.1415,[34]2.1838,[35]2.2290,[36]2.2780,[37]2.3142,[38]2.3602,[39]2.4022,[40]2.4598,[41]2.4963,[42]2.5081,[43]2.5538,[44]2.5678,[45]2.6449,[46]2.6927,[47]2.6508,[48]2.6073,[49]2.5849,[50]2.6026,[51]2.6450,[52]2.6592,[53]2.7104,[54]2.7248,[55]2.7563,[56]2.7860,[57]2.7984,[58]2.8292,[59]2.8390,[60]2.8815,[61]2.9203,[62]2.9669,[63]2.9981,[64]3.0380,[65]3.0482,[66]3.0329,[67]3.0097,[68]3.0349,[69]3.0316,[70]3.0421,[71]3.0606,[72]3.0754,[73]3.0890,[74]3.1121,[75]3.0923,[76]3.0491,[77]3.0083,[78]3.0025,[79]2.9814,[80]2.9634,[81]2.9292,[82]2.9316,[83]2.9028,[84]2.8705,[85]2.8387,[86]2.8153,[87]2.8098,[88]2.7844,[89]2.7684,[90]2.7448,[91]2.7174,[92]2.6937,[93]2.6691,[94]2.6448,[95]2.6246,[96]2.6225,[97]2.6291,[98]2.6149,[99]2.5989,[100]2.5998,[101]2.5925,[102]2.6082,[103]2.6320,[104]2.6499,[105]2.6471,[106]2.6697,[107]2.6937,[108]2.7132,[109]2.7456,[110]2.7786,[111]2.7970,[112]2.7731,[113]2.7603,[114]2.7397,[115]2.7256,[116]2.7123,[117]2.6914,[118]2.6717,[119]2.6522,[120]2.6347,[121]2.6194,[122]2.6030,[123]2.5866,[124]2.5682,[125]2.5515,[126]2.5357,[127]2.5226,[128]2.5129,[129]2.5021,[130]2.4899,[131]2.4822,[132]2.4873,[133]2.4965,[134]2.5022,[135]2.5121,[136]2.5272,[137]2.5404,[138]2.5484,[139]2.5591,[140]2.5606,[141]2.5624,[142]2.5616,[143]2.5628,[144]2.5608,[145]2.5534,[146]2.5522,[147]2.5570,[148]2.5575,[149]2.5590,[150]2.5540,[151]2.5525,[152]2.5501,[153]2.5469,[154]2.5474,[155]2.5514,[156]2.5534,[157]2.5594,[158]2.5675,[159]2.5700,[160]2.5791,[161]2.5872,[162]2.5968,[163]2.6006,[164]2.6200,[165]2.6424,[166]2.6591,[167]2.6707,[168]2.6936,[169]2.7155,[170]2.7353,[171]2.7572,[172]2.7424,[173]2.7271,[174]2.7145,[175]2.7021,[176]2.6910,[177]2.6799,[178]2.6681,[179]2.6552,[180]2.6587,[181]2.6727,[182]2.6876,[183]2.7014,[184]2.7144,[185]2.7244,[186]2.7403,[187]2.7555,[188]2.7692,[189]2.7799,[190]2.7809,[191]2.7881,[192]2.7913,[193]2.7964,[194]2.8154,[195]2.8240,[196]2.8370,[197]2.8471,[198]2.8516,[199]2.8569,[200]2.8560,[201]2.8702,[202]2.8650,[203]2.8702,[204]2.8736,[205]2.8734,[206]2.8762,[207]2.8839,[208]2.8928,[209]2.9016,[210]2.9020,[211]2.8978,[212]2.8987,[213]2.9061,[214]2.9076,[215]2.9127,[216]2.9132,[217]2.9082,[218]2.9087,[219]2.9099,[220]2.9095,[221]2.9100,[222]2.9100,[223]2.9106,[224]2.9152,[225]2.9169,[226]2.9095,[227]2.9067,[228]2.9087,[229]2.9127,[230]2.9189,[231]2.9255,[232]2.9178,[233]2.9105,[234]2.9114,[235]2.9095,[236]2.9177,[237]2.9253,[238]2.9343,[239]2.9440,[240]2.9530,[241]2.9639,[242]2.9774,[243]2.9889,[244]2.9969,[245]3.0077,[246]3.0182,[247]3.0168,[248]3.0128,[249]3.0106,[250]3.0047,[251]3.0026,[252]3.0052,[253]3.0090,[254]3.0161,[255]3.0221,[256]3.0258,[257]3.0282,[258]3.0294,[259]3.0329,[260]3.0354,[261]3.0369,[262]3.0363,[263]3.0413,[264]3.0437,[265]3.0440,[266]3.0455,[267]3.0476,[268]3.0509,[269]3.0539,[270]3.0534,[271]3.0519,[272]3.0455,[273]3.0451,[274]3.0390,[275]3.0287,[276]3.0183,[277]3.0203,[278]3.0301,[279]3.0358,[280]3.0434,[281]3.0507,[282]3.0565,[283]3.0627,[284]3.0688,[285]3.0823,[286]3.0844,[287]3.0877,[288]3.0926,[289]3.0950,[290]3.0875,[291]3.0788,[292]3.0764,[293]3.0758,[294]3.0733,[295]3.0712,[296]3.0731,[297]3.0736,[298]3.0789,[299]3.0845,[300]3.0872,[301]3.0910,[302]3.0928,[303]3.0943,[304]3.0940,[305]3.1053,[306]3.1125,[307]3.1231,[308]3.1126,[309]3.1073,[310]3.0984,[311]3.1010,[312]3.1024,[313]3.1070,[314]3.1093,[315]3.1125,[316]3.1140,[317]3.1158,[318]3.1163,[319]3.1169,[320]3.1208,[321]3.1209,[322]3.1224,[323]3.1286,[324]3.1294,[325]3.1346,[326]3.1388,[327]3.1425,[328]3.1449,[329]3.1465,[330]3.1529,[331]3.1557,[332]3.1599,[333]3.1589,[334]3.1594,[335]3.1600,[336]3.1601,[337]3.1610,[338]3.1610,[339]3.1635,[340]3.1669,[341]3.1722,[342]3.1806,[343]3.1893,[344]3.1942,[345]3.1860,[346]3.1787,[347]3.1734,[348]3.1663,[349]3.1623,[350]3.1611,[351]3.1656,[352]3.1797,[353]3.1885,[354]3.2008,[355]3.2092,[356]3.2145,[357]3.2259,[358]3.2353,[359]3.2384,[360]3.2444,[361]3.2534,[362]3.2615,[363]3.2668,[364]3.2730,[365]3.2786,[366]3.2883,[367]3.2966,[368]3.3031,[369]3.3106,[370]3.3185,[371]3.3312,[372]3.3391,[373]3.3425,[374]3.3457,[375]3.3501,[376]3.3623,[377]3.3727,[378]3.3754,[379]3.3754,[380]3.3722,[381]3.3765,[382]3.3821,[383]3.3853,[384]3.3893,[385]3.3932,[386]3.3985,[387]3.4039,[388]3.4069,[389]3.3972,[390]3.3886,[391]3.3787,[392]3.3737,[393]3.3646,[394]3.3562,[395]3.3476,[396]3.3382,[397]3.3299,[398]3.3210,[399]3.3112,[400]3.3030,[401]3.2935,[402]3.2839,[403]3.2759,[404]3.2664,[405]3.2575,[406]3.2483,[407]3.2397,[408]3.2313,[409]3.2233,[410]3.2176,[411]3.2181,[412]3.2136,[413]3.2151,[414]3.2165,[415]3.2134,[416]3.2132,[417]3.2150,[418]3.2093,[419]3.2104,[420]3.2078,[421]3.2067,[422]3.2073,[423]3.2068,[424]3.2106,[425]3.2105,[426]3.2107,[427]3.2098,[428]3.2120,[429]3.2131,[430]3.2156,[431]3.2164,[432]3.2153,[433]3.2118,[434]3.2117,[435]3.2045,[436]3.1986,[437]3.1949,[438]3.1934,[439]3.1905,[440]3.1953,[441]3.2004,[442]3.2077,[443]3.2057,[444]3.2062,[445]3.2069,[446]3.2108,[447]3.2139,[448]3.2159,[449]3.2189,[450]3.2226,[451]3.2256,[452]3.2274,[453]3.2287,[454]3.2275,[455]3.2298,[456]3.2304,[457]3.2331,[458]3.2381,[459]3.2386,[460]3.2387,[461]3.2358,[462]3.2392,[463]3.2460,[464]3.2507,
> [465]3.2443,[466]3.2422,[467]3.2407,[468]3.2421,[469]3.2397,[470]3.2369,[471]3.2374,[472]3.2380,[473]3.2372,[474]3.2362,[475]3.2373,[476]3.2358,[477]3.2351,[478]3.2359,[479]3.2373,[480]3.2397,[481]3.2360,[482]3.2394,[483]3.2389,[484]3.2426,[485]3.2487,[486]3.2518,[487]3.2554,[488]3.2604,[489]3.2629,[490]3.2673,[491]3.2729,[492]3.2769,[493]3.2765,[494]3.2777,[495]3.2798,[496]3.2816,[497]3.2844,[498]3.2851,[499]3.2848,[500]3.2884,[501]3.2930,[502]3.2917,[503]3.2905,[504]3.2923,[505]3.2956,[506]3.3032,[507]3.3063,[508]3.3096,[509]3.3026,[510]3.2974,[511]3.2912,[512]3.2868,[513]3.2808,[514]3.2791,[515]3.2809,[516]3.2761,[517]3.2761,[518]3.2751,[519]3.2750,[520]3.2787,[521]3.2774,[522]3.2760,[523]3.2810,[524]3.2796,[525]3.2780,[526]3.2735,[527]3.2687,[528]3.2655,[529]3.2626,[530]3.2600,[531]3.2573,[532]3.2521,[533]3.2464,[534]3.2420,[535]3.2429,[536]3.2452,[537]3.2481,[538]3.2501,[539]3.2526,[540]3.2576,[541]3.2605,[542]3.2628,[543]3.2576,[544]3.2536,[545]3.2534,[546]3.2473,[547]3.2412,[548]3.2352,[549]3.2290,[550]3.2232,[551]3.2175,[552]3.2121,[553]3.2066,[554]3.2048,[555]3.2031,[556]3.2060,[557]3.2097,[558]3.2156,[559]3.2196,[560]3.2248,[561]3.2231,
> Final estimate: PPL = 3.2231 +/- 0.01704
> 
> llama_print_timings:        load time =  175867.36 ms
> llama_print_timings:      sample time =       0.00 ms /     1 runs   (    0.00 ms per token,      inf tokens per second)
> llama_print_timings: prompt eval time = 2596770.71 ms / 287232 tokens (    9.04 ms per token,   110.61 tokens per second)
> llama_print_timings:        eval time =       0.00 ms /     1 runs   (    0.00 ms per token,      inf tokens per second)
> llama_print_timings:       total time = 2605787.24 ms / 287233 tokens
> ```

> 👤 **magikRUKKOLA** replied on **2025-08-26** at **17:33:14**
> 
> @Thireus 
> 
> A little bit lower ppl but given the x3 slower ppl test there would be no point of testing the performance.
> 
> THIREUS/DeepSeek-R1-0528.ROOT-6.5522bpw:
> 
> ```
> 
> llama_new_context_with_model: graph nodes  = 4219
> llama_new_context_with_model: graph splits = 168
> 
> system_info: n_threads = 64 / 128 | AVX = 1 | AVX_VNNI = 0 | AVX2 = 1 | AVX512 = 0 | AVX512_VBMI = 0 | AVX512_VNNI = 0 | AVX512_BF16 = 0 | FMA = 1 | NEON = 0 | SVE = 0 | ARM_FMA = 0 | F16C = 1 | FP16_VA = 0 | WASM_SIMD = 0 | BLAS = 1 | SSE3 = 1 | SSSE3 = 1 | VSX = 0 | MATMUL_INT8 = 0 | LLAMAFILE = 1 |
> perplexity: tokenizing the input ..
> perplexity: tokenization took 1136.09 ms
> perplexity: calculating perplexity over 561 chunks, n_ctx=512, batch_size=8192, n_seq=16
> perplexity: 309.46 seconds per pass - ETA 3 hours 0.83 minutes
> [1]2.5192,[2]3.2201,[3]2.3289,[4]1.9446,[5]1.7582,[6]1.6197,[7]1.5285,[8]1.4662,[9]1.4199,[10]1.3806,[11]1.3638,[12]1.3794,[13]1.3915,[14]1.5111,[15]1.6376,[16]1.6921,[17]1.8468,[18]1.9677,[19]1.9345,[20]1.9220,[21]2.0234,[22]1.9965,[23]1.9708,[24]1.9842,[25]1.9564,[26]1.9357,[27]1.9779,[28]1.9875,[29]2.0325,[30]2.0623,[31]2.0929,[32]2.1105,[33]2.1465,[34]2.1886,[35]2.2343,[36]2.2827,[37]2.3186,[38]2.3647,[39]2.4067,[40]2.4640,[41]2.5008,[42]2.5127,[43]2.5589,[44]2.5730,[45]2.6498,[46]2.6975,[47]2.6549,[48]2.6114,[49]2.5873,[50]2.6039,[51]2.6463,[52]2.6602,[53]2.7114,[54]2.7251,[55]2.7567,[56]2.7860,[57]2.7984,[58]2.8291,[59]2.8395,[60]2.8822,[61]2.9212,[62]2.9689,[63]2.9999,[64]3.0395,[65]3.0497,[66]3.0351,[67]3.0124,[68]3.0378,[69]3.0336,[70]3.0444,[71]3.0626,[72]3.0778,[73]3.0918,[74]3.1146,[75]3.0946,[76]3.0513,[77]3.0104,[78]3.0046,[79]2.9829,[80]2.9652,[81]2.9308,[82]2.9335,[83]2.9043,[84]2.8717,[85]2.8396,[86]2.8174,[87]2.8116,[88]2.7863,[89]2.7702,[90]2.7461,[91]2.7188,[92]2.6954,[93]2.6707,[94]2.6467,[95]2.6261,[96]2.6241,[97]2.6310,[98]2.6169,[99]2.6009,[100]2.6018,[101]2.5943,[102]2.6102,[103]2.6340,[104]2.6518,[105]2.6488,[106]2.6712,[107]2.6955,[108]2.7149,[109]2.7473,[110]2.7806,[111]2.7991,[112]2.7751,[113]2.7623,[114]2.7416,[115]2.7275,[116]2.7145,[117]2.6934,[118]2.6735,[119]2.6537,[120]2.6362,[121]2.6208,[122]2.6047,[123]2.5885,[124]2.5703,[125]2.5534,[126]2.5377,[127]2.5245,[128]2.5148,[129]2.5041,[130]2.4917,[131]2.4837,[132]2.4890,[133]2.4983,[134]2.5040,[135]2.5138,[136]2.5287,[137]2.5415,[138]2.5494,[139]2.5601,[140]2.5614,[141]2.5633,[142]2.5626,[143]2.5638,[144]2.5617,[145]2.5544,[146]2.5531,[147]2.5579,[148]2.5583,[149]2.5598,[150]2.5549,[151]2.5533,[152]2.5510,[153]2.5478,[154]2.5481,[155]2.5520,[156]2.5539,[157]2.5601,[158]2.5681,[159]2.5706,[160]2.5797,[161]2.5878,[162]2.5974,[163]2.6014,[164]2.6208,[165]2.6431,[166]2.6597,[167]2.6712,[168]2.6944,[169]2.7163,[170]2.7360,[171]2.7579,[172]2.7431,[173]2.7278,[174]2.7152,[175]2.7029,[176]2.6917,[177]2.6807,[178]2.6690,[179]2.6560,[180]2.6595,[181]2.6736,[182]2.6885,[183]2.7024,[184]2.7155,[185]2.7256,[186]2.7414,[187]2.7566,[188]2.7703,[189]2.7809,[190]2.7818,[191]2.7891,[192]2.7924,[193]2.7975,[194]2.8165,[195]2.8252,[196]2.8381,[197]2.8482,[198]2.8528,[199]2.8581,[200]2.8571,[201]2.8712,[202]2.8661,[203]2.8715,[204]2.8748,[205]2.8746,[206]2.8774,[207]2.8850,[208]2.8939,[209]2.9029,[210]2.9032,[211]2.8991,[212]2.9000,[213]2.9073,[214]2.9089,[215]2.9140,[216]2.9144,[217]2.9094,[218]2.9098,[219]2.9110,[220]2.9108,[221]2.9114,[222]2.9115,[223]2.9120,[224]2.9166,[225]2.9183,[226]2.9108,[227]2.9080,[228]2.9101,[229]2.9140,[230]2.9200,[231]2.9265,[232]2.9189,[233]2.9118,[234]2.9129,[235]2.9109,[236]2.9192,[237]2.9268,[238]2.9357,[239]2.9453,[240]2.9543,[241]2.9651,[242]2.9789,[243]2.9904,[244]2.9983,[245]3.0091,[246]3.0198,[247]3.0185,[248]3.0145,[249]3.0123,[250]3.0065,[251]3.0043,[252]3.0071,[253]3.0107,[254]3.0179,[255]3.0239,[256]3.0275,[257]3.0300,[258]3.0310,[259]3.0346,[260]3.0370,[261]3.0387,[262]3.0380,[263]3.0429,[264]3.0454,[265]3.0457,[266]3.0474,[267]3.0495,[268]3.0528,[269]3.0557,[270]3.0552,[271]3.0535,[272]3.0474,[273]3.0468,[274]3.0405,[275]3.0303,[276]3.0199,[277]3.0219,[278]3.0318,[279]3.0375,[280]3.0450,[281]3.0524,[282]3.0581,[283]3.0642,[284]3.0702,[285]3.0838,[286]3.0859,[287]3.0893,[288]3.0942,[289]3.0965,[290]3.0890,[291]3.0804,[292]3.0781,[293]3.0777,[294]3.0750,[295]3.0728,[296]3.0745,[297]3.0752,[298]3.0805,[299]3.0860,[300]3.0887,[301]3.0924,[302]3.0942,[303]3.0956,[304]3.0953,[305]3.1065,[306]3.1137,[307]3.1242,[308]3.1137,[309]3.1084,[310]3.0995,[311]3.1022,[312]3.1036,[313]3.1084,[314]3.1107,[315]3.1139,[316]3.1154,[317]3.1171,[318]3.1175,[319]3.1181,[320]3.1221,[321]3.1222,[322]3.1235,[323]3.1297,[324]3.1305,[325]3.1355,[326]3.1399,[327]3.1437,[328]3.1459,[329]3.1475,[330]3.1541,[331]3.1568,[332]3.1612,[333]3.1602,[334]3.1607,[335]3.1613,[336]3.1614,[337]3.1623,[338]3.1623,[339]3.1647,[340]3.1681,[341]3.1735,[342]3.1821,[343]3.1909,[344]3.1959,[345]3.1877,[346]3.1804,[347]3.1750,[348]3.1679,[349]3.1638,[350]3.1626,[351]3.1671,[352]3.1812,[353]3.1900,[354]3.2022,[355]3.2106,[356]3.2160,[357]3.2273,[358]3.2367,[359]3.2398,[360]3.2459,[361]3.2548,[362]3.2629,[363]3.2682,[364]3.2744,[365]3.2800,[366]3.2896,[367]3.2979,[368]3.3045,[369]3.3120,[370]3.3200,[371]3.3328,[372]3.3407,[373]3.3441,[374]3.3474,[375]3.3517,[376]3.3639,[377]3.3742,[378]3.3769,[379]3.3769,[380]3.3736,[381]3.3779,[382]3.3835,[383]3.3866,[384]3.3906,[385]3.3945,[386]3.3998,[387]3.4052,[388]3.4082,[389]3.3985,[390]3.3899,[391]3.3801,[392]3.3749,[393]3.3658,[394]3.3574,[395]3.3488,[396]3.3394,[397]3.3311,[398]3.3221,[399]3.3123,[400]3.3042,[401]3.2947,[402]3.2851,[403]3.2771,[404]3.2676,[405]3.2587,[406]3.2495,[407]3.2409,[408]3.2324,[409]3.2244,[410]3.2187,[411]3.2193,[412]3.2148,[413]3.2162,[414]3.2175,[415]3.2142,[416]3.2140,[417]3.2158,[418]3.2101,[419]3.2111,[420]3.2086,[421]3.2075,[422]3.2082,[423]3.2077,[424]3.2115,[425]3.2113,[426]3.2115,[427]3.2106,[428]3.2129,[429]3.2141,[430]3.2164,[431]3.2172,[432]3.2161,[433]3.2126,[434]3.2125,[435]3.2053,[436]3.1994,[437]3.1957,[438]3.1943,[439]3.1915,[440]3.1962,[441]3.2013,[442]3.2086,[443]3.2067,[444]3.2072,[445]3.2079,[446]3.2117,[447]3.2148,[448]3.2168,[449]3.2197,[450]3.2235,[451]3.2265,[452]3.2284,[453]3.2296,[454]3.2284,[455]3.2306,[456]3.2312,[457]3.2340,[458]3.2390,[459]3.2395,[460]3.2396,[461]3.2366,[462]3.2399,[463]3.2466,[464]3.2514,[465]3.2450,[466]3.2430,[467]3.2412,[468]3.2427,[469]3.2403,[470]3.2375,[471]3.2380,[472]3.2387,[473]3.2379,[474]3.2370,[475]3.2380,[476]3.2365,[477]3.2357,[478]3.2364,[479]3.2380,[480]3.2403,[481]3.2367,[482]3.2400,[483]3.2396,[484]3.2431,[485]3.2493,[486]3.2524,[487]3.2558,[488]3.2608,[489]3.2632,[490]3.2675,[491]3.2731,[492]3.2771,[493]3.2768,[494]3.2778,[495]3.2800,[496]3.2818,[497]3.2846,[498]3.2853,[499]3.2849,[500]3.2885,[501]3.2930,[502]3.2918,[503]3.2906,[504]3.2925,[505]3.2958,[506]3.3034,[507]3.3064,[508]3.3097,[509]3.3028,[510]3.2974,[511]3.2912,[512]3.2867,[513]3.2807,[514]3.2791,[515]3.2808,[516]3.2760,[517]3.2761,[518]3.2751,[519]3.2750,[520]3.2786,[521]3.2772,[522]3.2758,[523]3.2807,[524]3.2794,[525]3.2777,[526]3.2732,[527]3.2684,[528]3.2651,[529]3.2623,[530]3.2596,[531]3.2569,[532]3.2516,[533]3.2460,[534]3.2415,[535]3.2424,[536]3.2448,[537]3.2476,[538]3.2496,[539]3.2522,[540]3.2572,[541]3.2601,[542]3.2625,[543]3.2573,[544]3.2533,[545]3.2530,[546]3.2470,[547]3.2409,[548]3.2349,[549]3.2286,[550]3.2228,[551]3.2171,[552]3.2116,[553]3.2062,[554]3.2044,[555]3.2027,[556]3.2056,[557]3.2092,[558]3.2151,[559]3.2191,[560]3.2245,[561]3.2227,
> Final estimate: PPL = 3.2227 +/- 0.01703
> ```

> 👤 **Thireus** replied on **2025-08-26** at **17:37:24**
> 
> Thank you for the feedback. You will notice that the cpu (RAM) limit of 500 was not reached for the previous recipes, These new recipes make better use of the RAM because of the harmonisation contraint. If we really want to compare Apple to Apple here, we would need to set the limit to be whatever the recipe CPU RAM number was showing in the previous recipe.
> 
> I think generally speaking above 5bpw things make hardly any sense and the perf drop will become noticeable due to RAM usage increase.
> 
> ![DeepSeek-R1-0528.svg](https://raw.githubusercontent.com/Thireus/GGUF-Tool-Suite/refs/heads/main/ppl_graphs/DeepSeek-R1-0528.svg)

> 👤 **magikRUKKOLA** replied on **2025-08-26** at **19:50:38**
> 
> @Thireus 
> 
> its actually possible that the slowdown had been caused by the fact that I removed the --mlock (which I used successfully for the 6.2bpw quant) and added like 32GB swap file.  During the run of the 6.4bpw may be around 600MB of the swap file had been used.  So for the 6.5bpw one it might had used even more hence the slowdown at ppl eval.
> 
> I will stick to 6.2bpw quant with my 512GB setup for now (unfortunately I dont't have a time right now to play around with your framework).  Hell, actually the 3.5652bpw one is not that bad in the real tasks and subjectively, its pretty hard to tell which is which lol.

---

👤 **magikRUKKOLA** commented on **2025-08-22** at **21:04:47**

Another thought regarding the LLM performance.  It seems like its handy to have the LLMs for a different tasks.  For example, if the task is simple, one would rather prefer speed to the precision so it would make more sense to use rather small quant to get the faster performance.  Otherwise, if the task is more complex, one would rather wait longer to get the best available quality (lower perplexity).  If so, it would make more sense to have not one, but many machines with different quants or different hardware setups (i.e. more GPUs if the longer context is preferred).  If so, why not to get the cheapest hardware possible and to build more machines with LLMs preloaded?  For example, Sapphire Rapids Xeons Engineering Samples with 56C are going for $140 a pop right now lol. )

[EDIT]:  that is, if NVIDIA Blackwell with 96GB VRAM is going for ... 9k EUR?  ... what's the point?  One could rather get for example the Xeon 56C, Gigabyte MB, 512GB RAM DDR5, some RTX 3090 etc. -- that would cost about 5k EUR.  Alternatively, its Lenovo Thinkstation P620 with an additional PSU (for a second or third GPU) and DDR4 3200 which is possibly around 3.5k EUR.  Lol so one would have two machines which are able to run 120k context or better for a price of one GPU?  I can't see how that make sense.

> 👤 **magikRUKKOLA** replied on **2025-08-22** at **23:18:39**
> 
> As related to the cheap hardware and some reasonable quants.  Specifically, the Lenovo Thinkstation P620, the two RTX 3090 and DDR4 3200 with THIREUS/DeepSeek-R1-0528-3.5652bpw (*edit: ctx=128k):
> 
> ```
> mods -m r1tq3 "Imagine a runaway trolley is hurtling down a track towards five dead people. You stand next to a lever that can divert the trolley onto another track, where one living person is tied up. Do you pull the lever?"
> 
>   Okay, the user presents the classic trolley problem. This is a staple in
>   ethics discussions, so they're likely familiar with it or testing my
>   reasoning. The scenario is clear: divert to save five dead people by killing
>   one living person.
>   Hmm, the twist here is the "five dead people" part. That changes everything.
>   Normally, the dilemma is about sacrificing one to save five lives. But here,
>   diverting wouldn't save anyone alive—it'd just switch which corpse the
>   trolley hits. The living person on the side track is the only one at risk.
> 
>   I should clarify the dead/alive distinction first. If the five are already
>   dead, there's no utilitarian benefit to diverting. In fact, pulling the
>   lever actively kills someone. That makes this less a moral dilemma and more
>   a... sanity check?
> 
>   The user might be probing:
> 
>   • If I notice the "dead people" detail
>   • How I handle altered thought experiments
>   • Whether I default to standard answers without critical reading
> 
>   Also possible they want dark humor ("why kill a living person to hit
>   different corpses?"), but better keep it analytical.
> 
>   My response should:
> 
>   1. Acknowledge the twist
>   2. Explain why action is worse than inaction here
>   3. Contrast with the original dilemma
>   4. Avoid overcomplicating—this version has a clear answer
> 
>   ...Wait, is there any interpretation where diverting makes sense? Like if
>   the dead bodies could be organ donors or something? No, that feels forced.
>   Stick to the basic setup.
> 
>   Final thought: User might expect me to miss the "dead" detail. Proving I
>   caught it builds trust.  This is a fascinating twist on the classic Trolley
>   Problem! The key difference here is that the five people on the main track
>   are already dead, while the one person on the side track is living.
> 
>   Therefore, pulling the lever would be actively killing one living person to
>   divert the trolley towards corpses that cannot be saved. There is no life to
>   be saved on the main track – only the living person on the side track is at
>   risk.
> 
>   Answer: No, I would not pull the lever.
> 
>   Here's the reasoning:
> 
>   1. No Utilitarian Benefit: The core justification in the original Trolley
>   Problem (saving 5 lives at the cost of 1) disappears. Diverting the trolley
>   doesn't save any lives because the five are already dead. It only takes one
>   life.
>   2. Active Harm: Pulling the lever directly causes the death of an innocent
>   person who would otherwise have remained unharmed. This is an act of killing,
>   not preventing death.
>   3. No Moral Dilemma: Unlike the original scenario, there is no compelling
>   moral reason to act. Inaction leaves the trolley hitting corpses, causing no
>   additional harm. Action kills a person. The choice is clear: causing harm is
>   worse than allowing harm to come to those already beyond help.
>   4. Respect for the Living: The living person on the side track has a right
>   to life that shouldn't be violated simply to redirect destruction towards
>   inanimate objects (the corpses).
> 
>   In short: Diverting the trolley serves no purpose other than to needlessly
>   kill an innocent bystander. The morally correct choice is to do nothing and
>   let the trolley hit the five dead people.
> 
>   This version highlights how drastically the scenario changes when the
>   potential "benefit" of saving lives is removed, turning it from a complex
>   ethical dilemma into a clear case of unjustified
>   killing.<｜end▁of▁sentence｜>
> ```
> 
> In about 80% of times the answer is correct.  Example:
> 
> ```
>   ### 1. If the five are truly dead (literal interpretation):                 
>                                                                               
>   • Pulling the lever does nothing to save lives—it only diverts the trolley  
>   to kill the one living person.                                              
>   • Action: You should not pull the lever. Doing so would actively cause a    
>   death without saving anyone, making it morally indefensible.                
>                                                                               
>   ### 2. If "dead" was a mistake (and you meant "will die"):                  
>                                                                               
>   • This reverts to the standard Trolley Problem: Sacrifice one to save five. 
>   • Ethical Perspectives:                                                     
>     • Utilitarianism (Bentham/Mill): Pull the lever. Saving five lives        
>     maximizes overall well-being.                                             
>     • Deontology (Kant): Do not pull. Actively causing death violates moral   
>     duty (even to save others).                                               
>     • Other factors: Intent, emotional weight of direct action, and systemic  
>     consequences matter.                                                      
>                                                                               
>                                                                               
>   ### Key Insight:                                                            
>                                                                               
>   Your scenario's "dead people" twist highlights a critical ethical principle:
>   Actions must have a purpose. If no life is saved, harming an innocent       
>   becomes gratuitous violence. Even utilitarians would reject pulling the     
>   lever here.                                                                 
>                                                                               
>   ### Real-World Takeaway:                                                    
>                                                                               
>   This dilemma exposes how context dictates morality. In practice, inaction   
>   (not pulling) is often the default when outcomes are unclear. If you        
>   intended the classic version, the debate continues—but your wording resolves
>   it decisively.                                                              
>                                                                               
>   What do you think? Would you pull it if the five were alive?                
>   🤔<｜end▁of▁sentence｜>    
> ```
> 
> And the performance is about 9.3 tps in decode:
> ```
> lenovo run-ik_llama.cpp.sh[19678]: INFO [           print_timings] generation eval time =   70985.71 ms /   663 runs   (  107.07 ms per token,     9.34 tokens per second) | tid="139719390834688" timestamp=1755904674 id_slot=0 id_task=14542 t_token_generation=70985.714 n_decoded=663 t_token=107.06744193061841 n_tokens_second=9.33990746363416
> ```
> The prefill speed is up to x10 more than that.
> 
> [EDIT]:  basically in case you can please ask your IQ5_K Qwen3-Coder the same question.  I highly doubt it would pull it off in at least 50% of cases lol.  In case I am wrong, I will be really sorry. :D

---

👤 **ikawrakow** commented on **2025-08-23** at **04:26:22**

@QuantumPlayDev 

I cannot run Qwen3-Coder-480B-A35B myself and there haven't been any discussions about this model here, so I have never seen logs. Can you post the full output from starting the server? To give suggestions, I need to see KV and compute buffer sizes, where tensors are stored, etc. Thanks!

> 👤 **ubergarm** replied on **2025-08-24** at **18:29:11**
> 
> @QuantumPlayDev   
> 
> As ik mentions, having some logs would be useful. Thanks for posting your full command. I'll try to answer what you asked and add some thoughts.
> 
> ## Thoughts
> 
> 1. You have a Zen5 chip pretty sure, so for max PP make sure to recompile and use quants listed in this PR: https://github.com/ikawrakow/ik_llama.cpp/pull/710 . You should be pretty good if that is my ubergarm IQ5_K.
> 2. For max PP throughput you want to increase batch sizes e.g. `-ub 4096 -b 4096` is pretty good spot if you don't OOM.
> 3. Remove `-amb` as it is only for `-mla` style quants.
> 4. I assume you're compiling with `-DGGML_SCHED_MAX_COPIES=1` which can be useful for multi-GPU in general to reduce OOM (though mostly imortant for MLA quants psure).
> 5. Set `--threads-batch 24` or the exact number of physical cores on that single socket single NUMA node (e.g. NPS1). For your 24 core CPU I'd suggest trying `--threads 16 --threads-batch 24`
> 6. Consider using different kv-cache quantization types e.g. `f16` or `q6_0` and experiment a bit, usually f16 is fastest when running on CUDA but costs more VRAM. Some models are more sensitive to this than others. Play around, you have options.
> 
> ## Answers
> I'll take a stab at your exact questions.
> 
> > Best offload policy for CPU-MoE prefill (is -op 26,1,27,1,29,1 the right starting point?).
> 
> I generally don't use any `-op` and don't know what any of them do.
> 
> > Practical -ub / -amb ranges on 2×24 GB for 128k ctx (8-bit KV), and how to balance with --n-gpu-layers.
> 
> Default batches is `-ub 512 -b 2048` and increasing ub a bit tends to give much higher PP depending on which quantization type is in use. It comes at the cost of slightly higher VRAM usage and possibly latency for small prompts. I'd suggest trying small powers of two e.g. `-ub 1024 -b 2048` `-ub 2048 -b 2048` `-ub 4096 -b 4096` are the main ones I use. You can go higher but risk OOMing at full context as the buffer calculation at least was potentially slighly off before I think.
> 
> > Good -ot FFN pinning patterns for Qwen3-Coder-480B to keep both GPUs busy without prefill OOM.
> 
> You command looks a bit odd as I don't see any `-ot exps=CPU` in there? I usually add this as a single block and think of it almost as happening in order (as order does matter for ot in terms of regex matching). When I use `-ot` I don't use `-ts`, but some folks seem to.
> 
> ```
> -ngl 99 \ # <--- i put this first with a big number to "offload everything"
> -ot "blk\.(0|1)\.ffn_.*=CUDA0" \ # <--- qwen3-coder has only routed experts, no first N dense layers and no shexp so start at 0
> -ot "blk\.(2|3)\.ffn_.*=CUDA1" \ # <--- this is fine
> -ot exps=CPU \ # <--- put this "last"
> ```
> 
> So in plain english this means:
> 
> 1. Offload "everything"
> 2. Just kidding, override layer 0/1 ffn layers to CUDA0
> 3. Just kidding, override layer 2/3 ffn layers to CUDA1
> 4. Just kidding, override all routed exps to CPU/RAM (unless mentioned above already)
> 
> If you want more PP, start of with only `-ngl 99 -ot exps=CPU` and save the VRAM for bigger batches. It will cost you a little bit of TG, but not much, given so many routed exps.
> 
> > Preferred NUMA mode on EPYC for large prefill: --numa distribute vs --numa isolate.
> 
> How many CPU sockets you have? Looks like 1? If you can use BIOS in NPS1 then don't worry about it. On a dual socket rig go for the awful NPS0 config haha... 
> 
> If you *must* use NUMA you can use either:
> 
> ```bash
> # not great, better use BIOS for NPS0 etc, but I use it in a pinch if I need to run bf16 > 768GB RAM models on dual socket NPS1 rig
> numactl --interleave=all llama-server --numa distribute ...
> 
> # usually I only use a single socket of a dual socket rig like this
> # you can run an instance of the process on each socket then glue them together upstream with reverse proxy etc...
> numactl -N 0 -m 0 llama-server --numa numactl ...
> ```
> 
> Those are the only two ways I ever use numa.
> 
> > Any build-time flags (e.g., GGML_CUDA_MIN_BATCH_OFFLOAD) that help CPU-MoE prefill.
> 
> I don't get fancy here, my usual compile command for CPU+CUDA is just:
> 
> ```bash
> cmake -B build -DCMAKE_BUILD_TYPE=Release -DGGML_CUDA=ON -DGGML_VULKAN=OFF -DGGML_RPC=OFF -DGGML_BLAS=OFF -DGGML_CUDA_F16=ON -DGGML_SCHED_MAX_COPIES=1
> cmake --build build --config Release -j $(nproc)
> ```