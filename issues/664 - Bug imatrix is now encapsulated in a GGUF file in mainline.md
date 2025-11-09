## 📌 [Issue #664](https://github.com/ikawrakow/ik_llama.cpp/issues/664) - Bug:  imatrix is now encapsulated in a GGUF file in mainline.

| **Author** | `whatever1983` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-08-01 |
| **Updated** | 2025-08-12 |

---

## 📄 Description

### What happened?

Mainline llama.cpp just wrapped the imatrix.dat file in a gguf format, which means Bartowski's imatrix and mradermacher's imatrix.gguf file can't be used to quantize low bit ik_llama.cpp GGUFs.  What a below the belt hit on you @ikawrakow

mradermacher/Llama-3_3-Nemotron-Super-49B-v1_5-i1-GGUF/blob/main/Llama-3_3-Nemotron-Super-49B-v1_5.imatrix.gguf

Probably need to merge the imatrix GGUF patch from mainline to maintain compatibility.

### Name and Version

llama.cpp

### What operating system are you seeing the problem on?

_No response_

### Relevant log output

```shell

```

---

## 💬 Conversation

👤 **ikawrakow** commented on **2025-08-01** at **06:33:01**

The imatrix here will not be GG-fied.

I'll leave the issue open for a few days for visibility, but then close as "will not fix".

---

👤 **whatever1983** commented on **2025-08-01** at **06:52:52**

Then you are just increasing the computation cost of using ik_llama.cpp if people can't just grab the imatrix from bartowski.  imatrix then would have to be recomputed for each new model.  Luckily, I pretty much only use IQ6K these days without the need for imatrix but you do realize that it is a hidden negative force against you right?

---

👤 **ikawrakow** commented on **2025-08-01** at **08:13:47**

1. I don't think there is anything special about imatrix data from Bartowski, mrademacher, or Unsloth. As far as I can tell, @ubergarm is making much better quantized models without any use of llama.cpp 
2. If one insists on using a GG-fied imatrix, one can convert to the original format. See the GG-fication PR in llama.cpp for details

---

👤 **ubergarm** commented on **2025-08-01** at **15:28:36**

@whatever1983 

Please see here where the ability to convert to the original format is shown: https://github.com/ikawrakow/ik_llama.cpp/issues/659#issuecomment-3141417415

---

👤 **espen96** commented on **2025-08-04** at **07:29:28**

My understanding is that the gguf imatrix had some special benefits for MoE models? 

No personal opinions, but I'm curious as to why we want to stick to the classic format if this one is allegedly better

---

👤 **ikawrakow** commented on **2025-08-04** at **08:42:38**

It is not better. The sole purpose of the mainline activity is to slap the GG label onto the imatrix.

---

👤 **Nexesenex** commented on **2025-08-11** at **22:26:45**

@ikawrakow, sadly the conversion doesn't always work.

First, I had to revert on mainline the '--output format dat" commit which bungled the process. Otherwise it would not even recognize the command.

Ok, the converts works.. In appearance Then, I get this when I convert : Q:\LLAMA_CUDA_MASTER_MMQ>llama-imatrix --in-file Q:\iMatrix\mradermacher\gpt-oss-120b.imatrix.gguf -o Q:\iMatrix\mradermacher\gpt-oss-120b.imatrix.dat
ggml_cuda_init: GGML_CUDA_FORCE_MMQ:    yes
ggml_cuda_init: GGML_CUDA_FORCE_CUBLAS: no
ggml_cuda_init: found 3 CUDA devices:
  Device 0: NVIDIA GeForce RTX 3090, compute capability 8.6, VMM: yes
  Device 1: NVIDIA GeForce RTX 3090, compute capability 8.6, VMM: yes
  Device 2: NVIDIA RTX A4000, compute capability 8.6, VMM: yes
build: 6123 (79c1160b0) with MSVC 19.44.35213.0 for Windows AMD64
main : loading imatrix from 'Q:\iMatrix\mradermacher\gpt-oss-120b.imatrix.gguf'
No prompt provided; combining precomputed matrices only.
main : saving imatrix to 'Q:\iMatrix\mradermacher\gpt-oss-120b.imatrix.dat'

save_imatrix: saving imatrix using GGUF format with a different suffix than .gguf
save_imatrix: if you want the previous imatrix format, use --output-format dat

