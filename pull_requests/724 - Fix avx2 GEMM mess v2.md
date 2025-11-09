## 🔀 [Pull Request #724](https://github.com/ikawrakow/ik_llama.cpp/pull/724) - Fix avx2 GEMM mess (v2)

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/fix_avx2_gemm_mess` |
| **Target Branch** | `main` |
| **Created** | 2025-08-24 |
| **Updated** | 2025-08-27 |
| **Merged** | 2025-08-27 |

---

## 📄 Description

_No description provided._

---

## 💬 Conversation

👤 **Ph0rk0z** commented on **2025-08-27** at **11:41:08**

Here's the full message:

```
/home/supermicro/ai/ik_llama.cpp/ggml/src/iqk/iqk_gemm_legacy_quants.cpp: In function 'bool iqk_set_kernels_legacy_quants(int, int, int, std::array<void (*)(int, const void*, long unsigned int, const DataInfo&, int), 8>&, void (*&)(int, const void*, size_t, const DataInfo&, int))':
/home/supermicro/ai/ik_llama.cpp/ggml/src/iqk/iqk_gemm_legacy_quants.cpp:1991:27: error: 'IQ4_NL_UnpackernS' was not declared in this scope
 1991 |             set_functions<IQ4_NL_UnpackernS>(kernels);
      |                           ^~~~~~~~~~~~~~~~~
/home/supermicro/ai/ik_llama.cpp/ggml/src/iqk/iqk_gemm_legacy_quants.cpp:1991:45: error: no matching function for call to 'set_functions<<expression error> >(std::array<void (*)(int, const void*, long unsigned int, const DataInfo&, int), 8>&)'
 1991 |             set_functions<IQ4_NL_UnpackernS>(kernels);
      |             ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~^~~~~~~~~
/home/supermicro/ai/ik_llama.cpp/ggml/src/iqk/iqk_gemm_legacy_quants.cpp:1916:38: note: candidate: 'template<class Dequantizer> void {anonymous}::set_functions(std::array<void (*)(int, const void*, long unsigned int, const DataInfo&, int), 8>&)'
 1916 | template <typename Dequantizer> void set_functions(std::array<mul_mat_t, IQK_MAX_NY>& funcs) {
      |                                      ^~~~~~~~~~~~~
/home/supermicro/ai/ik_llama.cpp/ggml/src/iqk/iqk_gemm_legacy_quants.cpp:1916:38: note:   template argument deduction/substitution failed:
/home/supermicro/ai/ik_llama.cpp/ggml/src/iqk/iqk_gemm_legacy_quants.cpp:1991:45: error: template argument 1 is invalid
 1991 |             set_functions<IQ4_NL_UnpackernS>(kernels);
      |             ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~^~~~~~~~~
make[2]: *** [ggml/src/CMakeFiles/ggml.dir/build.make:2515: ggml/src/CMakeFiles/ggml.dir/iqk/iqk_gemm_legacy_quants.cpp.o] Error 1
make[1]: *** [CMakeFiles/Makefile2:1997: ggml/src/CMakeFiles/ggml.dir/all] Error 2
make: *** [Makefile:146: all] Error 2
```

---

👤 **ikawrakow** commented on **2025-08-27** at **11:44:42**

Sorry about that. Should be fixed on the main branch now.

---

👤 **Ph0rk0z** commented on **2025-08-27** at **11:46:13**

Yup. compiles now. Couldn't even tell it was a typo.