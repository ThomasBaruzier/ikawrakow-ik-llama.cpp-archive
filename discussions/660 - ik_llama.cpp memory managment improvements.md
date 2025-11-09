## 🗣️ [Discussion #660](https://github.com/ikawrakow/ik_llama.cpp/discussions/660) - ik_llama.cpp memory managment improvements

| **Author** | `xkmire` |
| :--- | :--- |
| **State** | ✅ **Open** |
| **Created** | 2025-07-28 |
| **Updated** | 2025-07-28 |

---

## 📄 Description

Hi!
I did a patch to ik_llama.cpp, to optimize memory behaivor due to issues I had running the very large DeepSeek with 670B parameters, even if it is heavily quantized (my system only have 128GByte of RAM and an RTX5090 with 32 GByte of VRAM).

Issue:
It took too long time to load the model and I also experienced swapping due to that the gguf files was sligtly larger than my entire system RAM, so everything on my system went very slow including token generation.

Solution:
I skipped prefetching the entire model into RAM and let the memory manager load the memory mapped pages on demand. This is very benefitial in a model like deepseek, where only a subset of the model is executed for each query. 
I offload part of the model to GPU, and I hint the linux memory manager that these parts are no longer needed system RAM, so feel free to drop these instead of swapping other stuff to file system.

Result:
When starting llama-server with this huge model, I can start getting the first tokens out after approx 15 seconds vs. several times longer before. Token generation is quicker due to less pressure on memory system overall. Also my system remains possible to use in parallel for other applications without swapping.
Command I use is 
llama-server  --model DeepSeek-R1-0528-IQ1_S_R4-00001-of-00003.gguf --alias ubergarm/DeepSeek-R1-0528-IQ1_S_R4  -fa  -fmoe  -mla 3  -amb 512  -ctk q8_0  -c 16384  -b 4096  -ub 4096  -ngl 99 -ot "blk\.(0|1|2|3|4|5|6|7|8)\.ffn_.*=CUDA0"  -ot exps=CPU --parallel 1  --threads 16  --no-warmup --host 192.168.0.40  --port 8080

Limitations:
I run ubuntu, so patch is only for ubuntu memory manager. Not sure how windows works.
Also, this is a corner case, which only helps to run models that are slightly larger the optimal. If you have enough RAM/VRAM to start with, this wont improve anything.

I attach the patch, if anyone else is interested, I can do a pull request ?
Obviously, I also asked my now running Deepseek AI to review the patch. Also attached :-)
[diff.txt](https://github.com/user-attachments/files/21476417/diff.txt)
[review.txt](https://github.com/user-attachments/files/21476420/review.txt)

---

## 💬 Discussion

👤 **saood06** commented on **2025-07-28** at **20:16:59**

I'm curious how much of this can be achieved by just using the `--no-warmup` flag on launch.

> 👤 **xkmire** replied on **2025-07-28** at **20:50:36**
> 
> I did try that, but that flag only skipped running a test run later on when everything was loaded. 
> The code still did this:
>  // advise the kernel to preload the mapped memory
>   if (posix_madvise(addr, std::min(file->size, prefetch), POSIX_MADV_WILLNEED)) 
>   as well as as put a flag causing long load times: POSIX_FADV_SEQUENTIAL.