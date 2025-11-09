## 🗣️ [Discussion #808](https://github.com/ikawrakow/ik_llama.cpp/discussions/808) - Successfully built, but encountering errors during building and generating gibberish

| **Author** | `knguyen298` |
| :--- | :--- |
| **State** | ✅ **Open** |
| **Created** | 2025-09-29 |
| **Updated** | 2025-09-30 |

---

## 📄 Description

While I'm able to build ik_llama to the point where it runs, I'm not able to get meaningful generation with any model. During building, I'm encountering the following messages:

```
[ 19%] Built target build_info
In function ‘SHA1Update’,
    inlined from ‘SHA1Final’ at /ssd/ik_llama.cpp/examples/gguf-hash/deps/sha1/s                                                                                                                                                                                                                                             ha1.c:265:5:
/ssd/ik_llama.cpp/examples/gguf-hash/deps/sha1/sha1.c:219:13: warning: ‘SHA1Tran                                                                                                                                                                                                                                             sform’ reading 64 bytes from a region of size 0 [-Wstringop-overread]
  219 |             SHA1Transform(context->state, &data[i]);
      |             ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
/ssd/ik_llama.cpp/examples/gguf-hash/deps/sha1/sha1.c:219:13: note: referencing                                                                                                                                                                                                                                              argument 2 of type ‘const unsigned char[64]’
/ssd/ik_llama.cpp/examples/gguf-hash/deps/sha1/sha1.c: In function ‘SHA1Final’:
/ssd/ik_llama.cpp/examples/gguf-hash/deps/sha1/sha1.c:54:6: note: in a call to f                                                                                                                                                                                                                                             unction ‘SHA1Transform’
   54 | void SHA1Transform(
      |      ^~~~~~~~~~~~~
In function ‘SHA1Update’,
    inlined from ‘SHA1Final’ at /ssd/ik_llama.cpp/examples/gguf-hash/deps/sha1/s                                                                                                                                                                                                                                             ha1.c:269:9:
/ssd/ik_llama.cpp/examples/gguf-hash/deps/sha1/sha1.c:219:13: warning: ‘SHA1Tran                                                                                                                                                                                                                                             sform’ reading 64 bytes from a region of size 0 [-Wstringop-overread]
  219 |             SHA1Transform(context->state, &data[i]);
      |             ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
/ssd/ik_llama.cpp/examples/gguf-hash/deps/sha1/sha1.c:219:13: note: referencing                                                                                                                                                                                                                                              argument 2 of type ‘const unsigned char[64]’
/ssd/ik_llama.cpp/examples/gguf-hash/deps/sha1/sha1.c: In function ‘SHA1Final’:
/ssd/ik_llama.cpp/examples/gguf-hash/deps/sha1/sha1.c:54:6: note: in a call to f                                                                                                                                                                                                                                             unction ‘SHA1Transform’
   54 | void SHA1Transform(
```
The following log output appears a few times while building `mmq-instance-iq4_ks_id.cu.o`:
```
/ssd/ik_llama.cpp/ggml/src/ggml-cuda/template-instances/../mmq.cuh(400): warning #177-D: variable "shift" was declared but never referenced
          detected during:
            instantiation of class "mmq_type_traits<mmq_x, mmq_y, nwarps, need_check, GGML_TYPE_IQ1_S_R4> [with mmq_x=8, mmq_y=64, nwarps=8, need_check=false]"
(3854): here
            instantiation of "void mul_mat_q_process_tile<type,mmq_x,nwarps,need_check,fixup>(const char *, const char *, float *, float *, const int &, const int &, const int &, const int &, const int &, const int &, const int &, const int &, const int &, const int &, const int &) [with type=GGML_TYPE_IQ1_S_R4, mmq_x=8, nwarps=8, need_check=false, fixup=false]"
(3954): here
            instantiation of "void mul_mat_q<type,mmq_x,nwarps,need_check>(const char *, const char *, float *, float *, int, int, int, int, int, int, int) [with type=GGML_TYPE_IQ1_S_R4, mmq_x=8, nwarps=8, need_check=false]"
(4122): here
            instantiation of "void launch_mul_mat_q<type,mmq_x>(ggml_backend_cuda_context &, const mmq_args &, cudaStream_t) [with type=GGML_TYPE_IQ1_S_R4, mmq_x=8]"
(4206): here
            instantiation of "void mul_mat_q_case<type>(ggml_backend_cuda_context &, const mmq_args &, cudaStream_t) [with type=GGML_TYPE_IQ1_S_R4]"
```
Despite these messages, I can still run the server and access the web GUI and API. Both methods of input return garbage text, using various different models (GPT-OSS 20B/120B and GLM 4.5 Air) and both special ik_llama quants and normal quants. Adjusting parameters does not fix the issue. The model files work fine in normal llama.cpp. llama.cpp is not installed system-wide. I have pulled the repo so all files are updated.

