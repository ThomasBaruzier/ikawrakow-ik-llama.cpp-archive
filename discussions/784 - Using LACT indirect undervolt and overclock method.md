## 🗣️ [Discussion #784](https://github.com/ikawrakow/ik_llama.cpp/discussions/784) - Using LACT indirect undervolt and overclock method!

| **Author** | `ubergarm` |
| :--- | :--- |
| **State** | ✅ **Open** |
| **Created** | 2025-09-17 |
| **Updated** | 2025-09-22 |

---

## 📄 Description

Dropping a PSA here for folks who haven't tried this out yet, but with both NVIDIA and AMD GPUs you have access to some clocks/power state configurations in Linux with LACT.

There is a recent method specific to nvidia GPUs and I just did a benchmark comparing LACT overclock profile vs naieve `nvidia-smi -pl 400` power cap showing the LACT method to give better performance with lower energy usage.

Writeup and details here: 

https://forum.level1techs.com/t/some-gpu-5090-4090-3090-a600-idle-power-consumption-headless-on-linux-fedora-42-and-some-undervolt-overclock-info/237064/6

Cheers!

---

## 💬 Discussion

👤 **magikRUKKOLA** commented on **2025-09-21** at **22:01:12**

@ubergarm , Good info!  Thanks for pointing that out.  Am i correctly understanding that by setting the P-state offsets to some positive delta and the locking the clocks up (plus the power limit) you was able to force your NVIDIA GPUs to overclock better than the default boost algo? etc.

I have compiled the lact but lol its seem to use the DRM to check for the GPUs available (to overclock) which the p2p-enabled driver doesn't expose.  :) 

lactd:
```
WARN lact_daemon::server::handler: no GPUs were found
```

at the same time:
nvidia-smi
```
Sun Sep 21 21:39:07 2025       
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 580.82.09              Driver Version: 580.82.09      CUDA Version: 13.0     |
+-----------------------------------------+------------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
|                                         |                        |               MIG M. |
|=========================================+========================+======================|
|   0  NVIDIA GeForce RTX 3090        Off |   00000000:41:00.0 Off |                  N/A |
| 72%   79C    P2            172W /  400W |   23398MiB /  24576MiB |      0%      Default |
|                                         |                        |                  N/A |
+-----------------------------------------+------------------------+----------------------+
|   1  NVIDIA GeForce RTX 3090        Off |   00000000:42:00.0 Off |                  N/A |
| 30%   60C    P2            147W /  400W |   23366MiB /  24576MiB |      0%      Default |
|                                         |                        |                  N/A |
+-----------------------------------------+------------------------+----------------------+
|   2  NVIDIA GeForce RTX 3090        Off |   00000000:61:00.0 Off |                  N/A |
| 64%   76C    P2            171W /  400W |   23638MiB /  24576MiB |      0%      Default |
|                                         |                        |                  N/A |
+-----------------------------------------+------------------------+----------------------+

+-----------------------------------------------------------------------------------------+
| Processes:                                                                              |
|  GPU   GI   CI              PID   Type   Process name                        GPU Memory |
|        ID   ID                                                               Usage      |
|=========================================================================================|
|    0   N/A  N/A           30018      C   ...ma.cpp/build/bin/llama-server      23386MiB |
|    1   N/A  N/A           30018      C   ...ma.cpp/build/bin/llama-server      23354MiB |
|    2   N/A  N/A           30018      C   ...ma.cpp/build/bin/llama-server      23626MiB |
+-----------------------------------------------------------------------------------------+
```

The software (LACT) seems pretty bulky (its 400+ rust packages) etc.  May be I should rather write a C code that uses NVML directly?

