## 📌 [Issue #915](https://github.com/ikawrakow/ik_llama.cpp/issues/915) - Bug: Embedding API returns empty embedding array with ik-llama build

| **Author** | `Kotali-2019` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-11-07 |
| **Updated** | 2025-11-09 |

---

## 📄 Description

### What happened?

### Description:
When testing the embedding functionality using the ik-llama build, the API returns an **empty** embedding array instead of the expected vector.
I don’t believe the issue is caused by the downloaded GGUF model because I tested the same model with llama.cpp and the embedding result was as expected.
Additionally, text generation works fine with this build (tested with “think” and “no-think” models), so the problem seems isolated to embedding.

### Steps to Reproduce:
**Server command:**
```bash
./llama-server -m /root/models/Draft/Embedding-Qwen3-0.6B-Q8_0.gguf \
-t 72 --host 0.0.0.0 --port 10002 --parallel 2 --embedding \
--pooling last --embd-output-format json --alias Embedding-Qwen3-0.6B-Q8_0
```
**Curl command:**
```bash
curl --location 'http://192.168.11.101:10002/v1/embeddings' \
--header 'Content-Type: application/json' \
--data '{
    "model": "Embedding-Qwen3-0.6B-Q8_0",
    "input": "Test"
}'
```
**llama.cpp result:**
```json
{"model":"Embedding-Qwen3-0.6B-Q8_0","object":"list","usage":{"prompt_tokens":2,"total_tokens":2},"data":[{"embedding":[-0.012152200564742088,-0.059889741241931915,...0.01874898374080658,0.004860232584178448],"index":0,"object":"embedding"}]}
```
**ik-llama build result:**
```json
{"model":"Embedding-Qwen3-0.6B-Q8_0","object":"list","usage":{"prompt_tokens":0,"total_tokens":0},"data":[{"embedding":[],"index":0,"object":"embedding"}]}
```

### Name and Version

./llama-cli --version
version: 3958 (575e2c28)
built with cc (Debian 12.2.0-14+deb12u1) 12.2.0 for x86_64-linux-gnu

### What operating system are you seeing the problem on?

Linux

### Relevant log output

```shell

```

---

## 💬 Conversation

👤 **firecoperana** commented on **2025-11-07** at **17:25:58**

Embedding is never tested, so bugs are expected. I will fix it later when I have time.

---

👤 **ikawrakow** commented on **2025-11-08** at **14:30:55**

@firecoperana 

Do you want to try to fix this? I see that there is no way the embedding endpoint is going to work in the server version on the main branch, but I feel completely lost in the server code.

---

👤 **firecoperana** commented on **2025-11-08** at **14:34:56**

Yes,  I will.

---

👤 **firecoperana** commented on **2025-11-08** at **20:52:04**

Can you check if https://github.com/ikawrakow/ik_llama.cpp/pull/924 fixed it?

---

👤 **Kotali-2019** commented on **2025-11-09** at **03:13:36**

**Hi @firecoperana thank you so much for fixing this!**
New version spilled out embedding vectors, however, I think it calculation is not accurate compare to the llama.cpp version
```
./llama-server --version
version: 3976 (116ab7e3)
built with cc (Debian 12.2.0-14+deb12u1) 12.2.0 for x86_64-linux-gnu
```
I ran the same request as before, and here are the results. Please note the difference in prompt_tokens usage: in the ik-llama build it is 0.

IK_llama build :
```json
{"model":"Embedding-Qwen3-0.6B-Q8_0","object":"list","usage":{"prompt_tokens":0,"total_tokens":0},"data":[{"embedding":[-0.011915850453078747,-0.06013014540076256,...,-0.017192969098687172,0.01830339804291725,0.004634905140846968],"index":0,"object":"embedding"}]}
```

Llama.cpp version:
```json
{"model":"Embedding-Qwen3-0.6B-Q8_0","object":"list","usage":{"prompt_tokens":2,"total_tokens":2},"data":[{"embedding":[-0.012152200564742088,-0.059889741241931915,...,-0.017190614715218544,0.01874898374080658,0.004860232584178448],"index":0,"object":"embedding"}]}
```