Specs:

- CPU: 2x EPYC 7601
- GPU: 2x Tesla P40
- OS: Ubuntu Server 24.04
- nvcc version: `Cuda compilation tools, release 12.0, V12.0.140 Build cuda_12.0.r12.0/compiler.32267302_0`
- gcc version: `gcc (Ubuntu 13.3.0-6ubuntu2~24.04) 13.3.0`
-Nvidia driver version: 570.172.08 

Any assistance would be appreciated - the garbage text that does generate does generate much faster than llama.cpp, so looking forward to using it in a functioning state!

---

## 💬 Discussion

👤 **magikRUKKOLA** commented on **2025-09-29** at **22:13:27**

This is very interesting, but why you're using such an old software?  Have you tried to upgrade?  Below is my config and I am having about zero issues -- the software is crazy stable.

```
nvcc --version
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2025 NVIDIA Corporation
Built on Wed_Aug_20_01:58:59_PM_PDT_2025
Cuda compilation tools, release 13.0, V13.0.88
Build cuda_13.0.r13.0/compiler.36424714_0
```

```
gcc --version
gcc (Debian 15.2.0-4) 15.2.0
Copyright (C) 2025 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```

```
Detected 3 NVIDIA GPU(s)

NVOC System Report
==================
System: Linux xxx 6.16.7+deb14-amd64 x86_64
Driver Version: 580.82.09

GPU 0: NVIDIA GeForce RTX 3090
------------------------------------------------
  PCI Bus ID:        00000000:41:00.0
  VBIOS Version:     94.02.4B.00.0B
  Persistence Mode:  Disabled
  Core Temperature:  38°C
  Power Usage:       28W
  Current Power Limit: 350W
  Power Limits:      Default: 350W, Min: 100W, Max: 400W
  GPU Clock:         210 MHz
  VRAM Clock:        405 MHz
  GPU Utilization:   0%
  VRAM Utilization:  0%
  Memory Usage:      21.8 / 24.0 GB
  Applied Offsets:   GPU: 0 MHz, VRAM: 0 MHz

GPU 1: NVIDIA GeForce RTX 3090
------------------------------------------------
  PCI Bus ID:        00000000:42:00.0
  VBIOS Version:     94.02.4B.00.0B
  Persistence Mode:  Disabled
  Core Temperature:  36°C
  Power Usage:       28W
  Current Power Limit: 350W
  Power Limits:      Default: 350W, Min: 100W, Max: 400W
  GPU Clock:         210 MHz
  VRAM Clock:        405 MHz
  GPU Utilization:   0%
  VRAM Utilization:  0%
  Memory Usage:      22.4 / 24.0 GB
  Applied Offsets:   GPU: 0 MHz, VRAM: 0 MHz

GPU 2: NVIDIA GeForce RTX 3090
------------------------------------------------
  PCI Bus ID:        00000000:61:00.0
  VBIOS Version:     94.02.4B.00.0B
  Persistence Mode:  Disabled
  Core Temperature:  38°C
  Power Usage:       31W
  Current Power Limit: 350W
  Power Limits:      Default: 350W, Min: 100W, Max: 400W
  GPU Clock:         210 MHz
  VRAM Clock:        405 MHz
  GPU Utilization:   0%
  VRAM Utilization:  0%
  Memory Usage:      22.5 / 24.0 GB
  Applied Offsets:   GPU: 0 MHz, VRAM: 0 MHz


Peer-to-Peer (P2P) Support Matrix:
=================================
GPU 0 -> GPU 1: Supported
GPU 0 -> GPU 2: Supported
GPU 1 -> GPU 0: Supported
GPU 1 -> GPU 2: Supported
GPU 2 -> GPU 0: Supported
GPU 2 -> GPU 1: Supported

Report generated at: Mon Sep 29 22:10:23 2025
==================
```

> 👤 **knguyen298** replied on **2025-09-30** at **17:37:18**
> 
> That's just the version that is available by default on Ubuntu Server 24.04. I attempted to uninstall `nvidia-cuda-toolkit` and installed CUDA toolkit 12.9, but that just resulted in CMake not finding nvcc, even though running `nvcc --version` works. Any insight on this?

