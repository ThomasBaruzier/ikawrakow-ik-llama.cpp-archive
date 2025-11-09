## 🔀 [Pull Request #680](https://github.com/ikawrakow/ik_llama.cpp/pull/680) - Fix quantized K cache without FA

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/fix_quantized_kv_nofa` |
| **Target Branch** | `main` |
| **Created** | 2025-08-08 |
| **Updated** | 2025-08-08 |
| **Merged** | 2025-08-08 |

---

## 📄 Description

Was broken in PR [#374](https://github.com/ikawrakow/ik_llama.cpp/issues/374). In that PR I added MMQ support for `IQ4_KS`, which has a per tensor row scale, so the usual `ggml`-style `row_size = ne00 / block_size * type_size` does not work. The change I made to make it work is fine for everything but quantized K cache without flash attention. The breakage in that case was not noticed. Strange that nobody else noticed since then (~3 months ago), until [#679](https://github.com/ikawrakow/ik_llama.cpp/issues/679) from today, which was based on investigating failures in speculative sampling observed for PR [#645](https://github.com/ikawrakow/ik_llama.cpp/issues/645).

Closes [#679](https://github.com/ikawrakow/ik_llama.cpp/issues/679)