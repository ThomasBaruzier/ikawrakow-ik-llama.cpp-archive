## 📌 [Issue #627](https://github.com/ikawrakow/ik_llama.cpp/issues/627) - Feature Request: Tensor Parallelism

| **Author** | `rankaiyx` |
| :--- | :--- |
| **State** | ✅ **Open** |
| **Created** | 2025-07-18 |
| **Updated** | 2025-09-14 |
| **Labels** | `enhancement` |

---

## 📄 Description

### Prerequisites

- [x] I am running the latest code. Mention the version if possible as well.
- [x] I carefully followed the [README.md](https://github.com/ggerganov/llama.cpp/blob/master/README.md).
- [x] I searched using keywords relevant to my issue to make sure that I am creating a new issue that is not already open (or closed).
- [x] I reviewed the [Discussions](https://github.com/ggerganov/llama.cpp/discussions), and have a new and useful enhancement to share.

### Feature Description

Tensor Parallelism is a model-parallelism technique used in Large Language Model (LLM) inference to distribute the model's tensor computations (e.g., matrix multiplications) across multiple devices (like GPUs or TPUs). This allows different parts of the model's layers to be processed in parallel, improving inference speed and scalability.

**Key Features:**

- **Model Splitting:** Splits model layers (especially large weight matrices) across multiple devices.
- **Distributed Computation:** Performs tensor operations in parallel, reducing computation time.
- **Communication Overhead:** Requires inter-device communication (e.g., using AllReduce) to synchronize results.
- **Efficient Scaling:** Enables inference on larger models that don't fit on a single device.

**Use Case:** Ideal for large-scale LLM inference where model size exceeds a single GPU's memory capacity.

### Motivation

The performance of current methods(--split-mode  row) is much worse than vllm or mlc-llm.

On the 4xP100 platform, using the vLLM or mlc-llm for inference with the Qwen2.5-72B-4bit model achieves a generation speed of approximately 20 tok/s. In contrast, when using the llama.cpp with "--split-mode  row", the generation speed only reaches 10 tok/s, which is merely 50% of the former speed.

mlc-llm development is less active and supports fewer models.
In the upcoming 1.0 version, vllm will abandon a large number of Turing and older hardware.

### Possible Implementation

_No response_

---

## 💬 Conversation

👤 **ikawrakow** commented on **2025-07-18** at **08:05:35**

Have you tried raising the issue with the `llama.cpp` project?

Support for old hardware is not one of the strengths of this project, while exactly this is one of the strengths of mainline `llama.cpp`.

---

👤 **rankaiyx** commented on **2025-07-18** at **08:21:28**

There is an issue.
But it's expired.
Maybe the mainline llama.cpp focuses on versatility rather than SOTA.
https://github.com/ggml-org/llama.cpp/issues/9086

---

👤 **Ph0rk0z** commented on **2025-07-19** at **10:12:01**

Originally Cuda Dev was supposed to work on backend agnostic TP. Someone else volunteered and made partial PRs but appears to have abandoned them. Progress is stalled.

My split mode row gives higher T/G but lower PP as of this month in mainline. Since last year, some progress has been made. I tested with command-A. Wanted to compare with IK but then realized command-A isn't supported.

What's interesting is fastllm, who claims to fully utilize numa and supports hybrid inference. I aim to try out qwen-235b and compare speeds at some point. Can use both at 4 bit.

---

👤 **saood06** commented on **2025-07-19** at **10:19:14**

>Wanted to compare with IK but then realized command-A isn't supported.

I thought it was from [#341](https://github.com/ikawrakow/ik_llama.cpp/issues/341)

---

👤 **Ph0rk0z** commented on **2025-07-19** at **15:01:49**

Damn.. I missed that. Will give it a go.

_Well.. this is dildos.._

IK: Same prompt processing speed as mainline. In SM row, 17t/s generation. Loads GPU 0 like mainline used to. Unfortunately, command-A outputs what looks like parts of the training data or random text. Without SM it is coherent but only does ~12T/s

Mainline: I unfortunately pulled today. My speed in parallel is only 12t/s. Without it, drops down to 9. Prompt processing for both backends is about half speed when SM is row.

---

👤 **saood06** commented on **2025-07-20** at **01:09:42**

> IK: Same prompt processing speed as mainline. In SM row, 17t/s generation. Loads GPU 0 like mainline used to. Unfortunately, command-A outputs what looks like parts of the training data or random text. Without SM it is coherent but only does ~12T/s
> 
> Mainline: I unfortunately pulled today. My speed in parallel is only 12t/s. Without it, drops down to 9. Prompt processing for both backends is about half speed when SM is row.

So it looks like this repo gives you the fastest usable generation, I suggest you file an issue for the coherency issues with row enabled (and maybe also for the PP speed dropping by half).

---

👤 **aikitoria** commented on **2025-07-31** at **12:29:04**

This was also recently implemented by sglang: https://lmsys.org/blog/2025-07-14-intel-xeon-optimization/#multi-numa-parallelism

By splitting the weights between NUMA nodes, and then doing tensor parallel between those nodes, bandwidth utilization of CPU inference can be significantly improved on multi socket systems

---

👤 **Ph0rk0z** commented on **2025-07-31** at **15:11:49**

It's also in fastllm https://github.com/ztxz16/fastllm

I have 200gb/s wonder what proper numa would accomplish. To try that one, you have to download their weights.

---

👤 **aikitoria** commented on **2025-07-31** at **15:40:06**

> wonder what proper numa would accomplish

Let's use dual socket Eypc Turin as an example. When all 24 channels are filled, it will have a total memory bandwidth of roughly 1200GB/s, split into 600GB/s per socket.

When not using NUMA, we can assume each CPU will need roughly half of its bandwidth from memory modules installed on the other socket, so 300GB/s.

... but there are only 3 xGMI links between sockets, with combined unidirectional bandwidth of ~160GB/s in practice.

---

👤 **Ph0rk0z** commented on **2025-08-13** at **12:35:39**

So food for thought. I have gotten fastllm going. Just not yet fully using my GPUs with qwen235b on TG. I did memory benchmark using the tools in https://github.com/ggml-org/llama.cpp/pull/14969.

IK llama without numa distribute gets 40gb/s or so, with numa 55gb/s. But simply using GPU only for CTX, fastllm pulls 223GB/s to 150GB/s in the same pcm-memory utility.

My textgen output is over 7t/s for qwen, where in IK, the max for this config would be 3.x, not offloading any tensors to GPU.

Even for DDR4 systems, we're leaving possibly 1/2 the performance on the table. Feels pretty big.

---

👤 **guqiong96** commented on **2025-08-20** at **19:58:14**

like this :  https://github.com/guqiong96/lktransformers

---

👤 **rankaiyx** commented on **2025-09-14** at **04:41:34**

Defending the multi-GPU rig:
Now that Tesla P100 cards are priced at just $70 each, each offering 16GB of VRAM and 732 GB/s memory bandwidth, deploying four of them builds a system with 64GB of total VRAM and a staggering 2.8 TB/s memory bandwidth. 

This setup is exceptionally well-suited for Qwen3-Next-80B-A3B. Given that its activation size is comparable to Qwen3-30B-A3B, we can estimate a tg speed of ~30 tokens per second (t/s) without tensor parallelism—possibly even exceeding that. 

The reason? Qwen3-Next features numerous layers with linear attention, which significantly reduce memory and compute bottlenecks.

With tensor parallelism enabled, we should easily achieve at least 60 t/s. I suspect the tensor parallelism implementation of MLC on Qwen3-30B-A3B is flawed—performance with four GPUs is nearly identical to that with just two, while the ideal scaling would be 30 × 4 = 120 t/s.

My rig configuration:

CPU: E5-2680v4 (14 cores / 28 threads), used, $7
— 40 PCIe 3.0 lanes
— 4 memory channels

Motherboard: GA-X99P-SLI, used, $70 ( It also includes 4 x DDR4 8G memory sticks )
— Excellent support for 4 GPUs in peer-to-peer (P2P) mode
— Each GPU connected via PCIe 3.0 ×8
— Total of 32 PCIe lanes consumed for GPU communication
— 4 additional lanes for NVMe devices
— 4 remaining lanes for other high-speed peripherals

GPUs: 4 × Tesla P100, used, $70 each
— Total GPU cost: $280
— Total system cost (including CPU and motherboard): $427

Total VRAM: 64 GB
Total memory bandwidth: 2.8 TB/s
Ideal theoretical tg speed: 120 t/s
Realistic observed tg speed: ~60t/s
Even without tensor parallelism: ~30 t/s
— All this for under $500, with high-end server-class hardware and excellent PCIe bandwidth utilization.

This isn’t just a good setup—it’s a scientifically optimized system.
Please.