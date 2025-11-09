## 🔀 [Pull Request #787](https://github.com/ikawrakow/ik_llama.cpp/pull/787) - Fix logprobs

| **Author** | `sayap` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `logprobs` |
| **Target Branch** | `main` |
| **Created** | 2025-09-22 |
| **Updated** | 2025-09-25 |
| **Merged** | 2025-09-25 |

---

## 📄 Description

This commit is mostly a cherry-pick of ggml-org/llama.cpp[#10783](https://github.com/ikawrakow/ik_llama.cpp/issues/10783).

That PR and friends were partially cherry-picked by [#723](https://github.com/ikawrakow/ik_llama.cpp/issues/723), but wasn't really in a working state yet.

A couple of additional changes:
* Include timing information in response, which was (unintentionally?) done in mainline since ggml-org/llama.cpp[#10643](https://github.com/ikawrakow/ik_llama.cpp/issues/10643).
* Also return the actual logprobs for accepted draft tokens. This is still a TODO in mainline [1].

Note that there is a TG performance penalty to return the logprobs. Here are some numbers I got with Qwen2.5-Coder-32B-Instruct:
* no draft, no logprobs: 12.81 tok/s
* no draft, with logprobs: 12.02 tok/s (6.2% drop)
* with draft, no logprobs: 36.59 tok/s
* with draft, with logprobs: 29.08 tok/s (20.5% drop)

[1] https://github.com/ggml-org/llama.cpp/blob/b6548/tools/server/server.cpp#L4019



- [x] I have read the [contributing guidelines](https://github.com/ggerganov/llama.cpp/blob/master/CONTRIBUTING.md)
- Self-reported review complexity:
  - [ ] Low
  - [x] Medium
  - [ ] High

---

## 💬 Conversation

👤 **sayap** commented on **2025-09-22** at **15:58:47**

Previously, I had a more straight-forward but partial cherry-pick of ggml-org/llama.cpp[#10783](https://github.com/ikawrakow/ik_llama.cpp/issues/10783)  at sayap/ik_llama.cpp@a8e39f6e, but that can't be applied anymore after [#723](https://github.com/ikawrakow/ik_llama.cpp/issues/723).

---

👤 **ikawrakow** started a conversation on `examples/server/utils.hpp` on **2025-09-23** at **07:09:43**

Do these need to be sorted?

> 👤 **sayap** replied on **2025-09-23** at **10:30:44**
> 
> I think so, the caller expects the vector to be sorted when returning the top n_probs tokens.

> 👤 **ikawrakow** replied on **2025-09-23** at **10:46:10**
> 
> But is this just to be able to display the logprobs in the UI? If so, wouldn't it be better to have a command line argument to enable/disable this relatively expensive operation to avoid the performance penalty?

> 👤 **sayap** replied on **2025-09-23** at **11:38:42**
> 
> This is controlled by the per-request parameter `logprobs` and `top_logprobs`. For example, I will use:
> ```
> "top_k": 1,
> "logprobs": true,
> "top_logprobs": 3,
> ```
> to choose the most probable token, and also to retrieve the top 3 logprobs for comparison, e.g. to make sure that the prompt is clear enough such that the most probable token has >= 80% probability.
> 
> When I don't need logprobs, then I will just go with the default of `"logprobs": false` and `"top_logprobs": 0`. No performance penalty when n_probs is 0.

> 👤 **ikawrakow** replied on **2025-09-23** at **12:20:18**
> 
> In that case, and especially if `top_logprobs` is typically much smaller than the vocabulary size, one could speed it up by using partial sort. E.g.
> 
> ```c++
> static std::vector<llama_token_data> get_token_probabilities(llama_context * ctx, int idx, int n_sorted) {
>     const auto * logits = llama_get_logits_ith(ctx, idx);
>     const int n_vocab = llama_n_vocab(llama_get_model(ctx));
>     n_sorted = std::min(n_sorted, n_vocab);
>                                  
>     std::vector<std::pair<float, llama_token>> sorted(n_vocab);
>     for (llama_token token_id = 0; token_id < n_vocab; token_id++) sorted[token_id] = {logits[token_id], token_id};
>                    
>     std::partial_sort(sorted.begin(), sorted.begin() + n_sorted, sorted.end(), std::greater<std::pair<float,llama_token>>{});
>                    
>     float max_l = sorted.front().first;
>     float cum_sum = 0.0f;
>     std::vector<llama_token_data> cur(n_sorted);
>     for (int i = 0; i < n_sorted; ++i) {
>         float p = expf(sorted[i].first - max_l);
>         cum_sum += p;
>         cur[i] = {sorted[i].second, sorted[i].first, p};
>     }              
>     for (int i = n_sorted; i < n_vocab; ++i) cum_sum += expf(sorted[i].first - max_l);
>                    
>     float inv_cum_sum = 1/cum_sum;
>     for (int i = 0; i < n_sorted; ++i) cur[i].p *= inv_cum_sum;
>                    
>     return cur;    
> }                  
> ```

> 👤 **sayap** replied on **2025-09-23** at **12:36:23**
> 
> Ah very nice. Let me try that

> 👤 **sayap** replied on **2025-09-24** at **01:33:37**
> 
> Updated the commit to do partial sort. The performance penalty is much smaller now :tada: 
> 
> partial sort:
> * no draft, no logprobs: 12.87 tok/s
> * no draft, with top 3 logprobs: 12.61 tok/s (2.0% drop)
> * with draft, no logprobs: 36.74 tok/s
> * with draft, with top 3 logprobs: 36.12 tok/s (1.7% drop)
>    
> full sort:
> * no draft, no logprobs: 12.81 tok/s
> * no draft, with top 3 logprobs: 12.02 tok/s (6.2% drop)
> * with draft, no logprobs: 36.59 tok/s
> * with draft, with top 3 logprobs: 29.08 tok/s (20.5% drop)

---

👤 **firecoperana** started a conversation on `examples/server/server.cpp` on **2025-09-23** at **12:25:38**

Don't delete anything for `res.data` in send partial and final response. It will break /completions and /completion endpoint.

> 👤 **sayap** replied on **2025-09-23** at **12:38:10**
> 
> I see. Indeed I didn't test those endpoints. Let me check how is it handled in mainline..

> 👤 **saood06** replied on **2025-09-23** at **12:56:20**
> 
> > I see. Indeed I didn't test those endpoints. Let me check how is it handled in mainline..
> 
> Do not follow mainline when it comes to logprobs and those endpoints. They needlessly broke things by changing the output stream when `n_probs` is requested on those endpoints. Not only that but it put a performance cost on an API parameter (n_probs) that did not have one before as the default (requiring the use of passing post_sampling_probs to get rid of the performance penalty, but even then the output stream is still modified `probs` vs `top_probs`).
> 
> Following the OAI spec is a good thing for the v1 endpoints as that is their purpose, but the /completions and /completion are not based on those specs, and there are third party front ends that do make use of `n_probs` on the non OAI endpoints in a non OAI way.

> 👤 **saood06** replied on **2025-09-23** at **16:32:28**
> 
> > I see. Indeed I didn't test those endpoints.
> 
> Also if you are looking for a frontend to test the /completion endpoint instead of just sending network requests over the console, the WebUI added in [#558](https://github.com/ikawrakow/ik_llama.cpp/issues/558) uses it.
> 
> Just a note, the performance numbers in that endpoint are measured different than the other WebUI, because it measures from the user's perspective which accounts for any network/browser latency, and does not use the timings sent from the server. (Both are useful and valid, and I may end up adding an option to display both or just the differential which would be the network/browser overhead). I just thought it should be noted especially if you try to directly compare them.

> 👤 **sayap** replied on **2025-09-24** at **15:54:47**
> 
> I have reverted the `res.data` changes. Quick test with curl looks fine:
> ```
> $ curl -s http://127.0.0.1:8080/completions -d '{"prompt": "hello", "n_predict": 32, "n_probs": 3}' | jq .
> ...
>   "completion_probabilities": [
>     {
>       "content": "/",
>       "probs": [
>         {
>           "tok_str": ",",
>           "prob": 0.19238649308681488
>         },
>         {
>           "tok_str": " ",
>           "prob": 0.13548995554447174
>         },
>         {
>           "tok_str": "\n",
>           "prob": 0.08068522810935974
>         }
>       ]
>     },
> ...
> ```

---

👤 **ikawrakow** commented on **2025-09-24** at **14:44:09**

@saood06 @firecoperana 

Were your concerns addressed?

---

👤 **firecoperana** commented on **2025-09-24** at **16:29:20**

`res.data` is still deleted, which is needed in /completions and /completion endpoints. It needs to be restored to keep the endpoints working.

---

👤 **sayap** commented on **2025-09-24** at **19:24:34**

@firecoperana Please check the latest commit, thanks

---

👤 **firecoperana** commented on **2025-09-24** at **21:56:38**

They look good now, but I had a crash in  `populate_token_probs()`  when using /v1/completions. It happens when post_sampling_probs = False and n_probs>0. It's good when post_sampling_probs = True
Code I test:



```python
async def create_completion(url):

    headers = {
        "Content-Type": "application/json",
    }

    data = {
        "model": "gpt-3.5-t Turbo",
        "prompt": "Earth is a",
        "max_tokens": 20,
        "n_probs": 3,
        "post_sampling_probs": False
    }

    response = requests.post(url, headers=headers, json=data)
    if response.status_code == 200:
        return response.json()
    else:
        print(f"Error: {response.status_code}, {response.text}")
        return None

if __name__=="__main__":
    url = "ip/v1/completions"
    result = asyncio.run(create_completion(url))
    print(json.dumps(result, indent=2))

---

👤 **sayap** commented on **2025-09-24** at **22:07:42**

Can you share more about the crash? It seems to be fine here.

With `post_sampling_probs` set to true:
```
$ curl -s http://127.0.0.1:8080/v1/completions -d '{"prompt": "Earth is a", "max_tokens": 20, "n_probs": 3, "post_sampling_probs": true}' | jq . | pp
{
  "choices": [
    {
      "text": " beautiful, living planet, teeming with life and surrounded by a blanket of air that makes life possible",
      "index": 0,
      "logprobs": {
        "content": [
          {
            "id": 6233,
            "token": " beautiful",
            "bytes": [32,98,101,97,117,116,105,102,117,108],
            "prob": 0.06725937128067017,
            "top_probs": [
              {
                "id": 11580,
                "token": " planet",
                "bytes": [32,112,108,97,110,101,116],
                "prob": 0.4042002260684967
              },
              {
                "id": 4911,
                "token": " unique",
                "bytes": [32,117,110,105,113,117,101],
                "prob": 0.10278917104005814
              },
...
```

With `post_sampling_probs` set to false:
```
$ curl -s http://127.0.0.1:8080/v1/completions -d '{"prompt": "Earth is a", "max_tokens": 20, "n_probs": 3, "post_sampling_probs": false}' | jq . | pp
{
  "choices": [
    {
      "text": " dynamic system with various processes occurring at different scales, from the microscopic to the global. These processes are",
      "index": 0,
      "logprobs": {
        "content": [
          {
            "id": 8741,
            "token": " dynamic",
            "bytes": [32,100,121,110,97,109,105,99],
            "logprob": -3.365861177444458,
            "top_logprobs": [
              {
                "id": 11580,
                "token": " planet",
                "bytes": [32,112,108,97,110,101,116],
                "logprob": -1.5791088342666626
              },
              {
                "id": 4911,
                "token": " unique",
                "bytes": [32,117,110,105,113,117,101],
                "logprob": -2.674492359161377
              },
...
```

---

👤 **firecoperana** commented on **2025-09-24** at **22:25:39**

```
  std::vector<llama_token_data> cur = get_token_probabilities(ctx, idx, n_probs);

  // set probability for sampled token
  for (size_t i = 0; i < n_vocab; i++) {
      // set probability for sampled token
      if (cur[i].id == result.tok) {
          result.prob = cur[i].p;
          break;
      }
  }
```
size of cur[i] is 3, and it got `vector subscript` out of range error. I was using Qwen2.5-7B.i1-Q4_K_S.gguf to test

---

👤 **sayap** commented on **2025-09-24** at **22:49:51**

Ah good catch. It doesn't crash here with GCC, but it is indeed a bug. Let me try to fix it..

---

👤 **sayap** commented on **2025-09-25** at **02:18:38**

The latest commit should have fixed the bug already, please help to verify. Thanks :grin:

---

👤 **firecoperana** commented on **2025-09-25** at **13:29:29**

Looks good now.

---

👤 **ikawrakow** approved this pull request ✅ on **2025-09-25** at **13:43:20**