> 👤 **ubergarm** replied on **2025-09-21** at **23:43:15**
> 
> Heya @magikRUKKOLA 
> 
> Yes, I have my GPU tuned up for both windows and Linux and believe it offers better performance and/or energy efficiency depending exactly how you tune. Seems like keeping my 3090TI FE in P2 (perforrmance state 2) [0 is most "performance" out of 16 possible states, of which only like 8 or so are used on this model with 8 being like idle maybe[.
> 
> For my specific silicon locking GPU clock at 1950MHz at 890mv in pstate 2 seems about the sweet spot given my cooling/fan noise/max power envelope. No need to set `nvidia-smi -pl 350` power cap (unless you need a guaranteed hard-cap due to PSU and multiGPUs etc).
> 
> Oh interesting your P2P drivers might not be quite compatible with `LACT`'s implementation of NVML or whatnot. Yeah you could write a python script with NVML bindings `nvidia-ml-py` or there is at least one c project I've seen doing similar.
> 
> I did some more benchmarks with ik_llama.cpp and for full GPU offload dense models it is quite nice, though I've heard for big MoEs it doesn't help much given CPU/RAM bottleneck. It also helps for ComfyUI stuff in my experience.
> 
> More benchmarks and links in this comment thread here: https://www.reddit.com/r/LocalLLaMA/comments/1nkycpq/comment/nf3c08a/
> 
> Sorry my stuff is all spread out, hard to do research in 2025 xD
> 
> Curious your experience with this technique once you get it going! (i also had to increase the GPU fans profile to keep it under 80 degC as it will temperature throttle at 83C... i have some more notes in the github thread for LACT too about some other infrequent almost random seeming throttling i see using this method which i don't understand yet).

> 👤 **magikRUKKOLA** replied on **2025-09-22** at **12:17:10**
> 
> @ubergarm 
> > though I've heard for big MoEs it doesn't help much given CPU/RAM bottleneck.
> 
> For my setup the GPU overclocking seem to boost the performance by ~1% in prefill and about ~ 2.5% in decode.
> 
> graphs:
> 
> <img width="1257" height="997" alt="nvidia-stats" src="https://github.com/user-attachments/assets/21ff88c2-a74b-4ce7-bdd5-6c4bae9ec514" />
> 
> 
> > Curious your experience with this technique once you get it going! (i also had to increase the GPU fans profile to keep it under 80 degC as it will temperature throttle at 83C... 
> 
> Yeah, I noticed the similar stuff -- it would not reach higher GPU core MHz if the temperature is over 80C.
> 
> frequency ranges during test:
> ```
> GPU: 2050-2100 MHz
> VRAM: 10246-10251 MHz
> ```
> 
> system report:
> ```
> System: Linux xxx 6.16.7+deb14-amd64 x86_64
> Driver Version: 580.82.09
> 
> GPU 0: NVIDIA GeForce RTX 3090
> ------------------------------------------------
>   PCI Bus ID:        00000000:41:00.0
>   VBIOS Version:     94.02.4B.00.0B
>   Persistence Mode:  Disabled
>   Core Temperature:  74°C
>   Power Usage:       213W
>   Current Power Limit: 400W
>   Power Limits:      Default: 350W, Min: 100W, Max: 400W
>   GPU Clock:         2070 MHz
>   VRAM Clock:        10251 MHz
>   GPU Utilization:   75%
>   VRAM Utilization:  16%
>   Memory Usage:      23.3 / 24.0 GB
>   Applied Offsets:   GPU: 150 MHz, VRAM: 1500 MHz
> 
> GPU 1: NVIDIA GeForce RTX 3090
> ------------------------------------------------
>   PCI Bus ID:        00000000:42:00.0
>   VBIOS Version:     94.02.4B.00.0B
>   Persistence Mode:  Disabled
>   Core Temperature:  60°C
>   Power Usage:       178W
>   Current Power Limit: 400W
>   Power Limits:      Default: 350W, Min: 100W, Max: 400W
>   GPU Clock:         2070 MHz
>   VRAM Clock:        10251 MHz
>   GPU Utilization:   28%
>   VRAM Utilization:  15%
>   Memory Usage:      23.1 / 24.0 GB
>   Applied Offsets:   GPU: 150 MHz, VRAM: 1500 MHz
> 
> GPU 2: NVIDIA GeForce RTX 3090
> ------------------------------------------------
>   PCI Bus ID:        00000000:61:00.0
>   VBIOS Version:     94.02.4B.00.0B
>   Persistence Mode:  Disabled
>   Core Temperature:  74°C
>   Power Usage:       168W
>   Current Power Limit: 400W
>   Power Limits:      Default: 350W, Min: 100W, Max: 400W
>   GPU Clock:         2070 MHz
>   VRAM Clock:        10251 MHz
>   GPU Utilization:   0%
>   VRAM Utilization:  0%
>   Memory Usage:      23.2 / 24.0 GB
>   Applied Offsets:   GPU: 150 MHz, VRAM: 1500 MHz
> 
> 
> Peer-to-Peer (P2P) Support Matrix:
> =================================
> GPU 0 -> GPU 1: Supported
> GPU 0 -> GPU 2: Supported
> GPU 1 -> GPU 0: Supported
> GPU 1 -> GPU 2: Supported
> GPU 2 -> GPU 0: Supported
> GPU 2 -> GPU 1: Supported
> ```
> 
> Deepseek-R1 (160k ctx) no GPU overclock:
> ```
> |    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
> |-------|--------|--------|----------|----------|----------|----------|
> |  4096 |   1024 |      0 |   34.567 |   118.50 |  156.103 |     6.56 |
> |  4096 |   1024 |   4096 |   35.056 |   116.84 |  159.154 |     6.43 |
> ```
> 
> Deepseek-R1 (160k ctx) with GPU overclock:
> 
> ```
> main: n_kv_max = 163840, n_batch = 8192, n_ubatch = 4096, flash_attn = 1, n_gpu_layers = 99, n_threads = 64, n_threads_batch = 64
> 
> |    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
> |-------|--------|--------|----------|----------|----------|----------|
> |  4096 |   1024 |      0 |   34.215 |   119.72 |  152.193 |     6.73 |
> |  4096 |   1024 |   4096 |   34.694 |   118.06 |  154.959 |     6.61 |
> ```

> 👤 **magikRUKKOLA** replied on **2025-09-22** at **12:38:38**
> 
> @ubergarm 
> 
> Basically I made a tool that is checking [if] the GPU utilization is above a certain threshold it just overclocks it (P-state offset, clocks locking and [a] power limit).  If for a number of consequent checks the daemon detects if the system is idle, then it just picks up the lowest frequences supported and sets it.  I also added the temperature check -- if the temperature is above a certain threshold (80C default) it turns on the fans of all GPUs to the max. (mine are stacked together one on top of another so that is beneficial).
> 
> Works great!  Thanks for the info!
> 
> code snippet:
> 
> ```c
> void apply_settings(nvmlDevice_t device, int index) {                                                                                                                                                                                                                                                                 
>     nvmlReturn_t result;                                                                                                                                                                                                                                                                                              
>                                                                                                                                                                                                                                                                                                                       
>     if (config.reset) {                                                                                                                                                                                                                                                                                               
>         if (config.graphics) {                                                                                                                                                                                                                                                                                        
>             char buf[MESSAGE_LEN];                                                                                                                                                                                                                                                                                    
>             snprintf(buf, sizeof(buf), "Resetting settings for GPU %d", index);                                                                                                                                                                                                                                       
>             enqueue_message(buf);                                                                                                                                                                                                                                                                                     
>         } else {                                                                                                                                                                                                                                                                                                      
>             printf("Resetting settings for GPU %d\n", index);                                                                                                                                                                                                                                                         
>         }                                                                                                                                                                                                                                                                                                             
>         reset_settings(device);                                                                                                                                                                                                                                                                                       
>         return;                                                                                                                                                                                                                                                                                                       
>     }                                                                                                                                                                                                                                                                                                                 
>                                                                                                                                                                                                                                                                                                                       
>     // Set power limit                                                                                                                                                                                                                                                                                                
>     if (config.power_limit > 0) {                                                                                                                                                                                                                                                                                     
>         unsigned int current_power;                                                                                                                                                                                                                                                                                   
>         result = nvmlDeviceGetPowerManagementLimit(device, &current_power);                                                                                                                                                                                                                                           
>         if (result == NVML_SUCCESS) {                                                                                                                                                                                                                                                                                 
>             unsigned int new_power = config.power_limit * 1000; // Convert to mW                                                                                                                                                                                                                                      
>             if (current_power != new_power) {                                                                                                                                                                                                                                                                         
>                 result = nvmlDeviceSetPowerManagementLimit(device, new_power);                                                                                                                                                                                                                                        
>                 if (result == NVML_SUCCESS) {                                                                                                                                                                                                                                                                         
>                     if (config.graphics) {                                                                                                                                                                                                                                                                            
>                         char buf[MESSAGE_LEN];                                                                                                                                                                                                                                                                        
>                         snprintf(buf, sizeof(buf), "GPU %d: Power set to %dW", index, config.power_limit);                                                                                                                                                                                                            
>                         enqueue_message(buf);                                                                                                                                                                                                                                                                         
>                     } else {                                                                                                                                                                                                                                                                                          
>                         printf("GPU %d: Power limit set to %dW\n", index, config.power_limit);                                                                                                                                                                                                                        
>                     }                                                                                                                                                                                                                                                                                                 
>                 } else if (result != NVML_ERROR_NOT_SUPPORTED) {                                                                                                                                                                                                                                                      
>                     fprintf(stderr, "GPU %d: Failed to set power limit: %s\n",                                                                                                                                                                                                                                        
>                             index, nvmlErrorString(result));                                                                                                                                                                                                                                                          
>                 }                                                                                                                                                                                                                                                                                                     
>             }                                                                                                                                                                                                                                                                                                         
>         }                                                                                                                                                                                                                                                                                                             
>     }                                                                                                                                                                                                                                                                                                                 
>                                                                                                                                                                                                                                                                                                                       
>     // Set clock offsets                                                                                                                                                                                                                                                                                              
>     if (config.gpu_offset != 0) {                                                                                                                                                                                                                                                                                     
>         result = nvmlDeviceSetGpcClkVfOffset(device, config.gpu_offset);                                                                                                                                                                                                                                              
>         if (result == NVML_SUCCESS) {                                                                                                                                                                                                                                                                                 
>             if (config.graphics) {                                                                                                                                                                                                                                                                                    
>                 char buf[MESSAGE_LEN];                                                                                                                                                                                                                                                                                
>                 snprintf(buf, sizeof(buf), "GPU %d: GPU offset %dMHz", index, config.gpu_offset);                                                                                                                                                                                                                     
>                 enqueue_message(buf);                                                                                                                                                                                                                                                                                 
>             } else {                                                                                                                                                                                                                                                                                                  
>                 printf("GPU %d: GPU offset set to %dMHz\n", index, config.gpu_offset);                                                                                                                                                                                                                                
>             }                                                                                                                                                                                                                                                                                                         
>         } else if (result != NVML_ERROR_NOT_SUPPORTED) {                                                                                                                                                                                                                                                              
>             fprintf(stderr, "GPU %d: Failed to set GPU offset: %s\n",                                                                                                                                                                                                                                                 
>                     index, nvmlErrorString(result));                                                                                                                                                                                                                                                                  
>         }                                                                                                                                                                                                                                                                                                             
>     }                                                                                                                                                                                                                                                                                                                 
>                                                                                                                                                                                                                                                                                                                       
>     if (config.vram_offset != 0) {                                                                                                                                                                                                                                                                                    
>         result = nvmlDeviceSetMemClkVfOffset(device, config.vram_offset);                                                                                                                                                                                                                                             
>         if (result == NVML_SUCCESS) {                                                                                                                                                                                                                                                                                 
>             if (config.graphics) {                                                                                                                                                                                                                                                                                    
>                 char buf[MESSAGE_LEN];                                                                                                                                                                                                                                                                                
>                 snprintf(buf, sizeof(buf), "GPU %d: VRAM offset %dMHz", index, config.vram_offset);                                                                                                                                                                                                                   
>                 enqueue_message(buf);                                                                                                                                                                                                                                                                                 
>             } else {                                                                                                                                                                                                                                                                                                  
>                 printf("GPU %d: VRAM offset set to %dMHz\n", index, config.vram_offset);                                                                                                                                                                                                                              
>             }                                                                                                                                                                                                                                                                                                         
>         } else if (result != NVML_ERROR_NOT_SUPPORTED) {                                                                                                                                                                                                                                                              
>             fprintf(stderr, "GPU %d: Failed to set VRAM offset: %s\n",                                                                                                                                                                                                                                                
>                     index, nvmlErrorString(result));                                                                                                                                                                                                                                                                  
>         }                                                                                                                                                                                                                                                                                                             
>     }                                                                                                                                                                                                                                                                                                                 
>                                                                                                                                                                                                                                                                                                                       
>     // Set locked clocks                                                                                                                                                                                                                                                                                              
>     if (config.gpu_lock) {                                                                                                                                                                                                                                                                                            
>         unsigned int min_clock, max_clock;                                                                                                                                                                                                                                                                            
>         result = nvmlDeviceGetMinMaxClockOfPState(device, NVML_CLOCK_GRAPHICS,                                                                                                                                                                                                                                        
>                                                  NVML_PSTATE_0, &min_clock, &max_clock);                                                                                                                                                                                                                              
>         if (result == NVML_SUCCESS) {                                                                                                                                                                                                                                                                                 
>             unsigned int target = max_clock + config.gpu_offset;                                                                                                                                                                                                                                                      
>             result = nvmlDeviceSetGpuLockedClocks(device, target, target);                                                                                                                                                                                                                                            
>             if (result == NVML_SUCCESS) {                                                                                                                                                                                                                                                                             
>                 printf("GPU %d: GPU locked at %dMHz\n", index, target);                                                                                                                                                                                                                                               
>                 overclock_applied = 1;                                                                                                                                                                                                                                                                                
>             } else if (result != NVML_ERROR_NOT_SUPPORTED) {                                                                                                                                                                                                                                                          
>                 fprintf(stderr, "GPU %d: Failed to lock GPU clock: %s\n",                                                                                                                                                                                                                                             
>                         index, nvmlErrorString(result));                                                                                                                                                                                                                                                              
>             }                                                                                                                                                                                                                                                                                                         
>         } else if (result != NVML_ERROR_NOT_SUPPORTED) {                                                                                                                                                                                                                                                              
>             fprintf(stderr, "GPU %d: Failed to get base GPU clock: %s\n",                                                                                                                                                                                                                                             
>                     index, nvmlErrorString(result));                                                                                                                                                                                                                                                                  
>         }                                                                                                                                                                                                                                                                                                             
>     }                                                                                                                                                                                                                                                                                                                 
>                                                                                                                                                                                                                                                                                                                       
>     if (config.vram_lock) {                                                                                                                                                                                                                                                                                           
>         unsigned int min_clock, max_clock;                                                                                                                                                                                                                                                                            
>         result = nvmlDeviceGetMinMaxClockOfPState(device, NVML_CLOCK_MEM,                                                                                                                                                                                                                                             
>                                                  NVML_PSTATE_0, &min_clock, &max_clock);                                                                                                                                                                                                                              
>         if (result == NVML_SUCCESS) {                                                                                                                                                                                                                                                                                 
>             unsigned int target = max_clock + config.vram_offset;                                                                                                                                                                                                                                                     
>             result = nvmlDeviceSetMemoryLockedClocks(device, target, target);                                                                                                                                                                                                                                         
>             if (result == NVML_SUCCESS) {                                                                                                                                                                                                                                                                             
>                 printf("GPU %d: VRAM locked at %dMHz\n", index, target);                                                                                                                                                                                                                                              
>                 overclock_applied = 1;                                                                                                                                                                                                                                                                                
>             } else if (result != NVML_ERROR_NOT_SUPPORTED) {                                                                                                                                                                                                                                                          
>                 fprintf(stderr, "GPU %d: Failed to lock VRAM clock: %s\n",                                                                                                                                                                                                                                            
>                         index, nvmlErrorString(result));                                                                                                                                                                                                                                                              
>             }                                                                                                                                                                                                                                                                                                         
>         } else if (result != NVML_ERROR_NOT_SUPPORTED) {                                                                                                                                                                                                                                                              
>             fprintf(stderr, "GPU %d: Failed to get base VRAM clock: %s\n",                                                                                                                                                                                                                                            
>                     index, nvmlErrorString(result));                                                                                                                                                                                                                                                                  
>         }                                                                                                                                                                                                                                                                                                             
>     }                                                                                                                                                                                                                                                                                                                 
>                                                                                                                                                                                                                                                                                                                       
>     config.apply_count++;                                                                                                                                                                                                                                                                                             
> }
> ```
> 
> The tool itself is quite long (1400 lines).  If anyone wants the full code LMK.