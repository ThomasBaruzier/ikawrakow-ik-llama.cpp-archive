## 🗣️ [Discussion #869](https://github.com/ikawrakow/ik_llama.cpp/discussions/869) - Has anyone found a solution to the wrong parsing from GPT-OSS models ? [#16779](https://github.com/ikawrakow/ik_llama.cpp/issues/16779)

| **Author** | `Alekyuk` |
| :--- | :--- |
| **State** | ✅ **Open** |
| **Created** | 2025-10-26 |
| **Updated** | 2025-10-30 |

---

## 📄 Description

Hi everyone.

I tried many, many things but can't seem to find a way to use GPT-OSS-120b (model originally downloaded with LM Studio) so that it stops outputting the harmony tags like <|channel|>.

I use it with Open WebUI and Codex and the problem happens with the two.
I tried everything that I read on this pull request : [https://github.com/ikawrakow/ik_llama.cpp/pull/723](url)

Here is the command that I used :

> llama-server \
>     --model ~/.lmstudio/models/lmstudio-community/gpt-oss-120b-GGUF/gpt-oss-120b-MXFP4-00001-of-00002.gguf \
>     --no-warmup \
>     --host 0.0.0.0 \
>     --port 8080 \
>     --ctx-size 10192 \
>     --threads 8 \
>     --threads-batch 40 \
>     --batch-size 512 \
>     --ubatch-size 128 \
>     --gpu-layers 200 \
>     --n-cpu-moe 27 \
>     --parallel 2 \
>     --reasoning-format auto \
>     --temp 0.4 \
>     --top-k 0 \
>     --top-p 1.0 \
>     --repeat-penalty 1.1 \
>     --presence-penalty 0.4 \
>     --jinja \
>     --alias "GPT-OSS-120b"

Example of the issue :
`<|channel|>analysis
<|channel|>analysis<|message|>The user just says "Hey". Likely a greeting. Should respond accordingly. Keep tone friendly. Possibly ask how can help.<|end|><|start|>assistant<|channel|>finalHello! How can I assist you today?`

version :
`build/bin/llama-server --version
version: 3928 (16f30fcf3)
built with cc (Ubuntu 11.4.0-1ubuntu1~22.04.2) 11.4.0 for x86_64-linux-gnu`

I opened this discussion and not an issue as I have seen that other issues are already opened but not primarily for this model. I hope that I don't make any mistake here, this is my first time posting on a GitHub repo.

Thank you for your help.

---

## 💬 Discussion

👤 **firecoperana** commented on **2025-10-27** at **21:33:02**

Have you updated Open WebUI to the latest build? If it works with built in webui, but not Open Webui, then Open Webui does not parse it correctly. Open an issue there.

> 👤 **Alekyuk** replied on **2025-10-27** at **22:22:39**
> 
> HI,
> 
> Thank you for your reply, I do not see this issue only with Open WebUI. It happens everywhere (Codex, IK_llama UI, Roo Code ...).

> 👤 **firecoperana** replied on **2025-10-28** at **02:52:07**
> 
> Can you update to the latest main? I don't see such issue. Also make sure you have  `\` at the end of each line.

---

👤 **edyapd** commented on **2025-10-28** at **14:06:48**

This is actually an Open WebUI issue, not an ik-llama one.
It’s very easy to fix:
You need to click “Workspace” -> “Models” tab -> “+ New Model”

1. Give the model a name
2. Select the base model
3. Open “Advanced settings”
4. Set “Reasoning Tags”
Start tag:  <|channel|>analysis<|message|>
End tag:  <|end|>

5. Click “Save & Create”

For future runs, select this model to use.

---

👤 **magikRUKKOLA** commented on **2025-10-30** at **22:21:41**

**NOISE STARTED**

Hey guys, looking simply at the objective history as it is, what do we have?

The Elon Musk initially financed the OpenAI with a premise that everything they do will remain opensource!  AFAIK, it was about 30M USD.  And we ended up with what?  With a puny 120B parameter openweights model from OpenAI???  I mean, even their latest GPT5 with their "smart" router is an absolute garbage.  And after all of this you're chosing to discuss what is the most proper way to run OSS?

Eh...  Am I getting too old?  WTF is going on here?  Can anyone explain?