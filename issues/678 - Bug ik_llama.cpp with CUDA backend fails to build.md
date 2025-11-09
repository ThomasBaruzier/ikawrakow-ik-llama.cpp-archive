## 📌 [Issue #678](https://github.com/ikawrakow/ik_llama.cpp/issues/678) - Bug: ik_llama.cpp with CUDA backend fails to build

| **Author** | `Orion-zhen` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-08-08 |
| **Updated** | 2025-08-12 |

---

## 📄 Description

## Build command

```bash
cmake -B build -DGGML_CUDA=ON -DGGML_LTO=ON -DGGML_RPC=ON && cmake --build build --config Release
```

## Output

```
-- The C compiler identification is GNU 15.1.1
-- The CXX compiler identification is GNU 15.1.1
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working C compiler: /usr/bin/cc - skipped
-- Detecting C compile features
-- Detecting C compile features - done
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Check for working CXX compiler: /usr/bin/c++ - skipped
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Found Git: /usr/bin/git (found version "2.50.1")
-- Performing Test CMAKE_HAVE_LIBC_PTHREAD
-- Performing Test CMAKE_HAVE_LIBC_PTHREAD - Success
-- Found Threads: TRUE
-- Found OpenMP_C: -fopenmp (found version "4.5")
-- Found OpenMP_CXX: -fopenmp (found version "4.5")
-- Found OpenMP: TRUE (found version "4.5")
-- OpenMP found
-- Using optimized iqk matrix multiplications
-- Enabling IQK Flash Attention kernels
-- Using llamafile
-- Found CUDAToolkit: /opt/cuda/targets/x86_64-linux/include (found version "12.9.86")
-- CUDA found
-- Using CUDA architectures: native
CMake Error at /usr/share/cmake/Modules/CMakeDetermineCompilerId.cmake:909 (message):
  Compiling the CUDA compiler identification source file
  "CMakeCUDACompilerId.cu" failed.

  Compiler: /opt/cuda/bin/nvcc

  Build flags:

  Id flags: --keep;--keep-dir;tmp -v



  The output was:

  2

  nvcc warning : Support for offline compilation for architectures prior to
  '<compute/sm/lto>_75' will be removed in a future release (Use
  -Wno-deprecated-gpu-targets to suppress warning).

  #$ _NVVM_BRANCH_=nvvm

  #$ _SPACE_=

  #$ _CUDART_=cudart

  #$ _HERE_=/opt/cuda/bin

  #$ _THERE_=/opt/cuda/bin

  #$ _TARGET_SIZE_=

  #$ _TARGET_DIR_=

  #$ _TARGET_DIR_=targets/x86_64-linux

  #$ TOP=/opt/cuda/bin/..

  #$ CICC_PATH=/opt/cuda/bin/../nvvm/bin

  #$ NVVMIR_LIBRARY_DIR=/opt/cuda/bin/../nvvm/libdevice

  #$ LD_LIBRARY_PATH=/opt/cuda/bin/../lib:

  #$
  PATH=/opt/cuda/bin/../nvvm/bin:/opt/cuda/bin:/usr/local/sbin:/usr/local/bin:/usr/bin:/home/orion/.local/bin:/home/orion/.cargo/bin:/opt/cuda/bin:/usr/bin/site_perl:/usr/bin/vendor_perl:/usr/bin/core_perl

  #$ INCLUDES="-I/opt/cuda/bin/../targets/x86_64-linux/include"

  #$ LIBRARIES= "-L/opt/cuda/bin/../targets/x86_64-linux/lib/stubs"
  "-L/opt/cuda/bin/../targets/x86_64-linux/lib"

  #$ CUDAFE_FLAGS=

  #$ PTXAS_FLAGS=

  #$ rm tmp/a_dlink.reg.c

  #$ gcc -D__CUDA_ARCH_LIST__=520 -D__NV_LEGACY_LAUNCH -E -x c++ -D__CUDACC__
  -D__NVCC__ "-I/opt/cuda/bin/../targets/x86_64-linux/include"
  -D__CUDACC_VER_MAJOR__=12 -D__CUDACC_VER_MINOR__=9
  -D__CUDACC_VER_BUILD__=86 -D__CUDA_API_VER_MAJOR__=12
  -D__CUDA_API_VER_MINOR__=9 -D__NVCC_DIAG_PRAGMA_SUPPORT__=1
  -D__CUDACC_DEVICE_ATOMIC_BUILTINS__=1 -include "cuda_runtime.h" -m64
  "CMakeCUDACompilerId.cu" -o "tmp/CMakeCUDACompilerId.cpp4.ii"

  #$ cudafe++ --c++17 --gnu_version=150101 --display_error_number
  --orig_src_file_name "CMakeCUDACompilerId.cu" --orig_src_path_name
  "/home/orion/repo/ik-llama.cpp-cuda/src/build/CMakeFiles/4.0.3-dirty/CompilerIdCUDA/CMakeCUDACompilerId.cu"
  --allow_managed --m64 --parse_templates --gen_c_file_name
  "tmp/CMakeCUDACompilerId.cudafe1.cpp" --stub_file_name
  "CMakeCUDACompilerId.cudafe1.stub.c" --gen_module_id_file
  --module_id_file_name "tmp/CMakeCUDACompilerId.module_id"
  "tmp/CMakeCUDACompilerId.cpp4.ii"

  /usr/include/c++/15.1.1/type_traits(554): error: type name is not allowed

        : public __bool_constant<__is_pointer(_Tp)>
                                              ^



  /usr/include/c++/15.1.1/type_traits(554): error: identifier "__is_pointer"
  is undefined

        : public __bool_constant<__is_pointer(_Tp)>
                                 ^



  /usr/include/c++/15.1.1/type_traits(876): error: type name is not allowed

        : public __bool_constant<__is_volatile(_Tp)>
                                               ^



  /usr/include/c++/15.1.1/type_traits(876): error: identifier "__is_volatile"
  is undefined

        : public __bool_constant<__is_volatile(_Tp)>
                                 ^



  /usr/include/c++/15.1.1/type_traits(1491): error: type name is not allowed

        : public integral_constant<std::size_t, __array_rank(_Tp)> { };
                                                             ^



  /usr/include/c++/15.1.1/type_traits(1491): error: identifier "__array_rank"
  is undefined

        : public integral_constant<std::size_t, __array_rank(_Tp)> { };
                                                ^



  /usr/include/c++/15.1.1/type_traits(1843): error: incomplete type
  "std::__cv_selector<std::__make_unsigned_selector<wchar_t, false,
  true>::__unsigned_type, false, <error-constant>>" (aka
  "std::__cv_selector<unsigned int, false, <error-constant>>") is not allowed

          using __type = typename __match::__type;
                                  ^
            detected during:
              instantiation of class "std::__match_cv_qualifiers<_Qualified, _Unqualified, _IsConst, _IsVol> [with _Qualified=wchar_t, _Unqualified=unsigned int, _IsConst=false, _IsVol=<error-constant>]" at line 1952
              instantiation of class "std::__make_unsigned_selector<_Tp, false, true> [with _Tp=wchar_t]" at line 1963



  /usr/include/c++/15.1.1/type_traits(1843): error: incomplete type
  "std::__cv_selector<std::__make_unsigned_selector<char16_t, false,
  true>::__unsigned_type, false, <error-constant>>" (aka
  "std::__cv_selector<unsigned short, false, <error-constant>>") is not
  allowed

          using __type = typename __match::__type;
                                  ^
            detected during:
              instantiation of class "std::__match_cv_qualifiers<_Qualified, _Unqualified, _IsConst, _IsVol> [with _Qualified=char16_t, _Unqualified=unsigned short, _IsConst=false, _IsVol=<error-constant>]" at line 1952
              instantiation of class "std::__make_unsigned_selector<_Tp, false, true> [with _Tp=char16_t]" at line 1979



  /usr/include/c++/15.1.1/type_traits(1843): error: incomplete type
  "std::__cv_selector<std::__make_unsigned_selector<wchar_t, false,
  true>::__unsigned_type, false, <error-constant>>" (aka
  "std::__cv_selector<unsigned int, false, <error-constant>>") is not allowed

          using __type = typename __match::__type;
                                  ^
            detected during:
              instantiation of class "std::__match_cv_qualifiers<_Qualified, _Unqualified, _IsConst, _IsVol> [with _Qualified=char32_t, _Unqualified=unsigned int, _IsConst=false, _IsVol=<error-constant>]" at line 1952
              instantiation of class "std::__make_unsigned_selector<_Tp, false, true> [with _Tp=char32_t]" at line 1986



  /usr/include/c++/15.1.1/type_traits(1843): error: incomplete type
  "std::__cv_selector<std::__make_unsigned_selector<wchar_t, true,
  false>::__unsigned_type, false, <error-constant>>" is not allowed

          using __type = typename __match::__type;
                                  ^
            detected during:
              instantiation of class "std::__match_cv_qualifiers<_Qualified, _Unqualified, _IsConst, _IsVol> [with _Qualified=wchar_t, _Unqualified=<error-type>, _IsConst=false, _IsVol=<error-constant>]" at line 1914
              instantiation of class "std::__make_unsigned_selector<_Tp, true, false> [with _Tp=wchar_t]" at line 2081
              instantiation of class "std::__make_signed_selector<_Tp, false, true> [with _Tp=wchar_t]" at line 2095



  /usr/include/c++/15.1.1/type_traits(2084): error: incomplete type
  "std::__make_signed_selector<std::__make_signed_selector<wchar_t, false,
  true>::__unsigned_type, false, <error-constant>>" is not allowed

          using __type = typename __make_signed_selector<__unsigned_type>::__type;
                                  ^
            detected during instantiation of class "std::__make_signed_selector<_Tp, false, true> [with _Tp=wchar_t]" at line 2095



  /usr/include/c++/15.1.1/type_traits(1843): error: incomplete type
  "std::__cv_selector<std::__make_unsigned_selector<wchar_t, true,
  false>::__unsigned_type, false, <error-constant>>" is not allowed

          using __type = typename __match::__type;
                                  ^
            detected during:
              instantiation of class "std::__match_cv_qualifiers<_Qualified, _Unqualified, _IsConst, _IsVol> [with _Qualified=char16_t, _Unqualified=<error-type>, _IsConst=false, _IsVol=<error-constant>]" at line 1914
              instantiation of class "std::__make_unsigned_selector<_Tp, true, false> [with _Tp=char16_t]" at line 2081
              instantiation of class "std::__make_signed_selector<_Tp, false, true> [with _Tp=char16_t]" at line 2111



  /usr/include/c++/15.1.1/type_traits(2084): error: incomplete type
  "std::__make_signed_selector<std::__make_signed_selector<wchar_t, false,
  true>::__unsigned_type, false, <error-constant>>" is not allowed

          using __type = typename __make_signed_selector<__unsigned_type>::__type;
                                  ^
            detected during instantiation of class "std::__make_signed_selector<_Tp, false, true> [with _Tp=char16_t]" at line 2111



  /usr/include/c++/15.1.1/type_traits(1843): error: incomplete type
  "std::__cv_selector<std::__make_unsigned_selector<wchar_t, true,
  false>::__unsigned_type, false, <error-constant>>" is not allowed

          using __type = typename __match::__type;
                                  ^
            detected during:
              instantiation of class "std::__match_cv_qualifiers<_Qualified, _Unqualified, _IsConst, _IsVol> [with _Qualified=char32_t, _Unqualified=<error-type>, _IsConst=false, _IsVol=<error-constant>]" at line 1914
              instantiation of class "std::__make_unsigned_selector<_Tp, true, false> [with _Tp=char32_t]" at line 2081
              instantiation of class "std::__make_signed_selector<_Tp, false, true> [with _Tp=char32_t]" at line 2118



  /usr/include/c++/15.1.1/type_traits(2084): error: incomplete type
  "std::__make_signed_selector<std::__make_signed_selector<wchar_t, false,
  true>::__unsigned_type, false, <error-constant>>" is not allowed

          using __type = typename __make_signed_selector<__unsigned_type>::__type;
                                  ^
            detected during instantiation of class "std::__make_signed_selector<_Tp, false, true> [with _Tp=char32_t]" at line 2118



  /usr/include/c++/15.1.1/type_traits(3302): error: type name is not allowed

        : public __bool_constant<__is_invocable(_Fn, _ArgTypes...)>
                                                ^



  /usr/include/c++/15.1.1/type_traits(3302): error: type name is not allowed

        : public __bool_constant<__is_invocable(_Fn, _ArgTypes...)>
                                                     ^



  /usr/include/c++/15.1.1/type_traits(3332): error: type name is not allowed

        : public __bool_constant<__is_nothrow_invocable(_Fn, _ArgTypes...)>
                                                        ^



  /usr/include/c++/15.1.1/type_traits(3332): error: type name is not allowed

        : public __bool_constant<__is_nothrow_invocable(_Fn, _ArgTypes...)>
                                                             ^



  /usr/include/c++/15.1.1/type_traits(3408): error: type name is not allowed

      inline constexpr bool is_pointer_v = __is_pointer(_Tp);
                                                        ^



  /usr/include/c++/15.1.1/type_traits(3521): error: type name is not allowed

      inline constexpr bool is_volatile_v = __is_volatile(_Tp);
                                                          ^



  /usr/include/c++/15.1.1/type_traits(3663): error: type name is not allowed

      inline constexpr size_t rank_v = __array_rank(_Tp);
                                                    ^



  /usr/include/c++/15.1.1/bits/stl_algobase.h(1239): error: type name is not
  allowed

        || __is_pointer(_ValueType1)
                        ^



  /usr/include/c++/15.1.1/bits/stl_algobase.h(1412): error: type name is not
  allowed

      && __is_pointer(_II1) && __is_pointer(_II2)
                      ^



  /usr/include/c++/15.1.1/bits/stl_algobase.h(1412): error: type name is not
  allowed

      && __is_pointer(_II1) && __is_pointer(_II2)
                                            ^



  25 errors detected in the compilation of "CMakeCUDACompilerId.cu".

  # --error 0x2 --





Call Stack (most recent call first):
  /usr/share/cmake/Modules/CMakeDetermineCompilerId.cmake:8 (CMAKE_DETERMINE_COMPILER_ID_BUILD)
  /usr/share/cmake/Modules/CMakeDetermineCompilerId.cmake:53 (__determine_compiler_id_test)
  /usr/share/cmake/Modules/CMakeDetermineCUDACompiler.cmake:139 (CMAKE_DETERMINE_COMPILER_ID)
  ggml/src/CMakeLists.txt:346 (enable_language)


-- Configuring incomplete, errors occurred!
```

