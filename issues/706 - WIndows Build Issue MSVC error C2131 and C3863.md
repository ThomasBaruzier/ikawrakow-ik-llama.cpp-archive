## 📌 [Issue #706](https://github.com/ikawrakow/ik_llama.cpp/issues/706) - WIndows Build Issue (MSVC), error: C2131 and C3863

| **Author** | `sebagallo` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-08-19 |
| **Updated** | 2025-08-20 |

---

## 📄 Description

BUILD COMMAND:
```
set GGML_NLOOP=3
set GGML_N_THREADS=1
set LLAMA_LOG_COLORS=1
set LLAMA_LOG_PREFIX=1
set LLAMA_LOG_TIMESTAMPS=1
set CUDA_PATH=C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v12.4
set CUDA_PATH_V12_4=C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v12.4
set CURL_PATH=D:\Work\AI\curl-8.6.0_6-win64-mingw

call "C:\Program Files\Microsoft Visual Studio\2022\Enterprise\VC\Auxiliary\Build\vcvarsall.bat" x64
cmake -S . -B build -G "Ninja Multi-Config" ^
  -DLLAMA_BUILD_SERVER=ON ^
  -DGGML_NATIVE=OFF ^
  -DGGML_BACKEND_DL=ON ^
  -DGGML_CPU_ALL_VARIANTS=ON ^
  -DGGML_CUDA=ON ^
  -DGGML_RPC=ON ^
  -DCURL_LIBRARY="%CURL_PATH%/lib/libcurl.dll.a" -DCURL_INCLUDE_DIR="%CURL_PATH%/include"
set /A NINJA_JOBS=%NUMBER_OF_PROCESSORS%-1
cmake --build build --config Release -j %NINJA_JOBS% -t ggml
cmake --build build --config Release
```

OUTPUT:
```
[1/2] Building CXX object examples\quantize-stats\CMakeFiles\llama-quantize-stats.dir\Release\quantize-stats.cpp.obj
FAILED: [code=2] examples/quantize-stats/CMakeFiles/llama-quantize-stats.dir/Release/quantize-stats.cpp.obj
ccache C:\PROGRA~2\MICROS~2\2022\BUILDT~1\VC\Tools\MSVC\1444~1.352\bin\Hostx64\x64\cl.exe  /nologo /TP -DGGML_USE_CUDA -DGGML_USE_RPC -D_CRT_SECURE_NO_WARNINGS -DCMAKE_INTDIR=\"Release\" -ID:\Work\AI\ik_llama.cpp\examples -ID:\Work\AI\ik_llama.cpp\examples\quantize-stats\..\..\common -ID:\Work\AI\ik_llama.cpp\src\. -ID:\Work\AI\ik_llama.cpp\src\..\include -ID:\Work\AI\ik_llama.cpp\src\..\common -ID:\Work\AI\ik_llama.cpp\ggml\src\..\include /DWIN32 /D_WINDOWS /W3 /GR /EHsc /MD /O2 /Ob2 /DNDEBUG -std:c++17 /utf-8 /bigobj /showIncludes /Foexamples\quantize-stats\CMakeFiles\llama-quantize-stats.dir\Release\quantize-stats.cpp.obj /Fdexamples\quantize-stats\CMakeFiles\llama-quantize-stats.dir\Release\ /FS -c D:\Work\AI\ik_llama.cpp\examples\quantize-stats\quantize-stats.cpp
D:\Work\AI\ik_llama.cpp\examples\quantize-stats\quantize-stats.cpp(955): error C2131: expression did not evaluate to a constant
D:\Work\AI\ik_llama.cpp\examples\quantize-stats\quantize-stats.cpp(955): note: failure was caused by a read of a variable outside its lifetime
D:\Work\AI\ik_llama.cpp\examples\quantize-stats\quantize-stats.cpp(955): note: see usage of 'this'
D:\Work\AI\ik_llama.cpp\examples\quantize-stats\quantize-stats.cpp(972): error C3863: array type 'float [this->8<L_ALIGN>8]' is not assignable
D:\Work\AI\ik_llama.cpp\examples\quantize-stats\quantize-stats.cpp(974): error C3863: array type 'float [this->8<L_ALIGN>8]' is not assignable
ninja: build stopped: subcommand failed.
```

OS: Windows 11
Build tools: VS2022

---

## 💬 Conversation

👤 **sebagallo** commented on **2025-08-19** at **11:27:05**

changing
955 to std::vector<float> weight(kBlockSize);
and
1178 to std::vector<float> weight(kBlockSize);

made it compile

---

👤 **ikawrakow** commented on **2025-08-19** at **11:36:04**

> changing 955 to std::vector weight(kBlockSize);

You don't want to do a heap allocation each time this lambda is invoked. I think you could file a bug report with MSVC as const expressions known at compile time can be used in lambdas without the need to be capturing them.

But apart from this, this is not mainline `llama.cpp`. `GGML_BACKEND_DL` and `GGML_CPU_ALL_VARIANTS` don't have any meaning here. You most likely want to use `-DGGML_BLAS=OFF`. The Windows build of `ik_llama.cpp`  is nowhere near the polish of `llama.cpp`, so you will need to work with `-GGML_ARCH_FLAGS` to enable specific CPU features (AVX2, AVX512, etc).

---

👤 **sebagallo** commented on **2025-08-19** at **14:07:23**

