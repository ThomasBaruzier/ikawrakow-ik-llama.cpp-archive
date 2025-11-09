## 🗣️ [Discussion #811](https://github.com/ikawrakow/ik_llama.cpp/discussions/811) - cuda build error

| **Author** | `calvin2021y` |
| :--- | :--- |
| **State** | ✅ **Open** |
| **Created** | 2025-10-01 |
| **Updated** | 2025-10-03 |

---

## 📄 Description

build with:

```
cmake ..
    -DCMAKE_CUDA_ARCHITECTURES=75
    -DGGML_CUDA=ON
    -DGGML_CUDA_F16=1
    -DGGML_CUDA_FA_ALL_QUANTS=1
```

```sh
/usr/bin/nvcc -forward-unknown-to-host-compiler -DGGML_CUDA_DMMV_X=32 -DGGML_CUDA_F16 -DGGML_CUDA_FA_ALL_QUANTS -DGGML_CUDA_MIN_BATCH_OFFLOAD=32 -DGGML_CUDA_MMV_Y=1 -DGGML_CUDA_PEER_MAX_BATCH_SIZE=128 -DGGML_CUDA_USE_GRAPHS -DGGML_IQK_FA_ALL_QUANTS -DGGML_IQK_FLASH_ATTENTION -DGGML_SCHED_MAX_COPIES=1 -DGGML_USE_CUDA -DGGML_USE_IQK_MULMAT -DGGML_USE_OPENMP -DK_QUANTS_PER_ITERATION=2 -DNDEBUG -D_GNU_SOURCE -D_XOPEN_SOURCE=600 -Iik_llama.cpp/ggml/src/../include -Iik_llama.cpp/ggml/src/. -O3 -DNDEBUG -std=c++17 "--generate-code=arch=compute_75,code=[compute_75,sm_75]" -Xcompiler=-fPIC -use_fast_math -extended-lambda -Xcompiler "-Wmissing-declarations -Wmissing-noreturn -Wall -Wextra -Wpedantic -Wcast-qual -Wno-unused-function -Wno-array-bounds -Wno-format-truncation -Wextra-semi -Wno-pedantic -march=native -mf16c -mfma -mavx -mavx2" -MD -MT ggml/src/CMakeFiles/ggml.dir/ggml-cuda/cpy.cu.o -MF ggml/src/CMakeFiles/ggml.dir/ggml-cuda/cpy.cu.o.d -x cu -c ik_llama.cpp/ggml/src/ggml-cuda/cpy.cu -o ggml/src/CMakeFiles/ggml.dir/ggml-cuda/cpy.cu.o


ik_llama.cpp/ggml/src/ggml-cuda/cpy.cu(138): error: class "ggml_cuda_graph" has no member "dest_ptrs_size"

ik_llama.cpp/ggml/src/ggml-cuda/cpy.cu(140): error: class "ggml_cuda_graph" has no member "dest_ptrs_d"

ik_llama.cpp/ggml/src/ggml-cuda/cpy.cu(141): error: class "ggml_cuda_graph" has no member "dest_ptrs_d"

ik_llama.cpp/ggml/src/ggml-cuda/cpy.cu(143): error: class "ggml_cuda_graph" has no member "dest_ptrs_d"

ik_llama.cpp/ggml/src/ggml-cuda/cpy.cu(144): error: class "ggml_cuda_graph" has no member "dest_ptrs_size"

ik_llama.cpp/ggml/src/ggml-cuda/cpy.cu(147): error: class "ggml_cuda_graph" has no member "dest_ptrs_d"

ik_llama.cpp/ggml/src/ggml-cuda/cpy.cu(148): error: class "ggml_cuda_graph" has no member "graph_cpynode_index"

ik_llama.cpp/ggml/src/ggml-cuda/cpy.cu(410): error: class "ggml_cuda_graph" has no member "use_cpy_indirection"

ik_llama.cpp/ggml/src/ggml-cuda/cpy.cu(411): error: class "ggml_cuda_graph" has no member "dest_ptrs_d"

ik_llama.cpp/ggml/src/ggml-cuda/cpy.cu(412): error: class "ggml_cuda_graph" has no member "graph_cpynode_index"

ik_llama.cpp/ggml/src/ggml-cuda/cpy.cu(484): error: class "ggml_cuda_graph" has no member "use_cpy_indirection"

ik_llama.cpp/ggml/src/ggml-cuda/cpy.cu(485): error: class "ggml_cuda_graph" has no member "graph_cpynode_index"


```

https://github.com/ggml-org/llama.cpp work

---

## 💬 Discussion

👤 **magikRUKKOLA** commented on **2025-10-02** at **01:56:44**

Plz check the version of your libcudart.so.  Is it outdated or so?

> 👤 **the-butterfly** replied on **2025-10-03** at **12:22:16**
> 
> SAME! get this with V100 / cuda 11.7

---

👤 **the-butterfly** commented on **2025-10-03** at **12:40:54**

I've just try t0002 tag version, and compile passed.

> 👤 **magikRUKKOLA** replied on **2025-10-03** at **23:34:06**
> 
> > I've just try t0002 tag version, and compile passed.
> 
> No, the other way around.  You do not downgrade the target software to fit your outdated libraries.  You just upgrading everything.