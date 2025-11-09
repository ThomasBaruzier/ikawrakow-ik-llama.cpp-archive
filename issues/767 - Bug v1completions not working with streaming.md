## 📌 [Issue #767](https://github.com/ikawrakow/ik_llama.cpp/issues/767) - Bug: `/v1/completions` not working with streaming

| **Author** | `MRGRD56` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-09-08 |
| **Updated** | 2025-09-09 |

---

## 📄 Description

### What happened?

I'm trying to run DeepSeek V3.1 UD-Q2_K_XL with ik_llama.cpp and there seems to be a bug regarding the `/v1/completions` endpoint when used with streaming. It returns all pieces of output as empty text, also returning `"finish_reason": "length"`:

## `/v1/completions`

### Request

```http
POST /v1/completions

{
    "prompt": "You are a helpful assistant.<｜User｜>Hi there!<｜Assistant｜></think>",
    "model": "DeepSeek-V3.1:Q2_K_XL",
    "stream": true
}
```

### ❌ Response w/ streaming (gives no text)

```http
data: {"choices":[{"text":"","index":0,"logprobs":null,"finish_reason":"length"}],"created":1757338637,"model":"","object":"text_completion","usage":{"completion_tokens":1,"prompt_tokens":13,"total_tokens":14},"id":""}

data: {"choices":[{"text":"","index":0,"logprobs":null,"finish_reason":"length"}],"created":1757338637,"model":"","object":"text_completion","usage":{"completion_tokens":2,"prompt_tokens":13,"total_tokens":15},"id":""}

data: {"choices":[{"text":"","index":0,"logprobs":null,"finish_reason":"length"}],"created":1757338637,"model":"","object":"text_completion","usage":{"completion_tokens":3,"prompt_tokens":13,"total_tokens":16},"id":""}

data: {"choices":[{"text":"","index":0,"logprobs":null,"finish_reason":"length"}],"created":1757338638,"model":"","object":"text_completion","usage":{"completion_tokens":4,"prompt_tokens":13,"total_tokens":17},"id":""}

data: {"choices":[{"text":"","index":0,"logprobs":null,"finish_reason":"length"}],"created":1757338638,"model":"","object":"text_completion","usage":{"completion_tokens":5,"prompt_tokens":13,"total_tokens":18},"id":""}

data: {"choices":[{"text":"","index":0,"logprobs":null,"finish_reason":"length"}],"created":1757338638,"model":"","object":"text_completion","usage":{"completion_tokens":6,"prompt_tokens":13,"total_tokens":19},"id":""}

data: {"choices":[{"text":"","index":0,"logprobs":null,"finish_reason":"length"}],"created":1757338638,"model":"","object":"text_completion","usage":{"completion_tokens":7,"prompt_tokens":13,"total_tokens":20},"id":""}

data: {"choices":[{"text":"","index":0,"logprobs":null,"finish_reason":"length"}],"created":1757338639,"model":"","object":"text_completion","usage":{"completion_tokens":8,"prompt_tokens":13,"total_tokens":21},"id":""}

data: {"choices":[{"text":"","index":0,"logprobs":null,"finish_reason":"length"}],"created":1757338639,"model":"","object":"text_completion","usage":{"completion_tokens":9,"prompt_tokens":13,"total_tokens":22},"id":""}

data: {"choices":[{"text":"","index":0,"logprobs":null,"finish_reason":"length"}],"created":1757338640,"model":"","object":"text_completion","usage":{"completion_tokens":11,"prompt_tokens":13,"total_tokens":24},"id":""}

data: {"choices":[{"text":"","index":0,"logprobs":null,"finish_reason":"length"}],"created":1757338640,"model":"","object":"text_completion","usage":{"completion_tokens":12,"prompt_tokens":13,"total_tokens":25},"id":""}

data: {"choices":[{"text":"","index":0,"logprobs":null,"finish_reason":"stop"}],"created":1757338640,"model":"","object":"text_completion","usage":{"completion_tokens":12,"prompt_tokens":13,"total_tokens":25},"id":"","timings":{"prompt_n":7,"prompt_ms":1544.566,"prompt_per_token_ms":220.6522857142857,"prompt_per_second":4.532017408126295,"predicted_n":12,"predicted_ms":3467.021,"predicted_per_token_ms":288.9184166666667,"predicted_per_second":3.461184688526547}}
```

