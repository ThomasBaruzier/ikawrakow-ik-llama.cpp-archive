## 🔀 [Pull Request #871](https://github.com/ikawrakow/ik_llama.cpp/pull/871) - Try removing copy indirection

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 📝 **Draft** |
| **Source Branch** | `ik/try_remove_cpy_indirection` |
| **Target Branch** | `main` |
| **Created** | 2025-10-27 |
| **Updated** | 2025-10-27 |

---

## 📄 Description

_No description provided._

---

## 💬 Conversation

👤 **CISC** started a conversation on `ggml/src/ggml-cuda.cu` on **2025-10-27** at **10:23:28**

Why do this still?

> 👤 **ikawrakow** replied on **2025-10-27** at **10:36:28**
> 
> Oh, I forgot to comment it out. But it does not change the behavior.

> 👤 **CISC** replied on **2025-10-27** at **10:38:25**
> 
> Didn't expect it to, guess something else is triggering the graph disable, no hints in debug logs?

---

👤 **CISC** reviewed this pull request 💬 on **2025-10-27** at **10:32:00**

A bit hard to tell since there are other changes here and you are keeping the indirection code, but apart from the pointer copy it should be ok.

---

👤 **ikawrakow** commented on **2025-10-27** at **10:42:29**

I didn't spend too much time on it as the reward will be very modest (1% or less in TG). But my guess is that it has something to do with the fact that mainline now uses `SET_ROWS` in the KV cache handling, so this must somehow result in the copy operation always being to the same pointer. In `ik_llama.cpp` the copy goes into the corresponding location of the KV caches, so changes with each token, so CUDA_GRAPHS cannot work without the copy indirection.

But if the answer is not obvious, and you don't know it, don't waste your time on it.

---

👤 **CISC** commented on **2025-10-27** at **10:48:09**

> I didn't spend too much time on it as the reward will be very modest (1% or less in TG). But my guess is that it has something to do with the fact that mainline now uses `SET_ROWS` in the KV cache handling, so this must somehow result in the copy operation always being to the same pointer. In `ik_llama.cpp` the copy goes into the corresponding location of the KV caches, so changes with each token, so CUDA_GRAPHS cannot work without the copy indirection.

Ah, I was not aware you didn't fully sync that change, could very well be part of the issue.

> But if the answer is not obvious, and you don't know it, don't waste your time on it.

No worries.

---

👤 **ikawrakow** commented on **2025-10-27** at **11:17:18**

> Ah, I was not aware you didn't fully sync that change, could very well be part of the issue.

Oh, I thought you were aware of it. When it comes to the computational part, I (almost) never sync with mainline, and I think we both know that most of the syncing happens the other way around.

---

👤 **CISC** commented on **2025-10-27** at **11:36:06**

> > Ah, I was not aware you didn't fully sync that change, could very well be part of the issue.
> 
> Oh, I thought you were aware of it. When it comes to the computational part, I (almost) never sync with mainline, and I think we both know that most of the syncing happens the other way around.

I know your fork is far off sync, but I saw you synced `SET_ROWS` with the `BailingMoeV2` PR, so I assumed you had synced this part completely.

---

👤 **ikawrakow** commented on **2025-10-27** at **12:05:08**

> I know your fork is far off sync, but I saw you synced SET_ROWS with the BailingMoeV2 PR

The `SET_ROWS` function was necessary because of the way you had built the grouped expert routing, and not because it is used anywhere else during inference (or graph building). But after implementing the actually correct grouped expert routing, `SET_ROWS` is no longer required, so in principle could be removed again.

---

👤 **CISC** commented on **2025-10-27** at **12:17:35**

> > I know your fork is far off sync, but I saw you synced SET_ROWS with the BailingMoeV2 PR
> 
> The `SET_ROWS` function was necessary because of the way you had built the grouped expert routing, and not because it is used anywhere else during inference (or graph building). But after implementing the actually correct grouped expert routing, `SET_ROWS` is no longer required, so in principle could be removed again.

Since your problem stems from `ggml_cpy` being used to update the KV cache you might want to keep `SET_ROWS` though.

---

👤 **ikawrakow** commented on **2025-10-27** at **12:34:34**

> Since your problem stems from ggml_cpy being used to update the KV cache you might want to keep SET_ROWS though.

It took several months in mainline to get to the point of the KV cache reorganization being in a working state and most bugs ironed out. With the main purpose of this massive change being? (apart from demonstrating activity :wink: )

---

👤 **CISC** commented on **2025-10-27** at **12:45:14**

> > Since your problem stems from ggml_cpy being used to update the KV cache you might want to keep SET_ROWS though.
> 
> It took several months in mainline to get to the point of the KV cache reorganization being in a working state and most bugs ironed out. With the main purpose of this massive change being? (apart from demonstrating activity 😉 )

The meaning of life is to keep busy, no? I think you know there's more to the KV cache changes than that though, and as you say it's mostly free of kinks now. :)

---

👤 **ikawrakow** commented on **2025-10-27** at **14:09:45**

So, this does not work.