yeah I was mainly looking for a fast way to make it compile. Would switching to clang 'fix' this issue?
Also, yes, I mainly went with the mainline instructions on how to compile (basically copied what their gh action for the cuda build was doing), it would be nice to have a sort of 'guide' on how to compile this in the recommended way for ik_llama in windows (and linux too?)

```
cmake -S . -B build -G "Ninja Multi-Config" ^
  -DLLAMA_BUILD_SERVER=ON ^
  -DGGML_NATIVE=OFF ^
  -DGGML_BLAS=OFF ^
  -DGGML_CUDA=ON ^
  -DGGML_RPC=ON ^
  -DGGML_ARCH_FLAGS="/arch:AVX512" ^
  -DCURL_LIBRARY="%CURL_PATH%/lib/libcurl.dll.a" -DCURL_INCLUDE_DIR="%CURL_PATH%/include"
```
Anyway, I followed your recommendation and added avx512 flag support for msvc (I know that in clang you could pass something like -march=native to allow it to automatically pickup the supported featureset).
If there's no plan to work around this issue for MSVC, I guess you could close this. Thanks for your reply.

---

👤 **ikawrakow** commented on **2025-08-19** at **14:20:15**

Please see [#707](https://github.com/ikawrakow/ik_llama.cpp/issues/707) and confirm that this fixes the MSVC compilation.

On Windows, your last command looks OK, but perhaps @Thireus and/or other Windows users can also tell us how they build the code. I myself have no access to a Windows box.

On Linux just `cmake -B build -DGGML_CUDA=ON [-DDGGML_RPC=ON]`

---

👤 **saood06** commented on **2025-08-19** at **14:33:43**

>other Windows users can also tell us how they build the code. I myself have no access to a Windows box.

I used to build with `cmake .. -DGGML_CUDA=ON -DGGML_RPC=ON -DGGML_CUDA_FA_ALL_QUANTS=ON` followed by `cmake --build . --config Release -j 6` but after updating cmake on that machine I now build with `cmake .. -G "Visual Studio 16 2019" -DGGML_CUDA=ON -DGGML_RPC=ON -DGGML_IQK_FA_ALL_QUANTS=1 -DCMAKE_TOOLCHAIN_FILE="[...]\vcpkg\scripts\buildsystems\vcpkg.cmake"` the last bit is for sqlite3 which is needed for [#558](https://github.com/ikawrakow/ik_llama.cpp/issues/558)

---

👤 **sebagallo** commented on **2025-08-19** at **15:01:57**

> Please see [[#707](https://github.com/ikawrakow/ik_llama.cpp/issues/707)](https://github.com/ikawrakow/ik_llama.cpp/pull/707) and confirm that this fixes the MSVC compilation.
> 
> On Windows, your last command looks OK, but perhaps [@Thireus](https://github.com/Thireus) and/or other Windows users can also tell us how they build the code. I myself have no access to a Windows box.
> 
> On Linux just `cmake -B build -DGGML_CUDA=ON [-DDGGML_RPC=ON]`

```
D:\Work\AI\ik_llama.cpp>cmake --build build --config Release -j 15 -t ggml
[1/2] Building CUDA object ggml\src\CMakeFiles\ggml.dir\Release\ggml-cuda\cpy.cu.obj
cpy.cu
tmpxft_00006a0c_00000000-10_cpy.compute_80.cudafe1.cpp
[2/2] Linking CXX shared library bin\Release\ggml.dll

D:\Work\AI\ik_llama.cpp>cmake --build build --config Release
[1/58] Generating build details from Git
-- Found Git: C:/Users/sebag/scoop/shims/git.exe (found version "2.49.0.windows.1")
[58/58] Linking CXX executable bin\Release\llama-quantize-stats.exe
```
Success! It built without issues.

---

👤 **ikawrakow** commented on **2025-08-19** at **15:10:24**

Closed via [#707](https://github.com/ikawrakow/ik_llama.cpp/issues/707)

---

👤 **sousekd** commented on **2025-08-19** at **19:46:52**

@sebagallo I build using this on Windows (AMD/Zen5):

```
cmake -B build -G Ninja `
  -DCMAKE_BUILD_TYPE=Release `
  -DCMAKE_C_COMPILER="clang-cl" `
  -DCMAKE_CXX_COMPILER="clang-cl" `
  -DCMAKE_CUDA_HOST_COMPILER="cl.exe" `
  -DGGML_CUDA=ON `
  -DGGML_CUDA_F16=ON `
  -DGGML_AVX512=ON `
  -DGGML_AVX512_VNNI=ON `
  -DGGML_AVX512_VBMI=ON `
  -DGGML_AVX512_BF16=ON `
  -DGGML_SCHED_MAX_COPIES=1 `
  -DGGML_BLAS=OFF `
  -DGGML_CCACHE=OFF `
  -DCMAKE_C_FLAGS='/clang:-march=znver5' `
  -DCMAKE_CXX_FLAGS='/EHsc /clang:-march=znver5' `
  -DCMAKE_CUDA_ARCHITECTURES="native" `
  -DCMAKE_INTERPROCEDURAL_OPTIMIZATION=ON `
  -DLLAMA_CURL=OFF `
  -DBUILD_SHARED_LIBS=OFF

cmake --build build --config Release -j
```

Just for reference.

---

👤 **sebagallo** commented on **2025-08-20** at **21:14:59**

@sousekd Appreciate it, seems like the way to go is using clang instead of msvc/msbuild on windows