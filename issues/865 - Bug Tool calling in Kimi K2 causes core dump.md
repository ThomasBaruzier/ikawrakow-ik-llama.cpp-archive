## 📌 [Issue #865](https://github.com/ikawrakow/ik_llama.cpp/issues/865) - Bug: Tool calling in Kimi K2 causes core dump

| **Author** | `whoisjeremylam` |
| :--- | :--- |
| **State** | ✅ **Open** |
| **Created** | 2025-10-25 |
| **Updated** | 2025-11-02 |

---

## 📄 Description

### What happened?

When tool calling is used with Kimi K2, the request causes `ik_llama.cpp` to segfault.

I've tried a couple of different apps (opencode.ai and the K2 Vendor Verifier) that use tool calling with the same result. It is probably the easiest to reproduce using the K2 Vendor Verifier since it is a simple Python script.

Command to start Kimi K2:
```
~/ik_llama.cpp/build/bin/llama-server \
  -t 23 \
  -m /home/ai/models/ubergarm/Kimi-K2-Instruct-0905-GGUF/Kimi-K2-Instruct-0905-IQ2_KS.gguf \
  --alias Kimi-K2 \
  --jinja \
  --host 0.0.0.0 \
  --port 5000 \
  -c 131072 -ctk q8_0 --no-mmap -ngl 999 \
  -ot "blk.(0|1|2|3|4|5).ffn.=CUDA0" \
  -ot "blk.(11|12|13|14|15|16).ffn.=CUDA1" \
  -ot "blk.(21|22).ffn.=CUDA2" \
  -ot "blk.(31|32).ffn.=CUDA3" \
  -ot exps=CPU \
  -mg 0 -ub 4096 -b 4096 -mla 3 -amb 512 \
  --temp 0.6 --min_p 0.01
```

