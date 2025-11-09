## 📌 [Issue #780](https://github.com/ikawrakow/ik_llama.cpp/issues/780) - Bug: Get a lot of “�” characters next to emoji now in responses.

| **Author** | `Ph0rk0z` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-09-14 |
| **Updated** | 2025-10-13 |

---

## 📄 Description

### What happened?

Since all the tool calling, I fired up qwen and a significant amount of “�” are in replies. Usually before or after an emoji. The front end hasn't changed, my browser hasn't changed. I think those were major mods to the server and something happened to the unicode handling. Anyone else?

Also everything dumps to console by default, as if I added the verbose flag. Could be related to not setting a "RELEASE" flag, but I don't do that on mainline. While it's nice to see your prompt as the backend gets it, the updates literally every token are a bit much.

### Name and Version

head

### What operating system are you seeing the problem on?

Linux

### Relevant log output

```shell

```

---

## 💬 Conversation

👤 **Ph0rk0z** commented on **2025-09-14** at **13:20:41**

Ok, I found the flag to stop compiling the server with debug. But I rolled back a few commits and the unicode issue remains. Did mainline change something in how unicode is processed that frontends updated?

---

👤 **sousekd** commented on **2025-09-14** at **21:05:57**

Just want to confirm I noticed issues with some non-English characters too, on recent bulds. In the past, non-English characters like "Ř", "Ň", "Ž" etc. worked. Not anymore.

---

👤 **Panchovix** commented on **2025-09-15** at **22:52:56**

Can confirm this issue as well, though I was doing something wrong lol.

---

👤 **ikawrakow** commented on **2025-09-23** at **07:13:10**

Someone knows what is the last good version before this issue started showing up?

---

👤 **loge-gh** commented on **2025-09-23** at **14:21:22**

@ikawrakow It looks like the last version without this bug was b66cecca4525550fddf609bb875e58a4fad5f0ce, the one just before the tool call support introduction. 0f9ecaec04cf40dd7524fdc6625f43c13b8038f8 was already buggy.

---

👤 **ikawrakow** commented on **2025-09-23** at **14:30:16**

Thanks!

@firecoperana  Do you have time to look into this?

---

👤 **firecoperana** commented on **2025-09-23** at **16:20:49**

I've never had this issue. @Ph0rk0z Can you post your start up command and the prompt you use? Also what front end do you use?

---

👤 **firecoperana** commented on **2025-09-23** at **16:41:33**

https://github.com/ggml-org/llama.cpp/issues/15607 Is the issue similar to this?

---

👤 **Ph0rk0z** commented on **2025-09-24** at **12:13:15**

That bug report is on windows and my build is set to release. It's on any prompt. I simply run llama-server the way I have previously and use it through sillytavern. Nothing crashes, just see the unknown unicode symbol mixed near emojis. I can try a debug build and see if it asserts. No tool calling is used and happens in text completion as well.

My solution with this so far has just been to apply a regex to delete those characters.

---

👤 **Panchovix** commented on **2025-09-24** at **12:15:53**

I get the issue on Linux - Fedora 42, and basically it replaces 90% emojis with the symbol, or puts the symbol alongside an emoji.

---

👤 **firecoperana** commented on **2025-09-24** at **22:12:40**

I couldn't reproduce this on linux either. Could be a system setting issue. When you type `locale` on Linux, do they show `UTF-8`?

---

👤 **Panchovix** commented on **2025-09-24** at **22:14:37**

Yes I think

```
locale
LANG=en_US.UTF-8
LC_CTYPE="en_US.UTF-8"
LC_NUMERIC="en_US.UTF-8"
LC_TIME="en_US.UTF-8"
LC_COLLATE="en_US.UTF-8"
LC_MONETARY="en_US.UTF-8"
LC_MESSAGES="en_US.UTF-8"
LC_PAPER="en_US.UTF-8"
LC_NAME="en_US.UTF-8"
LC_ADDRESS="en_US.UTF-8"
LC_TELEPHONE="en_US.UTF-8"
LC_MEASUREMENT="en_US.UTF-8"
LC_IDEN
```

