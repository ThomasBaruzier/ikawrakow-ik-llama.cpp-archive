## 🔀 [Pull Request #842](https://github.com/ikawrakow/ik_llama.cpp/pull/842) - Make only-active-expert-offload on by default and add to llama-bench

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/ooae_on_by_default` |
| **Target Branch** | `main` |
| **Created** | 2025-10-20 |
| **Updated** | 2025-10-20 |
| **Merged** | 2025-10-20 |

---

## 📄 Description

It is always at least as good as offloading all experts. For some models (e.g., GPT-OSS) it is much better.
So, no reason to have the user remember to add the `-ooae` flag.

But if offloading only active experts in hybrid inference causes issues, it can be turned off via `-no-ooae` or `-=no-offload-only-active-experts`.

Also added corresponding flags to `llama-bench`.