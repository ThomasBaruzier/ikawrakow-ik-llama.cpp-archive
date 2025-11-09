## 📌 [Issue #854](https://github.com/ikawrakow/ik_llama.cpp/issues/854) - Bug: ling 64ctx crash

| **Author** | `magikRUKKOLA` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-10-21 |
| **Updated** | 2025-10-22 |

---

## 📄 Description

### What happened?

```
Oct 21 15:35:22 xxx run-ik_llama.cpp.sh[2789904]: VERB [            update_slots] prompt processing progress | tid="140138671423488" timestamp=1761060922 id_slot=0 n_past=8192 n_ctx=65536 n_tokens=8192 progress=0.25340262055397034
Oct 21 15:35:22 xxx run-ik_llama.cpp.sh[2789904]: VERB [            update_slots] decoding batch | tid="140138671423488" timestamp=1761060922 n_tokens=8192
Oct 21 15:35:23 xxx run-ik_llama.cpp.sh[2789904]: CUDA error: invalid argument
Oct 21 15:35:23 xxx run-ik_llama.cpp.sh[2789904]:   current device: 0, in function cuda_bailingmoev2_experts at /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml-cuda/argsort.cu:519
Oct 21 15:35:23 xxx run-ik_llama.cpp.sh[2789904]:   cudaGetLastError()
Oct 21 15:35:23 xxx run-ik_llama.cpp.sh[2789904]: /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml-cuda.cu:121: CUDA error
Oct 21 15:35:23 xxx run-ik_llama.cpp.sh[2790927]: gdb: warning: Couldn't determine a path for the index cache directory.
Oct 21 15:35:23 xxx run-ik_llama.cpp.sh[2790927]: [New LWP 2790232]
Oct 21 15:35:23 xxx run-ik_llama.cpp.sh[2790927]: [New LWP 2790231]
Oct 21 15:35:23 xxx run-ik_llama.cpp.sh[2790927]: [New LWP 2790230]
Oct 21 15:35:23 xxx run-ik_llama.cpp.sh[2790927]: [New LWP 2790229]
Oct 21 15:35:23 xxx run-ik_llama.cpp.sh[2790927]: [New LWP 2790228]
Oct 21 15:35:23 xxx run-ik_llama.cpp.sh[2790927]: [New LWP 2790227]
Oct 21 15:35:23 xxx run-ik_llama.cpp.sh[2790927]: [New LWP 2790226]
Oct 21 15:35:23 xxx run-ik_llama.cpp.sh[2790927]: [New LWP 2790225]
Oct 21 15:35:23 xxx run-ik_llama.cpp.sh[2790927]: [New LWP 2790224]
Oct 21 15:35:23 xxx run-ik_llama.cpp.sh[2790927]: [New LWP 2790223]
Oct 21 15:35:23 xxx run-ik_llama.cpp.sh[2790927]: [New LWP 2790222]
Oct 21 15:35:23 xxx run-ik_llama.cpp.sh[2790927]: [New LWP 2790221]
Oct 21 15:35:23 xxx run-ik_llama.cpp.sh[2790927]: [New LWP 2790220]
Oct 21 15:35:23 xxx run-ik_llama.cpp.sh[2790927]: [New LWP 2790219]
Oct 21 15:35:23 xxx run-ik_llama.cpp.sh[2790927]: [New LWP 2790218]
Oct 21 15:35:23 xxx run-ik_llama.cpp.sh[2790927]: [New LWP 2790217]
Oct 21 15:35:23 xxx run-ik_llama.cpp.sh[2790927]: [New LWP 2790216]
Oct 21 15:35:23 xxx run-ik_llama.cpp.sh[2790927]: [New LWP 2790215]
Oct 21 15:35:23 xxx run-ik_llama.cpp.sh[2790927]: [New LWP 2790214]
Oct 21 15:35:23 xxx run-ik_llama.cpp.sh[2790927]: [New LWP 2790213]
Oct 21 15:35:23 xxx run-ik_llama.cpp.sh[2790927]: [New LWP 2790212]
Oct 21 15:35:23 xxx run-ik_llama.cpp.sh[2790927]: [New LWP 2790211]
Oct 21 15:35:23 xxx run-ik_llama.cpp.sh[2790927]: [New LWP 2790210]
Oct 21 15:35:23 xxx run-ik_llama.cpp.sh[2790927]: [New LWP 2790209]
Oct 21 15:35:23 xxx run-ik_llama.cpp.sh[2790927]: [New LWP 2790208]
Oct 21 15:35:23 xxx run-ik_llama.cpp.sh[2790927]: [New LWP 2790118]
Oct 21 15:35:23 xxx run-ik_llama.cpp.sh[2790927]: [New LWP 2790117]
Oct 21 15:35:23 xxx run-ik_llama.cpp.sh[2790927]: [New LWP 2790116]
Oct 21 15:35:23 xxx run-ik_llama.cpp.sh[2790927]: [New LWP 2790115]
Oct 21 15:35:23 xxx run-ik_llama.cpp.sh[2790927]: [New LWP 2790114]
Oct 21 15:35:23 xxx run-ik_llama.cpp.sh[2790927]: [New LWP 2790113]
Oct 21 15:35:23 xxx run-ik_llama.cpp.sh[2790927]: [New LWP 2790112]
Oct 21 15:35:23 xxx run-ik_llama.cpp.sh[2790927]: [New LWP 2790111]
Oct 21 15:35:23 xxx run-ik_llama.cpp.sh[2790927]: [New LWP 2790110]
Oct 21 15:35:23 xxx run-ik_llama.cpp.sh[2790927]: [New LWP 2790109]
Oct 21 15:35:23 xxx run-ik_llama.cpp.sh[2790927]: [New LWP 2790108]
Oct 21 15:35:23 xxx run-ik_llama.cpp.sh[2790927]: [New LWP 2790035]
Oct 21 15:35:23 xxx run-ik_llama.cpp.sh[2790927]: [New LWP 2790034]
Oct 21 15:35:23 xxx run-ik_llama.cpp.sh[2790927]: [New LWP 2790033]
Oct 21 15:35:23 xxx run-ik_llama.cpp.sh[2790927]: [New LWP 2790032]
Oct 21 15:35:23 xxx run-ik_llama.cpp.sh[2790927]: [New LWP 2790031]
Oct 21 15:35:23 xxx run-ik_llama.cpp.sh[2790927]: [New LWP 2789907]
Oct 21 15:35:23 xxx run-ik_llama.cpp.sh[2790927]: [New LWP 2789906]
Oct 21 15:35:23 xxx run-ik_llama.cpp.sh[2790927]: [New LWP 2789905]
Oct 21 15:35:23 xxx run-ik_llama.cpp.sh[2790927]: [Thread debugging using libthread_db enabled]
Oct 21 15:35:23 xxx run-ik_llama.cpp.sh[2790927]: Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".
Oct 21 15:35:23 xxx run-ik_llama.cpp.sh[2790927]: 0x00007f747c8a49ee in ?? () from /lib/x86_64-linux-gnu/libc.so.6
Oct 21 15:35:23 xxx run-ik_llama.cpp.sh[2790927]: #0  0x00007f747c8a49ee in ?? () from /lib/x86_64-linux-gnu/libc.so.6
Oct 21 15:35:23 xxx run-ik_llama.cpp.sh[2790927]: #1  0x00007f747c899668 in ?? () from /lib/x86_64-linux-gnu/libc.so.6
Oct 21 15:35:23 xxx run-ik_llama.cpp.sh[2790927]: #2  0x00007f747c8996ad in ?? () from /lib/x86_64-linux-gnu/libc.so.6
Oct 21 15:35:23 xxx run-ik_llama.cpp.sh[2790927]: #3  0x00007f747c904787 in wait4 () from /lib/x86_64-linux-gnu/libc.so.6
Oct 21 15:35:23 xxx run-ik_llama.cpp.sh[2790927]: #4  0x00007f747cf10ae8 in ggml_compute_forward_set_rows () from /opt/ik_llama.cpp/ik_llama.cpp/build/ggml/src/libggml.so
Oct 21 15:35:23 xxx run-ik_llama.cpp.sh[2790927]: #5  0x00007f747e2a33b0 in ?? () from /opt/ik_llama.cpp/ik_llama.cpp/build/ggml/src/libggml.so
Oct 21 15:35:23 xxx run-ik_llama.cpp.sh[2790927]: #6  0x0000000000000207 in ?? ()
Oct 21 15:35:23 xxx run-ik_llama.cpp.sh[2790927]: #7  0x00007f747d15ce83 in ggml_cuda_op_mul_mat(ggml_backend_cuda_context&, ggml_tensor const*, ggml_tensor const*, ggml_tensor*, void (*)(ggml_backend_cuda_context&, ggml_tensor const*, ggml_tenso>
Oct 21 15:35:23 xxx run-ik_llama.cpp.sh[2790927]: Backtrace stopped: Cannot access memory at address 0x8cc9050000000028
```

It does NOT fail with -ger disabled.

### Name and Version

build/bin/llama-server --version
version: 3922 (caf9759c)
built with cc (Debian 15.2.0-4) 15.2.0 for x86_64-linux-gnu


### What operating system are you seeing the problem on?

_No response_

### Relevant log output

```shell

```

---

## 💬 Conversation

👤 **ikawrakow** commented on **2025-10-21** at **15:52:06**

Can you run with `cuda-gdb --args insert_your_command_here` ? Thanks.

---

👤 **magikRUKKOLA** commented on **2025-10-21** at **15:59:58**

@ikawrakow 

