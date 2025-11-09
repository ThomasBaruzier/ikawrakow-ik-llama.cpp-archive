## 📌 [Issue #773](https://github.com/ikawrakow/ik_llama.cpp/issues/773) - Bug: CUDA error: an illegal memory access was encountered

| **Author** | `abxis` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-09-11 |
| **Updated** | 2025-09-13 |

---

## 📄 Description

### What happened?

使用4并发运行glm4.5air报错，截图如下：

<img width="3113" height="596" alt="Image" src="https://github.com/user-attachments/assets/2984e044-7a59-4e69-b8a6-450359818f31" />

运行命令如下：env CUDA_VISIBLE_DEVICES=2 numactl --interleave=all \
nohup ./build/bin/llama-server --parallel 20 -m /root/model/glm/GLM-4-5-Air-GGUF/UD-Q6_K_XL/GLM-4.5-Air-UD-Q6_K_XL-00001-of-00003.gguf  \
  -rtr -mla 1,3 -amb 512 -fmoe -fa -ngl 48  -t 60 -ub 4096 -b 4096  \
  -ot exps=CPU  \
 --alias GLM4.5-Air\
 --host 0.0.0.0 \
 --port 8081 \
 --numa numactl --cache-type-k q8_0 --cache-type-v q8_0  > output.log 2>&1 &

### Name and Version

ubuntu22.04,cuda 12.2

### What operating system are you seeing the problem on?

_No response_

### Relevant log output

```shell

```