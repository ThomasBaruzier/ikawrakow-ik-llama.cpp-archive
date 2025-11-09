## 📌 [Issue #733](https://github.com/ikawrakow/ik_llama.cpp/issues/733) - Bug: mmq.cuh:4254: fatal error

| **Author** | `magikRUKKOLA` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-08-26 |
| **Updated** | 2025-10-08 |

---

## 📄 Description

### What happened?

```
export MALLOC_CONF="background_thread:true,percpu_arena:phycpu,metadata_thp:auto,dirty_decay_ms:10000,muzzy_decay_ms:60000"
export LD_PRELOAD=/usr/local/lib/libjemalloc.so

#    --seed 3407 \
#    -fmoe \

ulimit -n 9999
ulimit -l unlimited

CUDA_VISIBLE_DEVICES="0,1" \
/opt/ik_llama.cpp/ik_llama.cpp/build/bin/llama-server \
    --model /mnt/data/opt/THIREUS-R1-3.5652bpw/DeepSeek-R1-0528-THIREUS-BF16-SPECIAL_TENSOR-00001-of-01148.gguf \
    --alias THIREUS/DeepSeek-R1-0528-3.5652bpw \
    --ctx-size $((128 * 1024)) \
    --temp 0.5 --top-k 0 --top-p 1.0 --min-p 0.1 --repeat-penalty 1.0 \
    -ctk q8_0 \
    -mla 3 -fa \
    -amb 512 \
    -b $((4 * 1024)) -ub $((2 * 1024)) \
    -fmoe \
    --split-mode layer \
    --tensor-split 7,8 \
    --main-gpu 1 \
    --override-tensor exps=CPU \
    --n-gpu-layers 99 \
    --threads $(grep ^cpu\\scores /proc/cpuinfo | uniq | awk '{print $4}' | xargs -I{} echo "{}-0" | bc) \
    --host 0.0.0.0 \
    --port 8080 \
    --lookup-cache-dynamic /mnt/data/ik_llama.kv.dump \
    --log-enable \
    --logdir /var/log/ \
    --jinja \
    --special \
    --verbose-prompt --verbosity 2
```

### Name and Version

/opt/ik_llama.cpp/ik_llama.cpp/build/bin/llama-server --version
version: 3860 (0cc32ff0)
built with cc (Debian 14.2.0-19) 14.2.0 for x86_64-linux-gnu

### What operating system are you seeing the problem on?

_No response_

### Relevant log output