save_imatrix: entry '               blk.35.ffn_up_exps.weight' has partial data (98.44%)
save_imatrix: entry '               blk.34.ffn_up_exps.weight' has partial data (99.22%)
save_imatrix: entry '             blk.32.ffn_down_exps.weight' has partial data (99.22%)
save_imatrix: entry '               blk.32.ffn_up_exps.weight' has partial data (99.22%)
save_imatrix: entry '             blk.32.ffn_gate_exps.weight' has partial data (99.22%)
save_imatrix: entry '             blk.34.ffn_down_exps.weight' has partial data (99.22%)
save_imatrix: entry '             blk.34.ffn_gate_exps.weight' has partial data (99.22%)
save_imatrix: entry '             blk.35.ffn_down_exps.weight' has partial data (98.44%)
save_imatrix: entry '             blk.35.ffn_gate_exps.weight' has partial data (98.44%)
PS C:\Users\Addlmeign>

So, ok, let's try to use the produced .dat, I do not need those ffns anyway, I just requant attn_q and attn_output.

2. I try to quantize, and..
Q:\LLAMA_IK_CUSTOM>llama-quantize --allow-requantize --keep-split --imatrix Q:\iMatrix\mradermacher\gpt-oss-120b.imatrix.dat --custom-q "blk\..*\.attn_q.*=q6_0, blk\..*\.attn_output.*=q6_0, blk\..*\.attn_k.*=q8_0, blk\..*\.attn_v.*=q8_0, token_embd\.weight=q8_0, output.weight=q8_0" X:\text-generation-webui\models\gpt-oss-120b-mxfp4-v1-00001-of-00003.gguf X:\text-generation-webui\models\gpt-oss-120b-mxfp4-v1_aqo60.gguf mxfp4
Adding custom rule blk\..*\.attn_q.* -> q6_0
Adding custom rule blk\..*\.attn_output.* -> q6_0
Adding custom rule blk\..*\.attn_k.* -> q8_0
Adding custom rule blk\..*\.attn_v.* -> q8_0
Adding custom rule token_embd\.weight -> q8_0
Adding custom rule output.weight -> q8_0
load_imatrix: failed reading number of values for entry 1

Maybe I'm doing something wrong, but this is very problematic. I know you have no responsibility in this "rebranding" of your very own work by the mainline team, and I find it quite inopportune. But it's problematic to not be able to use the published imatrixes on Hugging Face on IK_Llama.

I know I'm not an expert about those things, but I'm so f****ng tired of all this hostile coding coming from somewhere under the guise of neat factoring and packaging. It's not only in dismissal of your work, but also a middle finger to all your contributors and users, their time, and their enjoyment of dwelling (or trying to, in my case) with the Llama.cpp ecosystem. It's like I'm plugging a hole, and 2 more appear.

I'm sorry, I needed to vent somewhere after hours of trying to conciliate all approaches through my forks without success for now.

Please, allow GGUF imatrixes compatibility on IK_Llama, IK. I understand quite well why you're opposed to the idea, and that you say "will not fix". But, please.

I feel better. ^^
Cheers to you all, sincerely.

---

👤 **compilade** commented on **2025-08-11** at **22:56:02**

@Nexesenex 
Sorry about that.

> Ok, the converts works.. In appearance Then, I get this when I convert : 
> ```
> Q:\LLAMA_CUDA_MASTER_MMQ>llama-imatrix --in-file Q:\iMatrix\mradermacher\gpt-oss-120b.imatrix.gguf -o Q:\iMatrix\mradermacher\gpt-oss-120b.imatrix.dat
> ```

This warning
```
save_imatrix: saving imatrix using GGUF format with a different suffix than .gguf
save_imatrix: if you want the previous imatrix format, use --output-format dat
```
means the file was saved in the GGUF format. Use the `--output-format dat` option when converting the imatrix file.

In retrospect, <https://github.com/ggml-org/llama.cpp/pull/14842> (which made the output type GGUF by default ***regardless of the output filename suffix***) was a mistake, and a warning (<https://github.com/ggml-org/llama.cpp/pull/15076>) was apparently not enough.

