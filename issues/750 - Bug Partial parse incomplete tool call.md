## 📌 [Issue #750](https://github.com/ikawrakow/ik_llama.cpp/issues/750) - Bug:  Partial parse: incomplete tool call

| **Author** | `magikRUKKOLA` |
| :--- | :--- |
| **State** | ✅ **Open** |
| **Created** | 2025-09-02 |
| **Updated** | 2025-10-26 |

---

## 📄 Description

### What happened?

The tool calls seems to be broken?

### Name and Version

llama-server --version
version: 3872 (f8d511a3)
built with cc (Debian 14.3.0-5) 14.3.0 for x86_64-linux-gnu


### What operating system are you seeing the problem on?

_No response_

### Relevant log output

```shell
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  Let me try to create the file in the home directory of the current user? But the user is root, so home is /root.
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  Alternatively, we can try:
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:    mkdir -p /root
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:    touch /root/testfile.txt
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  But we already tried that.
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  Given the constraints, I think we need to report that we are unable to create the file due to permission issues, even as root.
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  However, let me try one more thing: use `sudo` inside the command? But if we are root, we don't need it. And if we are not, then the tool is not running as root.
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  Alternatively, we can check the user:
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:    id -un
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  If it returns "root", then we are root and the issue is elsewhere.
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  So I will run:
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:    id -un
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  to see the current user.
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  Then, if it's root, we try to create the file again and see the error.
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  If it's not root, then we have a problem with the tool.
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  But note: the tool is named "execute_root_bash", so it should run as root.
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  Let me run:
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:    id -un
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  and
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:    echo $HOME
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  to see the home directory.
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  But note: the user request is to create in the home directory of the root user, which is /root.
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  So, I will run:
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:    id -un && touch /root/testfile.txt
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  and see the output.
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  Alternatively, we can break it into two commands? But we can do it in one.
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  However, the tool only allows one command per call? Actually, we can chain with `&&` or `;`.
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  Given the previous error, let me try to run the command to check the user and then create the file.
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  But note: if the user is not root, we might get permission denied on both.
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  Alternatively, we can run:
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:    sudo touch /root/testfile.txt
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  but if we are not root, and if sudo is not set up, it will ask for a password.
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  Since we are in a non-interactive environment, we cannot enter a password.
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  Therefore, I will run:
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:    whoami
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  and then
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:    touch /root/testfile.txt
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  in two separate tool calls? But we are limited to one call per response.
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  We can do:
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:    whoami > /tmp/whoami.txt && touch /root/testfile.txt
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  and then read the /tmp/whoami.txt? But we cannot read files.
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  Alternatively, we can output the whoami to stdout:
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:    whoami && touch /root/testfile.txt
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  Then the output will be:
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:    root
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:    touch: cannot touch '/root/testfile.txt': Permission denied
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  That would be odd.
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  Or:
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:    whoami ; touch /root/testfile.txt
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  But same thing.
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  Given the time, I will run:
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:    id -u
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  If it returns 0, then we are root.
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  Then we can try:
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:    ls -ld /root
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  to see the permissions.
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  But we are getting permission denied on creating a file, so we might not have write permission on the directory.
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  Let me check the permissions of /root:
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:    ls -ld /root
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  We can run:
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:    ls -ld /root
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  and then
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:    touch /root/testfile.txt
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  in one command.
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  But note: the tool returns the output of the entire command.
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  Alternatively, we can run:
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:    ls -ld /root && touch /root/testfile.txt
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  If the first command fails, the second won't run.
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  Or:
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:    ls -ld /root ; touch /root/testfile.txt
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  I'll run:
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:    ls -ld /root ; touch /root/testfile.txt
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  and see the output.
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  However, we are getting a lot of errors and the user just wants a test file.
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  Let me try to create the file in the current working directory and then move it? But we don't know the current working directory.
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  We can do:
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:    current_dir=$(pwd)
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:    touch testfile.txt
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:    sudo mv testfile.txt /root/
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  But we are root, so we don't need sudo. And we are in a temporary shell? The working directory might be /tmp or something.
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  Alternatively, we can do:
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:    touch /tmp/testfile.txt && mv /tmp/testfile.txt /root/
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  But moving to /root requires write permission on /root.
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  And we are root, so it should work.
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  But we got permission denied when touching in /root, so moving to /root might also fail.
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  However, moving a file into a directory requires write permission on the directory, not on the file.
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  And we are root, so we should have write permission on /root.
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  So I will try:
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:    touch /tmp/testfile.txt && mv /tmp/testfile.txt /root/
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  If the touch works and the mv fails, then we know the issue is with the /root directory.
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  Alternatively, we can do:
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:    echo "test" > /tmp/testfile.txt && sudo mv /tmp/testfile.txt /root/
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  But again, we are root so we don't need sudo.
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  Let me try without sudo.
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  Given the above, I will run:
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:    touch /tmp/testfile.txt && mv /tmp/testfile.txt /root/ && echo "File created successfully in /root"
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  If it fails, we will see the error.
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  But note: the initial error was on touching in /root, not on moving.
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  So let's try this.
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  But note: the user wants the file in the home directory of the root user, which is /root.
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  We are creating it in /tmp and moving to /root.
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  This should be acceptable.
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  Let me run:
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:    touch /tmp/testfile.txt && mv /tmp/testfile.txt /root/ && echo "Success"
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  If we get an error on the mv, then we know.
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  But if we get an error on the touch in /tmp? That would be odd because /tmp is writable by everyone.
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:  So I'll run that command.
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]: </think>
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]: I'll try a different approach to create the test file:<｜tool▁calls▁begin｜><｜tool▁call▁begin｜>function<｜tool▁sep｜>execute_root_bash
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]: 
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]: {"command":"echo 'Test content' | tee \/
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]: [1756778659] [/opt/ik_llama.cpp/ik_llama.cpp/common/json-partial.cpp:  129][       common_json_parse] Failed to parse up to error: [json.exception.parse_error.101] parse error at line 1, column 41: syntax error while parsing value - invalid string: missing closing quote; last read: '"echo 'Test content' | tee \/': <<<{"command":"echo 'Test content' | tee \/>>>
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]: [1756778659] Parsed partial JSON: {"command":"echo 'Test content' | tee /649989314"} (json_healing_marker: 649989314)
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]: [1756778659] Cleaned up JSON {"command":"echo 'Test content' | tee /649989314"} to "{\"command\":\"echo 'Test content' | tee /" (json_healing_marker : '649989314')
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]: [1756778659] Partial parse: incomplete tool call
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]: terminate called after throwing an instance of 'std::runtime_error'
Sep 02 02:04:19 xxx run-ik_llama.cpp.sh[66384]:   what():  Invalid diff: '{"command":"echo 'Test content' | tee \' not found at start of '{"command":"echo 'Test content' | tee /'
Sep 02 02:04:33 xxx run-ik_llama.cpp.sh[66365]: /opt/THIREUS-6.2478bpw/run-ik_llama.cpp.sh: line 61: 66384 Aborted                 CUDA_VISIBLE_DEVICES="0,1,2" /opt/ik_llama.cpp/ik_llama.cpp/build/bin/llama-server --model /opt/GGUF-Tool-Suite/GGUF-Tool-Suite/DeepSeek-R1-0528.ROOT-6.2478bpw/DeepSeek-R1-0528-THIREUS-BF16-SPECIAL_TENSOR-00001-of-01148.gguf --alias THIREUS/DeepSeek-R1-0528-6.2478bpw --ctx-size $(
(160 * 1024)) -b $((16 * 512)) -ub $((8 * 512)) --mlock --temp 0.5 --top-k 0 --top-p 1.0 --min-p 0.1 --repeat-penalty 1.0 -ctk q8_0 -mla 3 -fa -fmoe -amb 512 --split-mode layer --tensor-split 10,21,20 --main-gpu 1 --override-tensor exps=CPU --n-gpu-layers 99 --threads $(grep ^cpu\\scores /proc/cpuinfo | uniq | awk '{print $4}' | xargs -I{} echo "{}-0" | bc) --host 0.0.0.0 --port 8080 --log-enable --logdir /var
/log/ --jinja --special --verbose-prompt --verbosity 2 --prompt-cache "$HOME/.cache/ik_llama.cpp/prompt-cache.bin" --prompt-cache-all --slot-save-path "$HOME/.cache/ik_llama.cpp/slot.bin" --lookup-cache-dynamic "$HOME/.cache/ik_llama.cpp/slot.bin" --keep -1 --slot-prompt-similarity 0.35 --metrics
Sep 02 02:04:33 xxx systemd[1]: ik_llama.cpp-deepseek-r1.service: Main process exited, code=exited, status=134/n/a
Sep 02 02:04:33 xxx systemd[1]: ik_llama.cpp-deepseek-r1.service: Failed with result 'exit-code'.
Sep 02 02:04:33 xxx systemd[1]: ik_llama.cpp-deepseek-r1.service: Consumed 2d 16h 29min 32.173s CPU time, 22G memory peak.
Sep 02 02:05:33 xxx systemd[1]: ik_llama.cpp-deepseek-r1.service: Scheduled restart job, restart counter is at 2.
Sep 02 02:05:33 xxx systemd[1]: Started ik_llama.cpp-deepseek-r1.service - ik_llama.cpp Service.
```

