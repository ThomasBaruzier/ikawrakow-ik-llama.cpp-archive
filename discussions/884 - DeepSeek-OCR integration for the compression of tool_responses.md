## 🗣️ [Discussion #884](https://github.com/ikawrakow/ik_llama.cpp/discussions/884) - DeepSeek-OCR integration for the compression of tool_responses.

| **Author** | `magikRUKKOLA` |
| :--- | :--- |
| **State** | ✅ **Open** |
| **Created** | 2025-10-31 |
| **Updated** | 2025-11-01 |

---

## 📄 Description

Suppose we are using the Ling-1T or any other LLM in the loop of tool-calls.  Suppose we are having the prompt:

```
Output the mandelbrot fractal using python in ascii format /python
```

For example the DeepSeek-R1 would chose the 80x40 resolution.   So when we will get the tool-response and it would go to the model's prefill, it would eat up 3.2k tokens (roughly).

What if, for example, for the long tool-responses we would just convert it into the graphical representation for the DeepSeek-OCR to produce a text/markdown as a short text representation of the tool call execution.  And instead of the original long tool call response we would send what the DeepSeek-OCR would produce.

[1] https://github.com/deepseek-ai/DeepSeek-OCR