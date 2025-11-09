## 🔀 [Pull Request #762](https://github.com/ikawrakow/ik_llama.cpp/pull/762) - A slightly better version of offloading only activated experts

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/ooae2` |
| **Target Branch** | `main` |
| **Created** | 2025-09-05 |
| **Updated** | 2025-09-07 |
| **Merged** | 2025-09-05 |

---

## 📄 Description

See [#698](https://github.com/ikawrakow/ik_llama.cpp/issues/698) 

@Ph0rk0z

Does this fix the problem with your multi-GPU setup?
You need to add `--ooae` to your command line to activate the **O**float **O**nly **A**ctivated **E**experts feature.

---

## 💬 Conversation

👤 **Ph0rk0z** commented on **2025-09-05** at **13:13:42**

I will have to test. I've been dicking with distributed wan. Will test by EOD and see what happened post all of these commits. Currently I am built to: https://github.com/ikawrakow/ik_llama.cpp/commit/46968d4ab15c00397c236376864ab6203379beae so I will use it as a baseline.

---

👤 **Ph0rk0z** commented on **2025-09-05** at **18:53:30**

Well.. it's certainly cranking now.

```
        CUDA_VISIBLE_DEVICES=0,1,2,3 numactl --interleave=all ./bin/llama-sweep-bench \
    -m /Qwen3-235B-A22B-Instruct-2507-IQ4_KS/Qwen3-235B-A22B-Instruct-pure-IQ4_KS-00001-of-00003.gguf \
    -t 48 \
    -c 32768 \
    --numa distribute \
    -ngl 95 \
    -ctk q8_0 \
    -ctv q8_0 \
    -fa \
    -rtr \
    -fmoe \
    -ub 1024 \
    -ot "blk\.(0|1|2|3|4|5|6|7|8|9|10|11|12|13|14|15)\.ffn_.*.=CUDA0" \
    -ot "blk\.(16|17|18|19|20|21|22|23|24|25|26|27|28|29|30|31|32)\.ffn_.*.=CUDA1" \
    -ot "blk\.(33|34|35|36|37|38|39|40|41|42|43|44|45|46|47|48|49)\.ffn_.*.=CUDA2" \
    -ot "blk\.(50|51|52|53|54|55|56|57|58|59|60|61|62|63|64|65)\.ffn_.*.=CUDA3" \
    -ot "\.ffn_.*_exps.=CPU" \
    -ooae