```
Tensor blk.79.ffn_up_exps.weight buffer type overriden to CPU                                                               
warning: Cuda Driver error detected: No CUDA context is current to the calling thread                                       
warning: Cuda Driver error detected: Returning 201 (CUDA_ERROR_INVALID_CONTEXT) from cuCtxGetDevice_v2                      
warning: Cuda Driver error detected: No CUDA context is current to the calling thread                                       
warning: Cuda Driver error detected: Returning 201 (CUDA_ERROR_INVALID_CONTEXT) from cuCtxGetDevice_v2                      
[New Thread 0x7f89a2dff000 (LWP 2802701)]                                                                                   
[New Thread 0x7f89a25fe000 (LWP 2802702)]                                                                                   
[New Thread 0x7f89897ff000 (LWP 2802709)]                                                                                   
[New Thread 0x7f8988ffe000 (LWP 2802710)]                                                                                   
llm_load_tensors: offloading 80 repeating layers to GPU                                                                     
llm_load_tensors: offloading non-repeating layers to GPU                                                                    
llm_load_tensors: offloaded 81/81 layers to GPU                                                                             
llm_load_tensors:        CPU buffer size = 42986.73 MiB                                                                     
llm_load_tensors:        CPU buffer size = 43907.32 MiB                                                                     
llm_load_tensors:        CPU buffer size = 44553.77 MiB                                                                     
llm_load_tensors:        CPU buffer size = 44013.15 MiB                                                                     
llm_load_tensors:        CPU buffer size = 43907.32 MiB                                                                     
llm_load_tensors:        CPU buffer size = 44638.58 MiB                                                                     
llm_load_tensors:        CPU buffer size = 43907.32 MiB                                                                     
llm_load_tensors:        CPU buffer size = 44553.77 MiB                                                                     
llm_load_tensors:        CPU buffer size = 44013.15 MiB                                                                     
llm_load_tensors:        CPU buffer size = 43907.32 MiB                                                                     
llm_load_tensors:        CPU buffer size = 39672.03 MiB                                                                     
llm_load_tensors:        CPU buffer size =   690.75 MiB                                                                     
llm_load_tensors:      CUDA0 buffer size =  6695.50 MiB                                                                     
llm_load_tensors:      CUDA1 buffer size =  8006.87 MiB                                                                     
....................................................................................................                        
llama_new_context_with_model: n_ctx      = 65536                                                                            
llama_new_context_with_model: n_batch    = 8192                                                                             
llama_new_context_with_model: n_ubatch   = 8192                                                                             
llama_new_context_with_model: flash_attn = 1                                                                                
llama_new_context_with_model: mla_attn   = 0                                                                                
llama_new_context_with_model: attn_max_b = 512                                                                              
llama_new_context_with_model: fused_moe  = 1                                                                                
llama_new_context_with_model: grouped er = 1                                                                                
llama_new_context_with_model: fused_up_gate = 1                                                                             
llama_new_context_with_model: ser        = -1, 0                                                                            
llama_new_context_with_model: freq_base  = 600000.0                                                                         
llama_new_context_with_model: freq_scale = 1                                                                                
llama_kv_cache_init:      CUDA0 KV buffer size =  4896.02 MiB                                                               
llama_kv_cache_init:      CUDA1 KV buffer size =  5984.02 MiB                                                               
llama_new_context_with_model: KV self size  = 10880.00 MiB, K (q8_0): 5440.00 MiB, V (q8_0): 5440.00 MiB                    
llama_new_context_with_model:  CUDA_Host  output buffer size =     1.20 MiB                                                 
llama_new_context_with_model: pipeline parallelism enabled (n_copies=1)                                                     
llama_new_context_with_model:      CUDA0 compute buffer size =  8212.28 MiB                                                 
llama_new_context_with_model:      CUDA1 compute buffer size =  5168.00 MiB                                                 
llama_new_context_with_model:  CUDA_Host compute buffer size =  1280.09 MiB                                                 
llama_new_context_with_model: graph nodes  = 3369                                                                           
llama_new_context_with_model: graph splits = 199                                                                            
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context                                    
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context                                    
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context                                    
[New Thread 0x7f9368bff000 (LWP 2802786)]                                                                                   
[New Thread 0x7f93557ff000 (LWP 2802787)]                                                                                   
[New Thread 0x7f9354ffe000 (LWP 2802788)]                                                                                   
[New Thread 0x7f93547fd000 (LWP 2802789)]                                    
[New Thread 0x7f9353ffc000 (LWP 2802790)]                                    
[New Thread 0x7f93537fb000 (LWP 2802791)]                                    
[New Thread 0x7f9352ffa000 (LWP 2802792)]                                    
[New Thread 0x7f87e2dde000 (LWP 2802793)]                                    
[New Thread 0x7f87dcbff000 (LWP 2802794)]                                    
[New Thread 0x7f7d65fff000 (LWP 2802795)]                                    
[New Thread 0x7f7d657fe000 (LWP 2802796)]                                    
warning: Cuda Driver error detected: Returning 910 (CUDA_ERROR_GRAPH_EXEC_UPDATE_FAILURE) from cuGraphExecUpdate_v2                                        
warning: Cuda Runtime API error detected: cudaGraphExecUpdate returned cudaErrorGraphExecUpdateFailure(CUresult=910): the graph update was not performed because it included changes which violated constraint
s specific to instantiated graph update                                                                                                                    

warning: Cuda Runtime API error detected: cudaGetLastError returned cudaErrorGraphExecUpdateFailure(CUresult=910): the graph update was not performed because it included changes which violated constraints s
pecific to instantiated graph update                                                    

warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context                                                                                         
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context                                                                                         
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context                                                                                         
warning: Cuda Driver error detected: Returning 910 (CUDA_ERROR_GRAPH_EXEC_UPDATE_FAILURE) from cuGraphExecUpdate_v2                                                              
warning: Cuda Runtime API error detected: cudaGraphExecUpdate returned cudaErrorGraphExecUpdateFailure(CUresult=910): the graph update was not performed because it included changes which violated constraint
s specific to instantiated graph update                                                 

warning: Cuda Runtime API error detected: cudaGraphExecUpdate returned cudaErrorGraphExecUpdateFailure(CUresult=910): the graph update was not performed because it included changes which violated constraint
s specific to instantiated graph update                                                 

warning: Cuda Runtime API error detected: cudaGetLastError returned cudaErrorGraphExecUpdateFailure(CUresult=910): the graph update was not performed because it included changes which violated constraints s
pecific to instantiated graph update                                                    

warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context                                                                                                                      
INFO [                    init] initializing slots | tid="140737353117696" timestamp=1761062314 n_slots=1                                                                                                     
INFO [                    init] new slot | tid="140737353117696" timestamp=1761062314 id_slot=0 n_ctx_slot=65536                                                                                              
INFO [                    main] model loaded | tid="140737353117696" timestamp=1761062314       
```

---

👤 **ikawrakow** commented on **2025-10-21** at **16:02:35**

