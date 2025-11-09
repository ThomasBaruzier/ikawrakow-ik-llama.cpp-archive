## 🔀 [Pull Request #717](https://github.com/ikawrakow/ik_llama.cpp/pull/717) - Remove scary warning about incompatible model

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/remove_scary_warning` |
| **Target Branch** | `main` |
| **Created** | 2025-08-22 |
| **Updated** | 2025-08-22 |
| **Merged** | 2025-08-22 |

---

## 📄 Description

When loading a DeepSeek model that does not have the attention `wkv_b` tensors included, `ik_llama.cpp` prints this scary warning 
```
==========================================================================
Detected incompatible DeepSeek model.
Will try to fix, but there are no guarantees

*** Your prompt processing speed will be crippled ***

Consider making your own ik_llama.cpp compatible model or
ask the model provider to make one for you,
==========================================================================
```

As the performance issues around missing `wkv_b` tensors were solved quite a while ago, there is no longer the need for this message. This PR removes it. It still logs
```
================= Adjusted mainline llama.cpp MLA tensors to ik_llama.cpp
```
so we know the adjustment of MLA-related model parameters worked.