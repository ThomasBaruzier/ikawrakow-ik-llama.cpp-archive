## 🗣️ [Discussion #819](https://github.com/ikawrakow/ik_llama.cpp/discussions/819) - KV cache dump and reload with different -b and -ub sizes?

| **Author** | `magikRUKKOLA` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-10-06 |
| **Updated** | 2025-10-08 |

---

## 📄 Description

I noticed that the increase of the -u and -ub from 4k to 8k in my case resulted in increase in prefill from ~135 tps to 232 tps.  I was wondering if its possible to first run llama-server with 8k batches, then, once the prefill is finished, dump the KV cache, then restart the llama-server with lesser batch size in order to push for more context and load up the previously saved KV-cache?

If its possible, then it would allow to use the maximum prefill speed and at the same time to support as much context as one can squeeze the -b and -ub parameters. :)

---

## 💬 Discussion

👤 **saood06** commented on **2025-10-06** at **05:28:54**

Yes it is possible, it was also suggested [here](https://github.com/ikawrakow/ik_llama.cpp/issues/686#issuecomment-3190421967) when I made a related change.