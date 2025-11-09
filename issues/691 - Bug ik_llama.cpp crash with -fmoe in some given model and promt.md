## 📌 [Issue #691](https://github.com/ikawrakow/ik_llama.cpp/issues/691) - Bug: ik_llama.cpp crash with -fmoe in some given model and promt

| **Author** | `oovloveme` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-08-15 |
| **Updated** | 2025-08-21 |

---

## 📄 Description

### What happened?

### **Bug: ik_llama.cpp crash with -fmoe in some given model and promt**

model:

/home/ee/ik_llama.cpp/ik_llama.cpp/build/bin/llama-server \
    --model /data/models/Qwen3-30B-A3B-Thinking-2507-UD-Q8_K_XL/Qwen3-30B-A3B-Thinking-2507-UD-Q8_K_XL.gguf \
    --alias 30b \
    -c 80000 \
    -mla 3 -fa -amb 512 \
    --host 0.0.0.0 --port 8000 \
    --parallel 1  --threads 56 \
    -ngl 99 -ot ".ffn_.*_exps."=CPU  -fmoe


prompt:

有一个圆，圆的内部空间里有n个点，请计算这n个点均落在同一个半圆内的概率，并给出详细推导和计算步骤

### **When removing -fmoe function with the same question and prompt, the output were all right:**

/home/ee/ik_llama.cpp/ik_llama.cpp/build/bin/llama-server \
    --model /data/models/Qwen3-30B-A3B-Thinking-2507-UD-Q8_K_XL/Qwen3-30B-A3B-Thinking-2507-UD-Q8_K_XL.gguf \
    --alias 30b \
    -c 80000 \
    -mla 3 -fa -amb 512 \
    --host 0.0.0.0 --port 8000 \
    --parallel 1  --threads 56 \
    -ngl 99 -ot ".ffn_.*_exps."=CPU

### Name and Version

/home/ee/ik_llama.cpp/ik_llama.cpp/build/bin/llama-server --version
version: 3838 (fc06bc9d)
built with cc (Ubuntu 13.3.0-6ubuntu2~24.04) 13.3.0 for x86_64-linux-gnu

### What operating system are you seeing the problem on?

Ubuntu 24.04
cuda 13.0
epyc 7b13 with 512g ddr4 ram and 5070ti

### Relevant log output

