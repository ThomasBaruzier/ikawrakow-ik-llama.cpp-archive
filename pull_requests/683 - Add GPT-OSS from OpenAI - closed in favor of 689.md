## 🔀 [Pull Request #683](https://github.com/ikawrakow/ik_llama.cpp/pull/683) - Add GPT-OSS from OpenAI - closed in favor of 689

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Source Branch** | `ik/gpt-oss` |
| **Target Branch** | `main` |
| **Created** | 2025-08-10 |
| **Updated** | 2025-08-15 |

---

## 📄 Description

This PR adds support for OpenAI's GPT-OSS models.

Metal and Vulkan are missing, but I want to still declare it ready for testing to get feedback on the main platforms `ik_llama.cpp` is being used.

I had to do a fairly significant refactoring to accommodate the required vocabulary changes, so please test with other models as well to make sure that I have not broken something.

In terms of performance, the PR is significantly faster than mainline CPU-only (see graphs below). On CUDA, I get ~40% better PP performance than mainline, but ~10% lower TG performance (RTX-4080, fully offloading the 20B variant).   

### Original description

Not ready, only CPU works (but there are issues with EOG tokens that need to be resolved).
The model uses biases for the MoE ops, so `-fmoe` is not functional (and if specified, it will be ignored).
The optimized CPU flash attention is also not used as I need to add attention sinks there. 

Still opening a draft PR for visibility.

Getting the same (very high) PPL as mainline `llama.cpp`.

