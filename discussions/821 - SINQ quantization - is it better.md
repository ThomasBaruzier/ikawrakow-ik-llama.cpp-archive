## 🗣️ [Discussion #821](https://github.com/ikawrakow/ik_llama.cpp/discussions/821) - SINQ quantization - is it better?

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | ✅ **Open** |
| **Created** | 2025-10-09 |
| **Updated** | 2025-10-09 |

---

## 📄 Description

A new quantization method called SINQ (from SINKHORN-NORMALIZED QUANTIZATION) has been published, see [paper](https://arxiv.org/pdf/2509.22944), and promptly there is an [issue](https://github.com/ggml-org/llama.cpp/issues/16478) in mainline `llama.cpp` to implement that as a "low hanging fruit".

The main idea of the SINQ paper is to use what they call "double scaling". I.e., if we consider a model tensor as a collection of `N x M` tiles, instead of having a single float scale per tile, we represent it as $S_{ij} = f_i g_j$, where $i$ and $j$ are the tensor row ans scale, respectively. The idea seemed intriguing enough, but it adds an additional matrix - vector multiplication to the computation graph (activations times column scales $g_j$), which adds a significant overhead to a potential implementation in `ik_llama.cpp` (or `llama.cpp`) Hence, I decided to first see how this quantization method compares to the quantization types available in `ik_llama.cpp`.

In their evaluation the SINQ authors focus on the dense Qwen3 series of models (Qwen3-1.7B, Qqwen3-14B and Qqwen3-32B). In the interest of time spent, I'll use just Qwen3-14B (which I happened to have lying around on my computer). They have results for Wikitext2 perplexity (Table 1 of the paper). Given that they don't state the context used to evaluate PPL, and given that PPL values computed with `llama/ik_llama.cpp` are different from what one gets using Python evaluation tools, we will look at **quantization error** $E_Q$ defined as

$$E_Q = \frac{PPL(Q)}{PPL(bf16)} - 1$$

where $Q$ is the quantization type, $PPL(Q)$ is the perplexity of the quantized model, and $PPL(bf16)$ is the perplexity of the `bf16` model. $E_Q$ is (nearly) independent of the specifics of perplexity implementation and the context length. For Qwen3-14B the paper quotes $PPL(bf16) = 8.64$. The table show what we get with `ik_llama.cpp` for a few context lengths

| Context | PPL(bf16) |
| ---: | ---: |
| 512 | 9.026 |
| 768 |  8.352 |
| 1024 | 8.095 |

I.e., their perplexity calculation with unknown context length corresponds to `ik_llama.cpp` perplexity with a context length of somewhere between 512 and 768. For simplicity I will just use context of 512.

Based on model sizes quoted in the paper, the output tensor and the token embedding tensor must have been left unquantized at `bf16`. For Qwen3-14B the embedding size is 5120 and the vocabulary size is 151936, so these two tensors contribute 3.112 GB to the model size. Excluding output and embeddings, Qwen3-14B has 13.212 B parameters. Their "3-bit" model size is 9.25 GB, so `(9.25 - 3.112)/13.212 x 8 = 3.72` bits-per-weight (bpw). The "4-bit" model size is 10.56 GB, so `(10.56 - 3.112)/13.212 x 8 = 4.51` bpw. I.e., their "3-bit" is larger than `IQ3_K, IQ3_S, Q3_K` (3.4375 bpw), while the "4-bit" is about the same as `Q4_K` and `IQ4_K`. 

The next table shows a comparison between the SINQ quants and a few `ik_llama.cpp` quants for $E_Q$. To be consistent with the SINQ paper, output and embedding tensors are left as `bf16`, and the effective bpw listed is only for the repeating layers. All `ik_llama.cpp` quantizations are "pure" (`--pure` command line option), except for `IQ2_KL`, which uses `IQ4_KS` for `attn_v` and `attn_output`.

| Quantization | bpw | E_Q (%) |
| ---: | ---: | ---: |
| SINQ 3 bit | 3.72 | 7.99 |
| SINQ 4 bit | 4.51 | 1.39 |
| IQ2_KL | 2.84 | 8.32 |
| IQ3_KT | 3.13 | 5.32 |
| IQ3_K  | 3.44 | 3.39 |
| IQ4_KS | 4.26 | 0.49 |
| Q3_K | 3.44 | 4.75 |
| Q4_K  | 4.51 | 1.23 |

A plot of the data in the above table is shown in the following graph (note that the y-axis is logarithmic) 
 
<img width="792" height="612" alt="u2" src="https://github.com/user-attachments/assets/27efd0d2-c154-4cf8-afdb-b6dce21a2baa" />


We see that SINQ cannot even match k-quants (published ~2.5 years ago). The newer `ik_llama.cpp` quants achieve the same quantization error as SINQ using 0.7-0.9 fewer bits per weight.

So, in short, not worth adding this new quantization method.