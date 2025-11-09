## 🔀 [Pull Request #922](https://github.com/ikawrakow/ik_llama.cpp/pull/922) - ggml : add ARM Grace Blackwell support for IQK operations

| **Author** | `novalis78` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `arm-grace-blackwell-support` |
| **Target Branch** | `main` |
| **Created** | 2025-11-08 |
| **Updated** | 2025-11-09 |
| **Merged** | 2025-11-09 |

---

## 📄 Description

## Summary
Enables IQK quantization operations on ARM-based systems, specifically NVIDIA DGX Spark with GB10 Grace Blackwell architecture.

## Changes
**File**: `ggml/src/iqk/iqk_cpu_ops.cpp`
- Added `IQK_IMPLEMENT` macro definition to enable IQK functions on ARM
- Added `arm_neon.h` header include for ARM NEON SIMD intrinsics

## Build Instructions for ARM
```bash
  cmake .. -DGGML_CUDA=ON \
           -DCMAKE_CXX_FLAGS="-march=armv8.2-a+dotprod+fp16" \
           -DCMAKE_C_FLAGS="-march=armv8.2-a+dotprod+fp16"
  cmake --build . --config Release

```
Fixes build errors:
- 'float32x4_t' does not name a type
- 'vld1q_f32' was not declared in this scope
- 'v_expf' was not declared in this scope
- Missing FP16 NEON intrinsics

Testing

Platform: NVIDIA DGX Spark
  - Architecture: aarch64 (ARM)
  - CPU: GB10 Grace Blackwell 
  - Memory: 128GB unified memory
  - GPU: NVIDIA GB10

  Verification:
  - Build completes successfully with CUDA support
  -  Model loading tested with Kimi-K2-Thinking (219GB GGUF, IQ1_KT quantization)
  - CUDA backend correctly detects GB10 GPU
  - IQK quantization types properly recognized

Note: Full CI pipeline not run locally due to ARM hardware requirement. Available for ongoing ARM-specific testing and maintenance.


- [x] I have read the [contributing guidelines](https://github.com/ggerganov/llama.cpp/blob/master/CONTRIBUTING.md)
- Self-reported review complexity:
  - [ ] Low
  - [x ] Medium
  - [ ] High

---

## 💬 Conversation

👤 **ikawrakow** approved this pull request ✅ on **2025-11-08** at **11:03:28**

Thank you for the fix.

The PR is fine like this, but I would like to understand why we end up needing to explicitly define `IQK_IMPLEMENT` (so the same mistake doen't happen again in the future).

If we build with `-DCMAKE_CXX_FLAGS="-march=armv8.2-a+dotprod+fp16"` as per PR build instructions, this should define `__ARM_FEATURE_DOTPROD`. `iqk_cpu_ops.cpp` includes `iqk_utils.h`, which in turn includes `iqk_config.h`, where `IQK_IMPLEMENT` should get defined when `__ARM_FEATURE_DOTPROD` is defined.