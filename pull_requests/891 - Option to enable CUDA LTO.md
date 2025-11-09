## 🔀 [Pull Request #891](https://github.com/ikawrakow/ik_llama.cpp/pull/891) - Option to enable CUDA LTO

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | ✅ **Open** |
| **Source Branch** | `ik/cuda_lto` |
| **Target Branch** | `main` |
| **Created** | 2025-11-03 |
| **Updated** | 2025-11-05 |

---

## 📄 Description

@hksdpc255 raised the question if CUDA link-time optimization (LTO) would improve performance (see [#890](https://github.com/ikawrakow/ik_llama.cpp/issues/890)).

This PR adds the ability to enable CUDA LTO via
```
cmake -DGGML_CUDA=ON -DGGML_CUDA_LTO=ON $other_cmake_args
```

In my case (single RTX-4080 GPU), run time performance actually dropped by a few percent. Still, curious to see if it does something for others.

**Be warned**: If you enable CUDA LTO, better go for a **very extended** coffee break. Otherwise, when you trigger the build, after all `*.cu` files have been compiled, you will see
```
Linking CUDA device code CMakeFiles/ggml.dir/cmake_device_link.o
```
and you will get bored because it will stay there forever (30+ minutes !!! on my reasonably fast Ryzen-7950X CPU). During that time, `nvlink` will be consuming 100% of one CPU core and 20+ GB of RAM (i.e., you having many cores is irrelevant, single-threaded performance is the only thing that counts).

---

## 💬 Conversation

👤 **hksdpc255** commented on **2025-11-03** at **09:59:05**

Wow, 30+ minutes of link time for just a few percent slower runtime — that’s quite the trade-off

Looks like CUDA LTO is one of those things that sounds great in theory but doesn’t really pay off in practice (at least for now). Thanks for sharing the results!

BTW. I will test this on my computer and share the result.

---

👤 **Downtown-Case** commented on **2025-11-03** at **23:42:09**

No Cuda LTO:

```
~/AI/ik_llama.cpp main* 17s
❯ ./build/bin/llama-bench --model /home/alpha/Models/GGUF/gemma-3-27b-it-qat-iq4_ks.gguf  -ngl 999
ggml_cuda_init: GGML_CUDA_FORCE_MMQ:    no
ggml_cuda_init: GGML_CUDA_FORCE_CUBLAS: no
ggml_cuda_init: found 1 CUDA devices:
  Device 0: NVIDIA GeForce RTX 3090, compute capability 8.6, VMM: yes
| model                          |       size |     params | backend    | ngl |          test |              t/s |
| ------------------------------ | ---------: | ---------: | ---------- | --: | ------------: | ---------------: |
| gemma3 27B IQ4_KS - 4.25 bpw   |  15.49 GiB |    28.42 B | CUDA       | 999 |         pp512 |   1696.34 ± 4.09 |
| gemma3 27B IQ4_KS - 4.25 bpw   |  15.49 GiB |    28.42 B | CUDA       | 999 |         tg128 |     45.05 ± 0.02 |

build: 1cfd1986 (3946)

~/AI/ik_llama.cpp main* 17s
❯ ./build/bin/llama-bench --model /home/alpha/Models/GGUF/Qwen3-30B-A3B-mix-IQ4_K.gguf -ngl 999
ggml_cuda_init: GGML_CUDA_FORCE_MMQ:    no
ggml_cuda_init: GGML_CUDA_FORCE_CUBLAS: no
ggml_cuda_init: found 1 CUDA devices:
  Device 0: NVIDIA GeForce RTX 3090, compute capability 8.6, VMM: yes
| model                          |       size |     params | backend    | ngl |          test |              t/s |
| ------------------------------ | ---------: | ---------: | ---------- | --: | ------------: | ---------------: |
| qwen3moe ?B IQ4_K - 4.5 bpw    |  17.68 GiB |    30.53 B | CUDA       | 999 |         pp512 |  2846.63 ± 36.10 |
| qwen3moe ?B IQ4_K - 4.5 bpw    |  17.68 GiB |    30.53 B | CUDA       | 999 |         tg128 |    114.85 ± 0.01 |

build: 1cfd1986 (3946)
```


With Cuda LTO:
```
~/AI/ik_llama.cpp main* 18s
❯ ./build/bin/llama-bench --model /home/alpha/Models/GGUF/gemma-3-27b-it-qat-iq4_ks.gguf  -ngl 999
ggml_cuda_init: GGML_CUDA_FORCE_MMQ:    no
ggml_cuda_init: GGML_CUDA_FORCE_CUBLAS: no
ggml_cuda_init: found 1 CUDA devices:
  Device 0: NVIDIA GeForce RTX 3090, compute capability 8.6, VMM: yes
| model                          |       size |     params | backend    | ngl |          test |              t/s |
| ------------------------------ | ---------: | ---------: | ---------- | --: | ------------: | ---------------: |
| gemma3 27B IQ4_KS - 4.25 bpw   |  15.49 GiB |    28.42 B | CUDA       | 999 |         pp512 |   1692.48 ± 1.13 |
| gemma3 27B IQ4_KS - 4.25 bpw   |  15.49 GiB |    28.42 B | CUDA       | 999 |         tg128 |     44.19 ± 0.00 |

build: 1cfd1986 (3946)

~/AI/ik_llama.cpp main* 18s
❯ ./build/bin/llama-bench --model /home/alpha/Models/GGUF/Qwen3-30B-A3B-mix-IQ4_K.gguf -ngl 999
ggml_cuda_init: GGML_CUDA_FORCE_MMQ:    no
ggml_cuda_init: GGML_CUDA_FORCE_CUBLAS: no
ggml_cuda_init: found 1 CUDA devices:
  Device 0: NVIDIA GeForce RTX 3090, compute capability 8.6, VMM: yes
| model                          |       size |     params | backend    | ngl |          test |              t/s |
| ------------------------------ | ---------: | ---------: | ---------- | --: | ------------: | ---------------: |
| qwen3moe ?B IQ4_K - 4.5 bpw    |  17.68 GiB |    30.53 B | CUDA       | 999 |         pp512 |  2861.62 ± 18.14 |
| qwen3moe ?B IQ4_K - 4.5 bpw    |  17.68 GiB |    30.53 B | CUDA       | 999 |         tg128 |    111.20 ± 0.02 |

build: 1cfd1986 (3946)
```

On a 3090/7800/Linux/CUDA 13. 3090 clocks locked to 1590Mhz for consistency.

*Shrug*.

Builds look the same because I just edited the changes into current master, and manually changed the nvcc architecture/code flags to `lto_86`. It did, indeed, take forever to build.

---

👤 **ikawrakow** commented on **2025-11-04** at **05:59:29**

At some level LTO not providing performance benefit for the CUDA code is understandable. Most of the time is spent in matrix multiplications and flash attention, where templates and inlining is already heavily used, so there is not much LTO can do on top of that.

I'll leave the PR open so it is visible that this has been tried, but yes, *shrug*.

---

👤 **Downtown-Case** commented on **2025-11-04** at **22:07:35**

One thing I forgot to look at is the artifact file size and inference VRAM usage.

...Would *slightly* smaller CUDA binaries translate to a little less VRAM usage? Or is that basically a rounding error?

---

👤 **Ph0rk0z** commented on **2025-11-05** at **12:05:02**

I turn on regular LTO, could it also have negative performance?