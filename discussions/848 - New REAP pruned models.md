## 🗣️ [Discussion #848](https://github.com/ikawrakow/ik_llama.cpp/discussions/848) - New REAP pruned models.

| **Author** | `Ph0rk0z` |
| :--- | :--- |
| **State** | ✅ **Open** |
| **Created** | 2025-10-20 |
| **Updated** | 2025-10-20 |

---

## 📄 Description

Anyone have success with pruned weights such as: https://huggingface.co/AesSedai/GLM-4.6-REAP-266B-A32B

The evals look pretty decent and it would give a much needed quant size/speed buff to weaker systems. Some models might now even fit in vram.

Apparently there are conversion issues with mainline so it could be worth looking into. Probably my only chance to run kimi at better than Q1 if a 50% prune appears. Certainly looking forward to minified GLM already.