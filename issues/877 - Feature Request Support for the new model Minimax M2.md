## 📌 [Issue #877](https://github.com/ikawrakow/ik_llama.cpp/issues/877) - Feature Request: Support for the new model Minimax M2

| **Author** | `hksdpc255` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-10-29 |
| **Updated** | 2025-11-06 |
| **Labels** | `enhancement` |

---

## 📄 Description

### Feature Description

Minimax M2 is a new competitor to GLM 4.6, with a strong focus on agent-related capabilities, and It seems a MOE-based model.

### Motivation

Maybe the MoE optimization implemented in ik_llama.cpp could provide better performance than the upstream llama.cpp?

### Possible Implementation

There’s already a pending upstream PR in llama.cpp: [[#16831](https://github.com/ikawrakow/ik_llama.cpp/issues/16831)](https://github.com/ggml-org/llama.cpp/pull/16831)

Would it be feasible to add support by merging that PR and adjusting any conflicts?

---

## 💬 Conversation

👤 **hksdpc255** commented on **2025-10-29** at **06:30:25**

## Another implementation

https://github.com/cturan/llama.cpp/compare/ggml-org%3Allama.cpp%3Amaster...minimax

https://github.com/ggml-org/llama.cpp/commit/54ed0123a687f5947a85b843472448675f888316

---

👤 **magikRUKKOLA** commented on **2025-10-30** at **21:58:19**

**NOISE started**

I never thought that just testing the LLMs etc. is a full-time job lol.

The amount of stuff the China releases is just ridiculous.  I am, as a regular vibe-coder, just can't keep up.  May be we can already figure out what are the best models for coding and fix all the bugs regarding the tool calls etc.?

Like, for example, this issue: https://github.com/ggml-org/llama.cpp/issues/16420

Unfortunately, somehow, I just don't have a time to figure this shit up. :/  And I don't know how to solve it.

The main question is that -- are you sure the adding of the new models is more important than figuring out how to run the older models properly?

---

👤 **magikRUKKOLA** commented on **2025-10-30** at **22:06:50**

I mean like, the collaborator from the mainline indeed confirmed that the issue exists.

> Yeah, as I said, I'm aware of this, but the Qwen3 Next conversion is proving to be extremely time consuming, to say the least. [...]

But as we can see, that specific collaborator chose to prioritise the new LLMs instead of fixing the bugs in the older ones.

I just don't get it.

Why to add newer models while your code is a pile of shit?  (I mean, the tool calls and streaming ... its just ... a pile of fucking shit!  This is absolutely ridiculous!  Like, WTF?!).

But every time I am voicing such issues I am having a response like -- don't worry too much, its kinda working already.

I just don't get it.  The fixup for the streaming tool calls is really important or I am imagining things?

---

👤 **firecoperana** commented on **2025-10-31** at **14:59:55**

I can port it after it's merged there. @hksdpc255 I want to port your GLM4.5 tool call support PR in mainline here.  Do you think it's ready?

---

👤 **hksdpc255** commented on **2025-11-01** at **01:41:34**

> I want to port your GLM4.5 tool call support PR in mainline here.

@firecoperana This implementation currently fails the unit tests. I’m working on a more robust solution that supports most XML-style tool calls (including Minimax M2!).

You’re welcome to port it, or I can handle the port myself once it’s ready.

---

👤 **hksdpc255** commented on **2025-11-03** at **02:51:37**

I’ve got tool calls working on llama.cpp. Let’s do some testing before porting it over to ik_llama.cpp.