Quick performance comparison on a Ryzen-7950X CPU using [the 20B MXFP4 model from ggml-org](https://huggingface.co/ggml-org/gpt-oss-20b-GGUF)

### ik_llama.cpp

| model                          |       size |     params | backend    | threads |          test |              t/s |
| ------------------------------ | ---------: | ---------: | ---------- | ------: | ------------: | ---------------: |
| gpt-oss ?B MXFP4 - 4.25 bpw    |  11.27 GiB |    20.91 B | CPU        |      16 |         pp512 |    444.12 ± 3.33 |
| gpt-oss ?B MXFP4 - 4.25 bpw    |  11.27 GiB |    20.91 B | CPU        |      16 |         tg128 |     21.24 ± 0.05 |

### llama.cpp

| model                          |       size |     params | backend    | threads |            test |                  t/s |
| ------------------------------ | ---------: | ---------: | ---------- | ------: | --------------: | -------------------: |
| gpt-oss ?B MXFP4 MoE           |  11.27 GiB |    20.91 B | CPU        |      16 |           pp512 |        141.89 ± 0.56 |
| gpt-oss ?B MXFP4 MoE           |  11.27 GiB |    20.91 B | CPU        |      16 |           tg128 |         21.29 ± 0.01 |

So, TG is about the same, PP is ~3X faster.

## Update 1

CUDA seems to be working now. Haven't added attention sinks to all FA implementations, so at this point FA will only work on Turing or newer.

It seems the mainline FA implementation is better for this model (head size is 64, a head size I haven't paid much attention to). Hence, PP is better than mainline up to a context of 8k tokens, but then falls behind. TG is about the same as mainline without FA, but is ~20% lower with FA, so clearly something is not quite right there. This model has a surprisingly large performance difference between FA and no FA even for small contexts. 

### Update 2

Added the ability to use `-fmoe` (CUDA only for now). With this PP performance is now ~40% percent better than mainline for short contexts, and still better at 16k tokens (I cannot go beyond that with my 16 GB GPU and the MXFP4 model from ggml-org). TG performance also improves but is still lower than than mainline. Hence I decided to test mainline without CUDA graphs, and, to my surprise, observe a ~10% lower performance (surprise because when I last tested the benefit of CUDA graphs in mainline on Linux it was very minor, a 2-3% sort of thing).

There is another very interesting observation. While working on `-fmoe` I had a bug, which caused a crash when trying to run `llama-bench` or `llama-sweep-bench`, but `perplexity` was working fine for the default context of 512. So, I decided to see if I'll get the crash when running `perplexity` with a larger context. I used a context of 4096, it still worked, and I got a PPL of over 2000. I checked with mainline and, lo and behold, the same PPL also there (2155!!! to be precise). Based on that, I'm now almost 100% sure the implementation in mainline is incorrect. There is [issue 15155](https://github.com/ggml-org/llama.cpp/issues/15155) in mainline discussing the high PPL, and concerned users are being told that a high perplexity is normal for instruction tuned models. Ha ha, seriously? A PPL of 2000 after looking at a context of 4000 English-only tokens means the model has absolutely no idea how to continue the given text. 

### Update 3

Added attention sinks to the optimized CPU flash attention implementation. Here is what we get with `sweep-bench` for the 20B GPT-OSS model with `Q8_0` KV cache running on a Ryzen-7950X CPU

Prompt processing:
 
<img width="792" height="612" alt="gpt_oss_cpu_pp" src="https://github.com/user-attachments/assets/3ee03289-c7e5-4ab5-a9df-7f5b7ef1ff85" />

Token generation:

<img width="792" height="612" alt="gpt_oss_cpu_tg" src="https://github.com/user-attachments/assets/0d22e17b-65ae-4d48-b994-9ff8626c9490" />

`ik_llama.cpp` PP is 3.4X faster at zero context and 5.5X faster for 8k tokens in the cache.
`ik_llama.cpp` TG is about the same at zero context and 19% faster for 8k tokens in the cache.
This is before having added `-fmoe` CPU implementation.

<details>
<summary>llama.cpp</summary>

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|   512 |    128 |      0 |    3.551 |   144.18 |    6.013 |    21.29 |
|   512 |    128 |    512 |    3.948 |   129.67 |    6.074 |    21.07 |
|   512 |    128 |   1024 |    4.272 |   119.85 |    6.255 |    20.47 |
|   512 |    128 |   1536 |    4.647 |   110.18 |    6.392 |    20.03 |
|   512 |    128 |   2048 |    5.055 |   101.30 |    6.559 |    19.51 |
|   512 |    128 |   2560 |    5.507 |    92.98 |    6.692 |    19.13 |
|   512 |    128 |   3072 |    5.781 |    88.57 |    6.867 |    18.64 |
|   512 |    128 |   3584 |    6.250 |    81.92 |    7.008 |    18.26 |
|   512 |    128 |   4096 |    6.608 |    77.48 |    7.170 |    17.85 |
|   512 |    128 |   4608 |    7.012 |    73.02 |    7.295 |    17.55 |
|   512 |    128 |   5120 |    7.546 |    67.85 |    7.581 |    16.89 |
|   512 |    128 |   5632 |    7.915 |    64.69 |    7.677 |    16.67 |
|   512 |    128 |   6144 |    8.364 |    61.22 |    7.857 |    16.29 |
|   512 |    128 |   6656 |    8.686 |    58.94 |    7.940 |    16.12 |
|   512 |    128 |   7168 |    9.058 |    56.53 |    8.205 |    15.60 |
|   512 |    128 |   7680 |    9.455 |    54.15 |    8.298 |    15.42 |
|   512 |    128 |   8192 |    9.790 |    52.30 |    8.421 |    15.20 |
</details>

<details>
<summary>ik_llama.cpp</summary>

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|   512 |    128 |      0 |    1.042 |   491.59 |    5.997 |    21.34 |
|   512 |    128 |    512 |    1.050 |   487.83 |    6.061 |    21.12 |
|   512 |    128 |   1024 |    1.100 |   465.48 |    6.188 |    20.69 |
|   512 |    128 |   1536 |    1.139 |   449.69 |    6.234 |    20.53 |
|   512 |    128 |   2048 |    1.177 |   435.18 |    6.260 |    20.45 |
|   512 |    128 |   2560 |    1.240 |   412.89 |    6.328 |    20.23 |
|   512 |    128 |   3072 |    1.270 |   403.24 |    6.389 |    20.03 |
|   512 |    128 |   3584 |    1.364 |   375.23 |    6.517 |    19.64 |
|   512 |    128 |   4096 |    1.360 |   376.47 |    6.522 |    19.62 |
|   512 |    128 |   4608 |    1.439 |   355.70 |    6.596 |    19.41 |
|   512 |    128 |   5120 |    1.473 |   347.51 |    6.722 |    19.04 |
|   512 |    128 |   5632 |    1.503 |   340.54 |    6.764 |    18.92 |
|   512 |    128 |   6144 |    1.553 |   329.62 |    6.863 |    18.65 |
|   512 |    128 |   6656 |    1.566 |   326.86 |    6.947 |    18.43 |
|   512 |    128 |   7168 |    1.614 |   317.23 |    6.992 |    18.31 |
|   512 |    128 |   7680 |    1.716 |   298.35 |    7.043 |    18.17 |
|   512 |    128 |   8192 |    1.790 |   286.05 |    7.058 |    18.13 |

</details>

---

## 💬 Conversation

👤 **espen96** commented on **2025-08-11** at **11:07:27**

I would take prompt processing wins any day. 

while messing with these models on mainline through open webui it has become clear they are extremely sensitive to any deviations from the Harmony template. Toolcalls can be really tricky if it is not already familiar with the tools, at least for the 120b variant.

I have a lot more notes here, I'll get more into the weeds as things progress. 
I am just letting you know ahead of time that these two models are very sensitive.

---

👤 **ikawrakow** commented on **2025-08-11** at **11:11:31**

> while messing with these models on mainline through open webui it has become clear they are axtremely sensitive to any deviations from the Harmony template.

Thanks for the heads up. It looks like there will be bigger changes required in `ik_llama.cpp` to accommodate Harmony, so I have left that for the end. If you have specific notes on what is important, I would appreciate.

---

👤 **espen96** commented on **2025-08-11** at **12:16:55**

for tools it was not trained for, or if they look familiar but not quite right, I have seen the model get a bit confused at times. 

It can be hard to decipher what the "caveman" simplified cot reasoning is hinting at at times.

a proper tool call looks like this:

`<|channel|>analysis<|message|>Need to use function get_weather.<|end|><|start|>assistant<|channel|>commentary to=functions.get_weather <|constrain|>json<|message|>{"location":"San Francisco"}<|call|>`

the tool result should then be given back in a new tool message a such:

`<|start|>functions.get_weather to=assistant<|channel|>commentary<|message|>{"sunny": true, "temperature": 20}<|end|>`

it has to be in this format. it expects there to be a tool call by it a one complete message, then the rsult must be in the new message. Before a few things got a bit better through some unmerged PR's and template fixes, I would see the model get confused  as it was waiting for the tool result to return. It knew it had the data, but it was waiting, and was generally confused. 

I was using a simple forecast tool, and it would remark on how it was waiting for the tool fucntion to return. somehow the tool call had commentary and probably an example result? but it was waiting for the tool to return. 

sometimes it would return the data properly to me, sometimes it would result in more toolcalls as it did not understand what was going on.

it appears that the way tools are registrered might leave it confused at times.
it can then forget to output "to=functions.get_weather " 


without OAI's own jinja template and some PR's that have helped a lot errors like this occur:
Note that the weather tools are enabled.

<img width="1087" height="600" alt="image" src="https://github.com/user-attachments/assets/5f92ba03-1abd-4bd8-ac01-402976341ce0" />

<img width="1038" height="421" alt="image" src="https://github.com/user-attachments/assets/79d4532f-68e0-440c-80b4-284b3371a3c6" />

<img width="1098" height="493" alt="image" src="https://github.com/user-attachments/assets/a4b77125-e8bd-4075-a0c9-6cb6dc48b5fe" />

<img width="1060" height="314" alt="image" src="https://github.com/user-attachments/assets/c8595be4-ddd1-40a9-a94c-03f76538de5a" />



with the jinja template by OAI:

<img width="1042" height="307" alt="image" src="https://github.com/user-attachments/assets/e034312c-f097-4c76-b627-b5d0023fe6bd" />



without the updated template, but with some other [fixes](https://github.com/ggml-org/llama.cpp/pull/15181):
<img width="1034" height="590" alt="image" src="https://github.com/user-attachments/assets/46367b75-dc5b-4c1c-80a9-2dd7d9345f11" />

<img width="1140" height="279" alt="image" src="https://github.com/user-attachments/assets/5bfe0972-2c2b-4a9a-8104-7b86e26a53c1" />
<img width="1106" height="215" alt="image" src="https://github.com/user-attachments/assets/5647e38c-530b-4514-b613-9b68c5faa9ce" />


here is something it said back when I first tried using the model:
```

<|channel|>analysis<|message|>The user says \"check the weather in nyc\". According to instructions, we should call a tool to get the current weather for NYC. We have tools: get_current_weather. We need to output a single raw JSON object, with arguments field as JSON-encoded string.\n\nThus, we should construct:\n\n{\"name\":\"functions.get_current_weather\",\"arguments\":\"{\\\"city\\\":\\\"NYC\\\"}\"}\n\nWait, the tool name: defined as functions.get_current_weather, or maybe \"get_current_weather\"? The example shows name \"calculator\" inside \"name\": \"calculator\". In this specification, it\\'s a \"function\" under namespace functions. So we need to refer to tool name exactly as provided. In the tool definitions, the function name is get_current_weather. So name should be \"functions.get_current_weather\" or just \"get_current_weather\"? That is ambiguous. In the example, they used \"calculator\" directly. In the tool definitions, it\\'s within \"class functions { get_current_weather(...)... }\" but maybe the tool name recognized by system is \"functions.get_current_weather\". But typical usage is inline \"name\": \"functions.get_current_weather\". However, the instructions say \"When calling a tool, you must produce a single, raw JSON object.\" Eg: {\"name\":\"calculator\",\"arguments\":\"{\\\"expression\\\":\\\"2 * (6 + 3)\\\"}\"}. So maybe we use \"get_current_weather\" as name. Hard to know. The system call will respond as soon as we produce. It might refer to \"get_current_weather\". Probably \"get_current_weather\". Because guidelines say \"call tool named `calculator` ...\". But in the list, there is no tool named \"calculator\". But \"Get the current weather for a given city.\" It\\'s in the \"functions\" namespace. Usually, the tool name would be just the function name. In the JSON usage in OpenAI tool calling, it\\'s \"name\":\"get_current_weather\". But maybe the prefix \"functions.\" is required? In the provided example of using get_current_weather, which includes the type, they didn\\'t give example usage. The typical usage in these prompts: we use name \"get_current_weather\". So I\\'ll use that.\n\nOk.\n\nThus output: {\"name\":\"get_current_weather\",\"arguments\":\"{\\\"city\\\":\\\"nyc\\\"}\"}\n\nThat meets guidelines.\n\nRemember to not add any other text.\n\nAlso we must adhere to the instructions: environment unique etc. So output exactly JSON.\n\nThus answer: output that JSON object.\n\n<|end|>

```



LM Studio's implementation seems to work. they however is bypassing jinja completely at all times for this model it seems, and added a library that can be updated separately to parse the template.  

that variant seems to work fine from my testing so far, but there's no source code as far as I can tell.

for debugging at least, I think it will be important to log what is happening on the Harmony level.



this is the PR over on mainline that for now is unmerged, but appears to fix the main bulk of the issues, and the official OAI jinja appears to help with the rest, or at least most of it.

https://github.com/ggml-org/llama.cpp/pull/15181



Edit: appears that the ggml-org variant has been updated with corrections to the template, as of... a day ago or so. 
That should help a bit.

Edit 2: it did not. I still get better results from using a specified jinja template

---

👤 **ikawrakow** commented on **2025-08-12** at **09:23:01**

The Wikitext2 perplexity (PPL) computed with the mainline `llama.cpp` implementation of the OpenAI GPT-OSS models is very high. "Very high" as in an order of magnitude or more higher than other instruction tuned models of similar size. The issue is being dismissed by mainline developers in [issue 15155](https://github.com/ggml-org/llama.cpp/issues/15155) with arguments such as
* It is an instruction tuned model, so it is normal to have a higher PPL
* Maybe the model desperately wants to output `<think>`, so other tokens have a low probability, so PPL becomes very high
* Maybe the model gets confused because Wikitext has lines such as `T I T L E`, so the presence of spaces causes very high PPL
* If we compute PPL on a few tokens request, it has a low PPL, so the model works correctly.

Well, let's compute PPL as a function of context length. We all know PPL **decreases** with context length (at least up to the original training context), instruction tuned or not (or strange spaces at unexpected locations or not). Here is what we get (using mainline `llama.cpp` pulled yesterday):
 
<img width="792" height="612" alt="ppl_gpt_oss" src="https://github.com/user-attachments/assets/8f7e4e08-6557-4219-ac33-d6cd1280d852" />

The black circles are for GPT-OSS-20B computed with mainline `llama.cpp`. The red circles are for DeepSeek-Lite-16B, a reasonably recent MoE model with a similar number of total and active parameters (16B/2.4B). The green circles are for GPT-OSS-20B computed with this PR but a slightly modified attention (more on that below). Note the logarithmic scales for the x- and y-axis.

As expected, the PPL of the DeepSeek model decreases with context length. We also see that PPL of the GPT-OSS-20B model decreases with context length up to 64 tokens, but then goes completely crazy.  Interestingly enough, the model has a head size of 64 and there are 64 attention heads, so I'm wondering if something went wrong with the MHA implementation/GGUF attention tensor preparation? But that's just a speculation. In any case, I wouldn't know why the model's desperation to output `<think>` would increase with context length. As a context for people not familiar with PPL values, a simple N-gram model will typically have `PPL < 1000`, while we observe a PPL of 5400 for a context of 8k tokens.  

For the green data points, I have turned on usage of [this FA implementation](https://github.com/ikawrakow/ik_llama.cpp/blob/ik/gpt-oss/ggml/src/ggml-cuda/fattn-new-mma.cu) in `ik_llama.cpp`. There is a bug there so that the row sums of `V*softmax(Q*K)` are computed **before** applying the attention sinks (bug in the sense that [this code block](https://github.com/ikawrakow/ik_llama.cpp/blob/5abc39481bdbbaae163f6a200b5d380a0ffb0d9d/ggml/src/ggml-cuda/fattn-new-mma.cu#L981) should come after the attention sinks block if I'm to believe the mainline implementation). This effectively turns off Evan Miller's [Attention is off by one](https://www.evanmiller.org/attention-is-off-by-one.html) proposition, which got implemented in the GPT-OSS models with modifications for the first time.  This does reduce PPL by nearly a factor of 2, but is nowhere near to solving the PPL increase with context length. I did a few quick tests with that, and did not notice any significant difference in response quality.

But given these results, it is not unlikely mainline's implementation is incorrect.

---

👤 **espen96** commented on **2025-08-12** at **11:20:01**

Given this appears to have been a rushjob with a lot of new technologies, I would not be surprised.  

These models do appear to be a bit odd at times. 
I have seen them think that they should "open up the 5th link" in a search, and struggle to output the ID tag after I messed with the way the browser tool works.

I am not familiar enough with perplexity measurements to know if that could be related 

Any way to test the perplexity of the other implementations?

---

👤 **ikawrakow** commented on **2025-08-12** at **11:27:37**

> Any way to test the perplexity of the other implementations?

You want to test perplexity with other inference frameworks such as vLLM? I'm not familiar with other tool kits, so perhaps someone else can chime in (or do the calculations)?

Before someone else says that perplexity values computed with other tool kits are not directly comparable to `llama.cpp/ik_llama.cpp`: while this is true, observed differences have been at the few percent level in the past. Here we are looking at factors/orders of magnitude, not percentages. So, someone computing PPL as a function of context length with another tool kit would be a very useful piece of information.

---

👤 **espen96** commented on **2025-08-12** at **11:42:08**

I will give it a shot. 

We are looking for major errors here. That graph you showed does not look right to me. I do not like how it goes up like that. 

I have had issues getting vllm to work in the past, but I will try to get whatever data I can

---

👤 **saood06** commented on **2025-08-12** at **11:54:46**

>Any way to test the perplexity of the other implementations?

Transformers has a doc about it [here](https://huggingface.co/docs/transformers/perplexity).

---

👤 **espen96** commented on **2025-08-12** at **19:13:56**

I implemented, with some help from Claude, what is probaly a mediocre variant of the perplexity measurement tool llama.cpp uses.

I would not vouch for its accuracy or the numbers, but hopefully it will do fine. @ikawrakow predicted, and demonstrated that IT models do just fine and follow this nice little graph. 
I had to use the new Qwen 3 4b rather than deepseek, but the trend is the same.

<img width="1500" height="900" alt="perplexity_plot_20250812_200701" src="https://github.com/user-attachments/assets/f3cc585f-5126-48a2-b9f3-bec49fb1e444" />



I think that is enough to demonstrate that my script isn't completely off the mark. 

<img width="1500" height="900" alt="perplexity_plot_20250812_210207" src="https://github.com/user-attachments/assets/818ce8b7-a6f8-4b39-bc08-bb0c7deecbf5" />

<img width="1610" height="1125" alt="image" src="https://github.com/user-attachments/assets/a65cbcb7-c3bf-4905-9a4c-0d3453abd516" />



next up I am adding support for using llama.cpp with this perplexity calculation script, to verify that this will largely replicate what @ikawrakow showed.

however, so far the vllm results show the trendline we want to see, I would not focus on the numbers as they might be inflated

---

👤 **espen96** commented on **2025-08-13** at **12:56:20**

So, first attempt at running it wih llama.cpp as the backend showed the same trendline as in vllm. However soem errors showed up during evaluation

the error was of the type `Failed to get logprobs for position <number>`


Context Length 2048:
Positions: `8, 9, 16, 18`
Context Length 4096:
Positions: `2, 8, 9, 14, 16, 18`
Context Length 8192:
Positions: `2, 3, 5, 6, 7, 8, 9, 10, 11, 13, 14, 15, 16, 17, 18, 19, 20`


So while we did not get to replicate and confirm the upwards trend we were looking for, we still saw signs of something going wrong

---

👤 **espen96** commented on **2025-08-13** at **13:56:14**

I was able to adjust things, and I am now on a path to being able to look at vllm vs the IK or mainline perplexity.

<img width="1428" height="544" alt="image" src="https://github.com/user-attachments/assets/384cf55d-2449-43aa-8693-6a5fff3a8cc3" />
<img width="1584" height="366" alt="image" src="https://github.com/user-attachments/assets/d6a0314a-0611-4fe4-b6a3-b20caf223b8f" />


however, I will take a break from this, we can return to the perplexity later. Mainline is also preparing some big updates.  
I will test the PR. 

I assume you have been using the smaller 20b model for testing? I'll focus on the 120b model

---

👤 **ikawrakow** commented on **2025-08-13** at **14:01:34**

> I assume you have been using the smaller 20b model for testing? I'll focus on the 120b model

Yes.

---

👤 **espen96** commented on **2025-08-13** at **14:06:55**

Getting a compile error

/mnt/CAC6A027C6A015AB/llama-swap_123_windows_amd64/engines/ik_llama.cpp/ggml/src/ggml-cuda/fattn-new-mma.cu(496): error: expression must have a constant value
if constexpr (ncols2 > 1 || mask_h2) {
^
/mnt/CAC6A027C6A015AB/llama-swap_123_windows_amd64/engines/ik_llama.cpp/ggml/src/ggml-cuda/fattn-new-mma.cu(496): note [#2689](https://github.com/ikawrakow/ik_llama.cpp/issues/2689)-D: the value of parameter "mask_h2" (declared at line 435) cannot be used as a constant
if constexpr (ncols2 > 1 || mask_h2) {
^
detected during:
instantiation of "void flash_attn_ext_f16_iter<DKQ,DV,ncols1,ncols2,nwarps,ntiles,use_logit_softcap,mla,needs_fixup,is_fixup,last_iter>(const float2 *, const half2 *, const half2 *, const half2 *, float2 *, float2 *, float, float, float, int, int, int, int, int, int, half2 *, half2 *, half2 *, half2 *, const tile_B *, tile_C_VKQ *, float *, float *, int) [with DKQ=64, DV=64, ncols1=8, ncols2=1, nwarps=4, ntiles=1, use_logit_softcap=false, mla=false, needs_fixup=false, is_fixup=false, last_iter=false]" at line 964
instantiation of "void flash_attn_ext_f16_process_tile<DKQ,DV,ncols1,ncols2,nwarps,ntiles,use_logit_softcap,mla,needs_fixup,is_fixup>(const float2 *, const half2 *, const half2 *, const half2 *, const float *, float2 *, float2 *, float, float, float, int, int, int, int, int, int, int, int, int, int) [with DKQ=64, DV=64, ncols1=8, ncols2=1, nwarps=4, ntiles=1, use_logit_softcap=false, mla=false, needs_fixup=false, is_fixup=false]" at line 1388
instantiation of "void flash_attn_ext_f16<DKQ,DV,ncols1,ncols2,nwarps,ntiles,use_logit_softcap,mla>(const char *, const char *, const char *, const char *, const char *, float *, float2 *, float, float, float, float, float, uint32_t, int, int, int, int, int, int, int, int, int, int, int, int, int, int, int, int, int, int, int, int, int, int, int) [with DKQ=64, DV=64, ncols1=8, ncols2=1, nwarps=4, ntiles=1, use_logit_softcap=false, mla=false]" at line 1843
instantiation of "void ggml_cuda_flash_attn_ext_mma_f16_case<DKQ,DV,ncols1,ncols2>(ggml_backend_cuda_context &, ggml_tensor *) [with DKQ=64, DV=64, ncols1=8, ncols2=1]" at line 1875
instantiation of "void ggml_cuda_flash_attn_ext_mma_f16_switch_ncols1<DKQ,DV,ncols2>(ggml_backend_cuda_context &, ggml_tensor *) [with DKQ=64, DV=64, ncols2=1]" at line 1928
instantiation of "void ggml_cuda_flash_attn_ext_mma_f16_switch_ncols2<DKQ,DV>(ggml_backend_cuda_context &, ggml_tensor *) [with DKQ=64, DV=64]" at line 1948

---

👤 **ikawrakow** commented on **2025-08-13** at **14:10:37**

Oops, I see. I tested the latest changes CPU-only. The CUDA build needs more work. Give me a few minutes.

---

👤 **ikawrakow** commented on **2025-08-13** at **14:24:10**

Should be good now also on CUDA.

---

👤 **ikawrakow** commented on **2025-08-13** at **14:34:10**

> Mainline is also preparing some big updates.

What are the big updates in mainline?

---

👤 **espen96** commented on **2025-08-13** at **14:58:41**

> > Mainline is also preparing some big updates.
> 
> What are the big updates in mainline?

the biggest one would be related to harmony
https://github.com/ggml-org/llama.cpp/pull/15181

I have seen a few PR's related to the attention sink, mostly further implementation and mxfp4 implementations, various fixes etc. but in hindsight now that I went back and checked, it's mostly about harmony and tool calling right now.
My bad, I think I miscalculated the amount of core implementation fixes going on over there.

---

👤 **espen96** commented on **2025-08-13** at **15:00:21**

Back to testing. 

on windows I will generally get about 14 tps with oss-120b, on linux I would get about 19. 


this first set of tests is giving me about 30 tps on linux
That is a significant bump in usability already

without `-ub 2048 -b 2048` I can move two more ffn layers onto my gpus, bumping the token generation speed up a few tps, 31-33

---

👤 **ikawrakow** commented on **2025-08-13** at **15:11:55**

Btw, I'm observing a strange PP performance behavior for this model as a function of `-ub`. In my case, testing with the 20B parameter model, on CUDA I get the best PP performance with `-ub 512`, it then degrades with increasing u-batch size. Not sure yet why. The 120B model may behave differently as there the ratio of total to active experts is higher. Then again, partial offload is going to benefit from larger u-batches.

So, please experiment with `-ub` as this model seems to be different in many ways from what we are used to.

---

👤 **SmallAndSoft** commented on **2025-08-13** at **15:13:07**

Maybe previous models with off-by-one attention bug just learned to be lulled into some level of perplexity and this fixed thing is just genuinely loosing track of what all this wiki-text as instruction is supposed to mean?

---

👤 **espen96** commented on **2025-08-13** at **16:51:30**

20b model, fully loaded exclusively on my 3090.

Mainline:
prompt eval time =      74.07 ms /    65 tokens (    1.14 ms per token,   877.55 tokens per second)
eval time =    9215.70 ms /   493 tokens (   18.69 ms per token,    53.50 tokens per second)
total time =    9289.77 ms /   558 tokens

IK:
prompt eval time =     103.73 ms /    65 tokens (    1.60 ms per token,   626.61 tokens per second) | tid="139990291603456" timestamp=1755103753 id_slot=0 id_task=2 t_prompt_processing=103.732 n_prompt_tokens_processed=65 t_token=1.595876923076923 n_tokens_second=626.6147379786373
generation eval time =   23949.75 ms /  1420 runs   (   16.87 ms per token,    59.29 tokens per second) | tid="139990291603456" timestamp=1755103753 id_slot=0 id_task=2 t_token_generation=23949.753 n_decoded=1420 t_token=16.866023239436622 n_tokens_second=59.29079936649033
total time =   24053.49 ms | tid="139990291603456" timestamp=1755103753 id_slot=0 id_task=2 t_prompt_processing=103.732 t_token_generation=23949.753 t_total=24053.485

---

👤 **saood06** commented on **2025-08-13** at **16:54:01**

> 20b model, fully loaded exclusively on my 3090.
> 
since you have reported results on both windows and linux, which is this?

---

👤 **espen96** commented on **2025-08-13** at **16:55:37**

> > 20b model, fully loaded exclusively on my 3090.
> 
> since you have reported results on both windows and linux, which is this?

ah my bad, Linux

---

👤 **espen96** commented on **2025-08-13** at **17:04:05**

ok, I was using some outdated settings!

with `-ngl 99 -np 1 --temp 0.8  --min-p 0.05 --repeat-penalty 1.1 --top-p 0.8 --top-k 40 --parallel 1 --threads 11 --main-gpu 0 -fa --tensor-split 100,0 -fmoe `


we are now at this for IK

prompt eval time     =      96.98 ms /    65 tokens (    1.49 ms per token,   670.26 tokens per second) 
generation eval time =   29000.94 ms /  3096 runs   (    9.37 ms per token,   106.76 tokens per second) 


and mainline is getting this with the same settings

prompt eval time =     114.47 ms /    65 tokens (    1.76 ms per token,   567.84 tokens per second)
eval time =   21011.09 ms /  2490 tokens (    8.44 ms per token,   118.51 tokens per second)


I had ` --temp 1.0 --min-p 0.0 --top-p 1.0 --top-k 0.0 ` which had been the settings Unsloth suggested at first, the settings now used are the ones LM Studio had as their pick.


Still on Linux, the 20B model

---

👤 **saood06** commented on **2025-08-13** at **17:09:00**

>the settings now used are the ones LM Studio had as their pick.

I would try and get rid of `--repeat-penalty 1.1` as it hurts newer models far more often than it helps in my opinion (and even for older models DRY is better), and also why `--threads 11` if you are running fully offloaded one thread should be tested.

---

👤 **espen96** commented on **2025-08-13** at **17:15:02**

> > the settings now used are the ones LM Studio had as their pick.
> 
> I would try and get rid of `--repeat-penalty 1.1` as it hurts newer models far more often than it helps in my opinion (and even for older models DRY is better), and also why `--threads 11` if you are running fully offloaded one thread should be tested.

ah good catch! I didn't think about the threads. the setup was mainly copied over from the 120b model.

also, DRY? I'm not sure what this means in this context

---

👤 **saood06** commented on **2025-08-13** at **17:20:04**

> also, DRY? I'm not sure what this means in this context

I don't think it would be useful for this model (I feel like repetition penalty should be off with newer models), but it is a sampler with support added here in [#513](https://github.com/ikawrakow/ik_llama.cpp/issues/513).

---

👤 **espen96** commented on **2025-08-13** at **18:10:44**

I see. 

By the way it seems the model started repeating itself badly without `--repeat-penalty`
even with no custom options at all.
<img width="1019" height="1042" alt="image" src="https://github.com/user-attachments/assets/335b5733-deae-40a9-9838-fe38edee2b8f" />

It can happen in mainline but appears to be less frequent? 
<img width="1019" height="633" alt="image" src="https://github.com/user-attachments/assets/7a0162a8-0851-4427-8a27-baf99f8f068d" />
<img width="1019" height="1100" alt="image" src="https://github.com/user-attachments/assets/4a7d1d9e-e2aa-49c6-9d21-8bc3b741f399" />


as for the speed, after a reboot my speeds really shot up. Same trend though on cuda. Still Linux, still the 20b model.

mainline:
prompt eval time =     101.09 ms /    65 tokens (    1.56 ms per token,   642.97 tokens per second)
eval time =   23489.45 ms /  3319 tokens (    7.08 ms per token,   141.30 tokens per second)

IK:
prompt eval tim =      92.88 ms /    65 tokens (    1.43 ms per token,   699.86 tokens per second)
generation eval time =   11746.86 ms /  1564 runs   (    7.51 ms per token,   133.14 tokens per second)



In any case, I will go back to testing the 120b model now

---

👤 **saood06** commented on **2025-08-13** at **18:25:20**

>By the way it seems the model started repeating itself badly without --repeat-penalty

Try `--dry_multiplier 0.55 --dry_base 1.5`?

---

👤 **espen96** commented on **2025-08-13** at **18:31:58**

> --dry_multiplier 0.55 --dry_base 1.5

at first glace, that appears to have done the trick. I'll continue to generate these stories and look for these extreme repititons


edit: the 20b model does appear to show signs of repition still. But it is not as bad. It's now more subtle. saying the same things in slighy different ways or in new contexts. but yes, that for sure made a difference. 
I will say, I am not surpised a 20b moe model will sound a bit repeditive in a creative writing context.
Asking for a story does appear to be a good way to spot repetition errors though.

<img width="1019" height="1037" alt="image" src="https://github.com/user-attachments/assets/69870e8a-f1d8-479d-aa04-b04b42629c95" />

---

👤 **saood06** commented on **2025-08-14** at **04:14:18**

> But it is not as bad. It's now more subtle. saying the same things in slighy different ways or in new contexts. but yes, that for sure made a difference.

I do think that DRY is better as a repetition penalty than the repetition penalty sampler, but it still only handles literal repetition, and doesn't handle semantic repetition, but it also has a downside as it has "false positives" (I use quotes because correct is subjective).

>I will say, I am not surpised a 20b moe model will sound a bit repeditive in a creative writing context.

Yep, given the size and architecture, but it still looks a lot worse than I'd have expected.

---

👤 **ikawrakow** commented on **2025-08-14** at **07:22:55**

@espen96 

Thank you for testing. A prompt of 65 tokens does not fully show the PP performance advantage of `ik_llama.cpp`. If you want to see that you need prompts of a few thousand tokens.

It is interesting that TG performance for the 20B model on your 3090 is lower than TG performance on my RTX-4080, which has a lower memory bandwidth than the 3090. This would indicate that TG for this model is at least partially compute/latency bound.

---

👤 **espen96** commented on **2025-08-14** at **11:47:54**

> @espen96
> 
> Thank you for testing. A prompt of 65 tokens does not fully show the PP performance advantage of `ik_llama.cpp`. If you want to see that you need prompts of a few thousand tokens.
> 
> It is interesting that TG performance for the 20B model on your 3090 is lower than TG performance on my RTX-4080, which has a lower memory bandwidth than the 3090. This would indicate that TG for this model is at least partially compute/latency bound.

No problem! glad to help!
right, I was looking at generation more than processing. Again, I should have noted that.
And yes, I feel like I have been seeing more and more numbers lately of people with faster speeds than me. With single gpu fully offloaded performance.


in any case, it's not building on windows.

```
MSBuild version 17.14.10+8b8e13593 for .NET Framework

  xxhash.vcxproj -> F:\llama-swap_123_windows_amd64\engines\ik_llama.cpp\build\examples\gguf-hash\xxhash.dir\Release\xxhash.lib
  sha256.vcxproj -> F:\llama-swap_123_windows_amd64\engines\ik_llama.cpp\build\examples\gguf-hash\sha256.dir\Release\sha256.lib
  sha1.vcxproj -> F:\llama-swap_123_windows_amd64\engines\ik_llama.cpp\build\examples\gguf-hash\sha1.dir\Release\sha1.lib
  build_info.vcxproj -> F:\llama-swap_123_windows_amd64\engines\ik_llama.cpp\build\common\build_info.dir\Release\build_info.lib
  Auto build dll exports
  ggml.vcxproj -> F:\llama-swap_123_windows_amd64\engines\ik_llama.cpp\build\bin\Release\ggml.dll
  llama-mmap.cpp
  llama-model-loader.cpp
F:\llama-swap_123_windows_amd64\engines\ik_llama.cpp\src\llama-mmap.cpp(119,89): warning C4267: 'argument': conversion from 'size_t' to 'DWORD', possible loss of data [F:\llama-swap_123_windows_amd64\engines\ik_llama.cpp\build\src\llama.vcxproj]
F:\llama-swap_123_windows_amd64\engines\ik_llama.cpp\src\llama-mmap.cpp(142,99): warning C4267: 'argument': conversion from 'size_t' to 'DWORD', possible loss of data [F:\llama-swap_123_windows_amd64\engines\ik_llama.cpp\build\src\llama.vcxproj]
C:\Program Files (x86)\Microsoft Visual Studio\2022\BuildTools\VC\Tools\MSVC\14.44.35207\include\memory(3630,35): error C2661: 'llama_mmap::impl::impl': no overloaded function takes 4 arguments [F:\llama-swap_123_windows_amd64\engines\ik_llama.cpp\build\src\llama.vcxproj]
  (compiling source file '../../src/llama-mmap.cpp')
      C:\Program Files (x86)\Microsoft Visual Studio\2022\BuildTools\VC\Tools\MSVC\14.44.35207\include\memory(3630,35):
      while trying to match the argument list '(llama_file *, size_t, bool, bool)'
      C:\Program Files (x86)\Microsoft Visual Studio\2022\BuildTools\VC\Tools\MSVC\14.44.35207\include\memory(3630,35):
      the template instantiation context (the oldest one first) is
          F:\llama-swap_123_windows_amd64\engines\ik_llama.cpp\src\llama-mmap.cpp(491,16):
          see reference to function template instantiation 'std::unique_ptr<llama_mmap::impl,std::default_delete<llama_mmap::impl>> std::make_unique<llama_mmap::impl,llama_file*&,size_t&,bool&,bool&,0>(llama_file *&,size_t &,bool &,bool &)' being compiled

F:\llama-swap_123_windows_amd64\engines\ik_llama.cpp\src\llama-model-loader.cpp(260,27): error C2065: 'PATH_MAX': undeclared identifier [F:\llama-swap_123_windows_amd64\engines\ik_llama.cpp\build\src\llama.vcxproj]
F:\llama-swap_123_windows_amd64\engines\ik_llama.cpp\src\llama-model-loader.cpp(269,25): error C2065: 'PATH_MAX': undeclared identifier [F:\llama-swap_123_windows_amd64\engines\ik_llama.cpp\build\src\llama.vcxproj]
F:\llama-swap_123_windows_amd64\engines\ik_llama.cpp\src\llama-model-loader.cpp(307,29): warning C4267: '=': conversion from 'size_t' to 'int', possible loss of data [F:\llama-swap_123_windows_amd64\engines\ik_llama.cpp\build\src\llama.vcxproj]
F:\llama-swap_123_windows_amd64\engines\ik_llama.cpp\src\llama-model-loader.cpp(261,47): error C1903: unable to recover from previous error(s); stopping compilation [F:\llama-swap_123_windows_amd64\engines\ik_llama.cpp\build\src\llama.vcxproj]
  Generating Code...

```

---

👤 **ikawrakow** commented on **2025-08-14** at **11:52:27**

In the process of of submitting a PR that enables CUDA graphs, I'll look into the Windows build failure when I'm done with that.

---

👤 **ikawrakow** commented on **2025-08-14** at **12:47:19**

@espen96 Does the last commit fix the Windows build issue?

---

👤 **espen96** commented on **2025-08-14** at **13:02:06**

> @espen96 Does the last commit fix the Windows build issue?

not quite

```
F:\llama-swap_123_windows_amd64\engines\ik_llama.cpp\src\llama-model-loader.cpp(260,27): error C2065: 'PATH_MAX': undeclared identifier [F:\llama-swap_123_windows_amd64\engines\ik_llama.cpp\build\src\llama.vcxproj]
F:\llama-swap_123_windows_amd64\engines\ik_llama.cpp\src\llama-model-loader.cpp(269,25): error C2065: 'PATH_MAX': undeclared identifier [F:\llama-swap_123_windows_amd64\engines\ik_llama.cpp\build\src\llama.vcxproj]
F:\llama-swap_123_windows_amd64\engines\ik_llama.cpp\src\llama-model-loader.cpp(307,29): warning C4267: '=': conversion from 'size_t' to 'int', possible loss of data [F:\llama-swap_123_windows_amd64\engines\ik_llama.cpp\build\src\llama.vcxproj]
F:\llama-swap_123_windows_amd64\engines\ik_llama.cpp\src\llama-model-loader.cpp(261,47): error C1903: unable to recover from previous error(s); stopping compilation [F:\llama-swap_123_windows_amd64\engines\ik_llama.cpp\build\src\llama.vcxproj]
```

---

👤 **ikawrakow** commented on **2025-08-14** at **13:07:40**

And now?

---

👤 **espen96** commented on **2025-08-14** at **13:08:52**

that seems to have done it! we have a build

---

👤 **espen96** commented on **2025-08-14** at **14:13:15**

We are testing the 120b model today, on windows. 

I gave it a bunch of glsl code to explain, and here on IK that gives us:

```
prompt eval time     =   16549.91 ms /  1971 tokens (    8.40 ms per token,   119.09 tokens per second)
generation eval time =  187501.14 ms /  2998 runs   (   62.54 ms per token,    15.99 tokens per second)
```

and on mainline:

```
prompt eval time =   11912.99 ms /  2032 tokens (    5.86 ms per token,   170.57 tokens per second)
eval time =  184227.84 ms /  3149 tokens (   58.50 ms per token,    17.09 tokens per second)
```

for a simple request for info on napoleon:

IK gives

```
prompt eval time     =     323.20 ms /     9 tokens (   35.91 ms per token,    27.85 tokens per second)
generation eval time =   28731.62 ms /   490 runs   (   58.64 ms per token,    17.05 tokens per second)
```
and mainline:

```
prompt eval time =     380.29 ms /     9 tokens (   42.25 ms per token,    23.67 tokens per second)
eval time =   90739.36 ms /  1632 tokens (   55.60 ms per token,    17.99 tokens per second)
```


I will note that my rig is not ideal
Ryzen 9 7900X with 64GB of ram, RTX 3090 with 24 GB, and an RTX 2060 SUPER with 8 GB.
and on windows it would have been better with just one GPU in use. I will try that later, but I doubt it would would impact the relative score,

To me, I feel like these number are within run to run variance on my rig.
I will do a proper benchmark later, using the benchmarking tool. This was more of a "real world" test.


I should note that mainline just added swa checkpoints. I don't have a build with that yet,
 https://github.com/ggml-org/llama.cpp/pull/15293
That should help a lot for long prompt re-processing times for the 120b model. Unsure how or if this applies to your fork @ikawrakow

---

👤 **ikawrakow** commented on **2025-08-14** at **14:20:17**

@espen96 

Thanks for testing.

Can you also post the commands that you used for the tests? Thanks.

---

👤 **ikawrakow** commented on **2025-08-14** at **14:26:07**

> I should note that mainline just added swa checkpoints. I don't have a build with that yet,

I haven't dealt with SWA at all in `ik_llama.cpp`. In the GPT-OSS models every second layer uses SWA, and the SWA window is only 128 tokens. Hence, there is a potential for a very significant speedup for long contexts (this is on top of the much smaller KV cache size that the linked mainline PR deals with). I'll look into that eventually.

---

👤 **espen96** commented on **2025-08-14** at **14:30:35**

> @espen96
> 
> Thanks for testing.
> 
> Can you also post the commands that you used for the tests? Thanks.

```
-ngl 99 -fa -fmoe -ot "blk\.([0-9]|10)\.ffn.*=CUDA0" -ot "blk\.(11|12)\.ffn.*=CUDA1" -ot "blk.*\.ffn.*=CPU" --dry_multiplier 0.55 --dry_base 1.5 --temp 0.8 --min-p 0.05 --top-p 0.8 --top-k 40 --parallel 1 --threads 11 --no-mmap  --main-gpu 0
 ```

---

👤 **ikawrakow** commented on **2025-08-14** at **14:40:22**

@espen96 Can you try the 120B mode with `-t 12`? For TG a lot of the computation is done on the CPU and `-t 11` effectively disables the faster CPU FA implementation.

---

👤 **espen96** commented on **2025-08-14** at **15:01:29**

> @espen96 Can you try the 120B mode with `-t 12`? For TG a lot of the computation is done on the CPU and `-t 11` effectively disables the faster CPU FA implementation.

that I can!

on IK this bumps us up a bit, we are now closer to 19 TG/s on the simple napoleon query. Similar results with mainline.

and my glsl query is up to around 18-19 TG/s on IK and mainline. 
So they are neck in neck. In the past however, hybrid has always been noticably faster with IK, so this is either model specific, or mainline has gotten some serious boosts.
I will test with a single GPU later, maybe that changes things.

---

👤 **ikawrakow** commented on **2025-08-14** at **15:12:25**

> So they are neck in neck. In the past however, hybrid has always been noticably faster with IK, so this is either model specific, or mainline has gotten some serious boosts.

I have noticed that mainline has become better compared to last time I tested, but it is also model specific. In the tests I ran for PR [#689](https://github.com/ikawrakow/ik_llama.cpp/issues/689) `ik_llama.cpp` was faster for PP and TG for DeepSeek-Lite and Qwen3-30B-A3B. For the GPT-OSS model `ik_llama.cpp` is faster for PP for all tested context lengths. For TG it is slightly faster initially, but becomes slower after about 8k tokens. There is something about the attention head size of 64 (not used by any other notable LLM) that seems to not be optimal in `ik_llama.cpp`.

---

👤 **ikawrakow** commented on **2025-08-15** at **06:19:50**

The changes in this PR were added to [#689](https://github.com/ikawrakow/ik_llama.cpp/issues/689), so closing.