## 📌 [Issue #721](https://github.com/ikawrakow/ik_llama.cpp/issues/721) - Bug: libcuda.so.575.64.05 segfault, iqk_mul_mat.cpp:551

| **Author** | `magikRUKKOLA` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-08-23 |
| **Updated** | 2025-08-23 |

---

## 📄 Description

### What happened?

Not sure if its AVX2 related or its the driver's issue.

```
INFO [                    main] system info | tid="140211887566848" timestamp=1755947275 n_threads=64 n_threads_batch=-1 total_threads=128 system_info="AVX = 1 | AVX_VNNI = 0 | AVX2 = 1 | AVX512 = 0 | AVX512_VBMI = 0 | AVX512_VNNI = 0 | AVX512_BF16 = 0 | FMA = 1 | NEON = 0 | SVE = 0 | ARM_FMA = 0 | F16C = 1 | FP16_VA = 0 | WASM_SIMD = 0 | BLAS = 1 | SSE3 = 1 | SSSE3 = 1 | VSX = 0 | MATMUL_INT8 = 0 | LLAMAFILE = 1 | "
```

```
/opt/GGUF-Tool-Suite/GGUF-Tool-Suite/DeepSeek-R1-0528.ROOT-6.2478bpw# nvidia-smi
Sat Aug 23 11:08:34 2025
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 575.64.05              Driver Version: 575.64.05      CUDA Version: 12.9     |
|-----------------------------------------+------------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
|                                         |                        |               MIG M. |
|=========================================+========================+======================|
|   0  NVIDIA GeForce RTX 3090        Off |   00000000:41:00.0 Off |                  N/A |
| 30%   51C    P8             20W /  350W |       3MiB /  24576MiB |      0%      Default |
|                                         |                        |                  N/A |
+-----------------------------------------+------------------------+----------------------+
|   1  NVIDIA GeForce RTX 3090        Off |   00000000:42:00.0 Off |                  N/A |
| 30%   38C    P8             16W /  350W |       3MiB /  24576MiB |      0%      Default |
|                                         |                        |                  N/A |
+-----------------------------------------+------------------------+----------------------+
|   2  NVIDIA GeForce RTX 3090        Off |   00000000:61:00.0 Off |                  N/A |
| 30%   49C    P8             20W /  350W |       3MiB /  24576MiB |      0%      Default |
|                                         |                        |                  N/A |
+-----------------------------------------+------------------------+----------------------+

+-----------------------------------------------------------------------------------------+
| Processes:                                                                              |
|  GPU   GI   CI              PID   Type   Process name                        GPU Memory |
|        ID   ID                                                               Usage      |
|=========================================================================================|
|  No running processes found                                                             |
+-----------------------------------------------------------------------------------------+
```

### Name and Version

/opt/ik_llama.cpp/ik_llama.cpp/build/bin/llama-server --version
version: 3856 (e919e89d)
built with cc (Debian 14.2.0-19) 14.2.0 for x86_64-linux-gnu

### What operating system are you seeing the problem on?

_No response_

### Relevant log output