---

## 💬 Conversation

👤 **magikRUKKOLA** commented on **2025-10-02** at **12:37:44**

tried to apply the following:

```diff
--- ik_llama.cpp.bak/common/json-partial.cpp    2025-10-02 10:12:42.317893442 +0000
+++ ik_llama.cpp/common/json-partial.cpp        2025-10-02 11:54:39.609344819 +0000
@@ -179,7 +179,16 @@
                     str += (out.healing_marker.json_dump_marker = "\"" + magic_seed) + "\": 1" + closing;
                 } else if (can_parse(str + "\"" + closing)) {
                     // Was inside an object value string
-                    str += (out.healing_marker.json_dump_marker = magic_seed) + "\"" + closing;
+                    //str += (out.healing_marker.json_dump_marker = magic_seed) + "\"" + closing;
+                    // Check if we're immediately after a backslash (escape sequence)
+                    if (!str.empty() && str.back() == '\\') {
+                        // We're in an escape sequence - backtrack to before the backslash
+                        // to avoid breaking the escape sequence
+                        std::string truncated_str = str.substr(0, str.length() - 1);
+                        str = truncated_str + (out.healing_marker.json_dump_marker = magic_seed) + "\"" + closing;
+                    } else {
+                        str += (out.healing_marker.json_dump_marker = magic_seed) + "\"" + closing;
+                    }
                 } else if (str[str.length() - 1] == '\\' && can_parse(str + "\\\"" + closing)) {
                     // Was inside an object value string after an escape
                     str += (out.healing_marker.json_dump_marker = "\\" + magic_seed) + "\"" + closing;
```