Command for the official [K2 Vendor Verifier](https://github.com/MoonshotAI/K2-Vendor-Verifier/) app:
```
python tool_calls_eval.py samples.jsonl \
    --model kimi-k2-0905 \
    --base-url http://192.168.100.200:5000/v1 \
    --api-key not_used \
    --concurrency 1 \
    --output results.jsonl \
    --summary summary.json
```

### Name and Version
```
$ ~/ik_llama.cpp/build/bin/llama-server --version
version: 3928 (16f30fcf)
built with cc (Ubuntu 11.4.0-1ubuntu1~22.04) 11.4.0 for x86_64-linux-gnu
```

### What operating system are you seeing the problem on?

Linux

### Relevant log output

```shell
Grammar lazy: false
Chat format: Generic
INFO [   launch_slot_with_task] slot is processing task | tid="137164878520320" timestamp=1761389674 id_slot=0 id_task=0
INFO [            update_slots] kv cache rm [p0, end) | tid="137164878520320" timestamp=1761389674 id_slot=0 id_task=0 p0=0
INFO [   launch_slot_with_task] slot is processing task | tid="137164878520320" timestamp=1761389674 id_slot=0 id_task=00=0minate called after throwing an instance of 'std::runtime_error'
  what():  Invalid diff: '{                                                                                             d in the background, so the shell expects another complete command after it. When you type:\n\n\ncurl -XPOST -H \"host: wechat-verify-apisix.yunzhanghu.net\" http://[IP_ADDRESS]:80/weixin-xiaochengxu-apisix-add-route?addPathTxt=test1.txt&addPathUuid=test1111223444411&addDomain=thirdparty3.yunzhanghu.com\n\n\nthe part after the first `&` is interpreted as a new command, which causes the parse error.\n\nFix: wrap the entire URL in quotes so the shell treats it as a single argument:\n\n\ncurl -XPOST -H \"host: wechat-verify-apisix.yunzhanghu.net\" \"http://[IP_ADDRESS]:80/weixin-xiaochengxu-apisix-add-route?addPathTxt=test1.txt&addPathUuid=test1111223444411&addDomain=thirdparty3.yunzhanghu.com\"\n\n\n(Replace `[IP_ADDRESS]` with the actual IP address or hostname.)"
' not found at start of 'The error you're seeing—`zsh: parse error near '&'`—comes from the shell, not from the remote server.

In zsh (and most other POSIX shells) the ampersand `&` is treated as a command separator that puts the preceding command in the background, so the shell expects another complete command after it. When you type:


curl -XPOST -H "host: wechat-verify-apisix.yunzhanghu.net" http://[IP_ADDRESS]:80/weixin-xiaochengxu-apisix-add-route?a
ddPathTxt=                                                                                                             =
test1.txt&addPathUuid=test1111223444411&addDomain=thirdparty3.yunzhanghu.com"


(Replace `[IP_ADDRESS]` with the actual IP address or hostname.)'
Aborted (core dumped)
```

---

## 💬 Conversation

👤 **magikRUKKOLA** commented on **2025-10-25** at **12:25:08**

Over here:

https://github.com/ikawrakow/ik_llama.cpp/issues/750

---

👤 **magikRUKKOLA** commented on **2025-10-25** at **12:26:49**

Try this patch lol:

```diff
diff --git a/common/chat.cpp b/common/chat.cpp
index f384bfa7d..177b65008 100644
--- a/common/chat.cpp
+++ b/common/chat.cpp
@@ -40,7 +40,10 @@ static std::string string_diff(const std::string & last, const std::string & cur
             // and the current ended on a stop word (erased).
             return "";
         }
-        throw std::runtime_error("Invalid diff: '" + last + "' not found at start of '" + current + "'");
+        //throw std::runtime_error("Invalid diff: '" + last + "' not found at start of '" + current + "'");
+        // Log a warning and return the entire current string as a fallback.
+        LOG("Warning: Invalid diff detected. Last: '%s', Current: '%s'. Returning full current string.\n", last.c_str(), current.c_str());
+        return current;
     }
     return current.substr(last.size());
 }
@@ -1446,39 +1449,113 @@ static void common_chat_parse_deepseek_v3_1_content(common_chat_msg_parser & bui
 }
 
 static void common_chat_parse_deepseek_v3_1(common_chat_msg_parser & builder) {
-    // DeepSeek V3.1 outputs reasoning content between "<think>" and "</think>" tags, followed by regular content
-    // First try to parse using the standard reasoning parsing method
     LOG("%s: thinking_forced_open: %s\n", __func__, std::to_string(builder.syntax().thinking_forced_open).c_str());
 
-    auto start_pos = builder.pos();
-    auto found_end_think = builder.try_find_literal("</think>");
-    builder.move_to(start_pos);
-
-    if (builder.syntax().thinking_forced_open && !builder.is_partial() && !found_end_think) {
-        LOG("%s: no end_think, not partial, adding content\n", __func__);
-        common_chat_parse_deepseek_v3_1_content(builder);
-    } else if (builder.try_parse_reasoning("<think>", "</think>")) {
-        // If reasoning was parsed successfully, the remaining content is regular content
-        LOG("%s: parsed reasoning, adding content\n", __func__);
-        // </think><｜tool▁calls▁begin｜><｜tool▁call▁begin｜>function<｜tool▁sep｜>NAME\n```json\nJSON\n```<｜tool▁call▁end｜><｜tool▁calls▁end｜>
-        common_chat_parse_deepseek_v3_1_content(builder);
+    // First, check if we're in a forced thinking scenario with no explicit tags
+    if (builder.syntax().thinking_forced_open && !builder.try_find_literal("</think>")) {
+        LOG("%s: thinking forced open and no </think> found, treating everything as reasoning\n", __func__);
+
+        // If there's an explicit <think> tag, parse up to where it should end
+        if (auto start_res = builder.try_consume_literal("<think>")) {
+            auto rest = builder.consume_rest();
+            builder.add_reasoning_content(rest);
+
+            LOG("%s: added content after <think> as reasoning (thinking forced open)\n", __func__);
+        } else {
+            // No explicit tags at all, treat everything as reasoning
+            auto rest = builder.consume_rest();
+            builder.add_reasoning_content(rest);
+
+            LOG("%s: no explicit tags, added all content as reasoning (thinking forced open)\n", __func__);
+        }
+        return;
+    }
+
+    // Standard parsing flow with explicit <think>/</think> tags
+    if (builder.try_parse_reasoning("<think>", "</think>")) {
+        LOG("%s: parsed reasoning section, now parsing remaining content\n", __func__);
+
+        // Parse tool calls or regular content after reasoning
+        static const common_regex function_regex("(?:<｜tool▁calls▁begin｜>)?([^\n<]+)(?:<｜tool▁sep｜>)");
+        static const common_regex close_regex("(?:[\\s]*)?}<?");
+        static const common_regex tool_calls_begin("(?:<｜tool▁calls▁begin｜>|<｜tool_calls_begin｜>|<｜tool calls begin｜>|<｜tool\\\\_calls\\\\_begin｜>|<｜tool▁calls｜>)");
+        static const common_regex tool_calls_end("}<?");
+
+        if (builder.syntax().parse_tool_calls) {
+            LOG("%s: attempting to parse tool calls\n", __func__);
+
+            parse_json_tool_calls(
+                builder,
+                /* block_open= */ tool_calls_begin,
+                /* function_regex_start_only= */ std::nullopt,
+                function_regex,
+                close_regex,
+                tool_calls_end);
+        } else {
+            LOG("%s: not parsing tool calls, adding as content\n", __func__);
+            builder.add_content(builder.consume_rest());
+        }
     } else {
+        // No reasoning tags found
         if (builder.syntax().reasoning_format == COMMON_REASONING_FORMAT_NONE) {
-          LOG("%s: reasoning_format none, adding content\n", __func__);
-          common_chat_parse_deepseek_v3_1_content(builder);
-          return;
-        }
-        // If no reasoning tags found, check if we should treat everything as reasoning
-        if (builder.syntax().thinking_forced_open) {
-            // If thinking is forced open but no tags found, treat everything as reasoning
-            LOG("%s: thinking_forced_open, adding reasoning content\n", __func__);
-            builder.add_reasoning_content(builder.consume_rest());
+            LOG("%s: reasoning_format none, adding content directly\n", __func__);
+
+            static const common_regex function_regex("(?:<｜tool▁calls▁begin｜>)?([^\n<]+)(?:<｜tool▁sep｜>)");
+            static const common_regex close_regex("(?:[\\s]*)?}<?");
+            static const common_regex tool_calls_begin("(?:<｜tool▁calls▁begin｜>|<｜tool_calls_begin｜>|<｜tool\\\\_calls\\\\_begin｜>|<｜tool▁calls｜>)");
+            static const common_regex tool_calls_end("}<?");
+
+            if (builder.syntax().parse_tool_calls) {
+                LOG("%s: attempting to parse tool calls without reasoning\n", __func__);
+
+                parse_json_tool_calls(
+                    builder,
+                    /* block_open= */ tool_calls_begin,
+                    /* function_regex_start_only= */ std::nullopt,
+                    function_regex,
+                    close_regex,
+                    tool_calls_end);
+            } else {
+                LOG("%s: not parsing tool calls, adding as content\n", __func__);
+                builder.add_content(builder.consume_rest());
+            }
+        } else if (builder.syntax().thinking_forced_open) {
+            // Thinking forced open but no tags found - treat everything as reasoning
+            LOG("%s: thinking_forced_open with no explicit tags, adding all as reasoning\n", __func__);
+
+            auto rest = builder.consume_rest();
+            if (builder.syntax().reasoning_in_content) {
+                builder.add_content(rest);
+            } else {
+                builder.add_reasoning_content(rest);
+            }
         } else {
-            LOG("%s: no thinking_forced_open, adding content\n", __func__);
-            // <｜tool▁call▁begin｜>NAME<｜tool▁sep｜>JSON<｜tool▁call▁end｜>
-            common_chat_parse_deepseek_v3_1_content(builder);
+            // Regular content without reasoning
+            LOG("%s: no thinking forced open, parsing as regular content\n", __func__);
+
+            static const common_regex function_regex("(?:<｜tool▁calls▁begin｜>)?([^\n<]+)(?:<｜tool▁calls▁begin｜>)");
+            static const common_regex close_regex("(?:[\\s]*)?}<?");
+            static const common_regex tool_calls_begin("(?:<｜tool▁calls▁begin｜>|<｜tool_calls_begin｜>|<｜tool\\\\_calls\\\\_begin｜>|<｜tool▁calls｜>)");
+            static const common_regex tool_calls_end("}<?");
+
+            if (builder.syntax().parse_tool_calls) {
+                LOG("%s: attempting to parse tool calls\n", __func__);
+
+                parse_json_tool_calls(
+                    builder,
+                    /* block_open= */ tool_calls_begin,
+                    /* function_regex_start_only= */ std::nullopt,
+                    function_regex,
+                    close_regex,
+                    tool_calls_end);
+            } else {
+                LOG("%s: not parsing tool calls, adding as content\n", __func__);
+                builder.add_content(builder.consume_rest());
+            }
         }
     }
+
+    builder.finish();
 }
 
 static common_chat_params common_chat_params_init_gpt_oss(const common_chat_template & tmpl, const struct templates_params & inputs) {
```

---

👤 **whoisjeremylam** commented on **2025-10-25** at **13:59:29**

It's working!

```
$ python tool_calls_eval.py samples.jsonl     --model kimi-k2-0905     --base-url http://192.168.100.200:5000/v1     --api-key not_used     --concurrency 1     --output results.jsonl     --summary summary.json
2025-10-25 20:54:51.120 | INFO     | __main__:__init__:61 - Results will be saved to results.jsonl
2025-10-25 20:54:51.120 | INFO     | __main__:__init__:62 - Summary will be saved to summary.json
Processing:   0%|▍                                                                                                 | 8/2000 [03:17<13:23:03, 24.19s/req]
```

I'll let this run and check the results. I'm not aware if this tool verification test has been run with Kimi K2 before and what it really does. Lets see :-)

