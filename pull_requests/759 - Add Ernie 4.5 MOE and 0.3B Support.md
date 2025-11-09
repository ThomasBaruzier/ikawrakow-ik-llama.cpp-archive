## 🔀 [Pull Request #759](https://github.com/ikawrakow/ik_llama.cpp/pull/759) - Add Ernie 4.5 MOE and 0.3B Support

| **Author** | `firecoperana` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `fcp/ernie4.5_moe` |
| **Target Branch** | `main` |
| **Created** | 2025-09-04 |
| **Updated** | 2025-10-26 |
| **Merged** | 2025-09-05 |
| **Assignees** | `firecoperana` |

---

## 📄 Description

This is the port of Ernie 4.5 MoE model and Ernie 4.5 0.3B
In my test, response from the model looks ok. 

Didn't test the python convert script. It's a straight copy/paste. 

- [x] I have read the [contributing guidelines](https://github.com/ggerganov/llama.cpp/blob/master/CONTRIBUTING.md)
- Self-reported review complexity:
  - [ ] Low
  - [x] Medium
  - [ ] High

---

## 💬 Conversation

👤 **ikawrakow** approved this pull request ✅ on **2025-09-05** at **06:31:10**

---

👤 **Ph0rk0z** commented on **2025-09-05** at **19:56:07**

This works for big ernie too?

---

👤 **firecoperana** commented on **2025-09-05** at **19:59:48**

Yes.

---

👤 **AesSedai** commented on **2025-09-11** at **01:00:49**

The python convert script is broken, because of the following error:
```
Traceback (most recent call last):
  File "/home/jarvis/development/ik_llama.cpp/convert_hf_to_gguf.py", line 2198, in <module>
    @ModelBase.register("Ernie4_5_ForCausalLM", "Ernie4_5ForCausalLM")
NameError: name 'ModelBase' is not defined
```

The mainline may use `ModelBase`, but in ik_llama it's just `@Model.register`

Edit: also, `TextModel` should just be `Model` in `class Ernie4_5Model(Model):`

---

👤 **firecoperana** commented on **2025-09-11** at **03:04:04**

Can you verify if https://github.com/ikawrakow/ik_llama.cpp/pull/774 fix the error?

---

👤 **AesSedai** commented on **2025-09-11** at **04:42:45**

@firecoperana confirmed that PR un-breaks the convert script. I haven't tried an ERNIE convert, I was doing a GLM-4.5 quant and just noticed that the script was totally busted, that PR fixes the issue I was seeing.