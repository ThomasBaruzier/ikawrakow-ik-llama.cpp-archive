## 📌 [Issue #849](https://github.com/ikawrakow/ik_llama.cpp/issues/849) - Bug: segfault with -ger enabled

| **Author** | `magikRUKKOLA` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-10-21 |
| **Updated** | 2025-10-21 |

---

## 📄 Description

### What happened?

teleported from: https://github.com/ikawrakow/ik_llama.cpp/pull/838#issuecomment-3424371949

As related to the segfault.

```
Thread 1 "llama-server" received signal SIGSEGV, Segmentation fault.
0x00007fffe11b3d36 in cuda_bailingmoev2_experts(ggml_backend_cuda_context&, ggml_tensor*, ggml_tensor*) ()
   from /opt/ik_llama.cpp/ik_llama.cpp/build/ggml/src/libggml.so
```

```
(gdb) thread apply all bt

Thread 6 (Thread 0x7f87e7fff000 (LWP 2402202) "cuda-EvtHandlr"):
#0  0x00007fffe0aa49ee in ?? () from /lib/x86_64-linux-gnu/libc.so.6
#1  0x00007fffe0a99668 in ?? () from /lib/x86_64-linux-gnu/libc.so.6
#2  0x00007fffe0a996ad in ?? () from /lib/x86_64-linux-gnu/libc.so.6
#3  0x00007fffe0b0d9c6 in poll () from /lib/x86_64-linux-gnu/libc.so.6
#4  0x00007fffd731cb37 in ?? () from /lib/x86_64-linux-gnu/libcuda.so.1
#5  0x00007fffd740bd77 in ?? () from /lib/x86_64-linux-gnu/libcuda.so.1
#6  0x00007fffd7308883 in ?? () from /lib/x86_64-linux-gnu/libcuda.so.1
#7  0x00007fffe0a9cb7b in ?? () from /lib/x86_64-linux-gnu/libc.so.6
#8  0x00007fffe0b1a7b8 in ?? () from /lib/x86_64-linux-gnu/libc.so.6

Thread 5 (Thread 0x7f89a4ddc000 (LWP 2402201) "llama-server"):
#0  0x00007fffe0aa49ee in ?? () from /lib/x86_64-linux-gnu/libc.so.6
#1  0x00007fffe0a99668 in ?? () from /lib/x86_64-linux-gnu/libc.so.6
#2  0x00007fffe0a99c9c in ?? () from /lib/x86_64-linux-gnu/libc.so.6
#3  0x00007fffe0a9c31d in pthread_cond_timedwait () from /lib/x86_64-linux-gnu/libc.so.6
#4  0x00007fffd7269b5a in ?? () from /lib/x86_64-linux-gnu/libcuda.so.1
#5  0x00007fffd7308883 in ?? () from /lib/x86_64-linux-gnu/libcuda.so.1
#6  0x00007fffe0a9cb7b in ?? () from /lib/x86_64-linux-gnu/libc.so.6
#7  0x00007fffe0b1a7b8 in ?? () from /lib/x86_64-linux-gnu/libc.so.6

Thread 4 (Thread 0x7f89a67fe000 (LWP 2402200) "cuda-EvtHandlr"):
#0  0x00007fffe0aa49ee in ?? () from /lib/x86_64-linux-gnu/libc.so.6
#1  0x00007fffe0a99668 in ?? () from /lib/x86_64-linux-gnu/libc.so.6
#2  0x00007fffe0a996ad in ?? () from /lib/x86_64-linux-gnu/libc.so.6
#3  0x00007fffe0b0d9c6 in poll () from /lib/x86_64-linux-gnu/libc.so.6
#4  0x00007fffd731cb37 in ?? () from /lib/x86_64-linux-gnu/libcuda.so.1
#5  0x00007fffd740bd77 in ?? () from /lib/x86_64-linux-gnu/libcuda.so.1
#6  0x00007fffd7308883 in ?? () from /lib/x86_64-linux-gnu/libcuda.so.1
#7  0x00007fffe0a9cb7b in ?? () from /lib/x86_64-linux-gnu/libc.so.6
#8  0x00007fffe0b1a7b8 in ?? () from /lib/x86_64-linux-gnu/libc.so.6

Thread 3 (Thread 0x7f89a6fff000 (LWP 2402199) "llama-server"):
#0  0x00007fffe0aa49ee in ?? () from /lib/x86_64-linux-gnu/libc.so.6
#1  0x00007fffe0a99668 in ?? () from /lib/x86_64-linux-gnu/libc.so.6
#2  0x00007fffe0a99c9c in ?? () from /lib/x86_64-linux-gnu/libc.so.6
#3  0x00007fffe0a9c31d in pthread_cond_timedwait () from /lib/x86_64-linux-gnu/libc.so.6
#4  0x00007fffd7269b5a in ?? () from /lib/x86_64-linux-gnu/libcuda.so.1
#5  0x00007fffd7308883 in ?? () from /lib/x86_64-linux-gnu/libcuda.so.1
#6  0x00007fffe0a9cb7b in ?? () from /lib/x86_64-linux-gnu/libc.so.6
#7  0x00007fffe0b1a7b8 in ?? () from /lib/x86_64-linux-gnu/libc.so.6

Thread 2 (Thread 0x7fffb13ff000 (LWP 2402120) "cuda00001400006"):
#0  0x00007fffe0aa49ee in ?? () from /lib/x86_64-linux-gnu/libc.so.6
#1  0x00007fffe0a99668 in ?? () from /lib/x86_64-linux-gnu/libc.so.6
#2  0x00007fffe0a996ad in ?? () from /lib/x86_64-linux-gnu/libc.so.6
#3  0x00007fffe0b0d9c6 in poll () from /lib/x86_64-linux-gnu/libc.so.6
#4  0x00007fffd731cb37 in ?? () from /lib/x86_64-linux-gnu/libcuda.so.1
#5  0x00007fffd740bd77 in ?? () from /lib/x86_64-linux-gnu/libcuda.so.1
#6  0x00007fffd7308883 in ?? () from /lib/x86_64-linux-gnu/libcuda.so.1
#7  0x00007fffe0a9cb7b in ?? () from /lib/x86_64-linux-gnu/libc.so.6
#8  0x00007fffe0b1a7b8 in ?? () from /lib/x86_64-linux-gnu/libc.so.6

Thread 1 (Thread 0x7ffff7d3a000 (LWP 2402116) "llama-server"):
#0  0x00007fffe11b3d36 in cuda_bailingmoev2_experts(ggml_backend_cuda_context&, ggml_tensor*, ggml_tensor*) () from /opt/ik_llama.cpp/ik_llama.cpp/build/ggml/src/libggml.so
#1  0x00007fffe1376118 in ggml_backend_cuda_graph_compute(ggml_backend*, ggml_cgraph*) () from /opt/ik_llama.cpp/ik_llama.cpp/build/ggml/src/libggml.so
#2  0x00007fffe116ffa2 in ggml_backend_sched_graph_compute_async () from /opt/ik_llama.cpp/ik_llama.cpp/build/ggml/src/libggml.so
#3  0x00007ffff7e6d28c in llama_decode () from /opt/ik_llama.cpp/ik_llama.cpp/build/src/libllama.so
#4  0x00005555556fab08 in llama_init_from_gpt_params(gpt_params&) ()
#5  0x0000555555610d63 in server_context::load_model(gpt_params const&) ()
#6  0x000055555559fd70 in main ()

```

