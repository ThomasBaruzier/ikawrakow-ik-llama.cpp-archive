## 📌 [Issue #747](https://github.com/ikawrakow/ik_llama.cpp/issues/747) - Bug: ggml-cuda.cu:116: CUDA error in commit [#741](https://github.com/ikawrakow/ik_llama.cpp/issues/741)

| **Author** | `ChicoPinto70` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-09-01 |
| **Updated** | 2025-09-18 |

---

## 📄 Description

### What happened?

The llama-server crashes at any prompt. The command line is:

CUDA_VISIBLE_DEVICES="1,2,0" ./build/bin/llama-server  --alias DeepSeek-V3.1-IQ4_KSS  -m /home/chico/.lmstudio/models/ubergarm/DeepSeek-V3.1-GGUF/DeepSeek-V3.1-smol-IQ4_KSS-00001-of-00007.gguf  -ngl 64 -c 65536 -mla 3 -fa -amb 512 -fmoe -t 28 -ctk q8_0 -ot "blk\.[0-6]\..*_exps\.=CUDA1,blk\.(7|8|9|10)\..*_exps\.=CUDA2,exps=CPU"  --parallel 1  --numa distribute -b 512 -ub 512 -ts 1,0,0 -ser 7,1 --host 192.168.0.9 --port 1235 --chat-template-kwargs '{"thinking": true}' --jinja 

And the traceback is in the log output.



### Name and Version

./build/bin/llama-server  --version
version: 3871 (d10d90ae)
built with cc (Ubuntu 13.3.0-6ubuntu2~24.04) 13.3.0 for x86_64-linux-gnu


### What operating system are you seeing the problem on?

_No response_

### Relevant log output

