## 🔀 [Pull Request #774](https://github.com/ikawrakow/ik_llama.cpp/pull/774) - Fix convert_hf_to_gguf error for Ernie 4.5

| **Author** | `firecoperana` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `fcp/convert_script_ernie` |
| **Target Branch** | `main` |
| **Created** | 2025-09-11 |
| **Updated** | 2025-10-26 |
| **Merged** | 2025-09-11 |
| **Assignees** | `firecoperana` |

---

## 📄 Description

Hopefully this fixed the following error:

`Traceback (most recent call last):
  File "ik_llama.cpp/convert_hf_to_gguf.py", line 2198, in <module>
    @ModelBase.register("Ernie4_5_ForCausalLM", "Ernie4_5ForCausalLM")
NameError: name 'ModelBase' is not defined`

---

## 💬 Conversation

👤 **ikawrakow** approved this pull request ✅ on **2025-09-11** at **05:59:16**