```shell
VERB [            update_slots] prompt tokenized | tid="139832978124800" timestamp=1756227110 id_slot=0 id_task=0 n_ctx=131072 n_keep=0 n_pr
ompt_tokens=50 prompt_tokens="<｜begin▁of▁sentence｜><｜begin▁of▁sentence｜><｜User｜>Imagine a runaway trolley is hurtling down a track tow
ards five dead people. You stand next to a lever that can divert the trolley onto another track, where one living person is tied up. Do you
pull the lever?<｜Assistant｜>"
INFO [            update_slots] kv cache rm [p0, end) | tid="139832978124800" timestamp=1756227110 id_slot=0 id_task=0 p0=0
VERB [            update_slots] prompt processing progress | tid="139832978124800" timestamp=1756227110 id_slot=0 n_past=50 n_ctx=131072 n_t
okens=50 progress=1.0
VERB [            update_slots] prompt done | tid="139832978124800" timestamp=1756227110 id_slot=0 n_past=50 n_ctx=131072 n_tokens=50
VERB [            update_slots] decoding batch | tid="139832978124800" timestamp=1756227110 n_tokens=50
mmq_x_best=0
/opt/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml-cuda/template-instances/../mmq.cuh:4254: fatal error
[New LWP 26792]
[New LWP 26791]
[New LWP 26790]
[New LWP 26789]
[New LWP 26788]
[New LWP 26787]
[New LWP 26786]
[New LWP 26785]
[New LWP 26784]
[New LWP 26783]
[New LWP 26782]
[New LWP 26781]
[New LWP 26780]
[New LWP 26779]
[New LWP 26778]
[New LWP 26777]
[New LWP 26776]
[New LWP 26775]
[New LWP 26774]
[New LWP 26773]
[New LWP 26772]
[New LWP 26771]
[New LWP 26770]
[New LWP 26769]
[New LWP 26768]
[New LWP 26767]
[New LWP 26766]
[New LWP 26765]
[New LWP 26764]
[New LWP 26763]
[New LWP 26762]
[New LWP 26761]
[New LWP 26760]
[New LWP 26759]
[New LWP 26758]
[New LWP 26757]
[New LWP 26756]
[New LWP 26755]
[New LWP 26754]
[New LWP 26753]
[New LWP 26752]
[New LWP 26751]
...

[New LWP 26610]
[New LWP 26609]
[New LWP 26608]
[New LWP 26607]
[New LWP 26606]
[New LWP 26605]
[New LWP 26604]
[New LWP 26603]
[New LWP 26602]
[New LWP 26601]
[New LWP 26600]
[New LWP 26599]
[New LWP 26598]
[New LWP 26597]
[New LWP 26596]
[New LWP 26585]
[New LWP 26584]
[New LWP 26583]
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".
__syscall_cancel_arch () at ../sysdeps/unix/sysv/linux/x86_64/syscall_cancel.S:56
warning: 56     ../sysdeps/unix/sysv/linux/x86_64/syscall_cancel.S: No such file or directory
#0  __syscall_cancel_arch () at ../sysdeps/unix/sysv/linux/x86_64/syscall_cancel.S:56
56      in ../sysdeps/unix/sysv/linux/x86_64/syscall_cancel.S
#1  0x00007f2d43699668 in __internal_syscall_cancel (a1=<optimized out>, a2=<optimized out>, a3=<optimized out>, a4=<optimized out>, a5=a5@entry=0, a6=a6@entry=0, nr=61) at ./nptl/cancellation.c:49
warning: 49     ./nptl/cancellation.c: No such file or directory
#2  0x00007f2d436996ad in __syscall_cancel (a1=<optimized out>, a2=<optimized out>, a3=<optimized out>, a4=<optimized out>, a5=a5@entry=0, a6=a6@entry=0, nr=61) at ./nptl/cancellation.c:75
75      in ./nptl/cancellation.c
#3  0x00007f2d43704787 in __GI___wait4 (pid=<optimized out>, stat_loc=<optimized out>, options=<optimized out>, usage=<optimized out>) at ../sysdeps/unix/sysv/linux/wait4.c:30
warning: 30     ../sysdeps/unix/sysv/linux/wait4.c: No such file or directory
#4  0x00007f2d43cf31a8 in ggml_abort () from /opt/ik_llama.cpp/ik_llama.cpp/build/ggml/src/libggml.so
#5  0x00007f2d44513036 in void mul_mat_q_case<(ggml_type)340>(ggml_backend_cuda_context&, mmq_args const&, CUstream_st*) () from /opt/ik_llama.cpp/ik_llama.cpp/build/ggml/src/libggml.so
#6  0x00007f2d43e5ac44 in ggml_cuda_op_mul_mat_q(ggml_backend_cuda_context&, ggml_tensor const*, ggml_tensor const*, ggml_tensor*, char const*, float const*, char const*, float*, long, long, long, long, CUstream_st*) () from /opt/ik_llama.cpp/ik_llama.cpp/build/ggml/src/libggml.so
#7  0x00007f2d43f16ef3 in ggml_cuda_op_mul_mat(ggml_backend_cuda_context&, ggml_tensor const*, ggml_tensor const*, ggml_tensor*, void (*)(ggml_backend_cuda_context&, ggml_tensor const*, ggml_tensor const*, ggml_tensor*, char const*, float const*, char const*, float*, long, long, long, long, CUstream_st*), void (*)(float const*, void*, long, long, long, long, ggml_type, CUstream_st*)) [clone .constprop.1] () from /opt/ik_llama.cpp/ik_llama.cpp/build/ggml/src/libggml.so
#8  0x00007f2d43f1e6f0 in ggml_backend_cuda_graph_compute(ggml_backend*, ggml_cgraph*) () from /opt/ik_llama.cpp/ik_llama.cpp/build/ggml/src/libggml.so
#9  0x00007f2d43d492e4 in ggml_backend_sched_graph_compute_async () from /opt/ik_llama.cpp/ik_llama.cpp/build/ggml/src/libggml.so
#10 0x00007f2d670b06b6 in llama_decode () from /opt/ik_llama.cpp/ik_llama.cpp/build/src/libllama.so
#11 0x000055bca6865612 in server_context::update_slots() ()
#12 0x000055bca682d147 in server_queue::start_loop() ()
#13 0x000055bca67aa714 in main ()
[Inferior 1 (process 26582) detached]
/mnt/data/opt/THIREUS-R1-3.5652bpw/run-ik_llama.cpp.sh: line 55: 26582 Aborted                 CUDA_VISIBLE_DEVICES="0,1" /opt/ik_llama.cpp/ik_llama.cpp/build/bin/llama-server --model /mnt/data/opt/THIREUS-R1-3.5652bpw/DeepSeek-R1-0528-THIREUS-BF16-SPECIAL_TENSOR-00001-of-01148.gguf --alias THIREUS/DeepSeek-R1-0528-3.5652bpw --ctx-size $((128 * 1024)) --temp 0.5 --top-k 0 --top-p 1.0 --min-p 0.1 --repeat-penalty 1.0 -ctk q8_0 -mla 3 -fa -amb 512 -b $((4 * 1024)) -ub $((2 * 1024)) -fmoe --split-mode layer --tensor-split 7,8 --main-gpu 1 --override-tensor exps=CPU --n-gpu-layers 99 --threads $(grep ^cpu\\scores /proc/cpuinfo | uniq | awk '{print $4}' | xargs -I{} echo "{}-0" | bc) --host 0.0.0.0 --port 8080 --lookup-cache-dynamic /mnt/data/ik_llama.kv.dump --log-enable --logdir /var/log/ --jinja --special --verbose-prompt --verbosity 2
```

