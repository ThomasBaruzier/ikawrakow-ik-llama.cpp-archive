## 📌 [Issue #743](https://github.com/ikawrakow/ik_llama.cpp/issues/743) - compilation error

| **Author** | `bchtrue` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-08-30 |
| **Updated** | 2025-09-26 |

---

## 📄 Description

```
user@m3pc:~/ik_llama.cpp$ cmake --build ./build --config Release -j $(nproc)
[  0%] Built target build_info
[  0%] Built target sha256
[  1%] Built target xxhash
[  1%] Built target sha1
[  1%] Building CXX object ggml/src/CMakeFiles/ggml.dir/iqk/iqk_quantize.cpp.o
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp: In function ‘void {anonymous}::quantize_row_q8_1_x4_T(const float*, Block*, int64_t)’:
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp:977:20: error: there are no arguments to ‘hsum_i32_8’ that depend on a template parameter, so a declaration of ‘hsum_i32_8’ must be available [-fpermissive]
  977 |         int isum = hsum_i32_8(_mm256_add_epi32(_mm256_add_epi32(i0, i1), _mm256_add_epi32(i2, i3)));
      |                    ^~~~~~~~~~
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp:977:20: note: (if you use ‘-fpermissive’, G++ will accept your code, but allowing the use of an undeclared name is deprecated)
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp: In function ‘void repack_q8_KV(int, int, const char*, char*, bool)’:
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp:6745:21: error: ‘y’ was not declared in this scope
 6745 |                     y[ib].qs[32*l+4*k+i+  0] = x8[k][ib].qs[i+4*l+ 0];
      |                     ^
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp:6745:58: error: request for member ‘qs’ in ‘*(x8[k] + ((sizetype)ib))’, which is of non-class type ‘const int8_t’ {aka ‘const signed char’}
 6745 |                     y[ib].qs[32*l+4*k+i+  0] = x8[k][ib].qs[i+4*l+ 0];
      |                                                          ^~
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp:6746:58: error: request for member ‘qs’ in ‘*(x8[k] + ((sizetype)ib))’, which is of non-class type ‘const int8_t’ {aka ‘const signed char’}
 6746 |                     y[ib].qs[32*l+4*k+i+128] = x8[k][ib].qs[i+4*l+16];
      |                                                          ^~
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp:6687:14: warning: unused variable ‘qy’ [-Wunused-variable]
 6687 |         auto qy = (int8_t *)(dy + 8);
      |              ^~
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp: In instantiation of ‘void {anonymous}::quantize_row_q8_1_x4_T(const float*, Block*, int64_t) [with Block = block_q8_1; Block_x4 = block_q8_1_x4; int64_t = long int]’:
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp:1017:54:   required from here
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp:977:30: error: ‘hsum_i32_8’ was not declared in this scope
  977 |         int isum = hsum_i32_8(_mm256_add_epi32(_mm256_add_epi32(i0, i1), _mm256_add_epi32(i2, i3)));
      |                    ~~~~~~~~~~^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp: In instantiation of ‘void {anonymous}::quantize_row_q8_1_x4_T(const float*, Block*, int64_t) [with Block = block_q8_2; Block_x4 = block_q8_2_x4; int64_t = long int]’:
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp:1021:54:   required from here
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp:977:30: error: ‘hsum_i32_8’ was not declared in this scope
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp: In instantiation of ‘void {anonymous}::QuantizerIQKT<block_size, group_size, num_bits, is_abs, is_int>::find_best_match(float, const float*, const float*, int*) const [with int block_size = 32; int group_size = 8; int num_bits = 13; bool is_abs = false; bool is_int = true]’:
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp:8755:38:   required from here
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp:8273:9: warning: unused variable ‘ncluster’ [-Wunused-variable]
 8273 |     int ncluster = m_clusters.size()/kGroupSize;
      |         ^~~~~~~~
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp:8274:11: warning: unused variable ‘id’ [-Wunused-variable]
 8274 |     float id = 1/d;
      |           ^~
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp:8268:110: warning: unused parameter ‘xb’ [-Wunused-parameter]
 8268 | void QuantizerIQKT<block_size, group_size, num_bits, is_abs, is_int>::find_best_match(float d, const float * xb, const float * weight, int * best_idx) const {
      |                                                                                                ~~~~~~~~~~~~~~^~
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp:8268:128: warning: unused parameter ‘weight’ [-Wunused-parameter]
 8268 | void QuantizerIQKT<block_size, group_size, num_bits, is_abs, is_int>::find_best_match(float d, const float * xb, const float * weight, int * best_idx) const {
      |                                                                                                                  ~~~~~~~~~~~~~~^~~~~~
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp: In instantiation of ‘void {anonymous}::QuantizerIQKT<block_size, group_size, num_bits, is_abs, is_int>::find_best_match(float, const float*, const float*, int*) const [with int block_size = 32; int group_size = 8; int num_bits = 16; bool is_abs = false; bool is_int = true]’:
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp:9020:38:   required from here
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp:8273:9: warning: unused variable ‘ncluster’ [-Wunused-variable]
 8273 |     int ncluster = m_clusters.size()/kGroupSize;
      |         ^~~~~~~~
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp:8274:11: warning: unused variable ‘id’ [-Wunused-variable]
 8274 |     float id = 1/d;
      |           ^~
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp:8268:110: warning: unused parameter ‘xb’ [-Wunused-parameter]
 8268 | void QuantizerIQKT<block_size, group_size, num_bits, is_abs, is_int>::find_best_match(float d, const float * xb, const float * weight, int * best_idx) const {
      |                                                                                                ~~~~~~~~~~~~~~^~
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp:8268:128: warning: unused parameter ‘weight’ [-Wunused-parameter]
 8268 | void QuantizerIQKT<block_size, group_size, num_bits, is_abs, is_int>::find_best_match(float d, const float * xb, const float * weight, int * best_idx) const {
      |                                                                                                                  ~~~~~~~~~~~~~~^~~~~~
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp: In instantiation of ‘void {anonymous}::QuantizerIQKT<block_size, group_size, num_bits, is_abs, is_int>::find_best_match(float, const float*, const float*, int*) const [with int block_size = 32; int group_size = 8; int num_bits = 16; bool is_abs = true; bool is_int = true]’:
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp:9321:42:   required from here
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp:8273:9: warning: unused variable ‘ncluster’ [-Wunused-variable]
 8273 |     int ncluster = m_clusters.size()/kGroupSize;
      |         ^~~~~~~~
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp:8274:11: warning: unused variable ‘id’ [-Wunused-variable]
 8274 |     float id = 1/d;
      |           ^~
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp:8268:110: warning: unused parameter ‘xb’ [-Wunused-parameter]
 8268 | void QuantizerIQKT<block_size, group_size, num_bits, is_abs, is_int>::find_best_match(float d, const float * xb, const float * weight, int * best_idx) const {
      |                                                                                                ~~~~~~~~~~~~~~^~
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp:8268:128: warning: unused parameter ‘weight’ [-Wunused-parameter]
 8268 | void QuantizerIQKT<block_size, group_size, num_bits, is_abs, is_int>::find_best_match(float d, const float * xb, const float * weight, int * best_idx) const {
      |                                                                                                                  ~~~~~~~~~~~~~~^~~~~~
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp: In instantiation of ‘void {anonymous}::QuantizerIQKT<block_size, group_size, num_bits, is_abs, is_int>::find_best_match(float, const float*, const float*, int*) const [with int block_size = 32; int group_size = 4; int num_bits = 15; bool is_abs = false; bool is_int = true]’:
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp:9605:43:   required from here
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp:8273:9: warning: unused variable ‘ncluster’ [-Wunused-variable]
 8273 |     int ncluster = m_clusters.size()/kGroupSize;
      |         ^~~~~~~~
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp:8274:11: warning: unused variable ‘id’ [-Wunused-variable]
 8274 |     float id = 1/d;
      |           ^~
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp:8268:110: warning: unused parameter ‘xb’ [-Wunused-parameter]
 8268 | void QuantizerIQKT<block_size, group_size, num_bits, is_abs, is_int>::find_best_match(float d, const float * xb, const float * weight, int * best_idx) const {
      |                                                                                                ~~~~~~~~~~~~~~^~
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp:8268:128: warning: unused parameter ‘weight’ [-Wunused-parameter]
 8268 | void QuantizerIQKT<block_size, group_size, num_bits, is_abs, is_int>::find_best_match(float d, const float * xb, const float * weight, int * best_idx) const {
      |                                                                                                                  ~~~~~~~~~~~~~~^~~~~~
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp: In function ‘void quantize_row_q8_0_x4(const float*, void*, int64_t)’:
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp:786:40: warning: AVX vector return without AVX enabled changes the ABI [-Wpsabi]
  786 |         __m256 v0 = _mm256_loadu_ps( x );
      |                                        ^
gmake[2]: *** [ggml/src/CMakeFiles/ggml.dir/build.make:2529: ggml/src/CMakeFiles/ggml.dir/iqk/iqk_quantize.cpp.o] Error 1
gmake[1]: *** [CMakeFiles/Makefile2:2025: ggml/src/CMakeFiles/ggml.dir/all] Error 2
gmake: *** [Makefile:146: all] Error 2

```

 gcc --version