```shell
Grammar lazy: false
Chat format: DeepSeek R1
INFO [   launch_slot_with_task] slot is processing task | tid="140736901140480" timestamp=1756735747 id_slot=0 id_task=0
INFO [            update_slots] kv cache rm [p0, end) | tid="140736901140480" timestamp=1756735747 id_slot=0 id_task=0 p0=0
ggml_backend_sched_alloc_splits: failed to allocate graph, reserving (backend_ids_changed = 1)
CUDA error: an illegal memory access was encountered
  current device: 1, in function launch_mul_mat_q_id at /home/chico/ik_llama.cpp/ggml/src/ggml-cuda/template-instances/../mmq_id_common.cuh:3972
  cudaFuncSetAttribute((mul_mat_q_id<type, mmq_x, false>), cudaFuncAttributeMaxDynamicSharedMemorySize, nbytes_shared)
/home/chico/ik_llama.cpp/ggml/src/ggml-cuda.cu:116: CUDA error
[Detaching after fork from child process 84513]
Could not attach to process.  If your uid matches the uid of the target
process, check the setting of /proc/sys/kernel/yama/ptrace_scope, or try
again as the root user.  For more details, see /etc/sysctl.d/10-ptrace.conf
warning: process 81836 is already traced by process 81798
ptrace: Operation not permitted.
No stack.
The program is not being run.

Thread 1 "llama-server" received signal SIGABRT, Aborted.
Download failed: Invalid argument.  Continuing without source file ./nptl/./nptl/pthread_kill.c.
__pthread_kill_implementation (no_tid=0, signo=6, threadid=<optimized out>) at ./nptl/pthread_kill.c:44
warning: 44	./nptl/pthread_kill.c: No such file or directory
(gdb) backtrace
#0  __pthread_kill_implementation (no_tid=0, signo=6, threadid=<optimized out>) at ./nptl/pthread_kill.c:44
#1  __pthread_kill_internal (signo=6, threadid=<optimized out>) at ./nptl/pthread_kill.c:78
#2  __GI___pthread_kill (threadid=<optimized out>, signo=signo@entry=6) at ./nptl/pthread_kill.c:89
#3  0x00007fffdc84527e in __GI_raise (sig=sig@entry=6) at ../sysdeps/posix/raise.c:26
#4  0x00007fffdc8288ff in __GI_abort () at ./stdlib/abort.c:79
#5  0x00007fffdd0b554c in ggml_abort (file=0x7fffded61468 "/home/chico/ik_llama.cpp/ggml/src/ggml-cuda.cu", line=116, 
    fmt=0x7fffded6145d "CUDA error") at /home/chico/ik_llama.cpp/ggml/src/ggml.c:270
#6  0x00007fffdd3518ca in ggml_cuda_error (
    stmt=0x7fffdef67710 "cudaFuncSetAttribute((mul_mat_q_id<type, mmq_x, false>), cudaFuncAttributeMaxDynamicSharedMemorySize, nbytes_shared)", func=0x7fffdef676f6 "launch_mul_mat_q_id", 
    file=0x7fffdef63250 "/home/chico/ik_llama.cpp/ggml/src/ggml-cuda/template-instances/../mmq_id_common.cuh", line=3972, 
    msg=0x7fffdc497208 "an illegal memory access was encountered") at /home/chico/ik_llama.cpp/ggml/src/ggml-cuda.cu:116
#7  0x00007fffdda0cf73 in launch_mul_mat_q_id<(ggml_type)146, 64> (ctx=..., args=..., stream=0x5555562db680)
    at /home/chico/ik_llama.cpp/ggml/src/ggml-cuda/template-instances/../mmq_id_common.cuh:3972
#8  0x00007fffdda3411f in mul_mat_q_case_id<(ggml_type)146> (ctx=..., args=..., stream=0x5555562db680)
    at /home/chico/ik_llama.cpp/ggml/src/ggml-cuda/template-instances/../mmq_id_common.cuh:4116
#9  0x00007fffdd2891d2 in ggml_cuda_mul_mat_q_switch_type_id (ctx=..., args=..., stream=0x5555562db680)
    at /home/chico/ik_llama.cpp/ggml/src/ggml-cuda/mmq_id.cu:235
#10 0x00007fffdd28ac02 in ggml_cuda_mul_mat_q_id (ctx=..., src0=0x55555851bca0, src1=0x7fff8c4e6480, ids_tensor=0x7fff8c4e65f0, 
    dst=0x7fffffff9a10, ids_data=0xd02000000 "", src1_quantized_data=0xd04264900 "", is_next=false)
    at /home/chico/ik_llama.cpp/ggml/src/ggml-cuda/mmq_id.cu:516
#11 0x00007fffdd35db4f in ggml_cuda_moe_up_gate_unary (ctx=..., dst=0x7fff8d61e760, next=0x7fff8d61e8d0)
    at /home/chico/ik_llama.cpp/ggml/src/ggml-cuda.cu:2721
#12 0x00007fffdd360dde in ggml_cuda_compute_forward (ctx=..., dst=0x7fff8d61e760, next=0x7fff8d61e8d0, skip_next=@0x7fffffffa2e0: false)
    at /home/chico/ik_llama.cpp/ggml/src/ggml-cuda.cu:3154
#13 0x00007fffdd3629fa in evaluate_and_capture_cuda_graph (cuda_ctx=0x555556c0b710, cgraph=0x5555565e70d8, 
    graph_evaluated_or_captured=@0x7fffffffa348: false, use_cuda_graph=@0x7fffffffa33a: false, 
    cuda_graph_update_required=@0x7fffffffa33b: false) at /home/chico/ik_llama.cpp/ggml/src/ggml-cuda.cu:3592
#14 0x00007fffdd36320a in ggml_backend_cuda_graph_compute (backend=0x555555fcaa00, cgraph=0x5555565e70d8)
    at /home/chico/ik_llama.cpp/ggml/src/ggml-cuda.cu:3710
#15 0x00007fffdd110b8f in ggml_backend_graph_compute_async (backend=0x555555fcaa00, cgraph=0x5555565e70d8)
    at /home/chico/ik_llama.cpp/ggml/src/ggml-backend.c:317
#16 0x00007fffdd1156db in ggml_backend_sched_compute_splits (sched=0x5555564a57c0)
    at /home/chico/ik_llama.cpp/ggml/src/ggml-backend.c:1887
#17 0x00007fffdd1162ed in ggml_backend_sched_graph_compute_async (sched=0x5555564a57c0, graph=0x7fff8d40b030)
    at /home/chico/ik_llama.cpp/ggml/src/ggml-backend.c:2081
#18 0x00007ffff7d0888d in llama_graph_compute (lctx=..., gf=0x7fff8d40b030, n_threads=28) at /home/chico/ik_llama.cpp/src/llama.cpp:16405
#19 0x00007ffff7d09512 in llama_decode_internal (lctx=..., batch_all=...) at /home/chico/ik_llama.cpp/src/llama.cpp:16624
#20 0x00007ffff7d1c0e4 in llama_decode (ctx=0x555556c06dc0, batch=...) at /home/chico/ik_llama.cpp/src/llama.cpp:21141
#21 0x0000555555684f33 in server_context::update_slots (this=0x7fffffffc880) at /home/chico/ik_llama.cpp/examples/server/server.cpp:3149
#22 0x0000555555742db7 in std::__invoke_impl<void, void (server_context::*&)(), server_context*&> (
    __f=@0x555555ee3bb0: (void (server_context::*)(struct server_context * const)) 0x55555567efb8 <server_context::update_slots()>, 
    __t=@0x555555ee3bc0: 0x7fffffffc880) at /usr/include/c++/13/bits/invoke.h:74
#23 0x0000555555735a49 in std::__invoke<void (server_context::*&)(), server_context*&> (
    __fn=@0x555555ee3bb0: (void (server_context::*)(struct server_context * const)) 0x55555567efb8 <server_context::update_slots()>)
    at /usr/include/c++/13/bits/invoke.h:96
#24 0x0000555555723e4b in std::_Bind<void (server_context::*(server_context*))()>::__call<void, , 0ul>(std::tuple<>&&, std::_Index_tuple<0ul>) (this=0x555555ee3bb0, __args=...) at /usr/include/c++/13/functional:506
#25 0x00005555557150b3 in std::_Bind<void (server_context::*(server_context*))()>::operator()<, void>() (this=0x555555ee3bb0)
    at /usr/include/c++/13/functional:591
#26 0x0000555555700eda in std::__invoke_impl<void, std::_Bind<void (server_context::*(server_context*))()>&>(std::__invoke_other, std::_Bind<void (server_context::*(server_context*))()>&) (__f=...) at /usr/include/c++/13/bits/invoke.h:61
#27 0x00005555556ecd3a in std::__invoke_r<void, std::_Bind<void (server_context::*(server_context*))()>&>(std::_Bind<void (server_context::*(server_context*))()>&) (__fn=...) at /usr/include/c++/13/bits/invoke.h:111
#28 0x00005555556cea89 in std::_Function_handler<void (), std::_Bind<void (server_context::*(server_context*))()> >::_M_invoke(std::_Any_data const&) (__functor=...) at /usr/include/c++/13/bits/std_function.h:290
--Type <RET> for more, q to quit, c to continue without paging--
#29 0x000055555569283e in std::function<void ()>::operator()() const (this=0x7fffffffd6b0) at /usr/include/c++/13/bits/std_function.h:591
#30 0x000055555565c4c5 in server_queue::start_loop (this=0x7fffffffd5c8) at /home/chico/ik_llama.cpp/examples/server/server.cpp:1017
#31 0x00005555556210a7 in main (argc=40, argv=0x7fffffffd998) at /home/chico/ik_llama.cpp/examples/server/server.cpp:5138
(gdb)
```

