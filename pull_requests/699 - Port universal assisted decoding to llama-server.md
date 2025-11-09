## 🔀 [Pull Request #699](https://github.com/ikawrakow/ik_llama.cpp/pull/699) - Port universal assisted decoding to llama-server

| **Author** | `g2mt` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `universal-decoding` |
| **Target Branch** | `main` |
| **Created** | 2025-08-16 |
| **Updated** | 2025-08-18 |
| **Merged** | 2025-08-18 |

---

## 📄 Description

- [x] I have read the [contributing guidelines](https://github.com/ggerganov/llama.cpp/blob/master/CONTRIBUTING.md)
- Self-reported review complexity:
  - [ ] Low
  - [x] Medium
  - [ ] High

See https://github.com/ggml-org/llama.cpp/pull/12635 for equivalent PR in mainline. Related to [#645](https://github.com/ikawrakow/ik_llama.cpp/issues/645)

---

## 💬 Conversation

👤 **ikawrakow** commented on **2025-08-17** at **04:56:08**

It would be nice to have an example telling us how to test and use (which models to use, command line, etc.). For people like me who don't have the hardware to run giant models, it would be nice if there was an examples with main_draft models that can be loaded in 64 GB RAM + 16 GB VRAM.

---

👤 **g2mt** commented on **2025-08-18** at **04:16:35**

It should work the same as passing a compatible draft model. I also added the `--spec-replace` argument for translating the template tags.

Here are my prompt/generation speeds for a simple repeat prompt. I'm using Devstral Small with Qwen2 coder 0.5 as the draft model.

**without speculative decoding:**

```
"ik_llama.cpp/build/bin/Release/llama-server.exe"  "-m" "Devstral-Small-2507-UD-Q4_K_XL.gguf" "-c" "32768" "--run-time-repack" "-t" "12"
```

<img width="1308" height="635" alt="before-spec" src="https://github.com/user-attachments/assets/6e8e1f16-de60-4f3f-abfe-4b69aed17b3c" />

**with it enabled:**

```
"ik_llama.cpp/build/bin/Release/llama-server.exe"  "--port" "10006" "-m" "Devstral-Small-2507-UD-Q4_K_XL.gguf" "-c" "32768" "--run-time-repack" "-t" "12" "-md" "Qwen2.5-Coder-0.5B-Instruct-Q8_0.gguf" "--spec-replace" "[INST]" "<|im_begin|>user\n" "--spec-replace" "[/INST]" "<|im_end|><|im_begin|>assistant" "--spec-replace" "</s>" "<|im_begin|>user\n"  
```

<img width="1271" height="625" alt="after-spec" src="https://github.com/user-attachments/assets/c0abaa90-6896-49d2-ba3c-616bb3925054" />

---

👤 **ikawrakow** started a conversation on `common/common.cpp` on **2025-08-18** at **05:21:36**

```
params.replacements_draft.emplace_back(std::move(target), std::move(draft));
```
avoids the 4 string copies. Not that this is performance critical or something, but still.

---

👤 **ikawrakow** started a conversation on `common/speculative.cpp` on **2025-08-18** at **05:29:40**

Why do we need to sort according to length?

> 👤 **g2mt** replied on **2025-08-18** at **05:55:24**
> 
> This is so that longer substrings get replaced first. If it's not a problem then I'll remove the sorting

> 👤 **ikawrakow** replied on **2025-08-18** at **05:57:26**
> 
> I get that. I was just curious why we want to replace the longer substrings first.

> 👤 **g2mt** replied on **2025-08-18** at **06:10:02**
> 
> This is how I usually do text replacements. It prevents replacement conflicts. But yeah I can see it being unnecessary.

---

👤 **ikawrakow** approved this pull request ✅ on **2025-08-18** at **05:31:59**