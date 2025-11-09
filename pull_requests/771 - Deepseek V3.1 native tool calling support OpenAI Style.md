## 🔀 [Pull Request #771](https://github.com/ikawrakow/ik_llama.cpp/pull/771) - Deepseek V3.1 native tool calling support (OpenAI Style)

| **Author** | `firecoperana` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `fcp/deepseek3.1_toolcall` |
| **Target Branch** | `main` |
| **Created** | 2025-09-09 |
| **Updated** | 2025-10-26 |
| **Merged** | 2025-09-13 |
| **Assignees** | `firecoperana` |

---

## 📄 Description

Enable DeepSeek V3.1 thinking mode as the default. Disable with --reasoning-budget 0.
It also implements tool calling support.
Thinking model disabled assistant prefill.

Merges https://github.com/ggml-org/llama.cpp/pull/15533 and https://github.com/ggml-org/llama.cpp/pull/15404

- [x] I have read the [contributing guidelines](https://github.com/ggerganov/llama.cpp/blob/master/CONTRIBUTING.md)
- Self-reported review complexity:
  - [ ] Low
  - [x] Medium
  - [ ] High

---

## 💬 Conversation

👤 **ikawrakow** commented on **2025-09-10** at **15:17:39**

Can somebody test this? Thanks!

---

👤 **ChicoPinto70** commented on **2025-09-10** at **19:42:50**

I've just tested it and it seems to be working fine. 

My test was with the Ubergarm's DeepSeek-V3.1-smol-IQ4_KSS model in Roo Code, and, in reasoning mode, it uses the tools and it shows the thinking output in the proper frame, but:
1) To make it work, I must to replace the chat-template-file for the provided by the Unsloth version (I believe, in the Mainline page, they are using the Unsloth ones).  
2) I've noticed it, sometimes, fails in the tools calling. It may be because this model doesn't support tools calling with reasoning natively or because the chat-template-file injection is not a perfect solution.

That was the command line I've used: 
> CUDA_VISIBLE_DEVICES="1,2,0" ./build/bin/llama-server  --alias DeepSeek-V3.1-IQ4_KSS  -m /home/chico/.lmstudio/models/ubergarm/DeepSeek-V3.1-GGUF/DeepSeek-V3.1-smol-IQ4_KSS-00001-of-00007.gguf  -ngl 64 -c 65536 -mla 3 -fa -amb 512 -fmoe -t 28 -ctk q8_0 -ot "blk\.[0-6]\..*_exps\.=CUDA1,blk\.(7|8|9|10)\..*_exps\.=CUDA2,exps=CPU"  --parallel 1  --numa distribute -b 512 -ub 512 -ts 1,0,0 --host 192.168.0.9 --port 1235 --jinja --chat-template-file /home/chico/ik_llama.cpp/models/templates/Unsloth-DeepSeek-V3.1.jinja --reasoning-format auto

and, that is the chat-template-file I've used: https://huggingface.co/unsloth/DeepSeek-V3.1/blob/main/chat_template.jinja

I almost forgot!!! Thanks, @firecoperana!

---

👤 **arichiardi** commented on **2025-09-10** at **20:25:44**

@ikawrakow I can test this as well, compiling as we speak - FWIW I am using `GLM-4.5-Air` with a custom chat template

---

👤 **arichiardi** commented on **2025-09-10** at **20:37:50**

Noticed this branch spits out

`ik-llama@GLM-4.5-Air-ik[35756]: Enable thinking? 1`

But my command line reads:

```
--chat-template-kwargs '{"enable_thinking":false}'
```

I tried to remove it and I still get the same. I was expecting false or 0 for what is worth but it might be unrelated.

EDIT: the patch is definitely applied as I see the correct error response when sending a payload with `{"enable_thinking": "false"}` (should be the literal `false`).

```
ik-llama@GLM-4.5-Air-ik[36148]: INFO [      log_server_request] request | tid="140108745404416" timestamp=1757536917 remote_addr="..." remote_port=62582 status=500 method="POST" path="/v1/chat/completions" params={}
```

---

👤 **firecoperana** commented on **2025-09-11** at **00:24:21**

Checked with mainline and it also shows Enable thinking? 1.
It does not use --chat-template-kwargs, just reasoning-budget to set this value. I will remove this to avoid confusion.

---

👤 **arichiardi** commented on **2025-09-11** at **13:21:32**

@firecoperana that makes me wonder where the above `"enable_thinking"` is actually used. I would suggest you double check if it works on your side as well.

Shouldn't `"enable_thinking"` and `"reasoning_budget=0"` do the same thing after all?

---

👤 **firecoperana** commented on **2025-09-11** at **14:03:00**

Yes, they do the same thing. The only caveat is that is when you have both, enable_thinking will now override reasoning_budget.

---

👤 **arichiardi** commented on **2025-09-12** at **19:19:10**

FWIW, this looks good here (I compiled rebasing onto `main` as well)