Without streaming it works fine:

### ✅ Response w/o streaming

```http
{
    "choices": [
        {
            "text": "Hello! How can I assist you today? 😊",
            "index": 0,
            "logprobs": null,
            "finish_reason": "stop"
        }
    ],
    "created": 1757338754,
    "model": "",
    "object": "text_completion",
    "usage": {
        "completion_tokens": 12,
        "prompt_tokens": 13,
        "total_tokens": 25
    },
    "id": "chatcmpl-pEcdjHvTaRehwKyLYXMDJUXEkkVMtbmh",
    "timings": {
        "prompt_n": 1,
        "prompt_ms": 531.116,
        "prompt_per_token_ms": 531.116,
        "prompt_per_second": 1.8828278568147072,
        "predicted_n": 12,
        "predicted_ms": 2222.289,
        "predicted_per_token_ms": 185.19075,
        "predicted_per_second": 5.399837734876066
    }
}
```

## `/v1/chat/completions`

This works fine as well:

### Request

```http
POST /v1/chat/completions

{
    "messages": [
        {
            "role": "system",
            "content": "You are a helpful assistant."
        },
        {
            "role": "user",
            "content": "Hi there!"
        }
    ],
    "model": "DeepSeek-V3.1:Q2_K_XL",
    "stream": true // without streaming it works too
}
```

### ✅ Response w/ streaming

```http
data: {"choices":[{"finish_reason":null,"index":0,"delta":{"role":"assistant","content":null}}],"created":1757338798,"id":"chatcmpl-TRl3UBH4cWLpeUD8Zfyc6BpmP58qmXov","model":"","object":"chat.completion.chunk","usage":{"completion_tokens":1,"prompt_tokens":12,"total_tokens":13}}

data: {"choices":[{"finish_reason":null,"index":0,"delta":{"content":"Hello"}}],"created":1757338798,"id":"chatcmpl-TRl3UBH4cWLpeUD8Zfyc6BpmP58qmXov","model":"","object":"chat.completion.chunk","usage":{"completion_tokens":1,"prompt_tokens":12,"total_tokens":13}}

data: {"choices":[{"finish_reason":null,"index":0,"delta":{"content":"!"}}],"created":1757338799,"id":"chatcmpl-TRl3UBH4cWLpeUD8Zfyc6BpmP58qmXov","model":"","object":"chat.completion.chunk","usage":{"completion_tokens":2,"prompt_tokens":12,"total_tokens":14}}

data: {"choices":[{"finish_reason":null,"index":0,"delta":{"content":" How"}}],"created":1757338799,"id":"chatcmpl-TRl3UBH4cWLpeUD8Zfyc6BpmP58qmXov","model":"","object":"chat.completion.chunk","usage":{"completion_tokens":3,"prompt_tokens":12,"total_tokens":15}}

data: {"choices":[{"finish_reason":null,"index":0,"delta":{"content":" can"}}],"created":1757338799,"id":"chatcmpl-TRl3UBH4cWLpeUD8Zfyc6BpmP58qmXov","model":"","object":"chat.completion.chunk","usage":{"completion_tokens":4,"prompt_tokens":12,"total_tokens":16}}

data: {"choices":[{"finish_reason":null,"index":0,"delta":{"content":" I"}}],"created":1757338799,"id":"chatcmpl-TRl3UBH4cWLpeUD8Zfyc6BpmP58qmXov","model":"","object":"chat.completion.chunk","usage":{"completion_tokens":5,"prompt_tokens":12,"total_tokens":17}}

data: {"choices":[{"finish_reason":null,"index":0,"delta":{"content":" help"}}],"created":1757338800,"id":"chatcmpl-TRl3UBH4cWLpeUD8Zfyc6BpmP58qmXov","model":"","object":"chat.completion.chunk","usage":{"completion_tokens":6,"prompt_tokens":12,"total_tokens":18}}

data: {"choices":[{"finish_reason":null,"index":0,"delta":{"content":" you"}}],"created":1757338800,"id":"chatcmpl-TRl3UBH4cWLpeUD8Zfyc6BpmP58qmXov","model":"","object":"chat.completion.chunk","usage":{"completion_tokens":7,"prompt_tokens":12,"total_tokens":19}}

data: {"choices":[{"finish_reason":null,"index":0,"delta":{"content":" today"}}],"created":1757338800,"id":"chatcmpl-TRl3UBH4cWLpeUD8Zfyc6BpmP58qmXov","model":"","object":"chat.completion.chunk","usage":{"completion_tokens":8,"prompt_tokens":12,"total_tokens":20}}

data: {"choices":[{"finish_reason":null,"index":0,"delta":{"content":"?"}}],"created":1757338801,"id":"chatcmpl-TRl3UBH4cWLpeUD8Zfyc6BpmP58qmXov","model":"","object":"chat.completion.chunk","usage":{"completion_tokens":9,"prompt_tokens":12,"total_tokens":21}}

data: {"choices":[{"finish_reason":"stop","index":0,"delta":{}}],"created":1757338801,"id":"chatcmpl-TRl3UBH4cWLpeUD8Zfyc6BpmP58qmXov","model":"DeepSeek-V3.1:Q2_K_XL","object":"chat.completion.chunk"}

data: {"choices":[],"created":1757338801,"id":"chatcmpl-TRl3UBH4cWLpeUD8Zfyc6BpmP58qmXov","model":"DeepSeek-V3.1:Q2_K_XL","object":"chat.completion.chunk","usage":{"completion_tokens":10,"prompt_tokens":12,"total_tokens":22},"timings":{"prompt_n":6,"prompt_ms":1520.843,"prompt_per_token_ms":253.47383333333335,"prompt_per_second":3.945180403236889,"predicted_n":10,"predicted_ms":2745.682,"predicted_per_token_ms":274.5682,"predicted_per_second":3.642082367877999}}

data: [DONE]
```