> 👤 **magikRUKKOLA** replied on **2025-09-30** at **20:45:13**
> 
> @knguyen298 
> 
> > That's just the version that is available by default on Ubuntu Server 24.04. I attempted to uninstall `nvidia-cuda-toolkit` and installed CUDA toolkit 12.9, but that just resulted in CMake not finding nvcc, even though running `nvcc --version` works. Any insight on this?
> 
> I am using [the] latest Debian.  As related to the nvidia software here is the make.sh:
> 
> ```
> #!/usr/bin/env bash
> 
> apt install openssl
> 
> mkdir -p /lib/modules/$(uname -r)/build/certs
> cd /lib/modules/$(uname -r)/build/certs
> 
> sudo tee x509.genkey > /dev/null << 'EOF'
> [ req ]
> default_bits = 4096
> distinguished_name = req_distinguished_name
> prompt = no
> string_mask = utf8only
> x509_extensions = myexts
> [ req_distinguished_name ]
> CN = Modules
> [ myexts ]
> basicConstraints=critical,CA:FALSE
> keyUsage=digitalSignature
> subjectKeyIdentifier=hash
> authorityKeyIdentifier=keyid
> EOF
> openssl req -new -nodes -utf8 -sha512 -days 36500 -batch -x509 -config x509.genkey -outform DER -out signing_key.x509 -keyout signing_key.pem
> ln -fs /lib/modules/$(uname -r)/build/certs /usr/src/linux-headers-$(uname -r | cut -d'-' -f1)-common/
> 
> cd -
> 
> # https://developer.download.nvidia.com/compute/cuda/repos/debian12/x86_64/cuda-keyring_1.1-1_all.deb
> # https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2404/x86_64/cuda-keyring_1.1-1_all.deb
> apt -y install -f ./cuda-keyring_1.1-1_all.deb
> apt -y update
> 
> #./cuda_13.0.0_580.65.06_linux.run
> apt -y install cuda-13-0
> 
> apt install --reinstall -y cudnn libglx-nvidia0
> ./NVIDIA-Linux-x86_64-580.82.09.run
> #ln -rs /usr/src/linux-headers-6.12.41+deb13-common/certs /usr/src/linux-headers-6.12.41+deb13-common/output
> 
> # P2P support for SM=86+
> # https://github.com/aikitoria/open-gpu-kernel-modules
> cd open-gpu-kernel-modules/
> export IGNORE_CC_MISMATCH=1
> ./install.sh
> ```
> 
> You will not need the P2P support since your GPU doesn't support it so just ski[p] the last step (as well as the step of the creation of the certificates in the beginning).  So basically I am using the cuda-keyring and installing cuda-13 which have the nvcc included:
> 
> ```
> /usr/local/cuda/bin/nvcc --version
> ```
> 
> Make sure to include something similar to the following into .bashrc of course:
> 
> ```
> export PATH="/usr/sbin:$PATH"
> export PATH="$HOME/.local/bin:$PATH"
> export PATH="/usr/local/cuda/bin:$PATH"
> export LD_LIBRARY_PATH="/usr/local/cuda/lib64:$LD_LIBRARY_PATH"
> export LD_LIBRARY_PATH="/usr/local/lib:$LD_LIBRARY_PATH"
> ```

---

👤 **magikRUKKOLA** commented on **2025-09-29** at **22:17:46**

BTW what is your build command?

Here is the example:

make.sh 
```
#!/usr/bin/env bash
cd ik_llama.cpp
cmake -B build \
  -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_CUDA_ARCHITECTURES="86" \
  -DGGML_CUDA=ON \
  -DGGML_CUDA_FA_ALL_QUANTS=1 \
  -DGGML_SCHED_MAX_COPIES=1 \
  -DGGML_CUDA_IQK_FORCE_BF16=1 \
  -DGGML_MAX_CONTEXTS=2048 \
  -DGGML_VULKAN=OFF \
  -DGGML_CUDA_F16=ON
cmake --build build --config Release -j $(nproc)
```

> 👤 **knguyen298** replied on **2025-09-30** at **18:05:23**
> 
> I've kept it pretty simple for ease of troubleshooting:
> ```
> cmake -B build \
>   -DCMAKE_BUILD_TYPE=Release \
>   -DCMAKE_CUDA_ARCHITECTURES="61" \
>   -DGGML_CUDA=ON \
>   -DGGML_SCHED_MAX_COPIES=1
> cmake --build build --config Release -j $(nproc)
> ```

> 👤 **magikRUKKOLA** replied on **2025-09-30** at **20:46:36**
> 
> @knguyen298 
> 
> Well, cool then.  But keep in mind that you will not be able to use quants from Thireus with such config.  Make sure to always add:
> 
> ```
> -DGGML_MAX_CONTEXTS=2048 
> ```