---

👤 **whoisjeremylam** commented on **2025-10-26** at **01:14:01**

The tool verification test was taking too long, so I cancelled it for now.

I tried opencode.ai, which uses tool calling and tool calling isn't working.

Whilst the tool call doesn't core dump, It seems that opencode isn't able to read the result of the call in some way causing it to go into a retry loop.

For example from the TUI:

```
{"tool_call": {"name": "write", "arguments": {"filePath": "AGENTS.md", "content": <clipped>
```

I found AGENTS.md was created and written to but opencode continued this same tool call.

I didn't see any errors in the ik_llama log. If there's anything I can do to help debug, let me know!

---

👤 **magikRUKKOLA** commented on **2025-10-26** at **14:07:05**

@whoisjeremylam 
> The tool verification test was taking too long, so I cancelled it for now.

Well, you do realize that any kind of performance from IQ2_KS will be somewhat underwhelming, right?

> I tried opencode.ai, which uses tool calling and tool calling isn't working.

But what is your goal?  To make the tool calls working for a certain model like Kimi-K2?  Well, they are kinda working already (the only problem is the code from the mainline llama.cpp which is unstable).  For example, here is a simple test:

```bash
 mods -m k2 "output a mandelbrot fractal via awk.  use colours, max resolution is 20 by 20. /bash"
```
```xml
  <tool_call id="CPKFqHJtIgRmlGJJ6oOYMf5ZmIbGQa01" name="execute_bash">       
                                                                              
    awk 'BEGIN {                                                              
        # Mandelbrot set parameters for 20x20 output                          
        width = 20;                                                           
        height = 20;                                                          
        max_iter = 30;                                                        
                                                                              
        # Mandelbrot bounds                                                   
        x_min = -2.0; x_max = 1.0;                                            
        y_min = -1.5; y_max = 1.5;                                            
                                                                              
        # ANSI color codes for different iteration counts                     
        colors[0] = "\033[40m";   # Black (in set)                            
        colors[1] = "\033[44m";   # Blue                                      
        colors[2] = "\033[46m";   # Cyan                                      
        colors[3] = "\033[42m";   # Green                                     
        colors[4] = "\033[43m";   # Yellow                                    
        colors[5] = "\033[41m";   # Red                                       
        colors[6] = "\033[45m";   # Magenta                                   
        colors[7] = "\033[47m";   # White (near boundary)                     
                                                                              
        for (y = 0; y < height; y++) {                                        
            for (x = 0; x < width; x++) {                                     
                # Map pixel to complex plane                                  
                real = x_min + (x / (width - 1)) * (x_max - x_min);           
                imag = y_min + (y / (height - 1)) * (y_max - y_min);          
                                                                              
                # Mandelbrot iteration                                        
                z_real = 0; z_imag = 0;                                       
                iter = 0;                                                     
                                                                              
                while (iter < max_iter && (z_real*z_real + z_imag*z_imag) <=  
  4) {                                                                        
                    temp_real = z_real*z_real - z_imag*z_imag + real;         
                    z_imag = 2*z_real*z_imag + imag;                          
                    z_real = temp_real;                                       
                    iter++;                                                   
                }                                                             
                                                                              
                # Color based on iteration count                              
                if (iter == max_iter) {                                       
                    color_idx = 0;  # In set - black                          
                } else {                                                      
                    color_idx = (iter % 7) + 1;                               
                }                                                             
                                                                              
                printf("%s  \033[0m", colors[color_idx]);                     
            }                                                                 
            print "";                                                         
        }                                                                     
    }'                                                                        
                                                                              
  </tool_call>                                                                
```
```xml                                                           
  <tool_response tool_call_id="CPKFqHJtIgRmlGJJ6oOYMf5ZmIbGQa01"              
  name="execute_bash">                                                        
```
<img width="400" height="400" alt="Image" src="https://github.com/user-attachments/assets/d5a4d9f3-64cf-4660-92b2-2f268f3badf3" />