gcc (Ubuntu 13.3.0-6ubuntu2~24.04) 13.3.0

Hello, strange... not first time I tried to compile (now second try maybe 3 month later, and again can not compile :-(

---

## 💬 Conversation

👤 **magikRUKKOLA** commented on **2025-08-30** at **19:12:02**

Just tried the latest master with gcc-14:

```
gcc --version
gcc (Debian 14.3.0-5) 14.3.0
Copyright (C) 2024 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```

(Adjust the DCMAKE_CUDA_ARCHITECTURES based on your GPU)
```
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

No issues.

```
-- OpenMP found
-- Using optimized iqk matrix multiplications
-- Enabling IQK Flash Attention kernels
-- Using llamafile
-- CUDA found
-- Using CUDA architectures: 86
-- CUDA host compiler is GNU 14.3.0

-- ccache found, compilation results will be cached. Disable with GGML_CCACHE=OFF.
-- CMAKE_SYSTEM_PROCESSOR: x86_64
-- x86 detected
-- ARCH_FLAGS = -march=native
-- Configuring done (0.6s)
-- Generating done (0.3s)
-- Build files have been written to: /opt/ik_llama.cpp/ik_llama.cpp/build
[  0%] Generating build details from Git
[  1%] Built target sha256
[  1%] Built target sha1
-- Found Git: /usr/bin/git (found version "2.50.1")
[  2%] Built target xxhash
[  2%] Building CXX object common/CMakeFiles/build_info.dir/build-info.cpp.o
[  2%] Built target build_info
[  2%] Building CUDA object ggml/src/CMakeFiles/ggml.dir/ggml-cuda.cu.o
...
[ 54%] Building CXX object ggml/src/CMakeFiles/ggml.dir/iqk/iqk_mul_mat.cpp.o
[ 54%] Building CXX object ggml/src/CMakeFiles/ggml.dir/iqk/iqk_quantize.cpp.o
[ 54%] Building CUDA object ggml/src/CMakeFiles/ggml.dir/ggml-cuda/mmq_id.cu.o
```

But how come your build process is getting started with iqk_quantize.cpp.o rather than ggml-cuda.cu.o?

lscpu:
```
Model name:                              AMD Ryzen Threadripper PRO 3995WX 64-Cores
Flags:                                   fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 h
t syscall nx mmxext fxsr_opt pdpe1gb rdtscp lm constant_tsc rep_good nopl xtopology nonstop_tsc cpuid extd_apicid aperfmperf rapl pni pclmul
qdq monitor ssse3 fma cx16 sse4_1 sse4_2 x2apic movbe popcnt aes xsave avx f16c rdrand lahf_lm cmp_legacy extapic cr8_legacy abm sse4a misal
ignsse 3dnowprefetch osvw ibs skinit wdt tce topoext perfctr_core perfctr_nb bpext perfctr_llc mwaitx cpb cat_l3 cdp_l3 hw_pstate ssbd mba i
bpb stibp vmmcall fsgsbase bmi1 avx2 smep bmi2 cqm rdt_a rdseed adx smap clflushopt clwb sha_ni xsaveopt xsavec xgetbv1 xsaves cqm_llc cqm_o
ccup_llc cqm_mbm_total cqm_mbm_local clzero irperf xsaveerptr rdpru wbnoinvd arat npt lbrv svm_lock nrip_save tsc_scale vmcb_clean flushbyas
id decodeassists pausefilter pfthreshold avic v_vmsave_vmload vgif v_spec_ctrl umip rdpid overflow_recov succor smca
```

---

👤 **bchtrue** commented on **2025-08-31** at **21:59:42**

The same error after re-cinfiguration with vars from @magikRUKKOLA 


```
[ 64%] Building CXX object ggml/src/CMakeFiles/ggml.dir/iqk/iqk_gemm_1bit.cpp.o
[ 64%] Building CXX object ggml/src/CMakeFiles/ggml.dir/iqk/iqk_gemm_legacy_quants.cpp.o
[ 64%] Building CXX object ggml/src/CMakeFiles/ggml.dir/iqk/iqk_quantize.cpp.o
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp: In function ‘void {anonymous}::quantize_row_q8_1_x4_T(const float*, Block*, int64_t)’:
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp:977:20: error: there are no arguments to ‘hsum_i32_8’ that depend on a template parameter, so a declaration of ‘hsum_i32_8’ must be available [-fpermissive]
  977 |         int isum = hsum_i32_8(_mm256_add_epi32(_mm256_add_epi32(i0, i1), _mm256_add_epi32(i2, i3)));
      |                    ^~~~~~~~~~
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp:977:20: note: (if you use ‘-fpermissive’, G++ will accept your code, but allowing the use of an undeclared name is deprecated)
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp: In function ‘void repack_q8_KV(int, int, const char*, char*, bool)’:
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp:6745:21: error: ‘y’ was not declared in this scope
 6745 |                     y[ib].qs[32*l+4*k+i+  0] = x8[k][ib].qs[i+4*l+ 0];
      |                     ^
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp:6745:58: error: request for member ‘qs’ in ‘*(x8[k] + ((sizetype)ib))’, which is of non-class type ‘const int8_t’ {aka ‘const signed char’}
 6745 |                     y[ib].qs[32*l+4*k+i+  0] = x8[k][ib].qs[i+4*l+ 0];
      |                                                          ^~
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp:6746:58: error: request for member ‘qs’ in ‘*(x8[k] + ((sizetype)ib))’, which is of non-class type ‘const int8_t’ {aka ‘const signed char’}
 6746 |                     y[ib].qs[32*l+4*k+i+128] = x8[k][ib].qs[i+4*l+16];
      |                                                          ^~
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp:6687:14: warning: unused variable ‘qy’ [-Wunused-variable]
 6687 |         auto qy = (int8_t *)(dy + 8);
      |              ^~
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp: In instantiation of ‘void {anonymous}::quantize_row_q8_1_x4_T(const float*, Block*, int64_t) [with Block = block_q8_1; Block_x4 = block_q8_1_x4; int64_t = long int]’:
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp:1017:54:   required from here
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp:977:30: error: ‘hsum_i32_8’ was not declared in this scope
  977 |         int isum = hsum_i32_8(_mm256_add_epi32(_mm256_add_epi32(i0, i1), _mm256_add_epi32(i2, i3)));
      |                    ~~~~~~~~~~^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp: In instantiation of ‘void {anonymous}::quantize_row_q8_1_x4_T(const float*, Block*, int64_t) [with Block = block_q8_2; Block_x4 = block_q8_2_x4; int64_t = long int]’:
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp:1021:54:   required from here
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp:977:30: error: ‘hsum_i32_8’ was not declared in this scope
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp: In instantiation of ‘void {anonymous}::QuantizerIQKT<block_size, group_size, num_bits, is_abs, is_int>::find_best_match(float, const float*, const float*, int*) const [with int block_size = 32; int group_size = 8; int num_bits = 13; bool is_abs = false; bool is_int = true]’:
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp:8755:38:   required from here
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp:8273:9: warning: unused variable ‘ncluster’ [-Wunused-variable]
 8273 |     int ncluster = m_clusters.size()/kGroupSize;
      |         ^~~~~~~~
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp:8274:11: warning: unused variable ‘id’ [-Wunused-variable]
 8274 |     float id = 1/d;
      |           ^~
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp:8268:110: warning: unused parameter ‘xb’ [-Wunused-parameter]
 8268 | void QuantizerIQKT<block_size, group_size, num_bits, is_abs, is_int>::find_best_match(float d, const float * xb, const float * weight, int * best_idx) const {
      |                                                                                                ~~~~~~~~~~~~~~^~
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp:8268:128: warning: unused parameter ‘weight’ [-Wunused-parameter]
 8268 | void QuantizerIQKT<block_size, group_size, num_bits, is_abs, is_int>::find_best_match(float d, const float * xb, const float * weight, int * best_idx) const {
      |                                                                                                                  ~~~~~~~~~~~~~~^~~~~~
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp: In instantiation of ‘void {anonymous}::QuantizerIQKT<block_size, group_size, num_bits, is_abs, is_int>::find_best_match(float, const float*, const float*, int*) const [with int block_size = 32; int group_size = 8; int num_bits = 16; bool is_abs = false; bool is_int = true]’:
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp:9020:38:   required from here
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp:8273:9: warning: unused variable ‘ncluster’ [-Wunused-variable]
 8273 |     int ncluster = m_clusters.size()/kGroupSize;
      |         ^~~~~~~~
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp:8274:11: warning: unused variable ‘id’ [-Wunused-variable]
 8274 |     float id = 1/d;
      |           ^~
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp:8268:110: warning: unused parameter ‘xb’ [-Wunused-parameter]
 8268 | void QuantizerIQKT<block_size, group_size, num_bits, is_abs, is_int>::find_best_match(float d, const float * xb, const float * weight, int * best_idx) const {
      |                                                                                                ~~~~~~~~~~~~~~^~
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp:8268:128: warning: unused parameter ‘weight’ [-Wunused-parameter]
 8268 | void QuantizerIQKT<block_size, group_size, num_bits, is_abs, is_int>::find_best_match(float d, const float * xb, const float * weight, int * best_idx) const {
      |                                                                                                                  ~~~~~~~~~~~~~~^~~~~~
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp: In instantiation of ‘void {anonymous}::QuantizerIQKT<block_size, group_size, num_bits, is_abs, is_int>::find_best_match(float, const float*, const float*, int*) const [with int block_size = 32; int group_size = 8; int num_bits = 16; bool is_abs = true; bool is_int = true]’:
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp:9321:42:   required from here
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp:8273:9: warning: unused variable ‘ncluster’ [-Wunused-variable]
 8273 |     int ncluster = m_clusters.size()/kGroupSize;
      |         ^~~~~~~~
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp:8274:11: warning: unused variable ‘id’ [-Wunused-variable]
 8274 |     float id = 1/d;
      |           ^~
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp:8268:110: warning: unused parameter ‘xb’ [-Wunused-parameter]
 8268 | void QuantizerIQKT<block_size, group_size, num_bits, is_abs, is_int>::find_best_match(float d, const float * xb, const float * weight, int * best_idx) const {
      |                                                                                                ~~~~~~~~~~~~~~^~
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp:8268:128: warning: unused parameter ‘weight’ [-Wunused-parameter]
 8268 | void QuantizerIQKT<block_size, group_size, num_bits, is_abs, is_int>::find_best_match(float d, const float * xb, const float * weight, int * best_idx) const {
      |                                                                                                                  ~~~~~~~~~~~~~~^~~~~~
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp: In instantiation of ‘void {anonymous}::QuantizerIQKT<block_size, group_size, num_bits, is_abs, is_int>::find_best_match(float, const float*, const float*, int*) const [with int block_size = 32; int group_size = 4; int num_bits = 15; bool is_abs = false; bool is_int = true]’:
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp:9605:43:   required from here
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp:8273:9: warning: unused variable ‘ncluster’ [-Wunused-variable]
 8273 |     int ncluster = m_clusters.size()/kGroupSize;
      |         ^~~~~~~~
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp:8274:11: warning: unused variable ‘id’ [-Wunused-variable]
 8274 |     float id = 1/d;
      |           ^~
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp:8268:110: warning: unused parameter ‘xb’ [-Wunused-parameter]
 8268 | void QuantizerIQKT<block_size, group_size, num_bits, is_abs, is_int>::find_best_match(float d, const float * xb, const float * weight, int * best_idx) const {
      |                                                                                                ~~~~~~~~~~~~~~^~
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp:8268:128: warning: unused parameter ‘weight’ [-Wunused-parameter]
 8268 | void QuantizerIQKT<block_size, group_size, num_bits, is_abs, is_int>::find_best_match(float d, const float * xb, const float * weight, int * best_idx) const {
      |                                                                                                                  ~~~~~~~~~~~~~~^~~~~~
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp: In function ‘void quantize_row_q8_0_x4(const float*, void*, int64_t)’:
/home/user/ik_llama.cpp/ggml/src/iqk/iqk_quantize.cpp:786:40: warning: AVX vector return without AVX enabled changes the ABI [-Wpsabi]
  786 |         __m256 v0 = _mm256_loadu_ps( x );
      |                                        ^
gmake[2]: *** [ggml/src/CMakeFiles/ggml.dir/build.make:3789: ggml/src/CMakeFiles/ggml.dir/iqk/iqk_quantize.cpp.o] Error 1
gmake[2]: *** Waiting for unfinished jobs....
gmake[1]: *** [CMakeFiles/Makefile2:2025: ggml/src/CMakeFiles/ggml.dir/all] Error 2
gmake: *** [Makefile:146: all] Error 2

```

---

👤 **magikRUKKOLA** commented on **2025-09-01** at **02:50:16**

@bchtrue can you drop the output of lscpu?

---

👤 **bchtrue** commented on **2025-09-01** at **06:30:17**

@magikRUKKOLA 
btw, I can compile llama.cpp at the same machine without any issues

```
Architecture:             x86_64
  CPU op-mode(s):         32-bit, 64-bit
  Address sizes:          39 bits physical, 48 bits virtual
  Byte Order:             Little Endian
CPU(s):                   4
  On-line CPU(s) list:    0-3
Vendor ID:                GenuineIntel
  Model name:             Intel(R) Pentium(R) CPU G4560 @ 3.50GHz
    CPU family:           6
    Model:                158
    Thread(s) per core:   2
    Core(s) per socket:   2
    Socket(s):            1
    Stepping:             9
    CPU(s) scaling MHz:   40%
    CPU max MHz:          3500.0000
    CPU min MHz:          800.0000
    BogoMIPS:             6999.82
    Flags:                fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe sys
                          call nx pdpe1gb rdtscp lm constant_tsc art arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc cpuid aperfmperf pni p
                          clmulqdq dtes64 monitor ds_cpl vmx est tm2 ssse3 sdbg cx16 xtpr pdcm pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_ti
                          mer aes xsave rdrand lahf_lm abm 3dnowprefetch cpuid_fault pti ssbd ibrs ibpb stibp tpr_shadow flexpriority ept vpid ept_ad
                           fsgsbase tsc_adjust smep erms invpcid mpx rdseed smap clflushopt intel_pt xsaveopt xsavec xgetbv1 xsaves dtherm arat pln p
                          ts hwp hwp_notify hwp_act_window hwp_epp vnmi md_clear flush_l1d arch_capabilities
Virtualization features:  
  Virtualization:         VT-x
Caches (sum of all):      
  L1d:                    64 KiB (2 instances)
  L1i:                    64 KiB (2 instances)
  L2:                     512 KiB (2 instances)
  L3:                     3 MiB (1 instance)
NUMA:                     
  NUMA node(s):           1
  NUMA node0 CPU(s):      0-3
Vulnerabilities:          
  Gather data sampling:   Not affected
  Itlb multihit:          KVM: Mitigation: VMX disabled
  L1tf:                   Mitigation; PTE Inversion; VMX conditional cache flushes, SMT vulnerable
  Mds:                    Mitigation; Clear CPU buffers; SMT vulnerable
  Meltdown:               Mitigation; PTI
  Mmio stale data:        Mitigation; Clear CPU buffers; SMT vulnerable
  Reg file data sampling: Not affected
  Retbleed:               Mitigation; IBRS
  Spec rstack overflow:   Not affected
  Spec store bypass:      Mitigation; Speculative Store Bypass disabled via prctl
  Spectre v1:             Mitigation; usercopy/swapgs barriers and __user pointer sanitization
  Spectre v2:             Mitigation; IBRS; IBPB conditional; STIBP conditional; RSB filling; PBRSB-eIBRS Not affected; BHI Not affected
  Srbds:                  Mitigation; Microcode
  Tsx async abort:        Not affected
```

---

👤 **ikawrakow** commented on **2025-09-01** at **06:40:26**

@bchtrue Your CPU does not support `AVX2`, which is the minimum requirement for `ik_llama.cpp` and `x86_64` (and it does not even support `AVX`). You should try mainline `llama.cpp`.

---

👤 **bchtrue** commented on **2025-09-01** at **07:29:27**

@magikRUKKOLA thank you! Not sure that processor change will give any boost on this old motherboard... And the main issue is that not ik_lamma and not lamma.cpp support tensor parallelism for my milti-gpu setup :-( 

Need to assemble a completely pc to work with llm with good performance.

---

👤 **Serjkov** commented on **2025-09-01** at **07:34:59**

Hello everyone. I have the same problem. The same compilation errors. I have 2 intel e5 2697v4 processors, they support avx2. Nvidia tesla v100 16gb graphics card. I am compiling on a Linux debian12 VM on proxmox. I also can compile llama.cpp at the same machine without any issues
The compilation parameters are as follows:
cmake -B build \
  -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_CUDA_ARCHITECTURES="70" \
  -DGGML_CUDA=ON \
  -DGGML_CUDA_FA_ALL_QUANTS=1 \
  -DGGML_SCHED_MAX_COPIES=1 \
  -DGGML_CUDA_IQK_FORCE_BF16=1 \
  -DGGML_MAX_CONTEXTS=2048 \
  -DGGML_VULKAN=OFF \
  -DGGML_CUDA_F16=ON
cmake --build build --config Release -j $(nproc)


### **lscpu on vm**
Architecture:                x86_64
  CPU op-mode(s):            32-bit, 64-bit
  Address sizes:             40 bits physical, 48 bits virtual
  Byte Order:                Little Endian
CPU(s):                      72
  On-line CPU(s) list:       0-71
Vendor ID:                   GenuineIntel
  BIOS Vendor ID:            QEMU
  Model name:                Common KVM processor
    BIOS Model name:         pc-q35-9.2  CPU @ 2.0GHz
    BIOS CPU family:         1
    CPU family:              15
    Model:                   6
    Thread(s) per core:      1
    Core(s) per socket:      36
    Socket(s):               2
    Stepping:                1
    BogoMIPS:                4589.36
    Flags:                   fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pg
                             e mca cmov pat pse36 clflush mmx fxsr sse sse2 ht s
                             yscall nx lm constant_tsc nopl xtopology cpuid tsc_
                             known_freq pni cx16 x2apic hypervisor lahf_lm cpuid
                             _fault pti
Virtualization features:     
  Hypervisor vendor:         KVM
  Virtualization type:       full
Caches (sum of all):         
  L1d:                       2.3 MiB (72 instances)
  L1i:                       2.3 MiB (72 instances)
  L2:                        288 MiB (72 instances)
  L3:                        32 MiB (2 instances)
NUMA:                        
  NUMA node(s):              1
  NUMA node0 CPU(s):         0-71
Vulnerabilities:             
  Gather data sampling:      Not affected
  Indirect target selection: Mitigation; Aligned branch/return thunks
  Itlb multihit:             KVM: Mitigation: VMX unsupported
  L1tf:                      Mitigation; PTE Inversion
  Mds:                       Vulnerable: Clear CPU buffers attempted, no microco
                             de; SMT Host state unknown
  Meltdown:                  Mitigation; PTI
  Mmio stale data:           Unknown: No mitigations
  Reg file data sampling:    Not affected
  Retbleed:                  Not affected
  Spec rstack overflow:      Not affected
  Spec store bypass:         Vulnerable
  Spectre v1:                Mitigation; usercopy/swapgs barriers and __user poi
                             nter sanitization
  Spectre v2:                Mitigation; Retpolines; STIBP disabled; RSB filling
                             ; PBRSB-eIBRS Not affected; BHI Retpoline
  Srbds:                     Not affected
  Tsa:                       Not affected
  Tsx async abort:           Not affected

### **lscpu on proxmox host**
root@artint:~# lscpu
Architecture:             x86_64
  CPU op-mode(s):         32-bit, 64-bit
  Address sizes:          46 bits physical, 48 bits virtual
  Byte Order:             Little Endian
CPU(s):                   72
  On-line CPU(s) list:    0-71
Vendor ID:                GenuineIntel
  BIOS Vendor ID:         Intel
  Model name:             Intel(R) Xeon(R) CPU E5-2697 v4 @ 2.30GHz
    BIOS Model name:      Intel(R) Xeon(R) CPU E5-2697 v4 @ 2.30GHz  CPU @ 2.3GHz
    BIOS CPU family:      179
    CPU family:           6
    Model:                79
    Thread(s) per core:   2
    Core(s) per socket:   18
    Socket(s):            2
    Stepping:             1
    CPU(s) scaling MHz:   98%
    CPU max MHz:          3600.0000
    CPU min MHz:          1200.0000
    BogoMIPS:             4589.39
    Flags:                fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall
                           nx pdpe1gb rdtscp lm constant_tsc arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc cpuid aperfmperf pni pclmulqdq dte
                          s64 monitor ds_cpl vmx smx est tm2 ssse3 sdbg fma cx16 xtpr pdcm pcid dca sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer 
                          aes xsave avx f16c rdrand lahf_lm abm 3dnowprefetch cpuid_fault epb cat_l3 cdp_l3 pti intel_ppin ssbd ibrs ibpb stibp tpr_shado
                          w flexpriority ept vpid ept_ad fsgsbase tsc_adjust bmi1 hle avx2 smep bmi2 erms invpcid rtm cqm rdt_a rdseed adx smap intel_pt 
                          xsaveopt cqm_llc cqm_occup_llc cqm_mbm_total cqm_mbm_local dtherm ida arat pln pts vnmi md_clear flush_l1d
Virtualization features:  
  Virtualization:         VT-x
Caches (sum of all):      
  L1d:                    1.1 MiB (36 instances)
  L1i:                    1.1 MiB (36 instances)
  L2:                     9 MiB (36 instances)
  L3:                     90 MiB (2 instances)
NUMA:                     
  NUMA node(s):           2
  NUMA node0 CPU(s):      0-17,36-53
  NUMA node1 CPU(s):      18-35,54-71
Vulnerabilities:          
  Gather data sampling:   Not affected
  Itlb multihit:          KVM: Mitigation: Split huge pages
  L1tf:                   Mitigation; PTE Inversion; VMX conditional cache flushes, SMT vulnerable
  Mds:                    Mitigation; Clear CPU buffers; SMT vulnerable
  Meltdown:               Mitigation; PTI
  Mmio stale data:        Mitigation; Clear CPU buffers; SMT vulnerable
  Reg file data sampling: Not affected
  Retbleed:               Not affected
  Spec rstack overflow:   Not affected
  Spec store bypass:      Mitigation; Speculative Store Bypass disabled via prctl
  Spectre v1:             Mitigation; usercopy/swapgs barriers and __user pointer sanitization
  Spectre v2:             Mitigation; Retpolines; IBPB conditional; IBRS_FW; STIBP conditional; RSB filling; PBRSB-eIBRS Not affected; BHI Not affected
  Srbds:                  Not affected
  Tsx async abort:        Mitigation; Clear CPU buffers; SMT vulnerable

---

👤 **ikawrakow** commented on **2025-09-01** at **07:42:38**

@Serjkov The host CPU does support `AVX2`, but not the CPU in the VM in which you are trying to build as per VM CPU flags.

---

👤 **Serjkov** commented on **2025-09-01** at **07:49:23**

Yes, thank you very much for your prompt response. I started to figure it out and realized that the avx2 instruction was not inserted into the VM.

---

👤 **bchtrue** commented on **2025-09-01** at **07:55:11**

It will be nice to make it more flexible and use GPU more for such computations if user have it and not request this as dependency. Yes, I know that it's too hard.... and a lot of users use only CPU or CPU offloading and it is also too hard to implement all computations at GPU

Very weak multi-GPU utilization at all alternatives :-(

---

👤 **ikawrakow** commented on **2025-09-01** at **08:08:51**

When it comes to the computational part I'm the sole developer in this project, so supporting every conceivable CPU/GPU setup is just not possible. Doing this is one of the strengths of the mainline `llama.cpp` project.

If tensor parallelism is important to you, there are multiple inference engines that provide this capability (vllm, sglang, exllama, to just name a few).

---

👤 **magikRUKKOLA** commented on **2025-09-01** at **10:21:42**

> And the main issue is that not ik_lamma and not lamma.cpp support tensor parallelism for my milti-gpu setup :-(

/offtopic
But what kind of usable LLM can be reasonably ran via the multi-gpu setup?  DeepSeek is 671B parameter model.  With 3.5bpw quant you will need about 280GB of VRAM to load just the weights.  And another 72GB VRAM for the KV cache.  Its 14 RTX 3090 GPUs.  And its not even be possible to use more than one batch haha.
Its much more reasonable to use RAM for weights and GPUs for the experts and KV-cache (just like its already done in ik_llama.cpp).

Or, you might be using some smaller models?  If so, I was wondering what kind -- in my experience, all of them are garbage.

---

👤 **bchtrue** commented on **2025-09-01** at **12:10:31**

Currently, I'm mostly using Qwen3Coder 30B-A3B with llama.cpp 200k context (and have /2 of the ram actually free). I thought to test how much performance boost I will get with ik_lamma.

I do not look at such large models yet. I have only 5 cards (1080 Ti and 1070). VLLM dropped support for such 'old' hardware. They provided a repository of VLLM that is still possible to use with these cards, but I ended up being unable to launch any model because, after installation, it began to request some special compression of the model to run etc. The same goes for Exllama. I successfully installed it, but it was not possible to install TidyAPI there. Just abandoned the idea to test, due to the difficulties to use and lack of models.

Upgrade to i7 7700k  (supported by me motherboard) will cost only 100$, but not sure that it worth it to do. After fast research I see that last gen processors start with 200$.
Currently, I'm mostly investigation and play with all of this, not sure that I need it as investment to other hardware.

p.s. Qwen3Coder work pretty interesting as agent with tool calls. 
p.p.s. Any insights about possible performance boost with a better processor if I don't currently plan to offload anything to the CPU?

---

👤 **Ph0rk0z** commented on **2025-09-01** at **12:33:34**

> But what kind of usable LLM can be reasonably ran via the multi-gpu setup? DeepSeek is 671B parameter model. With 3.5bpw quant you will need about 280GB of VRAM to load just the weights. And another 72GB VRAM for the KV cache. Its 14 RTX 3090 GPUs. And its not even be possible to use more than one batch haha.
> Its much more reasonable to use RAM for weights and GPUs for the experts and KV-cache (just like its already done in ik_llama.cpp).


Many... You mean full vram? Mistral-large, pixtral-large, small-glm, qwen-235b, wizard 8x22b, deepseek-v2.5. For hybrid inference having multiple GPUs helps to make up a bit for a lacking CPU. If you just put expert weights on CPU (literally 90% of the model), you're going to have a bad time sans $$$ DDR5 server. Even on dual proc 230GB/s xeon pure CPU not great. Ancient P40 has faster speeds than that.

Both tensor parallel and numa support would benefit things greatly. My GPU processing is gridlocked to 10-15%. Pure gpu sm-row can give a nice uplift with mainline. On fastllm, pure numa on the same system produces 1/2 of t/s as hybrid inference and uses all b/w rather than ~60gb/s only.


> VLLM dropped support for such 'old' hardware. They provided a repository of VLLM that is still possible to use with these cards, but I ended up being unable to launch any model because, after installation, it began to request some special compression of the model to run etc. The same goes for Exllama. I successfully installed it, but it was not possible to install TidyAPI there. Just abandoned the idea to test, due to the difficulties to use and lack of models.

exllama doesn't run on pascal. VLLM pascal does work though. try newer AWQ models. Come to think of it, ik support of pascal isn't great either. I think you need to turn off FA or something. I have 3xP40 and 1xP100 sitting out due to idle power and simply living with 96g of vram.

---

👤 **magikRUKKOLA** commented on **2025-09-01** at **20:45:08**

@bchtrue 
> p.p.s. Any insights about possible performance boost with a better processor if I don't currently plan to offload anything to the CPU?

Nope, absolutely no ideas.
I am sorry for your hardware.  I actually have RTX Gamerock 1080Ti laying around that I pulled out from Lenovo Thinkstation P620 lol.  I can send it to you free of charge.  Are you in EU?

[EDIT]:  The offer of the free GPU for this particular gentleman expires in the next 12 hours!

[EDIT2]:  Six more hours for the offer to be expired!

---

👤 **magikRUKKOLA** commented on **2025-09-01** at **21:22:24**

@Ph0rk0z 
> Mistral-large, pixtral-large, small-glm, qwen-235b, wizard 8x22b, deepseek-v2.5.

Okay cool.  Lets say, Mistral/Devstral are great for OCR tasks.  There is no doubt.  But I found myself using the OCR extremely rarely.  Anyways, let it be.
But how the other can compare to DeepSeek R1-0528 or the V3.1?  I don't get it -- for what tasks (text-based) they can be used?  80% of time the programmers are spending debugging the code.  So the best quality possible is the paramount.  Are you using it for something else?  I am just curious.

---

👤 **Ph0rk0z** commented on **2025-09-01** at **23:11:33**

I am doing chat and chat + images. I.E. *fun*. So the model has to be entertaining and intelligent. R1 is a bit too insane, V3.1 is meh. I do like the older V3. Sits outside the "assistant" paradigm and really stresses model semantic abilities.

For coding, even the largest models still struggle. Plus the T/S you need isn't well served by hybrid inference. I just refuculated sageattention for my turning card to work in comfy. Kimi, Chimera, V3.1, Qwen coder all went down a ton of dead ends. I mainly succeeded via bouncing ideas off them. If I was banging away at 10t/s, I'd still be at it.

For my main use, a dense model, especially one with images and the right training compares. Goes fast on GPU(s). Doesn't have to be run at Q2 quants. Only Qwen-235b instruct and V3-0324 seemed worth it, if we're keeping it text only, out of all those MoE releases this year.

---

👤 **magikRUKKOLA** commented on **2025-09-01** at **23:19:07**

@Ph0rk0z 

Ah I see.  I am using them a little bit differently -- I am providing the access to the data and the tools like bash execution.  And do the looping until the problem is solved.  Running multiple R1s like that then exploring what they came up with.

---

👤 **ikawrakow** commented on **2025-09-26** at **16:49:59**

Nothing to be done here.