Now I have got the issue only on DeepSeek based models, maybe that is related? Not sure what model is @Ph0rk0z using though.

I often use DeepSeek (V3-0324, R1 0528 and 3.1) and GLM 4.5 358B, but since the latter fits fully on GPU I use llama.cpp there so I haven't tested it, as it seems to be faster on full CUDA for some reason, but the former is faster when offloading on ik llamacpp.

---

👤 **firecoperana** commented on **2025-09-24** at **22:41:46**

@Panchovix  How often does it happen? If you use the browser on your phone to access llama-server, does it still have the issue? Trying to understand if it's the encoding/language issue in the front end.

---

👤 **Panchovix** commented on **2025-09-24** at **23:31:32**

@firecoperana it happens most of the time, I access via my PC on a browser.

These are some examples with DeepSeek 3.1 Terminus.

<img width="1934" height="454" alt="Image" src="https://github.com/user-attachments/assets/f7ba2157-c8e4-41a2-adf4-e615dc36e871" />

<img width="1911" height="544" alt="Image" src="https://github.com/user-attachments/assets/29e959ae-ee97-4581-adff-0ebafbfee2fe" />

SillyTavern is on English, despite saying the date in Spanish, for some reason.

---

👤 **firecoperana** commented on **2025-09-25** at **00:42:41**

How about using built in webui? Which endpoint does sillytavern use?

---

👤 **Panchovix** commented on **2025-09-25** at **00:56:05**

I'm using it like this.

<img width="1170" height="629" alt="Image" src="https://github.com/user-attachments/assets/3ad801e5-7e98-4cae-811d-dcf12a6cfda8" />

I went to sleep now so tomorrow I can try with the built in webui, sorry but I'm too sleepy :(

---

👤 **Ph0rk0z** commented on **2025-09-25** at **14:28:34**

It happened to me on qwen-235b. Other back ends don't have this issue. I see the symbols in the console too. I didn't check verbose output from the server and what they look like there. Might be the next step.

---

👤 **Panchovix** commented on **2025-09-26** at **16:57:50**

Sorry for late test. Seems built in webui works fine.

<img width="2782" height="755" alt="Image" src="https://github.com/user-attachments/assets/535a6f22-c792-4f6e-87ab-013ee88000ab" />

---

👤 **firecoperana** commented on **2025-09-26** at **17:15:02**

So it looks like a font issue. Maybe you need to reinstall the font for emoji that SillyTavern uses on Linux

---

👤 **Ph0rk0z** commented on **2025-09-26** at **18:46:16**

If it's a font issue, why now? Problem isn't present in exllama/tg-webui/mainline/openrouter, etc.

---

👤 **Panchovix** commented on **2025-09-26** at **18:58:03**

Yeah that's the thing, I tested llamacpp, exl3 (tabbyapi) and vLLM on ST, and emojis work fine. So not sure where the issue relies.

---

👤 **ikawrakow** commented on **2025-09-27** at **06:44:06**

A truncated multi-byte utf8 code point will also show up as � .The issue is that I don't see where/how the truncation happens.  Hence, it would be useful to have the tokens generated by the model, the text that was generated by detokenizing the tokens, and what ends up being sent to the UI after all the white space stripping and tool call processing.

---

👤 **loge-gh** commented on **2025-09-28** at **01:34:38**

I did some tests on llama-server (commit 3d4977c) and Sillytavern and here are my findings:
1) The problem appears in text completion mode when streaming is enabled.
2) It does not appear in chat completion mode.
3) It does not occur in text completion mode when streaming is disabled.
4) The generated text in llama.log (with verbose logging enabled) shows no anomalies.

It may be an incompatibility between ik_llama.cpp and Sillytavern, but it was introduced on ik_llama.cpp side with the commit 0f9ecae.

Example chat with Qwen3-235b:
`./llama-server -m ~/models/Qwen3-235B-A22B-Instruct-2507-UD-Q4_K_XL-00001-of-00003.gguf `

