## 📌 [Issue #685](https://github.com/ikawrakow/ik_llama.cpp/issues/685) - Bug: GGML_AVX512_VBMI does not compile on Ubuntu 22.04: cannot convert ‘{anonymous}::F16::Data’ {aka ‘__m512’} to ‘__m256’

| **Author** | `kkontosis` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-08-11 |
| **Updated** | 2025-08-11 |

---

## 📄 Description

### What happened?

After enabling GGML_AVX512 and then GGML_AVX512_VBMI, building with either g++ or g++-12 or g++-13 on Ubuntu 22.04 fails. I'm using the standard g++ from apt.
Trying clang++ but it does not get detected as a valid c++ compiler at all during cmake so I can't test it.
Edit: I found this in another issue but it didn't help either, added: -DCMAKE_CXX_FLAGS=-flax-vector-conversions

I've read in the discussion that these are useful because there are some AVX optimizations.
How can we build these extensions?




### Name and Version

```
llm@v48:~/ik_llama/ik_llama.cpp$ ./build2/bin/llama-server --version
version: 3835 (d60c8f4d)
built with cc (Ubuntu 11.4.0-1ubuntu1~22.04) 11.4.0 for x86_64-linux-gnu
```

Here are my processors flags in g++:
```shell
llm@v48:~/ik_llama/ik_llama.cpp$ g++ -march=native -### -E - </dev/null
Using built-in specs.
COLLECT_GCC=g++
OFFLOAD_TARGET_NAMES=nvptx-none:amdgcn-amdhsa
OFFLOAD_TARGET_DEFAULT=1
Target: x86_64-linux-gnu
Configured with: ../src/configure -v --with-pkgversion='Ubuntu 11.4.0-1ubuntu1~22.04' --with-bugurl=file:///usr/share/doc/gcc-11/README.Bugs --enable-languages=c,ada,c++,go,brig,d,fortran,objc,obj-c++,m2 --prefix=/usr --with-gcc-major-version-only --program-suffix=-11 --program-prefix=x86_64-linux-gnu- --enable-shared --enable-linker-build-id --libexecdir=/usr/lib --without-included-gettext --enable-threads=posix --libdir=/usr/lib --enable-nls --enable-bootstrap --enable-clocale=gnu --enable-libstdcxx-debug --enable-libstdcxx-time=yes --with-default-libstdcxx-abi=new --enable-gnu-unique-object --disable-vtable-verify --enable-plugin --enable-default-pie --with-system-zlib --enable-libphobos-checking=release --with-target-system-zlib=auto --enable-objc-gc=auto --enable-multiarch --disable-werror --enable-cet --with-arch-32=i686 --with-abi=m64 --with-multilib-list=m32,m64,mx32 --enable-multilib --with-tune=generic --enable-offload-targets=nvptx-none=/build/gcc-11-XeT9lY/gcc-11-11.4.0/debian/tmp-nvptx/usr,amdgcn-amdhsa=/build/gcc-11-XeT9lY/gcc-11-11.4.0/debian/tmp-gcn/usr --without-cuda-driver --enable-checking=release --build=x86_64-linux-gnu --host=x86_64-linux-gnu --target=x86_64-linux-gnu --with-build-config=bootstrap-lto-lean --enable-link-serialization=2
Thread model: posix
Supported LTO compression algorithms: zlib zstd
gcc version 11.4.0 (Ubuntu 11.4.0-1ubuntu1~22.04)
COLLECT_GCC_OPTIONS='-march=native' '-E' '-shared-libgcc'
 /usr/lib/gcc/x86_64-linux-gnu/11/cc1 -E -quiet -imultiarch x86_64-linux-gnu - "-march=znver2" -mmmx -mpopcnt -msse -msse2 -msse3 -mssse3 -msse4.1 -msse4.2 -mavx -mavx2 -msse4a -mno-fma4 -mno-xop -mfma -mno-avx512f -mbmi -mbmi2 -maes -mpclmul -mno-avx512vl -mno-avx512bw -mno-avx512dq -mno-avx512cd -mno-avx512er -mno-avx512pf -mno-avx512vbmi -mno-avx512ifma -mno-avx5124vnniw -mno-avx5124fmaps -mno-avx512vpopcntdq -mno-avx512vbmi2 -mno-gfni -mno-vpclmulqdq -mno-avx512vnni -mno-avx512bitalg -mno-avx512bf16 -mno-avx512vp2intersect -mno-3dnow -madx -mabm -mno-cldemote -mclflushopt -mclwb -mclzero -mcx16 -mno-enqcmd -mf16c -mfsgsbase -mfxsr -mno-hle -msahf -mno-lwp -mlzcnt -mmovbe -mno-movdir64b -mno-movdiri -mmwaitx -mno-pconfig -mno-pku -mno-prefetchwt1 -mprfchw -mno-ptwrite -mrdpid -mrdrnd -mrdseed -mno-rtm -mno-serialize -mno-sgx -msha -mno-shstk -mno-tbm -mno-tsxldtrk -mno-vaes -mno-waitpkg -mwbnoinvd -mxsave -mxsavec -mxsaveopt -mxsaves -mno-amx-tile -mno-amx-int8 -mno-amx-bf16 -mno-uintr -mno-hreset -mno-kl -mno-widekl -mno-avxvnni --param "l1-cache-size=32" --param "l1-cache-line-size=64" --param "l2-cache-size=512" "-mtune=znver2" -fasynchronous-unwind-tables -fstack-protector-strong -Wformat -Wformat-security -fstack-clash-protection -fcf-protection -dumpbase -
COMPILER_PATH=/usr/lib/gcc/x86_64-linux-gnu/11/:/usr/lib/gcc/x86_64-linux-gnu/11/:/usr/lib/gcc/x86_64-linux-gnu/:/usr/lib/gcc/x86_64-linux-gnu/11/:/usr/lib/gcc/x86_64-linux-gnu/
LIBRARY_PATH=/usr/lib/gcc/x86_64-linux-gnu/11/:/usr/lib/gcc/x86_64-linux-gnu/11/../../../x86_64-linux-gnu/:/usr/lib/gcc/x86_64-linux-gnu/11/../../../../lib/:/lib/x86_64-linux-gnu/:/lib/../lib/:/usr/lib/x86_64-linux-gnu/:/usr/lib/../lib/:/usr/lib/gcc/x86_64-linux-gnu/11/../../../:/lib/:/usr/lib/
COLLECT_GCC_OPTIONS='-march=native' '-E' '-shared-libgcc'
```

