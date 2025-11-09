## 🔀 [Pull Request #723](https://github.com/ikawrakow/ik_llama.cpp/pull/723) - Tool calls support from mainline

| **Author** | `firecoperana` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `fcp/tool_call_support` |
| **Target Branch** | `main` |
| **Created** | 2025-08-23 |
| **Updated** | 2025-10-26 |
| **Merged** | 2025-09-01 |
| **Assignees** | `firecoperana` |

---

## 📄 Description

This is kind of a startover of the existing tool calls support, which supports wider range of the models. The reason for this PR is that each model tends to need different templates for tool calls and the implementation is very easy to have bugs. This PR uses mainline's approach to handle tools calls, which makes it easy to support new models and fix existing/future bugs. It includes mainline's tool call related PRs up to 8/22/2205.
It's not thoroughly tested but in my simple tests, results look fine. If there are bugs, confirm if it exists in mainline first.  

Changes to API:
Chat completions API result should match latest mainline.
/completions and /v1/completions will use openAI type completions.

The important mainline's PRs are:
1. Tool call support (generic + native for Llama, Functionary, Hermes, Mistral, Firefunction, DeepSeek) w/ lazy grammars https://github.com/ggml-org/llama.cpp/pull/9639
2. `server`: streaming of tool calls and thoughts when `--jinja` is on [([#12379](https://github.com/ikawrakow/ik_llama.cpp/issues/12379))](https://github.com/ggml-org/llama.cpp/pull/12379)
3. server: add --reasoning-budget 0 to disable thinking (incl. qwen3 w/ enable_thinking:false) https://github.com/ggml-org/llama.cpp/pull/13771
4. server : support jinja extra template kwargs (Qwen3 enable_thinking feature), from command line and from client 
https://github.com/ggml-org/llama.cpp/pull/13196
5. model : gpt-oss add response_format support https://github.com/ggml-org/llama.cpp/pull/15494

- [x] I have read the [contributing guidelines](https://github.com/ggerganov/llama.cpp/blob/master/CONTRIBUTING.md)
- Self-reported review complexity:
  - [ ] Low
  - [ ] Medium
  - [x] High

---

## 💬 Conversation

👤 **saood06** started a conversation on `examples/server/server.cpp` on **2025-08-24** at **04:40:54**

Why is `"/completions"` being changed to call the OAI compatible method `handle_completions_oai`?

I just checked mainline and isn't done there, is there any reason for this change as I don't really think it makes sense.

Also on a related topic, the `handle_completions`, I haven't gone through all of the code in this PR or tested the PR yet but based on my skimming of it so far, the code looks like it might pull in a change which on mainline ended up changing default behavior for the non-OAI endpoint in a way that I don't really agree with because it changes things for downstream apps that result in worse performance due to different outputs

See this comment: https://github.com/ggml-org/llama.cpp/pull/10783#issuecomment-2544053116

You can see the downstream impacts it has due to:

https://github.com/lmg-anon/mikupad/issues/104 and https://github.com/lmg-anon/mikupad/issues/110

If this does make that change (feel free to tell me if it doesn't) I do feel like "post_sampling_probs" should default true, this way downstream applications can opt in to the new behavior but are not impacted otherwise (and mikupad isn't the only downstream app affected, but it was the only one I know impacted users noticed and said something). 

Sorry if what I said is not relevant (I wanted to comment this now, since testing may take a while and not sure if I can do it tonight)

Edit: Just saw the tests in [tools/server/tests/unit/test_completion.py](https://github.com/ikawrakow/ik_llama.cpp/pull/723/files#diff-ea703c43b54a0589cdfe7a217b6ff79f671a2b2a914054659c952201f10ef105) and if this test case is true, then it confirms what I was asking about above. 

Being able to have access to the logprobs if wanted via the non OAI endpoint is nice, but I don't think introducing that functionality should change the default behavior.

I know it is complicated in that mainline made this change so there have been two separate expectations. They also changed the output stream at the same time, which made the break obvious for affected users as it changed what you received when you also asked for `n_probs` on that endpoint. Not only that but it put a performance cost on an API parameter (`n_probs`) that did not have one before as the default (it could be removed if you pass `post_sampling_probs` as true, but that is a parameter that did not exist before).

> 👤 **firecoperana** replied on **2025-08-24** at **12:16:24**
> 
> I will change it back. Didn't realize it will break things.

---

👤 **ubergarm** commented on **2025-08-25** at **15:28:56**

Thanks for looking into this, I'm not exactly sure how to use it, but seems we need something like this to support enable/disable thinking for DeepSeek-V3.1 given how it handles that in the chat template: https://huggingface.co/deepseek-ai/DeepSeek-V3.1#chat-template

So as I understand it, `/v1/*` will use the newer OAI style endpoints, and the legacy stuff will remain at non `/v1/` API endpoints?

One thing I did test which works on mainline but was not working here is this part:

> server : support jinja extra template kwargs (Qwen3 enable_thinking feature), from command line and from client

It seems like `--chat-template-kwargs` is supported on mainline, but not this PR e.g.:
```bash
    --jinja \
    --chat-template-kwargs '{"thinking": true}' \
```

That said, I still don't know how to get mainline to <think> haha, but that is for me to figure out a proper OAI client "prompt" or whatever data it wants.

Anyway, I'd love to see a working example of how to enable thinking for DeepSeek-V3.1 with this PR.

---

👤 **firecoperana** commented on **2025-08-25** at **15:58:49**

You are correct about api usage. I forgot to add  --chat-template-kwargs for the command line args. Need to add it. Maybe try  --reasoning-budget 0 and see if it works. This disables thinking.

---

👤 **firecoperana** commented on **2025-08-26** at **01:34:15**

--chat-template-kwargs is added

---

👤 **saood06** started a conversation on `tools/server/tests/unit/test_completion.py` on **2025-08-26** at **03:12:50**

This is the specific test I was referring to before.

Assuming this test is accurate and passes this shows that the new default behavior is now to return `top_logprobs` which not only is different, but also as mentioned in my other comment based on what people have said comes at a performance penalty.

I just tested the current (without this PR) default behavior for `n_probs` (both in and outside of a stream).

It returned `probs` which is different from `top_probs` (which is what another test case says it will now return if you pass `post_sampling_probs` to be true), or `top_logprobs` (which is what this test case shows is the new default).

I don't see a benefit to this breaking change, `n_probs` returning `probs` by default makes sense to me, now allowing it to return logprobs by sending `post_sampling_probs` with false is also nice and is also not a breaking change.

> 👤 **firecoperana** replied on **2025-08-26** at **13:54:14**
> 
> Still trying to understand your issue here. Does this logprob issue occur when you use /v1/completions which has been changed to open ai compatible completions, or /completion and /completions. I actually removed probabilities in old completion end point. I can add them back if that is what you want. 
> 
> For open ai compatible completions, mainline defaults to **false** for post_sampling_probs (can be set by client), but it is **undefined** here. If I default it to true and allow it to be set by client, will it solve the performance issue with logprob?

> 👤 **saood06** replied on **2025-08-26** at **19:49:56**
> 
> >I actually removed probabilities in old completion end point. I can add them back if that is what you want.
> 
> ~I would appreciate that.~
> 
> Edit: No you didn't. I actually tested it things work. That does mean this test file is wrong, so either should be updated or removed.
> 
> >For open ai compatible completions, mainline defaults to false for post_sampling_probs (can be set by client), but it is undefined here. If I default it to true and allow it to be set by client, will it solve the performance issue with logprob?
> 
> Following whatever the openAI spec is for those endpoints makes most sense (as they are being provided as OAI compatible), but I was referring to the two non OAI endpoints to not make breaking changes there which this test says changed.
> 
> >Still trying to understand your issue here. 
> 
> This test file is wrong, so either should be updated or removed.

---

👤 **dpk-it** commented on **2025-08-26** at **17:28:54**

It looks like this PR is breaking usage/timings again. but I'm not sure, needs to be tested

---

👤 **ikawrakow** commented on **2025-08-27** at **06:26:15**

What is the status here? Are all disagreements resolved?

---

👤 **saood06** commented on **2025-08-27** at **07:06:37**

> What is the status here? Are all disagreements resolved?

The test files included in this PR would not pass when run (in this case it is the test that is wrong as this branch handles things the old way still). It does look like there may be a regression in usage/timings in some places.

I tested more and found another regression with token streaming off as with this PR the output does not contain any of the predicted tokens and also it is missing timings and usage. (I checked the KV cache with the new endpoint and was able to see it generated, just not output).

---

👤 **firecoperana** commented on **2025-08-28** at **22:11:19**

The new commit should fix all of the regression. I didn't look at the code in the tests.

---

👤 **dpk-it** started a conversation on `examples/server/server.cpp` on **2025-08-28** at **22:55:59**

Why did you remove "generated_text" from the final response? and "oaicompat_msg_diffs"... from `/v1/completions` endpoint? Some people may already be using this in their applications... In addition "usage" allows you to display statistics in "llama-swap" for this endpoint.

> 👤 **dpk-it** replied on **2025-08-28** at **23:12:30**
> 
> I meant old `/v1/completions`, which is now available at `/completions`

> 👤 **firecoperana** replied on **2025-08-29** at **01:06:14**
> 
> Usage is already added unless you need it for /completions.  generated_text is added back. oaicompat_msg_diffs is added in prior tool call PR, so should be fine. Just in case, we can add a message stating breaking changes and let people change their API when needed, or fix any regression in future PRs.

---

👤 **saood06** commented on **2025-08-29** at **01:30:24**

> I didn't look at the code in the tests.

I haven't tested the code you just pushed in yet, but I see the tests are still in the PR. Merging in tests that do not reflect what is being merged in is just confusing. Can you either remove (or update the tests)?

---

👤 **firecoperana** commented on **2025-08-29** at **13:35:29**

They are removed.

---

👤 **ultoris** commented on **2025-08-29** at **20:13:42**

Regarding GPT-OSS response format support, when using the main branch or this PR the thinking part is not parsed (the <|channel|>analysis<|message|> part is shown as is in clients like openwebui or aider instead of as a thinking block). This does not happen in llama.cpp where the thinking block appears as expected.
I've used the same model and the raw model output seems to be identical in both cases, looks like there is something missing when parsing the harmony format response.

---

👤 **firecoperana** commented on **2025-08-29** at **20:30:26**

> Regarding GPT-OSS response format support, when using the main branch or this PR the thinking part is not parsed (the <|channel|>analysis<|message|> part is shown as is in clients like openwebui or aider instead of as a thinking block). This does not happen in llama.cpp where the thinking block appears as expected. I've used the same model and the raw model output seems to be identical in both cases, looks like there is something missing when parsing the harmony format response.

Do you use --jinja flag? In my test, jinja should make thought process in thinking block.

---

👤 **ultoris** commented on **2025-08-29** at **21:30:13**

I'm using version: 
3866 (5877b06b)
built with cc (Ubuntu 11.4.0-1ubuntu1~22.04.2) 11.4.0 for x86_64-linux-gnu

I am using the --jinja flag, the command is:.
./build/bin/llama-server --jinja  -fa -m models/gpt-oss-20b-mxfp4.gguf -t 10 -c 65535 --temp 1 --top_k 0  --top_p 1 --min_p 0 --port 8080 -fmoe

The same command on main raises the "template not yet supported" warning and falls back to ChatML producing incorrect responses. Model is the one from ggml-org (SHA256=be37a636aca0fc1aae0d32325f82f6b4d21495f06823b5fbc1898ae0303e9935).

```
chat_template="{#-\n  In addition to the normal inputs of `messages` and `tools`, this template also accepts the\n  following kwargs:\n  - \"builtin_tools\": A list, can contain \"browser\" and/or \"python\".\n  - \"model_identity\": A string that optionally describes the model identity.\n  - \"reasoning_effort\": A string that describes the reasoning effort, defaults to \"medium\".\n #}\n\n{#- Tool Definition Rendering ============================================== #}\n{%- macro render_typescript_type(param_spec, required_params, is_nullable=false) -%}\n    {%- if param_spec.type == \"array\" -%}\n        {%- if param_spec['items'] -%}\n            {%- if param_spec['items']['type'] == \"string\" -%}\n                {{- \"string[]\" }}\n            {%- elif param_spec['items']['type'] == \"number\" -%}\n                {{- \"number[]\" }}\n            {%- elif param_spec['items']['type'] == \"integer\" -%}\n                {{- \"number[]\" }}\n            {%- elif param_spec['items']['type'] == \"boolean\" -%}\n                {{- \"boolean[]\" }}\n            {%- else -%}\n                {%- set inner_type = render_typescript_type(param_spec['items'], required_params) -%}\n                {%- if inner_type == \"object | object\" or inner_type|length > 50 -%}\n                    {{- \"any[]\" }}\n                {%- else -%}\n                    {{- inner_type + \"[]\" }}\n                {%- endif -%}\n            {%- endif -%}\n            {%- if param_spec.nullable -%}\n                {{- \" | null\" }}\n            {%- endif -%}\n        {%- else -%}\n            {{- \"any[]\" }}\n            {%- if param_spec.nullable -%}\n                {{- \" | null\" }}\n            {%- endif -%}\n        {%- endif -%}\n    {%- elif param_spec.type is defined and param_spec.type is iterable and param_spec.type is not string and param_spec.type is not mapping and param_spec.type[0] is defined -%}\n        {#- Handle array of types like [\"object\", \"object\"] from Union[dict, list] #}\n        {%- if param_spec.type | length > 1 -%}\n            {{- param_spec.type | join(\" | \") }}\n        {%- else -%}\n            {{- param_spec.type[0] }}\n        {%- endif -%}\n    {%- elif param_spec.oneOf -%}\n        {#- Handle oneOf schemas - check for complex unions and fallback to any #}\n        {%- set has_object_variants = false -%}\n        {%- for variant in param_spec.oneOf -%}\n            {%- if variant.type == \"object\" -%}\n                {%- set has_object_variants = true -%}\n            {%- endif -%}\n        {%- endfor -%}\n        {%- if has_object_variants and param_spec.oneOf|length > 1 -%}\n            {{- \"any\" }}\n        {%- else -%}\n            {%- for variant in param_spec.oneOf -%}\n                {{- render_typescript_type(variant, required_params) -}}\n                {%- if variant.description %}\n                    {{- \"// \" + variant.description }}\n                {%- endif -%}\n                {%- if variant.default is defined %}\n                    {{ \"// default: \" + variant.default|tojson }}\n                {%- endif -%}\n                {%- if not loop.last %}\n                    {{- \" | \" }}\n                {% endif -%}\n            {%- endfor -%}\n        {%- endif -%}\n    {%- elif param_spec.type == \"string\" -%}\n        {%- if param_spec.enum -%}\n            {{- '\"' + param_spec.enum|join('\" | \"') + '\"' -}}\n        {%- else -%}\n            {{- \"string\" }}\n            {%- if param_spec.nullable %}\n                {{- \" | null\" }}\n            {%- endif -%}\n        {%- endif -%}\n    {%- elif param_spec.type == \"number\" -%}\n        {{- \"number\" }}\n    {%- elif param_spec.type == \"integer\" -%}\n        {{- \"number\" }}\n    {%- elif param_spec.type == \"boolean\" -%}\n        {{- \"boolean\" }}\n\n    {%- elif param_spec.type == \"object\" -%}\n        {%- if param_spec.properties -%}\n            {{- \"{\\n\" }}\n            {%- for prop_name, prop_spec in param_spec.properties.items() -%}\n                {{- prop_name -}}\n                {%- if prop_name not in (param_spec.required or []) -%}\n                    {{- \"?\" }}\n                {%- endif -%}\n                {{- \": \" }}\n                {{ render_typescript_type(prop_spec, param_spec.required or []) }}\n                {%- if not loop.last -%}\n                    {{-\", \" }}\n                {%- endif -%}\n            {%- endfor -%}\n            {{- \"}\" }}\n        {%- else -%}\n            {{- \"object\" }}\n        {%- endif -%}\n    {%- else -%}\n        {{- \"any\" }}\n    {%- endif -%}\n{%- endmacro -%}\n\n{%- macro render_tool_namespace(namespace_name, tools) -%}\n    {{- \"## \" + namespace_name + \"\\n\\n\" }}\n    {{- \"namespace \" + namespace_name + \" {\\n\\n\" }}\n    {%- for tool in tools %}\n        {%- set tool = tool.function %}\n        {{- \"// \" + tool.description + \"\\n\" }}\n        {{- \"type \"+ tool.name + \" = \" }}\n        {%- if tool.parameters and tool.parameters.properties %}\n            {{- \"(_: {\\n\" }}\n            {%- for param_name, param_spec in tool.parameters.properties.items() %}\n                {%- if param_spec.description %}\n                    {{- \"// \" + param_spec.description + \"\\n\" }}\n                {%- endif %}\n                {{- param_name }}\n                {%- if param_name not in (tool.parameters.required or []) -%}\n                    {{- \"?\" }}\n                {%- endif -%}\n                {{- \": \" }}\n                {{- render_typescript_type(param_spec, tool.parameters.required or []) }}\n                {%- if param_spec.default is defined -%}\n                    {%- if param_spec.enum %}\n                        {{- \", // default: \" + param_spec.default }}\n                    {%- elif param_spec.oneOf %}\n                        {{- \"// default: \" + param_spec.default }}\n                    {%- else %}\n                        {{- \", // default: \" + param_spec.default|tojson }}\n                    {%- endif -%}\n                {%- endif -%}\n                {%- if not loop.last %}\n                    {{- \",\\n\" }}\n                {%- else %}\n                    {{- \",\\n\" }}\n                {%- endif -%}\n            {%- endfor %}\n            {{- \"}) => any;\\n\\n\" }}\n        {%- else -%}\n            {{- \"() => any;\\n\\n\" }}\n        {%- endif -%}\n    {%- endfor %}\n    {{- \"} // namespace \" + namespace_name }}\n{%- endmacro -%}\n\n{%- macro render_builtin_tools(browser_tool, python_tool) -%}\n    {%- if browser_tool %}\n        {{- \"## browser\\n\\n\" }}\n        {{- \"// Tool for browsing.\\n\" }}\n        {{- \"// The `cursor` appears in brackets before each browsing display: `[{cursor}]`.\\n\" }}\n        {{- \"// Cite information from the tool using the following format:\\n\" }}\n        {{- \"// `【{cursor}†L{line_start}(-L{line_end})?】`, for example: `【6†L9-L11】` or `【8†L3】`.\\n\" }}\n        {{- \"// Do not quote more than 10 words directly from the tool output.\\n\" }}\n        {{- \"// sources=web (default: web)\\n\" }}\n        {{- \"namespace browser {\\n\\n\" }}\n        {{- \"// Searches for information related to `query` and displays `topn` results.\\n\" }}\n        {{- \"type search = (_: {\\n\" }}\n        {{- \"query: string,\\n\" }}\n        {{- \"topn?: number, // default: 10\\n\" }}\n        {{- \"source?: string,\\n\" }}\n        {{- \"}) => any;\\n\\n\" }}\n        {{- \"// Opens the link `id` from the page indicated by `cursor` starting at line number `loc`, showing `num_lines` lines.\\n\" }}\n        {{- \"// Valid link ids are displayed with the formatting: `【{id}†.*】`.\\n\" }}\n        {{- \"// If `cursor` is not provided, the most recent page is implied.\\n\" }}\n        {{- \"// If `id` is a string, it is treated as a fully qualified URL associated with `source`.\\n\" }}\n        {{- \"// If `loc` is not provided, the viewport will be positioned at the beginning of the document or centered on the most relevant passage, if available.\\n\" }}\n        {{- \"// Use this function without `id` to scroll to a new location of an opened page.\\n\" }}\n        {{- \"type open = (_: {\\n\" }}\n        {{- \"id?: number | string, // default: -1\\n\" }}\n        {{- \"cursor?: number, // default: -1\\n\" }}\n        {{- \"loc?: number, // default: -1\\n\" }}\n        {{- \"num_lines?: number, // default: -1\\n\" }}\n        {{- \"view_source?: boolean, // default: false\\n\" }}\n        {{- \"source?: string,\\n\" }}\n        {{- \"}) => any;\\n\\n\" }}\n        {{- \"// Finds exact matches of `pattern` in the current page, or the page given by `cursor`.\\n\" }}\n        {{- \"type find = (_: {\\n\" }}\n        {{- \"pattern: string,\\n\" }}\n        {{- \"cursor?: number, // default: -1\\n\" }}\n        {{- \"}) => any;\\n\\n\" }}\n        {{- \"} // namespace browser\\n\\n\" }}\n    {%- endif -%}\n\n    {%- if python_tool %}\n        {{- \"## python\\n\\n\" }}\n        {{- \"Use this tool to execute Python code in your chain of thought. The code will not be shown to the user. This tool should be used for internal reasoning, but not for code that is intended to be visible to the user (e.g. when creating plots, tables, or files).\\n\\n\" }}\n        {{- \"When you send a message containing Python code to python, it will be executed in a stateful Jupyter notebook environment. python will respond with the output of the execution or time out after 120.0 seconds. The drive at '/mnt/data' can be used to save and persist user files. Internet access for this session is UNKNOWN. Depends on the cluster.\\n\\n\" }}\n    {%- endif -%}\n{%- endmacro -%}\n\n{#- System Message Construction ============================================ #}\n{%- macro build_system_message() -%}\n    {%- if model_identity is not defined %}\n        {%- set model_identity = \"You are ChatGPT, a large language model trained by OpenAI.\" %}\n    {%- endif %}\n    {{- model_identity + \"\\n\" }}\n    {{- \"Knowledge cutoff: 2024-06\\n\" }}\n    {{- \"Current date: \" + strftime_now(\"%Y-%m-%d\") + \"\\n\\n\" }}\n    {%- if reasoning_effort is not defined %}\n        {%- set reasoning_effort = \"medium\" %}\n    {%- endif %}\n    {{- \"Reasoning: \" + reasoning_effort + \"\\n\\n\" }}\n    {%- if builtin_tools %}\n        {{- \"# Tools\\n\\n\" }}\n        {%- set available_builtin_tools = namespace(browser=false, python=false) %}\n        {%- for tool in builtin_tools %}\n            {%- if tool == \"browser\" %}\n                {%- set available_builtin_tools.browser = true %}\n            {%- elif tool == \"python\" %}\n                {%- set available_builtin_tools.python = true %}\n            {%- endif %}\n        {%- endfor %}\n        {{- render_builtin_tools(available_builtin_tools.browser, available_builtin_tools.python) }}\n    {%- endif -%}\n    {{- \"# Valid channels: analysis, commentary, final. Channel must be included for every message.\" }}\n    {%- if tools -%}\n        {{- \"\\nCalls to these tools must go to the commentary channel: 'functions'.\" }}\n    {%- endif -%}\n{%- endmacro -%}\n\n{#- Main Template Logic ================================================= #}\n{#- Set defaults #}\n\n{#- Render system message #}\n{{- \"<|start|>system<|message|>\" }}\n{{- build_system_message() }}\n{{- \"<|end|>\" }}\n\n{#- Extract developer message #}\n{%- if messages[0].role == \"developer\" or messages[0].role == \"system\" %}\n    {%- set developer_message = messages[0].content %}\n    {%- set loop_messages = messages[1:] %}\n{%- else %}\n    {%- set developer_message = \"\" %}\n    {%- set loop_messages = messages %}\n{%- endif %}\n\n{#- Render developer message #}\n{%- if developer_message or tools %}\n    {{- \"<|start|>developer<|message|>\" }}\n    {%- if developer_message %}\n        {{- \"# Instructions\\n\\n\" }}\n        {{- developer_message }}\n        {{- \"\\n\\n\" }}\n    {%- endif %}\n    {%- if tools -%}\n        {{- \"# Tools\\n\\n\" }}\n        {{- render_tool_namespace(\"functions\", tools) }}\n    {%- endif -%}\n    {{- \"<|end|>\" }}\n{%- endif %}\n\n{#- Render messages #}\n{%- set last_tool_call = namespace(name=none) %}\n{%- for message in loop_messages -%}\n    {#- At this point only assistant/user/tool messages should remain #}\n    {%- if message.role == 'assistant' -%}\n        {#- Checks to ensure the messages are being passed in the format we expect #}\n        {%- if \"content\" in message %}\n            {%- if false %}\n                {{- raise_exception(\"You have passed a message containing <|channel|> tags in the content field. Instead of doing this, you should pass analysis messages (the string between '<|message|>' and '<|end|>') in the 'thinking' field, and final messages (the string between '<|message|>' and '<|end|>') in the 'content' field.\") }}\n            {%- endif %}\n        {%- endif %}\n        {%- if \"thinking\" in message %}\n            {%- if \"<|channel|>analysis<|message|>\" in message.thinking or \"<|channel|>final<|message|>\" in message.thinking %}\n                {{- raise_exception(\"You have passed a message containing <|channel|> tags in the thinking field. Instead of doing this, you should pass analysis messages (the string between '<|message|>' and '<|end|>') in the 'thinking' field, and final messages (the string between '<|message|>' and '<|end|>') in the 'content' field.\") }}\n            {%- endif %}\n        {%- endif %}\n        {%- if \"tool_calls\" in message %}\n            {#- We need very careful handling here - we want to drop the tool call analysis message if the model #}\n            {#- has output a later <|final|> message, but otherwise we want to retain it. This is the only case #}\n            {#- when we render CoT/analysis messages in inference. #}\n            {%- set future_final_message = namespace(found=false) %}\n            {%- for future_message in loop_messages[loop.index:] %}\n                {%- if future_message.role == 'assistant' and \"tool_calls\" not in future_message %}\n                    {%- set future_final_message.found = true %}\n                {%- endif %}\n            {%- endfor %}\n            {#- We assume max 1 tool call per message, and so we infer the tool call name #}\n            {#- in \"tool\" messages from the most recent assistant tool call name #}\n            {%- set tool_call = message.tool_calls[0] %}\n            {%- if tool_call.function %}\n                {%- set tool_call = tool_call.function %}\n            {%- endif %}\n            {%- if message.content and message.thinking %}\n                {{- raise_exception(\"Cannot pass both content and thinking in an assistant message with tool calls! Put the analysis message in one or the other, but not both.\") }}\n            {%- elif message.content and not future_final_message.found %}\n                {{- \"<|start|>assistant<|channel|>analysis<|message|>\" + message.content + \"<|end|>\" }}\n            {%- elif message.thinking and not future_final_message.found %}\n                {{- \"<|start|>assistant<|channel|>analysis<|message|>\" + message.thinking + \"<|end|>\" }}\n            {%- endif %}\n            {{- \"<|start|>assistant to=\" }}\n            {{- \"functions.\" + tool_call.name + \"<|channel|>commentary \" }}\n            {{- (tool_call.content_type if tool_call.content_type is defined else \"json\") + \"<|message|>\" }}\n            {{- tool_call.arguments|tojson }}\n            {{- \"<|call|>\" }}\n            {%- set last_tool_call.name = tool_call.name %}\n        {%- elif loop.last and not add_generation_prompt %}\n            {#- Only render the CoT if the final turn is an assistant turn and add_generation_prompt is false #}\n            {#- This is a situation that should only occur in training, never in inference. #}\n            {%- if \"thinking\" in message %}\n                {{- \"<|start|>assistant<|channel|>analysis<|message|>\" + message.thinking + \"<|end|>\" }}\n            {%- endif %}\n            {#- <|return|> indicates the end of generation, but <|end|> does not #}\n            {#- <|return|> should never be an input to the model, but we include it as the final token #}\n            {#- when training, so the model learns to emit it. #}\n            {{- \"<|start|>assistant<|channel|>final<|message|>\" + message.content + \"<|return|>\" }}\n        {%- else %}\n            {#- CoT is dropped during all previous turns, so we never render it for inference #}\n            {{- \"<|start|>assistant<|channel|>final<|message|>\" + message.content + \"<|end|>\" }}\n            {%- set last_tool_call.name = none %}\n        {%- endif %}\n    {%- elif message.role == 'tool' -%}\n        {%- if last_tool_call.name is none %}\n            {{- raise_exception(\"Message has tool role, but there was no previous assistant message with a tool call!\") }}\n        {%- endif %}\n        {{- \"<|start|>functions.\" + last_tool_call.name }}\n        {{- \" to=assistant<|channel|>commentary<|message|>\" + message.content|tojson + \"<|end|>\" }}\n    {%- elif message.role == 'user' -%}\n        {{- \"<|start|>user<|message|>\" + message.content + \"<|end|>\" }}\n    {%- endif -%}\n{%- endfor -%}\n\n{#- Generation prompt #}\n{%- if add_generation_prompt -%}\n<|start|>assistant\n{%- endif -%}"
INFO [                    main] chat template | tid="136902271338816" timestamp=1756502871 chat_example="<|start|>system<|message|>You are ChatGPT, a large language model trained by OpenAI.\nKnowledge cutoff: 2024-06\nCurrent date: 2025-08-29\n\nReasoning: medium\n\n# Valid channels: analysis, commentary, final. Channel must be included for every message.<|end|><|start|>developer<|message|># Instructions\n\nYou are a helpful assistant\n\n<|end|><|start|>user<|message|>Hello<|end|><|start|>assistant<|channel|>final<|message|>Hi there<|end|><|start|>user<|message|>How are you?<|end|><|start|>assistant" built_in=true

```

---

👤 **firecoperana** commented on **2025-08-30** at **02:26:55**

@ikawrakow The response from gpt-oss 20b is missing <|end|> under main, which makes reasoning parsing incorrect.
Under main without jinja, I got:
<|channel|>analysis<|message|>We need to respond: "Sure, here's how you can do it: ..." Provide code.<|start|>assistant<|channel|>final<|message|>Sure! In Python, printing a *Hello, World!* message is as simple as using the `print()` function:

but with mainline, I got:
<|channel|>analysis<|message|>We need to respond: "Sure, here's how you can do it: ..." Provide code.**<|end|>**<|start|>assistant<|channel|>final<|message|>Sure! In Python, 

Any idea?

---

👤 **ikawrakow** commented on **2025-08-30** at **05:24:23**

@firecoperana Does [#740](https://github.com/ikawrakow/ik_llama.cpp/issues/740) fix it?

---

👤 **firecoperana** commented on **2025-08-30** at **12:56:55**

> @firecoperana Does [#740](https://github.com/ikawrakow/ik_llama.cpp/issues/740) fix it?

Yes, now it's good. I will include this in my PR.

---

👤 **firecoperana** commented on **2025-08-30** at **13:21:55**

With https://github.com/ikawrakow/ik_llama.cpp/pull/740, tool calls also works properly with GPT-OSS. To move thinking process in separate field like mainline, use `--reasoning-format auto`. The default here is `--reasoning-format none`, which puts reasoning and content both in content field. Built-in webui use none, and it parses thinking process and content itself.

---

👤 **oovloveme** commented on **2025-08-30** at **15:01:47**

Seems \<think\> was missing when running deepseek v3.1 on open-webui in this branch ( have \</think\> at the end of COT but no \<think\> at the beginning)

The model run with
--jinja \
    --chat-template-kwargs '{"thinking": true}' \

---

👤 **firecoperana** commented on **2025-08-30** at **15:07:47**

Can you add --reasoning-format auto and compare the result with mainline? I don't have deepseek v3.1 to test myself.

---

👤 **ChicoPinto70** commented on **2025-08-30** at **15:10:50**

> Seems "" was missing when running deepseek v3.1 on open-webui in this branch ( have "" at the end of COT but no "" at the beginning)
> 
> The model run with --jinja --chat-template-kwargs '{"thinking": true}' \

I'm having this problem, too. I can turn on reasoning in DS v3.1 with --jinja --chat-template-kwargs '{"thinking": true}'  parameters, but the tag \<think\> doesn't appears in the beginning of the COT (just the \</think\> at the end).  

I'm using ubergarm/DeepSeek-V3.1-smol-IQ4_KSS model.

---

👤 **firecoperana** commented on **2025-08-30** at **15:34:15**

> > Seems "" was missing when running deepseek v3.1 on open-webui in this branch ( have "" at the end of COT but no "" at the beginning)
> > The model run with --jinja --chat-template-kwargs '{"thinking": true}' \
> 
> I'm having this problem, too. I can turn on reasoning in DS v3.1 with --jinja --chat-template-kwargs '{"thinking": true}' parameters, but the tag <think> doesn't appears in the beginning of the COT (just the </think> at the end).
> 
> I'm using ubergarm/DeepSeek-V3.1-smol-IQ4_KSS model.

Found this. https://github.com/ggml-org/llama.cpp/issues/15496
Wait for the fix to be merged in mainline

---

👤 **ChicoPinto70** commented on **2025-08-30** at **15:46:47**

> > > Seems "" was missing when running deepseek v3.1 on open-webui in this branch ( have "" at the end of COT but no "" at the beginning)
> > > The model run with --jinja --chat-template-kwargs '{"thinking": true}' \
> > 
> > 
> > I'm having this problem, too. I can turn on reasoning in DS v3.1 with --jinja --chat-template-kwargs '{"thinking": true}' parameters, but the tag  doesn't appears in the beginning of the COT (just the  at the end).
> > I'm using ubergarm/DeepSeek-V3.1-smol-IQ4_KSS model.
> 
> Found this. [ggml-org/llama.cpp[#15496](https://github.com/ikawrakow/ik_llama.cpp/issues/15496)](https://github.com/ggml-org/llama.cpp/issues/15496) Wait for the fix to be merged in mainline

OK! I've just tried to run the mainline (it was a long time that I don't use it.) but it doesn't support this ubergarm model. :+1:

---

👤 **ikawrakow** approved this pull request ✅ on **2025-08-31** at **07:22:05**

If there are no further objections I'll merge this tomorrow morning (my time).

---

👤 **saood06** commented on **2025-09-01** at **05:04:11**

> If there are no further objections I'll merge this tomorrow morning (my time).

I don't see any regressions anymore so no objection from me.

---

👤 **arichiardi** commented on **2025-09-05** at **23:33:21**

Haven't checked if included here but today this fix was merged into mainline (fixes a bug with enable_thinking kwarg logic)

https://github.com/ggml-org/llama.cpp/pull/15404

---

👤 **firecoperana** commented on **2025-09-08** at **20:15:48**

> Haven't checked if included here but today this fix was merged into mainline (fixes a bug with enable_thinking kwarg logic)
> 
> [ggml-org/llama.cpp[#15404](https://github.com/ikawrakow/ik_llama.cpp/issues/15404)](https://github.com/ggml-org/llama.cpp/pull/15404)

Deepseek v3.1 tool calling just merged in mainline. Will include the above PR when porting it.

---

👤 **firecoperana** commented on **2025-09-09** at **23:58:25**

[PR with Deepseek v3.1 tool calling](https://github.com/ikawrakow/ik_llama.cpp/pull/771)

@arichiardi Thinking model disabled assistant prefill is included in the above PR.