## System info

- OS: Arch Linux
- GPU: NVIDIA RTX 3060 12G
- nvidia driver: 575.64.05-4 from extra repo
- nvcc:

```
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2025 NVIDIA Corporation
Built on Tue_May_27_02:21:03_PDT_2025
Cuda compilation tools, release 12.9, V12.9.86
Build cuda_12.9.r12.9/compiler.36037853_0
```

## More info

I was able to build and run models before commit [`ae0ba31fd078282fe6ac675176862ed6955c52dc`](https://github.com/ikawrakow/ik_llama.cpp/commit/ae0ba31fd078282fe6ac675176862ed6955c52dc), but after I pulled new commits today, it failed to build.

---

## 💬 Conversation

👤 **ikawrakow** commented on **2025-08-08** at **05:30:28**

This looks like a compiler issue. You are sure you did not change compilers since your last successful build?

---

👤 **Orion-zhen** commented on **2025-08-08** at **08:19:49**

> This looks like a compiler issue. You are sure you did not change compilers since your last successful build?

Yes, I am sure that I did not change any compilers intendedly since my last successful build. But I did make a system upgrade.

I have just checkout to the last successful commit, but it failed again with the same error. I have no idea on this 😢

---

👤 **ikawrakow** commented on **2025-08-08** at **08:26:07**

> Yes, I am sure that I did not change any compilers intendedly since my last successful build. But I did make a system upgrade.

A system upgrade typically comes with compiler and driver changes. IIRC, I had a similar issue when upgrading to Ubuntu 24.04 and using the open Nvidia driver/Ubuntu CUDA packages. Installing the proprietary driver along with the CUDA toolkit from Nvidia fixed it.

---

👤 **Orion-zhen** commented on **2025-08-08** at **08:54:03**

All compilers and drivers are installed and managed via `pacman`, so it shouldn't break my system. How weird it is.

---

👤 **ikawrakow** commented on **2025-08-08** at **09:02:57**

In my case it is `apt`, and I also thought that the official Ubuntu packages should work.

Out of curiosity, can you successfully build mainline `llama.cpp` with your current setup?

---

👤 **Orion-zhen** commented on **2025-08-08** at **09:06:53**

Yes, I'm currently using mainline `llama.cpp` to serve models. I just launched a build task and it finished smoothly.

EDIT:

build args:

```
-DGGML_CUDA=ON
-DGGML_CUDA_F16=ON
-DGGML_NATIVE
-DGGML_LTO=ON
-DGGML_RPC=ON
-DGGML_CUDA_GRAPHS=ON
-DGGML_FORCE_CUBLAS=ON
-DGGML_CUDA_FA_ALL_QUANTS=ON
```

---

👤 **ikawrakow** commented on **2025-08-08** at **09:39:12**

A fresh build task in a new folder?

---

👤 **Orion-zhen** commented on **2025-08-08** at **09:42:22**

Yes

---

👤 **ikawrakow** commented on **2025-08-08** at **09:47:55**

OK, in that case I need to figure out what they have changed in mainline.

---

👤 **Ph0rk0z** commented on **2025-08-08** at **11:11:11**

>-DGGML_CUDA_FA_ALL_QUANTS=ON

I had linking errors for a couple weeks and had to turn this off. During LTO the build would fail. Gave up and just turned it off. I wasn't using mixed quanted cache anymore anyways. That's here, not in mainline.

---

👤 **ikawrakow** commented on **2025-08-08** at **11:49:37**

@Orion-zhen Can you attach the `CMakeCache.txt` in the `ik_llama.cpp` build folder? Thanks.

---

👤 **ikawrakow** commented on **2025-08-08** at **11:54:46**

> > -DGGML_CUDA_FA_ALL_QUANTS=ON
> 
> I had linking errors for a couple weeks and had to turn this off. During LTO the build would fail. Gave up and just turned it off. I wasn't using mixed quanted cache anymore anyways. That's here, not in mainline.

Well, I just did a build with `-DGGML_CUDA_FA_ALL_QUANTS=ON` and it works. Can you post the errors along with your build commands?

---

👤 **Ph0rk0z** commented on **2025-08-08** at **12:54:14**

It builds now, yay. I figured it would work out eventually so I never opened an issue for this reason.

---

👤 **Orion-zhen** commented on **2025-08-08** at **13:09:20**

> [@Orion-zhen](https://github.com/Orion-zhen) Can you attach the `CMakeCache.txt` in the `ik_llama.cpp` build folder? Thanks.

Sorry, but I don't know where the `CMakeCache.txt` is. Could you give me more detailed instructions?

---

👤 **ikawrakow** commented on **2025-08-08** at **13:14:01**

> Sorry, but I don't know where the CMakeCache.txt is. 

You ran the command
```
cmake -B build -DGGML_CUDA=ON -DGGML_LTO=ON -DGGML_RPC=ON && cmake --build build --config Release
```
in a terminal to build `ik_llama.cpp`. This creates a sub-folder `build` in the folder where you ran the command. In the sub-folder `build` there is the file `CMakeCache.txt`.

---

👤 **Orion-zhen** commented on **2025-08-08** at **13:22:56**

[CMakeCache.txt](https://github.com/user-attachments/files/21684861/CMakeCache.txt)

---

👤 **ikawrakow** commented on **2025-08-08** at **13:26:03**

Thanks. If you open a terminal and type `which nvcc`, what is the output?

---

👤 **Orion-zhen** commented on **2025-08-08** at **13:26:38**

`/opt/cuda/bin/nvcc`

---

👤 **Panchovix** commented on **2025-08-08** at **17:03:58**

> It builds now, yay. I figured it would work out eventually so I never opened an issue for this reason.

Same, I had to do some shenanigans to make it work with GCC 15.x

12.9 doesn't work IIRC but there is 13.0 now that should work.

---

👤 **Orion-zhen** commented on **2025-08-10** at **09:47:11**

I figured it out. It's because CUDA environment variables are not set correctly somehow. Run `source /etc/profile` solves the issue.

---

👤 **sayap** commented on **2025-08-12** at **04:20:26**

This breakage should be caused by glibc-2.42. On Gentoo, this was fixed by https://github.com/gentoo/gentoo/commit/03c678f5d, requiring a recompile of nvidia-cuda-toolkit.