but now getting slightly different issue:

```
Oct 02 12:22:06 xxx run-ik_llama.cpp.sh[3699712]: [1759407726] common_chat_parse_deepseek_v3_1: thinking_forced_open: 1
Oct 02 12:22:06 xxx run-ik_llama.cpp.sh[3699712]: [1759407726] common_chat_parse_deepseek_v3_1: reasoning_format none, adding content
Oct 02 12:22:06 xxx run-ik_llama.cpp.sh[3699712]: [1759407726] common_chat_parse_deepseek_v3_1_content: parse_tool_calls
Oct 02 12:22:06 xxx run-ik_llama.cpp.sh[3699712]: [1759407726] [/opt/ik_llama.cpp/ik_llama.cpp/common/json-partial.cpp:  127][       common_json_parse] Failed to parse up to error: [json.exception.parse_error.101] parse error at line 1, column 3: syntax error while parsing object key - invalid string: missing closing quote; last read: '"'; expected string literal: <<<{">>>
Oct 02 12:22:06 xxx run-ik_llama.cpp.sh[3699712]: [1759407726] Parsed partial JSON: {"951426815":1} (json_healing_marker: 951426815)
Oct 02 12:22:06 xxx run-ik_llama.cpp.sh[3699712]: [1759407726] Cleaned up JSON {"951426815":1} to "{\"" (json_healing_marker : '951426815')
Oct 02 12:22:06 xxx run-ik_llama.cpp.sh[3699712]: [1759407726] Partial parse: incomplete tool call
Oct 02 12:22:06 xxx run-ik_llama.cpp.sh[3699712]: VERB [                    send] send new result | tid="140355385217024" timestamp=1759407726 id_task=0
Oct 02 12:22:06 xxx run-ik_llama.cpp.sh[3699712]: VERB [                    send] queue_results.push_back | tid="140355385217024" timestamp=1759407726 id_task=0
Oct 02 12:22:06 xxx run-ik_llama.cpp.sh[3699712]: VERB [           process_token] next token | tid="140355385217024" timestamp=1759407726 id_slot=0 id_task=0 token=24313 token_text="{\"" has_next_token=true n_remain=-1 n_decoded=937 stopped_eos=false stopped_word=false stopped_limit=false stopping_word=""
Oct 02 12:22:06 xxx run-ik_llama.cpp.sh[3699712]: VERB [            update_slots] run slots completed | tid="140355385217024" timestamp=1759407726
Oct 02 12:22:06 xxx run-ik_llama.cpp.sh[3699712]: VERB [              start_loop] wait for new task | tid="140355385217024" timestamp=1759407726
Oct 02 12:22:06 xxx run-ik_llama.cpp.sh[3699712]: VERB [              start_loop] new task may arrive | tid="140355385217024" timestamp=1759407726
Oct 02 12:22:06 xxx run-ik_llama.cpp.sh[3699712]: VERB [              start_loop] callback_new_task | tid="140355385217024" timestamp=1759407726 id_task=937
Oct 02 12:22:06 xxx run-ik_llama.cpp.sh[3699712]: VERB [              start_loop] update_multitasks | tid="140355385217024" timestamp=1759407726
Oct 02 12:22:06 xxx run-ik_llama.cpp.sh[3699712]: VERB [              start_loop] callback_update_slots | tid="140355385217024" timestamp=1759407726
Oct 02 12:22:06 xxx run-ik_llama.cpp.sh[3699712]: VERB [            update_slots] posting NEXT_RESPONSE | tid="140355385217024" timestamp=1759407726
Oct 02 12:22:06 xxx run-ik_llama.cpp.sh[3699712]: VERB [                    post] new task id | tid="140355385217024" timestamp=1759407726 new_id=938
Oct 02 12:22:06 xxx run-ik_llama.cpp.sh[3699712]: VERB [            update_slots] slot decode token | tid="140355385217024" timestamp=1759407726 id_slot=0 id_task=0 n_ctx=143360 n_past=1160 n_system_tokens=0 n_cache_tokens=1160 truncated=false
Oct 02 12:22:06 xxx run-ik_llama.cpp.sh[3699712]: VERB [            update_slots] decoding batch | tid="140355385217024" timestamp=1759407726 n_tokens=1
Oct 02 12:22:06 xxx run-ik_llama.cpp.sh[3699712]: VERB [              operator()] data stream | tid="140080584839168" timestamp=1759407726 to_send="data: {\"choices\":[{\"finish_reason\":null,\"index\":0,\"delta\":{\"tool_calls\":[{\"index\":0,\"id\":\"gzEG3LmKcJa6l6w1XGD4kqVOd9APPGfC\",\"type\":\"function\",\"function\":{\"name\":\"execute_bash\",\"arguments\":\"{\\\"\"}}]}}],\"created\":1759407726,\"id\":\>
Oct 02 12:22:06 xxx run-ik_llama.cpp.sh[3699712]: [1759407726] Parsing input with format DeepSeek V3.1: First, the user wants to calculate the factorial of 142 using awk. Factorial of n (n!) is the product of all positive integers up to n. For large numbers like 142, this could be computationally intensive, but awk should handle it.
```

