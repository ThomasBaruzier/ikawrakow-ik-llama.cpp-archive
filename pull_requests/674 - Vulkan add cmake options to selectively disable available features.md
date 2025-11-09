## 🔀 [Pull Request #674](https://github.com/ikawrakow/ik_llama.cpp/pull/674) - Vulkan: add cmake options to selectively disable available features 

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/vulkan1` |
| **Target Branch** | `main` |
| **Created** | 2025-08-07 |
| **Updated** | 2025-08-07 |
| **Merged** | 2025-08-07 |

---

## 📄 Description

My RTX-4080 GPU with the 575 driver from Nvidia supports NV coopmat (a.k.a. coopmat2) and `bf16`, but it seems many AMD GPU users are only having KHR coopmat or even no coopmat support at all. Hence adding `cmake` options to selectively disable Vulkan features that I have available so I can test that things also work on a lower capabilities device.