```
(gdb) bt full
#0  0x00007fffe11b3d36 in cuda_bailingmoev2_experts(ggml_backend_cuda_context&, ggml_tensor*, ggml_tensor*) ()
   from /opt/ik_llama.cpp/ik_llama.cpp/build/ggml/src/libggml.so
No symbol table info available.
#1  0x00007fffe1376118 in ggml_backend_cuda_graph_compute(ggml_backend*, ggml_cgraph*) ()
   from /opt/ik_llama.cpp/ik_llama.cpp/build/ggml/src/libggml.so
No symbol table info available.
#2  0x00007fffe116ffa2 in ggml_backend_sched_graph_compute_async ()
   from /opt/ik_llama.cpp/ik_llama.cpp/build/ggml/src/libggml.so
No symbol table info available.
#3  0x00007ffff7e6d28c in llama_decode () from /opt/ik_llama.cpp/ik_llama.cpp/build/src/libllama.so
No symbol table info available.
#4  0x00005555556fab08 in llama_init_from_gpt_params(gpt_params&) ()
No symbol table info available.
#5  0x0000555555610d63 in server_context::load_model(gpt_params const&) ()
No symbol table info available.
#6  0x000055555559fd70 in main ()
No symbol table info available.
```

```
(gdb) info registers
rax            0x1                 1
rbx            0x7f93c64580f0      140272663363824
rcx            0x1                 1
rdx            0x1                 1
rsi            0x100               256
rdi            0x0                 0
rbp            0x7fffffffa6d0      0x7fffffffa6d0
rsp            0x7fffffffa520      0x7fffffffa520
r8             0x555555ebe610      93825002104336
r9             0x0                 0
r10            0x7fffffffa6f0      140737488332528
r11            0x7fffe11b3c90      140736970046608
r12            0x0                 0
r13            0x7f93c64586b0      140272663365296
r14            0x7f93c6458820      140272663365664
r15            0x555556c7cc80      93825016515712
rip            0x7fffe11b3d36      0x7fffe11b3d36 <cuda_bailingmoev2_experts(ggml_backend_cuda_context&, ggml_tensor*, ggml_tensor*)+166>
eflags         0x10202             [ IF RF ]
cs             0x33                51
ss             0x2b                43
ds             0x0                 0
es             0x0                 0
fs             0x0                 0
gs             0x0                 0
fs_base        0x7ffff7d3a000      140737351229440
gs_base        0x0                 0
```