```shell
[Sat Aug 23 11:05:44 2025] llama-server[383019]: segfault at 20e403fd8 ip 00007fe8ef318637 sp 00007fe84ee2ff80 error 4 in libcuda.so.575.64.05[518637,7fe8eefc5000+e97000] likely on CPU 114 (core 50, socket 0)
[Sat Aug 23 11:05:44 2025] Code: ef e8 3d cd ca ff 83 3d 1e 5b 2f 05 01 49 8b 1c 24 76 0a 8b 05 26 5b 2f 05 85 c0 74 56 49 8b 44 24 10 41 8b 4c 24 24 48 8b 13 <8b> 00 41 39 c6 74 52 8b b3 40 40 00 00 48 89 f0 89 8c b3 44 40 00
[Sat Aug 23 11:05:44 2025] llama-server[383020]: segfault at 20e403fd8 ip 00007fe8ef318637 sp 00007fe8558d2f80 error 4 in libcuda.so.575.64.05[518637,7fe8eefc5000+e97000] likely on CPU 29 (core 29, socket 0)
[Sat Aug 23 11:05:44 2025] Code: ef e8 3d cd ca ff 83 3d 1e 5b 2f 05 01 49 8b 1c 24 76 0a 8b 05 26 5b 2f 05 85 c0 74 56 49 8b 44 24 10 41 8b 4c 24 24 48 8b 13 <8b> 00 41 39 c6 74 52 8b b3 40 40 00 00 48 89 f0 89 8c b3 44 40 00
[Sat Aug 23 11:05:44 2025] llama-server[383021]: segfault at 20e403fd8 ip 00007fe8ef318637 sp 00007fe841276f80 error 4 in libcuda.so.575.64.05[518637,7fe8eefc5000+e97000] likely on CPU 117 (core 53, socket 0)
[Sat Aug 23 11:05:44 2025] Code: ef e8 3d cd ca ff 83 3d 1e 5b 2f 05 01 49 8b 1c 24 76 0a 8b 05 26 5b 2f 05 85 c0 74 56 49 8b 44 24 10 41 8b 4c 24 24 48 8b 13 <8b> 00 41 39 c6 74 52 8b b3 40 40 00 00 48 89 f0 89 8c b3 44 40 00
[Sat Aug 23 11:05:44 2025] llama-server[383022]: segfault at 20e403fd8 ip 00007fe8ef318637 sp 00007fe84c62af80 error 4 in libcuda.so.575.64.05[518637,7fe8eefc5000+e97000] likely on CPU 72 (core 8, socket 0)
[Sat Aug 23 11:05:44 2025] Code: ef e8 3d cd ca ff 83 3d 1e 5b 2f 05 01 49 8b 1c 24 76 0a 8b 05 26 5b 2f 05 85 c0 74 56 49 8b 44 24 10 41 8b 4c 24 24 48 8b 13 <8b> 00 41 39 c6 74 52 8b b3 40 40 00 00 48 89 f0 89 8c b3 44 40 00
[Sat Aug 23 11:05:44 2025] llama-server[383023]: segfault at 20e403fd8 ip 00007fe8ef318637 sp 00007fe84742df80 error 4 in libcuda.so.575.64.05[518637,7fe8eefc5000+e97000] likely on CPU 68 (core 4, socket 0)
[Sat Aug 23 11:05:44 2025] Code: ef e8 3d cd ca ff 83 3d 1e 5b 2f 05 01 49 8b 1c 24 76 0a 8b 05 26 5b 2f 05 85 c0 74 56 49 8b 44 24 10 41 8b 4c 24 24 48 8b 13 <8b> 00 41 39 c6 74 52 8b b3 40 40 00 00 48 89 f0 89 8c b3 44 40 00


Aug 23 11:05:49 xxx run-ik_llama.cpp.sh[367724]: VERB [            update_slots] prompt done | tid="140639186616320" timestamp=1755947149 id_slot=0 n_past=147 n_ctx=163840 n_tokens=144
Aug 23 11:05:49 xxx run-ik_llama.cpp.sh[367724]: VERB [            update_slots] decoding batch | tid="140639186616320" timestamp=1755947149 n_tokens=144
Aug 23 11:05:51 xxx run-ik_llama.cpp.sh[367724]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:551: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:551: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:551: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:551: GGML_ASSERT(Nx%num_rows == 0) failed
Aug 23 11:05:51 xxx run-ik_llama.cpp.sh[367724]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:551: GGML_ASSERT(Nx%num_rows == 0) failed
Aug 23 11:05:51 xxx run-ik_llama.cpp.sh[367724]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:551: GGML_ASSERT(Nx%num_rows == 0) failed
Aug 23 11:05:51 xxx run-ik_llama.cpp.sh[367724]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:551: GGML_ASSERT(Nx%num_rows == 0) failed
Aug 23 11:05:51 xxx run-ik_llama.cpp.sh[367724]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:551: GGML_ASSERT(Nx%num_rows == 0) failed
Aug 23 11:05:51 xxx run-ik_llama.cpp.sh[367724]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:551: GGML_ASSERT(Nx%num_rows == 0) failed
Aug 23 11:05:51 xxx run-ik_llama.cpp.sh[367724]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:551: GGML_ASSERT(Nx%num_rows == 0) failed
Aug 23 11:05:51 xxx run-ik_llama.cpp.sh[367724]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:551: GGML_ASSERT(Nx%num_rows == 0) failed
Aug 23 11:05:51 xxx run-ik_llama.cpp.sh[367724]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:551: GGML_ASSERT(Nx%num_rows == 0) failed
Aug 23 11:05:51 xxx run-ik_llama.cpp.sh[367724]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:551: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:551: GGML_ASSERT(Nx%num_rows == 0) failed
Aug 23 11:05:51 xxx run-ik_llama.cpp.sh[367724]: GGML_ASSERT(Nx%num_rows == 0) failed
Aug 23 11:05:51 xxx run-ik_llama.cpp.sh[367724]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:551: GGML_ASSERT(Nx%num_rows == 0) failed
Aug 23 11:05:51 xxx run-ik_llama.cpp.sh[367724]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:551: GGML_ASSERT(Nx%num_rows == 0) failed
Aug 23 11:05:51 xxx run-ik_llama.cpp.sh[367724]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:551: GGML_ASSERT(Nx%num_rows == 0) failed
Aug 23 11:05:51 xxx run-ik_llama.cpp.sh[367724]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:551: GGML_ASSERT(Nx%num_rows == 0) failed
Aug 23 11:05:51 xxx run-ik_llama.cpp.sh[367724]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:551: GGML_ASSERT(Nx%num_rows == 0) failed
Aug 23 11:05:51 xxx run-ik_llama.cpp.sh[367724]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:551: GGML_ASSERT(Nx%num_rows == 0) failed
Aug 23 11:05:51 xxx run-ik_llama.cpp.sh[367724]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:551: GGML_ASSERT(Nx%num_rows == 0) failed
Aug 23 11:05:51 xxx run-ik_llama.cpp.sh[367724]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:551: GGML_ASSERT(Nx%num_rows == 0) failed
Aug 23 11:05:51 xxx run-ik_llama.cpp.sh[367724]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:551: GGML_ASSERT(Nx%num_rows == 0) failed
Aug 23 11:05:51 xxx run-ik_llama.cpp.sh[367724]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:551: GGML_ASSERT(Nx%num_rows == 0) failed
Aug 23 11:05:51 xxx run-ik_llama.cpp.sh[367724]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:551: GGML_ASSERT(Nx%num_rows == 0) failed
Aug 23 11:05:51 xxx run-ik_llama.cpp.sh[367724]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:551: GGML_ASSERT(Nx%num_rows == 0) failed
Aug 23 11:05:51 xxx run-ik_llama.cpp.sh[367724]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:551: GGML_ASSERT(Nx%num_rows == 0) failed
Aug 23 11:05:51 xxx run-ik_llama.cpp.sh[367724]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:551: GGML_ASSERT(Nx%num_rows == 0) failed
Aug 23 11:05:51 xxx run-ik_llama.cpp.sh[367724]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:551: GGML_ASSERT(Nx%num_rows == 0) failed
Aug 23 11:05:51 xxx run-ik_llama.cpp.sh[367724]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:551: GGML_ASSERT(Nx%num_rows == 0) failed
Aug 23 11:05:51 xxx run-ik_llama.cpp.sh[367724]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:551: GGML_ASSERT(Nx%num_rows == 0) failed
Aug 23 11:05:51 xxx run-ik_llama.cpp.sh[367724]: GGML_ASSERT(Nx%num_rows == 0) failed
Aug 23 11:05:51 xxx run-ik_llama.cpp.sh[367724]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:551: GGML_ASSERT(Nx%num_rows == 0) failed
Aug 23 11:05:51 xxx run-ik_llama.cpp.sh[367724]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:551: GGML_ASSERT(Nx%num_rows == 0) failed
Aug 23 11:05:51 xxx run-ik_llama.cpp.sh[367724]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:551: GGML_ASSERT(Nx%num_rows == 0) failed
Aug 23 11:05:51 xxx run-ik_llama.cpp.sh[367724]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:551: GGML_ASSERT(Nx%num_rows == 0) failed
Aug 23 11:05:51 xxx run-ik_llama.cpp.sh[367724]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:551: GGML_ASSERT(Nx%num_rows == 0) failed
Aug 23 11:05:51 xxx run-ik_llama.cpp.sh[367724]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:551: GGML_ASSERT(Nx%num_rows == 0) failed
Aug 23 11:05:51 xxx run-ik_llama.cpp.sh[367724]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:551: GGML_ASSERT(Nx%num_rows == 0) failed
Aug 23 11:05:51 xxx run-ik_llama.cpp.sh[367724]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:551: GGML_ASSERT(Nx%num_rows == 0) failed
Aug 23 11:05:51 xxx run-ik_llama.cpp.sh[367724]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:551: GGML_ASSERT(Nx%num_rows == 0) failed
Aug 23 11:05:51 xxx run-ik_llama.cpp.sh[367724]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:551: GGML_ASSERT(Nx%num_rows == 0) failed
Aug 23 11:05:51 xxx run-ik_llama.cpp.sh[367724]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:551: GGML_ASSERT(Nx%num_rows == 0) failed
Aug 23 11:05:51 xxx run-ik_llama.cpp.sh[367724]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:551: GGML_ASSERT(Nx%num_rows == 0) failed
Aug 23 11:05:51 xxx run-ik_llama.cpp.sh[367724]: GGML_ASSERT(Nx%num_rows == 0) failed
Aug 23 11:05:51 xxx run-ik_llama.cpp.sh[367724]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:551: GGML_ASSERT(Nx%num_rows == 0) failed
Aug 23 11:05:51 xxx run-ik_llama.cpp.sh[367724]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:551: GGML_ASSERT(Nx%num_rows == 0) failed
Aug 23 11:05:51 xxx run-ik_llama.cpp.sh[367724]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:551: GGML_ASSERT(Nx%num_rows == 0) failed
Aug 23 11:05:51 xxx run-ik_llama.cpp.sh[367724]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:551: GGML_ASSERT(Nx%num_rows == 0) failed
Aug 23 11:05:51 xxx run-ik_llama.cpp.sh[367724]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:551: GGML_ASSERT(Nx%num_rows == 0) failed
Aug 23 11:05:51 xxx run-ik_llama.cpp.sh[367724]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:551: GGML_ASSERT(Nx%num_rows == 0) failed
Aug 23 11:05:51 xxx run-ik_llama.cpp.sh[367724]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:551: GGML_ASSERT(Nx%num_rows == 0) failed
Aug 23 11:05:51 xxx run-ik_llama.cpp.sh[367724]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:551: GGML_ASSERT(Nx%num_rows == 0) failed
Aug 23 11:05:51 xxx run-ik_llama.cpp.sh[367724]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:551: GGML_ASSERT(Nx%num_rows == 0) failed
Aug 23 11:05:51 xxx run-ik_llama.cpp.sh[367724]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:551: GGML_ASSERT(Nx%num_rows == 0) failed
Aug 23 11:05:51 xxx run-ik_llama.cpp.sh[367724]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:551: GGML_ASSERT(Nx%num_rows == 0) failed
Aug 23 11:05:51 xxx run-ik_llama.cpp.sh[367724]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:551: GGML_ASSERT(Nx%num_rows == 0) failed
Aug 23 11:05:51 xxx run-ik_llama.cpp.sh[367724]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:551: GGML_ASSERT(Nx%num_rows == 0) failed
Aug 23 11:05:51 xxx run-ik_llama.cpp.sh[367724]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:551: GGML_ASSERT(Nx%num_rows == 0) failed
Aug 23 11:05:51 xxx run-ik_llama.cpp.sh[367724]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:551: GGML_ASSERT(Nx%num_rows == 0) failed
Aug 23 11:05:51 xxx run-ik_llama.cpp.sh[367724]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:551: GGML_ASSERT(Nx%num_rows == 0) failed
Aug 23 11:05:51 xxx run-ik_llama.cpp.sh[367724]: GGML_ASSERT(Nx%num_rows == 0) failed
Aug 23 11:05:51 xxx run-ik_llama.cpp.sh[367724]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:551: GGML_ASSERT(Nx%num_rows == 0) failed
Aug 23 11:05:51 xxx run-ik_llama.cpp.sh[367724]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:551: GGML_ASSERT(Nx%num_rows == 0) failed
Aug 23 11:05:51 xxx run-ik_llama.cpp.sh[367724]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:551: GGML_ASSERT(Nx%num_rows == 0) failed
Aug 23 11:05:51 xxx run-ik_llama.cpp.sh[367724]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:551: GGML_ASSERT(Nx%num_rows == 0) failed
Aug 23 11:05:51 xxx run-ik_llama.cpp.sh[367724]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:551: GGML_ASSERT(Nx%num_rows == 0) failed
```