User: 
```
Расскажи сказку про розовые щечки
```
AI: 
```
Конечно! Вот добрая сказка про розовые щёчки:
---
**Сказка про розовые�ёчки**
Давным-давно
```
And here is sillytavern UI completion response data (excerpt) from browser, tokens around the buggy one - `овые�ё`:

data: {"content":"овые","stop":false,"id_slot":0,"multimodal":false,"completion_probabilities":[{"content":"овые","probs":[{"tok_str":"овые","prob":0.9999388456344604},{"tok_str":"ов","prob":5.945212615188211e-05},{"tok_str":"овый","prob":9.528305326966802e-07},{"tok_str":"овое","prob":3.460784228082048e-07},{"tok_str":"овых","prob":3.012355591636151e-07},{"tok_str":"ово","prob":2.898654827276914e-08},{"tok_str":"евые","prob":2.8666772067253987e-08},{"tok_str":"очные","prob":2.6714866763200007e-08},{"tok_str":"ные","prob":2.223476336382646e-08},{"tok_str":"о","prob":1.4953361215930272e-08}]}]}

data: {"content":"�","stop":false,"id_slot":0,"multimodal":false,"completion_probabilities":[{"content":"byte: \\x89","probs":[{"tok_str":"byte: \\x89","prob":0.9999974966049194},{"tok_str":"byte: \\x91","prob":1.7004509800244705e-06},{"tok_str":"byte: \\xa9","prob":3.074358119192766e-07},{"tok_str":"byte: \\x8e","prob":2.5964152428059606e-07},{"tok_str":"byte: \\xb9","prob":2.0727964056277415e-07},{"tok_str":"byte: \\xb1","prob":5.959130788824041e-08},{"tok_str":"byte: \\xb0","prob":1.2412594507793528e-08},{"tok_str":"byte: \\x9d","prob":1.198386456735534e-08},{"tok_str":"byte: \\xb2","prob":1.1074810402078583e-08},{"tok_str":"byte: \\x97","prob":1.0341875800179423e-08}]}]}

data: {"content":"ё","stop":false,"id_slot":0,"multimodal":false,"completion_probabilities":[{"content":"ё","probs":[{"tok_str":"ё","prob":0.9868274927139282},{"tok_str":"еч","prob":0.013123268261551857},{"tok_str":"ек","prob":4.425174120115116e-05},{"tok_str":"ë","prob":1.536725335427036e-06},{"tok_str":"ечно","prob":6.312461096058541e-07},{"tok_str":"Ё","prob":4.6330643499459256e-07},{"tok_str":"ѐ","prob":4.3093902490909386e-07},{"tok_str":"ед","prob":1.5523380625381833e-07},{"tok_str":"ёр","prob":1.1750338302363161e-07},{"tok_str":"ел","prob":1.0112514559068586e-07}]}]}


When I remove the "broken" part of the response and force SillyTavern to complete from just before the problematic token, there is no issue:
```
Конечно! Вот добрая сказка про розовые щёчки:
---
**Сказка про роз
```
completes with
```
овые щёчки**
Давным-давно
```
and the completion response looks like

data: {"content":"овые","stop":false,"id_slot":0,"multimodal":false,"completion_probabilities":[{"content":"овые","probs":[{"tok_str":"овые","prob":0.9999020099639893},{"tok_str":"ов","prob":9.359783143736422e-05},{"tok_str":"овый","prob":2.854701961041428e-06},{"tok_str":"овых","prob":8.126268653541047e-07},{"tok_str":"овое","prob":4.646468880764587e-07},{"tok_str":"ово","prob":7.917938660284563e-08},{"tok_str":"о","prob":6.529162988044845e-08},{"tok_str":"евые","prob":5.1289159586076494e-08},{"tok_str":"очные","prob":4.552504151433823e-08},{"tok_str":"ные","prob":4.3010182082525716e-08}]}]}