### ✅ Response w/o streaming

```json
{
    "choices": [
        {
            "finish_reason": "stop",
            "index": 0,
            "message": {
                "role": "assistant",
                "content": "Hello! How can I assist you today?"
            }
        }
    ],
    "created": 1757338903,
    "model": "DeepSeek-V3.1:Q2_K_XL",
    "object": "chat.completion",
    "usage": {
        "completion_tokens": 10,
        "prompt_tokens": 12,
        "total_tokens": 22
    },
    "id": "chatcmpl-jSPnOx1vh2lfu07oxE5pXkTpMDJ2SR1O",
    "timings": {
        "prompt_n": 1,
        "prompt_ms": 473.108,
        "prompt_per_token_ms": 473.108,
        "prompt_per_second": 2.113682288187898,
        "predicted_n": 10,
        "predicted_ms": 1805.319,
        "predicted_per_token_ms": 180.5319,
        "predicted_per_second": 5.539187257210498
    }
}
```

### Name and Version

PS C:\apps\ik_llama.cpp> .\build\bin\Release\llama-cli.exe --version
version: 3881 (c519d417)
built with MSVC 19.41.34123.0 for x64

### What operating system are you seeing the problem on?

Windows

### Relevant log output

```powershell
C:\apps\ik_llama.cpp\build\bin\Release\llama-server --port 15900
      --model "E:\AI\LLM\gguf\unsloth\DeepSeek-V3.1-GGUF\DeepSeek-V3.1-UD-Q2_K_XL-00001-of-00006.gguf"
      -rtr
      --ctx-size 16384
      -ctk q8_0 -ctv q8_0
      -mla 2 -fa
      -amb 512
      -fmoe
      --n-gpu-layers 63
      -mg 0
      -ot "blk\.(0?[0-2])\.ffn_.*_exps.=CUDA0"
      -ot ".ffn_.*_exps.=CPU"
      --parallel 1
      -tb 30 -t 15
```

---

## 💬 Conversation

👤 **firecoperana** commented on **2025-09-08** at **20:03:26**

I forgot to update the code for streaming v1/completions. Will fix.

---

👤 **firecoperana** commented on **2025-09-09** at **02:58:16**

@MRGRD56 Can you check if this fixes your issue? https://github.com/ikawrakow/ik_llama.cpp/pull/768

---

👤 **MRGRD56** commented on **2025-09-09** at **03:34:07**

