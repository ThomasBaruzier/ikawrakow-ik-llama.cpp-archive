## 🗣️ [Discussion #594](https://github.com/ikawrakow/ik_llama.cpp/discussions/594) - Is AVX2 a hard requirement on x64?

| **Author** | `SmallAndSoft` |
| :--- | :--- |
| **State** | ✅ **Open** |
| **Created** | 2025-07-08 |
| **Updated** | 2025-09-26 |

---

## 📄 Description

I am getting compilation errors on the older CPU with just AVX even if I want to offload everything to CUDA GPU.

---

## 💬 Discussion

👤 **ikawrakow** commented on **2025-07-09** at **08:41:22**

Yes, `AVX2` or better is a hard requirement on `x86_64`. I think `llama.cpp` is a better option for older hardware.

> 👤 **SmallAndSoft** replied on **2025-07-09** at **08:45:07**
> 
> Thank you for reply. Yes, I just wanted to try your advanced quants on GPU. It is sad that AVX2 is required even if CPU will be doing next to nothing.

> 👤 **pandruszkow** replied on **2025-09-25** at **21:20:50**
> 
> I was just trying to get this to compile on Xeon E5-2600v2 (Ivy Bridge EP) before I realised the project does not support pre-AVX2 to begin with. I've tried to get the project going with some vibe coding help (this sort of code is beyond my level, I usually stick to Python, Java, etc.), but failed. Lack of pre-AVX2 support is sad, but totally understandable from an ongoing maintenance perspective, so I can respect that. At least the lack of support is communicated clearly.
> 
> @ikawrakow Sorry to dig this up, but I found out that a project like [SIMD Everywhere](https://github.com/simd-everywhere/simde) might be used to reimplement any intrinsics that are missing from the target CPU. It won't be full-speed, but at least it'll compile and run. Is there a possibility to integrate something like this into the project? This should (hopefully) add minimum maintenance overhead, and allow ancient CPUs to still make use of the ik_llama's advanced quants. I'd help out by submitting a pull request with an implementation, but this is beyond my skill set at this point in time.

> 👤 **magikRUKKOLA** replied on **2025-09-26** at **02:30:46**
> 
> @pandruszkow 
> 
> https://www.ebay.com/itm/134899171071
> ```
> Intel Xeon Platinum 8480 ES QYFS 1.9GHz 56 Cores 112T 350W LGA4677 CPU Processor
> $132
> ```
> 
> profit

> 👤 **ikawrakow** replied on **2025-09-26** at **06:31:02**
> 
> @pandruszkow
> 
> There simply aren't enough hours in the day to do everything users are asking for. AMX support, better NUMA support, ARM SVE, GPU tensor parallelism, add `ik_llama.cpp` specific quants to the Vulkan back-end,  support for multi-modality models, to just name a few.
> 
> Concerning the use of SIMD abstractions such as `SIMD Everywhere`: I looked a bit into that at the beginning of the project, but didn't think that they would be helpful. But I can revisit when time permits to see if this can be used to also (easily) target `AVX` with the `AVX2` implementation.