Some relevant server logs output:
IK
```
INFO [                    main] HTTP server listening | tid="140272568453504" timestamp=1762662518 n_threads_http="71" port="10002" hostname="0.0.0.0"
INFO [            update_slots] all slots are idle | tid="140272568453504" timestamp=1762662518
INFO [   launch_slot_with_task] slot is processing task | tid="140272568453504" timestamp=1762662526 id_slot=0 id_task=0
INFO [            update_slots] kv cache rm [p0, end) | tid="140272568453504" timestamp=1762662526 id_slot=0 id_task=0 p0=0
INFO [            update_slots] slot released | tid="140272568453504" timestamp=1762662526 id_slot=0 id_task=0 n_ctx=32768 n_past=2 n_system_tokens=0 n_cache_tokens=2 truncated=false
INFO [            update_slots] all slots are idle | tid="140272568453504" timestamp=1762662526
INFO [      log_server_request] request | tid="140267154810560" timestamp=1762662526 remote_addr="192.168.11.111" remote_port=55340 status=200 method="POST" path="/v1/embeddings" params={}
```
Llama.cpp
```
srv  update_slots: all slots are idle
slot get_availabl: id  1 | task -1 | selected slot by LCP similarity, sim_best = 1.000 (> 0.100 thold), f_keep = 1.000
slot launch_slot_: id  1 | task 25 | processing task
slot update_slots: id  1 | task 25 | new prompt, n_ctx_slot = 2048, n_keep = 0, task.n_tokens = 2
slot update_slots: id  1 | task 25 | need to evaluate at least 1 token for each active slot (n_past = 2, task.n_tokens() = 2)
slot update_slots: id  1 | task 25 | n_past was set to 1
slot update_slots: id  1 | task 25 | n_tokens = 1, memory_seq_rm [1, end)
slot update_slots: id  1 | task 25 | prompt processing progress, n_tokens = 2, batch.n_tokens = 1, progress = 1.000000
slot update_slots: id  1 | task 25 | prompt done, n_tokens = 2, batch.n_tokens = 1
slot      release: id  1 | task 25 | stop processing: n_tokens = 2, truncated = 0
srv  update_slots: all slots are idle
srv  log_server_r: request: POST /v1/embeddings 192.168.11.111 200
```

BTW, will you consider adding rerank support? @ikawrakow

---

👤 **ikawrakow** commented on **2025-11-09** at **06:32:32**

> New version spilled out embedding vectors, however, I think it calculation is not accurate compare to the llama.cpp version

Not accurate in what sense? The output embeddings look basically the same to me. Differences in that order are to be expected due to the non-associativity of floating point operations. Have you tried computing the embeddings with `llama.cpp` using different numbers of threads? If so, do you get the exact same result?

The `prompt_tokens` and `total_tokens` values need to be fixed, but other than that it looks OK to me.

> BTW, will you consider adding rerank support? @ikawrakow 

My primary interest is the computation engine, so I guess it would get added if @firecoperana is interested in doing it.

---

👤 **ikawrakow** commented on **2025-11-09** at **10:33:16**

So, to follow up on the differences between `llama.cpp` and `ik_llama.cpp`, I ran the example with `llama.cpp`, once with the CPU back-end and once with the CUDA backend. I also ran the example with `ik_llama.cpp` CPU back-end (your primary interest). I then made a little utility to compute the root-mean-squared-difference (rmsd) and the maximum difference between two embeddings. Here is what I get

| comparison | rmds | max difference |
| ---: | ---: | ---: |
| llama.cpp CPU vs CUDA | 0.000802512 | 0.00286971 |
| ik_llama.cpp vs llama.cpp | 0.000549097 | 0.00204271 |

As you can see, differences between the CUDA and the CPU `llama.cpp` back-end are larger than the differences between `llama.cpp` and `ik_llama.cpp`.