## 🔀 [Pull Request #872](https://github.com/ikawrakow/ik_llama.cpp/pull/872) - A few server commits from mainline.

| **Author** | `Nexesenex` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `a-few-server-commits-from-mainline-` |
| **Target Branch** | `main` |
| **Created** | 2025-10-27 |
| **Updated** | 2025-10-28 |
| **Merged** | 2025-10-28 |

---

## 📄 Description

server : handle models with missing EOS token ([#8997](https://github.com/ikawrakow/ik_llama.cpp/issues/8997))

server : fix segfault on long system prompt ([#8987](https://github.com/ikawrakow/ik_llama.cpp/issues/8987))
* server : fix segfault on long system prompt
* server : fix parallel generation with very small batch sizes
* server : fix typo in comment

server : init stop and error fields of the result struct ([#9026](https://github.com/ikawrakow/ik_llama.cpp/issues/9026))

server : fix duplicated n_predict key in the generation_settings ([#8994](https://github.com/ikawrakow/ik_llama.cpp/issues/8994))

server : support reading arguments from environment variables ([#9105](https://github.com/ikawrakow/ik_llama.cpp/issues/9105))
* server : support reading arguments from environment variables
* add -fa and -dt
* readme : specify non-arg env var

server : add some missing env variables ([#9116](https://github.com/ikawrakow/ik_llama.cpp/issues/9116))
* server : add some missing env variables
* add LLAMA_ARG_HOST to server dockerfile
* also add LLAMA_ARG_CONT_BATCHING

Credits go to the respective authors and contributors on the Llama.cpp repo.

Not a single merge conflict occurred.
Compiled, then tested without bug.

Maybe it can be worth it to catch up a little bit with the work already done on mainline before modifying too much the server.cpp?

- [x] I have read the [contributing guidelines](https://github.com/ggerganov/llama.cpp/blob/master/CONTRIBUTING.md)
- Self-reported review complexity:
  - [x] Low
  - [ ] Medium
  - [ ] High

---

## 💬 Conversation

👤 **ikawrakow** approved this pull request ✅ on **2025-10-28** at **07:58:02**