## 🔀 [Pull Request #902](https://github.com/ikawrakow/ik_llama.cpp/pull/902) - Adding cmake option to disable CUDA fusion

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/option_to_disable_cuda_fusion` |
| **Target Branch** | `main` |
| **Created** | 2025-11-05 |
| **Updated** | 2025-11-05 |
| **Merged** | 2025-11-05 |

---

## 📄 Description

There have been reports about issues in recent `ik_llama.cpp` versions ([#893](https://github.com/ikawrakow/ik_llama.cpp/issues/893), [#895](https://github.com/ikawrakow/ik_llama.cpp/issues/895)). My hypothesis is that they are caused by bugs in one or more of the op fusions on CUDA introduced in recent PRs. To test this hypothesis, this PR adds the ability to enable or disable CUDA fusion via `cmake`

To disable
```
cmake -DGGML_CUDA_FUSION=0 $other_cmake_args
```