## 🔀 [Pull Request #881](https://github.com/ikawrakow/ik_llama.cpp/pull/881) - Server middleware refactor PRs

| **Author** | `Nexesenex` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Source Branch** | `2_more_server_PRs` |
| **Target Branch** | `main` |
| **Created** | 2025-10-30 |
| **Updated** | 2025-11-05 |

---

## 📄 Description

Pursuing on my quest to update a bit the server, here's the Middleware refactor, following straight the bunch of 6 server PRs merged recently.

Here are the mainline PRs involved:

server : refactor middleware and /health endpoint ([#9056](https://github.com/ikawrakow/ik_llama.cpp/issues/9056))

* server : refactor middleware and /health endpoint
* move "fail_on_no_slot" to /slots
* Update examples/server/server.cpp
* fix server tests
* fix CI
* update server docs

server : fix crash when error handler dumps invalid utf-8 json ([#9195](https://github.com/ikawrakow/ik_llama.cpp/issues/9195))

This time, I had to solve a bunch of conflicts, of course.

On my side, it works properly, and I tried speculative decoding as well without apparent bug, but I need a review by someone more competent before it can be merged, if pertinent.

- [x] I have read the [contributing guidelines](https://github.com/ggerganov/llama.cpp/blob/master/CONTRIBUTING.md)
- Self-reported review complexity:
  - [ ] Low
  - [x] Medium
  - [ ] High

---

## 💬 Conversation

👤 **ikawrakow** commented on **2025-10-30** at **16:18:22**

I'm not very familiar with the server code, so invite review by others.

---

👤 **Nexesenex** commented on **2025-10-30** at **18:17:25**

@saood06 , @firecoperana , could any of your folks get a look at this PR when you have some spare time?

---

👤 **firecoperana** commented on **2025-10-30** at **23:23:40**

I just tried this PR, and I don't see any errors when using the webui. This PR changes return results for endpoints like health. It also removed Access-Control-Allow-Origin in all endpoints. This is good from the security perspective, but not sure how it affects others. It could be no issue since it was merged in mainline a long time ago.

---

👤 **Nexesenex** commented on **2025-10-30** at **23:32:35**

Great! Then one more pair of eyes, and then, we're good to go!

---

👤 **Nexesenex** commented on **2025-11-05** at **17:59:45**

Well, whatever.