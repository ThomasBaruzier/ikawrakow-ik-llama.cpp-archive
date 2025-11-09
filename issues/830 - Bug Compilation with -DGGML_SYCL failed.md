## 📌 [Issue #830](https://github.com/ikawrakow/ik_llama.cpp/issues/830) - Bug: Compilation with -DGGML_SYCL failed

| **Author** | `hardWorker254` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-10-13 |
| **Updated** | 2025-10-13 |

---

## 📄 Description

### What happened?

Is SYCL currently supported or not?

cmake . \
-DCMAKE_BUILD_TYPE=Release \
-DGGML_BLAS=ON \
-DGGML_NATIVE=ON \
-DCMAKE_C_COMPILER=icx \
-DCMAKE_CXX_COMPILER=icpx \
-DCMAKE_C_FLAGS="-O3 -funroll-loops -ffast-math -flto -Wall -fp-model=fast" \
-DCMAKE_CXX_FLAGS="-O3 -funroll-loops -ffast-math -flto -Wall -qopenmp -fp-model=fast" \
-DGGML_BLAS_VENDOR=MKL \
-DBLA_VENDOR=Intel10_64lp \
-DBLAS_LIBRARIES="/opt/intel/oneapi/mkl/2025.2/lib/libmkl_intel_lp64.so;/opt/intel/oneapi/mkl/2025.2/lib/libmkl_sequential.so;/opt/intel/oneapi/mkl/2025.2/lib/libmkl_core.so" \
-DBLAS_INCLUDE_DIRS="/opt/intel/oneapi/mkl/2025.2/include" \
-DGGML_SYCL=ON
-- OpenMP found
-- BLAS found, Libraries: /opt/intel/oneapi/mkl/2025.2/lib/libmkl_intel_lp64.so;/opt/intel/oneapi/mkl/2025.2/lib/libmkl_sequential.so;/opt/intel/oneapi/mkl/2025.2/lib/libmkl_core.so
-- BLAS found, Includes: /opt/intel/oneapi/mkl/2025.2/include
-- Using optimized iqk matrix multiplications
-- Enabling IQK Flash Attention kernels
-- Using llamafile
-- Using oneAPI Release SYCL compiler (icpx).
-- SYCL found
-- ccache found, compilation results will be cached. Disable with GGML_CCACHE=OFF.
-- CMAKE_SYSTEM_PROCESSOR: x86_64
-- x86 detected
-- ARCH_FLAGS = -march=native
-- Configuring done (0.2s)
-- Generating done (0.2s)
-- Build files have been written to: /home/arlinux/ik_llama.cpp


cmake --build . --config Release --target llama-server llama-quantize --verbose -j

### Name and Version

Intel oneapi 2025.2

### What operating system are you seeing the problem on?

_No response_

### Relevant log output