@firecoperana Yep, looks like it's working on your branch, thanks!

```http
data: {"choices":[{"text":"Hello","index":0,"logprobs":null,"finish_reason":null}],"created":1757388198,"model":"","object":"text_completion","usage":{"completion_tokens":1,"prompt_tokens":13,"total_tokens":14},"id":""}

data: {"choices":[{"text":"!","index":0,"logprobs":null,"finish_reason":null}],"created":1757388199,"model":"","object":"text_completion","usage":{"completion_tokens":2,"prompt_tokens":13,"total_tokens":15},"id":""}

data: {"choices":[{"text":" How","index":0,"logprobs":null,"finish_reason":null}],"created":1757388199,"model":"","object":"text_completion","usage":{"completion_tokens":3,"prompt_tokens":13,"total_tokens":16},"id":""}

data: {"choices":[{"text":" can","index":0,"logprobs":null,"finish_reason":null}],"created":1757388199,"model":"","object":"text_completion","usage":{"completion_tokens":4,"prompt_tokens":13,"total_tokens":17},"id":""}

data: {"choices":[{"text":" I","index":0,"logprobs":null,"finish_reason":null}],"created":1757388200,"model":"","object":"text_completion","usage":{"completion_tokens":5,"prompt_tokens":13,"total_tokens":18},"id":""}

data: {"choices":[{"text":" help","index":0,"logprobs":null,"finish_reason":null}],"created":1757388200,"model":"","object":"text_completion","usage":{"completion_tokens":6,"prompt_tokens":13,"total_tokens":19},"id":""}

data: {"choices":[{"text":" you","index":0,"logprobs":null,"finish_reason":null}],"created":1757388200,"model":"","object":"text_completion","usage":{"completion_tokens":7,"prompt_tokens":13,"total_tokens":20},"id":""}

data: {"choices":[{"text":" today","index":0,"logprobs":null,"finish_reason":null}],"created":1757388201,"model":"","object":"text_completion","usage":{"completion_tokens":8,"prompt_tokens":13,"total_tokens":21},"id":""}

data: {"choices":[{"text":"?","index":0,"logprobs":null,"finish_reason":null}],"created":1757388201,"model":"","object":"text_completion","usage":{"completion_tokens":9,"prompt_tokens":13,"total_tokens":22},"id":""}

data: {"choices":[{"text":"","index":0,"logprobs":null,"finish_reason":null}],"created":1757388201,"model":"","object":"text_completion","usage":{"completion_tokens":10,"prompt_tokens":13,"total_tokens":23},"id":""}

data: {"choices":[{"text":"","index":0,"logprobs":null,"finish_reason":"stop"}],"created":1757388201,"model":"","object":"text_completion","usage":{"completion_tokens":10,"prompt_tokens":13,"total_tokens":23},"id":"","timings":{"prompt_n":1,"prompt_ms":521.473,"prompt_per_token_ms":521.473,"prompt_per_second":1.9176448253313212,"predicted_n":10,"predicted_ms":2873.832,"predicted_per_token_ms":287.3832,"predicted_per_second":3.4796745251636145}}
```

I'm not sure about this empty chunk just before the last one, though

```http
data: {"choices":[{"text":"","index":0,"logprobs":null,"finish_reason":null}],"created":1757388201,"model":"","object":"text_completion","usage":{"completion_tokens":10,"prompt_tokens":13,"total_tokens":23},"id":""}
```

I don't think it breaks anything but maybe it shouldn’t be there?

---

👤 **MRGRD56** commented on **2025-09-09** at **03:41:34**

Seems like the empty chunk is fine, since the original llama.cpp outputs it too

