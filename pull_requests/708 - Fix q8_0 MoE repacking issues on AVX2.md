## 🔀 [Pull Request #708](https://github.com/ikawrakow/ik_llama.cpp/pull/708) - Fix q8_0 MoE repacking issues on AVX2

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/fix_q80_moe_avx2` |
| **Target Branch** | `main` |
| **Created** | 2025-08-19 |
| **Updated** | 2025-08-20 |
| **Merged** | 2025-08-19 |

---

## 📄 Description

`Q8_0` on `AVX2` is a bit of a mess. 

`Q8_0` needs `Q0_0_X4` for the activations, but `Q8_0_R8` needs `Q8_2_X4`.
Hence, if we decide to repack a `Q8_0` MoE tensor to `Q8_0_R8`, ` iqk_moe_fused_up_gate` fails because the activations were prepared as `Q0_0_X4`, but we now need `Q8_2_X4`. This leads to an assert in `ggml_compute_forward_mul_mat_id_up_gate`.

For now a simple fix: if the fast path fails, take the slow path instead of just bailing out.

This is of course sub-optimal as it leads to a significantly lower performance for MoE models where the MoE tensors are `Q8_0`. But then again, why would anyone want to use that?

Closes [#691](https://github.com/ikawrakow/ik_llama.cpp/issues/691)

---

## 💬 Conversation

👤 **sousekd** commented on **2025-08-19** at **19:40:46**

> This is of course sub-optimal as it leads to a significantly lower performance for MoE models where the MoE tensors are `Q8_0`. But then again, why would anyone want to use that?

@ikawrakow I tend to use the largest possible quants I am able to run at satisfactory speed for the task. It is quite often Q8 or Q6. Do I do this wrong?

---

👤 **ikawrakow** commented on **2025-08-20** at **04:29:34**

@sousekd IIRC, you have a very recent CPU with Zen5 cores, so you are not affected by the `AVX2` issues. If you are satisfied with the TG performance you get with `Q8_0`, then sure, you can use that and you are on the safe side. But I doubt that a giant MoE model has more than 5 or 6 bits of information stored in the MoE tensor weights (and this is why people can get away with quantizing these models to 4, 3, or even 2 bpw without the model becoming useless).

---

👤 **Ph0rk0z** commented on **2025-08-20** at **10:22:25**

Some UD quants may have Q8_0 tensors in them on different parts of the model. I have never got this assert despite only having AVX512 with no fancy. Probably no up/gate are that size though, if it's confined there.