---

👤 **magikRUKKOLA** commented on **2025-10-02** at **20:18:08**

Apparently, the issues I was having were somehow related to the --special option that I was using.
So I had to do:

```diff
--- ../ik_llama.cpp.bak/common/chat.cpp 2025-10-02 10:12:42.317893442 +0000
+++ common/chat.cpp     2025-10-02 19:56:10.509515341 +0000
@@ -40,7 +40,8 @@
             // and the current ended on a stop word (erased).
             return "";
         }
-        throw std::runtime_error("Invalid diff: '" + last + "' not found at start of '" + current + "'");
+        //throw std::runtime_error("Invalid diff: '" + last + "' not found at start of '" + current + "'");
+        return current;
     }
     return current.substr(last.size());
 }
```

and the following arguments to the llama-server:

(the chat template is from unsloth)
```
    --jinja \                                                                                                               
    --chat-template-file /opt/ubergarm/DeepSeek-V3.1-Terminus/IQ5_K/chat_template.jinja \                                   
    --special \                                                                                                             
    --reasoning-format none \  
```

Now its seems to be working.  That is, that alone seems to fix the issue.   What was the purpose of that diff check?

---

👤 **magikRUKKOLA** commented on **2025-10-02** at **20:51:09**

Uh.  Oh.  So all this code with these crazy unescaping (/healing?) was made only to be able to stream the tool calls in a readable format?!  It reminded me how I wrote similar stuff for my lua wrapper -- to deal with the tool calls.  But that was like, very easy.  For example, to extract the unescaped data from the json data of the llm response I simply was regexping for '"command:"' : 
```lua
                                                                                                                                                                                                                                                                                              
                        -- Process new arguments if present                                                                                                                                                                                                                                                           
                        if tool_call["function"].arguments and tool_call["function"].arguments ~= "" then                                                                                                                                                                                                             
                            func_state.arguments = func_state.arguments .. tool_call["function"].arguments                                                                                                                                                                                                            
                            func_state.arg_buffer = func_state.arg_buffer .. tool_call["function"].arguments                                                                                                                                                                                                          
                                                                                                                                                                                                                                                                                                                      
                            -- Process the arg_buffer character by character                                                                                                                                                                                                                                          
                            local i = 1                                                                                                                                                                                                                                                                               
                            while i <= #func_state.arg_buffer do                                                                                                                                                                                                                                                      
                                local char = func_state.arg_buffer:sub(i, i)                                                                                                                                                                                                                                          
                                prev_chars = prev_chars .. char                                                                                                                                                                                                                                                       
                                                                                                                                                                                                                                                                                                                      
                                if func_state.escape_next then                                                                                                                                                                                                                                                        
                                    if func_state.command_started then                                                                                                                                                                                                                                                
                                        func_state.command_buffer = func_state.command_buffer .. char                                                                                                                                                                                                                 
                                    end                                                                                                                                                                                                                                                                               
                                    func_state.escape_next = false                                                                                                                                                                                                                                                    
                                    i = i + 1                                                                                                                                                                                                                                                                         
                                elseif char == "\\" then                                                                                                                                                                                                                                                              
                                    if func_state.command_started then                                                                                                                                                                                                                                                
                                        func_state.command_buffer = func_state.command_buffer .. char                                                                                                                                                                                                                 
                                    end                                                                                                                                                                                                                                                                               
                                    func_state.escape_next = true                                                                                                                                                                                                                                                     
                                    i = i + 1                                                                                                                                                                                                                                                                         
                                elseif char == '"' then                                                                                                                                                                                                                                                               
                                    if not func_state.in_string then                                                                                                                                                                                                                                                  
                                        func_state.in_string = true                                                                                                                                                                                                                                                   
                                                                                                                                                                                                                                                                                                                      
                                        -- Check if we just passed "command":" or "code":                                                                                                                                                                                                                             
                                        if prev_chars:match('["{]command":%s*"') or                                                                                                                                                                                                                                   
                                        prev_chars:match('["{]code":%s*"') then                                                                                                                                                                                                                                       
                                            func_state.command_started = true                                                                                                                                                                                                                                         
                                            func_state.command_buffer = ""                                                                                                                                                                                                                                            
                                            prev_chars = ""                                                                                                                                                                                                                                                           
                                        end                                                                                                                                                                                                                                                                           
                                    else                                                                                                                                                                                                                                                                              
                                        func_state.in_string = false                                                                                                                                                                                                                                                  
                                                                                                                                                                                                                                                                                                                      
                                        -- If we were collecting command content, send it                                                                                                                                                                                                                             
                                        if func_state.command_started then                                                                                                                                                                                                                                            
                                            if func_state.command_buffer ~= "" then                                                                                                                                                                                                                                   
                                                local unescaped_str = _G.common.smart_unescape(func_state.command_buffer)                                                                                                                                                                                             
                                                local frontend_response = {                                                                                                                                                                                                                                           
                                                    id = chat_id,                                                                                                                                                                                                                                                     
                                                    object = "chat.completion.chunk",                                                                                                                                                                                                                                 
                                                    created = os.time(),                                                                                                                                                                                                                                              
                                                    choices = {{                                                                                                                                                                                                                                                      
                                                        index = 0,                                                                                                                                                                                                                                                    
                                                        delta = {                                                                                                                                                                                                                                                     
                                                            role = "assistant",                                                                                                                                                                                                                                       
                                                            content = unescaped_str                                                                                                                                                                                                                                   
                                                        }                                                                                                                                                                                                                                                             
                                                    }}                                                                                                                                                                                                                                                                
                                                }                                                                                                                                                                                                                                                                     
                                                ngx.print("data: "..cjson.encode(frontend_response).."\n\n")                                                                                                                                                                                                          
                                                ngx.flush(true)                                                                                                                                                                                                                                                       
                                            end                                                                                                                                                                                                                                                                       
                                            func_state.command_started = false                                                                                                                                                                                                                                        
                                            func_state.command_buffer = ""                                                                                                                                                                                                                                            
                                        end                                                                                                                                                                                                                                                                           
                                    end                                                                                                                                                                                                                                                                               
                                    i = i + 1                                                                                                                                                                                                                                                                         
                                else                                                                                                                                                                                                                                                                                  
                                    if func_state.in_string and func_state.command_started then                                                                                                                                                                                                                       
                                        func_state.command_buffer = func_state.command_buffer .. char                                                                                                                                                                                                                 
                                        if func_state.command_buffer ~= "" then                                                                                                                                                                                                                                       
                                            local unescaped_str = _G.common.smart_unescape(func_state.command_buffer)                                                                                                                                                                                                 
                                            local frontend_response = {                                                                                                                                                                                                                                               
                                                id = chat_id,                                                                                                                                                                                                                                                         
                                                object = "chat.completion.chunk",                                                                                                                                                                                                                                     
                                                created = os.time(),                                                                                                                                                                                                                                                  
                                                choices = {{                                                                                                                                                                                                                                                          
                                                    index = 0,                                                                                                                                                                                                                                                        
                                                    delta = {                                                                                                                                                                                                                                                         
                                                        role = "assistant",                                                                                                                                                                                                                                           
                                                        content = unescaped_str                                                                                                                                                                                                                                       
                                                    }                                                                                                                                                                                                                                                                 
                                                }}                                                                                                                                                                                                                                                                    
                                            }                                                                                                                                                                                                                                                                         
                                            ngx.print("data: "..cjson.encode(frontend_response).."\n\n")                                                                                                                                                                                                              
                                            ngx.flush(true)                                                                                                                                                                                                                                                           
                                        end                                                                                                                                                                                                                                                                           
                                        func_state.command_buffer = ""                                                                                                                                                                                                                                                
                                    end                                                                                                                                                                                                                                                                               
                                    i = i + 1                                                                                                                                                                                                                                                                         
                                end                                                                                                                                                                                                                                                                                   
                            end                                                                                                                                                                                                                                                                                       
                                                                                                                                                                                                                                                                                                                      
                            -- Keep any incomplete sequence for next chunk                                                                                                                                                                                                                                            
                            func_state.arg_buffer = ""                                                                                                                                                                                                                                                                
                        end
```