```shell
INFO [   launch_slot_with_task] slot is processing task | tid="135066340544512" timestamp=1755243984 id_slot=0 id_task=0
INFO [            update_slots] kv cache rm [p0, end) | tid="135066340544512" timestamp=1755243984 id_slot=0 id_task=0 p0=0
/home/ee/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml.c:15618: fatal error
/home/ee/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml.c:15618: fatal error/home/ee/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml.c:15618: /home/ee/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml.c:15618:
/home/ee/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml.c:15618: fatal error
/home/ee/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml.c:15618: /home/ee/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml.c:15618: /home/ee/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml.c:15618: /home/ee/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml.c:15618: /home/ee/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml.c:15618: fatal errorfatal error
/home/ee/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml.c:15618: fatal error
/home/ee/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml.c:15618: fatal error
/home/ee/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml.c:15618: /home/ee/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml.c:15618: /home/ee/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml.c:15618: fatal error
/home/ee/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml.c:15618: fatal error
/home/ee/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml.c:15618: fatal error
/home/ee/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml.c:15618: fatal error
/home/ee/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml.c:15618: fatal error
/home/ee/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml.c:15618: fatal error
/home/ee/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml.c:15618: fatal error
/home/ee/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml.c:15618: fatal error
/home/ee/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml.c:15618: fatal error
/home/ee/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml.c:15618: fatal error
/home/ee/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml.c:15618: fatal error
/home/ee/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml.c:15618: fatal error
/home/ee/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml.c:15618: fatal error
fatal error
/home/ee/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml.c:15618: fatal error
/home/ee/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml.c:15618: fatal error
/home/ee/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml.c:15618: fatal error
/home/ee/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml.c:15618: fatal error
/home/ee/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml.c:15618: fatal error
/home/ee/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml.c:15618: fatal error
/home/ee/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml.c:15618: fatal error
/home/ee/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml.c:15618: fatal error
fatal error
/home/ee/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml.c:15618: fatal error
/home/ee/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml.c:15618: fatal error
/home/ee/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml.c:15618: fatal error
/home/ee/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml.c:15618: fatal error
/home/ee/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml.c:15618: fatal error
/home/ee/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml.c:15618: fatal error
/home/ee/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml.c:15618: fatal error
/home/ee/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml.c:15618: fatal error
fatal error
/home/ee/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml.c:15618: fatal error
/home/ee/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml.c:15618: fatal error
/home/ee/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml.c:15618: fatal error

/home/ee/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml.c:15618: fatal error
fatal error
/home/ee/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml.c:15618: fatal error
fatal error
/home/ee/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml.c:15618: fatal error
fatal error
/home/ee/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml.c:15618: fatal error
/home/ee/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml.c:15618: fatal error
/home/ee/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml.c:15618: fatal error
/home/ee/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml.c:15618: fatal error
/home/ee/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml.c:15618: fatal error
fatal error
/home/ee/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml.c:15618: fatal error
/home/ee/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml.c:15618: fatal error
Could not attach to process.  If your uid matches the uid of the target
Could not attach to process.  If your uid matches the uid of the target
process, check the setting of /proc/sys/kernel/yama/ptrace_scope, or try
process, check the setting of /proc/sys/kernel/yama/ptrace_scope, or try
again as the root user.  For more details, see /etc/sysctl.d/10-ptrace.conf
again as the root user.  For more details, see /etc/sysctl.d/10-ptrace.conf
Could not attach to process.  If your uid matches the uid of the target
process, check the setting of /proc/sys/kernel/yama/ptrace_scope, or try
again as the root user.  For more details, see /etc/sysctl.d/10-ptrace.conf
ptrace: Operation not permitted.ptrace: Operation not permitted.ptrace: Operation not permitted.


No stack.No stack.
No stack.

The program is not being run.The program is not being run.
The program is not being run.

Could not attach to process.  If your uid matches the uid of the target
process, check the setting of /proc/sys/kernel/yama/ptrace_scope, or try
again as the root user.  For more details, see /etc/sysctl.d/10-ptrace.conf
ptrace: Operation not permitted.
No stack.
The program is not being run.
Could not attach to process.  If your uid matches the uid of the target
process, check the setting of /proc/sys/kernel/yama/ptrace_scope, or try
again as the root user.  For more details, see /etc/sysctl.d/10-ptrace.conf
ptrace: Operation not permitted.
No stack.
The program is not being run.
Could not attach to process.  If your uid matches the uid of the target
process, check the setting of /proc/sys/kernel/yama/ptrace_scope, or try
again as the root user.  For more details, see /etc/sysctl.d/10-ptrace.conf
ptrace: Operation not permitted.
No stack.
The program is not being run.
Could not attach to process.  If your uid matches the uid of the target
process, check the setting of /proc/sys/kernel/yama/ptrace_scope, or try
again as the root user.  For more details, see /etc/sysctl.d/10-ptrace.conf
ptrace: Operation not permitted.
No stack.
The program is not being run.
Could not attach to process.  If your uid matches the uid of the target
process, check the setting of /proc/sys/kernel/yama/ptrace_scope, or try
again as the root user.  For more details, see /etc/sysctl.d/10-ptrace.conf
ptrace: Operation not permitted.
No stack.
The program is not being run.
Could not attach to process.  If your uid matches the uid of the target
process, check the setting of /proc/sys/kernel/yama/ptrace_scope, or try
again as the root user.  For more details, see /etc/sysctl.d/10-ptrace.conf
ptrace: Operation not permitted.
No stack.
The program is not being run.
Aborted (core dumped)
```

---

## 💬 Conversation

👤 **ikawrakow** commented on **2025-08-19** at **16:08:14**

Thanks for the bug report. It should be fixed by [#708](https://github.com/ikawrakow/ik_llama.cpp/issues/708)

---

👤 **ikawrakow** commented on **2025-08-19** at **17:00:25**

@oovloveme You may want to try [#709](https://github.com/ikawrakow/ik_llama.cpp/issues/709). This will give you better performance for the model you are using when prompts ae not long enough to be offloading to the GPU.

---

👤 **oovloveme** commented on **2025-08-21** at **10:03:25**

> Thanks for the bug report. It should be fixed by [[#708](https://github.com/ikawrakow/ik_llama.cpp/issues/708)](https://github.com/ikawrakow/ik_llama.cpp/pull/708)

Thanks. It's solved.