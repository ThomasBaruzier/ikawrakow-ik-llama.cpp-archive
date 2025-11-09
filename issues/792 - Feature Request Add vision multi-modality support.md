## 📌 [Issue #792](https://github.com/ikawrakow/ik_llama.cpp/issues/792) - Feature Request: Add vision / multi-modality support

| **Author** | `Thireus` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-09-24 |
| **Updated** | 2025-09-27 |
| **Labels** | `enhancement` |

---

## 📄 Description

### Prerequisites

- [x] I am running the latest code. Mention the version if possible as well.
- [x] I carefully followed the [README.md](https://github.com/ggerganov/llama.cpp/blob/master/README.md).
- [x] I searched using keywords relevant to my issue to make sure that I am creating a new issue that is not already open (or closed).
- [x] I reviewed the [Discussions](https://github.com/ggerganov/llama.cpp/discussions), and have a new and useful enhancement to share.

### Feature Description

Add support in `ik_llama.cpp` for vision and multi-modality, allowing models that process both images and text (vision-language models, VLMs) to run natively. This would involve:

- Loading and handling image inputs (e.g. image paths, image tensors, base64 / URL encoded) alongside textual prompts.  
- Converting image inputs into embeddings (via a vision encoder or projector) to feed into the language model.  
- Supporting `.mmproj` (or equivalent) multimodal projector metadata / files, where necessary, to enable models trained with separate projector components.  
- Enabling multimodal inference in both CLI mode and server mode (or whatever interface `ik_llama.cpp` offers), so users can mix text + image in prompts.  

### Motivation

Several reasons why this would be valuable:

1. **Parity with recent advances:** The `llama.cpp` library has recently added vision / multi-modality support via @ngxson’s work (especially the `libmtmd` library), `.mmproj` support, unified multimodal CLI tool (`llama-mtmd-cli`), and experimental integration into `llama-server`.  

2. **New model families demand support:**  
   - The Qwen series has released VLMs / Visual-Language variants (e.g. Qwen2-VL, Qwen2.5-VL, Qwen3-VL) that are increasingly used / requested.  
   - Other released models like **Gemma3 Vision** or **InternVL3_5** are prominent; users are asking for `ik_llama.cpp` to support such architectures. https://github.com/ikawrakow/ik_llama.cpp/issues/615, https://github.com/ikawrakow/ik_llama.cpp/issues/730

3. **Ecosystem growth:**  
   - Without image capability, `ik_llama.cpp` misses out on many use-cases (image captioning, visual question answering, OCR, etc.), which people increasingly expect from VLM frameworks.  

4. **Strategic value:** Supporting vision will broaden the user base, help keep `ik_llama.cpp` competitive, and reduce fragmentation (i.e. people might otherwise need to switch to `llama.cpp` or other tools for vision tasks).

### Possible Implementation

Here are possible ways this could be done, drawing on existing work (especially @ngxson / llama.cpp) [PR[#12898](https://github.com/ikawrakow/ik_llama.cpp/issues/12898)](https://github.com/ggml-org/llama.cpp/pull/12898) as reference, and suggestions specific to `ik_llama.cpp`.

| Step | Description |
|---|-------------|
| **Investigate existing `llama.cpp` implementation** | Review how `libmtmd` works: how image-projector / mmproj files are structured, how image input is parsed, how vision embedding is done, how multimodal tokens are merged with text tokens. Study the `llama-mtmd-cli` and `server` changes in llama.cpp. |
| **Define image input API for ik_llama.cpp** | Decide how users will provide images (file paths, base64 URLs, etc.), and integrate into existing prompt format. Possibly extend CLI and server interfaces. |
| **Implement vision encoder / projector support** | Either embed a compatible projector (e.g. via a shared library or porting `libmtmd` or equivalent) or provide support for reading `.mmproj` files / equivalent so that VLMs trained using separate projectors can run. Must handle image resizing, patch embedding, positional encoding (e.g. techniques like M-RoPE used in some models), etc. |
| **Modify model loading / metadata** | Extend model loader to recognize when a model is multimodal (maybe via a flag or presence of projector metadata), so that the model is configured appropriately. Possibly include conversion or checking tools. |
| **Inference integration** | Once image embeddings are ready, integrate them into the transformer stack alongside text. Handle batching, merging modalities, maintaining context, etc. Add support to `ik_llama.cpp`’s inference engine. |
| **Interface / server support** | Extend `ik_llama.cpp` server mode to support HTTP / API requests that include image payloads, as llama.cpp’s server does with `--mmproj` + model + appropriate request schemas. |
| **Backward compatibility & fallbacks** | Ensure that text-only models still work, with minimal overhead. Possibly make vision dependency optional. Handle missing projector files gracefully. |
| **Testing & examples** | Provide example VLMs (e.g. Qwen2-VL, Qwen2.5-VL, Qwen3-VL, Gemma3 Vision) and example prompts + images. Write tests (unit/integration) to ensure correctness on vision tasks (e.g. captioning). |

---

## 💬 Conversation

👤 **Ph0rk0z** commented on **2025-09-24** at **12:56:17**

Yea this is going to be necessary. Should be possible to port the code from mainline, assumptively. No way to run this new 235b with vision otherwise. EXL3 already full with 3.0 weights. Likewise there's no way to run the vision ernie model since it requires hybrid inference. There's support in VLLM but it's cpu support will be mega slow.

---

👤 **ikawrakow** commented on **2025-09-24** at **13:01:14**

Given all the alternative options that users have available, what is the benefit of being able to use `ik_llama.cpp` for vision / multi-modality?

---

👤 **Thireus** commented on **2025-09-24** at **13:52:25**

Along the lines of what I mentioned in 'Strategic value', for my own use case it complexifies my operating model since I have to support another framework just for vision, and it adds latency since I'd have to unload the current model and load a new one just for vision (which takes minutes). Another option would be to switch away from ik_llama.cpp, but I would lose on ik_llama.cpp benefits.

I plan on using Qwen3-VL for unified text and vision tasks, which would drastically simplify my design.

---

👤 **Ph0rk0z** commented on **2025-09-24** at **14:44:13**

There's really no alternative for larger models. I *can* use mainline but then will suffer slow prompt processing and text generation. 

Using a secondary captioning model is not as good as a real multimodal.

---

👤 **ikawrakow** commented on **2025-09-25** at **12:53:13**

WIP in [#798](https://github.com/ikawrakow/ik_llama.cpp/issues/798)

---

👤 **ikawrakow** commented on **2025-09-27** at **07:26:15**

Closed via [#798](https://github.com/ikawrakow/ik_llama.cpp/issues/798)

Adding to server can be a separate issue.