data: {"content":" щ","stop":false,"id_slot":0,"multimodal":false,"completion_probabilities":[{"content":"byte: \\x89","probs":[{"tok_str":"byte: \\x89","prob":0.9999977350234985},{"tok_str":"byte: \\xa9","prob":7.832657615836069e-07},{"tok_str":"byte: \\x91","prob":5.592763727690908e-07},{"tok_str":"byte: \\x8e","prob":3.682720546294149e-07},{"tok_str":"byte: \\xb9","prob":3.668978649784549e-07},{"tok_str":"byte: \\xb1","prob":1.3405542631517164e-07},{"tok_str":"byte: \\xb0","prob":5.524399426803939e-08},{"tok_str":"byte: \\xb2","prob":1.1750038275692987e-08},{"tok_str":"byte: \\xb3","prob":1.1400220323309895e-08},{"tok_str":"byte: \\x9d","prob":1.0797279514918046e-08}]}]}

data: {"content":"ё","stop":false,"id_slot":0,"multimodal":false,"completion_probabilities":[{"content":"ё","probs":[{"tok_str":"ё","prob":0.9896194338798523},{"tok_str":"еч","prob":0.010317721404135227},{"tok_str":"ек","prob":5.5728502047713846e-05},{"tok_str":"ë","prob":2.593350245660986e-06},{"tok_str":"ѐ","prob":7.506666293011222e-07},{"tok_str":"ечно","prob":5.893837169423932e-07},{"tok_str":"Ё","prob":5.170846293367504e-07},{"tok_str":"ед","prob":2.4183046321013535e-07},{"tok_str":"e","prob":1.480078566373777e-07},{"tok_str":"ёт","prob":1.401565441483399e-07}]}]}


Chat completion in streaming mode:
User:
```
Расскажи сказку про розовые щечки, которая начинается со **Сказка про розовые
```
AI:
```
**Сказка про розовые щёчки**
Сказка про розовые щёчки начинается
```

response data:
data: {"choices":[{"finish_reason":null,"index":0,"delta":{"content":"овые"},"logprobs":{"content":[{"id":129377,"token":"овые","bytes":[208,190,208,178,209,139,208,181],"logprob":-1.78815535036847e-05,"top_logprobs":[{"id":129377,"token":"овые","bytes":[208,190,208,178,209,139,208,181],"logprob":-1.78815535036847e-05},{"id":6715,"token":"ов","bytes":[208,190,208,178],"logprob":-11.392739295959473},{"id":138319,"token":"евые","bytes":[208,181,208,178,209,139,208,181],"logprob":-13.132184028625488},{"id":135882,"token":"очные","bytes":[208,190,209,135,208,189,209,139,208,181],"logprob":-14.002530097961426},{"id":42965,"token":"ные","bytes":[208,189,209,139,208,181],"logprob":-14.324105262756348}]}]}}],"created":1759023682,"id":"chatcmpl-MYkLnTREwtUWGJxZnUlNHMwnmZmynREK","model":"","object":"chat.completion.chunk","usage":{"completion_tokens":8,"prompt_tokens":59,"total_tokens":67}}

data: {"choices":[{"finish_reason":null,"index":0,"delta":{"content":" щ"},"logprobs":{"content":[{"id":231,"token":" щ","bytes":[32,209,137],"logprob":-6.9141628955549095e-06,"top_logprobs":[{"id":231,"token":"�","bytes":[137],"logprob":-6.9141628955549095e-06},{"id":102,"token":"�","bytes":[169],"logprob":-12.885977745056152},{"id":236,"token":"�","bytes":[142],"logprob":-13.736960411071777},{"id":239,"token":"�","bytes":[145],"logprob":-13.885497093200684},{"id":245,"token":"�","bytes":[151],"logprob":-15.30589771270752}]}]}}],"created":1759023683,"id":"chatcmpl-MYkLnTREwtUWGJxZnUlNHMwnmZmynREK","model":"","object":"chat.completion.chunk","usage":{"completion_tokens":10,"prompt_tokens":59,"total_tokens":69}}