### What operating system are you seeing the problem on?

Ubuntu 22.04

### Relevant log output

It breaks here iqk_fa_templates.h:918:

```shell
llm@v48:~/ik_llama/ik_llama.cpp$ cmake -B ./build3 -DGGML_CUDA=ON -DGGML_BLAS=OFF -D CMAKE_CXX_COMPILER=g++-13 -DCMAKE_CXX_FLAGS=-flax-vector-conversions -DGGML_AVX512_VBMI=ON -DGGML_AVX512=ON
...
llm@v48:~/ik_llama/ik_llama.cpp$ cmake --build ./build3 --config Release -j 24
...
[ 44%] Building CXX object ggml/src/CMakeFiles/ggml.dir/iqk/iqk_mul_mat.cpp.o
In file included from /srv/home/llm/ik_llama/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:1174:
/srv/home/llm/ik_llama/ik_llama.cpp/ggml/src/iqk/fa/iqk_fa_templates.h: In member function ‘float {anonymous}::FlashMS<q_step_in, k_step_in>::load_apply_mask_and_scale(int, {anonymous}::F16::Data*, const char*)’:
/srv/home/llm/ik_llama/ik_llama.cpp/ggml/src/iqk/fa/iqk_fa_templates.h:918:105: error: cannot convert ‘{anonymous}::F16::Data’ {aka ‘__m512’} to ‘__m256’
  918 |             for (int l = 0; l < k_step/F16::block_size; ++l) vk[l] = F16::mul(v_softcap, v_tanh(F16::mul(vscale, vk[l])));
      |                                                                                                 ~~~~~~~~^~~~~~~~~~~~~~~
      |                                                                                                         |
      |                                                                                                         {anonymous}::F16::Data {aka __m512}
In file included from /srv/home/llm/ik_llama/ik_llama.cpp/ggml/src/iqk/iqk_mul_mat.cpp:30:
/srv/home/llm/ik_llama/ik_llama.cpp/ggml/src/iqk/iqk_utils.h:180:36: note:   initializing argument 1 of ‘__m256 v_tanh(__m256)’
  180 | static inline __m256 v_tanh(__m256 x) {
      |                             ~~~~~~~^

```