```xml  
  </tool_response>
```
```text
  Here's a colorful Mandelbrot fractal rendered at 20×20 resolution using awk!
  The output shows the characteristic shape of the Mandelbrot set with        
  different colors representing how quickly points escape to infinity. The    
  black areas represent points that are in the Mandelbrot set (they don't     
  escape), while the colored areas show points outside the set, with colors   
  indicating the number of iterations before escaping. You can see the        
  distinctive bulbous shape forming even at this low resolution.              
```

So it read the available tools declaration from the user request, wrote a bash code, got the code executed, read the result, understood that its correct and wrote the final response.

What is not working in your opinion here?  It seems like everything is working.

May be you want to figure out why it doesn't work with opencode?  That will be a different story completely.

---

👤 **whoisjeremylam** commented on **2025-10-26** at **15:16:51**

> > The tool verification test was taking too long, so I cancelled it for now.
> 
> Well, you do realize that any kind of performance from IQ2_KS will be somewhat underwhelming, right?

If tool calling can be working correctly, the K2 vendor verification test can be used as an objective way of measuring one aspect of a quant's performance. Unfortunately, the test doesn't seem to write results as it goes along, so I can't tell if it was working or not.

> But what is your goal? To make the tool calls working for a certain model like Kimi-K2? Well, they are kinda working already (the only problem is the code from the mainline llama.cpp which is unstable). For example, here is a simple test:

The goal is to get tool calling working for with ik_llama.cpp and opencode, for which it is not working.

>  mods -m k2 "output a mandelbrot fractal via awk.  use colours, max resolution is 20 by 20. /bash"
...
> So it read the available tools declaration from the user request, wrote a bash code, got the code executed, read the result, understood that its correct and wrote the final response.
> 
> What is not working in your opinion here? It seems like everything is working.
> 
> May be you want to figure out why it doesn't work with opencode? That will be a different story completely.

Which application is `mods`? It's good that it is working there but how that application handles the return from tool calling might be different.

As I mentioned, I can see the tool call is being executed but the results are somehow not received by opencode. I have tool calling working in opencode working with vllm and other models like GLM V4.5-air.

---

👤 **magikRUKKOLA** commented on **2025-10-26** at **18:30:17**

@whoisjeremylam 
> If tool calling can be working correctly, the K2 vendor verification test can be used as an objective way of measuring one aspect of a quant's performance.

Well, for the quant performance its a custom to use llama-perplexity.  But your point is correct, the lower quants will show worse performance.

>  Unfortunately, the test doesn't seem to write results as it goes along, so I can't tell if it was working or not.

Well, may be I can run that test, but ... the code that handling the tool calls in llama.cpp (ik_llama.cpp) is very bad.  It's surprising that its working at all.  My idea was to fix that somehow first.  Unfortunately I wasn't able to find any time for that as of yet.

> The goal is to get tool calling working for with ik_llama.cpp and opencode, 

Okay cool.  May be I should look into it.  It should be something really simple.

> Which application is mods?

https://github.com/charmbracelet/mods/