---

## 💬 Conversation

👤 **ChicoPinto70** commented on **2025-09-01** at **14:25:59**

P.S. The same thing happens with:
CUDA_VISIBLE_DEVICES="1,2,0" ./build/bin/llama-server -m /home/chico/.lmstudio/models/ubergarm/Qwen3-Coder-480B-A35B-Instruct-GGUF/Qwen3-480B-A35B-Instruct-IQ4_K-00001-of-00006.gguf -ngl 64 -fa -fmoe -c 65536 -t 32 -tb 32 -ctk q8_0 -ctv q8_0 -ot "blk\.(0|1|2|3|4)\..*_exps\.=CUDA2,blk\.(5|6|7)\..*_exps\.=CUDA1,blk\.(8)\..*_exps\.=CUDA0,blk\..*_exps\.=CPU" --parallel 1  --numa distribute -b 1024 -ub 1024 --no-mmap  -ser 7,1 --host 192.168.0.9 --port 1235 -ts 1,0,0

BUT, the GLM 4.5 works fine!!!:
CUDA_VISIBLE_DEVICES="1,2,0" ./build/bin/llama-server  --alias Ubergarm/GLM-4.5-IQ5_K-GGUF  -m /home/chico/.lmstudio/models/ubergarm/GLM-4.5-IQ5_K-GGUF/GLM-4.5-IQ5_K-00001-of-00006.gguf  -ngl 99 -c 65536 -fa -amb 1024 -fmoe -t 32 -tb 32 -ctk q8_0 -ctv q8_0 -ot "blk.([3-9]|10)\..*_exps\.=CUDA2,blk.(11|12)\..*_exps\.=CUDA1,blk.(13|14)\..*_exps\.=CUDA0" -ot ".ffn_.*_exps.=CPU" --parallel 1  --numa distribute -b 2048 -ub 2048 --no-mmap --host 192.168.0.9 --port 1235 -ts 1,1,0