This error is also seen here in iqk_fa_templates.h:945:
```shell
/srv/home/llm/ik_llama/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:785:36: error: cannot convert ‘{anonymous}::F16::Data’ {aka ‘__m512’} to ‘__m256’
  785 |             vk[l] = v_expf(F16::sub(vk[l], vm));
      |                            ~~~~~~~~^~~~~~~~~~~
      |                                    |
      |                                    {anonymous}::F16::Data {aka __m512}
/srv/home/llm/ik_llama/ik_llama.cpp/ggml/src/./iqk/iqk_utils.h:142:36: note:   initializing argument 1 of ‘__m256 v_expf(__m256)’
  142 | static inline __m256 v_expf(__m256 x) {
      |                             ~~~~~~~^
/srv/home/llm/ik_llama/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h: In instantiation of ‘void {anonymous}::FlashMS<q_step_in, k_step_in>::update_S(int, {anonymous}::F16::Data*) [with int q_step_in = 8; int k_step_in = 32; {anonymous}::F16::Data = __m512]’:
/srv/home/llm/ik_llama/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:945:31:   required from ‘void {anonymous}::FlashMS<q_step_in, k_step_in>::update_M_S(int, {anonymous}::F16::Data*, const char*) [with int q_step_in = 8; int k_step_in = 32; {anonymous}::F16::Data = __m512]’
/srv/home/llm/ik_llama/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1300:27:   required from ‘static void {anonymous}::FlashQKfp32<D, q_step, k_step>::mul_mask_kq(const KHelper&, int, const block_q8*, const char*, {anonymous}::FlashMS<q_step, k_step>&) [with KHelper = {anonymous}::HelperQ80R8<576>; block_q8 = block_q8_2; int D = 576; int q_step = 8; int k_step = 32]’
/srv/home/llm/ik_llama/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1443:34:   required from ‘void {anonymous}::compute_helper_q(KHelper&, VHelper&, int, int, int, int, int, FlashMS<q_step, k_step>&, FlashQKV<Dv, q_step, k_step>&, const float*, const char*, float*, float*, float*, char*) [with int Dk = 576; int Dv = 512; int q_step = 8; int k_step = 32; KHelper = HelperQ80R8<576>; VHelper = HelperQ80; KQHelper = FlashQKfp32<576, 8, 32>]’
/srv/home/llm/ik_llama/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1535:124:   required from ‘void {anonymous}::FlashAttn<Dk, Dv, q_step, k_step>::compute(KHelper&, VHelper&, int, int, int, int, int, const float*, const char*, float*, float*, float*) [with KHelper = {anonymous}::HelperQ80; VHelper = {anonymous}::HelperQ80; int Dk = 576; int Dv = 512; int q_step = 8; int k_step = 32]’
/srv/home/llm/ik_llama/ik_llama.cpp/ggml/src/iqk/fa/iqk_fa_576_512.cpp:31:19:   required from ‘void {anonymous}::iqk_deepseek_helper(KHelper&, VHelper&, int, int, int, int, int, const float*, const char*, float, float, float*, float*, float*) [with int step_k = 32; KHelper = HelperQ80; VHelper = HelperQ80]’
/srv/home/llm/ik_llama/ik_llama.cpp/ggml/src/iqk/fa/iqk_fa_576_512.cpp:58:36:   required from ‘bool {anonymous}::iqk_deepseek_helper(ggml_type, int, int, int, int, int, int, int, const float*, const char*, const char*, const char*, float, float, float*, float*, float*) [with int step_k = 32]’
/srv/home/llm/ik_llama/ik_llama.cpp/ggml/src/iqk/fa/iqk_fa_576_512.cpp:115:35:   required from here
/srv/home/llm/ik_llama/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:785:36: error: cannot convert ‘{anonymous}::F16::Data’ {aka ‘__m512’} to ‘__m256’
```

---

## 💬 Conversation

👤 **kkontosis** commented on **2025-08-11** at **16:16:55**

I just figured it out, my CPU doesn't support AVX-512.
I was able to troubleshoot this via:
`g++ -march=native -### -E - </dev/null`

---

👤 **ikawrakow** commented on **2025-08-11** at **16:24:24**

You can also just do `cat /proc/cpuinfo` and look for the `flags` line. If you don't see `avx512` there your CPU does not support it.