---

## 💬 Conversation

👤 **magikRUKKOLA** commented on **2025-08-26** at **17:07:55**

No crash when reverted before the commit [#728](https://github.com/ikawrakow/ik_llama.cpp/issues/728)

```
/opt/ik_llama.cpp/ik_llama.cpp/build/bin/llama-server --version
version: 3859 (50f7119d)
built with cc (Debian 14.2.0-19) 14.2.0 for x86_64-linux-gnu
```

---

👤 **ikawrakow** commented on **2025-09-02** at **12:16:46**

@magikRUKKOLA  This is still relevant? I see a new issue from you ([#750](https://github.com/ikawrakow/ik_llama.cpp/issues/750)), which is using [#748](https://github.com/ikawrakow/ik_llama.cpp/issues/748), and where this issue does not occur.

---

👤 **magikRUKKOLA** commented on **2025-10-07** at **03:01:33**

@ikawrakow 

Ha.  Still relevant at least for this specific case:

```
llm_load_tensors:        CPU buffer size = 43751.77 MiB
llm_load_tensors:        CPU buffer size = 43751.77 MiB
llm_load_tensors:        CPU buffer size = 43751.77 MiB
llm_load_tensors:        CPU buffer size = 43751.77 MiB
llm_load_tensors:        CPU buffer size = 43751.77 MiB
llm_load_tensors:        CPU buffer size = 43751.77 MiB
llm_load_tensors:        CPU buffer size = 43751.77 MiB
llm_load_tensors:        CPU buffer size = 43751.77 MiB
llm_load_tensors:        CPU buffer size = 43751.77 MiB
llm_load_tensors:        CPU buffer size = 44679.30 MiB
llm_load_tensors:        CPU buffer size =   630.00 MiB
llm_load_tensors:      CUDA0 buffer size =  2506.50 MiB
llm_load_tensors:      CUDA1 buffer size =  8902.93 MiB
....................................................................................................
llama_new_context_with_model: n_ctx      = 131072
llama_new_context_with_model: n_batch    = 8192
llama_new_context_with_model: n_ubatch   = 4096
llama_new_context_with_model: flash_attn = 1
llama_new_context_with_model: mla_attn   = 3
llama_new_context_with_model: attn_max_b = 512
llama_new_context_with_model: fused_moe  = 0
llama_new_context_with_model: fused_up_gate = 1
llama_new_context_with_model: ser        = -1, 0
llama_new_context_with_model: freq_base  = 50000.0
llama_new_context_with_model: freq_scale = 0.03125
llama_kv_cache_init:      CUDA0 KV buffer size =   994.51 MiB
llama_kv_cache_init:      CUDA1 KV buffer size =  3672.02 MiB
llama_new_context_with_model: KV self size  = 4666.50 MiB, c^KV (q8_0): 4666.50 MiB, kv^T: not used
llama_new_context_with_model:  CUDA_Host  output buffer size =     0.62 MiB
llama_new_context_with_model: pipeline parallelism enabled (n_copies=1)
llama_new_context_with_model:      CUDA0 compute buffer size =  9032.02 MiB
llama_new_context_with_model:      CUDA1 compute buffer size =  7376.03 MiB
llama_new_context_with_model:  CUDA_Host compute buffer size =  2160.05 MiB
llama_new_context_with_model: graph nodes  = 13649
llama_new_context_with_model: graph splits = 231

main: n_kv_max = 131072, n_batch = 8192, n_ubatch = 4096, flash_attn = 1, n_gpu_layers = 99, n_threads = 64, n_threads_batch = 64

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
mmq_x_best=0
/opt/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml-cuda/template-instances/../mmq.cuh:4255: fatal error

./run-ik_llama.cpp.sh: line 54:  8778 Aborted                    CUDA_VISIBLE_DEVICES="0,1" /opt/ik_llama.cpp/ik_llama.cpp/build/bin/llama-sweep-bench --warmup-batch --model /opt/ubergarm/Kimi-K2-Instruct-GGUF/IQ3_KS/Kimi-K2-Instruct-IQ3_KS-00001-of-00010.gguf --alias ubergarm/Kimi-K2-Instruct-GGUF_IQ3_KS --ctx-size $((128 * 1024)) -b $((16 * 512)) -ub $((8 * 512)) --mlock --seed 3407 --temp 0.5 --top-k 0 --top-p 1.0 --min-p 0.1 --repeat-penalty 1.0 -ctk q8_0 -mla 3 -fa -amb 512 --main-gpu 1 --tensor-split 1,4 --split-mode layer --override-tensor exps=CPU --n-gpu-layers 99 --threads $(grep ^cpu\\scores /proc/cpuinfo | uniq | awk '{print $4}' | xargs -I{} echo "{}-0" | bc) --host 0.0.0.0 --port 8080 --prompt-cache "$HOME/.cache/ik_llama.cpp/prompt-cache.bin" --prompt-cache-all --slot-save-path "$HOME/.cache/ik_llama.cpp/slot.bin" --lookup-cache-dynamic "$HOME/.cache/ik_llama.cpp/slot.bin" --keep -1 --slot-prompt-similarity 0.35 --metrics

root@yyy:/opt/ubergarm/Kimi-K2-Instruct-GGUF/IQ3_KS# /opt/ik_llama.cpp/ik_llama.cpp/build/bin/llama-server --version
version: 3902 (f649e36a)
built with cc (Debian 15.2.0-4) 15.2.0 for x86_64-linux-gnu
```

---

👤 **ikawrakow** commented on **2025-10-07** at **06:24:12**

You need to post the portion of the log that tells us how many tensors there are from what type.

---

👤 **magikRUKKOLA** commented on **2025-10-07** at **14:18:34**

@ikawrakow 
> You need to post the portion of the log that tells us how many tensors there are from what type.

```
./run-ik_llama.cpp.sh 
ggml_cuda_init: GGML_CUDA_FORCE_MMQ:    no
ggml_cuda_init: GGML_CUDA_FORCE_CUBLAS: no
ggml_cuda_init: found 2 CUDA devices:
  Device 0: NVIDIA GeForce RTX 3090, compute capability 8.6, VMM: yes
  Device 1: NVIDIA GeForce RTX 3090, compute capability 8.6, VMM: yes
llama_model_loader: additional 9 GGUFs metadata loaded.
llama_model_loader: loaded meta data with 49 key-value pairs and 1157 tensors from /opt/ubergarm/Kimi-K2-Instruct-GGUF/IQ3_KS/Kimi-K2-Instruct-IQ3_KS-00001-of-00010.gguf (version GGUF V3 (latest))
llama_model_loader: Dumping metadata keys/values. Note: KV overrides do not apply in this output.
llama_model_loader: - kv   0:                       general.architecture str              = deepseek2
llama_model_loader: - kv   1:                               general.type str              = model
llama_model_loader: - kv   2:                               general.name str              = Kimi K2 Instruct Bf16 Safetensors
llama_model_loader: - kv   3:                           general.finetune str              = Instruct-safetensors
llama_model_loader: - kv   4:                           general.basename str              = Kimi-K2
llama_model_loader: - kv   5:                         general.size_label str              = 384x15B
llama_model_loader: - kv   6:                      deepseek2.block_count u32              = 61
llama_model_loader: - kv   7:                   deepseek2.context_length u32              = 131072
llama_model_loader: - kv   8:                 deepseek2.embedding_length u32              = 7168
llama_model_loader: - kv   9:              deepseek2.feed_forward_length u32              = 18432
llama_model_loader: - kv  10:             deepseek2.attention.head_count u32              = 64
llama_model_loader: - kv  11:          deepseek2.attention.head_count_kv u32              = 64
llama_model_loader: - kv  12:                   deepseek2.rope.freq_base f32              = 50000.000000
llama_model_loader: - kv  13: deepseek2.attention.layer_norm_rms_epsilon f32              = 0.000001
llama_model_loader: - kv  14:                deepseek2.expert_used_count u32              = 8
llama_model_loader: - kv  15:                          general.file_type u32              = 154
llama_model_loader: - kv  16:        deepseek2.leading_dense_block_count u32              = 1
llama_model_loader: - kv  17:                       deepseek2.vocab_size u32              = 163840
llama_model_loader: - kv  18:            deepseek2.attention.q_lora_rank u32              = 1536
llama_model_loader: - kv  19:           deepseek2.attention.kv_lora_rank u32              = 512
llama_model_loader: - kv  20:             deepseek2.attention.key_length u32              = 192
llama_model_loader: - kv  21:           deepseek2.attention.value_length u32              = 128
llama_model_loader: - kv  22:       deepseek2.expert_feed_forward_length u32              = 2048
llama_model_loader: - kv  23:                     deepseek2.expert_count u32              = 384
llama_model_loader: - kv  24:              deepseek2.expert_shared_count u32              = 1
llama_model_loader: - kv  25:             deepseek2.expert_weights_scale f32              = 2.827000
llama_model_loader: - kv  26:              deepseek2.expert_weights_norm bool             = true
llama_model_loader: - kv  27:               deepseek2.expert_gating_func u32              = 2
llama_model_loader: - kv  28:             deepseek2.rope.dimension_count u32              = 64
llama_model_loader: - kv  29:                deepseek2.rope.scaling.type str              = yarn
llama_model_loader: - kv  30:              deepseek2.rope.scaling.factor f32              = 32.000000
llama_model_loader: - kv  31: deepseek2.rope.scaling.original_context_length u32              = 4096
llama_model_loader: - kv  32: deepseek2.rope.scaling.yarn_log_multiplier f32              = 0.100000
llama_model_loader: - kv  33:                       tokenizer.ggml.model str              = gpt2
llama_model_loader: - kv  34:                         tokenizer.ggml.pre str              = kimi-k2
llama_model_loader: - kv  35:                      tokenizer.ggml.tokens arr[str,163840]  = ["!", "\"", "#", "$", "%", "&", "'", ...
llama_model_loader: - kv  36:                  tokenizer.ggml.token_type arr[i32,163840]  = [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, ...
llama_model_loader: - kv  37:                      tokenizer.ggml.merges arr[str,163328]  = ["Ġ Ġ", "ĠĠ ĠĠ", "Ġ t", "i n",...
llama_model_loader: - kv  38:                tokenizer.ggml.bos_token_id u32              = 163584
llama_model_loader: - kv  39:                tokenizer.ggml.eos_token_id u32              = 163585
llama_model_loader: - kv  40:                    tokenizer.chat_template str              = {% if tools -%}\n    {{ '<|im_system|>...
llama_model_loader: - kv  41:               general.quantization_version u32              = 2
llama_model_loader: - kv  42:                      quantize.imatrix.file str              = /mnt/raid/models/ubergarm/Kimi-K2-Ins...
llama_model_loader: - kv  43:                   quantize.imatrix.dataset str              = ubergarm-imatrix-calibration-corpus-v...
llama_model_loader: - kv  44:             quantize.imatrix.entries_count i32              = 729
llama_model_loader: - kv  45:              quantize.imatrix.chunks_count i32              = 826
llama_model_loader: - kv  46:                                   split.no u16              = 0
llama_model_loader: - kv  47:                                split.count u16              = 10
llama_model_loader: - kv  48:                        split.tensors.count i32              = 1157
llama_model_loader: - type  f32:  365 tensors
llama_model_loader: - type q8_0:  610 tensors
llama_model_loader: - type iq4_k:    1 tensors
llama_model_loader: - type iq6_k:    1 tensors
llama_model_loader: - type iq4_ks:   60 tensors
llama_model_loader: - type iq3_ks:  120 tensors
init_tokenizer: initializing tokenizer for type 2
load: special_eos_id is not in special_eog_ids - the tokenizer config may be incorrect
load: printing all EOG tokens:
load:   - 163585 ('[EOS]')
load:   - 163586 ('<|im_end|>')
load: special tokens cache size = 256
load: token to piece cache size = 1.0607 MB
llm_load_print_meta: format           = GGUF V3 (latest)
llm_load_print_meta: arch             = deepseek2
llm_load_print_meta: n_ctx_train      = 131072
llm_load_print_meta: n_embd           = 7168
llm_load_print_meta: n_layer          = 61
llm_load_print_meta: n_head           = 64
llm_load_print_meta: n_head_kv        = 64
llm_load_print_meta: n_rot            = 64
llm_load_print_meta: n_swa            = 0
llm_load_print_meta: n_swa_pattern    = 1
llm_load_print_meta: n_embd_head_k    = 192
llm_load_print_meta: n_embd_head_v    = 128
llm_load_print_meta: n_gqa            = 1
llm_load_print_meta: n_embd_k_gqa     = 12288
llm_load_print_meta: n_embd_v_gqa     = 8192
llm_load_print_meta: f_norm_eps       = 0.0e+00
llm_load_print_meta: f_norm_rms_eps   = 1.0e-06
llm_load_print_meta: f_clamp_kqv      = 0.0e+00
llm_load_print_meta: f_max_alibi_bias = 0.0e+00
llm_load_print_meta: f_logit_scale    = 0.0e+00
llm_load_print_meta: n_ff             = 18432
llm_load_print_meta: n_expert         = 384
llm_load_print_meta: n_expert_used    = 8
llm_load_print_meta: causal attn      = 1
llm_load_print_meta: pooling type     = 0
llm_load_print_meta: rope type        = 0
llm_load_print_meta: rope scaling     = yarn
llm_load_print_meta: freq_base_train  = 50000.0
llm_load_print_meta: freq_scale_train = 0.03125
llm_load_print_meta: n_ctx_orig_yarn  = 4096
llm_load_print_meta: rope_finetuned   = unknown
llm_load_print_meta: ssm_d_conv       = 0
llm_load_print_meta: ssm_d_inner      = 0
llm_load_print_meta: ssm_d_state      = 0
llm_load_print_meta: ssm_dt_rank      = 0
llm_load_print_meta: model type       = 671B
llm_load_print_meta: model ftype      = IQ3_KS - 3.1875 bpw
llm_load_print_meta: model params     = 1.027 T
llm_load_print_meta: model size       = 430.908 GiB (3.604 BPW) 
llm_load_print_meta: repeating layers = 429.387 GiB (3.600 BPW, 1024.571 B parameters)
llm_load_print_meta: general.name     = Kimi K2 Instruct Bf16 Safetensors
llm_load_print_meta: n_layer_dense_lead   = 1
llm_load_print_meta: n_lora_q             = 1536
llm_load_print_meta: n_lora_kv            = 512
llm_load_print_meta: n_ff_exp             = 2048
llm_load_print_meta: n_expert_shared      = 1
llm_load_print_meta: expert_weights_scale = 2.8
llm_load_print_meta: expert_weights_norm  = 1
llm_load_print_meta: expert_gating_func   = sigmoid
llm_load_print_meta: rope_yarn_log_mul    = 0.1000
print_info: vocab type       = BPE
print_info: n_vocab          = 163840
print_info: n_merges         = 163328
print_info: BOS token        = 163584 '[BOS]'
print_info: EOS token        = 163585 '[EOS]'
print_info: EOT token        = 163586 '<|im_end|>'
print_info: LF token         = 198 'Ċ'
print_info: EOG token        = 163585 '[EOS]'
print_info: EOG token        = 163586 '<|im_end|>'
print_info: max token length = 512
llm_load_tensors: ggml ctx size =    1.41 MiB
Tensor blk.1.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.1.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.1.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.2.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.2.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.2.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.3.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.3.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.3.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.4.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.4.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.4.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.5.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.5.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.5.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.6.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.6.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.6.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.7.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.7.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.7.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.8.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.8.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.8.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.9.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.9.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.9.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.10.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.10.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.10.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.11.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.11.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.11.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.12.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.12.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.12.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.13.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.13.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.13.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.14.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.14.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.14.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.15.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.15.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.15.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.16.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.16.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.16.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.17.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.17.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.17.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.18.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.18.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.18.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.19.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.19.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.19.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.20.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.20.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.20.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.21.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.21.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.21.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.22.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.22.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.22.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.23.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.23.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.23.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.24.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.24.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.24.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.25.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.25.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.25.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.26.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.26.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.26.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.27.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.27.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.27.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.28.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.28.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.28.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.29.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.29.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.29.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.30.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.30.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.30.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.31.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.31.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.31.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.32.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.32.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.32.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.33.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.33.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.33.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.34.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.34.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.34.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.35.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.35.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.35.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.36.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.36.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.36.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.37.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.37.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.37.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.38.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.38.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.38.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.39.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.39.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.39.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.40.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.40.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.40.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.41.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.41.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.41.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.42.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.42.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.42.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.43.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.43.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.43.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.44.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.44.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.44.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.45.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.45.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.45.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.46.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.46.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.46.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.47.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.47.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.47.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.48.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.48.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.48.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.49.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.49.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.49.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.50.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.50.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.50.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.51.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.51.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.51.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.52.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.52.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.52.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.53.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.53.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.53.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.54.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.54.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.54.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.55.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.55.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.55.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.56.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.56.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.56.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.57.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.57.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.57.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.58.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.58.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.58.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.59.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.59.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.59.ffn_up_exps.weight buffer type overriden to CPU
Tensor blk.60.ffn_gate_exps.weight buffer type overriden to CPU
Tensor blk.60.ffn_down_exps.weight buffer type overriden to CPU
Tensor blk.60.ffn_up_exps.weight buffer type overriden to CPU
 llm_load_tensors: offloading 61 repeating layers to GPU
llm_load_tensors: offloading non-repeating layers to GPU
llm_load_tensors: offloaded 62/62 layers to GPU
llm_load_tensors:        CPU buffer size = 43751.77 MiB
llm_load_tensors:        CPU buffer size = 43751.77 MiB
llm_load_tensors:        CPU buffer size = 43751.77 MiB
llm_load_tensors:        CPU buffer size = 43751.77 MiB
llm_load_tensors:        CPU buffer size = 43751.77 MiB
llm_load_tensors:        CPU buffer size = 43751.77 MiB
llm_load_tensors:        CPU buffer size = 43751.77 MiB
llm_load_tensors:        CPU buffer size = 43751.77 MiB
llm_load_tensors:        CPU buffer size = 43751.77 MiB
llm_load_tensors:        CPU buffer size = 44679.30 MiB
llm_load_tensors:        CPU buffer size =   630.00 MiB
llm_load_tensors:      CUDA0 buffer size =  2506.50 MiB
llm_load_tensors:      CUDA1 buffer size =  8902.93 MiB
....................................................................................................
llama_new_context_with_model: n_ctx      = 131072
llama_new_context_with_model: n_batch    = 4096
llama_new_context_with_model: n_ubatch   = 4096
llama_new_context_with_model: flash_attn = 1
llama_new_context_with_model: mla_attn   = 3
llama_new_context_with_model: attn_max_b = 512
llama_new_context_with_model: fused_moe  = 0
llama_new_context_with_model: fused_up_gate = 1
llama_new_context_with_model: ser        = -1, 0
llama_new_context_with_model: freq_base  = 50000.0
llama_new_context_with_model: freq_scale = 0.03125
llama_kv_cache_init:      CUDA0 KV buffer size =   994.51 MiB
llama_kv_cache_init:      CUDA1 KV buffer size =  3672.02 MiB
llama_new_context_with_model: KV self size  = 4666.50 MiB, c^KV (q8_0): 4666.50 MiB, kv^T: not used
llama_new_context_with_model:  CUDA_Host  output buffer size =     0.62 MiB
llama_new_context_with_model: pipeline parallelism enabled (n_copies=1)
llama_new_context_with_model:      CUDA0 compute buffer size =  9032.02 MiB
llama_new_context_with_model:      CUDA1 compute buffer size =  7376.03 MiB
llama_new_context_with_model:  CUDA_Host compute buffer size =  2160.05 MiB
llama_new_context_with_model: graph nodes  = 13649
llama_new_context_with_model: graph splits = 231

main: n_kv_max = 131072, n_batch = 4096, n_ubatch = 4096, flash_attn = 1, n_gpu_layers = 99, n_threads = 64, n_threads_batch = 64

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
mmq_x_best=0
/opt/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml-cuda/template-instances/../mmq.cuh:4255: fatal error
./run-ik_llama.cpp.sh: line 54: 89211 Aborted                    CUDA_VISIBLE_DEVICES="0,1" /opt/ik_llama.cpp/ik_llama.cpp/build/bin/llama-sweep-bench --warmup-batch --model /opt/ubergarm/Kimi-K2-Instruct-GGUF/IQ3_KS/Kimi-K2-Instruct-IQ3_KS-00001-of-00010.gguf --alias ubergarm/Kimi-K2-Instruct-GGUF_IQ3_KS --ctx-size $((128 * 1024)) -b $((8 * 512)) -ub $((8 * 512)) --mlock --seed 3407 --temp 0.5 --top-k 0 --top-p 1.0 --min-p 0.1 --repeat-penalty 1.0 -ctk q8_0 -mla 3 -fa -amb 512 --main-gpu 1 --tensor-split 1,4 --split-mode layer --override-tensor exps=CPU --n-gpu-layers 99 --threads $(grep ^cpu\\scores /proc/cpuinfo | uniq | awk '{print $4}' | xargs -I{} echo "{}-0" | bc) --host 0.0.0.0 --port 8080 --prompt-cache "$HOME/.cache/ik_llama.cpp/prompt-cache.bin" --prompt-cache-all --slot-save-path "$HOME/.cache/ik_llama.cpp/slot.bin" --lookup-cache-dynamic "$HOME/.cache/ik_llama.cpp/slot.bin" --keep -1 --slot-prompt-similarity 0.35 --metrics

```

---

👤 **ikawrakow** commented on **2025-10-07** at **14:56:44**

Can you run with [#820](https://github.com/ikawrakow/ik_llama.cpp/issues/820) and post the log? Just the part after the table header is enough. Thanks.

---

👤 **magikRUKKOLA** commented on **2025-10-07** at **21:16:43**

@ikawrakow 

> Can you run with [[#820](https://github.com/ikawrakow/ik_llama.cpp/issues/820)](https://github.com/ikawrakow/ik_llama.cpp/pull/820) and post the log? Just the part after the table header is enough. Thanks.

```
type=q8_0, mmq_x_best=0
  id = 0, nsm = 1, cc = 860, smpbo = 1
  mmq_x_max = 128, mmq_y = 128, use_stream_k = 1
    mmq_x = 8, granularity = 8, shmem = 40960
    mmq_x = 16, granularity = 8, shmem = 41984
    mmq_x = 24, granularity = 8, shmem = 43008
    mmq_x = 32, granularity = 8, shmem = 44032
    mmq_x = 40, granularity = 8, shmem = 45056
    mmq_x = 48, granularity = 16, shmem = 46080
    mmq_x = 56, granularity = 16, shmem = 47104
    mmq_x = 64, granularity = 16, shmem = 48128
    mmq_x = 72, granularity = 16, shmem = 50176
    mmq_x = 80, granularity = 16, shmem = 51200
    mmq_x = 88, granularity = 16, shmem = 52224
    mmq_x = 96, granularity = 16, shmem = 53248
    mmq_x = 104, granularity = 16, shmem = 54272
    mmq_x = 112, granularity = 16, shmem = 55296
    mmq_x = 120, granularity = 16, shmem = 56320
    mmq_x = 128, granularity = 16, shmem = 57344
/opt/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml-cuda/template-instances/../mmq.cuh:4263: fatal error
./run-ik_llama.cpp.sh: line 54: 141700 Aborted                    CUDA_VISIBLE_DEVICES="0,1" /opt/ik_llama.cpp/ik_llama.cpp/build/bin/llama-sweep-bench --warmup-batch --model /opt/ubergarm/Kimi-K2-Instruct-GGUF/IQ3_KS/Kimi-K2-Instruct-IQ3_KS-00001-of-00010.gguf --alias ubergarm/Kimi-K2-Instruct-GGUF_IQ3_KS --ctx-size $((128 * 1024)) -b $((8 * 512)) -ub $((8 * 512)) --mlock --seed 3407 --temp 0.5 --top-k 0 --top-p 1.0 --min-p 0.1 --repeat-penalty 1.0 -ctk q8_0 -mla 3 -fa -amb 512 --main-gpu 1 --tensor-split 1,4 --split-mode layer --override-tensor exps=CPU --n-gpu-layers 99 --threads $(grep ^cpu\\scores /proc/cpuinfo | uniq | awk '{print $4}' | xargs -I{} echo "{}-0" | bc) --host 0.0.0.0 --port 8080 --prompt-cache "$HOME/.cache/ik_llama.cpp/prompt-cache.bin" --prompt-cache-all --slot-save-path "$HOME/.cache/ik_llama.cpp/slot.bin" --lookup-cache-dynamic "$HOME/.cache/ik_llama.cpp/slot.bin" --keep -1 --slot-prompt-similarity 0.35 --metrics

```

For Deepseek-R1 6.2478bpw quant it fails too:

```
main: n_kv_max = 114688, n_batch = 4096, n_ubatch = 4096, flash_attn = 1, n_gpu_layers = 99, n_threads = 64, n_threads_batch = 64

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
type=iq4_xs, mmq_x_best=0
  id = 0, nsm = 1, cc = 860, smpbo = 1
  mmq_x_max = 128, mmq_y = 128, use_stream_k = 1
    mmq_x = 8, granularity = 8, shmem = 40960
    mmq_x = 16, granularity = 8, shmem = 41984
    mmq_x = 24, granularity = 8, shmem = 43008
    mmq_x = 32, granularity = 8, shmem = 44032
    mmq_x = 40, granularity = 8, shmem = 45056
    mmq_x = 48, granularity = 16, shmem = 46080
    mmq_x = 56, granularity = 16, shmem = 47104
    mmq_x = 64, granularity = 16, shmem = 48128
    mmq_x = 72, granularity = 16, shmem = 50176
    mmq_x = 80, granularity = 16, shmem = 51200
    mmq_x = 88, granularity = 16, shmem = 52224
    mmq_x = 96, granularity = 16, shmem = 53248
    mmq_x = 104, granularity = 16, shmem = 54272
    mmq_x = 112, granularity = 16, shmem = 55296
    mmq_x = 120, granularity = 16, shmem = 56320
    mmq_x = 128, granularity = 16, shmem = 57344
/opt/ik_llama.cpp/ik_llama.cpp/ggml/src/ggml-cuda/template-instances/../mmq.cuh:4263: fatal error
```

---

👤 **magikRUKKOLA** commented on **2025-10-08** at **01:44:23**

Uh.  Oh.  I just noticed that I was using the code [at this particular machine].  I have no idea how that happened ( I forgot to do git pull before git fetch https://github.com/ikawrakow/ik_llama.cpp refs/pull/820/head ? ).  Sorry about that.