And, the unescaping just by prefixing and postfixing the double quotes around the raw escaped json argument:

```lua
        -- Unescapes control characters only when they were evenly escaped                                                  
        smart_unescape = function (str)                                                                                     
--          return str:gsub("\\([\\\"'bfnrt])", {                                                                           
--              ['\\'] = "\\",                                                                                              
--              ['"'] = '"',                                                                                                
--              ["'"] = "'",                                                                                                
--              ['b'] = "\b",                                                                                               
--              ['f'] = "\f",                                                                                               
--              ['n'] = "\n",                                                                                               
--              ['r'] = "\r",                                                                                               
--              ['t'] = "\t"                                                                                                
--            })                                                                                                            
            local ok, decoded = pcall(cjson.decode, '"' .. str .. '"')                                                      
            if ok then return decoded                                                                                       
            else return str end                                                                                             
        end,   
```

That's it.  Pretty simple, huh?

But ... the approach from the llama.cpp ... ?   Why?  What for?  Who came up with the idea of "healing marker" etc.?  Why?

---

👤 **magikRUKKOLA** commented on **2025-10-03** at **22:33:06**

Another error.

```
Oct 03 20:56:35 xxx run-ik_llama.cpp.sh[450744]: [1759524995] common_chat_parse_deepseek_v3_1_content: parse_tool_calls
Oct 03 20:56:35 xxx run-ik_llama.cpp.sh[450744]: [1759524995] [/opt/ik_llama.cpp/ik_llama.cpp/common/json-partial.cpp:  127][       common_json_parse] Failed to parse up to error: [json.exception.parse_error.101] parse error at line 1, column 130:
 syntax error while parsing value - invalid string: missing closing quote; last read: '"import numpy as np\nimport math\nfro
m numba import cuda, jit\n\ndef is_point_inside_stationary_heptagon(x, y):\n    \"\"': <<<{"code":"import numpy as np\nimport math\nfrom numba import cuda, jit\n\ndef is_point_inside_stationary_heptagon(x, y):\n    \"\">>>
Oct 03 20:56:35 xxx run-ik_llama.cpp.sh[450744]: [1759524995] Parsed partial JSON: {"code":"import numpy as np\nimport math
\nfrom numba import cuda, jit\n\ndef is_point_inside_stationary_heptagon(x, y):\n    \"\"1237381409"} (json_healing_marker: 
1237381409)
Oct 03 20:56:35 xxx run-ik_llama.cpp.sh[450744]: [1759524995] Cleaned up JSON {"code":"import numpy as np\nimport math\nfro
m numba import cuda, jit\n\ndef is_point_inside_stationary_heptagon(x, y):\n    \"\"1237381409"} to "{\"code\":\"import nump
y as np\\nimport math\\nfrom numba import cuda, jit\\n\\ndef is_point_inside_stationary_heptagon(x, y):\\n    \\\"\\\"" (json_healing_marker : '1237381409')
Oct 03 20:56:35 xxx run-ik_llama.cpp.sh[450744]: [1759524995] Partial parse: incomplete tool call
Oct 03 20:56:35 xxx run-ik_llama.cpp.sh[450744]: VERB [                    send] send new result | tid="140408732569600" timestamp=1759524995 id_task=7306
```

