## 📌 [Issue #781](https://github.com/ikawrakow/ik_llama.cpp/issues/781) - Bug: build does not work on the current main branch.

| **Author** | `nimishchaudhari` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-09-15 |
| **Updated** | 2025-09-15 |

---

## 📄 Description

### What happened?

Hello, 

I did a git pull today and tried building, faced this error too many times: 



Built using: 
cmake -B ./build -DGGML_CUDA=ON -DGGML_BLAS=OFF

cmake --build ./build --config Release -j $(nproc)


Commit: d99cf7cb71392866c1356d26a2099b04be2141bf seems to be working though. 
Edit: It didn't work either.


### Name and Version

Main  branch as of 15/09/2025 19h GMT +1

### What operating system are you seeing the problem on?

Linux

### Relevant log output

```shell
rF16; KQHelper = FlashQKfp32<192, 1, 32>]’
/home/nimish/Programs/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:1576:102:   required from ‘void {anonymous}::FlashAttn<Dk, Dv, q_step, k_step>::compute(KHelper&, VHelper&, int, int, int, int, int, const float*, const char*, float*, float*, float*) [with KHelper = {anonymous}::HelperF16; VHelper = {anonymous}::HelperF16; int Dk = 192; int Dv = 128; int q_step = 1; int k_step = 32]’
/home/nimish/Programs/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:2076:15:   required from ‘void {anonymous}::iqk_flash_helper(KHelper&, VHelper&, int, int, int, int, int, const float*, const char*, float, float, float*, const float*, float*, float*) [with int Dk = 192; int Dv = 128; int k_step = 32; KHelper = HelperF16; VHelper = HelperF16]’
/home/nimish/Programs/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:2117:45:   required from ‘bool {anonymous}::iqk_flash_helper_T(KHelper&, ggml_type, int, int, int, int, int, int, const float*, const char*, const char*, float, float, float*, const float*, float*, float*) [with int Dk = 192; int Dv = 128; int k_step = 32; KHelper = HelperF16]’
/home/nimish/Programs/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:2166:56:   required from ‘bool {anonymous}::iqk_flash_helper_T(ggml_type, ggml_type, int, int, int, int, int, int, int, const float*, const char*, const char*, const char*, float, float, float*, const float*, float*, float*) [with int Dk = 192; int Dv = 128; int k_step = 32]’
/home/nimish/Programs/ik_llama.cpp/ggml/src/iqk/fa/iqk_fa_192_128.cpp:40:44:   required from here
/home/nimish/Programs/ik_llama.cpp/ggml/src/./iqk/fa/iqk_fa_templates.h:780:36: error: cannot convert ‘{anonymous}::F16::Data’ {aka ‘__m512’} to ‘__m256’
  780 |             vk[l] = v_expf(F16::sub(vk[l], vm));
      |                            ~~~~~~~~^~~~~~~~~~~
      |                                    |
      |                                    {anonymous}::F16::Data {aka __m512}
/home/nimish/Programs/ik_llama.cpp/ggml/src/./iqk/iqk_utils.h:160:36: note:   initializing argument 1 of ‘__m256 v_expf(__m256)’
  160 | static inline __m256 v_expf(__m256 x) {
      |                             ~~~~~~~^
gmake[2]: *** [ggml/src/CMakeFiles/ggml.dir/build.make:3730: ggml/src/CMakeFiles/ggml.dir/iqk/fa/iqk_fa_64_64.cpp.o] Error 1
gmake[2]: *** [ggml/src/CMakeFiles/ggml.dir/build.make:3674: ggml/src/CMakeFiles/ggml.dir/iqk/fa/iqk_fa_192_128.cpp.o] Error 1
gmake[1]: *** [CMakeFiles/Makefile2:1609: ggml/src/CMakeFiles/ggml.dir/all] Error 2
gmake: *** [Makefile:146: all] Error 2
Unable to build
```

---

## 💬 Conversation

👤 **nimishchaudhari** commented on **2025-09-15** at **17:12:05**

Complete log pasted here: 

https://pastebin.com/VYDz3paE

---

👤 **nimishchaudhari** commented on **2025-09-15** at **17:35:52**

False alert, recloning and re building worked, I close it :)