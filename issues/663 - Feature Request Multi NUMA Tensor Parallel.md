## 📌 [Issue #663](https://github.com/ikawrakow/ik_llama.cpp/issues/663) - Feature Request: Multi NUMA Tensor Parallel

| **Author** | `aikitoria` |
| :--- | :--- |
| **State** | ✅ **Open** |
| **Created** | 2025-07-31 |
| **Updated** | 2025-08-17 |
| **Labels** | `enhancement` |

---

## 📄 Description

### Prerequisites

- [x] I am running the latest code. Mention the version if possible as well.
- [x] I carefully followed the [README.md](https://github.com/ggerganov/llama.cpp/blob/master/README.md).
- [x] I searched using keywords relevant to my issue to make sure that I am creating a new issue that is not already open (or closed).
- [x] I reviewed the [Discussions](https://github.com/ggerganov/llama.cpp/discussions), and have a new and useful enhancement to share.

### Feature Description

By splitting the weights between NUMA nodes, and then doing tensor parallel between those nodes, bandwidth utilization can be significantly improved on multi socket systems, circumventing the problem of overloading the link between them when weights are shared globally. This was also recently implemented by sglang: https://lmsys.org/blog/2025-07-14-intel-xeon-optimization/#multi-numa-parallelism

### Motivation

Likely faster

### Possible Implementation

_No response_

---

## 💬 Conversation

👤 **aikitoria** commented on **2025-07-31** at **12:27:58**

I have failed to search apparently. Dupe of https://github.com/ikawrakow/ik_llama.cpp/issues/627

---

👤 **ikawrakow** commented on **2025-07-31** at **12:46:45**

I think you can leave this open. [#627](https://github.com/ikawrakow/ik_llama.cpp/issues/627) is about tensor parallel on a GPU. Although one may try to solve GPU and NUMA in the same way, it is not unlikely that the approach will be different.

---

👤 **aikitoria** commented on **2025-07-31** at **12:47:48**

I see, will reopen then.

---

👤 **saood06** commented on **2025-08-13** at **15:57:39**

>Although one may try to solve GPU and NUMA in the same way, it is not unlikely that the approach will be different.

The approach described in https://github.com/ggml-org/llama.cpp/pull/13776 seems like an approach that would solve both and you can add launching with NUMA backends instead of a global CPU one, or with sacrifice performance and just use RPC and 4 instances bound to each NUMA node, I don't know how complex doing this would be, but it feels like it would be quite complex.

Edit: [This](https://github.com/koush/llama.cpp/tree/parallel) also exists, which is a continuation of https://github.com/ggml-org/llama.cpp/pull/13818#issuecomment-2927411762 which as stated "Since the new GPU implementation is a wrapping backend, it should also work with heterogenous devices (if their respective backends implement the new backend requirements" which means it also solve both.

---

👤 **rankaiyx** commented on **2025-08-13** at **17:20:03**

https://github.com/ggml-org/llama.cpp/pull/9648
Here is a draft for comparison.

---

👤 **saood06** commented on **2025-08-13** at **17:23:24**

> [ggml-org/llama.cpp[#9648](https://github.com/ikawrakow/ik_llama.cpp/issues/9648)](https://github.com/ggml-org/llama.cpp/pull/9648) Here is a draft for comparison.

Thanks, I forgot to mention that one.

---

👤 **SamuelOliveirads** commented on **2025-08-17** at **00:44:42**

> > Although one may try to solve GPU and NUMA in the same way, it is not unlikely that the approach will be different.
> 
> The approach described in [ggml-org/llama.cpp[#13776](https://github.com/ikawrakow/ik_llama.cpp/issues/13776)](https://github.com/ggml-org/llama.cpp/pull/13776) seems like an approach that would solve both and you can add launching with NUMA backends instead of a global CPU one, or with sacrifice performance and just use RPC and 4 instances bound to each NUMA node, I don't know how complex doing this would be, but it feels like it would be quite complex.
> 
> Edit: [This](https://github.com/koush/llama.cpp/tree/parallel) also exists, which is a continuation of [ggml-org/llama.cpp[#13818](https://github.com/ikawrakow/ik_llama.cpp/issues/13818) (comment)](https://github.com/ggml-org/llama.cpp/pull/13818#issuecomment-2927411762) which as stated "Since the new GPU implementation is a wrapping backend, it should also work with heterogenous devices (if their respective backends implement the new backend requirements" which means it also solve both.

Do you think that [this](https://github.com/ggml-org/llama.cpp/pull/14232) approach could help as the first steps? Sounds like a good first progress, but I didn't see many users testing and sounds like they don't want to merge because of the lack of interest.

---

👤 **saood06** commented on **2025-08-17** at **00:54:33**

>Do you think that https://github.com/ggml-org/llama.cpp/pull/14232 approach could help as the first steps? Sounds like a good first progress, but I didn't see many users testing and sounds like they don't want to merge because of the lack of interest.

That solves NUMA awareness for general mulmats (which is useful in itself), but so far the discussion in this issue has been about tensor parallelism. I actually do want to port the PR you linked but I don't see myself having time in the near future.

---

👤 **SamuelOliveirads** commented on **2025-08-17** at **02:32:20**

> > Do you think that [ggml-org/llama.cpp[#14232](https://github.com/ikawrakow/ik_llama.cpp/issues/14232)](https://github.com/ggml-org/llama.cpp/pull/14232) approach could help as the first steps? Sounds like a good first progress, but I didn't see many users testing and sounds like they don't want to merge because of the lack of interest.
> 
> That solves NUMA awareness for general mulmats (which is useful in itself), but so far the discussion has been about tensor parallelism. I actually do want to port the PR you linked but I don't see myself having time in the near future.

Agreed, I don't have much knowledge or dual socket for testing NUMA, but my guess was that to reach the parallelism its necessary to improve a lot of things first and I was hoping that the PR that I linked was one of it.

---

👤 **saood06** commented on **2025-08-17** at **02:49:37**

> but my guess was that to reach the parallelism its necessary to improve a lot of things first and I was hoping that the PR that I linked was one of it.

To clarify tensor parallelism is (taken from the linked PR) "where a model's tensor computations (e.g., matrix multiplications) can be split across multiple devices (like GPUs or TPUs). This allows different parts of the model's layers to be processed in parallel, improving inference speed and scalability."

What you linked states it would make "mul_mat op [...] only do the computation in local numa part of tensor data src0 and dst according to the ith, cores would be set affinity with the option. also src1 data will quantized into the local numa wdata."

I do actually think if the PR you linked is ported [#287](https://github.com/ikawrakow/ik_llama.cpp/issues/287) should be revisited.