```
    
    
Remove double definition of LLAMA_LOG_DEBUG

 
 |    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  1024 |    256 |      0 |    7.955 |   128.72 |   12.990 |    19.71 |
|  1024 |    256 |   1024 |    7.697 |   133.04 |   13.228 |    19.35 |
|  1024 |    256 |   2048 |    7.733 |   132.42 |   13.406 |    19.10 |
|  1024 |    256 |   3072 |    7.764 |   131.89 |   13.606 |    18.81 |
|  1024 |    256 |   4096 |    7.801 |   131.26 |   13.782 |    18.58 |
|  1024 |    256 |   5120 |    7.828 |   130.81 |   14.159 |    18.08 |
|  1024 |    256 |   6144 |    7.872 |   130.08 |   14.376 |    17.81 |
|  1024 |    256 |   7168 |    7.917 |   129.35 |   14.649 |    17.48 |
|  1024 |    256 |   8192 |    7.954 |   128.74 |   14.937 |    17.14 |
|  1024 |    256 |   9216 |    7.978 |   128.36 |   15.229 |    16.81 |
|  1024 |    256 |  10240 |    8.021 |   127.66 |   15.529 |    16.48 |
|  1024 |    256 |  11264 |    8.060 |   127.04 |   15.865 |    16.14 |
|  1024 |    256 |  12288 |    8.102 |   126.38 |   16.054 |    15.95 |
|  1024 |    256 |  13312 |    8.145 |   125.72 |   16.288 |    15.72 |
|  1024 |    256 |  14336 |    8.171 |   125.33 |   16.513 |    15.50 |
|  1024 |    256 |  15360 |    8.210 |   124.73 |   16.750 |    15.28 |
|  1024 |    256 |  16384 |    8.257 |   124.01 |   17.103 |    14.97 |
|  1024 |    256 |  17408 |    8.288 |   123.54 |   17.293 |    14.80 |
|  1024 |    256 |  18432 |    8.326 |   122.99 |   17.500 |    14.63 |
|  1024 |    256 |  19456 |    8.365 |   122.42 |   17.743 |    14.43 |
|  1024 |    256 |  20480 |    8.385 |   122.13 |   17.969 |    14.25 |
|  1024 |    256 |  21504 |    8.425 |   121.55 |   18.324 |    13.97 |
|  1024 |    256 |  22528 |    8.463 |   121.00 |   18.477 |    13.85 |
|  1024 |    256 |  23552 |    8.498 |   120.49 |   18.712 |    13.68 |
|  1024 |    256 |  24576 |    8.541 |   119.89 |   18.941 |    13.52 |
|  1024 |    256 |  25600 |    8.572 |   119.46 |   19.170 |    13.35 |
|  1024 |    256 |  26624 |    8.615 |   118.86 |   19.532 |    13.11 |
|  1024 |    256 |  27648 |    8.641 |   118.50 |   19.710 |    12.99 |
|  1024 |    256 |  28672 |    8.669 |   118.12 |   19.922 |    12.85 |
|  1024 |    256 |  29696 |    8.720 |   117.43 |   20.173 |    12.69 |
|  1024 |    256 |  30720 |    8.751 |   117.02 |   20.403 |    12.55 |
|  1024 |    256 |  31744 |    8.786 |   116.56 |   20.774 |    12.32 |


head - no extra params
 
|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  1024 |    256 |      0 |    7.847 |   130.49 |   12.898 |    19.85 |
|  1024 |    256 |   1024 |    7.945 |   128.88 |   13.224 |    19.36 |
|  1024 |    256 |   2048 |    7.994 |   128.10 |   13.293 |    19.26 |
|  1024 |    256 |   3072 |    7.723 |   132.59 |   13.578 |    18.85 |
|  1024 |    256 |   4096 |    7.821 |   130.92 |   13.769 |    18.59 |
|  1024 |    256 |   5120 |    7.864 |   130.21 |   14.269 |    17.94 |
|  1024 |    256 |   6144 |    7.895 |   129.70 |   14.299 |    17.90 |
|  1024 |    256 |   7168 |    7.856 |   130.35 |   14.524 |    17.63 |
|  1024 |    256 |   8192 |    7.890 |   129.78 |   14.848 |    17.24 |
|  1024 |    256 |   9216 |    7.894 |   129.72 |   15.084 |    16.97 |
|  1024 |    256 |  10240 |    7.951 |   128.79 |   15.318 |    16.71 |
|  1024 |    256 |  11264 |    7.987 |   128.20 |   15.643 |    16.36 |
|  1024 |    256 |  12288 |    8.024 |   127.62 |   15.814 |    16.19 |
|  1024 |    256 |  13312 |    8.056 |   127.11 |   16.136 |    15.87 |
|  1024 |    256 |  14336 |    8.089 |   126.58 |   16.303 |    15.70 |
|  1024 |    256 |  15360 |    8.115 |   126.18 |   16.488 |    15.53 |
|  1024 |    256 |  16384 |    8.152 |   125.62 |   16.828 |    15.21 |
 
 
PR -ooae

llama_new_context_with_model: graph nodes  = 3672
llama_new_context_with_model: graph splits = 162
XXXXXXXXXXXXXXXXXXXXX Setting only active experts offload

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  1024 |    256 |      0 |    5.744 |   178.26 |   12.973 |    19.73 |
|  1024 |    256 |   1024 |    5.308 |   192.90 |   13.214 |    19.37 |
|  1024 |    256 |   2048 |    5.405 |   189.46 |   13.570 |    18.87 |
|  1024 |    256 |   3072 |    5.356 |   191.17 |   13.691 |    18.70 |
|  1024 |    256 |   4096 |    5.544 |   184.71 |   13.837 |    18.50 |
|  1024 |    256 |   5120 |    5.536 |   184.98 |   14.333 |    17.86 |
|  1024 |    256 |   6144 |    5.583 |   183.42 |   14.369 |    17.82 |
|  1024 |    256 |   7168 |    5.754 |   177.96 |   14.572 |    17.57 |
|  1024 |    256 |   8192 |    5.703 |   179.54 |   14.945 |    17.13 |
|  1024 |    256 |   9216 |    5.721 |   178.98 |   15.170 |    16.88 |
|  1024 |    256 |  10240 |    5.872 |   174.40 |   15.408 |    16.62 |
|  1024 |    256 |  11264 |    5.883 |   174.05 |   15.724 |    16.28 |
|  1024 |    256 |  12288 |    5.972 |   171.47 |   15.893 |    16.11 |
|  1024 |    256 |  13312 |    5.989 |   170.99 |   16.230 |    15.77 |
|  1024 |    256 |  14336 |    6.006 |   170.51 |   16.386 |    15.62 |
|  1024 |    256 |  15360 |    6.050 |   169.26 |   16.564 |    15.46 |
|  1024 |    256 |  16384 |    6.195 |   165.29 |   16.922 |    15.13 |
|  1024 |    256 |  17408 |    6.180 |   165.70 |   17.106 |    14.97 |
|  1024 |    256 |  18432 |    6.272 |   163.28 |   17.507 |    14.62 |
|  1024 |    256 |  19456 |    6.256 |   163.67 |   17.558 |    14.58 |
|  1024 |    256 |  20480 |    6.296 |   162.65 |   17.755 |    14.42 |
|  1024 |    256 |  21504 |    6.317 |   162.10 |   18.080 |    14.16 |
|  1024 |    256 |  22528 |    6.330 |   161.78 |   18.268 |    14.01 |
|  1024 |    256 |  23552 |    6.365 |   160.89 |   18.554 |    13.80 |
|  1024 |    256 |  24576 |    6.511 |   157.27 |   18.733 |    13.67 |
|  1024 |    256 |  25600 |    6.393 |   160.18 |   18.967 |    13.50 |
|  1024 |    256 |  26624 |    6.531 |   156.78 |   19.342 |    13.24 |
|  1024 |    256 |  27648 |    6.520 |   157.05 |   19.452 |    13.16 |
|  1024 |    256 |  28672 |    6.584 |   155.53 |   19.725 |    12.98 |
|  1024 |    256 |  29696 |    6.684 |   153.19 |   19.928 |    12.85 |
 

At the end it got stuck. I'm not sure it's from the PR or if it's something that happened with the HW. The console showed no errors and I was able to exit.

<img width="863" height="863" alt="Image" src="https://github.com/user-attachments/assets/da32414b-f233-47cf-bc36-93cb5b6211b8" />

---

👤 **ikawrakow** commented on **2025-09-05** at **19:30:57**

Wow, that's a pretty significant difference. I gain just 5% for Qwen3-30B-A3B. I'm going to merge it. I ran many times while testing up to a context of 65k tokens for 3 different models and didn't observe the system hanging. But if it turns out to be real, it is behind a command line argument, so one does not have to use it.

---

👤 **Ph0rk0z** commented on **2025-09-05** at **19:53:46**

I only experienced a similar bug with NCCL tensor parallel. A GPU gets out of sync and then stuck at "100%". Will give it some regular use and see if it reoccurs.

---

👤 **NabuchodonosorII** commented on **2025-09-07** at **14:16:31**

Is the flag avaible for llama-server ? I've compile the last version and they are not provided, or i didn't understand  "it is behind a command line argument, so one does not have to use it."

---

👤 **ikawrakow** commented on **2025-09-07** at **14:25:11**

Yes it is available. Just add -ooae to your other server command line arguments.