data: {"choices":[{"finish_reason":null,"index":0,"delta":{"content":"ё"},"logprobs":{"content":[{"id":44022,"token":"ё","bytes":[209,145],"logprob":-0.39498597383499146,"top_logprobs":[{"id":44022,"token":"ё","bytes":[209,145],"logprob":-0.39498597383499146},{"id":55757,"token":"еч","bytes":[208,181,209,135],"logprob":-1.1202590465545654},{"id":14949,"token":"ек","bytes":[208,181,208,186],"logprob":-9.130337715148926},{"id":12121,"token":"ел","bytes":[208,181,208,187],"logprob":-13.231945991516113},{"id":13103,"token":"ед","bytes":[208,181,208,180],"logprob":-13.925959587097168}]}]}}],"created":1759023683,"id":"chatcmpl-MYkLnTREwtUWGJxZnUlNHMwnmZmynREK","model":"","object":"chat.completion.chunk","usage":{"completion_tokens":11,"prompt_tokens":59,"total_tokens":70}}



And here's the completion response from an old llama-server version, commit b66cecc (streaming response,  generated anew):
```
Конечно! Вот добрая сказка про розовые щёчки:
---
**Сказка про розовые щёчки**
Давным-давно
```

and response data:

data: {"content":"овые","stop":false,"id_slot":0,"multimodal":false,"oaicompat_msg_diffs":[{"content_delta":"овые"}],"completion_probabilities":[{"content":"овые","probs":[{"tok_str":"овые","prob":1.0},{"tok_str":"ов","prob":0.0},{"tok_str":"овый","prob":0.0},{"tok_str":"овое","prob":0.0},{"tok_str":"овых","prob":0.0},{"tok_str":"очные","prob":0.0},{"tok_str":"евые","prob":0.0},{"tok_str":"ово","prob":0.0},{"tok_str":"ные","prob":0.0},{"tok_str":"о","prob":0.0}]}]}

data: {"content":" щ","stop":false,"id_slot":0,"multimodal":false,"oaicompat_msg_diffs":[{"content_delta":" щ"}],"completion_probabilities":[{"content":"byte: \\x89","probs":[{"tok_str":"byte: \\x89","prob":1.0},{"tok_str":"byte: \\x91","prob":0.0},{"tok_str":"byte: \\xa9","prob":0.0},{"tok_str":"byte: \\x8e","prob":0.0},{"tok_str":"byte: \\x8a","prob":0.0},{"tok_str":"byte: \\x9d","prob":0.0},{"tok_str":"byte: \\x97","prob":0.0},{"tok_str":"byte: \\xb9","prob":0.0},{"tok_str":"byte: \\x92","prob":0.0},{"tok_str":"byte: \\x94","prob":0.0}]}]}

data: {"content":"ё","stop":false,"id_slot":0,"multimodal":false,"oaicompat_msg_diffs":[{"content_delta":"ё"}],"completion_probabilities":[{"content":"ё","probs":[{"tok_str":"ё","prob":1.0},{"tok_str":"еч","prob":0.0},{"tok_str":"ек","prob":0.0},{"tok_str":"ë","prob":0.0},{"tok_str":"ечно","prob":0.0},{"tok_str":"Ё","prob":0.0},{"tok_str":"ѐ","prob":0.0},{"tok_str":"ед","prob":0.0},{"tok_str":"лё","prob":0.0},{"tok_str":"ёр","prob":0.0}]}]}


Vanilla llamacpp (b6488) streaming response data:
```
Конечно! Вот добрая сказка про розовые щёчки:
---
**Сказка про розовые щёчки**
Давным-давно
```