```
(gdb) x/10i $pc
=> 0x7fffe11b3d36 <_Z25cuda_bailingmoev2_expertsR25ggml_backend_cuda_contextP11ggml_tensorS2_+166>:
    cmpq   $0x1,0x18(%r12)
   0x7fffe11b3d3c <_Z25cuda_bailingmoev2_expertsR25ggml_backend_cuda_contextP11ggml_tensorS2_+172>:
    jne    0x7fffe11b45cf <_Z25cuda_bailingmoev2_expertsR25ggml_backend_cuda_contextP11ggml_tensorS2_+2367>
   0x7fffe11b3d42 <_Z25cuda_bailingmoev2_expertsR25ggml_backend_cuda_contextP11ggml_tensorS2_+178>:
    mov    -0x168(%rbp),%rax
   0x7fffe11b3d49 <_Z25cuda_bailingmoev2_expertsR25ggml_backend_cuda_contextP11ggml_tensorS2_+185>:
    mov    0x10(%rax),%rax
   0x7fffe11b3d4d <_Z25cuda_bailingmoev2_expertsR25ggml_backend_cuda_contextP11ggml_tensorS2_+189>:
    cmp    %rax,0x10(%r12)
   0x7fffe11b3d52 <_Z25cuda_bailingmoev2_expertsR25ggml_backend_cuda_contextP11ggml_tensorS2_+194>:
    jne    0x7fffe11b45ae <_Z25cuda_bailingmoev2_expertsR25ggml_backend_cuda_contextP11ggml_tensorS2_+2334>
   0x7fffe11b3d58 <_Z25cuda_bailingmoev2_expertsR25ggml_backend_cuda_contextP11ggml_tensorS2_+200>:
    mov    -0x190(%rbp),%rcx
   0x7fffe11b3d5f <_Z25cuda_bailingmoev2_expertsR25ggml_backend_cuda_contextP11ggml_tensorS2_+207>:
    movslq -0x180(%rbp),%rax
   0x7fffe11b3d66 <_Z25cuda_bailingmoev2_expertsR25ggml_backend_cuda_contextP11ggml_tensorS2_+214>:
    cmp    0x18(%rcx),%rax
   0x7fffe11b3d6a <_Z25cuda_bailingmoev2_expertsR25ggml_backend_cuda_contextP11ggml_tensorS2_+218>:
    jne    0x7fffe11b46bc <_Z25cuda_bailingmoev2_expertsR25ggml_backend_cuda_contextP11ggml_tensorS2_+2604>
```

[EDIT]:

so its null-pointer dereference.  the following patch proves (that is, no crash occurs but the data is a garbage) that something is wrong with topk.  the src is placed with 0x18 offset, right?

ggml/src/ggml-cuda/argsort.cu
```
476 void cuda_bailingmoev2_experts(ggml_backend_cuda_context & ctx, ggml_tensor * dst, ggml_tensor * topk) {  
477     if (!topk || !dst || !topk->src[0] || !topk->src[0]->src[0] ||                                                      
478         !topk->src[0]->src[0]->src[0] || !topk->src[0]->src[1]) {                                                       
479         return;                                                                                                         
480     } 
```

### Name and Version

/opt/ik_llama.cpp/ik_llama.cpp/build/bin/llama-server --version
version: 3919 (5ae87f6cd)
built with cc (Debian 15.2.0-4) 15.2.0 for x86_64-linux-gnu

### What operating system are you seeing the problem on?

_No response_

### Relevant log output

