## 🗣️ [Discussion #846](https://github.com/ikawrakow/ik_llama.cpp/discussions/846) - Slow compilation

| **Author** | `wallentri88` |
| :--- | :--- |
| **State** | ✅ **Answered** |
| **Created** | 2025-10-20 |
| **Updated** | 2025-10-20 |

---

## 📄 Description

Default compile options compile ALL quants:

```
[ 33%] Building CUDA object ggml/src/CMakeFiles/ggml.dir/ggml-cuda/template-instances/mmq-instance-iq2_k.cu.o
[ 35%] Building CUDA object ggml/src/CMakeFiles/ggml.dir/ggml-cuda/template-instances/mmq-instance-iq2_k_id.cu.o
[ 35%] Building CUDA object ggml/src/CMakeFiles/ggml.dir/ggml-cuda/template-instances/mmq-instance-iq2_k_r4.cu.o
[ 35%] Building CUDA object ggml/src/CMakeFiles/ggml.dir/ggml-cuda/template-instances/mmq-instance-iq2_kl.cu.o
[ 36%] Building CUDA object ggml/src/CMakeFiles/ggml.dir/ggml-cuda/template-instances/mmq-instance-iq2_kl_id.cu.o
[ 36%] Building CUDA object ggml/src/CMakeFiles/ggml.dir/ggml-cuda/template-instances/mmq-instance-iq2_ks.cu.o
[ 36%] Building CUDA object ggml/src/CMakeFiles/ggml.dir/ggml-cuda/template-instances/mmq-instance-iq2_ks_id.cu.o
[ 36%] Building CUDA object ggml/src/CMakeFiles/ggml.dir/ggml-cuda/template-instances/mmq-instance-iq2_kt.cu.o
[ 38%] Building CUDA object ggml/src/CMakeFiles/ggml.dir/ggml-cuda/template-instances/mmq-instance-iq2_kt_id.cu.o
[ 38%] Building CUDA object ggml/src/CMakeFiles/ggml.dir/ggml-cuda/template-instances/mmq-instance-iq2_s.cu.o
[ 38%] Building CUDA object ggml/src/CMakeFiles/ggml.dir/ggml-cuda/template-instances/mmq-instance-iq2_s_id.cu.o
[ 40%] Building CUDA object ggml/src/CMakeFiles/ggml.dir/ggml-cuda/template-instances/mmq-instance-iq2_xs.cu.o
[ 40%] Building CUDA object ggml/src/CMakeFiles/ggml.dir/ggml-cuda/template-instances/mmq-instance-iq2_xs_id.cu.o
[ 40%] Building CUDA object ggml/src/CMakeFiles/ggml.dir/ggml-cuda/template-instances/mmq-instance-iq2_xxs.cu.o
[ 40%] Building CUDA object ggml/src/CMakeFiles/ggml.dir/ggml-cuda/template-instances/mmq-instance-iq2_xxs_id.cu.o
[ 41%] Building CUDA object ggml/src/CMakeFiles/ggml.dir/ggml-cuda/template-instances/mmq-instance-iq3_k.cu.o
[ 41%] Building CUDA object ggml/src/CMakeFiles/ggml.dir/ggml-cuda/template-instances/mmq-instance-iq3_k_id.cu.o
[ 41%] Building CUDA object ggml/src/CMakeFiles/ggml.dir/ggml-cuda/template-instances/mmq-instance-iq3_k_r4.cu.o
[ 43%] Building CUDA object ggml/src/CMakeFiles/ggml.dir/ggml-cuda/template-instances/mmq-instance-iq3_ks.cu.o
[ 43%] Building CUDA object ggml/src/CMakeFiles/ggml.dir/ggml-cuda/template-instances/mmq-instance-iq3_ks_id.cu.o
[ 43%] Building CUDA object ggml/src/CMakeFiles/ggml.dir/ggml-cuda/template-instances/mmq-instance-iq3_kt.cu.o
[ 44%] Building CUDA object ggml/src/CMakeFiles/ggml.dir/ggml-cuda/template-instances/mmq-instance-iq3_kt_id.cu.o
[ 44%] Building CUDA object ggml/src/CMakeFiles/ggml.dir/ggml-cuda/template-instances/mmq-instance-iq3_s.cu.o
[ 44%] Building CUDA object ggml/src/CMakeFiles/ggml.dir/ggml-cuda/template-instances/mmq-instance-iq3_s_id.cu.o
[ 44%] Building CUDA object ggml/src/CMakeFiles/ggml.dir/ggml-cuda/template-instances/mmq-instance-iq3_xxs.cu.o
```

etc etc

Is there's a way to limit it to quants I actually use? I only use IQ4_K and IQ5_K ones. I feel unused quants are just wasting compilation time :(

---

## 💬 Discussion

👤 **ikawrakow** commented on **2025-10-20** at **15:15:05**

I know, it is painful when the CUDA mmq and/or mmvq kernels need to be rebuild. There is currently no way to select a subset of the quantization types. You can add a feature request if you like.