data: {"index":0,"content":"овые","tokens":[129377],"stop":false,"id_slot":-1,"tokens_predicted":27,"tokens_evaluated":31,"completion_probabilities":[{"id":129377,"token":"овые","bytes":[208,190,208,178,209,139,208,181],"logprob":-5.9367986978031695e-05,"top_logprobs":[{"id":129377,"token":"овые","bytes":[208,190,208,178,209,139,208,181],"logprob":-5.9367986978031695e-05},{"id":6715,"token":"ов","bytes":[208,190,208,178],"logprob":-9.755678176879883},{"id":130558,"token":"овый","bytes":[208,190,208,178,209,139,208,185],"logprob":-13.806585311889648},{"id":135885,"token":"овое","bytes":[208,190,208,178,208,190,208,181],"logprob":-15.119670867919922},{"id":129526,"token":"овых","bytes":[208,190,208,178,209,139,209,133],"logprob":-15.189775466918945},{"id":138319,"token":"евые","bytes":[208,181,208,178,209,139,208,181],"logprob":-17.453325271606445},{"id":135882,"token":"очные","bytes":[208,190,209,135,208,189,209,139,208,181],"logprob":-17.466766357421875},{"id":127015,"token":"ово","bytes":[208,190,208,178,208,190],"logprob":-17.636423110961914},{"id":42965,"token":"ные","bytes":[208,189,209,139,208,181],"logprob":-17.84261131286621},{"id":1456,"token":"о","bytes":[208,190],"logprob":-18.039216995239258}]}]}

data: {"index":0,"content":" щ","tokens":[231],"stop":false,"id_slot":-1,"tokens_predicted":29,"tokens_evaluated":31,"completion_probabilities":[{"id":231,"token":" щ","bytes":[32,209,137],"logprob":-1.5497220147153712e-06,"top_logprobs":[{"id":231,"token":"�","bytes":[137],"logprob":-1.5497220147153712e-06},{"id":239,"token":"�","bytes":[145],"logprob":-13.841035842895508},{"id":236,"token":"�","bytes":[142],"logprob":-15.026357650756836},{"id":102,"token":"�","bytes":[169],"logprob":-15.355897903442383},{"id":117,"token":"�","bytes":[185],"logprob":-16.00758934020996},{"id":109,"token":"�","bytes":[177],"logprob":-17.306081771850586},{"id":108,"token":"�","bytes":[176],"logprob":-18.334854125976563},{"id":245,"token":"�","bytes":[151],"logprob":-18.70400047302246},{"id":240,"token":"�","bytes":[146],"logprob":-18.757427215576172},{"id":110,"token":"�","bytes":[178],"logprob":-18.791486740112305}]}]}

data: {"index":0,"content":"ё","tokens":[44022],"stop":false,"id_slot":-1,"tokens_predicted":30,"tokens_evaluated":31,"completion_probabilities":[{"id":44022,"token":"ё","bytes":[209,145],"logprob":-0.014102671295404434,"top_logprobs":[{"id":44022,"token":"ё","bytes":[209,145],"logprob":-0.014102671295404434},{"id":55757,"token":"еч","bytes":[208,181,209,135],"logprob":-4.2705397605896},{"id":14949,"token":"ек","bytes":[208,181,208,186],"logprob":-10.558671951293945},{"id":12179,"token":"ë","bytes":[195,171],"logprob":-13.601837158203125},{"id":127537,"token":"ечно","bytes":[208,181,209,135,208,189,208,190],"logprob":-14.19041633605957},{"id":145911,"token":"ѐ","bytes":[209,144],"logprob":-14.728206634521484},{"id":144323,"token":"Ё","bytes":[208,129],"logprob":-14.871210098266602},{"id":13103,"token":"ед","bytes":[208,181,208,180],"logprob":-15.939306259155273},{"id":138408,"token":"лё","bytes":[208,187,209,145],"logprob":-16.280132293701172},{"id":136124,"token":"ёр","bytes":[209,145,209,128],"logprob":-16.303104400634766}]}]}

(edited: chat completion tests are done on wrong version)

---

👤 **firecoperana** commented on **2025-09-28** at **16:31:34**

@loge-gh Can you create a PR to fix it now that you have tracked down the issue to be streaming mode for text completions? I don't have time to look into this for now. But if you don't mind waiting, I can fix it later.

---

👤 **Ph0rk0z** commented on **2025-09-30** at **22:46:22**

Your PR went against the wrong repo.

---

👤 **loge-gh** commented on **2025-09-30** at **23:54:49**

> Your PR went against the wrong repo.

(facepalm) Thanks. Looks like I need to sleep more often... )