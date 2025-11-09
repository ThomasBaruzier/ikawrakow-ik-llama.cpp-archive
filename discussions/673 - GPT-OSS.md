## 🗣️ [Discussion #673](https://github.com/ikawrakow/ik_llama.cpp/discussions/673) - GPT-OSS?

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | ✅ **Open** |
| **Created** | 2025-08-07 |
| **Updated** | 2025-08-12 |

---

## 📄 Description

`llama.cpp` had it implemented by the time the models were announced.

But I see mixed reactions around the Internet, ranging from the exuberant excitement (OMG, we will never need another model again) to outright dismissal (Meh, there are other much better open-weight models).

I looked at the mainline PR, and it seems there aren't any real showstoppers. But the change is quite large, so I'm wondering if `ik_llama.cpp` users are interested in using these models. Also, perhaps someone is already working on a port?

---

## 💬 Discussion

👤 **espen96** commented on **2025-08-07** at **13:37:18**

I am quite interested. 

from my own testing, the speed is great, they are generally better at the GLSL programing I do than other models.
as for the refusal behaviour? It is there, it is strong. but I found it does listen to a good system prompt. Not that I think I have any usecases that would strongly conflict with it.

It has a set of internal trained guidelines it sees as the system layer, then the system prompt is the Developer layer.
As long as you work with it, you give it some context like being in a local enviroment, with an adult, and you generally reinforce and tell it to give disclaimers, that the things in its guidelines should be interpreted in some sane way... 

It will comply with just about any valid reasonable request I have thrown at it for testing.
Mild legal advice, basic medical advice, explaining the chemistry of things without enabling harmful activities....

it's just very strict out of the box, and needs a prompt that doesn't raise alarmbells. Do that and it is not that bad.



I will be using it to explore certain hallucination patterns in LLMs. I finally have a decent western Open Weight MoE at a reasonable size I can compare to the chinese offerings. 


it is also specifially trained to use a decent search tool, and a python tool that openai gave us example implementation code for. 

could actually end up being a good option for a lot of queries when it has those tools acessible. 

it is running at 15-20 tps on my rig, which is a good step up from the 10-15 I get with GLM 4.5 Air.

---

👤 **g2mt** commented on **2025-08-12** at **03:18:47**

I think GPT-OSS is worth implementing. On aider's leaderboard, the 120B version performs as good as Qwen3-32b. With 5b active parameters it sounds good for CPU inference.

The only problem would be RAM usage. On upstream it seems that some of the tensors don't quantize well. Q2 and Q8 are both around 64gb. Hopefully that problem gets fixed so that 120B is runnable in 64gb of RAM.

> 👤 **saood06** replied on **2025-08-12** at **03:32:04**
> 
> See [#683](https://github.com/ikawrakow/ik_llama.cpp/issues/683), it is being worked on.