```http
data: {"choices":[{"text":"Hello","index":0,"logprobs":null,"finish_reason":null}],"created":1757389182,"model":"DeepSeek-V3.1:Q2_K_XL","system_fingerprint":"b6351-5d804a49","object":"text_completion","id":"chatcmpl-qzz1Qobc2bVU2fwWIAtaxgjQ2qeFeNbo"}

data: {"choices":[{"text":"!","index":0,"logprobs":null,"finish_reason":null}],"created":1757389183,"model":"DeepSeek-V3.1:Q2_K_XL","system_fingerprint":"b6351-5d804a49","object":"text_completion","id":"chatcmpl-qzz1Qobc2bVU2fwWIAtaxgjQ2qeFeNbo"}

data: {"choices":[{"text":" How","index":0,"logprobs":null,"finish_reason":null}],"created":1757389183,"model":"DeepSeek-V3.1:Q2_K_XL","system_fingerprint":"b6351-5d804a49","object":"text_completion","id":"chatcmpl-qzz1Qobc2bVU2fwWIAtaxgjQ2qeFeNbo"}

data: {"choices":[{"text":" can","index":0,"logprobs":null,"finish_reason":null}],"created":1757389183,"model":"DeepSeek-V3.1:Q2_K_XL","system_fingerprint":"b6351-5d804a49","object":"text_completion","id":"chatcmpl-qzz1Qobc2bVU2fwWIAtaxgjQ2qeFeNbo"}

data: {"choices":[{"text":" I","index":0,"logprobs":null,"finish_reason":null}],"created":1757389183,"model":"DeepSeek-V3.1:Q2_K_XL","system_fingerprint":"b6351-5d804a49","object":"text_completion","id":"chatcmpl-qzz1Qobc2bVU2fwWIAtaxgjQ2qeFeNbo"}

data: {"choices":[{"text":" help","index":0,"logprobs":null,"finish_reason":null}],"created":1757389183,"model":"DeepSeek-V3.1:Q2_K_XL","system_fingerprint":"b6351-5d804a49","object":"text_completion","id":"chatcmpl-qzz1Qobc2bVU2fwWIAtaxgjQ2qeFeNbo"}

data: {"choices":[{"text":" you","index":0,"logprobs":null,"finish_reason":null}],"created":1757389184,"model":"DeepSeek-V3.1:Q2_K_XL","system_fingerprint":"b6351-5d804a49","object":"text_completion","id":"chatcmpl-qzz1Qobc2bVU2fwWIAtaxgjQ2qeFeNbo"}

data: {"choices":[{"text":" today","index":0,"logprobs":null,"finish_reason":null}],"created":1757389184,"model":"DeepSeek-V3.1:Q2_K_XL","system_fingerprint":"b6351-5d804a49","object":"text_completion","id":"chatcmpl-qzz1Qobc2bVU2fwWIAtaxgjQ2qeFeNbo"}

data: {"choices":[{"text":"?","index":0,"logprobs":null,"finish_reason":null}],"created":1757389184,"model":"DeepSeek-V3.1:Q2_K_XL","system_fingerprint":"b6351-5d804a49","object":"text_completion","id":"chatcmpl-qzz1Qobc2bVU2fwWIAtaxgjQ2qeFeNbo"}

data: {"choices":[{"text":" 😊","index":0,"logprobs":null,"finish_reason":null}],"created":1757389184,"model":"DeepSeek-V3.1:Q2_K_XL","system_fingerprint":"b6351-5d804a49","object":"text_completion","id":"chatcmpl-qzz1Qobc2bVU2fwWIAtaxgjQ2qeFeNbo"}

data: {"choices":[{"text":"","index":0,"logprobs":null,"finish_reason":null}],"created":1757389185,"model":"DeepSeek-V3.1:Q2_K_XL","system_fingerprint":"b6351-5d804a49","object":"text_completion","id":"chatcmpl-qzz1Qobc2bVU2fwWIAtaxgjQ2qeFeNbo"}

data: {"choices":[{"text":"","index":0,"logprobs":null,"finish_reason":"stop"}],"created":1757389185,"model":"DeepSeek-V3.1:Q2_K_XL","system_fingerprint":"b6351-5d804a49","object":"text_completion","usage":{"completion_tokens":12,"prompt_tokens":13,"total_tokens":25},"id":"chatcmpl-qzz1Qobc2bVU2fwWIAtaxgjQ2qeFeNbo","timings":{"prompt_n":1,"prompt_ms":557.744,"prompt_per_token_ms":557.744,"prompt_per_second":1.7929372615393442,"predicted_n":12,"predicted_ms":2078.104,"predicted_per_token_ms":173.17533333333333,"predicted_per_second":5.774494443011514}}

data: [DONE]


```