Can we please remove all the error checking for a streamed escaped json?  The code from llama.cpp mainline is really bad.  Actually, it makes the tool call functionality just unusable.

[EDIT]:

Ah, the missing closing quote supposed to be fixed via: https://github.com/ikawrakow/ik_llama.cpp/issues/750#issuecomment-3361029305

---

👤 **magikRUKKOLA** commented on **2025-10-04** at **00:45:30**

lol.  another error:

```
Oct 04 00:34:06  run-ik_llama.cpp.sh[487431]: terminate called after throwing an instance of 'nlohmann::json_abi_v3_12_0
::detail::parse_error'
Oct 04 00:34:06  run-ik_llama.cpp.sh[487431]:   what():  [json.exception.parse_error.101] parse error at line 1, column 
13: syntax error while parsing object separator - unexpected '}'; expected ':'
Oct 04 00:34:08  run-ik_llama.cpp.sh[487406]: /opt/thireus/DeepSeek-V3.1-Terminus-5.4498bpw/run-ik_llama.cpp.sh: line 75
: 487431 Aborted                    /opt/ik_llama.cpp/ik_llama.cpp/build/bin/llama-server --model /opt/thireus/DeepSeek-V3.1
-Terminus-5.4498bpw/DeepSeek-V3.1-Terminus-THIREUS-BF16-SPECIAL_TENSOR-00001-of-01087.gguf --alias THIREUS/DeepSeek-V3.1-Ter
minus_5.4498bpw --ctx-size $((160 * 1024)) -b $((16 * 512)) -ub $((8 * 512)) --mlock --temp 0.5 --top-k 0 --top-p 1.0 --min-
p 0.1 --repeat-penalty 1.1 -ctk q8_0 -mla 3 -fa -fmoe -amb 512 --split-mode layer --tensor-split 10,21,20 --main-gpu 1 --ove
rride-tensor exps=CPU --n-gpu-layers 99 --threads $(grep ^cpu\\scores /proc/cpuinfo | uniq | awk '{print $4}' | xargs -I{} e
cho "{}-0" | bc) --host 0.0.0.0 --port 8080 --jinja --chat-template-file /opt/thireus/DeepSeek-V3.1-Terminus-5.4498bpw/chat_
template.jinja --special --log-enable --logdir /var/log/ --verbosity 1 --verbose-prompt --reasoning-format none --prompt-cac
he "$HOME/.cache/ik_llama.cpp/prompt-cache.bin" --prompt-cache-all --slot-save-path "$HOME/.cache/ik_llama.cpp/slot.bin" --l
ookup-cache-dynamic "$HOME/.cache/ik_llama.cpp/slot.bin" --keep -1 --slot-prompt-similarity 0.35 --metrics
Oct 04 00:34:08  systemd[1]: ik_llama.cpp-deepseek-v3.1-termunus.service: Main process exited, code=exited, status=134/n
/a
Oct 04 00:34:08  systemd[1]: ik_llama.cpp-deepseek-v3.1-termunus.service: Failed with result 'exit-code'.
Oct 04 00:34:08  systemd[1]: ik_llama.cpp-deepseek-v3.1-termunus.service: Consumed 10h 34min 13.339s CPU time, 3.7G memo
ry peak.
Oct 04 00:35:08  systemd[1]: ik_llama.cpp-deepseek-v3.1-termunus.service: Scheduled restart job, restart counter is at 1
.
Oct 04 00:35:08  systemd[1]: Started ik_llama.cpp-deepseek-v3.1-termunus.service - ik_llama.cpp Service.
```

