## 📌 [Issue #862](https://github.com/ikawrakow/ik_llama.cpp/issues/862) - Bug: Kimi K2 fails to load

| **Author** | `whoisjeremylam` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-10-25 |
| **Updated** | 2025-10-25 |

---

## 📄 Description

### What happened?

I `git pull`ed HEAD this morning.

Command to start `ik_llama.cpp`:
```
~/ik_llama.cpp/build/bin/llama-server \
  -t 23 \
  -m /home/ai/models/ubergarm/Kimi-K2-Instruct-0905-GGUF/Kimi-K2-Instruct-0905-IQ2_KS.gguf \
  --alias Kimi-K2 \
  --jinja \
  --host 0.0.0.0 \
  --port 5000 \
  -c 100000 -ctk q8_0 --no-mmap -ngl 999 \
  -ot "blk.(0|1|2|3|4|5|6|7).ffn.=CUDA0" \
  -ot "blk.(11|12|13|14|15|16|17).ffn.=CUDA1" \
  -ot "blk.(21|22).ffn.=CUDA2" \
  -ot "blk.(31|32).ffn.=CUDA3" \
  -ot exps=CPU \
  -fa -mg 0 -ub 4096 -b 4096 -mla 3 -fmoe -amb 512 \
  --temp 0.6
```

### Name and Version

```
$ ~/ik_llama.cpp/build/bin/llama-server --version
version: 3925 (483cea52)
built with cc (Ubuntu 11.4.0-1ubuntu1~22.04) 11.4.0 for x86_64-linux-gnu
```

### What operating system are you seeing the problem on?

Linux

### Relevant log output

```shell
XXXXXXXXXXXXXXXXXXXXX Setting only active experts offload
/home/ai/ik_llama.cpp/build/bin/llama-server: symbol lookup error: /home/ai/ik_llama.cpp/build/ggml/src/libggml.so: undefined symbol: _Z26ggml_cuda_op_mul_multi_addR25ggml_backend_cuda_contextP11ggml_tensor
```

---

## 💬 Conversation

👤 **whoisjeremylam** commented on **2025-10-25** at **02:11:18**

I've gone back to [commit hash](https://github.com/ikawrakow/ik_llama.cpp/commit/483cea527d137e4888848d985073a5a8c529565b) `483cea527d137e4888848d985073a5a8c529565b` which doesn't have this problem

---

👤 **ikawrakow** commented on **2025-10-25** at **05:43:03**

You are sure you rebuild the project correctly?

---

👤 **whoisjeremylam** commented on **2025-10-25** at **08:42:54**

Sorry, it wasn't built properly.

For whatever reason, I had to run `cmake` with the build options and the release again.

---

👤 **ikawrakow** commented on **2025-10-25** at **08:53:22**

> For whatever reason, I had to run cmake with the build options and the release again

The reason is that the CUDA-related portion of `CMakeLists.txt` is not written to explicitly add the necessary `.cu` files, but instead uses `GLOB` in the `ggml-cuda` folder. When a new file is added (as in this case), one needs to rerun `cmake` to generate the correct list of CUDA files. This is inherited from `llama.cpp` (and `llama.cpp` still behaves the same way in this regard). I realize that this can lead to hiccups, but otherwise the list of CUDA files listed in  `CMakeLists.txt`  would be huge.

The strange thing is of course that the build somehow succeeded in your case. Normally I expect you to get a link error with the missing function.

---

👤 **whoisjeremylam** commented on **2025-10-25** at **08:55:27**

That makes sense. Thank you. I'll be more careful next time!