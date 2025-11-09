## 🔀 [Pull Request #863](https://github.com/ikawrakow/ik_llama.cpp/pull/863) - Change flash attention and fmoe to be on by default

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/change_fmoe_fa_defaults` |
| **Target Branch** | `main` |
| **Created** | 2025-10-25 |
| **Updated** | 2025-10-25 |
| **Merged** | 2025-10-25 |

---

## 📄 Description

This also changes the accepted command line arguments:

```
-fa | --flash-attn -> -no-fa | --no-flash-attn
```
and
```
-fmoe | --fused-moe -> -no-fmoe | --no-fused-moe
```