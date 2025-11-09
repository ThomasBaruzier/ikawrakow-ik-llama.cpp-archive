## 🗣️ [Discussion #890](https://github.com/ikawrakow/ik_llama.cpp/discussions/890) - Would enabling LTO for nvcc result in performance gains?

| **Author** | `hksdpc255` |
| :--- | :--- |
| **State** | ✅ **Open** |
| **Created** | 2025-11-03 |
| **Updated** | 2025-11-03 |

---

## 📄 Description

CMake has LTO support for nvcc: https://gitlab.kitware.com/cmake/cmake/-/commit/96bc59b1ca01be231347404d178445263687dd22

However, the build system of llama.cpp/ik_llama.cpp does not seem to utilize it.

I tried manually compiling ik_llama.cpp with LTO-enabled nvcc, but it failed:
```bash
cmake ik_llama.cpp -B ik_llama.cpp/build-gcc -G Ninja -DCMAKE_BUILD_TYPE=Release -DCMAKE_CUDA_FLAGS='-Xcompiler -fPIC' -DCMAKE_INTERPROCEDURAL_OPTIMIZATION=ON -DCMAKE_CUDA_SEPARABLE_COMPILATION=ON -DCMAKE_C_FLAGS="-fPIC -g0 -O3 -march=native -mtune=native -flto" -DCMAKE_CXX_FLAGS="-fPIC -g0 -O3 -march=native -mtune=native -flto" -DCMAKE_EXE_LINKER_FLAGS="-g0 -O3 -flto -fPIE" -DCMAKE_SHARED_LINKER_FLAGS="-fPIC -g0 -O3 -flto" -DCMAKE_POSITION_INDEPENDENT_CODE=ON -DBUILD_SHARED_LIBS=OFF -DLLAMA_CURL=OFF -DGGML_STATIC=ON -DGGML_NATIVE=ON -DGGML_LTO=ON -DGGML_BUILD_TESTS=OFF -DGGML_BUILD_EXAMPLES=OFF -DLLAMA_BUILD_TESTS=OFF -DLLAMA_BUILD_EXAMPLES=ON -DLLAMA_BUILD_SERVER=ON -DGGML_CUDA=ON -DGGML_CUDA_FA_ALL_QUANTS=ON -DGGML_CUDA_GRAPHS=ON -DGGML_CUDA_NO_VMM=ON -DCMAKE_CUDA_ARCHITECTURES="86;89" -DGGML_RPC=ON -DGGML_BLAS=OFF
```
with compilation errors
```
[1/32] Linking CXX executable bin/llama-quantize
FAILED: bin/llama-quantize 
: && /home/builder/miniforge3/envs/cuda13/bin/x86_64-conda-linux-gnu-c++ -fPIC -fopenmp -static-libgcc -static-libstdc++ -static-libgfortran -g0 -O3 -march=native -mtune=native -feliminate-unused-debug-types -pipe -Wall -fasynchronous-unwind-tables -Wl,-z,now -Wl,-z,relro -fno-semantic-interposition -fno-fat-lto-objects -fno-trapping-math -Wl,-sort-common -Wl,--enable-new-dtags -ffunction-sections -fvisibility-inlines-hidden -Wl,--enable-new-dtags -flto -O3 -DNDEBUG -flto=auto -fno-fat-lto-objects -g0 -O3 -flto -fno-fat-lto-objects -fPIE -ldl -Wl,-z,now -Wl,-z,relro -Wl,-sort-common -Wl,--enable-new-dtags    -Wl,--dependency-file=examples/quantize/CMakeFiles/llama-quantize.dir/link.d examples/quantize/CMakeFiles/llama-quantize.dir/quantize.cpp.o -o bin/llama-quantize  src/libllama.a  common/libcommon.a  src/libllama.a  ggml/src/libggml.a  /home/builder/miniforge3/envs/cuda13/lib/libcudart_static.a  /home/builder/miniforge3/envs/cuda13/targets/x86_64-linux/lib/libcublas_static.a  /home/builder/miniforge3/envs/cuda13/targets/x86_64-linux/lib/libcublasLt_static.a  -ldl  /home/builder/miniforge3/envs/cuda13/x86_64-conda-linux-gnu/sysroot/usr/lib/librt.so  /home/builder/miniforge3/envs/cuda13/lib/libculibos.a  /home/builder/miniforge3/envs/cuda13/x86_64-conda-linux-gnu/sysroot/usr/lib/libm.so && :
/home/builder/miniforge3/envs/cuda13/bin/../lib/gcc/x86_64-conda-linux-gnu/15.2.0/../../../../x86_64-conda-linux-gnu/bin/ld: ggml/src/libggml.a(ggml-cuda.cu.o): in function `__sti____cudaRegisterAll()':
tmpxft_000635b6_00000000-6_ggml-cuda.compute_89.cudafe1.cpp:(.text.startup+0x1d): undefined reference to `__cudaRegisterLinkedBinary_03b19f6c_12_ggml_cuda_cu_44dd4407_407193'
/home/builder/miniforge3/envs/cuda13/bin/../lib/gcc/x86_64-conda-linux-gnu/15.2.0/../../../../x86_64-conda-linux-gnu/bin/ld: ggml/src/libggml.a(acc.cu.o): in function `__sti____cudaRegisterAll()':
tmpxft_00063364_00000000-6_acc.compute_89.cudafe1.cpp:(.text.startup+0x1d): undefined reference to `__cudaRegisterLinkedBinary_b3cce064_6_acc_cu_44dd4407_406731'
/home/builder/miniforge3/envs/cuda13/bin/../lib/gcc/x86_64-conda-linux-gnu/15.2.0/../../../../x86_64-conda-linux-gnu/bin/ld: ggml/src/libggml.a(add-id.cu.o): in function `__sti____cudaRegisterAll()':
tmpxft_00063365_00000000-6_add-id.compute_89.cudafe1.cpp:(.text.startup+0x1d): undefined reference to `__cudaRegisterLinkedBinary_cbbc7c75_9_add_id_cu_75da6bba'
/home/builder/miniforge3/envs/cuda13/bin/../lib/gcc/x86_64-conda-linux-gnu/15.2.0/../../../../x86_64-conda-linux-gnu/bin/ld: ggml/src/libggml.a(arange.cu.o): in function `__sti____cudaRegisterAll()':
tmpxft_00063366_00000000-6_arange.compute_89.cudafe1.cpp:(.text.startup+0x1d): undefined reference to `__cudaRegisterLinkedBinary_ad10cab8_9_arange_cu_44dd4407_406718'
/home/builder/miniforge3/envs/cuda13/bin/../lib/gcc/x86_64-conda-linux-gnu/15.2.0/../../../../x86_64-conda-linux-gnu/bin/ld: ggml/src/libggml.a(argmax.cu.o): in function `__sti____cudaRegisterAll()':
tmpxft_00063367_00000000-6_argmax.compute_89.cudafe1.cpp:(.text.startup+0x1d): undefined reference to `__cudaRegisterLinkedBinary_6567f7ce_9_argmax_cu_bf916d26'
/home/builder/miniforge3/envs/cuda13/bin/../lib/gcc/x86_64-conda-linux-gnu/15.2.0/../../../../x86_64-conda-linux-gnu/bin/ld: ggml/src/libggml.a(argsort.cu.o): in function `__sti____cudaRegisterAll()':
tmpxft_00063368_00000000-6_argsort.compute_89.cudafe1.cpp:(.text.startup+0x1d): undefined reference to `__cudaRegisterLinkedBinary_09e538be_10_argsort_cu_44dd4407_406714'
/home/builder/miniforge3/envs/cuda13/bin/../lib/gcc/x86_64-conda-linux-gnu/15.2.0/../../../../x86_64-conda-linux-gnu/bin/ld: ggml/src/libggml.a(binbcast.cu.o): in function `__sti____cudaRegisterAll()':
tmpxft_00063369_00000000-6_binbcast.compute_89.cudafe1.cpp:(.text.startup+0x1d): undefined reference to `__cudaRegisterLinkedBinary_b40b08ae_11_binbcast_cu_44dd4407_406716'
/home/builder/miniforge3/envs/cuda13/bin/../lib/gcc/x86_64-conda-linux-gnu/15.2.0/../../../../x86_64-conda-linux-gnu/bin/ld: ggml/src/libggml.a(clamp.cu.o): in function `__sti____cudaRegisterAll()':
tmpxft_0006336a_00000000-6_clamp.compute_89.cudafe1.cpp:(.text.startup+0x1d): undefined reference to `__cudaRegisterLinkedBinary_4a4e9d19_8_clamp_cu_44dd4407_406712'
/home/builder/miniforge3/envs/cuda13/bin/../lib/gcc/x86_64-conda-linux-gnu/15.2.0/../../../../x86_64-conda-linux-gnu/bin/ld: ggml/src/libggml.a(concat.cu.o): in function `__sti____cudaRegisterAll()':
tmpxft_0006336b_00000000-6_concat.compute_89.cudafe1.cpp:(.text.startup+0x1d): undefined reference to `__cudaRegisterLinkedBinary_496b60a9_9_concat_cu_44dd4407_406717'
/home/builder/miniforge3/envs/cuda13/bin/../lib/gcc/x86_64-conda-linux-gnu/15.2.0/../../../../x86_64-conda-linux-gnu/bin/ld: ggml/src/libggml.a(conv-transpose-1d.cu.o): in function `__sti____cudaRegisterAll()':
tmpxft_0006336c_00000000-6_conv-transpose-1d.compute_89.cudafe1.cpp:(.text.startup+0x1d): undefined reference to `__cudaRegisterLinkedBinary_84663abd_20_conv_transpose_1d_cu_44dd4407_406719'
/home/builder/miniforge3/envs/cuda13/bin/../lib/gcc/x86_64-conda-linux-gnu/15.2.0/../../../../x86_64-conda-linux-gnu/bin/ld: ggml/src/libggml.a(conv2d-dw.cu.o): in function `__sti____cudaRegisterAll()':
tmpxft_0006336d_00000000-6_conv2d-dw.compute_89.cudafe1.cpp:(.text.startup+0x1d): undefined reference to `__cudaRegisterLinkedBinary_0e84b31e_12_conv2d_dw_cu_ac5b4b21'
/home/builder/miniforge3/envs/cuda13/bin/../lib/gcc/x86_64-conda-linux-gnu/15.2.0/../../../../x86_64-conda-linux-gnu/bin/ld: ggml/src/libggml.a(conv2d.cu.o): in function `__sti____cudaRegisterAll()':
tmpxft_0006336e_00000000-6_conv2d.compute_89.cudafe1.cpp:(.text.startup+0x1d): undefined reference to `__cudaRegisterLinkedBinary_35da1105_9_conv2d_cu_44dd4407_406723'
/home/builder/miniforge3/envs/cuda13/bin/../lib/gcc/x86_64-conda-linux-gnu/15.2.0/../../../../x86_64-conda-linux-gnu/bin/ld: ggml/src/libggml.a(convert.cu.o): in function `__sti____cudaRegisterAll()':
tmpxft_00063370_00000000-6_convert.compute_89.cudafe1.cpp:(.text.startup+0x1d): undefined reference to `__cudaRegisterLinkedBinary_bc735a4f_10_convert_cu_44dd4407_406736'
/home/builder/miniforge3/envs/cuda13/bin/../lib/gcc/x86_64-conda-linux-gnu/15.2.0/../../../../x86_64-conda-linux-gnu/bin/ld: ggml/src/libggml.a(cpy.cu.o): in function `__sti____cudaRegisterAll()':
tmpxft_00063373_00000000-6_cpy.compute_89.cudafe1.cpp:(.text.startup+0x1d): undefined reference to `__cudaRegisterLinkedBinary_e6e01bc6_6_cpy_cu_843ac19f'

......................

tmpxft_0006456d_00000000-6_fattn-vec-f32-instance-hs64-f16-q5_0.compute_89.cudafe1.cpp:(.text.startup+0x1d): undefined reference to `__cudaRegisterLinkedBinary_ff939246_39_fattn_vec_f32_instance_hs64_f16_q5_0_cu_44dd4407_411064'
/home/builder/miniforge3/envs/cuda13/bin/../lib/gcc/x86_64-conda-linux-gnu/15.2.0/../../../../x86_64-conda-linux-gnu/bin/ld: ggml/src/libggml.a(fattn-vec-f32-instance-hs64-f16-q5_1.cu.o): in function `__sti____cudaRegisterAll()':
tmpxft_00064577_00000000-6_fattn-vec-f32-instance-hs64-f16-q5_1.compute_89.cudafe1.cpp:(.text.startup+0x1d): undefined reference to `__cudaRegisterLinkedBinary_472ff523_39_fattn_vec_f32_instance_hs64_f16_q5_1_cu_44dd4407_411065'
/home/builder/miniforge3/envs/cuda13/bin/../lib/gcc/x86_64-conda-linux-gnu/15.2.0/../../../../x86_64-conda-linux-gnu/bin/ld: ggml/src/libggml.a(fattn-vec-f32-instance-hs64-f16-q8_0.cu.o): in function `__sti____cudaRegisterAll()':
tmpxft_0006459c_00000000-6_fattn-vec-f32-instance-hs64-f16-q8_0.compute_89.cudafe1.cpp:(.text.startup+0x1d): undefined reference to `__cudaRegisterLinkedBinary_430d8198_39_fattn_vec_f32_instance_hs64_f16_q8_0_cu_44dd4407_411087'
/home/builder/miniforge3/envs/cuda13/bin/../lib/gcc/x86_64-conda-linux-gnu/15.2.0/../../../../x86_64-conda-linux-gnu/bin/ld: ggml/src/libggml.a(fattn.cu.o): in function `__sti____cudaRegisterAll()':
tmpxft_00063392_00000000-6_fattn.compute_89.cudafe1.cpp:(.text.startup+0x1d): undefined reference to `__cudaRegisterLinkedBinary_ed54b4ca_8_fattn_cu_15d78620'
/home/builder/miniforge3/envs/cuda13/bin/../lib/gcc/x86_64-conda-linux-gnu/15.2.0/../../../../x86_64-conda-linux-gnu/bin/ld: ggml/src/libggml.a(mmq.cu.o): in function `__sti____cudaRegisterAll()':
tmpxft_00063399_00000000-6_mmq.compute_89.cudafe1.cpp:(.text.startup+0x1d): undefined reference to `__cudaRegisterLinkedBinary_812e016a_6_mmq_cu_44dd4407_406745'
/home/builder/miniforge3/envs/cuda13/bin/../lib/gcc/x86_64-conda-linux-gnu/15.2.0/../../../../x86_64-conda-linux-gnu/bin/ld: ggml/src/libggml.a(mmvq.cu.o): in function `__sti____cudaRegisterAll()':
tmpxft_0006339e_00000000-6_mmvq.compute_89.cudafe1.cpp:(.text.startup+0x1d): undefined reference to `__cudaRegisterLinkedBinary_1a1fa4d9_7_mmvq_cu_44dd4407_406725'
/home/builder/miniforge3/envs/cuda13/bin/../lib/gcc/x86_64-conda-linux-gnu/15.2.0/../../../../x86_64-conda-linux-gnu/bin/ld: ggml/src/libggml.a(fattn-mma-f16.cu.o): in function `__sti____cudaRegisterAll()':
tmpxft_0006337f_00000000-6_fattn-mma-f16.compute_89.cudafe1.cpp:(.text.startup+0x1d): undefined reference to `__cudaRegisterLinkedBinary_f4fde439_16_fattn_mma_f16_cu_44dd4407_406737'
/home/builder/miniforge3/envs/cuda13/bin/../lib/gcc/x86_64-conda-linux-gnu/15.2.0/../../../../x86_64-conda-linux-gnu/bin/ld: ggml/src/libggml.a(iqk_mmvq.cu.o): in function `__sti____cudaRegisterAll()':
tmpxft_00063397_00000000-6_iqk_mmvq.compute_89.cudafe1.cpp:(.text.startup+0x1d): undefined reference to `__cudaRegisterLinkedBinary_2fcf8971_11_iqk_mmvq_cu_44dd4407_406728'
collect2: error: ld returned 1 exit status
ninja: build stopped: subcommand failed.
```

I can confirm that LTO for GCC/Clang works using `-DCMAKE_INTERPROCEDURAL_OPTIMIZATION=ON -DGGML_LTO=ON`. However, when trying to enable LTO for nvcc (which should be automatic in modern CMake when using GCC as the nvcc backend), the linker fails.

Therefore, I cannot confirm whether enabling LTO for nvcc will actually provide better performance.

---

## 💬 Discussion

👤 **ikawrakow** commented on **2025-11-03** at **08:48:15**

See [#891](https://github.com/ikawrakow/ik_llama.cpp/issues/891) how to enable it. In my case performance with LTO is lower.