```shell
[ 52%] Building CXX object ggml/src/CMakeFiles/ggml.dir/ggml-sycl.cpp.o
cd /home/arlinux/ik_llama.cpp/ggml/src && ccache /opt/intel/oneapi/compiler/2025.2/bin/icpx -DGGML_BUILD -DGGML_IQK_FLASH_ATTENTION -DGGML_SCHED_MAX_COPIES=1 -DGGML_SHARED -DGGML_SYCL_WARP_SIZE=16 -DGGML_USE_BLAS -DGGML_USE_IQK_MULMAT -DGGML_USE_LLAMAFILE -DGGML_USE_OPENMP -DGGML_USE_SYCL -DNDEBUG -D_GNU_SOURCE -D_XOPEN_SOURCE=600 -Dggml_EXPORTS -I/home/arlinux/ik_llama.cpp/ggml/src/../include -I/home/arlinux/ik_llama.cpp/ggml/src/. -I/opt/intel/oneapi/mkl/2025.2/include -O3 -funroll-loops -ffast-math -flto -Wall -qopenmp -fp-model=fast -Wno-narrowing -fsycl -O3 -DNDEBUG -std=gnu++17 -fPIC -Wmissing-declarations -Wmissing-noreturn -Wall -Wextra -Wpedantic -Wcast-qual -Wno-unused-function -march=native -fiopenmp -MD -MT ggml/src/CMakeFiles/ggml.dir/ggml-sycl/rope.cpp.o -MF CMakeFiles/ggml.dir/ggml-sycl/rope.cpp.o.d -o CMakeFiles/ggml.dir/ggml-sycl/rope.cpp.o -c /home/arlinux/ik_llama.cpp/ggml/src/ggml-sycl/rope.cpp
cd /home/arlinux/ik_llama.cpp/ggml/src && ccache /opt/intel/oneapi/compiler/2025.2/bin/icpx -DGGML_BUILD -DGGML_IQK_FLASH_ATTENTION -DGGML_SCHED_MAX_COPIES=1 -DGGML_SHARED -DGGML_SYCL_WARP_SIZE=16 -DGGML_USE_BLAS -DGGML_USE_IQK_MULMAT -DGGML_USE_LLAMAFILE -DGGML_USE_OPENMP -DGGML_USE_SYCL -DNDEBUG -D_GNU_SOURCE -D_XOPEN_SOURCE=600 -Dggml_EXPORTS -I/home/arlinux/ik_llama.cpp/ggml/src/../include -I/home/arlinux/ik_llama.cpp/ggml/src/. -I/opt/intel/oneapi/mkl/2025.2/include -O3 -funroll-loops -ffast-math -flto -Wall -qopenmp -fp-model=fast -Wno-narrowing -fsycl -O3 -DNDEBUG -std=gnu++17 -fPIC -Wmissing-declarations -Wmissing-noreturn -Wall -Wextra -Wpedantic -Wcast-qual -Wno-unused-function -march=native -fiopenmp -MD -MT ggml/src/CMakeFiles/ggml.dir/ggml-sycl.cpp.o -MF CMakeFiles/ggml.dir/ggml-sycl.cpp.o.d -o CMakeFiles/ggml.dir/ggml-sycl.cpp.o -c /home/arlinux/ik_llama.cpp/ggml/src/ggml-sycl.cpp
[ 52%] Building CXX object ggml/src/CMakeFiles/ggml.dir/ggml-sycl/mmvq.cpp.o
cd /home/arlinux/ik_llama.cpp/ggml/src && ccache /opt/intel/oneapi/compiler/2025.2/bin/icpx -DGGML_BUILD -DGGML_IQK_FLASH_ATTENTION -DGGML_SCHED_MAX_COPIES=1 -DGGML_SHARED -DGGML_SYCL_WARP_SIZE=16 -DGGML_USE_BLAS -DGGML_USE_IQK_MULMAT -DGGML_USE_LLAMAFILE -DGGML_USE_OPENMP -DGGML_USE_SYCL -DNDEBUG -D_GNU_SOURCE -D_XOPEN_SOURCE=600 -Dggml_EXPORTS -I/home/arlinux/ik_llama.cpp/ggml/src/../include -I/home/arlinux/ik_llama.cpp/ggml/src/. -I/opt/intel/oneapi/mkl/2025.2/include -O3 -funroll-loops -ffast-math -flto -Wall -qopenmp -fp-model=fast -Wno-narrowing -fsycl -O3 -DNDEBUG -std=gnu++17 -fPIC -Wmissing-declarations -Wmissing-noreturn -Wall -Wextra -Wpedantic -Wcast-qual -Wno-unused-function -march=native -fiopenmp -MD -MT ggml/src/CMakeFiles/ggml.dir/ggml-sycl/mmvq.cpp.o -MF CMakeFiles/ggml.dir/ggml-sycl/mmvq.cpp.o.d -o CMakeFiles/ggml.dir/ggml-sycl/mmvq.cpp.o -c /home/arlinux/ik_llama.cpp/ggml/src/ggml-sycl/mmvq.cpp
cd /home/arlinux/ik_llama.cpp/ggml/src && ccache /opt/intel/oneapi/compiler/2025.2/bin/icpx -DGGML_BUILD -DGGML_IQK_FLASH_ATTENTION -DGGML_SCHED_MAX_COPIES=1 -DGGML_SHARED -DGGML_SYCL_WARP_SIZE=16 -DGGML_USE_BLAS -DGGML_USE_IQK_MULMAT -DGGML_USE_LLAMAFILE -DGGML_USE_OPENMP -DGGML_USE_SYCL -DNDEBUG -D_GNU_SOURCE -D_XOPEN_SOURCE=600 -Dggml_EXPORTS -I/home/arlinux/ik_llama.cpp/ggml/src/../include -I/home/arlinux/ik_llama.cpp/ggml/src/. -I/opt/intel/oneapi/mkl/2025.2/include -O3 -funroll-loops -ffast-math -flto -Wall -qopenmp -fp-model=fast -Wno-narrowing -fsycl -O3 -DNDEBUG -std=gnu++17 -fPIC -Wmissing-declarations -Wmissing-noreturn -Wall -Wextra -Wpedantic -Wcast-qual -Wno-unused-function -march=native -fiopenmp -MD -MT ggml/src/CMakeFiles/ggml.dir/ggml-sycl/tsembd.cpp.o -MF CMakeFiles/ggml.dir/ggml-sycl/tsembd.cpp.o.d -o CMakeFiles/ggml.dir/ggml-sycl/tsembd.cpp.o -c /home/arlinux/ik_llama.cpp/ggml/src/ggml-sycl/tsembd.cpp

A lot of warnings...

/home/arlinux/ik_llama.cpp/ggml/src/ggml-sycl.cpp:90:5: error: use of undeclared identifier 'ggml_backend_sycl_buffer_context'; did you mean 'ggml_backend_sycl_buffer_type'?
   90 |     ggml_backend_sycl_buffer_context * ctx = (ggml_backend_sycl_buffer_context *) buffer->context;
      |     ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
      |     ggml_backend_sycl_buffer_type
/home/arlinux/ik_llama.cpp/ggml/src/../include/ggml-sycl.h:23:37: note: 'ggml_backend_sycl_buffer_type' declared here
   23 | GGML_API ggml_backend_buffer_type_t ggml_backend_sycl_buffer_type(int device);
      |                                     ^

/home/arlinux/ik_llama.cpp/ggml/src/ggml-sycl.cpp:90:47: error: use of undeclared identifier 'ggml_backend_sycl_buffer_context'; did you mean 'ggml_backend_sycl_buffer_type'?
   90 |     ggml_backend_sycl_buffer_context * ctx = (ggml_backend_sycl_buffer_context *) buffer->context;
      |                                               ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
      |                                               ggml_backend_sycl_buffer_type
/home/arlinux/ik_llama.cpp/ggml/src/../include/ggml-sycl.h:23:37: note: 'ggml_backend_sycl_buffer_type' declared here
   23 | GGML_API ggml_backend_buffer_type_t ggml_backend_sycl_buffer_type(int device);
      |                                     ^
/home/arlinux/ik_llama.cpp/ggml/src/ggml-sycl.cpp:90:81: error: expected expression
   90 |     ggml_backend_sycl_buffer_context * ctx = (ggml_backend_sycl_buffer_context *) buffer->context;
      |                                                                                 ^
/home/arlinux/ik_llama.cpp/ggml/src/ggml-sycl.cpp:90:40: error: use of undeclared identifier 'ctx'
   90 |     ggml_backend_sycl_buffer_context * ctx = (ggml_backend_sycl_buffer_context *) buffer->context;
      |                                        ^
/home/arlinux/ik_llama.cpp/ggml/src/ggml-sycl.cpp:91:37: error: use of undeclared identifier 'ctx'
   91 |     SYCL_CHECK(ggml_sycl_set_device(ctx->device));
      |                                     ^
/home/arlinux/ik_llama.cpp/ggml/src/ggml-sycl.cpp:92:58: error: use of undeclared identifier 'ctx'
   92 |     auto stream = &(dpct::dev_mgr::instance().get_device(ctx->device).default_queue());
      |                                                          ^
/home/arlinux/ik_llama.cpp/ggml/src/ggml-sycl.cpp:87:74: warning: unused parameter 'buffer' [-Wunused-parameter]
   87 | static void ggml_backend_sycl_buffer_memset_tensor(ggml_backend_buffer_t buffer, ggml_tensor * tensor, uint8_t value,
      |                                                                          ^
/home/arlinux/ik_llama.cpp/ggml/src/ggml-sycl.cpp:110:5: error: use of undeclared identifier 'ggml_backend_sycl_buffer_context'; did you mean 'ggml_backend_sycl_buffer_reset'?
  110 |     ggml_backend_sycl_buffer_context * ctx = (ggml_backend_sycl_buffer_context *) buffer->context;
      |     ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
      |     ggml_backend_sycl_buffer_reset
/home/arlinux/ik_llama.cpp/ggml/src/ggml-sycl.cpp:104:13: note: 'ggml_backend_sycl_buffer_reset' declared here
  104 | static void ggml_backend_sycl_buffer_reset(ggml_backend_buffer_t buffer) {
      |             ^
/home/arlinux/ik_llama.cpp/ggml/src/ggml-sycl.cpp:110:47: error: use of undeclared identifier 'ggml_backend_sycl_buffer_context'; did you mean 'ggml_backend_sycl_buffer_reset'?
  110 |     ggml_backend_sycl_buffer_context * ctx = (ggml_backend_sycl_buffer_context *) buffer->context;
      |                                               ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
      |                                               ggml_backend_sycl_buffer_reset
/home/arlinux/ik_llama.cpp/ggml/src/ggml-sycl.cpp:104:13: note: 'ggml_backend_sycl_buffer_reset' declared here
  104 | static void ggml_backend_sycl_buffer_reset(ggml_backend_buffer_t buffer) {
      |             ^
/home/arlinux/ik_llama.cpp/ggml/src/ggml-sycl.cpp:110:81: error: expected expression
  110 |     ggml_backend_sycl_buffer_context * ctx = (ggml_backend_sycl_buffer_context *) buffer->context;
      |                                                                                 ^
/home/arlinux/ik_llama.cpp/ggml/src/ggml-sycl.cpp:110:40: error: use of undeclared identifier 'ctx'
  110 |     ggml_backend_sycl_buffer_context * ctx = (ggml_backend_sycl_buffer_context *) buffer->context;
      |                                        ^
/home/arlinux/ik_llama.cpp/ggml/src/ggml-sycl.cpp:112:9: error: use of undeclared identifier 'ctx'
  112 |     if (ctx != nullptr) {
      |         ^
/home/arlinux/ik_llama.cpp/ggml/src/ggml-sycl.cpp:113:46: error: use of undeclared identifier 'ctx'
  113 |         for (ggml_tensor_extra_gpu * extra : ctx->tensor_extras) {
      |                                              ^
/home/arlinux/ik_llama.cpp/ggml/src/ggml-sycl.cpp:114:13: error: use of undeclared identifier 'release_extra_gpu'
  114 |             release_extra_gpu(extra);
      |             ^
/home/arlinux/ik_llama.cpp/ggml/src/ggml-sycl.cpp:116:9: error: use of undeclared identifier 'ctx'
  116 |         ctx->tensor_extras.clear();  // reset the tensor_extras vector
      |         ^
10 warnings generated.
/home/arlinux/ik_llama.cpp/ggml/src/ggml-sycl.cpp:121:30: error: use of undeclared identifier 'ggml_backend_sycl_buffer_free_buffer'; did you mean 'ggml_backend_sycl_buffer_reset'?
  121 |     /* .free_buffer     = */ ggml_backend_sycl_buffer_free_buffer,
      |                              ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
      |                              ggml_backend_sycl_buffer_reset
/home/arlinux/ik_llama.cpp/ggml/src/ggml-sycl.cpp:104:13: note: 'ggml_backend_sycl_buffer_reset' declared here
  104 | static void ggml_backend_sycl_buffer_reset(ggml_backend_buffer_t buffer) {
      |             ^
/home/arlinux/ik_llama.cpp/ggml/src/ggml-sycl.cpp:122:30: error: use of undeclared identifier 'ggml_backend_sycl_buffer_get_base'; did you mean 'ggml_backend_buffer_get_base'?
  122 |     /* .get_base        = */ ggml_backend_sycl_buffer_get_base,
      |                              ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
      |                              ggml_backend_buffer_get_base
/home/arlinux/ik_llama.cpp/ggml/src/../include/ggml-backend.h:37:55: note: 'ggml_backend_buffer_get_base' declared here
   37 |     GGML_API           void *                         ggml_backend_buffer_get_base      (ggml_backend_buffer_t buffer);
      |                                                       ^

/home/arlinux/ik_llama.cpp/ggml/src/ggml-sycl.cpp:123:30: error: use of undeclared identifier 'ggml_backend_sycl_buffer_init_tensor'
  123 |     /* .init_tensor     = */ ggml_backend_sycl_buffer_init_tensor,
      |                              ^

fatal error: too many errors emitted, stopping now [-ferror-limit=]
7 warnings generated.
8 warnings and 20 errors generated.
make[3]: *** [ggml/src/CMakeFiles/ggml.dir/build.make:289: ggml/src/CMakeFiles/ggml.dir/ggml-sycl.cpp.o] Ошибка 1
make[3]: *** Ожидание завершения заданий…
29 warnings generated.
17 warnings generated.
40 warnings generated.
9 warnings generated.
54 warnings generated.
8 warnings generated.


I removed all the warnings and kept only the errors
```

---

## 💬 Conversation

👤 **ikawrakow** commented on **2025-10-13** at **15:01:25**

No, SYCL is not supported, and there are no plans to support it. I just haven't taken the time to completely remove it.