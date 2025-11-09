## 🔀 [Pull Request #707](https://github.com/ikawrakow/ik_llama.cpp/pull/707) - Disable experimental code that causes issues with MSVC

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/disable_experimental_code1` |
| **Target Branch** | `main` |
| **Created** | 2025-08-19 |
| **Updated** | 2025-08-19 |
| **Merged** | 2025-08-19 |

---

## 📄 Description

See [#706](https://github.com/ikawrakow/ik_llama.cpp/issues/706) 

The code that causes the MSVC build failure is just playing around with some statistics around quantization, so totally unused in practice. Instead of fighting with diverging compiler's opinions if `constexpr` can or cannot be used in lambdas, let's just disable this code.