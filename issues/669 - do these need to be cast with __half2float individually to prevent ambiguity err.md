## 📌 [Issue #669](https://github.com/ikawrakow/ik_llama.cpp/issues/669) - do these need to be cast with __half2float individually to prevent ambiguity errors

| **Author** | `aatri2021` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-08-02 |
| **Updated** | 2025-08-03 |

---

## 📄 Description

https://github.com/ikawrakow/ik_llama.cpp/blame/ae0ba31fd078282fe6ac675176862ed6955c52dc/ggml/src/ggml-cuda/dmmv.cu#L59

I think the casts in this section might be causing ambiguity errors?

bdot1.x += y[ 0].x * fabsf((float)(h[0] + h[1])) * (signs1 & mask1 ? -1 : 1);

should maybe instead be:

bdot1.x += y[ 0].x * fabsf(__half2float(h[0]) + __half2float(h[1])) * (signs1 & mask1 ? -1 : 1);


The compile error i currently get:
ik_llama.cpp/ggml/src/ggml-cuda/dmmv.cu(58): error: more than one conversion function from "const half" to a built-in type applies:
            function "__half::operator float() const" (declared at line 217 of /usr/local/cuda-12.1/include/cuda_fp16.hpp)
            function "__half::operator short() const" (declared at line 235 of /usr/local/cuda-12.1/include/cuda_fp16.hpp)
            function "__half::operator unsigned short() const" (declared at line 238 of /usr/local/cuda-12.1/include/cuda_fp16.hpp)
            function "__half::operator int() const" (declared at line 241 of /usr/local/cuda-12.1/include/cuda_fp16.hpp)
            function "__half::operator unsigned int() const" (declared at line 244 of /usr/local/cuda-12.1/include/cuda_fp16.hpp)
            function "__half::operator long long() const" (declared at line 247 of /usr/local/cuda-12.1/include/cuda_fp16.hpp)
            function "__half::operator unsigned long long() const" (declared at line 250 of /usr/local/cuda-12.1/include/cuda_fp16.hpp)
            function "__half::operator __nv_bool() const" (declared at line 254 of /usr/local/cuda-12.1/include/cuda_fp16.hpp)
      bdot1.x += y[ 0].x * fabsf(__half2float(h[0] + h[1])) * (signs1 & mask1 ? -1 : 1);

---

## 💬 Conversation

👤 **aatri2021** commented on **2025-08-02** at **05:59:44**

Quick update - compiles successfully after these changes if someone else hits the same problem.

---

👤 **ikawrakow** commented on **2025-08-02** at **16:40:14**

Feel free to submit a PR.

This code is not actually used. It is the original Trellis quant implementation based on fp16. I have left it behind for now, so even just commenting out the lines causing compilation errors would be OK.

---

👤 **aatri2021** commented on **2025-08-03** at **02:15:09**

Hi thanks for the response - looks like i forgot to  put in -DLLAMA_CUDA_F16=on during cmake hence it hit the other part of the compile. Once i re-ran cmake that code doesnt run as you said. I doubt anyone is making the build with that old gpu/cuda.