```shell
/opt/ik_llama.cpp/ik_llama.cpp/build/bin/llama-server \
    --model /opt/ubergarm/Ling-1T-GGUF/smol-IQ4-KSS/Ling-1T-smol-IQ4_KSS-00001-of-00011.gguf \
    --alias ubergarm/Ling-1T-smol-IQ4_KSS \
    --ctx-size $((64 * 1024)) \
    -b $((16 * 512)) -ub $((16 * 512)) \
    --mlock \
    --temp 0.7 --top-k 0 --top-p 0.95 --min-p 0.1 --repeat-penalty 1.1 \
    -ctk q8_0 -ctv q8_0 \
    -fa \
    -fmoe \
    -ger \
    -amb 512 \
    --split-mode layer \
    --tensor-split 32,40 \
    --main-gpu 1 \
    --override-tensor exps=CPU \
    -no-ooae \
    --n-gpu-layers 99 \
    --threads $(grep ^cpu\\scores /proc/cpuinfo | uniq | awk '{print $4}' | xargs -I{} echo "{}-0" | bc) \
    --host 0.0.0.0 \
    --port 8080 \
    --log-enable \
    --logdir /var/log/ \
    --jinja \
    --special \
    --verbosity 2 \
    --verbose-prompt \
    --reasoning-format auto \
    --prompt-cache "$HOME/.cache/ik_llama.cpp/prompt-cache.bin" --prompt-cache-all \
    --slot-save-path "$HOME/.cache/ik_llama.cpp/slot.bin" \
    --lookup-cache-dynamic "$HOME/.cache/ik_llama.cpp/slot.bin" \
    --keep -1 \
    --slot-prompt-similarity 0.35 \
    --metrics
```

---

## 💬 Conversation

👤 **ikawrakow** commented on **2025-10-21** at **05:04:45**

@magikRUKKOLA Thanks for the diagnosis.

Can you run with [#850](https://github.com/ikawrakow/ik_llama.cpp/issues/850) and post the output? Thanks.

---

👤 **magikRUKKOLA** commented on **2025-10-21** at **06:09:38**

@ikawrakow 

```
null bias  in cuda_bailingmoev2_experts
top_src is ffn_moe_probs-4 (reshaped), RESHAPE  dst is ffn_moe_weights-4, GET_ROWS
/opt/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml-cuda/argsort.cu:494: null probs/bias in cuda_bailingmoev2_experts

[New LWP 2528659]
[New LWP 2528658]
[New LWP 2528657]
[New LWP 2528656]
[New LWP 2528655]
[New LWP 2528583]
[New LWP 2528582]
[New LWP 2528581]
[New LWP 2528580]
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".
0x00007fd3ae8a49ee in ?? () from /lib/x86_64-linux-gnu/libc.so.6
#0  0x00007fd3ae8a49ee in ?? () from /lib/x86_64-linux-gnu/libc.so.6
#1  0x00007fd3ae899668 in ?? () from /lib/x86_64-linux-gnu/libc.so.6
#2  0x00007fd3ae8996ad in ?? () from /lib/x86_64-linux-gnu/libc.so.6
#3  0x00007fd3ae904787 in wait4 () from /lib/x86_64-linux-gnu/libc.so.6
#4  0x00007fd3aef10ae8 in ggml_abort () from /opt/ik_llama.cpp/ik_llama.cpp/build/ggml/src/libggml.so
#5  0x00007fd3aefb4767 in cuda_bailingmoev2_experts(ggml_backend_cuda_context&, ggml_tensor*, ggml_tensor*) () from /opt/ik_llama.cpp/ik_llama.cpp/build/ggml/src/libggml.so
#6  0x00007fd3af176298 in ggml_backend_cuda_graph_compute(ggml_backend*, ggml_cgraph*) () from /opt/ik_llama.cpp/ik_llama.cpp/build/ggml/src/libggml.so
#7  0x00007fd3aef6ffa2 in ggml_backend_sched_graph_compute_async () from /opt/ik_llama.cpp/ik_llama.cpp/build/ggml/src/libggml.so
#8  0x00007fd3c5cb028c in llama_decode () from /opt/ik_llama.cpp/ik_llama.cpp/build/src/libllama.so
#9  0x0000562d412a2b08 in llama_init_from_gpt_params(gpt_params&) ()
#10 0x0000562d411b8d63 in server_context::load_model(gpt_params const&) ()
#11 0x0000562d41147d70 in main ()
[Inferior 1 (process 2528579) detached]

```

---

👤 **ikawrakow** commented on **2025-10-21** at **06:34:57**

Thanks! Does [#851](https://github.com/ikawrakow/ik_llama.cpp/issues/851) fix it?

---

👤 **magikRUKKOLA** commented on **2025-10-21** at **07:01:51**

> Thanks! Does [[#851](https://github.com/ikawrakow/ik_llama.cpp/issues/851)](https://github.com/ikawrakow/ik_llama.cpp/pull/851) fix it?

@ikawrakow 

yes, it does