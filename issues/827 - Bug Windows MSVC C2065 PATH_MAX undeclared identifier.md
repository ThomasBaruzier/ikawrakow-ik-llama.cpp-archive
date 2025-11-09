## 📌 [Issue #827](https://github.com/ikawrakow/ik_llama.cpp/issues/827) - Bug: Windows MSVC C2065 "PATH_MAX" undeclared identifier

| **Author** | `GBLKai` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-10-13 |
| **Updated** | 2025-10-13 |

---

## 📄 Description

### What happened?

Windows MSVC seems unable to compile.
`cmake -B ./build -DGGML_CUDA=ON -DGGML_BLAS=OFF`
`cmake --build ./build --config Release -j 32`

error C2065 "PATH_MAX" undeclared identifier [ik_llama.cpp\build\src\llama.vcxproj]
error C1903


### Name and Version

2025 Oct 13,`0030bc89c993c3868252dfc34994ff8452b5abaf`

### What operating system are you seeing the problem on?

Windows

### Relevant log output

```shell
-- Building for: Visual Studio 17 2022
-- Selecting Windows SDK version 10.0.26100.0 to target Windows 10.0.19045.
-- The C compiler identification is MSVC 19.44.35214.0
-- The CXX compiler identification is MSVC 19.44.35214.0
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working C compiler: F:/Microsoft Visual Studio/2022/Community/VC/Tools/MSVC/14.44.35207/bin/Hostx64/x64/cl.exe - skipped
-- Detecting C compile features
-- Detecting C compile features - done
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Check for working CXX compiler: F:/Microsoft Visual Studio/2022/Community/VC/Tools/MSVC/14.44.35207/bin/Hostx64/x64/cl.exe - skipped
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Found Git: C:/Program Files/Git/cmd/git.exe (found version "2.38.0.windows.1")
-- Performing Test CMAKE_HAVE_LIBC_PTHREAD
-- Performing Test CMAKE_HAVE_LIBC_PTHREAD - Failed
-- Looking for pthread_create in pthreads
-- Looking for pthread_create in pthreads - not found
-- Looking for pthread_create in pthread
-- Looking for pthread_create in pthread - not found
-- Found Threads: TRUE
-- Found OpenMP_C: -openmp (found version "2.0")
-- Found OpenMP_CXX: -openmp (found version "2.0")
-- Found OpenMP: TRUE (found version "2.0")
-- OpenMP found
-- Using optimized iqk matrix multiplications
-- Enabling IQK Flash Attention kernels
-- Using llamafile
-- Found CUDAToolkit: C:/Program Files/NVIDIA GPU Computing Toolkit/CUDA/v12.8/include (found version "12.8.61")
-- CUDA found
-- Using CUDA architectures: native
-- The CUDA compiler identification is NVIDIA 12.8.61 with host compiler MSVC 19.44.35214.0
-- Detecting CUDA compiler ABI info
-- Detecting CUDA compiler ABI info - done
-- Check for working CUDA compiler: C:/Program Files/NVIDIA GPU Computing Toolkit/CUDA/v12.8/bin/nvcc.exe - skipped
-- Detecting CUDA compile features
-- Detecting CUDA compile features - done
-- Warning: ccache not found - consider installing it for faster compilation or disable this warning with GGML_CCACHE=OFF
-- CMAKE_SYSTEM_PROCESSOR: AMD64
-- CMAKE_GENERATOR_PLATFORM:
-- x86 detected
-- Performing Test HAS_AVX_1
-- Performing Test HAS_AVX_1 - Success
-- Performing Test HAS_AVX2_1
-- Performing Test HAS_AVX2_1 - Success
-- Performing Test HAS_FMA_1
-- Performing Test HAS_FMA_1 - Success
-- Performing Test HAS_AVX512_1
-- Performing Test HAS_AVX512_1 - Failed
-- Performing Test HAS_AVX512_2
-- Performing Test HAS_AVX512_2 - Failed
-- ARCH_FLAGS =
-- Configuring done (36.2s)
-- Generating done (1.0s)
-- Build files have been written to: /ik_llama.cpp/build
```

---

## 💬 Conversation

👤 **ikawrakow** commented on **2025-10-13** at **06:07:47**

Can you add the actual compiler error (including file and line)? Thanks.

---

👤 **GBLKai** commented on **2025-10-13** at **06:15:58**

> Can you add the actual compiler error (including file and line)? Thanks.

I just downloaded and successfully compiled the old version 2025 Sep 30, please wait, also I am not really sure how to know where the error file, I will provide the cmd print info.

---

👤 **GBLKai** commented on **2025-10-13** at **06:23:34**

> Can you add the actual compiler error (including file and line)? Thanks.

```
\ik_llama.cpp\src\llama-quantize.cpp(1183,29): error C2065: “PATH_MAX”: 未声明的标识符 [\ik_llama.cpp\build\src\llama.vcxproj]
\ik_llama.cpp\src\llama-quantize.cpp(1184,42): error C1903: 无法从以前的错误中恢复；正在停止编译 [\ik_llama.cpp\build\src\llama.vcxproj]
```
Hope this helps.

---

👤 **ikawrakow** commented on **2025-10-13** at **06:26:22**

Should be good now.

---

👤 **GBLKai** commented on **2025-10-13** at **06:28:13**

> Should be good now.

Very quick, I will try again, thanks.