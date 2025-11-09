## 🔀 [Pull Request #817](https://github.com/ikawrakow/ik_llama.cpp/pull/817) - Remove duplicate 99% KLD output, add additional percentiles to match mainline

| **Author** | `AesSedai` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `update-perplexity` |
| **Target Branch** | `main` |
| **Created** | 2025-10-05 |
| **Updated** | 2025-10-05 |
| **Merged** | 2025-10-05 |

---

## 📄 Description

There was a PR recently on mainline to add the additional percentiles to `llama-perplexity` [here](https://github.com/ggml-org/llama.cpp/pull/16321) and since I was doing some perplexity testing with ik_llama, I figured I'd put up a PR to fix the 90% being listed twice, and add in the other percentiles to match mainline.

I compiled and ran a `llama-perplexity` run to verify that the output is showing correctly:
```
...
====== KL divergence statistics ======
Mean    KLD:   0.017475 ±   0.000763
Maximum KLD:  11.152802
99.9%   KLD:   1.098822
99.0%   KLD:   0.221226
95.0%   KLD:   0.055542
90.0%   KLD:   0.029520
Median  KLD:   0.004062
10.0%   KLD:   0.000020
 5.0%   KLD:   0.000005
 1.0%   KLD:   0.000000
 0.1%   KLD:  -0.000003
Minimum KLD:  -0.000423
...
```

- [x] I have read the [contributing guidelines](https://github.com/ggerganov/llama.cpp/blob/master/CONTRIBUTING.md)
- Self-reported review complexity:
  - [x] Low
  - [ ] Medium
  - [ ] High

---

## 💬 Conversation

👤 **ikawrakow** approved this pull request ✅ on **2025-10-05** at **05:13:24**