---

👤 **ikawrakow** commented on **2025-09-01** at **17:15:03**

Oh, `-ser` no longer works after [#728](https://github.com/ikawrakow/ik_llama.cpp/issues/728).

My concept is that SER didn't turn out to be useful, and nobody is really using it, so I didn't think it was a problem to break it in [#728](https://github.com/ikawrakow/ik_llama.cpp/issues/728). If there is still interest in using SER, I can try to make it work with [#728](https://github.com/ikawrakow/ik_llama.cpp/issues/728), but for now just don't use `-ser`.

---

👤 **ChicoPinto70** commented on **2025-09-01** at **18:36:50**

Oh, sorry! I didn't know that. I've used this parameter for just a small TG speed increase but, as it has an impact in the response quality, it doesn't worth it. 

I just test it without the ser and the server works fine. Thanks!!!

---

👤 **whoisjeremylam** commented on **2025-09-13** at **14:09:31**

> Oh, `-ser` no longer works after [[#728](https://github.com/ikawrakow/ik_llama.cpp/issues/728)](https://github.com/ikawrakow/ik_llama.cpp/pull/728).

Oh, I'll raise my hand that `-ser` is useful. I've been using `ser 7,1` quite successfully with Kimi K2. 

Seemingly, [perplexity isn't really affected](https://huggingface.co/ubergarm/Kimi-K2-Instruct-0905-GGUF/discussions/1#68be9f502434d2bac742dcf4).

It isn't a huge rush to get `-ser` working again but it has been useful to eek out a little more performance without dropping to a lower quant.

---

👤 **saood06** commented on **2025-09-18** at **19:34:24**

>Seemingly, [perplexity isn't really affected](https://huggingface.co/ubergarm/Kimi-K2-Instruct-0905-GGUF/discussions/1#68be9f502434d2bac742dcf4).

That is really interesting.

>Oh, I'll raise my hand that -ser is useful. I've been using ser 7,1 quite successfully with Kimi K2.

Well if you are using, X,1 you can try to use `--override-kv deepseek2.expert_used_count=int:X` (or whatever the correct override is) as that is equivalent, and see if that works (since you are just reducing expert count, not 'smartly' reducing it).