---

## 💬 Conversation

👤 **ikawrakow** commented on **2025-08-23** at **11:25:27**

Please run with [#722](https://github.com/ikawrakow/ik_llama.cpp/issues/722) and post what it says before the assert.

---

👤 **magikRUKKOLA** commented on **2025-08-23** at **11:31:02**

@ikawrakow 
> Please run with [[#722](https://github.com/ikawrakow/ik_llama.cpp/issues/722)](https://github.com/ikawrakow/ik_llama.cpp/pull/722) and post what it says before the assert.

okay cool.  let me do that.

but apparently its definitely related to the recent changes.  I rolled back to:

```bash
git revert --no-commit f6f56db00d7d20c98266664fe793516ad167f87c..HEAD
```

and there is no crash.

---

👤 **ikawrakow** commented on **2025-08-23** at **11:39:03**

Well, the recent changes fix an issue. I'm not finding f6f56db00d7d20c98266664fe793516ad167f87c. What does `llama-server --version` say with the reverted commits?

---

👤 **magikRUKKOLA** commented on **2025-08-23** at **11:55:35**

@ikawrakow 
> Please run with [[#722](https://github.com/ikawrakow/ik_llama.cpp/issues/722)](https://github.com/ikawrakow/ik_llama.cpp/pull/722) and post what it says before the assert.

```

Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: iqk_mul_mat: Nx = 1, Ny = 1, ne00 = 2048, num_rows = 16, types = q8_0, q8_0_x4
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:554: iqk_mul_mat: Nx = 1, Ny = 1, ne00 = 2048, num_rows = 16, types = q8_0, q8_0_x4
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: iqk_mul_mat: Nx = 1, Ny = 1, ne00 = 2048, num_rows = 16, types = q8_0, q8_0_x4
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: iqk_mul_mat: Nx = 1, Ny = 1, ne00 = 2048, num_rows = 16, types = q8_0, q8_0_x4
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:554: iqk_mul_mat: Nx = 1, Ny = 1, ne00 = 2048, num_rows = 16, types = q8_0, q8_0_x4
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:554: iqk_mul_mat: Nx = 1, Ny = 1, ne00 = 2048, num_rows = 16, types = q8_0, q8_0_x4
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: iqk_mul_mat: Nx = 1, Ny = 1, ne00 = 2048, num_rows = 16, types = q8_0, q8_0_x4
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: iqk_mul_mat: Nx = 1, Ny = 1, ne00 = 2048, num_rows = 16, types = q8_0, q8_0_x4
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:554: GGML_ASSERT(false) failediqk_mul_mat: Nx = 1, Ny = 1, ne00 = 2048, num_rows = 16, types = q8_0, q8_0_x4
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:554: GGML_ASSERT(false) failed
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: GGML_ASSERT(false) failediqk_mul_mat: Nx = 1, Ny = 1, ne00 = 2048, num_rows = 16, types = q8_0, q8_0_x4
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:554: GGML_ASSERT(false) failed
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: iqk_mul_mat: Nx = 1, Ny = 1, ne00 = 2048, num_rows = 16, types = q8_0, q8_0_x4
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:554: GGML_ASSERT(false) failed
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: iqk_mul_mat: Nx = 1, Ny = 1, ne00 = 2048, num_rows = 16, types = q8_0, q8_0_x4
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:554: GGML_ASSERT(false) failediqk_mul_mat: Nx = 1, Ny = 1, ne00 = 2048, num_rows = 16, types = q8_0, q8_0_x4
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:554: GGML_ASSERT(false) failed
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: iqk_mul_mat: Nx = 1, Ny = 1, ne00 = 2048, num_rows = 16, types = q8_0, q8_0_x4
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:554: GGML_ASSERT(false) failediqk_mul_mat: Nx = 1, Ny = 1, ne00 = 2048, num_rows = 16, types = q8_0, q8_0_x4
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:554: GGML_ASSERT(false) failed
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:554: GGML_ASSERT(false) failed
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: GGML_ASSERT(false) failed
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: iqk_mul_mat: Nx = 1, Ny = 1, ne00 = 2048, num_rows = 16, types = q8_0, q8_0_x4
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:554: GGML_ASSERT(false) failed
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: iqk_mul_mat: Nx = 1, Ny = 1, ne00 = 2048, num_rows = 16, types = q8_0, q8_0_x4
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:554: GGML_ASSERT(false) failed
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: iqk_mul_mat: Nx = 1, Ny = 1, ne00 = 2048, num_rows = 16, types = q8_0, q8_0_x4
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:554: GGML_ASSERT(false) failed
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: iqk_mul_mat: Nx = 1, Ny = 1, ne00 = 2048, num_rows = 16, types = q8_0, q8_0_x4
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:554: GGML_ASSERT(false) failed
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: iqk_mul_mat: Nx = 1, Ny = 1, ne00 = 2048, num_rows = 16, types = q8_0, q8_0_x4
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:554: iqk_mul_mat: Nx = 1, Ny = 1, ne00 = 2048, num_rows = 16, types = q8_0, q8_0_x4
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:554: GGML_ASSERT(false) failed
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: iqk_mul_mat: Nx = 1, Ny = 1, ne00 = 2048, num_rows = 16, types = q8_0, q8_0_x4
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:554: GGML_ASSERT(false) failed
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: iqk_mul_mat: Nx = 1, Ny = 1, ne00 = 2048, num_rows = 16, types = q8_0, q8_0_x4
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:554: GGML_ASSERT(false) failed
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: iqk_mul_mat: Nx = 1, Ny = 1, ne00 = 2048, num_rows = 16, types = q8_0, q8_0_x4
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:554: GGML_ASSERT(false) failed
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: iqk_mul_mat: Nx = 1, Ny = 1, ne00 = 2048, num_rows = 16, types = q8_0, q8_0_x4
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:554: GGML_ASSERT(false) failed
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: iqk_mul_mat: Nx = 1, Ny = 1, ne00 = 2048, num_rows = 16, types = q8_0, q8_0_x4
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:554: GGML_ASSERT(false) failediqk_mul_mat: Nx = 1, Ny = 1, ne00 = 2048, num_rows = 16, types = q8_0, q8_0_x4
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:554: GGML_ASSERT(false) failed
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: iqk_mul_mat: Nx = 1, Ny = 1, ne00 = 2048, num_rows = 16, types = q8_0, q8_0_x4
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:554: GGML_ASSERT(false) failed
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: iqk_mul_mat: Nx = 1, Ny = 1, ne00 = 2048, num_rows = 16, types = q8_0, q8_0_x4
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:554: iqk_mul_mat: Nx = 1, Ny = 1, ne00 = 2048, num_rows = 16, types = q8_0, q8_0_x4
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:554: GGML_ASSERT(false) failed
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: iqk_mul_mat: Nx = 1, Ny = 1, ne00 = 2048, num_rows = 16, types = q8_0, q8_0_x4
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:554: GGML_ASSERT(false) failed
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: iqk_mul_mat: Nx = 1, Ny = 1, ne00 = 2048, num_rows = 16, types = q8_0, q8_0_x4
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:554: GGML_ASSERT(false) failed
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: iqk_mul_mat: Nx = 1, Ny = 1, ne00 = 2048, num_rows = 16, types = q8_0, q8_0_x4
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:554: GGML_ASSERT(false) failed
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: iqk_mul_mat: Nx = 1, Ny = 1, ne00 = 2048, num_rows = 16, types = q8_0, q8_0_x4
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:554: GGML_ASSERT(false) failed
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: GGML_ASSERT(false) failed
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: iqk_mul_mat: Nx = 1, Ny = 1, ne00 = 2048, num_rows = 16, types = q8_0, q8_0_x4
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:554: GGML_ASSERT(false) failed
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: iqk_mul_mat: Nx = 1, Ny = 1, ne00 = 2048, num_rows = 16, types = q8_0, q8_0_x4
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:554: GGML_ASSERT(false) failed
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: iqk_mul_mat: Nx = 1, Ny = 1, ne00 = 2048, num_rows = 16, types = q8_0, q8_0_x4
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:554: GGML_ASSERT(false) failed
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: iqk_mul_mat: Nx = 1, Ny = 1, ne00 = 2048, num_rows = 16, types = q8_0, q8_0_x4
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:554: GGML_ASSERT(false) failediqk_mul_mat: Nx = 1, Ny = 1, ne00 = 2048, num_rows = 16, types = q8_0, q8_0_x4
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:554: GGML_ASSERT(false) failediqk_mul_mat: Nx = 1, Ny = 1, ne00 = 2048, num_rows = 16, types = q8_0, q8_0_x4
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:554: GGML_ASSERT(false) failed
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: iqk_mul_mat: Nx = 1, Ny = 1, ne00 = 2048, num_rows = 16, types = q8_0, q8_0_x4
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:554: GGML_ASSERT(false) failed
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: iqk_mul_mat: Nx = 1, Ny = 1, ne00 = 2048, num_rows = 16, types = q8_0, q8_0_x4
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:554: GGML_ASSERT(false) failediqk_mul_mat: Nx = 1, Ny = 1, ne00 = 2048, num_rows = 16, types = q8_0, q8_0_x4
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:554: GGML_ASSERT(false) failed
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: iqk_mul_mat: Nx = 1, Ny = 1, ne00 = 2048, num_rows = 16, types = q8_0, q8_0_x4
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:554: GGML_ASSERT(false) failed
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: iqk_mul_mat: Nx = 1, Ny = 1, ne00 = 2048, num_rows = 16, types = q8_0, q8_0_x4
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:554: GGML_ASSERT(false) failed
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: iqk_mul_mat: Nx = 1, Ny = 1, ne00 = 2048, num_rows = 16, types = q8_0, q8_0_x4
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:554: GGML_ASSERT(false) failed
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: iqk_mul_mat: Nx = 1, Ny = 1, ne00 = 2048, num_rows = 16, types = q8_0, q8_0_x4
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:554: GGML_ASSERT(false) failed
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: iqk_mul_mat: Nx = 1, Ny = 1, ne00 = 2048, num_rows = 16, types = q8_0, q8_0_x4
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:554: GGML_ASSERT(false) failed
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: iqk_mul_mat: Nx = 1, Ny = 1, ne00 = 2048, num_rows = 16, types = q8_0, q8_0_x4
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:554: GGML_ASSERT(false) failed
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:554: GGML_ASSERT(false) failed
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: iqk_mul_mat: Nx = 1, Ny = 1, ne00 = 2048, num_rows = 16, types = q8_0, q8_0_x4
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:554: GGML_ASSERT(false) failed
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: iqk_mul_mat: Nx = 1, Ny = 1, ne00 = 2048, num_rows = 16, types = q8_0, q8_0_x4
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:554: GGML_ASSERT(false) failed
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: iqk_mul_mat: Nx = 1, Ny = 1, ne00 = 2048, num_rows = 16, types = q8_0, q8_0_x4
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:554: GGML_ASSERT(false) failed
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: iqk_mul_mat: Nx = 1, Ny = 1, ne00 = 2048, num_rows = 16, types = q8_0, q8_0_x4
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:554: GGML_ASSERT(false) failed
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: iqk_mul_mat: Nx = 1, Ny = 1, ne00 = 2048, num_rows = 16, types = q8_0, q8_0_x4
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:554: GGML_ASSERT(false) failed
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: iqk_mul_mat: Nx = 1, Ny = 1, ne00 = 2048, num_rows = 16, types = q8_0, q8_0_x4
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:554: GGML_ASSERT(false) failed
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: iqk_mul_mat: Nx = 1, Ny = 1, ne00 = 2048, num_rows = 16, types = q8_0, q8_0_x4
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:554: GGML_ASSERT(false) failed
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: iqk_mul_mat: Nx = 1, Ny = 1, ne00 = 2048, num_rows = 16, types = q8_0, q8_0_x4
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:554: GGML_ASSERT(false) failediqk_mul_mat: Nx = 1, Ny = 1, ne00 = 2048, num_rows = 16, types = q8_0, q8_0_x4
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:554: GGML_ASSERT(false) failediqk_mul_mat: Nx = 1, Ny = 1, ne00 = 2048, num_rows = 16, types = q8_0, q8_0_x4
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:554: GGML_ASSERT(false) failed
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: GGML_ASSERT(false) failed
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: iqk_mul_mat: Nx = 1, Ny = 1, ne00 = 2048, num_rows = 16, types = q8_0, q8_0_x4
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:554: GGML_ASSERT(false) failed
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: iqk_mul_mat: Nx = 1, Ny = 1, ne00 = 2048, num_rows = 16, types = q8_0, q8_0_x4
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:554: GGML_ASSERT(false) failed
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: iqk_mul_mat: Nx = 1, Ny = 1, ne00 = 2048, num_rows = 16, types = q8_0, q8_0_x4
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:554: GGML_ASSERT(false) failedGGML_ASSERT(false) failed
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:554: GGML_ASSERT(false) failed
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: iqk_mul_mat: Nx = 1, Ny = 1, ne00 = 2048, num_rows = 16, types = q8_0, q8_0_x4
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:554: GGML_ASSERT(false) failed
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:554: GGML_ASSERT(false) failed
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: iqk_mul_mat: Nx = 1, Ny = 1, ne00 = 2048, num_rows = 16, types = q8_0, q8_0_x4
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388881]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:554: GGML_ASSERT(false) failed
Aug 23 11:54:30 xxx run-ik_llama.cpp.sh[388863]: /opt/THIREUS-6.2478bpw/run-ik_llama.cpp.sh: line 55: 388881 Aborted                 CUDA_VISIBLE_DEVICES="0,1,2" /opt/ik_llama.cpp/ik_llama.cpp/build/bin/llama-server --model /opt/GGUF-Tool-Suite/GGUF-Tool-Suite/DeepSeek-R1-0528.ROOT-6.2478bpw/DeepSeek-R1-0528-THIREUS-BF16-SPECIAL_TENSOR-00001-of-01148.gguf --alias THIREUS/DeepSeek-R1-0528-6.2478bpw --ctx-size $((160 * 1024)) -b $((16 * 512)) -ub $((8 * 512)) --mlock --temp 0.5 --top-k 0 --top-p 1.0 --min-p 0.1 --repeat-penalty 1.0 -ctk q8_0 -mla 3 -fa -amb 512 --split-mode layer --tensor-split 10,21,20 --main-gpu 1 --override-tensor exps=CPU --n-gpu-layers 99 --threads $(grep ^cpu\\scores /proc/cpuinfo | uniq | awk '{print $4}' | xargs -I{} echo "{}-0" | bc) --host 0.0.0.0 --port 8080 --lookup-cache-dynamic /mnt/data/ik_llama.kv.dump --log-enable --logdir /var/log/ --jinja --special --verbose-prompt --verbosity 2
```

---

👤 **ikawrakow** commented on **2025-08-23** at **11:59:40**

And now? (added another commit to [#722](https://github.com/ikawrakow/ik_llama.cpp/issues/722))

---

👤 **magikRUKKOLA** commented on **2025-08-23** at **12:20:14**

@ikawrakow 
> And now? (added another commit to [[#722](https://github.com/ikawrakow/ik_llama.cpp/issues/722)](https://github.com/ikawrakow/ik_llama.cpp/pull/722))

No crash.