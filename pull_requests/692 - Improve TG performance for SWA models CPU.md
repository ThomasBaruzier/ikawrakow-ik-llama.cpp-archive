## 🔀 [Pull Request #692](https://github.com/ikawrakow/ik_llama.cpp/pull/692) - Improve TG performance for SWA models (CPU)

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/cpu_swa_v0` |
| **Target Branch** | `main` |
| **Created** | 2025-08-15 |
| **Updated** | 2025-08-15 |
| **Merged** | 2025-08-15 |

---

## 📄 Description

I was playing with the newly released 270M parameter Gemma3 models, and wasn't particularly impressed with the TG performance of `ik_llama.cpp` when running CPU-only (or put differently, I was hoping to get better CPU performance).

The Gemma3 models use SWA, and this is currently totally ignored in `ik_llama.cpp`. So, I decided to add a quick hack that reduces the cost of the self-attention calculation. CPU-only for now, flash attention required.
 
Tested on a Ryzen-7950X CPU with Gemma3-270M-it quantized with Q8_0, and the MXFP4 GPT-OSS-20B model. In both cases `Q8_0` is used for KV cache.

### Gemma3-270M-it, Q8_0

<img width="792" height="612" alt="g3_swa" src="https://github.com/user-attachments/assets/98bb4458-fa3d-4ef6-98ef-fcfd0ccd91ad" />

### GPT-OSS-20B, MXFP4

<img width="792" height="612" alt="gpt_oss_swa" src="https://github.com/user-attachments/assets/1fc9af72-5e47-4212-8f71-7a404dac225d" />

We see a factor of ~2X improvement at 32k tokens for Gemma3-270M, and ~10% improvement for GPT-OSS-20B at 16k tokens. In Gemma3 5 out of 6 layers are SWA with a window size of 512, while in GPT-OSS every second layer uses SWA with a window size of 128. Although the number of "ON" tokens is approximately equal, the Gemma3 situation is better because one can use 16 threads, each processing 32 K and V rows. The FA implementation here uses a minimum chunk size of 32, so for GPT-OSS-20B only 4 threads can meaningfully do computation in layers using SWA, so the performance gain is less.  

As the SWA attention mask is fixed for the entire compute graph, a better implementation would compute it once per graph evaluation instead of repeating the same calculation in each SWA layer. I may add a separate op for that in a follow up PR.