Does [#855](https://github.com/ikawrakow/ik_llama.cpp/issues/855) fix it?

---

👤 **magikRUKKOLA** commented on **2025-10-21** at **16:03:51**

@ikawrakow 
> Does [[#855](https://github.com/ikawrakow/ik_llama.cpp/issues/855)](https://github.com/ikawrakow/ik_llama.cpp/pull/855) fix it?

~~Yes, it does.~~

Hold on.  There is an additional crash in decoding after that.  Should I create a new issue?

---

👤 **magikRUKKOLA** commented on **2025-10-21** at **16:07:02**

@ikawrakow 

```
llama_new_context_with_model: n_ctx      = 65536
llama_new_context_with_model: n_batch    = 8192
llama_new_context_with_model: n_ubatch   = 8192
llama_new_context_with_model: flash_attn = 1
llama_new_context_with_model: mla_attn   = 0
llama_new_context_with_model: attn_max_b = 512
llama_new_context_with_model: fused_moe  = 1
llama_new_context_with_model: grouped er = 1
llama_new_context_with_model: fused_up_gate = 1
llama_new_context_with_model: ser        = -1, 0
llama_new_context_with_model: freq_base  = 600000.0
llama_new_context_with_model: freq_scale = 1
llama_kv_cache_init:      CUDA0 KV buffer size =  4896.02 MiB
llama_kv_cache_init:      CUDA1 KV buffer size =  5984.02 MiB
llama_new_context_with_model: KV self size  = 10880.00 MiB, K (q8_0): 5440.00 MiB, V (q8_0): 5440.00 MiB
llama_new_context_with_model:  CUDA_Host  output buffer size =     1.20 MiB
llama_new_context_with_model: pipeline parallelism enabled (n_copies=1)
llama_new_context_with_model:      CUDA0 compute buffer size =  8212.28 MiB
llama_new_context_with_model:      CUDA1 compute buffer size =  5168.00 MiB
llama_new_context_with_model:  CUDA_Host compute buffer size =  1280.09 MiB
llama_new_context_with_model: graph nodes  = 3369
llama_new_context_with_model: graph splits = 199
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context
[New Thread 0x7f9368bff000 (LWP 2807438)]
[New Thread 0x7f9352fff000 (LWP 2807439)]
[New Thread 0x7f87e29ff000 (LWP 2807440)]
[New Thread 0x7f7d67fff000 (LWP 2807441)]
[New Thread 0x7f7d677fe000 (LWP 2807442)]
[New Thread 0x7f7d66ffd000 (LWP 2807443)]
[New Thread 0x7f7d667fc000 (LWP 2807444)]
[New Thread 0x7f7d65ffb000 (LWP 2807445)]
[New Thread 0x7f7d657fa000 (LWP 2807446)]
[New Thread 0x7f7d64ff9000 (LWP 2807447)]
[New Thread 0x7f7d647f8000 (LWP 2807448)]
warning: Cuda Driver error detected: Returning 910 (CUDA_ERROR_GRAPH_EXEC_UPDATE_FAILURE) from cuGraphExecUpdate_v2
warning: Cuda Runtime API error detected: cudaGraphExecUpdate returned cudaErrorGraphExecUpdateFailure(CUresult=910): the graph update was not performed because it included changes which violated constraints specific to instantiated graph update

warning: Cuda Runtime API error detected: cudaGetLastError returned cudaErrorGraphExecUpdateFailure(CUresult=910): the graph update was not performed because it included changes which violated constraints specific to instantiated graph update

warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context
warning: Cuda Driver error detected: Returning 910 (CUDA_ERROR_GRAPH_EXEC_UPDATE_FAILURE) from cuGraphExecUpdate_v2
warning: Cuda Runtime API error detected: cudaGraphExecUpdate returned cudaErrorGraphExecUpdateFailure(CUresult=910): the graph update was not performed because it included changes which violated constraints specific to instantiated graph update

warning: Cuda Runtime API error detected: cudaGraphExecUpdate returned cudaErrorGraphExecUpdateFailure(CUresult=910): the graph update was not performed because it included changes which violated constraints specific to instantiated graph update

warning: Cuda Runtime API error detected: cudaGetLastError returned cudaErrorGraphExecUpdateFailure(CUresult=910): the graph update was not performed because it included changes which violated constraints specific to instantiated graph update

warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context
INFO [                    init] initializing slots | tid="140737353117696" timestamp=1761062769 n_slots=1
INFO [                    init] new slot | tid="140737353117696" timestamp=1761062769 id_slot=0 n_ctx_slot=65536
INFO [                    main] model loaded | tid="140737353117696" timestamp=1761062769
INFO [                    main] chat template | tid="140737353117696" timestamp=1761062769 chat_template="{% set thinking_option = 'off' %}\n{{- '<role>SYSTEM</role>' }}\n{%- if messages[0].role == 'system' %}\n    {{- messages[0].content + '\\n' }}\n{%- endif %}\n{%- if tools %}\n    {{- \"# Tools\\n\\nYou may call one or more functions to assist with the user query.\\n\\nYou are provided with function signatures within <tools></tools> XML tags:\\n<tools>\" }}\n    {%- for tool in tools %}\n        {{- \"\\n\" }}\n        {{- tool | tojson }}\n    {%- endfor %}\n    {{- \"\\n</tools>\\n\\nFor each function call, return a json object with function name and arguments within <tool_call></tool_call> XML tags:\\n<tool_call>\\n{\\\"name\\\": <function-name>, \\\"arguments\\\": <args-json-object>}\\n</tool_call>\\n\" }}\n{%- endif %}\n{{- 'detailed thinking ' + thinking_option + '<|role_end|>' }}\n{%- set ns = namespace(multi_step_tool=true, last_query_index=messages|length - 1) %}\n{%- for message in messages[::-1] %}\n    {%- set index = (messages|length - 1) - loop.index0 %}\n    {%- if ns.multi_step_tool and message.role == \"user\" and message.content is string and not(message.content.startswith('<tool_response>') and message.content.endswith('</tool_response>')) %}\n        {%- set ns.multi_step_tool = false %}\n        {%- set ns.last_query_index = index %}\n    {%- endif %}\n{%- endfor %}\n{%- for message in messages %}\n    {%- if message.content is string %}\n        {%- set content = message.content %}\n    {%- else %}\n        {%- set content = '' %}\n    {%- endif %}\n    {%- if message.role == \"user\" %}\n        {{- '<role>HUMAN</role>' + message.content + '<|role_end|>' }}\n    {%- elif message.role == \"system\" and not loop.first %}\n        {{- '<role>SYSTEM</role>' + message.content + '<|role_end|>' }}\n    {%- elif message.role == \"assistant\" %}\n        {%- set reasoning_content = '' %}\n        {%- if message.reasoning_content is string %}\n            {%- set reasoning_content = message.reasoning_content %}\n        {%- else %}\n            {%- if '</think>' in content %}\n                {%- set reasoning_content = content.split('</think>')[0].rstrip('\\n').split('<think>')[-1].lstrip('\\n') %}\n                {%- set content = content.split('</think>')[-1].lstrip('\\n') %}\n            {%- endif %}\n        {%- endif %}\n        {%- if loop.index0 > ns.last_query_index %}\n            {%- if reasoning_content %}\n                {{- '<role>ASSISTANT</role>' + '\\n<think>\\n' + reasoning_content.strip('\\n') + '\\n</think>\\n\\n' + content.lstrip('\\n') }}\n            {%- else %}\n                {{- '<role>ASSISTANT</role>' + content }}\n            {%- endif %}\n        {%- else %}\n            {{- '<role>ASSISTANT</role>' + content }}\n        {%- endif %}\n        {%- if message.tool_calls %}\n            {%- for tool_call in message.tool_calls %}\n                {%- if (loop.first and content) or (not loop.first) %}\n                    {{- '\\n' }}\n                {%- endif %}\n                {%- if tool_call.function %}\n                    {%- set tool_call = tool_call.function %}\n                {%- endif %}\n                {{- '<tool_call>\\n{\"name\": \"' }}\n                {{- tool_call.name }}\n                {{- '\", \"arguments\": ' }}\n                {%- if tool_call.arguments is string %}\n                    {{- tool_call.arguments }}\n                {%- else %}\n                    {{- tool_call.arguments | tojson }}\n                {%- endif %}\n                {{- '}\\n</tool_call>' }}\n            {%- endfor %}\n        {%- endif %}\n        {{- '<|role_end|>' }}\n    {%- elif message.role == \"tool\" %}\n        {%- if loop.first or (messages[loop.index0 - 1].role != \"tool\") %}\n            {{- '<role>OBSERVATION</role>' }}\n        {%- endif %}\n        {{- '\\n<tool_response>\\n' }}\n        {{- content }}\n        {{- '\\n</tool_response>' }}\n        {%- if loop.last or (messages[loop.index0 + 1].role != \"tool\") %}\n            {{- '<|role_end|>' }}\n        {%- endif %}\n    {%- endif %}\n{%- endfor %}\n{%- if add_generation_prompt %}\n    {{- '<role>ASSISTANT</role>' }}\n{%- endif %}"
INFO [                    main] chat template | tid="140737353117696" timestamp=1761062769 chat_example="<role>SYSTEM</role>You are a helpful assistant\ndetailed thinking off<|role_end|><role>HUMAN</role>Hello<|role_end|><role>ASSISTANT</role>Hi there<|role_end|><role>HUMAN</role>How are you?<|role_end|><role>ASSISTANT</role>" built_in=true
INFO [                    main] HTTP server listening | tid="140737353117696" timestamp=1761062769 n_threads_http="23" port="8080" hostname="0.0.0.0"
[New Thread 0x7f7d63ff7000 (LWP 2807514)]
VERB [              start_loop] new task may arrive | tid="140737353117696" timestamp=1761062769
VERB [              start_loop] update_multitasks | tid="140737353117696" timestamp=1761062769
VERB [              start_loop] callback_update_slots | tid="140737353117696" timestamp=1761062769
INFO [            update_slots] all slots are idle | tid="140737353117696" timestamp=1761062769
VERB [          kv_cache_clear] clearing KV cache | tid="140737353117696" timestamp=1761062769
[New Thread 0x7f7d637f6000 (LWP 2807515)]
[New Thread 0x7f7d62ff5000 (LWP 2807516)]
[New Thread 0x7f7d53fff000 (LWP 2807517)]
[New Thread 0x7f7d537fe000 (LWP 2807518)]
[New Thread 0x7f7d52ffd000 (LWP 2807519)]
[New Thread 0x7f7d527fc000 (LWP 2807520)]
[New Thread 0x7f7d51ffb000 (LWP 2807521)]
[New Thread 0x7f7d517fa000 (LWP 2807522)]
[New Thread 0x7f7d50ff9000 (LWP 2807523)]
[New Thread 0x7f7d507f8000 (LWP 2807524)]
[New Thread 0x7f7d4fff7000 (LWP 2807525)]
[New Thread 0x7f7d4f7f6000 (LWP 2807526)]
[New Thread 0x7f7d4eff5000 (LWP 2807527)]
[New Thread 0x7f7d4e7f4000 (LWP 2807528)]
[New Thread 0x7f7d4dff3000 (LWP 2807529)]
[New Thread 0x7f7d4d7f2000 (LWP 2807530)]
[New Thread 0x7f7d4cff1000 (LWP 2807531)]
[New Thread 0x7f7d4c7f0000 (LWP 2807532)]
[New Thread 0x7f7d4bfef000 (LWP 2807533)]
[New Thread 0x7f7d4b7ee000 (LWP 2807534)]
[New Thread 0x7f7d4afed000 (LWP 2807535)]
[New Thread 0x7f7d4a7ec000 (LWP 2807536)]
[New Thread 0x7f7d49feb000 (LWP 2807537)]
VERB [              start_loop] wait for new task | tid="140737353117696" timestamp=1761062769

...


warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context
[New Thread 0x7f7d497ea000 (LWP 2807722)]
[New Thread 0x7f7d48fe9000 (LWP 2807723)]
[New Thread 0x7f7d487e8000 (LWP 2807724)]
[New Thread 0x7f7d47fe7000 (LWP 2807725)]
[New Thread 0x7f7d477e6000 (LWP 2807726)]
[New Thread 0x7f7d46fe5000 (LWP 2807727)]
[New Thread 0x7f7d467e4000 (LWP 2807728)]
[New Thread 0x7f7d45fe3000 (LWP 2807729)]
[New Thread 0x7f7d457e2000 (LWP 2807730)]
[New Thread 0x7f7d44fe1000 (LWP 2807731)]
[New Thread 0x7f7d447e0000 (LWP 2807732)]
[Thread 0x7f7d497ea000 (LWP 2807722) exited]
[Thread 0x7f7d46fe5000 (LWP 2807727) exited]
[Thread 0x7f7d47fe7000 (LWP 2807725) exited]
[Thread 0x7f7d44fe1000 (LWP 2807731) exited]
[Thread 0x7f7d45fe3000 (LWP 2807729) exited]
[Thread 0x7f7d477e6000 (LWP 2807726) exited]
[Thread 0x7f7d48fe9000 (LWP 2807723) exited]
[Thread 0x7f7d467e4000 (LWP 2807728) exited]
[Thread 0x7f7d457e2000 (LWP 2807730) exited]
[Thread 0x7f7d447e0000 (LWP 2807732) exited]
[Thread 0x7f7d487e8000 (LWP 2807724) exited]
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context
warning: Cuda Driver error detected: Grid Dimension Y of 65536 exceeds maximum value of 65535
warning: Cuda Driver error detected: Returning 1 (CUDA_ERROR_INVALID_VALUE) from cuLaunchKernel
warning: Cuda Runtime API error detected: cudaLaunchKernel returned cudaErrorInvalidValue(CUresult=1): invalid argument

warning: Cuda Runtime API error detected: cudaGetLastError returned cudaErrorInvalidValue(CUresult=1): invalid argument

CUDA error: invalid argument
  current device: 0, in function cuda_bailingmoev2_experts at /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml-cuda/argsort.cu:503
  cudaGetLastError()
/opt/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml-cuda.cu:121: CUDA error
[Detaching after fork from child process 2807757]
warning: process 2807237 is already traced by process 2807140
ptrace: Operation not permitted.
No stack.
The program is not being run.

Thread 1 "llama-server" received signal SIGABRT, Aborted.

```

---

👤 **ikawrakow** commented on **2025-10-21** at **16:09:13**

I added another commit that should fix it. Now both changes are on the main branch. Does it still crash?

---

👤 **magikRUKKOLA** commented on **2025-10-21** at **16:16:42**

@ikawrakow 
> I added another commit that should fix it. Now both changes are on the main branch. Does it still crash?

No, it does not crash.

seeing alot of warnings though:

```
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context

```

---

👤 **magikRUKKOLA** commented on **2025-10-21** at **16:23:46**

@ikawrakow 

Yeah, it does not crash but right now for some reason the output is getting cut off after about 512 tokens and about 32k prefill:

```
                                                                                      
VERB [                    send] send new result | tid="140737353117696" timestamp=1761063661 id_task=0                                                                                                        
VERB [                    send] queue_results.push_back | tid="140737353117696" timestamp=1761063661 id_task=0                                                                                                
VERB [           process_token] next token | tid="140737353117696" timestamp=1761063661 id_slot=0 id_task=0 token=2010 token_text=".c" has_next_token=true n_remain=-1 n_decoded=438 stopped_eos=false stopped
_word=false stopped_limit=false stopping_word=""                                                                                                                                                              
VERB [            update_slots] run slots completed | tid="140737353117696" timestamp=1761063661                                                                                                              
VERB [              start_loop] wait for new task | tid="140737353117696" timestamp=1761063661                                                                                                                
VERB [              start_loop] new task may arrive | tid="140737353117696" timestamp=1761063661                                                                                                              
VERB [              start_loop] callback_new_task | tid="140737353117696" timestamp=1761063661 id_task=441                                                                                                    
VERB [              operator()] data stream | tid="140176223367168" timestamp=1761063661 to_send="data: {\"choices\":[{\"finish_reason\":null,\"index\":0,\"delta\":{\"content\":\".c\"}}],\"created\":1761063
661,\"id\":\"chatcmpl-7MbAKxpL6yNevv5LeaULNDHzrEVVwOW9\",\"model\":\"\",\"object\":\"chat.completion.chunk\",\"usage\":{\"completion_tokens\":438,\"prompt_tokens\":32328,\"total_tokens\":32766}}\n\n"       
VERB [              start_loop] update_multitasks | tid="140737353117696" timestamp=1761063661                                                                                                                
VERB [              start_loop] callback_update_slots | tid="140737353117696" timestamp=1761063661                                                                                                            
VERB [            update_slots] posting NEXT_RESPONSE | tid="140737353117696" timestamp=1761063661                                                                                                            
VERB [                    post] new task id | tid="140737353117696" timestamp=1761063661 new_id=442                                                                                                           
VERB [            update_slots] slot decode token | tid="140737353117696" timestamp=1761063661 id_slot=0 id_task=0 n_ctx=65536 n_past=32766 n_system_tokens=0 n_cache_tokens=32766 truncated=false            
VERB [            update_slots] decoding batch | tid="140737353117696" timestamp=1761063661 n_tokens=1                                                                                                        
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context                                                                                                                      
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context                                                                                                                      
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context                                                                                                                      
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context                                                                                                                      
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context                                                                                                                      
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context                                                                                                                      
VERB [                    send] send new result | tid="140737353117696" timestamp=1761063661 id_task=0                                                                                                        
VERB [                    send] queue_results.push_back | tid="140737353117696" timestamp=1761063661 id_task=0                                                                                                
VERB [           process_token] next token | tid="140737353117696" timestamp=1761063661 id_slot=0 id_task=0 token=4403 token_text=" output" has_next_token=true n_remain=-1 n_decoded=439 stopped_eos=false st
opped_word=false stopped_limit=false stopping_word=""                                                                                                                                                         
VERB [            update_slots] run slots completed | tid="140737353117696" timestamp=1761063661                                                                                                              
VERB [              start_loop] wait for new task | tid="140737353117696" timestamp=1761063661                                                                                                                
VERB [              start_loop] new task may arrive | tid="140737353117696" timestamp=1761063661                                                                                                              
VERB [              start_loop] callback_new_task | tid="140737353117696" timestamp=1761063661 id_task=442                                                                                                    VERB [              start_loop] update_multitasks | tid="140737353117696" timestamp=1761063661                                                                                                                
VERB [              start_loop] callback_update_slots | tid="140737353117696" timestamp=1761063661                                                                                                            
VERB [            update_slots] posting NEXT_RESPONSE | tid="140737353117696" timestamp=1761063661                                                                                                            
VERB [                    post] new task id | tid="140737353117696" timestamp=1761063661 new_id=443                                                                                                           
VERB [              operator()] data stream | tid="140176223367168" timestamp=1761063661 to_send="data: {\"choices\":[{\"finish_reason\":null,\"index\":0,\"delta\":{\"content\":\" output\"}}],\"created\":17
61063661,\"id\":\"chatcmpl-7MbAKxpL6yNevv5LeaULNDHzrEVVwOW9\",\"model\":\"\",\"object\":\"chat.completion.chunk\",\"usage\":{\"completion_tokens\":439,\"prompt_tokens\":32328,\"total_tokens\":32767}}\n\n"  
VERB [            update_slots] slot decode token | tid="140737353117696" timestamp=1761063661 id_slot=0 id_task=0 n_ctx=65536 n_past=32767 n_system_tokens=0 n_cache_tokens=32767 truncated=false            
VERB [            update_slots] decoding batch | tid="140737353117696" timestamp=1761063661 n_tokens=1                                                                                                        
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context                                                                                                                      
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context                                                                                                                      
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context                                                                                                                      
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context                                                                                                                      
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context                                                                                                                      
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context                                                                                                                      
VERB [                    send] send new result | tid="140737353117696" timestamp=1761063662 id_task=0                                                                                                        
VERB [                    send] queue_results.push_back | tid="140737353117696" timestamp=1761063662 id_task=0                                                                                                
WARN [           process_token] n_predict is not set and self-context extend is disabled. Limiting generated tokens to n_ctx_train to avoid EOS-less generation infinite loop | tid="140737353117696" timestam
p=1761063662 id_slot=0 params.n_predict=-1 slot.n_prompt_tokens=32328 slot.n_decoded=440 slot.n_predict=-1 n_slots=1 slot.n_ctx=65536 n_ctx=65536 n_ctx_train=32768 ga_n=1                                    
VERB [           process_token] next token | tid="140737353117696" timestamp=1761063662 id_slot=0 id_task=0 token=198 token_text="\n" has_next_token=false n_remain=-1 n_decoded=440 stopped_eos=false stopped
_word=false stopped_limit=true stopping_word=""                                                                                                                                                               
INFO [           print_timings] prompt eval time     =  171024.48 ms / 32328 tokens (    5.29 ms per token,   189.03 tokens per second) | tid="140737353117696" timestamp=1761063662 id_slot=0 id_task=0 t_pro
mpt_processing=171024.483 n_prompt_tokens_processed=32328 t_token=5.290289625092799 n_tokens_second=189.02556776037733                                                                                        
VERB [              operator()] data stream | tid="140176223367168" timestamp=1761063662 to_send="data: {\"choices\":[{\"finish_reason\":null,\"index\":0,\"delta\":{\"content\":\"\\n\"}}],\"created\":176106
3662,\"id\":\"chatcmpl-7MbAKxpL6yNevv5LeaULNDHzrEVVwOW9\",\"model\":\"\",\"object\":\"chat.completion.chunk\",\"usage\":{\"completion_tokens\":440,\"prompt_tokens\":32328,\"total_tokens\":32768}}\n\n"      
INFO [           print_timings] generation eval time =  159576.42 ms /   440 runs   (  362.67 ms per token,     2.76 tokens per second) | tid="140737353117696" timestamp=1761063662 id_slot=0 id_task=0 t_tok
en_generation=159576.417 n_decoded=440 t_token=362.67367499999995 n_tokens_second=2.757299657881152                                                                                                           
INFO [           print_timings]           total time =  330600.90 ms | tid="140737353117696" timestamp=1761063662 id_slot=0 id_task=0 t_prompt_processing=171024.483 t_token_generation=159576.417 t_total=330
600.9                                                                                                                                                                                                         
VERB [                    send] send new result | tid="140737353117696" timestamp=1761063662 id_task=0                                                                                                        
VERB [                    send] queue_results.push_back | tid="140737353117696" timestamp=1761063662 id_task=0                                                                                                
VERB [            update_slots] run slots completed | tid="140737353117696" timestamp=1761063662                                                                                                              
VERB [              start_loop] wait for new task | tid="140737353117696" timestamp=1761063662                                                                                                                
VERB [              start_loop] new task may arrive | tid="140737353117696" timestamp=1761063662                                                                                                              
VERB [              start_loop] callback_new_task | tid="140737353117696" timestamp=1761063662 id_task=443                                                                                                    
VERB [              start_loop] update_multitasks | tid="140737353117696" timestamp=1761063662                                                                                                                
VERB [              start_loop] callback_update_slots | tid="140737353117696" timestamp=1761063662                                                                                                            
INFO [            update_slots] slot released | tid="140737353117696" timestamp=1761063662 id_slot=0 id_task=0 n_ctx=65536 n_past=32767 n_system_tokens=0 n_cache_tokens=32767 truncated=true                 
INFO [            update_slots] all slots are idle | tid="140737353117696" timestamp=1761063662                                                                                                               
VERB [              start_loop] wait for new task | tid="140737353117696" timestamp=1761063662                                                                                                                
VERB [              operator()] data stream | tid="140176223367168" timestamp=1761063662 to_send="data: {\"choices\":[{\"finish_reason\":\"stop\",\"index\":0,\"delta\":{}}],\"created\":1761063662,\"id\":\"c
hatcmpl-7MbAKxpL6yNevv5LeaULNDHzrEVVwOW9\",\"model\":\"Ling-1T-smol-IQ4_KSS\",\"object\":\"chat.completion.chunk\"}\n\n"                                                                                      
VERB [              operator()] data stream | tid="140176223367168" timestamp=1761063662 to_send="data: {\"choices\":[],\"created\":1761063662,\"id\":\"chatcmpl-7MbAKxpL6yNevv5LeaULNDHzrEVVwOW9\",\"model\":
\"Ling-1T-smol-IQ4_KSS\",\"object\":\"chat.completion.chunk\",\"usage\":{\"completion_tokens\":440,\"prompt_tokens\":32328,\"total_tokens\":32768},\"timings\":{\"prompt_n\":32328,\"prompt_ms\":171024.483,\"
prompt_per_token_ms\":5.290289625092799,\"prompt_per_second\":189.02556776037733,\"predicted_n\":440,\"predicted_ms\":159576.417,\"predicted_per_token_ms\":362.67367499999995,\"predicted_per_second\":2.7572
99657881152}}\n\n"                                                                                                                                                              
```

---

👤 **ikawrakow** commented on **2025-10-21** at **16:31:04**

The output is being cutoff how? Is this with a draft model? I have never seen the `CUDA Stream does not belong to the expected context` warnings before.

---

👤 **magikRUKKOLA** commented on **2025-10-21** at **16:33:35**

> The output is being cutoff how?

That is, I am asking the model to make patch and after about 512 tokens it just stops.

Example:

```
 cat /tmp/ccc | mods -m ling "plz rewrite nvoc.c to use the colour palettle that actually displayed in a seperate program colours.c (for terminals with less than  256 colours use key colours blue, green, yellow and red).  use only the colour palette that you see in the output.  the colour indexes are specified there.  provide only diffs (patches) so i can apply them to my file and fix the issue"

  Here is the minimal diff (patch) to update  nvoc.c  so that it uses only the
  exact color palette indices displayed in your  colours.c  output, with      
  fallback to key ANSI colors ( blue ,  green ,  yellow ,  red ) for terminals
  with <256 colors.                                                           
                                                                              
  The changes focus on replacing the existing  map_value_to_color()  and      
  temperature coloring logic with one that strictly follows the observed      
  palette from your  colours.c  run (i.e., steel heating gradient: 20 → 27 →  
  ... → 160), using hardcoded intermediate indices, not interpolation.        
                                                                              
  --------                                                                    
                                                                              
  ### ✅ Patch for  /root/utils/nvok/src/nvoc.c                               
                                                                              
    --- a/nvoc.c                                                              
    +++ b/nvoc.c                                                              
    @@ -1849,32 +1849,57 @@ int map_value_to_color(double value, int n, int*  
  key_colors, int num_key_colors)                                             
                                                                              
         // For terminals with less than 256 colors use key ANSI colors:      
  blue(12), green(10), yellow(11), red(9)                                     
         if (COLORS < 256 || COLOR_PAIRS <= 8) {                              
    -        if (ratio < 0.25) return key_colors[segment];                    
    -        else if (ratio < 0.75) {                                         
    +        // Map segments to blue, green, yellow, red                      
    +        int ansi_palette[] = {12, 10, 11, 9}; // blue, green, yellow, red
    +        double seg_pos = normalized / (1.0 / (num_key_colors - 1));      
    +        int idx = (int)seg_pos;                                          
    +        if (idx >= num_key_colors - 1) idx = num_key_colors - 2;         
    +        return ansi_palette[idx];                                        
    +    }                                                                    
    +                                                                         
    +    // Otherwise, use exact indices from the observed palette in colours.
  c output                                                                    
```
(SIC)

>  Is this with a draft model?

No, I am not using spec. decoding because on a test rig I got only two GPUs.

Here is the command I am using:

```
cuda-gdb --args \
/opt/ik_llama.cpp/ik_llama.cpp/build/bin/llama-server \
    --ctx-size $((64 * 1024)) \
    --model /opt/ubergarm/Ling-1T-GGUF/smol-IQ4-KSS/Ling-1T-smol-IQ4_KSS-00001-of-00011.gguf \
    --alias ubergarm/Ling-1T-smol-IQ4_KSS \
    -b $((16 * 512)) -ub $((16 * 512)) \
    --mlock \
    --temp 0.7 --top-k 0 --top-p 0.95 --min-p 0.1 --repeat-penalty 1.1 \
    -ctk q8_0 -ctv q8_0 \
    -fa \
    -fmoe \
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

👤 **magikRUKKOLA** commented on **2025-10-21** at **16:37:42**

@ikawrakow 
> The output is being cutoff how? Is this with a draft model? I have never seen the `CUDA Stream does not belong to the expected context` warnings before.

Yeah, getting about 5-10 of them each second during prefill.

And from 2 to 6 at each decode token.

---

👤 **ikawrakow** commented on **2025-10-21** at **16:40:29**

Can you try [#853](https://github.com/ikawrakow/ik_llama.cpp/issues/853) and let me know if this fixes the issues you are observing? Thanks.

---

👤 **magikRUKKOLA** commented on **2025-10-21** at **16:52:29**

@ikawrakow 

Still seeing the warnings during decode.

```
VERB [            update_slots] decoding batch | tid="140737353117696" timestamp=1761065474 n_tokens=1                      
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context                                    
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context                                    
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context                                    
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context                                    
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context                                    
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context                                    
VERB [                    send] send new result | tid="140737353117696" timestamp=1761065474 id_task=0                      
VERB [                    send] queue_results.push_back | tid="140737353117696" timestamp=1761065474 id_task=0              
VERB [           process_token] next token | tid="140737353117696" timestamp=1761065474 id_slot=0 id_task=0 token=268 token_
text=" the" has_next_token=true n_remain=-1 n_decoded=19 stopped_eos=false stopped_word=false stopped_limit=false stopping_w
ord=""                                                                                                                      
VERB [            update_slots] run slots completed | tid="140737353117696" timestamp=1761065474                            
VERB [              start_loop] wait for new task | tid="140737353117696" timestamp=1761065474                              
VERB [              start_loop] new task may arrive | tid="140737353117696" timestamp=1761065474                            
VERB [              start_loop] callback_new_task | tid="140737353117696" timestamp=1761065474 id_task=22                   
VERB [              start_loop] update_multitasks | tid="140737353117696" timestamp=1761065474                              
VERB [              start_loop] callback_update_slots | tid="140737353117696" timestamp=1761065474                          
VERB [              operator()] data stream | tid="140176089067520" timestamp=1761065474 to_send="data: {\"choices\":[{\"fin
ish_reason\":null,\"index\":0,\"delta\":{\"content\":\" the\"}}],\"created\":1761065474,\"id\":\"chatcmpl-RFVy0UY3XwLTNDVdUk
bvVnR897yrZki5\",\"model\":\"\",\"object\":\"chat.completion.chunk\",\"usage\":{\"completion_tokens\":19,\"prompt_tokens\":3
2328,\"total_tokens\":32347}}\n\n"                                                                                          
VERB [            update_slots] posting NEXT_RESPONSE | tid="140737353117696" timestamp=1761065474                          
VERB [                    post] new task id | tid="140737353117696" timestamp=1761065474 new_id=23                          
VERB [            update_slots] slot decode token | tid="140737353117696" timestamp=1761065474 id_slot=0 id_task=0 n_ctx=655
36 n_past=32347 n_system_tokens=0 n_cache_tokens=32347 truncated=false                                                      
VERB [            update_slots] decoding batch | tid="140737353117696" timestamp=1761065474 n_tokens=1                      
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context                                    
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context                                    
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context                                    
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context                                    
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context                                    
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context                                    
VERB [                    send] send new result | tid="140737353117696" timestamp=1761065474 id_task=0                      
VERB [                    send] queue_results.push_back | tid="140737353117696" timestamp=1761065474 id_task=0              
VERB [           process_token] next token | tid="140737353117696" timestamp=1761065474 id_slot=0 id_task=0 token=3797 token
_text=" color" has_next_token=true n_remain=-1 n_decoded=20 stopped_eos=false stopped_word=false stopped_limit=false stoppin
g_word=""                                                                                                                   
VERB [            update_slots] run slots completed | tid="140737353117696" timestamp=1761065474                            
VERB [              start_loop] wait for new task | tid="140737353117696" timestamp=1761065474                              
VERB [              start_loop] new task may arrive | tid="140737353117696" timestamp=1761065474                            
VERB [              operator()] data stream | tid="140176089067520" timestamp=1761065474 to_send="data: {\"choices\":[{\"fin
ish_reason\":null,\"index\":0,\"delta\":{\"content\":\" color\"}}],\"created\":1761065474,\"id\":\"chatcmpl-RFVy0UY3XwLTNDVd
UkbvVnR897yrZki5\",\"model\":\"\",\"object\":\"chat.completion.chunk\",\"usage\":{\"completion_tokens\":20,\"prompt_tokens\"
:32328,\"total_tokens\":32348}}\n\n"                                                                                        
VERB [              start_loop] callback_new_task | tid="140737353117696" timestamp=1761065474 id_task=23                   
VERB [              start_loop] update_multitasks | tid="140737353117696" timestamp=1761065474                              
VERB [              start_loop] callback_update_slots | tid="140737353117696" timestamp=1761065474                          
VERB [            update_slots] posting NEXT_RESPONSE | tid="140737353117696" timestamp=1761065474                          
VERB [                    post] new task id | tid="140737353117696" timestamp=1761065474 new_id=24                          
VERB [            update_slots] slot decode token | tid="140737353117696" timestamp=1761065474 id_slot=0 id_task=0 n_ctx=655
36 n_past=32348 n_system_tokens=0 n_cache_tokens=32348 truncated=false                                                      
VERB [            update_slots] decoding batch | tid="140737353117696" timestamp=1761065474 n_tokens=1                      
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context                                    
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context                                    
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context                                    
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context                                    
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context                                    
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context  
```

Lets see if its still doing the cutoff. ...

[EDIT]: yes, it still abruptly stops:

```
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context                                                                                                                                             16:53:45 [457/1276]
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context                                                                                                                                                                
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context                                                                                                                                                                
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context                                                                                                                                                                
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context                                                                                                                                                                
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context                                                                                                                                                                
VERB [                    send] send new result | tid="140737353117696" timestamp=1761065624 id_task=0                                                                                                                                                  
VERB [                    send] queue_results.push_back | tid="140737353117696" timestamp=1761065624 id_task=0                                                                                                                                          
VERB [           process_token] next token | tid="140737353117696" timestamp=1761065624 id_slot=0 id_task=0 token=394 token_text="       " has_next_token=true n_remain=-1 n_decoded=437 stopped_eos=false stopped_word=false stopped_limit=false stoppi
ng_word=""                                                                                                                                                                                                                                              
VERB [            update_slots] run slots completed | tid="140737353117696" timestamp=1761065624                                                                                                                                                        
VERB [              start_loop] wait for new task | tid="140737353117696" timestamp=1761065624                                                                                                                                                          
VERB [              operator()] data stream | tid="140176089067520" timestamp=1761065624 to_send="data: {\"choices\":[{\"finish_reason\":null,\"index\":0,\"delta\":{\"content\":\"       \"}}],\"created\":1761065624,\"id\":\"chatcmpl-RFVy0UY3XwLTNDV
dUkbvVnR897yrZki5\",\"model\":\"\",\"object\":\"chat.completion.chunk\",\"usage\":{\"completion_tokens\":437,\"prompt_tokens\":32328,\"total_tokens\":32765}}\n\n"                                                                                      
VERB [              start_loop] new task may arrive | tid="140737353117696" timestamp=1761065624                                                                                                                                                        
VERB [              start_loop] callback_new_task | tid="140737353117696" timestamp=1761065624 id_task=440                                                                                                                                              
VERB [              start_loop] update_multitasks | tid="140737353117696" timestamp=1761065624                                                                                                                                                          
VERB [              start_loop] callback_update_slots | tid="140737353117696" timestamp=1761065624                                                                                                                                                      
VERB [            update_slots] posting NEXT_RESPONSE | tid="140737353117696" timestamp=1761065624                                                                                                                                                      
VERB [                    post] new task id | tid="140737353117696" timestamp=1761065624 new_id=441                                                                                                                                                     
VERB [            update_slots] slot decode token | tid="140737353117696" timestamp=1761065624 id_slot=0 id_task=0 n_ctx=65536 n_past=32765 n_system_tokens=0 n_cache_tokens=32765 truncated=false                                                      
VERB [            update_slots] decoding batch | tid="140737353117696" timestamp=1761065624 n_tokens=1                                                                                                                                                  
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context                                                                                                                                                                
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context                                                                                                                                                                
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context                                                                                                                                                                
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context                                                                                                                                                                
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context                                                                                                                                                                
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context                                                                                                                                                                
VERB [                    send] send new result | tid="140737353117696" timestamp=1761065625 id_task=0                                                                                                                                                  
VERB [                    send] queue_results.push_back | tid="140737353117696" timestamp=1761065625 id_task=0                                                                                                                                          
VERB [           process_token] next token | tid="140737353117696" timestamp=1761065625 id_slot=0 id_task=0 token=624 token_text=" if" has_next_token=true n_remain=-1 n_decoded=438 stopped_eos=false stopped_word=false stopped_limit=false stopping_w
ord=""                                                                                                                                                                                                                                                  
VERB [            update_slots] run slots completed | tid="140737353117696" timestamp=1761065625                                                                                                                                                        
VERB [              start_loop] wait for new task | tid="140737353117696" timestamp=1761065625                                                                                                                                                          
VERB [              start_loop] new task may arrive | tid="140737353117696" timestamp=1761065625                                                                                                                                                        
VERB [              start_loop] callback_new_task | tid="140737353117696" timestamp=1761065625 id_task=441                                                                                                                                              
VERB [              start_loop] update_multitasks | tid="140737353117696" timestamp=1761065625                                                                                                                                                          
VERB [              start_loop] callback_update_slots | tid="140737353117696" timestamp=1761065625                                                                                                                                                      
VERB [            update_slots] posting NEXT_RESPONSE | tid="140737353117696" timestamp=1761065625                                                                                                                                                      
VERB [                    post] new task id | tid="140737353117696" timestamp=1761065625 new_id=442                                                                                                                                                     
VERB [            update_slots] slot decode token | tid="140737353117696" timestamp=1761065625 id_slot=0 id_task=0 n_ctx=65536 n_past=32766 n_system_tokens=0 n_cache_tokens=32766 truncated=false                                                      
VERB [              operator()] data stream | tid="140176089067520" timestamp=1761065625 to_send="data: {\"choices\":[{\"finish_reason\":null,\"index\":0,\"delta\":{\"content\":\" if\"}}],\"created\":1761065625,\"id\":\"chatcmpl-RFVy0UY3XwLTNDVdUkb
vVnR897yrZki5\",\"model\":\"\",\"object\":\"chat.completion.chunk\",\"usage\":{\"completion_tokens\":438,\"prompt_tokens\":32328,\"total_tokens\":32766}}\n\n"                                                                                          
VERB [            update_slots] decoding batch | tid="140737353117696" timestamp=1761065625 n_tokens=1                                                                                                                                                  
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context                                                                                                                                                                
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context                                                                                                                                                                
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context                                                                                                                                                                
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context                                                                                                                                                                
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context                                                                                                                                                                
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context                                                                                                                                                                
VERB [                    send] send new result | tid="140737353117696" timestamp=1761065625 id_task=0                                                                                                                                                  
VERB [                    send] queue_results.push_back | tid="140737353117696" timestamp=1761065625 id_task=0                                                                                                                                          
VERB [           process_token] next token | tid="140737353117696" timestamp=1761065625 id_slot=0 id_task=0 token=363 token_text=" (" has_next_token=true n_remain=-1 n_decoded=439 stopped_eos=false stopped_word=false stopped_limit=false stopping_wo
rd=""                                                                                                                                                                                                                                                   
VERB [            update_slots] run slots completed | tid="140737353117696" timestamp=1761065625                                                                                                                                                        
VERB [              start_loop] wait for new task | tid="140737353117696" timestamp=1761065625                                                                                                                                                          
VERB [              start_loop] new task may arrive | tid="140737353117696" timestamp=1761065625                                                                                                                                                        
VERB [              start_loop] callback_new_task | tid="140737353117696" timestamp=1761065625 id_task=442                                                                                                                                              
VERB [              start_loop] update_multitasks | tid="140737353117696" timestamp=1761065625                                                                                                                                                          
VERB [              start_loop] callback_update_slots | tid="140737353117696" timestamp=1761065625                                                                                                                                                      
VERB [              operator()] data stream | tid="140176089067520" timestamp=1761065625 to_send="data: {\"choices\":[{\"finish_reason\":null,\"index\":0,\"delta\":{\"content\":\" (\"}}],\"created\":1761065625,\"id\":\"chatcmpl-RFVy0UY3XwLTNDVdUkbv
VnR897yrZki5\",\"model\":\"\",\"object\":\"chat.completion.chunk\",\"usage\":{\"completion_tokens\":439,\"prompt_tokens\":32328,\"total_tokens\":32767}}\n\n"                                                                                           
VERB [            update_slots] posting NEXT_RESPONSE | tid="140737353117696" timestamp=1761065625                                                                                                                                                      
VERB [                    post] new task id | tid="140737353117696" timestamp=1761065625 new_id=443                                                                                                                                                     
VERB [            update_slots] slot decode token | tid="140737353117696" timestamp=1761065625 id_slot=0 id_task=0 n_ctx=65536 n_past=32767 n_system_tokens=0 n_cache_tokens=32767 truncated=false                                                      
VERB [            update_slots] decoding batch | tid="140737353117696" timestamp=1761065625 n_tokens=1                                                                                                                                                  
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context                                                                                                                                                                
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context                                                                                                                                                                
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context                                                                                                                                                                
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context                                                                                                                                                                
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context                                                                                                                                                                
warning: Cuda Driver error detected: CUDA Stream does not belong to the expected context                                                                                                                                                                
VERB [                    send] send new result | tid="140737353117696" timestamp=1761065625 id_task=0                                                                                                                                                  
VERB [                    send] queue_results.push_back | tid="140737353117696" timestamp=1761065625 id_task=0                                                                                                                                          
WARN [           process_token] n_predict is not set and self-context extend is disabled. Limiting generated tokens to n_ctx_train to avoid EOS-less generation infinite loop | tid="140737353117696" timestamp=1761065625 id_slot=0 params.n_predict=-1
 slot.n_prompt_tokens=32328 slot.n_decoded=440 slot.n_predict=-1 n_slots=1 slot.n_ctx=65536 n_ctx=65536 n_ctx_train=32768 ga_n=1                                                                                                                        
VERB [           process_token] next token | tid="140737353117696" timestamp=1761065625 id_slot=0 id_task=0 token=72908 token_text="ratio" has_next_token=false n_remain=-1 n_decoded=440 stopped_eos=false stopped_word=false stopped_limit=true stoppi
ng_word=""                                                                                                                                                                                                                                              
VERB [              operator()] data stream | tid="140176089067520" timestamp=1761065625 to_send="data: {\"choices\":[{\"finish_reason\":null,\"index\":0,\"delta\":{\"content\":\"ratio\"}}],\"created\":1761065625,\"id\":\"chatcmpl-RFVy0UY3XwLTNDVdU
kbvVnR897yrZki5\",\"model\":\"\",\"object\":\"chat.completion.chunk\",\"usage\":{\"completion_tokens\":440,\"prompt_tokens\":32328,\"total_tokens\":32768}}\n\n"                                                                                        
INFO [           print_timings] prompt eval time     =  170675.16 ms / 32328 tokens (    5.28 ms per token,   189.41 tokens per second) | tid="140737353117696" timestamp=1761065625 id_slot=0 id_task=0 t_prompt_processing=170675.156 n_prompt_tokens_
processed=32328 t_token=5.279483914872556 n_tokens_second=189.4124532102376                                                 
INFO [           print_timings] generation eval time =  158055.39 ms /   440 runs   (  359.22 ms per token,     2.78 tokens per second) | tid="140737353117696" timestamp=1761065625 id_slot=0 id_task=0 t_token_generation=158055.391 n_decoded=440 t_t
oken=359.21679772727276 n_tokens_second=2.783834181271299     
INFO [           print_timings]           total time =  328730.55 ms | tid="140737353117696" timestamp=1761065625 id_slot=0 id_task=0 t_prompt_processing=170675.156 t_token_generation=158055.391 t_total=328730.547
VERB [                    send] send new result | tid="140737353117696" timestamp=1761065625 id_task=0
VERB [                    send] queue_results.push_back | tid="140737353117696" timestamp=1761065625 id_task=0
VERB [            update_slots] run slots completed | tid="140737353117696" timestamp=1761065625
VERB [              start_loop] wait for new task | tid="140737353117696" timestamp=1761065625
VERB [              start_loop] new task may arrive | tid="140737353117696" timestamp=1761065625
VERB [              start_loop] callback_new_task | tid="140737353117696" timestamp=1761065625 id_task=443
VERB [              start_loop] update_multitasks | tid="140737353117696" timestamp=1761065625
VERB [              start_loop] callback_update_slots | tid="140737353117696" timestamp=1761065625
INFO [            update_slots] slot released | tid="140737353117696" timestamp=1761065625 id_slot=0 id_task=0 n_ctx=65536 n_past=32767 n_system_tokens=0 n_cache_tokens=32767 truncated=true                                                           
INFO [            update_slots] all slots are idle | tid="140737353117696" timestamp=1761065625
VERB [              start_loop] wait for new task | tid="140737353117696" timestamp=1761065625
VERB [              operator()] data stream | tid="140176089067520" timestamp=1761065625 to_send="data: {\"choices\":[{\"finish_reason\":\"stop\",\"index\":0,\"delta\":{}}],\"created\":1761065625,\"id\":\"chatcmpl-RFVy0UY3XwLTNDVdUkbvVnR897yrZki5\"
,\"model\":\"Ling-1T-smol-IQ4_KSS\",\"object\":\"chat.completion.chunk\"}\n\n"                                              
VERB [              operator()] data stream | tid="140176089067520" timestamp=1761065625 to_send="data: {\"choices\":[],\"created\":1761065625,\"id\":\"chatcmpl-RFVy0UY3XwLTNDVdUkbvVnR897yrZki5\",\"model\":\"Ling-1T-smol-IQ4_KSS\",\"object\":\"chat
.completion.chunk\",\"usage\":{\"completion_tokens\":440,\"prompt_tokens\":32328,\"total_tokens\":32768},\"timings\":{\"prompt_n\":32328,\"prompt_ms\":170675.156,\"prompt_per_token_ms\":5.279483914872556,\"prompt_per_second\":189.4124532102376,\"pr
edicted_n\":440,\"predicted_ms\":158055.391,\"predicted_per_token_ms\":359.21679772727276,\"predicted_per_second\":2.783834181271299}}\n\n"
VERB [              operator()] data stream | tid="140176089067520" timestamp=1761065625 to_send="data: [DONE]\n\n"
VERB [  remove_waiting_task_id] remove waiting for task id | tid="140176089067520" timestamp=1761065625 id_task=0
INFO [      log_server_request] request | tid="140176089067520" timestamp=1761065625 remote_addr="192.168.1.100" remote_port=58084 status=200 method="POST" path="/v1/chat/completions" params={}         
```

---

👤 **ikawrakow** commented on **2025-10-21** at **17:03:48**

Do you remember when you started observing these issues?

---

👤 **magikRUKKOLA** commented on **2025-10-21** at **18:49:13**

@ikawrakow 
> Do you remember when you started observing these issues?

Unfortunately, I do not even remember if I ever ran Ling-T1 without issues.  I just started the testing.
I can do the following -- I can try each commit inrementally up to main to see if that solves the issue.  What other cli options I can alter besides -ger for each iteration?  Can you pinpoint from what commit I should start ?

@ikawrakow 
Alternatively, if you meant those warnings as the issues, then its hard to say since its basically my first time using gdb for ik_llama.cpp etc.

---

👤 **magikRUKKOLA** commented on **2025-10-21** at **21:41:05**

@ikawrakow 

Ling suggested to do the following:

```diff
--- /opt/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml-cuda/argsort.cu	2025-10-21 16:43:05.093964951 +0000
+++ /tmp/argsort.cu	2025-10-21 21:27:36.510256406 +0000
@@ -59,7 +59,7 @@
 //        int min_experts, float thresh_experts) {
     // bitonic sort
     int col = threadIdx.x;
-    int row = blockIdx.y;
+    int row = blockIdx.x;  // ← CHANGED from blockIdx.y
 
     if (col >= ncols_pad) {
         return;
@@ -101,7 +101,7 @@
         size_t nb_ids) {
     // bitonic sort
     int col = threadIdx.x;
-    int row = blockIdx.y;
+    int row = blockIdx.x;  // ← CHANGED from blockIdx.y
 
     if (col >= ncols_pad) {
         return;
@@ -129,7 +129,7 @@
         size_t nb_ids) {
     // bitonic sort
     int col = threadIdx.x;
-    int row = blockIdx.y;
+    int row = blockIdx.x;  // ← CHANGED from blockIdx.y
 
     if (col >= ncols_pad) {
         return;
@@ -158,7 +158,7 @@
         size_t nb_ids) {
     // bitonic sort
     int col = threadIdx.x;
-    int row = blockIdx.y;
+    int row = blockIdx.x;  // ← CHANGED from blockIdx.y
 
     if (col >= ncols_pad) {
         return;
@@ -204,7 +204,7 @@
 static __global__ void k_topk_sum(const float * x, const float * bias, float * x_p, float * dst, const int ncols, int ncols_pad, int n_top_k) {
     // bitonic sort
     int col = threadIdx.x;
-    int row = blockIdx.y;
+    int row = blockIdx.x;  // ← CHANGED from blockIdx.y
 
     if (col >= ncols_pad) {
         return;
@@ -252,7 +252,7 @@
 
 static __global__ void k_apply_mask(float * dst, const int * groups,
         const int n_top_groups, const int n_per_group, const int ncols) {
-    int row = blockIdx.x;
+    int row = blockIdx.x;  // ← CHANGED from blockIdx.x (was correct)
     for (int col = threadIdx.x; col < n_top_groups*n_per_group; col += blockDim.x) {
         int ig = groups[row*n_top_groups + col / n_per_group];
         int ic = col % n_per_group;
@@ -275,7 +275,7 @@
     const int ncols_pad = next_power_of_2(ncols);
 
     const dim3 block_dims(ncols_pad, 1, 1);
-    const dim3 block_nums(1, nrows, 1);
+    const dim3 grid_dims(nrows, 1, 1);  // ← CHANGED: (nrows,1) instead of (1,nrows)
     const size_t shared_mem = ncols_pad * sizeof(int);
 
     // FIXME: this limit could be raised by ~2-4x on Ampere or newer
@@ -283,17 +283,17 @@
 
     if (order == GGML_SORT_ORDER_ASC) {
         if (min_experts >= 0 && min_experts < ncols && thresh_experts > 0) {
-            k_argsort_f32_T<GGML_SORT_ORDER_ASC, store_ser><<<block_nums, block_dims, shared_mem, stream>>>(x, dst, ncols, ncols_pad,
+            k_argsort_f32_T<GGML_SORT_ORDER_ASC, store_ser><<<grid_dims, block_dims, shared_mem, stream>>>(x, dst, ncols, ncols_pad,
                     ntop, {min_experts, thresh_experts});
         } else {
-            k_argsort_f32_T<GGML_SORT_ORDER_ASC, store><<<block_nums, block_dims, shared_mem, stream>>>(x, dst, ncols, ncols_pad, ntop, {});
+            k_argsort_f32_T<GGML_SORT_ORDER_ASC, store><<<grid_dims, block_dims, shared_mem, stream>>>(x, dst, ncols, ncols_pad, ntop, {});
         }
     } else if (order == GGML_SORT_ORDER_DESC) {
         if (min_experts >= 0 && min_experts < ncols && thresh_experts > 0) {
-            k_argsort_f32_T<GGML_SORT_ORDER_DESC, store_ser><<<block_nums, block_dims, shared_mem, stream>>>(x, dst, ncols, ncols_pad,
+            k_argsort_f32_T<GGML_SORT_ORDER_DESC, store_ser><<<grid_dims, block_dims, shared_mem, stream>>>(x, dst, ncols, ncols_pad,
                     ntop, {min_experts, thresh_experts});
         } else {
-            k_argsort_f32_T<GGML_SORT_ORDER_DESC, store><<<block_nums, block_dims, shared_mem, stream>>>(x, dst, ncols, ncols_pad, ntop, {});
+            k_argsort_f32_T<GGML_SORT_ORDER_DESC, store><<<grid_dims, block_dims, shared_mem, stream>>>(x, dst, ncols, ncols_pad, ntop, {});
         }
     } else {
         GGML_ABORT("fatal error");
@@ -306,17 +306,17 @@
     const int ncols_pad = next_power_of_2(ncols);
 
     const dim3 block_dims(ncols_pad, 1, 1);
-    const dim3 block_nums(1, nrows, 1);
+    const dim3 grid_dims(nrows, 1, 1);  // ← CHANGED
     const size_t shared_mem = ncols_pad * sizeof(int);
 
     // FIXME: this limit could be raised by ~2-4x on Ampere or newer
     GGML_ASSERT(shared_mem <= ggml_cuda_info().devices[ggml_cuda_get_device()].smpb);
 
     if (order == GGML_SORT_ORDER_ASC) {
-        k_argsort_f32_f32_i32<GGML_SORT_ORDER_ASC><<<block_nums, block_dims, shared_mem, stream>>>(x_biased, x, weights, ids,
+        k_argsort_f32_f32_i32<GGML_SORT_ORDER_ASC><<<grid_dims, block_dims, shared_mem, stream>>>(x_biased, x, weights, ids,
                 ncols, ncols_pad, ntop, nb_ids);
     } else if (order == GGML_SORT_ORDER_DESC) {
-        k_argsort_f32_f32_i32<GGML_SORT_ORDER_DESC><<<block_nums, block_dims, shared_mem, stream>>>(x_biased, x, weights, ids,
+        k_argsort_f32_f32_i32<GGML_SORT_ORDER_DESC><<<grid_dims, block_dims, shared_mem, stream>>>(x_biased, x, weights, ids,
                 ncols, ncols_pad, ntop, nb_ids);
     } else {
         GGML_ABORT("fatal error");
@@ -329,17 +329,17 @@
     const int ncols_pad = next_power_of_2(ncols);
 
     const dim3 block_dims(ncols_pad, 1, 1);
-    const dim3 block_nums(1, nrows, 1);
+    const dim3 grid_dims(nrows, 1, 1);  // ← CHANGED
     const size_t shared_mem = ncols_pad * (sizeof(int) + sizeof(float));
 
     // FIXME: this limit could be raised by ~2-4x on Ampere or newer
     GGML_ASSERT(shared_mem <= ggml_cuda_info().devices[ggml_cuda_get_device()].smpb);
 
     if (order == GGML_SORT_ORDER_ASC) {
-        k_argsort_biased_f32_f32_i32<GGML_SORT_ORDER_ASC><<<block_nums, block_dims, shared_mem, stream>>>(x, bias, weights, ids,
+        k_argsort_biased_f32_f32_i32<GGML_SORT_ORDER_ASC><<<grid_dims, block_dims, shared_mem, stream>>>(x, bias, weights, ids,
                 ncols, ncols_pad, ntop, nb_ids);
     } else if (order == GGML_SORT_ORDER_DESC) {
-        k_argsort_biased_f32_f32_i32<GGML_SORT_ORDER_DESC><<<block_nums, block_dims, shared_mem, stream>>>(x, bias, weights, ids,
+        k_argsort_biased_f32_f32_i32<GGML_SORT_ORDER_DESC><<<grid_dims, block_dims, shared_mem, stream>>>(x, bias, weights, ids,
                 ncols, ncols_pad, ntop, nb_ids);
     } else {
         GGML_ABORT("fatal error");
@@ -352,17 +352,17 @@
     const int ncols_pad = next_power_of_2(ncols);
 
     const dim3 block_dims(ncols_pad, 1, 1);
-    const dim3 block_nums(1, nrows, 1);
+    const dim3 grid_dims(nrows, 1, 1);  // ← CHANGED
     const size_t shared_mem = (ncols_pad + ncols_pad > WARP_SIZE ? WARP_SIZE : 0) * sizeof(int);
 
     // FIXME: this limit could be raised by ~2-4x on Ampere or newer
     GGML_ASSERT(shared_mem <= ggml_cuda_info().devices[ggml_cuda_get_device()].smpb);
 
     if (order == GGML_SORT_ORDER_ASC) {
-        k_openai_f32_f32_i32<GGML_SORT_ORDER_ASC><<<block_nums, block_dims, shared_mem, stream>>>(x, weights, ids,
+        k_openai_f32_f32_i32<GGML_SORT_ORDER_ASC><<<grid_dims, block_dims, shared_mem, stream>>>(x, weights, ids,
                 ncols, ncols_pad, ntop, nb_ids);
     } else if (order == GGML_SORT_ORDER_DESC) {
-        k_openai_f32_f32_i32<GGML_SORT_ORDER_DESC><<<block_nums, block_dims, shared_mem, stream>>>(x, weights, ids,
+        k_openai_f32_f32_i32<GGML_SORT_ORDER_DESC><<<grid_dims, block_dims, shared_mem, stream>>>(x, weights, ids,
                 ncols, ncols_pad, ntop, nb_ids);
     } else {
         GGML_ABORT("fatal error");
@@ -415,11 +415,11 @@
     const int ncols_pad = next_power_of_2(ncols);
 
     const dim3 block_dims(ncols_pad, 1, 1);
-    const dim3 block_nums(1, nrows, 1);
+    const dim3 grid_dims(nrows, 1, 1);  // ← CHANGED
     const size_t shared_mem = (ncols_pad + WARP_SIZE) * sizeof(int);
     GGML_ASSERT(shared_mem <= ggml_cuda_info().devices[ggml_cuda_get_device()].smpb);
 
-    k_topk_sum<GGML_SORT_ORDER_DESC><<<block_nums, block_dims, shared_mem, ctx.stream()>>>(src, bias, src_p, dst, ncols, ncols_pad, n_top_k);
+    k_topk_sum<GGML_SORT_ORDER_DESC><<<grid_dims, block_dims, shared_mem, ctx.stream()>>>(src, bias, src_p, dst, ncols, ncols_pad, n_top_k);
 }
 
 void ggml_cuda_op_grouped_topk(ggml_backend_cuda_context & ctx, ggml_tensor * dst) {
@@ -463,9 +463,9 @@
 
     {
         const dim3 block_dims(WARP_SIZE, 1, 1);
-        const dim3 block_nums(nrows, 1, 1);
+        const dim3 grid_dims_apply(nrows, 1, 1);  // ← CHANGED
         cudaStream_t stream = ctx.stream();
-        k_apply_mask<<<block_nums, block_dims, 0, ctx.stream()>>>((float *)src->data, discarded_groups.get(), n_discarded_groups, n_per_group, ne00);
+        k_apply_mask<<<grid_dims_apply, block_dims, 0, ctx.stream()>>>((float *)src->data, discarded_groups.get(), n_discarded_groups, n_per_group, ne00);
         CUDA_CHECK(cudaGetLastError());
     }
 
@@ -508,8 +508,8 @@
 
     {
         const dim3 block_dims(WARP_SIZE, 1, 1);
-        const dim3 block_nums(nrows, 1, 1);
-        k_apply_mask<<<block_nums, block_dims, 0, ctx.stream()>>>((float *)topk_src->data, discarded_groups.get(), n_discarded_groups, n_per_group, ne00);
+        const dim3 grid_dims_apply(nrows, 1, 1);  // ← CHANGED
+        k_apply_mask<<<grid_dims_apply, block_dims, 0, ctx.stream()>>>((float *)topk_src->data, discarded_groups.get(), n_discarded_groups, n_per_group, ne00);
         CUDA_CHECK(cudaGetLastError());
     }
 
@@ -555,9 +555,8 @@
         printf("Oops: ntop = %d, ne0 = %d\n", ntop, ne0);
         GGML_ASSERT(false);
     }
-    //GGML_ASSERT(ne0 == ntop);
 
     argsort_openai_f32_f32_i32_cuda((const float *)probs->data, (float *)softmax->data, (int *)topk->data,
             ne00, nrows, ne0, topk->nb[1], GGML_SORT_ORDER_DESC, ctx.stream());
 
-}
+}
\ No newline at end of file
```
It does not solve the cut-off issue but I noticed that it cuts off exactly when the ctx=32k:


```
VERB [              operator()] data stream | tid="140176248528896" timestamp=1761082583 to_send="data: {\"choices\":[{\"fin
ish_reason\":null,\"index\":0,\"delta\":{\"content\":\"];\"}}],\"created\":1761082583,\"id\":\"chatcmpl-B3ENE7oRWOJ4VAXV2aY7
cBrmhHCSL7mh\",\"model\":\"\",\"object\":\"chat.completion.chunk\",\"usage\":{\"completion_tokens\":440,\"prompt_tokens\":32
328,\"total_tokens\":32768}}\n\n"
```

Yeah, and then it stops.

The exact command I ran:

```
/opt/ik_llama.cpp/ik_llama.cpp/build/bin/llama-server --ctx-size 65535 --model /opt/ubergarm/Ling-1T-GGUF/smol-IQ4-KSS/Ling-1T-smol-IQ4_KSS-00001-of-00011.gguf --alias ubergarm/Ling-1T-smol-IQ4_KSS -b 8192 -ub 8192 --mlock --temp 0.7 --top-k 0 --top-p 0.95 --min-p 0.1 --repeat-penalty 1.1 -ctk q8_0 -ctv q8_0 -fa -fmoe -ger -amb 512 --split-mode layer --tensor-split 32,40 --main-gpu 1 --override-tensor exps=CPU -no-ooae --n-gpu-layers 99 --threads 12 --host 0.0.0.0 --port 8080 --log-enable --logdir /var/log/ --jinja --special --verbosity 2 --verbose-prompt --reasoning-format auto --prompt-cache /root/.cache/ik_llama.cpp/prompt-cache.bin --prompt-cache-all --slot-save-path /root/.cache/ik_llama.cpp/slot.bin --lookup-cache-dynamic /root/.cache/ik_llama.cpp/slot.bin --keep -1 --slot-prompt-similarity 0.35 --metrics
```

So 64k context supposed to be available but ...

Ah lol.  Ling supports only 32k context!


https://huggingface.co/inclusionAI/Ling-1T
> Model	Context Length	Download
> Ling-1T	32K -> 128K (YaRN)	[🤗 HuggingFace](https://huggingface.co/inclusionAI/Ling-1T)    [🤖 ModelScope](https://www.modelscope.cn/models/inclusionAI/Ling-1T)

Its extendable by YaRN ?

---

👤 **ikawrakow** commented on **2025-10-22** at **04:59:33**

@magikRUKKOLA 

Yes, I had noticed the `blockIds.y` issue myself, and the latest version of [#853](https://github.com/ikawrakow/ik_llama.cpp/issues/853) does that. But I have also disabled op fusion altogether in [#853](https://github.com/ikawrakow/ik_llama.cpp/issues/853) to see if this solves the issues you are observing. Can you try PR [#853](https://github.com/ikawrakow/ik_llama.cpp/issues/853) again? Thanks.

YaRN is supported by `ik_llama.cpp`. The YaRN stuff predates the fork by quite some time.

---

👤 **magikRUKKOLA** commented on **2025-10-22** at **18:37:18**

@ikawrakow 
> Can you try PR [[#853](https://github.com/ikawrakow/ik_llama.cpp/issues/853)](https://github.com/ikawrakow/ik_llama.cpp/pull/853) again? Thanks.

Sorry for being late.
I can see its in the master already.  I tried it.  Its the same as before:  https://github.com/ikawrakow/ik_llama.cpp/issues/854#issuecomment-3427492304

No crashes, just a bunch of warnings.