(there are things which the GGUF imatrix format makes possible which weren't before, like <https://github.com/ggml-org/llama.cpp/pull/15060>, which relies on per-expert evaluation counts. Going from `imatrix.gguf` to `imatrix.dat` is lossy (mostly regarding the per-expert counts))

To avoid this conversion problem from happening in the future, I will change back the behavior of mainline so that the `.dat` suffix should be considered when `--output-format` is not provided, otherwise this leads to the situation you've bumped into.

---

👤 **saood06** commented on **2025-08-11** at **23:10:17**

>I know I'm not an expert about those things, but I'm so f****ng tired of all this hostile coding coming from somewhere under the guise of neat factoring and packaging.

I don't think it is fair to call @compilade's work that. Yes, I can see that it is causing you problems but if you actually look at all his communication around https://github.com/ggml-org/llama.cpp/pull/9400 I don't see hostile coding.

>my forks

I do respect the work you put into these (is there a reason you don't have issues or discussions available on `croco.cpp`?). It's just I feel like it is unwarranted.

---

👤 **Nexesenex** commented on **2025-08-11** at **23:27:33**

@compilade : Thanks for writing this, and sorry for my earlier tone, I was quite a bit in dismay because that commit was indeed problematic (because it was broken on my last llama.cpp compilation, the dat format not even being recognized as a possible output) beyond the small benefit it was to provide, thus my harsh comment. I came to the same conclusion about simply dismissing the commit which I couldn't make work.

@saood06 : I'm prompt to react since the GGUF v14 episode, which already prevented IKL's quants to be loaded for no apparent reason but "coding orthodoxy", to the disbenefit of compatibility between IKL and mainline.

As for my forks, honestly, they are not well-maintained enough to be worth of a community involvement into them, they are more of a personal project (I'm not a dev, I don't know how to code, I just copy paste, refactor what I can, and sometimes narrow down some bits of code with the help of AI like for the numpy converting of q6_0, but my competences do not go beyond that). BUT, the functionalities my forks (may it be Croco, may it be my bungled hybrids between IKL and mainline) try to provide should be focal to mainline llama.cpp, who is still stuck on non-SOTA quants, including for an issue as critical as quantizing properly the ffn_down of irregular shapes of the Qwen architecture and derivatives (Q6_0 not being available on mainline).

---

👤 **saood06** commented on **2025-08-11** at **23:40:55**

>As for my forks, honestly, they are not well-maintained enough to be worth of a community involvement into them

They do have over 100 stars and 2 people creating and committing on forks within the last month.

---

👤 **Nexesenex** commented on **2025-08-12** at **00:44:30**

> They do have over 100 stars and 2 people creating and committing on forks within the last month.

True.. And I finally solved my problem by implanting properly Q6_0 on the current mainline. I had left a typo (a q5_0 instead of a q6_0 somewhere). Yet another drama of copy-pasting. :X

---

👤 **ikawrakow** commented on **2025-08-12** at **06:42:49**

> (there are things which the GGUF imatrix format makes possible which weren't before, like https://github.com/ggml-org/llama.cpp/pull/15060, which relies on per-expert evaluation counts.

I personally don't find the linked feature useful. But I'm happy to be proven wrong, and if this happens, I'll implement it here without requiring GGUF to do so.

> Going from imatrix.gguf to imatrix.dat is lossy (mostly regarding the per-expert counts))

A lot has been said about this particular topic, so let me address it in some more detail. The absolute normalization of the imatrix data is irrelevant (normalization cancels out when used for quantization). Hence, the "lossyness" of the format is only relevant when combining imatrix data files. There are two main considerations here
1. Combining imatrix data files computed with different context windows
2.  Combining imatrix data files for MoE models

### 1. Combining imatrix data files computed with different context windows

If we look at my [original imatrix PR](https://github.com/ggml-org/llama.cpp/pull/4861), you will see that what got saved there for imatrix entry $x_i$ was $x_i = \sum_j a_{ij}^2$, where $a_{ij}$ is the activation for entry $i$ and token $j$. Hence, correctly combining imatrix data files from different calculations $C$ is trivial (and independent of context window) as all one needs to do is $x_i = \sum_C x_i^C$, where $x_i^C$ is the saved result for calculation $C$. My PR was then "improved" by the mainline maintainers many times, so what happens today is just plain wrong. I have inherited the wrong implementation when I forked the project, but haven't bothered to fix it. In the early imatrix days I played a lot with that sort of thing, and have my own unpublished tools to do that, including the ability to manually assign weights to the individual imatrix data files (more on that below). But, not having found any real advantage from combining imatrix data files with different context windows, I never published these tools and, at least on my book, this is not an important use case. But if someone feels that it is important to them, they can add a feature request, and I'll fix it. No GGUF required.

### 2.  Combining imatrix data files for MoE models

If I understand correctly, here the concern is this: if I run calculation $C_1$, and expert $X$ gets activated $N_1$ times, then run a calculation $C_2$, and expert $X$ gets activated $N_2$ times, we cannot meaningfully combine $C_1$ and $C_2$ because $N_1$ and $N_2$ were not recorded in the imatrx data. If mainline maintainers had not "improved" the way imatrix data is handled, also this will just-work<sup>TM</sup> as it is effectively equivalent to combining calculations with different context windows. But because it was broken, it was claimed that GGUF is required to store additional information, so one can fix it. It is fixable without GGUF, see above.

If one wants to seriously engage with combining imatrix data from different calculations (different context lengths, different calibration data sets, with/without special tokens, etc.), what is actually required is the ability to assign weights to the individual calculations. With that in place, the normalization of the individual calculations becomes irrelevant because it can be handled via the manually assigned weights. That should have been the feature that gets implemented, along with reverting the "improvements", and not packaging the data in GGUFs.

Hence, as I said above, the sole purpose of the GGUF imatrix activity is to slap the GG label on the imatrix.

---

👤 **ikawrakow** commented on **2025-08-12** at **06:44:29**

The issue has been now open long enough. With my last comment there is enough context why adding the misguided GGUF packaging is not required and hence will not happen here, so closing.

---

👤 **compilade** commented on **2025-08-12** at **14:36:28**

@ikawrakow Thank you for giving more context.

> The absolute normalization of the imatrix data is irrelevant (normalization cancels out when used for quantization). Hence, the "lossyness" of the format is only relevant when combining imatrix data files.

True.

> If we look at my [original imatrix PR](https://github.com/ggml-org/llama.cpp/pull/4861), you will see that what got saved

The original format does indeed seem easier to handle for correctly merging multiple files, since the per-tensor evaluation counts aren't stored at all.

Even stacked MoE could be handled correctly, I think. Partial data, not so much, or maybe at least case-by-case when the values are 0.

The main problematic quirk (which was **not added by you**, it was added in <https://github.com/ggml-org/llama.cpp/pull/7099>, and I'm not sure what it attempted to fix) in the current `imatrix.dat` format is how the evaluation counts are handled (which, as you've described, don't even need to be handled at all).

>  If mainline maintainers had not "improved" the way imatrix data is handled \[...\]
> \[...\]
> That should have been the feature that gets implemented, along with reverting the "improvements"

I didn't want to break existing imatrix files (unlike <https://github.com/ggml-org/llama.cpp/pull/7099>), and I still wanted extensibility for future changes. Removing the evaluation counts (as in the original imatrix format) or adding per-expert counts (as in the GGUF imatrix format) without breaking existing files is hard because (almost) nothing in the `imatrix.dat` format can be used to distinguish the shape of the content.
I *could* have used the entries count as a version field (as you suggested in <https://github.com/ikawrakow/ik_llama.cpp/discussions/15#discussioncomment-10626253>), but since it's a breaking change anyway, I felt like I might as well use an existing extensible format as the basis for imatrix data.

Something as banal as changing the type of a metadata field in `imatrix.dat` (e.g. storing a list of dataset names instead of only one) would require a version bump, which can easily be forgotten.

Optional metadata also isn't possible except at the end, and requires all previous fields to be present unless the format is changed to store the key names.

Similarly, storing imatrix data in other types than F32 in `imatrix.dat` is also not exactly trivial without imatrix versions, and separate reader functions.

> Hence, as I said above, the sole purpose of the GGUF imatrix activity is to slap the GG label on the imatrix.

I chose GGUF, but it could also have been Safetensors (and can still be). My intention had nothing to do with slapping the GG label on imatrix.

It could be argued that the format doesn't need to be extended. However, sometimes experiments require storing more types of data, and if that ends up being useful (and gets pushed to a PR), extensibility makes it easier to avoid breaking existing files.

So I agree it's not absolutely necessary to use GGUF for storing imatrix data, but I think it makes it easier for other contributors to avoid breaking existing files when changing what is stored.

If you change the `imatrix.dat` format, as long as it's distinguishable, I can add support for it as another `--output-format` on mainline so that you won't have to handle GGUF imatrix if you don't want to (because people *will* try to use imatrix files calculated on mainline with `ik_llama.cpp` (as in <https://github.com/ikawrakow/ik_llama.cpp/issues/659> and here), and I suspect the other way around also happens).

---

👤 **ikawrakow** commented on **2025-08-12** at **15:48:37**

@compilade 

Thank you for your detailed and, as always, thoughtful response.

> I chose GGUF, but it could also have been Safetensors (and can still be)

I would have been really curious to see the fate of a PR changing the format without using GGUF.

As for GGUF, it is not really a general purpose data format. The reference implementation does not handle anything other than `ggml` data types (try loading one of ubergarm's quants and see what happens). The reference implementation continues to depend on a lot of `ggml` stuff, and it gets bundled into a `ggml-base` library that can not be built without `ggml.c, ggml-backend.cpp`, and the kitchen sink. Granted, `ggml-base` is now "just" a 700 kB shared library (on Linux), so that's a huge improvement compared to the 300 MB `libggml.so` file one needed when the discussion about using GGUF first started. Still, I cannot just drop the GGUF reader/writer from mainline here and use it, as Nexesenex had to find out the hard way.  

But in any case, thank you for offering to make changes to the mainline imatrix implementation.

I don't think there is a need for changes at this point (other than perhaps the auto-saving as GGUF when combining imatrix data that was mentioned above), but I'll come back to you in case the need arises.