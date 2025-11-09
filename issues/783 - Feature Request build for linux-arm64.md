## 📌 [Issue #783](https://github.com/ikawrakow/ik_llama.cpp/issues/783) - Feature Request: build for linux-arm64

| **Author** | `GrittyChen` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-09-17 |
| **Updated** | 2025-09-17 |
| **Labels** | `enhancement` |

---

## 📄 Description

### Prerequisites

- [x] I am running the latest code. Mention the version if possible as well.
- [x] I carefully followed the [README.md](https://github.com/ggerganov/llama.cpp/blob/master/README.md).
- [x] I searched using keywords relevant to my issue to make sure that I am creating a new issue that is not already open (or closed).
- [x] I reviewed the [Discussions](https://github.com/ggerganov/llama.cpp/discussions), and have a new and useful enhancement to share.

### Feature Description

It seems that the Makefile currently does not support compilation on the linux-arm64 platform. Could you please improve the Makefile so that it can be successfully compiled and run on the linux-arm64 platform? Thanks!

### Motivation

The linux-arm64 platform is commonly used. The enhancement will improve the scope of application of the project.

### Possible Implementation

_No response_

---

## 💬 Conversation

👤 **ikawrakow** commented on **2025-09-17** at **13:02:35**

The Makefile is not maintained. Use cmake instead

---

👤 **GrittyChen** commented on **2025-09-17** at **13:10:00**

> The Makefile is not maintained. Use cmake instead

@ikawrakow Thanks for your reply. In practice, I used CMake to build, but got the following error:
/usr/bin/ld: ../../ggml/src/libggml.so: undefined reference to `iqk_flash_attn_noalibi'
collect2: error: ld returned 1 exit status

I also used the same commands to compile on the amd64 platform and built successfully.

The CMake commands are as follows:
cmake -B build -DGGML_CUDA=OFF -DGGML_BLAS=ON -DGGML_BLAS_VENDOR=OpenBLAS -DLLAMA_CURL=OFF
cmake --build build --target llama-server -j 8

---

👤 **ikawrakow** commented on **2025-09-17** at **13:27:20**

It seems your compiler does not automatically enable the necessary flags. In that case you need to manually specify via -DGGML_ARCH_FLAGS=... Arm8.2 is the minimum supported version. You most likely want to build without BLAS as this tends to give better prefill performance. 

What is the CPU?

---

👤 **GrittyChen** commented on **2025-09-17** at **15:36:46**

> It seems your compiler does not automatically enable the necessary flags. In that case you need to manually specify via -DGGML_ARCH_FLAGS=... Arm8.2 is the minimum supported version. You most likely want to build without BLAS as this tends to give better prefill performance.
> 
> What is the CPU?

@ikawrakow Thanks for your suggestion! I built successfully with the following commands:
cmake -B build -DGGML_CUDA=OFF -DGGML_BLAS=OFF -DLLAMA_CURL=OFF -DGGML_ARCH_FLAGS="-march=native"
cmake --build build --target llama-server -j 8

---

👤 **GrittyChen** commented on **2025-09-17** at **15:38:13**

The issue has been solved.