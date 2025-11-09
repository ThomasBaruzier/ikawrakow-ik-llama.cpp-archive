## 🔀 [Pull Request #803](https://github.com/ikawrakow/ik_llama.cpp/pull/803) - Fix gemma3 vision

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/fix_gemma3_vision` |
| **Target Branch** | `main` |
| **Created** | 2025-09-27 |
| **Updated** | 2025-09-27 |
| **Merged** | 2025-09-27 |

---

## 📄 Description

Basically remove completely unnecessary asserts in `im2col`. Else vision inference for Gemma3 fails as `v.patch_embd.weight` is left as `f32` when creating the mmproj file, but `im2col` asserts that the type of `src0` is `f16`.  

~Wonder if anyone has ever used Gemma3 vision with mainline.~ I see they now have separate `im2col` versions for `f16` and `f32`.