## 🔀 [Pull Request #695](https://github.com/ikawrakow/ik_llama.cpp/pull/695) - Added "usage" to server response

| **Author** | `dpk-it` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `fix_usage_metrics` |
| **Target Branch** | `main` |
| **Created** | 2025-08-15 |
| **Updated** | 2025-08-17 |
| **Merged** | 2025-08-17 |

---

## 📄 Description

I added sending usage metrics that llama-swap and others use to display statistics, also corrected the formula for calculating timings and enabled `timings_per_token` by default (in theory it shouldn't have much of an impact on performance)

- [x] I have read the [contributing guidelines](https://github.com/ggerganov/llama.cpp/blob/master/CONTRIBUTING.md)
- Self-reported review complexity:
  - [x] Low
  - [ ] Medium
  - [ ] High

<img width="1041" height="815" alt="Screenshot 2025-08-15 at 3 06 20 PM" src="https://github.com/user-attachments/assets/847bc3b0-7e53-404d-82b8-2f67ae41baee" />

---

## 💬 Conversation

👤 **saood06** commented on **2025-08-15** at **22:41:08**

I'm curious why the `/metrics` endpoint doesn't cover it, to me it looks sufficient? Example output of it below

```
# HELP llamacpp:prompt_tokens_total Number of prompt tokens processed.
# TYPE llamacpp:prompt_tokens_total counter
llamacpp:prompt_tokens_total 6886
# HELP llamacpp:prompt_seconds_total Prompt process time
# TYPE llamacpp:prompt_seconds_total counter
llamacpp:prompt_seconds_total 2903.63
# HELP llamacpp:tokens_predicted_total Number of generation tokens processed.
# TYPE llamacpp:tokens_predicted_total counter
llamacpp:tokens_predicted_total 3469
# HELP llamacpp:tokens_predicted_seconds_total Predict process time
# TYPE llamacpp:tokens_predicted_seconds_total counter
llamacpp:tokens_predicted_seconds_total 3544.27
# HELP llamacpp:prompt_tokens_seconds Average prompt throughput in tokens/s.
# TYPE llamacpp:prompt_tokens_seconds gauge
llamacpp:prompt_tokens_seconds 2.37151
# HELP llamacpp:predicted_tokens_seconds Average generation throughput in tokens/s.
# TYPE llamacpp:predicted_tokens_seconds gauge
llamacpp:predicted_tokens_seconds 0.978764
# HELP llamacpp:kv_cache_usage_ratio KV-cache usage. 1 means 100 percent usage.
# TYPE llamacpp:kv_cache_usage_ratio gauge
llamacpp:kv_cache_usage_ratio 0.491063
# HELP llamacpp:kv_cache_tokens KV-cache tokens.
# TYPE llamacpp:kv_cache_tokens gauge
llamacpp:kv_cache_tokens 31428
# HELP llamacpp:requests_processing Number of request processing.
# TYPE llamacpp:requests_processing gauge
llamacpp:requests_processing 0
# HELP llamacpp:requests_deferred Number of request deferred.
# TYPE llamacpp:requests_deferred gauge
llamacpp:requests_deferred 0
```

---

👤 **dpk-it** commented on **2025-08-15** at **23:07:01**

as far as I understand "usage" should always be sent if finish_reason is not empty, and the way it works now is a bug. timings were also calculated incorrectly ...

this is useful when using something like llama-swap when the server stops or restarts, and it is always more convenient to be able to view metrics in the ui along with the message if there is support for them out of the box

---

👤 **dpk-it** commented on **2025-08-15** at **23:32:49**

But perhaps `usage` should come as the very last message after `finish_reason` has been filled in, and this message should look something like this.. at least openrouter does it this way... I can't check it with openai now.
```
{
  "id": "gen-1755300171-axbfuztqWJDURQNs6dD2",
  "model": "openai/gpt-5-mini",
  "object": "chat.completion.chunk",
  "created": 1755300171,
  "choices": [
    {
      "index": 0,
      "delta": {
        "role": "assistant",
        "content": ""
      },
      "finish_reason": null,
      "native_finish_reason": null,
      "logprobs": null
    }
  ],
  "usage": {
    "prompt_tokens": 713,
    "completion_tokens": 17,
    "total_tokens": 730
  }
}
```

I don't know how critical this is. `llama.cpp` does it this way so it's probably ok.. 

```
...
data: {"choices":[{"finish_reason":null,"index":0,"delta":{"content":" to"}}],"created":1755304024,"id":"chatcmpl-kQFsJWnvaMyPYcKhTwTPK348Z9VFswDz","model":"smollm2-Q4_K_M","system_fingerprint":"b6097-9515c613","object":"chat.completion.chunk"}

data: {"choices":[{"finish_reason":null,"index":0,"delta":{"content":" the"}}],"created":1755304024,"id":"chatcmpl-kQFsJWnvaMyPYcKhTwTPK348Z9VFswDz","model":"smollm2-Q4_K_M","system_fingerprint":"b6097-9515c613","object":"chat.completion.chunk"}

data: {"choices":[{"finish_reason":"length","index":0,"delta":{}}],"created":1755304024,"id":"chatcmpl-kQFsJWnvaMyPYcKhTwTPK348Z9VFswDz","model":"smollm2-Q4_K_M","system_fingerprint":"b6097-9515c613","object":"chat.completion.chunk","usage":{"completion_tokens":25,"prompt_tokens":12,"total_tokens":37},"timings":{"prompt_n":12,"prompt_ms":12.462,"prompt_per_token_ms":1.0385,"prompt_per_second":962.9272989889264,"predicted_n":25,"predicted_ms":74.373,"predicted_per_token_ms":2.97492,"predicted_per_second":336.14349293426375}}

data: [DONE]
```

---

👤 **firecoperana** started a conversation on `examples/server/server.cpp` on **2025-08-16** at **14:09:40**

Most information in "usage" is already available in "timings". Can you just add new values in "timings"?

> 👤 **dpk-it** replied on **2025-08-16** at **14:29:11**
> 
> In this case I added "usage" to this endpoint for compatibility with ui that expects "usage" and "timings" in the response  (in my case for example it automagically made "llama-swap" start showing metrics for that endpoint too, but maybe "llama-swap" should take into account different formats when applied specifically to this endpoint). but since this is not an oai compatible endpoint, it is not necessarily needed in this form.

---

👤 **firecoperana** started a conversation on `examples/server/server.cpp` on **2025-08-16** at **14:11:40**

"timings" is also added here.

> 👤 **dpk-it** replied on **2025-08-16** at **14:29:46**
> 
> `usage` is part of the openai api response, it should be sent if `finish_reason` is not empty, no `usage` is a bug. many applications use `usage` to understand what is happening with the context window, for example, compaction needs to be performed, or simply shows how many tokens were processed and generated

---

👤 **firecoperana** started a conversation on `examples/server/server.cpp` on **2025-08-16** at **16:56:02**

I think you just reuse t_token_generation for predicted_ms. There is no reason to calculate time in get_timings(). In practice, do you see large difference between what you have and the current method?

> 👤 **dpk-it** replied on **2025-08-16** at **17:07:22**
> 
> `t_token_generation` calculated in `release()`, but `timings` are now returned for each event that is sent when streaming is enabled

> 👤 **firecoperana** replied on **2025-08-16** at **18:10:21**
> 
> You are right. t_token_generation was wrong the whole time, but I still prefer the approach to calculate the time in update_slots() and use t_token_generation in get_timings():
> const int64_t t_current = ggml_time_us();
> if (slot.n_decoded == 1) {
>     slot.t_start_generation = t_current;
>     slot.t_prompt_processing = (slot.t_start_generation - slot.t_start_process_prompt) / 1e3;
>     metrics.on_prompt_eval(slot);
> }
> slot.t_token_generation = (t_current - slot.t_start_generation) / 1e3;

> 👤 **dpk-it** replied on **2025-08-16** at **22:12:29**
> 
> Updated PR, did it a little differently

---

👤 **firecoperana** requested changes on this pull request 🔄 on **2025-08-17** at **01:30:28**

Looks good to me. Just delete the lines about server_task_result_dict.

---

👤 **firecoperana** approved this pull request ✅ on **2025-08-17** at **02:22:32**