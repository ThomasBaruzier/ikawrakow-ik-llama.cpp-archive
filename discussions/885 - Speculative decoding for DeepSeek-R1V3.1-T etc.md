## 🗣️ [Discussion #885](https://github.com/ikawrakow/ik_llama.cpp/discussions/885) - Speculative decoding for DeepSeek-R1/V3.1-T etc.

| **Author** | `magikRUKKOLA` |
| :--- | :--- |
| **State** | ✅ **Open** |
| **Created** | 2025-11-01 |
| **Updated** | 2025-11-02 |

---

## 📄 Description

@ikawrakow 

Since there is no decent draft models for the DeepSeek-R1/V3.1-T etc. I was thinking to run very small quant like IQ1_T and the target quant (say, 4bpw+) simultaneously.

The problem is there there seems to be no options to make the draft model to be partially offloaded to the RAM(CPU).

Can we add this feature perhaps?  I believe the acceptance rate should be 90% and above.  And if the draft model is sufficiently fast, there could be a decent performance boost.  Right?

[1] https://github.com/ggml-org/llama.cpp/blob/bea04522ff1a0d8559ccfd353aa018dcfbb608cc/tools/server/README.md?plain=1#L170

```
| `--n-cpu-moe-draft, -ncmoed N` | keep the Mixture of Experts (MoE) weights of the first N layers in the CPU for the draft model<br/>(env: LLAMA_ARG_N_CPU_MOE_DRAFT) |
```

---

## 💬 Discussion

👤 **ikawrakow** commented on **2025-11-01** at **10:19:45**

Didn't jukofyork have various draft models for DeepSeek? Here is one: https://huggingface.co/jukofyork/DeepSeek-V3-DRAFT-0.6B-v3.0-GGUF. I haven't followed this closely, but I had the impression that he did put quite some effort into these draft models.

But if you want to go the `IQ1_KT` route, keep in mind that the `*_KT` quants tend to be slow for TG when running on the CPU, especially on older CPU's such as your 3995WX. They are fast on CUDA and fast on the CPU for PP, but slower than other quants for TG on older CPUs.

> 👤 **magikRUKKOLA** replied on **2025-11-01** at **13:59:08**
> 
> @ikawrakow 
> > Didn't jukofyork have various draft models for DeepSeek? Here is one: https://huggingface.co/jukofyork/DeepSeek-V3-DRAFT-0.6B-v3.0-GGUF. 
> 
> Not suitable.  The acceptance rate is like 40-50% at best.  The decode speed of the target drops in this case.  So the acceptance rate have to be improved in order to pursue real gains.
> 
> > But if you want to go the `IQ1_KT` route, keep in mind that the `*_KT` quants tend to be slow for TG when running on the CPU
> 
> Ok, cool.  Noted.
> But still, can the `--n-cpu-moe-draft` etc. be ported in order to check if running 671b+ MoE will be faster?
> 
> Is there any way to know beforehand?  Like how to capture the amount of time the target models spends on verification of the draft?

> 👤 **ikawrakow** replied on **2025-11-01** at **14:09:35**
> 
> > But still, can the --n-cpu-moe-draft etc. be ported in order to check if running 671b+ MoE will be faster?
> 
> Yes, it can be. If someone is generous enough to make the draft model a fully emancipated citizen. As it stands, people have added a few command line parameters to control some aspects of the draft model. What needs to happen instead is that **all** draft model parameters can be controlled via command line arguments just the same as the main model. I was thinking that one should add in analogy to compilers passing arguments to linkers something like
> ```
> --draft,arg_name,arg_value
> ```
> In that framework, setting tensor overrides would be
> ```
> --draft,-ot,"exps=CPU"
> ```
> which will that get converted to a pair of arguments `-ot` and `exps=CPU` that only apply when loading the draft model.
> 
> It is not that it is hard, just someone needs to do it.

> 👤 **magikRUKKOLA** replied on **2025-11-01** at **14:25:11**
> 
> @ikawrakow 
> > What needs to happen instead is that all draft model parameters can be controlled via command line arguments just the same as the main model.
> 
> Yeah, this is exactly the proper solution.  +1

---

👤 **magikRUKKOLA** commented on **2025-11-01** at **14:07:23**

**note1**:

https://www.together.ai/blog/adaptive-learning-speculator-system-atlas
```
Figure 4: We train Qwen/Qwen2.5-7B-Instruct-1M on DeepScaler subsets using ATLAS for RL on NVIDIA Hopper H100 GPUs. The acceptance rate rises from below 10% to above 80% over 1.4k training steps, resulting in a more than 60% reduction for overall training time without changing RL training algorithm.
```

> 👤 **magikRUKKOLA** replied on **2025-11-01** at **20:12:17**
> 
> **note2**:
> 
> <details>
> https://huggingface.co/agentica-org/DeepCoder-14B-Preview#evaluation
> ```
> Model | LCB (v5)(8/1/24-2/1/25) | Codeforces Rating | Codeforces Percentile | HumanEval+
> -- | -- | -- | -- | --
> DeepCoder-14B-Preview (ours) | 60.6 | 1936 | 95.3 | 92.6
> DeepSeek-R1-Distill-Qwen-14B | 53.0 | 1791 | 92.7 | 92.0
> O1-2024-12-17 (Low) | 59.5 | 1991 | 96.1 | 90.8
> O3-Mini-2025-1-31 (Low) | 60.9 | 1918 | 94.9 | 92.6
> O1-Preview | 42.7 | 1658 | 88.5 | 89
> Deepseek-R1 | 62.8 | 1948 | 95.4 | 92.6
> Llama-4-Behemoth | 49.4 | - | - | -
> ```
> </details>
> 
> ~~TODO: I should try DeepCoder-14B for the DeepSeek-R1-0528 inference.~~
> Acceptance rate of 25% for the DeepSeek-R1-0528?  lol.  unusable.

---

👤 **magikRUKKOLA** commented on **2025-11-02** at **00:33:50**

**note 3:**

I am going to have to build a separate machine specifically for the draft model.  I am going to use 7 x RTX3090 (168GB VRAM) to load a model like DeepSeek-V3.1-Terminus_IQ1_KT or so.  The model alone would occupy about 150GB.  Plus the buffers and KV cache for a small context of 32k ctx or so.  There will be no PCIe lanes left after that so I have to use another machine with a hybrid inference for a target model.  I am going to use Thunderbolt 4 for P2P networking.

Is there any info how to minimize the latency etc.?

Can anyone consult me regarding the RPC hooks I can use to implement that?