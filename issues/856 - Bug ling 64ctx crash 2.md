## 📌 [Issue #856](https://github.com/ikawrakow/ik_llama.cpp/issues/856) - Bug: ling 64ctx crash [#2](https://github.com/ikawrakow/ik_llama.cpp/issues/2)

| **Author** | `magikRUKKOLA` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-10-21 |
| **Updated** | 2025-10-21 |

---

## 📄 Description

### What happened?

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

### Name and Version

build/bin/llama-server --version
version: 3922 (caf9759c)
built with cc (Debian 15.2.0-4) 15.2.0 for x86_64-linux-gnu

(with the patch from the first issue applied)

### What operating system are you seeing the problem on?

_No response_

### Relevant log output

```shell

```