---

👤 **magikRUKKOLA** commented on **2025-10-06** at **07:14:47**

It seems that the issue was related to the fact that Deepseek-V3.1-Terminus doesn't output the opening <think> tag which interfere with the way parsing of tool calls is done.  Currently the following is working on a few of my tests:

```diff
--- ik_llama.cpp.master/common/chat.cpp	2025-10-02 10:12:42.317893442 +0000
+++ ik_llama.cpp/common/chat.cpp	2025-10-06 06:37:49.530685267 +0000
@@ -1446,39 +1446,113 @@
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

I was wondering if its rather possible to change the chat template to always output the <think> tag so that the original code would work without changes?

---

👤 **shun095** commented on **2025-10-09** at **14:30:43**

FYI: The JSON healing logic appears to have been implemented in this PR. 
https://github.com/ggml-org/llama.cpp/pull/12379

---

👤 **magikRUKKOLA** commented on **2025-10-10** at **12:40:30**

@shun095 
> FYI: The JSON healing logic appears to have been implemented in this PR. [ggml-org/llama.cpp[#12379](https://github.com/ikawrakow/ik_llama.cpp/issues/12379)](https://github.com/ggml-org/llama.cpp/pull/12379)

Uh oh.  Check this out, from the official pull request page:

> [ ] Send partial JSON (common_json) as separate PR(?) or fold into chat-parser.cpp

That is, unimplemented.  lool.

So in order to make the json tool calls work in streaming format with the llama.cpp code would be to get rid of such code and implement the proper streaming (in decode) via other means.  I suggest LUA.

---

👤 **magikRUKKOLA** commented on **2025-10-10** at **12:47:43**

Additionally, I suggest there supposed to be a robust testing of the tool call functionality.  I suggest the following.  There are various benchmarks of the llms available.  At the DeepSeek-V3.1-Terminus there is the following info:

> Agentic Tool Use	
> BrowseComp	30.0	38.5
> BrowseComp-zh	49.2	45.0
> SimpleQA	93.4	96.8
> SWE Verified	66.0	68.4
> SWE-bench Multilingual	54.5	57.8
> Terminal-bench	31.3	36.7

The test should actually verify if the LLM is getting the declared success rates.  If not, something is broken.  Huh?

---

👤 **hksdpc255** commented on **2025-10-24** at **02:24:38**

The parsing logic in `chat.cpp` looks identical to the upstream llama.cpp. Maybe the issue comes from unsynced changes in the supporting libraries like `chat-parser.cpp` or `common.cpp`?

---

👤 **magikRUKKOLA** commented on **2025-10-26** at **14:18:21**

@hksdpc255 
> The parsing logic in `chat.cpp` looks identical to the upstream llama.cpp. Maybe the issue comes from unsynced changes in the supporting libraries like `chat-parser.cpp` or `common.cpp`?

Well, are you saying this bug doesn't occur in mainline?  Looking at the code, the comments, the docs etc.  I don't think its the case.  The code is a garbage.

---

👤 **magikRUKKOLA** commented on **2025-10-26** at **14:22:26**

@hksdpc255 

Quotes form the llama.cpp collaborator:

```
Yes, I can confirm from my template implementation that this code is buggy as hell.
```

```
 From my personal experience, the only reliable workaround has been to disable the partial tool streaming whatsoever - leave partial streaming, but for tool calls only stream then when the entire tool call has been parsed.
```

LOOL

Also:

```
 We don't really need tests on live models, but more robust tests for streaming would certainly be required.
```

You see, they don't need test on live models -- they are going to make some synthetic tests, "merge it as it is" and hope for the best.

The thing is that, llama.cpp (and ik_llama.cpp hence) already got the tool call testing.  I looked into it and could not understand what they are for.  They don't test the actual tool calls lol.  So the synthetic tool calls test are **NOT** working.  Like at all.