## 📌 [Issue #659](https://github.com/ikawrakow/ik_llama.cpp/issues/659) - Bug: imatrix quantization failing for nvidia Nemotron 49B v1.5

| **Author** | `erazortt` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-07-28 |
| **Updated** | 2025-08-22 |

---

## 📄 Description

### What happened?

Trying to quantize the bf16 gguf and also using the imatrix file from here here: https://huggingface.co/bartowski/nvidia_Llama-3_3-Nemotron-Super-49B-v1_5-GGUF/tree/main

quantizing with the following command:
./llama-quantize --imatrix ../models/nvidia_Llama-3_3-Nemotron-Super-49B-v1_5-imatrix.gguf --allow-requantize --output-tensor-type q8_0 --token-embedding-type q8_0 ../models/nvidia_Llama-3_3-Nemotron-Super-49B-v1_5-bf16-00001-of-00003.gguf model-q4_k_l.gguf q4_k_m

fails with the error message:
load_imatrix: failed reading data for entry 1

quantization works when not using the imatrix flag 

### Name and Version

version: 1 (bb4c917)
built with MSVC 19.44.35213.0 for

### What operating system are you seeing the problem on?

Windows

### Relevant log output

```shell
load_imatrix: failed reading data for entry 1
```

---

## 💬 Conversation

👤 **ubergarm** commented on **2025-07-31** at **21:25:01**

@erazortt 

So given bartowski's imatrix is in the new `.gguf` format only recently added in mainline I don't think that will work. You might be able to use mainlines feature to convert it from `.gguf` format to the `.dat` format used here and on mainline until a week ago.

I'm curious if you do convert it to `.dat` if you can get it to work. You will have to find the closed PR on mainline by compilade to read up on it, dont' have the reference handy atm.

Keep us posted!

---

👤 **compilade** commented on **2025-07-31** at **21:38:40**

@erazortt You should be able to use mainline `llama-imatrix` to convert `nvidia_Llama-3_3-Nemotron-Super-49B-v1_5-imatrix.gguf` to the `.dat` format:

```console
$ ./llama-imatrix --in-file nvidia_Llama-3_3-Nemotron-Super-49B-v1_5-imatrix.gguf --output-format dat -o nvidia_Llama-3_3-Nemotron-Super-49B-v1_5-imatrix.dat
```

Note the `.dat` of the output file, and `--output-format dat` (since <https://github.com/ggml-org/llama.cpp/pull/14842>).

See also <https://github.com/ggml-org/llama.cpp/pull/9400#issuecomment-3066040107>.

Then you should be able to use the resulting `.dat` imatrix file with `llama-quantize` from `ik_llama.cpp` (which is what you're using here).

---

👤 **erazortt** commented on **2025-08-01** at **15:19:19**

Yes the suggested conversion of the imatrix worked.

---

👤 **ikawrakow** commented on **2025-08-22** at **16:39:20**

GGUF imatrix confusion -> closing.