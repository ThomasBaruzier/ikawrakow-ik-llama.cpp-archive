## 🔀 [Pull Request #735](https://github.com/ikawrakow/ik_llama.cpp/pull/735) - Sanitize imatrix

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/sanitize_importance_iqk` |
| **Target Branch** | `main` |
| **Created** | 2025-08-27 |
| **Updated** | 2025-08-29 |
| **Merged** | 2025-08-29 |

---

## 📄 Description

This PR "sanitizes" the provided imatrix before using it for quantization, so that hopefully we will no longer find NaNs in quantized models.

For now we don't cover quantization types where the quantization is done via a function `ggml-quants.c`.  These are basically all legacy quants, k-quants, and i-quants (but repacked variants of these are covered).

@ubergarm Hopefully this PR prevents NaNs in your models where we have observed them.

---

## 💬 Conversation

👤 **ubergarm** commented on **2025-08-27** at **18:07:37**

Thanks this looks like a great feature given no need to re-make an imatrix. I've tested this to re-make two previously broken/nan quants:

* Qwen3-Coder-30B-A3B-Instruct-IQ3_K
 
Unfortunately, it didn't help here given the nans were in `blk.1.ffn_(gate|up)_exps.weight`  which are set to IQ3_K ~~which doesn't look covered in the list of quants mentioned in the commit~~ which looks like the first WIP commit patch covers both IQ3_K and IQ3_KS.

I'll tried this a second time back at my desk, but still not passing the `--validate-quants`.

* Qwen3-Coder-480B-A35B-Instruct-IQ2_KL.gguf

Yes, it does seem to fix this one. I re-quantizied using the existing imatrix with this PR and the resulting gguf passes the new `--validate-quants` test and so far running clean on my usual `llama-perplexity` test against wiki.test.raw on CPU-only backend after 128 chunks in.

I released this model now here: https://huggingface.co/ubergarm/Qwen3-Coder-480B-A35B-Instruct-GGUF#iq2_kl-169597-gib-3034-bpw

---

👤 **ikawrakow** commented on **2025-08-28** at **11:10:59**

I added even more checks specifically for `iq3_ks` and `iq3_k`. Hopefully this finally fixes the NaNs.

---

👤 **ubergarm** commented on **2025-08-28** at **14:30:57**

@ikawrakow   

> Hopefully this finally fixes the NaNs.

Great! Yes! Just re-build this PR735 @ `aa34097` and quantized the Qwen3-Coder-30B-A3B-Instruct-IQ3_K again. Then went back to tip of main and it passes `--validate-quants` and also runs clean perplexity too so I released it here: https://huggingface.co/ubergarm/Qwen3-Coder-30B-A3B-Instruct-GGUF#iq3_k-14509-gib-4082-bpw

I was not able to rebase [#624](https://github.com/ikawrakow/ik_llama.cpp/issues/624) on it now cleanly so didn't use the quantization tweaks despite this being IQ3_K which would likely benefit from that PR.

I do not have any broken nan quants left now, so can't test `ik/sanitize_importance_kt_quants` easily.

Thanks for adding this feature!

---

👤 **Thireus** commented on **2025-08-29** at **07:52:51**

How can I identify if some of the GGUFs I've produced contain Nan?

---

👤 **ikawrakow** commented on **2025-08-29** at **09:57:47**

> How can I identify if some of the GGUFs I've produced contain Nan?

Just add `-vq` to your normal command (`llama-server, llama-cli`, etc.) If there are NaNs in the model, you will get info about the tensors containing NaNs, and the command will terminate.