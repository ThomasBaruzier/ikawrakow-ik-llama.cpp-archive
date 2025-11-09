## 🗣️ [Discussion #818](https://github.com/ikawrakow/ik_llama.cpp/discussions/818) - TG speed prediction via RAM BW test

| **Author** | `magikRUKKOLA` |
| :--- | :--- |
| **State** | ✅ **Open** |
| **Created** | 2025-10-05 |
| **Updated** | 2025-10-05 |

---

## 📄 Description

It seems that token generation speed can be predicted via the RAM read speed.  Specifically, for example I am having two machines one with AMD Threadripper PRO 3995wx (64C) and one with AMD Threadripper 3445wx (12C) with DDR4 3200 MT/s 8 channel.

I can measure the RAM BW via the stream tool:

```bash
sudo apt install libopenmpi-dev
wget https://www.cs.virginia.edu/stream/FTP/Code/stream.c
gcc -O3 -fopenmp -DSTREAM_ARRAY_SIZE=100000000 -DNTIMES=100 stream.c -o stream
./stream
```

So, for the 64C machine its:

```
Triad:         115646.5     0.021680     0.020753     0.028800
```

And for the 12C machine its:

```
Triad:          70829.8     0.036779     0.033884     0.044056
```

Now, lets get the sweep-bench result for the 64C machine (GLM-4.6 IQ5_K):

```
main: n_kv_max = 81920, n_batch = 8192, n_ubatch = 4096, flash_attn = 1, n_gpu_layers = 99, n_threads = 64, n_threads_batch = 64

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  4096 |   1024 |      0 |   19.103 |   214.42 |  123.031 |     8.32 |
```

Now if I want to predict the TG for 12C machine I would just:

```
echo "scale=8;8.32*70829.8/115646.5"|bc
5.09573515
```

So the predicted speed is about 5.1 tps.  Which is very close to the actual result:

```
main: n_kv_max = 81920, n_batch = 8192, n_ubatch = 4096, flash_attn = 1, n_gpu_layers = 99, n_threads = 12, n_threads_batch = 12

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  4096 |   1024 |      0 |   18.564 |   220.64 |  197.036 |     5.20 |
```

Can anyone post their ./stream and llama-sweep-bench results in order to compare how it relates to TG speed?