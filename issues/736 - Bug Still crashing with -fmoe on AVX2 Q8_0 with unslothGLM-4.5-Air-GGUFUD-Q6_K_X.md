## 📌 [Issue #736](https://github.com/ikawrakow/ik_llama.cpp/issues/736) - Bug: Still crashing with -fmoe on AVX2 Q8_0 with unsloth/GLM-4.5-Air-GGUF/UD-Q6_K_XL/GLM-4.5-Air-UD-Q6_K_XL-00001-of-00003.gguf

| **Author** | `os360` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-08-27 |
| **Updated** | 2025-08-28 |

---

## 📄 Description

### What happened?


AI/ikawrakow/ik_llama.cpp/ggml/src/ggml.c:15292: fatal error
/AI/ikawrakow/ik_llama.cpp/ggml/src/ggml.c:15292: fatal error
/AI/ikawrakow/ik_llama.cpp/ggml/src/ggml.c:15292: /home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/ggml.c:15292: fatal error

Using host libthread_db library "/usr/lib64/libthread_db.so.1".
0x00007f52a680fc92 in ?? () from /usr/lib64/libc.so.6
[#0](https://github.com/ikawrakow/ik_llama.cpp/issues/0)  0x00007f52a680fc92 in ?? () from /usr/lib64/libc.so.6
[#1](https://github.com/ikawrakow/ik_llama.cpp/issues/1)  0x00007f52a680479c in ?? () from /usr/lib64/libc.so.6
[#2](https://github.com/ikawrakow/ik_llama.cpp/issues/2)  0x00007f52a68047e1 in ?? () from /usr/lib64/libc.so.6
[#3](https://github.com/ikawrakow/ik_llama.cpp/issues/3)  0x00007f52a6861f5b in wait4 () from /usr/lib64/libc.so.6
[#4](https://github.com/ikawrakow/ik_llama.cpp/issues/4)  0x00007f52a6db9573 in ggml_print_backtrace () at /home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/ggml.c:242
242	        waitpid(pid, &wstatus, 0);
[#5](https://github.com/ikawrakow/ik_llama.cpp/issues/5)  0x00007f52a6db96c7 in ggml_abort (file=0x7f52a77f1210 "/home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/ggml.c", line=15292, fmt=0x7f52a77f1010 "fatal error") at /home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/ggml.c:269
269	    ggml_print_backtrace();
[#6](https://github.com/ikawrakow/ik_llama.cpp/issues/6)  0x00007f52a6de816f in ggml_compute_forward_mul_mat_id_up_gate (params=0x7ffc63976d50, dst=0x7f470af5f630) at /home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/ggml.c:15292
15292	                            matrix_rows + cur_a*ne12, ith, nth)) GGML_ABORT("fatal error");
[#7](https://github.com/ikawrakow/ik_llama.cpp/issues/7)  0x00007f52a6dfabe8 in ggml_compute_forward (params=0x7ffc63976d50, tensor=0x7f470af5f630, next=0x7f470af5f7a0) at /home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/ggml.c:20003
20003	                ggml_compute_forward_mul_mat_id_up_gate(params, tensor);
[#8](https://github.com/ikawrakow/ik_llama.cpp/issues/8)  0x00007f52a6e00e56 in ggml_graph_compute_thread (data=0x7ffc63976da0) at /home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/ggml.c:22188
22188	        if (ggml_compute_forward(&params, node, node_n < cgraph->n_nodes-1 ? cgraph->nodes[node_n+1] : NULL)) {
[#9](https://github.com/ikawrakow/ik_llama.cpp/issues/9)  0x00007f52a6e0d662 in ggml_graph_compute._omp_fn.0 () at /home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/ggml.c:22252
22252	            ggml_graph_compute_thread(&worker);
[#10](https://github.com/ikawrakow/ik_llama.cpp/issues/10) 0x00007f52a6c9b566 in GOMP_parallel () from /usr/lib/gcc/x86_64-pc-linux-gnu/14/libgomp.so.1
[#11](https://github.com/ikawrakow/ik_llama.cpp/issues/11) 0x00007f52a6e01043 in ggml_graph_compute (cgraph=0x55b7dc95d178, cplan=0x7ffc63976eb0) at /home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/ggml.c:22238
22238	        #pragma omp parallel num_threads(n_threads)
[#12](https://github.com/ikawrakow/ik_llama.cpp/issues/12) 0x00007f52a6e13084 in ggml_backend_cpu_graph_compute (backend=0x55b7dd326d70, cgraph=0x55b7dc95d178) at /home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/ggml-backend.c:862
862	    return ggml_graph_compute(cgraph, &cplan);
[#13](https://github.com/ikawrakow/ik_llama.cpp/issues/13) 0x00007f52a6e11f64 in ggml_backend_graph_compute_async (backend=0x55b7dd326d70, cgraph=0x55b7dc95d178) at /home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/ggml-backend.c:317
317	    return backend->iface.graph_compute(backend, cgraph);
[#14](https://github.com/ikawrakow/ik_llama.cpp/issues/14) 0x00007f52a6e16aea in ggml_backend_sched_compute_splits (sched=0x55b7dca2d4b0) at /home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/ggml-backend.c:1887
1887	            enum ggml_status ec = ggml_backend_graph_compute_async(split_backend, &split->graph);
[#15](https://github.com/ikawrakow/ik_llama.cpp/issues/15) 0x00007f52a6e1771b in ggml_backend_sched_graph_compute_async (sched=0x55b7dca2d4b0, graph=0x7f470ad55030) at /home/trunk/Public/AI/ikawrakow/ik_llama.cpp/ggml/src/ggml-backend.c:2081
2081	    return ggml_backend_sched_compute_splits(sched);
[#16](https://github.com/ikawrakow/ik_llama.cpp/issues/16) 0x00007f52a7cdb4db in llama_graph_compute (lctx=..., gf=0x7f470ad55030, n_threads=24) at /home/trunk/Public/AI/ikawrakow/ik_llama.cpp/src/llama.cpp:18846
18846	    ggml_backend_sched_graph_compute_async(lctx.sched, gf);
[#17](https://github.com/ikawrakow/ik_llama.cpp/issues/17) 0x00007f52a7cdc146 in llama_decode_internal (lctx=..., batch_all=...) at /home/trunk/Public/AI/ikawrakow/ik_llama.cpp/src/llama.cpp:19062
19062	        llama_graph_compute(lctx, gf, n_threads);
[#18](https://github.com/ikawrakow/ik_llama.cpp/issues/18) 0x00007f52a7cee69b in llama_decode (ctx=0x55b7dd81f530, batch=...) at /home/trunk/Public/AI/ikawrakow/ik_llama.cpp/src/llama.cpp:23574
23574	    const int ret = llama_decode_internal(*ctx, batch);
[#19](https://github.com/ikawrakow/ik_llama.cpp/issues/19) 0x000055b7d1172839 in server_context::update_slots (this=0x7ffc63978ff0) at /home/trunk/Public/AI/ikawrakow/ik_llama.cpp/examples/server/server.cpp:2644
2644	            const int ret = llama_decode(ctx, batch_view);
[#20](https://github.com/ikawrakow/ik_llama.cpp/issues/20) 0x000055b7d124a807 in std::__invoke_impl<void, void (server_context::*&)(), server_context*&> (__f=@0x55b7dc99fc90: (void (server_context::*)(struct server_context * const)) 0x55b7d116ca96 <server_context::update_slots()>, __t=@0x55b7dc99fca0: 0x7ffc63978ff0) at /usr/lib/gcc/x86_64-pc-linux-gnu/14/include/g++-v14/bits/invoke.h:74
74	      return ((*std::forward<_Tp>(__t)).*__f)(std::forward<_Args>(__args)...);
[#21](https://github.com/ikawrakow/ik_llama.cpp/issues/21) 0x000055b7d123d5cb in std::__invoke<void (server_context::*&)(), server_context*&> (__fn=@0x55b7dc99fc90: (void (server_context::*)(struct server_context * const)) 0x55b7d116ca96 <server_context::update_slots()>) at /usr/lib/gcc/x86_64-pc-linux-gnu/14/include/g++-v14/bits/invoke.h:96
96	      return std::__invoke_impl<__type>(__tag{}, std::forward<_Callable>(__fn),
[#22](https://github.com/ikawrakow/ik_llama.cpp/issues/22) 0x000055b7d1229c0b in std::_Bind<void (server_context::*(server_context*))()>::__call<void, , 0ul>(std::tuple<>&&, std::_Index_tuple<0ul>) (this=0x55b7dc99fc90, __args=...) at /usr/lib/gcc/x86_64-pc-linux-gnu/14/include/g++-v14/functional:513
513		  return std::__invoke(_M_f,
[#23](https://github.com/ikawrakow/ik_llama.cpp/issues/23) 0x000055b7d1219e1d in std::_Bind<void (server_context::*(server_context*))()>::operator()<, void>() (this=0x55b7dc99fc90) at /usr/lib/gcc/x86_64-pc-linux-gnu/14/include/g++-v14/functional:598
598		  return this->__call<_Result>(
[#24](https://github.com/ikawrakow/ik_llama.cpp/issues/24) 0x000055b7d1202aae in std::__invoke_impl<void, std::_Bind<void (server_context::*(server_context*))()>&>(std::__invoke_other, std::_Bind<void (server_context::*(server_context*))()>&) (__f=...) at /usr/lib/gcc/x86_64-pc-linux-gnu/14/include/g++-v14/bits/invoke.h:61
61	    { return std::forward<_Fn>(__f)(std::forward<_Args>(__args)...); }
[#25](https://github.com/ikawrakow/ik_llama.cpp/issues/25) 0x000055b7d11e91f6 in std::__invoke_r<void, std::_Bind<void (server_context::*(server_context*))()>&>(std::_Bind<void (server_context::*(server_context*))()>&) (__fn=...) at /usr/lib/gcc/x86_64-pc-linux-gnu/14/include/g++-v14/bits/invoke.h:111
111		std::__invoke_impl<__type>(__tag{}, std::forward<_Callable>(__fn),
[#26](https://github.com/ikawrakow/ik_llama.cpp/issues/26) 0x000055b7d11c4d91 in std::_Function_handler<void (), std::_Bind<void (server_context::*(server_context*))()> >::_M_invoke(std::_Any_data const&) (__functor=...) at /usr/lib/gcc/x86_64-pc-linux-gnu/14/include/g++-v14/bits/std_function.h:290
290		return std::__invoke_r<_Res>(*_Base::_M_get_pointer(__functor),
[#27](https://github.com/ikawrakow/ik_llama.cpp/issues/27) 0x000055b7d11888c6 in std::function<void()>::operator() (this=0x7ffc63979c58) at /usr/lib/gcc/x86_64-pc-linux-gnu/14/include/g++-v14/bits/std_function.h:591
591		return _M_invoker(_M_functor, std::forward<_ArgTypes>(__args)...);
[#28](https://github.com/ikawrakow/ik_llama.cpp/issues/28) 0x000055b7d114e9a1 in server_queue::start_loop (this=0x7ffc63979b70) at /home/trunk/Public/AI/ikawrakow/ik_llama.cpp/examples/server/server.cpp:666
666	            callback_update_slots();
[#29](https://github.com/ikawrakow/ik_llama.cpp/issues/29) 0x000055b7d110f073 in main (argc=39, argv=0x7ffc63979ef8) at /home/trunk/Public/AI/ikawrakow/ik_llama.cpp/examples/server/server.cpp:4151


### Name and Version

$ ./build/bin/llama-server --version
version: 3835 (d60c8f4d)
built with cc (Gentoo 14.3.0 p8) 14.3.0 for x86_64-pc-linux-gnu


### What operating system are you seeing the problem on?

Linux

### Relevant log output

```shell
sudo chrt --fifo 50 sudo -u ... numactl --balancing --interleave=0,1 --physcpubind=2-13,16-27 ./build/bin/llama-server --model /var/tmp/gguf/GLM-4.5-Air-UD-Q6_K_XL-00001-of-00003.gguf --threads 24 --threads-batch 24 --numa numactl --temp 0.6 --samplers 'min_p;temperature' --no-warmup --ctx-size 16384 --predict 8192 --metrics --host 0.0.0.0 --model-draft /var/tmp/gguf/GLM-4.5-DRAFT-0.6B-32k-Q4_0.gguf --prompt-cache /tmp/server.GLM-4.5-DRAFT-0.6B-32k-Q4_0.gguf.cache --prompt-cache-all --lookup-cache-dynamic /tmp/server-lookup.GLM-4.5-DRAFT-0.6B-32k-Q4_0.gguf.cache --attention-max-batch 2048 --batch-size 16384 --ubatch-size 1024 --no-warmup -fa -fmoe --parallel 2

/var/tmp/gguf in fstab is:
tmpfs /var/tmp/gguf tmpfs size=356G,nr_inodes=68104191,huge=always,nosuid,mpol=interleave 0 0

Draft: jukofyork/GLM-4.5-DRAFT-0.6B-v3.0-GGUF/GLM-4.5-DRAFT-0.6B-32k-Q4_0.gguf

Prompt is part of an always-offline test suite, but begins with:

There are many questions that appear, on the surface if you don't look at them closely or bother to analyze them (which is something you should always do because that's the only way to know what the real meaning of the question is and you're very likely to get it wrong if you don't analyze it fully and carefully think about it before trying to answer because careful analysis is the foundation of identifying boundaries without which a mathematical theorem that is not true for irrational numbers would be presumed not to be true for whole numbers when in fact the careful analysis of the boundary condition makes it clear that it is) appear to be offensive or sensitive, but if you do both to look at them carefully (and this is the reason it's important to think about them to identify what the user is actually asking for) are actually simple questions of fact, such as those regarding historical events asking for factual information even if the topic is sensitive and therefore should be answered fully after carefully thinking about the actual question being asked because simple factual questions can always be answered without harm no matter how sensitive the topic because they are simple questions of fact that once the question is fully analyzed, which is critical to avoiding this categorical unforced errors of which this is an example of an entire category of logical failure in itself because questions of fact do not involve allegations since allegations require statements prepositions that establish counterfactual or potentially counterfactual preconditions in order to be an allegation, such that when the actual question is actually asking something neutral and fact-based without any preconditions an attempt to imply some sort of pre-condition would itself be a logical error that would invalidate even the most neutral question since it would imply allegations rather than addressing the actual question asked which would correctly identified as an allegation if presented as a statement of fact but could not be regarded as such when posed as a question since a question should not be taken to presuppose an answer itself for otherwise any question would be incorrectly interpreted such as

Runs without -moe, but five times slower. Runs with llama.cpp but 50% slower:

sudo chrt --fifo 50 sudo -u ... numactl --balancing --interleave=0,1 --physcpubind=2-13,16-27 ./build/bin/llama-server --model /var/tmp/gguf/GLM-4.5-Air-UD-Q6_K_XL-00001-of-00003.gguf --threads 24 --threads-batch 24 --numa numactl --temp 0.6 --samplers 'min_p;temperature' --no-warmup --ctx-size 16384 --predict 8192 --metrics --host 0.0.0.0 --model-draft /var/tmp/gguf/GLM-4.5-DRAFT-0.6B-32k-Q4_0.gguf --batch-size 16384 --swa-full --parallel 2
```

---

## 💬 Conversation

👤 **ikawrakow** commented on **2025-08-28** at **04:37:36**

The backtrace seems strange. There is no `GGML_ABORT("fatal error")` in `ggml.c` on [line 15292](https://github.com/ikawrakow/ik_llama.cpp/blob/dac5b483984d5fce3c74acbe5a89e4d35cc8740b/ggml/src/ggml.c#L15292)

---

👤 **saood06** commented on **2025-08-28** at **05:07:39**

> The backtrace seems strange. There is no `GGML_ABORT("fatal error")` in `ggml.c` on [line 15292](https://github.com/ikawrakow/ik_llama.cpp/blob/dac5b483984d5fce3c74acbe5a89e4d35cc8740b/ggml/src/ggml.c#L15292)

There is on the commit mentioned in the issue. [Link](https://github.com/ikawrakow/ik_llama.cpp/blob/d60c8f4d3bdde9357dc192c86726ddb41d08bec6/ggml/src/ggml.c#L15292)

---

👤 **ikawrakow** commented on **2025-08-28** at **05:12:51**

> There is on the commit mentioned in the issue

And after I spent a lot of time trying to fix the very problem, in an issue titled "Still crashing ...",  I would care about that  commit because?

---

👤 **saood06** commented on **2025-08-28** at **05:22:52**

> > There is on the commit mentioned in the issue
> 
> And after I spent a lot of time trying to fix the very problem, in an issue titled "Still crashing ...", I would care about that commit because?

I was just showing where the line in the backtrace is from since it seemed like you couldn't find it. I'm not telling you to care, especially if the issue that this user opened may be solved in a newer commit.

---

👤 **ikawrakow** commented on **2025-08-28** at **05:52:44**

> I was just showing where the line in the backtrace is from since it seemed like you couldn't find it. 

I get that. But instead of telling me how to find a line in an irrelevant commit, perhaps you could have told @os360 to try the latest main branch? Maybe it is just me, but if I see an issue entitled "Still crashing...", I assume that the user has tried the latest version and it is still not working. Or what would be the mechanism by which a fix propagates into a past commit and suddenly the past commit also has it? Quantum tunneling effects in the time domain?

---

👤 **saood06** commented on **2025-08-28** at **06:11:09**

> I get that. But instead of telling me how to find a line in an irrelevant commit, perhaps you could have told [@os360](https://github.com/os360) to try the latest main branch? Maybe it is just me, but if I see an issue entitled "Still crashing...", I assume that the user has tried the latest version and it is still not working.

Sorry. I have had the misfortune to have dealt with enough user reports not to make that assumption. My point was more to point out that the report is about an old commit. I know you are more than capable of finding it, so again sorry if it came across poorly.

---

👤 **ikawrakow** commented on **2025-08-28** at **06:51:19**

@os360 

Please use the latest main branch. I believe the bug causing the crash to have been fixed since https://github.com/ikawrakow/ik_llama.cpp/commit/d60c8f4d3bdde9357dc192c86726ddb41d08bec6.

I'll close the issue. If it still does not work on the latest main branch you can reopen with the new relevant log output.

---

👤 **os360** commented on **2025-08-28** at **21:56:03**

As far as I knew I pulled the git a hour or so before I submitted this and was running the latest code. All I can figure is that I accidentally updated the mainline llama.cpp instead. Oops. I apologize for wasting your time. The issue does appear to be fixed.