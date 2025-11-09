## 📌 [Issue #704](https://github.com/ikawrakow/ik_llama.cpp/issues/704) - Bug: Assert/Crash as PP ctx size exceeds threshold (PR[#689](https://github.com/ikawrakow/ik_llama.cpp/issues/689) Regression)

| **Author** | `usrlocalben` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-08-19 |
| **Updated** | 2025-08-19 |

---

## 📄 Description

### What happened?

During PP phase, assert/crash occurs when ctx size reaches 32768.

```
...
INFO [            update_slots] kv cache rm [p0, end) | tid="139920049950720" timestamp=1755555078 id_slot=0 id_task=6 p0=18432
INFO [            update_slots] kv cache rm [p0, end) | tid="139920049950720" timestamp=1755555172 id_slot=0 id_task=6 p0=20480
INFO [            update_slots] kv cache rm [p0, end) | tid="139920049950720" timestamp=1755555267 id_slot=0 id_task=6 p0=22528
INFO [            update_slots] kv cache rm [p0, end) | tid="139920049950720" timestamp=1755555362 id_slot=0 id_task=6 p0=24576
INFO [            update_slots] kv cache rm [p0, end) | tid="139920049950720" timestamp=1755555457 id_slot=0 id_task=6 p0=26624
INFO [            update_slots] kv cache rm [p0, end) | tid="139920049950720" timestamp=1755555554 id_slot=0 id_task=6 p0=28672
INFO [            update_slots] kv cache rm [p0, end) | tid="139920049950720" timestamp=1755555650 id_slot=0 id_task=6 p0=30720
INFO [            update_slots] kv cache rm [p0, end) | tid="139920049950720" timestamp=1755555748 id_slot=0 id_task=6 p0=32768
/path/to/ik_llama.cpp/ggml/src/ggml-cuda/cpy.cu:380: GGML_ASSERT(ggml_nbytes(src0) <= INT_MAX) failed
```

I bisected using `d60c8f4d` as a known-good and reached the commit for CUDA graphs / gpt-oss, pr [#689](https://github.com/ikawrakow/ik_llama.cpp/issues/689) .

The triggering assert/line is here:https://github.com/ikawrakow/ik_llama.cpp/commit/fc06bc9d2720efbb2bfaab25114ca51cfb39a41a#diff-0ad629544d728ec8bf0e035d0250c5e382c669db9429750db9296dab14c68794R380

Curiously, the assert statements were previously disabled with "These asserts appear to be unnecessary." and the PR enabled them.

Tested with and without `-DGGML_CUDA_USER_GRAPHS=OFF` -- same result.
Tested with 2048/4096 batch size -- same result.
 
I added the debug printf that was previously commented, and get
```
ggml_cuda_cpy: k_nope_f32-0 has 2415918592 bytes
```

I see that _k_nope_f32_ is part of MLA>1 computation.

If I remove the assertions, prefill completes without error and I observe reasonable looking output during TG. (i.e. not gibberish nor other badness)

Model is Kimi-K2, Q8_0.
Hardware is 2S EPYC 9115 + RTX 8000 (Turing), CUDA 12.6

server invocation:
```
-op 26,0,27,0,29,0
-b 4096 -ub 4096
-mla 2 -fa -fmoe
-c 131072
-ngl 999 -ot exps=CPU
--slot-save-path ${slotDir}/K2
-m ${modelDir}/k2/k2-Q8_0-00001-of-00023.gguf
--temp 0.6 --min-p 0.01
```

### Name and Version

version: 3838 (fc06bc9d)
built with cc (Debian 12.2.0-14+deb12u1) 12.2.0 for x86_64-linux-gnu

### What operating system are you seeing the problem on?

Linux