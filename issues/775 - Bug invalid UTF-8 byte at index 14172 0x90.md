## 📌 [Issue #775](https://github.com/ikawrakow/ik_llama.cpp/issues/775) - Bug: invalid UTF-8 byte at index 14172: 0x90

| **Author** | `abxis` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-09-11 |
| **Updated** | 2025-10-16 |
| **Labels** | `wontfix` |

---

## 📄 Description

### What happened?

<img width="3142" height="766" alt="Image" src="https://github.com/user-attachments/assets/33739e3b-027d-4b2c-bd18-ea5b3c202969" />
运行命令是：env CUDA_VISIBLE_DEVICES=1 numactl --interleave=all \
 ./build/bin/llama-server --parallel 20 -m /root/model/glm/GLM-4-5-Air-GGUF/UD-Q6_K_XL/GLM-4.5-Air-UD-Q6_K_XL-00001-of-00003.gguf  \
  -rtr -mla 1,3 -amb 512 -fmoe -fa -ngl 48  -t 60 -ub 4096 -b 4096  \
  -ot exps=CPU  \
 --alias GLM4.5-Air\
 --host 0.0.0.0 \
 --port 8081 \
 --numa numactl  -c 65536 --cache-type-k q8_0 --cache-type-v q8_0 

### Name and Version

ubuntu22.04,cuda12.2

### What operating system are you seeing the problem on?

_No response_

### Relevant log output

```shell

```

---

## 💬 Conversation

👤 **abxis** commented on **2025-09-12** at **03:38:34**

<img width="3116" height="290" alt="Image" src="https://github.com/user-attachments/assets/54aee26a-6555-485d-8c68-1763ca1f7ad9" /> ,开了4并发，（两个编程问题，两个写10000字小说）

---

👤 **firecoperana** commented on **2025-09-13** at **12:34:59**

It seems to happen when you are doing context shift and it truncates past token incorrectly. To avoid this, just increase your context size. When you use `--parallel 20` the effect size for each request is only 65535/20, so please reduce parallel size or increase context size based on your need

---

👤 **ikawrakow** commented on **2025-09-23** at **12:55:54**

I see the same issue also submitted in mainline.