---

👤 **ikawrakow** approved this pull request ✅ on **2025-09-13** at **05:51:28**

---

👤 **kirnat** commented on **2025-09-23** at **16:48:16**

Thanks for adding this. For some reason I can't get tool calling working properly with DeepSeek V3.1. It works fine with upstream. Could it be a chat template issue? The template ChicoPinto70 used should be the same as the one included with the Unsloth model I tested with, but I am a bit unsure if RooCode actually uses native OpenAI tool calling, I thought they used a custom XML-tag styled formatting/parsing.

Model used:
unsloth/DeepSeek-V3.1-GGUF/UD-Q3_K_XL

Notable arguments:
`--jinja --reasoning-format auto`
## Test Case
```python
payload = {
    "model": "model", 
    "messages": [{"role": "user", "content": "List files in /tmp"}],
    "tools": [{
        "type": "function",
        "function": {
            "name": "list_directory",
            "description": "List files in a directory",
            "parameters": {"type": "object", "properties": {"path": {"type": "string"}}}
        }
    }],
    "tool_choice": "auto"
}
```

## Response (ik_llama.cpp 18f0435):
```
"choices": [
    {
      "finish_reason": "stop",
      "index": 0,
      "message": {
        "role": "assistant",
        "reasoning_content": "Okay, the user wants me to list the files in the /tmp directory. This is a straightforward request that requires a simple directory listing operation. \n\nI'll use the list_directory function with the path parameter set to \"/tmp\". This should return the contents of that directory. \n\nThe function is designed to handle this exact type of request, so no additional parameters or special handling is needed.",
        "content": "list_directory{\"path\": \"/tmp\"}"
      }
    }
  ]
```

## Response (llama.cpp):
```
"choices": [
    {
      "finish_reason": "tool_calls",
      "index": 0,
      "message": {
        "role": "assistant",
        "reasoning_content": "First, the user is asking to list the files in the /tmp directory. I have a function called list_directory that can handle this. The function requires a path parameter, which in this case is \"/tmp\".\n\nI need to call the list_directory function with the path set to \"/tmp\". The tool call syntax is:<\uff5ctool\u2581calls\u2581begin\uff5c><\uff5ctool\u2581call\u2581begin\uff5c>tool_name<\uff5ctool\u2581sep\uff5c>{\"arg1\": \"some_value\"}<\uff5ctool\u2581call\u2581end\uff5c><\uff5ctool\u2581calls\u2581end\uff5c> So for this, it should be:<\uff5ctool\u2581calls\u2581begin\uff5c><\uff5ctool\u2581call\u2581begin\uff5c>list_directory<\uff5ctool\u2581sep\uff5c>{\"path\": \"/tmp\"}<\uff5ctool\u2581call\u2581end\uff5c><\uff5ctool\u2581calls\u2581end\uff5c>",
        "content": "I'll list the files in the /tmp directory for you.",
        "tool_calls": [
          {
            "type": "function",
            "function": {
              "name": "list_directory",
              "arguments": "{\"path\":\"/tmp\"}"
            },
            "id": "xgYG5r2rWS2WBewIvPmYLNEuPfkTHsGC"
          }
        ]
      }
    }
  ]
```

---

👤 **firecoperana** commented on **2025-09-23** at **18:13:06**

So the reasoning works for deepseek v3.1. Can you try to send the same payload a few times? The success rate of tool calls varies by model.

---

👤 **kirnat** commented on **2025-09-23** at **19:04:54**

Thanks. Yes, the reasoning works great. I have run the test about 20-30 times, and the only thing I have noticed is that the LLM sometimes outputs the tool call in reasoning content and sometimes in the regular content. This particular model has been performing really well with tool calling in llama.cpp.

---

👤 **firecoperana** commented on **2025-09-23** at **20:07:33**

If you change `tool_choice` from `auto` to `required`, does it force it to generate tool call? Seems like not a parsing issue, but the model does not generate tool call content.

---

👤 **kirnat** commented on **2025-09-25** at **18:14:04**

If I set too_choice to required it does the same thing - output tool call in content like in my example output above. I don't know what else to test, same model works flawless in llama.cpp with the same options and after making 100's of tool calls it hasn't failed formatting even once, DeepSeek 3.1 is exceptionally good at this task, especially compared to V3 and R1.

---

👤 **firecoperana** commented on **2025-09-25** at **19:04:45**

Ok. Unfortunately I don't have Deepseek V3.1 at hand to test and it will be a while before I have time to try it. Hope someone who have used tool call successfully on Deepseek V3.1 can share their experience.

---

👤 **kirnat** commented on **2025-09-26** at **08:30:56**

Thanks a lot. I will try to do more debugging and report any potential findings.

---

👤 **firecoperana** commented on **2025-09-26** at **13:29:27**

https://github.com/ikawrakow/ik_llama.cpp/pull/799 See if this makes any difference for you.