Its a cli tool, but I can't recomend it because its just slow (that is, its eating up about two CPU cores for no reason at all while its working with a really long output).

> It's good that it is working there but how that application handles the return from tool calling might be different.

Of course its different.  You know why?  Because I wrote it in pure LUA.  That is, mods itself is not aware of the tool calls etc. its a messenger by itself.

---

👤 **magikRUKKOLA** commented on **2025-10-26** at **18:39:04**

@whoisjeremylam 

lol I just downloaded the opencode and went to the docs how to install it and the first thing I see is:

```bash
npm i -g opencode-ai@latest        # or bun/pnpm/yarn
```

Its a very bad sign.  npm got hacked alot.  Let me see if they actually got all the code together or they are relying on a third-party modules...

[EDIT]:

https://github.com/sst/opencode/blob/dev/package.json

OMG what a mess!  I am not sure if I want to install this stuff at all.  Possibly I have to make a virtual machine first.  And then install it there.

---

👤 **usrlocalben** commented on **2025-11-02** at **16:32:37**

I ran the n=2000 test (since I started they posted a n=4000 version) w/llama.cpp (and some ik_llama)

The results are [here](https://github.com/usrlocalben/k2vv-llamacpp). See the bottom of the README for partial ik_llama results.

When running ik_llama I never encountered any faults/coredumps, nor did I apply any patch.

I intend to do a full run w/ik_llama and post more results.

---

👤 **magikRUKKOLA** commented on **2025-11-02** at **17:07:27**

@usrlocalben 

Does the K2V test using streaming?  Judging by the dump of responses I can see only fully parsed responses.

---

👤 **usrlocalben** commented on **2025-11-02** at **17:14:10**

@magikRUKKOLA 
If you look at the test executor, it has impl to handle both stream={True,False} at [line 104](https://github.com/MoonshotAI/K2-Vendor-Verifier/blob/main/tool_calls_eval.py#L104) but the samples.json has stream=True in every request. (for the samples I used)
The event stream is handled in tool_calls_eval near [line 124.](https://github.com/MoonshotAI/K2-Vendor-Verifier/blob/main/tool_calls_eval.py#L124)

```
% grep -c 'stream": true' samples.jsonl
2000
% grep -c 'stream": false' samples.jsonl
0
% wc -l samples.jsonl
2000 samples.jsonl
```

---

👤 **magikRUKKOLA** commented on **2025-11-02** at **17:22:14**

> If you look at the test executor, it has impl to handle both stream={True,False} at [line 104](https://github.com/MoonshotAI/K2-Vendor-Verifier/blob/main/tool_calls_eval.py#L104) but the samples.json has stream=True in every request. (for the samples I used)

Yeah, I noticed.  And given that you had no crashes this is very strange.  I would understand if you'd had stream=false, but now I don't understand anything. )

[EDIT]:

can you take a look at the system journal?  Like the llama-server instance uptime etc.  Are you sure it never crashed during the testing?

---

👤 **usrlocalben** commented on **2025-11-02** at **17:27:20**

@magikRUKKOLA maybe relevant is my GGUF lineage. My impression is there is some badness in various K2 quants. Multiple people had parallel WIP conversion scripts at the time it was new, which had differences. Unsloths (and I think ubergarm's) have different special tokens than some others (anikifoss, DevQuasar). I noticed this once when trying jukofyork's draft models which had conflicts with unsloth/ubergarm special tokens.

Add to the mix Unsloth's dubious "chat template fixes" and now there's a lot of variation floating around. (Or at least that's my impression of the situation)

Also, Moonshot has updated the tokenizer and chat-template at various points over time since release. I used the latest available.

For these reasons, I "build from source" and detailed the process in the README.

---

👤 **usrlocalben** commented on **2025-11-02** at **17:32:46**

> can you take a look at the system journal? Like the llama-server instance uptime etc. Are you sure it never crashed during the testing?

I don't believe it crashed, but I do run ik_llama under llama-swap which I think would cause silent restarts in the face of errors. 

When I do a full and proper ik_llama run I'll take care to monitor this.

The n=2000 llama.cpp run was done on a single invocation of llama.cpp w/o any llama-swap/systemd etc restart, and never crashed. I only stopped/resumed the test sequence a few times manually to make the system available for other purposes.