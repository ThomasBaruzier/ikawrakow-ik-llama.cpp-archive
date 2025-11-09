## 🔀 [Pull Request #558](https://github.com/ikawrakow/ik_llama.cpp/pull/558) - Add mikupad to ik_llama as an alternative WebUI

| **Author** | `saood06` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `s6/mikupad` |
| **Target Branch** | `main` |
| **Created** | 2025-06-26 |
| **Updated** | 2025-08-24 |
| **Merged** | 2025-08-24 |

---

## 📄 Description

This PR adds [mikupad](https://github.com/lmg-anon/mikupad) (and new endpoints to `server.cpp` that mikupad uses to manage its sql database).

It must be built with `-DLLAMA_SERVER_SQLITE3=ON`.

> [!Tip]
><details><summary>Click here if you see `Could NOT find SQLite3 (missing: SQLite3_INCLUDE_DIR SQLite3_LIBRARY)` when building</summary>
><details><summary>Fix for Windows</summary>
>
>In an install directory:
>```
>git clone https://github.com/microsoft/vcpkg
>.\vcpkg\bootstrap-vcpkg.bat -disableMetrics
>.\vcpkg\vcpkg.exe install sqlite3:x64-windows
>
>```
>then pass `-DCMAKE_TOOLCHAIN_FILE="[...]\vcpkg\scripts\buildsystems\vcpkg.cmake"` where [...] is the location of vcpkg installed above
></details>
><details><summary>Fix for Linux</summary>
>Look into installing some form of libsqlite3-dev or sqlite-autoconf-dev or similar for your distro.
></details> 
></details> 


It can be launched with `--path ../../examples/server/public_mikupad --sql-save-file [...]` with an optional `--sqlite-zstd-ext-file [...]`. 


The path serves the index.html, but the methods the endpoint rely on are only enabled when a `sql-save-file` is passed.

The provided mikupad file is built on top of https://github.com/lmg-anon/mikupad/pull/113 but additionally it streamlines the code (and UI sections), by removing support for other LLM endpoints and data storage models but additionally has the following:

- Add a sidebar section and pop out modal for managing save slots and KV cache slots (new standard feature).
- Add a sidebar section to manage Database Compression (new optional feature [will only be turned on if `--sqlite-zstd-ext-file` is passed] )
- Add a second list of auto-grouped sessions and made the sidebar and sessions sections resizable
- Add an Export All button (to complement the import many functionality I contributed to mikupad)
- add top-n σ sampler
- fixed a longstanding bug with highlight misalignment (using the fix in this comment: https://github.com/lmg-anon/mikupad/issues/78#issuecomment-2164023253)
- move current page storage from database to browser URL




The compression feature requires supports dynamically loading [phiresky/sqlite-zstd](https://github.com/phiresky/sqlite-zstd) which for allows one to use compressed sql databases, results may vary but for me it is very useful:

size before | size after | row count
--|--|--
31.04GB | 3.40GB | 14752
8.62GB | 581.33MB | 8042
12.54 GB | 2.04 GB | 1202
30.54 GB | 5.02 GB | 6180


[sqlite_modern_cpp](https://github.com/SqliteModernCpp/sqlite_modern_cpp) will be built if `-LLAMA_SERVER_SQLITE3=ON`.

Potential future roadmap items:
- [ ] Add a mode that creates new sessions on branching or prediction
- [ ] SQLite Wasm option (this would allow for you to choose to save to the server or the browser)
- [ ] Add themes to the database, and add a user friendly way to create and manage them
- [ ] Add better way to organize sessions together as opposed to the current solution of just grouping those with the same name
- [ ] Add the ability to mask tokens from being processed (for use with think tokens as they are supposed to be removed once the response is finished).
- [ ] Add generic template tokens (for system prompt, and instruct), making stored prompts more flexible with different templates.
- [ ] max content length should be obtained from server (based on `n_ctx`) and not from user input, and also changing or even removing the usage of that variable (or just from the UI). It is used for setting maximums for Penalty Range for some samplers (useful but could be frustrating if set wrong as knowing that is not very clear), and to truncate the context in some situations potentially.
- [ ] Allow for slot saves to be in the database. This would allow for it to be compressed (similar to prompts there can often be a lot of redundancy between saves). Edit: This may not be as useful as expected.
- [ ] Move template selected to sampling, and make sampling have it's own saves like sessions (and available templates) do.  This would make it easy to have preset profiles of templates/sampler. The downside of this, it would break import/export functionality with older versions and I am not sure if would even be a better experience let alone worth that tradeoff.

---

## 💬 Conversation

👤 **saood06** commented on **2025-06-28** at **01:46:03**

Now that I have removed the hardcoded extension loading, I do think this is in a state where it can be used by others (and potentially provide feedback), but I will still be working on completing things from the "To-do" list above until it is ready for review (and will update the post above).

---

👤 **ubergarm** commented on **2025-06-30** at **14:34:30**

Heya @saood06 I had some time this morning to kick the tires on this PR.

My high level understanding is that this PR adds new web endpoint for Mikupad as an alternative to the default built-in web interface.

I don't typically use the built-in web interface, but I did by mest to try it out. Here is my experience:

<details>

<summary>👈logs and screenshots</summary>

```bash
# get setup
$ cd ik_llama.cpp
$ git fetch upstream
$ git checkout s6/mikupad
$ git rev-parse --short HEAD
3a634c7a

# i already had the sqllite OS level lib installed apparently:
$ pacman -Ss libsql
core/sqlite 3.50.2-1 [installed]
    A C library that implements an SQL database engine

# compile
$ cmake -B build -DGGML_CUDA=ON -DGGML_VULKAN=OFF -DGGML_RPC=OFF -DGGML_BLAS=OFF -DGGML_CUDA_F16=ON -DGGML_SCHED_MAX_COPIES=1 -DGGML_CUDA_IQK_FORCE_BF16=1
$ cmake --build build --config Release -j $(nproc)
```

Then I tested my usual command like so:
```bash
# run llama-server
model=/mnt/astrodata/llm/models/ubergarm/Qwen3-14B-GGUF/Qwen3-14B-IQ4_KS.gguf
CUDA_VISIBLE_DEVICES="0" \
  ./build/bin/llama-server \
    --model "$model" \
    --alias ubergarm/Qwen3-14B-IQ4_KS \
    -fa \
    -ctk f16 -ctv f16 \
    -c 32768 \
    -ngl 99 \
    --threads 1 \
    --host 127.0.0.1 \
    --port 8080
```

When I open a browser to 127.0.0.1:8080 I get a nice looking Web UI that is simple and sleek with a just a few options for easy quick configuring:

![ik_llama-saood06-mikupad-pr558](https://github.com/user-attachments/assets/4c294d58-a60c-4eb5-ad80-d5b1dc12f6f5)


Then I added the extra arguments you mention above and run again:
```bash
# run llama-server
model=/mnt/astrodata/llm/models/ubergarm/Qwen3-14B-GGUF/Qwen3-14B-IQ4_KS.gguf
CUDA_VISIBLE_DEVICES="0" \
  ./build/bin/llama-server \
    --model "$model" \
    --alias ubergarm/Qwen3-14B-IQ4_KS \
    -fa \
    -ctk f16 -ctv f16 \
    -c 32768 \
    -ngl 99 \
    --threads 1 \
    --host 127.0.0.1 \
    --port 8080 \
    --path ./examples/server/public_mikupad \
    --sql-save-file sqlite-save.sql
```

This time a different color background appears but seems throw an async error in the web debug console as shown in this screenshot:

![ik_llama-saood06-mikupad-pr558-test-2](https://github.com/user-attachments/assets/19dc38f3-e36c-4479-b4fa-4166fe0574ef)

The server seems to be throwing 500's so maybe I didn't go to the correct endpoint or do I need to do something else to properly access it?

```bash
NFO [                    init] initializing slots | tid="140147414781952" timestamp=1751293931 n_slots=1
INFO [                    init] new slot | tid="140147414781952" timestamp=1751293931 id_slot=0 n_ctx_slot=32768
INFO [                    main] model loaded | tid="140147414781952" timestamp=1751293931
INFO [                    main] chat template | tid="140147414781952" timestamp=1751293931 chat_example="<|im_start|>system\nYou are a helpful assistant<|im_end|>\n<|im_start|>user\nHello<|im_end|>\n<|im_start|>assistant\nHi there<|im_end|>\n<|im_start|>user\nHow are you?<|im_end|>\n<|im_start|>assistant\n" built_in=true
INFO [                    main] HTTP server listening | tid="140147414781952" timestamp=1751293931 n_threads_http="31" port="8080" hostname="127.0.0.1"
INFO [            update_slots] all slots are idle | tid="140147414781952" timestamp=1751293931
INFO [      log_server_request] request | tid="140145881767936" timestamp=1751293939 remote_addr="127.0.0.1" remote_port=54320 status=200 method="GET" path="/" params={}
INFO [      log_server_request] request | tid="140145881767936" timestamp=1751293939 remote_addr="127.0.0.1" remote_port=54320 status=200 method="GET" path="/version" params={}
INFO [      log_server_request] request | tid="140145881767936" timestamp=1751293939 remote_addr="127.0.0.1" remote_port=54320 status=500 method="POST" path="/load" params={}
INFO [      log_server_request] request | tid="140145873375232" timestamp=1751293944 remote_addr="127.0.0.1" remote_port=54336 status=200 method="GET" path="/" params={}
INFO [      log_server_request] request | tid="140145873375232" timestamp=1751293944 remote_addr="127.0.0.1" remote_port=54336 status=200 method="GET" path="/version" params={}
INFO [      log_server_request] request | tid="140145873375232" timestamp=1751293944 remote_addr="127.0.0.1" remote_port=54336 status=500 method="POST" path="/load" params={}
INFO [      log_server_request] request | tid="140145873375232" timestamp=1751293944 remote_addr="127.0.0.1" remote_port=54336 status=404 method="GET" path="/favicon.ico" params={}
```
</details>

---

👤 **Downtown-Case** commented on **2025-06-30** at **15:42:14**

I am interested in this.

Mikupad is *excellent* for testing prompt formatting and sampling, with how it shows logprobs over generated tokens. It's also quite fast with big blocks of text.

---

👤 **saood06** commented on **2025-06-30** at **18:30:02**

> I am interested in this.
> 
> Mikupad is _excellent_ for testing prompt formatting and sampling, with how it shows logprobs over generated tokens. It's also quite fast with big blocks of text.

Glad to hear it. I agree. I love being able to see probs for each token (and even be able to pick a replacement from the specified tokens).

If you are an existing mikupad user you may need to use the DB migration script I put in https://github.com/lmg-anon/mikupad/pull/113 if you want to migrate a whole database, migrating individual sessions via import and export should work just fine I think.

>This time a different color background appears but seems throw an async error in the web debug console as shown in this screenshot:
>...
>The server seems to be throwing 500's so maybe I didn't go to the correct endpoint or do I need to do something else to properly access it?

You are doing the correct steps, I was able to reproduce the issue of not working with a fresh sql file (so far my testing was done with backup databases with existing data). Thanks for testing, I'll let you know when it works so that you can test it again if you so choose.

---

👤 **ubergarm** commented on **2025-06-30** at **19:41:28**

> You are doing the correct steps, I was able to reproduce the issue of not working with a fresh sql file (so far my testing was done with backup databases with existing data). Thanks for testing, I'll let you know when it works so that you can test it again if you so choose.

Thanks for confirming, correct I didn't have a `.sql` file already in place but just made up that name. Happy to try again whenever u are ready!

---

👤 **saood06** commented on **2025-06-30** at **19:54:11**

> Thanks for confirming, correct I didn't have a `.sql` file already in place but just made up that name. Happy to try again whenever u are ready!

Just pushed a fix. ( The issue was with something that is on my to-do list to refactor and potentially remove but for now a quick fix for the code as is).

Edit: The fix is in the html only so no compile or even relaunch needed just a reload should fix it

---

👤 **ubergarm** commented on **2025-06-30** at **22:34:56**

@saood06 

Aye! It fired right up this time and I was able to play with it a little and have a successful generation. It is cool how it I can mouse over the tokens to see the probabilities!

![mikupad-testing-works](https://github.com/user-attachments/assets/5dfee61a-6915-45ba-abe2-652163c9a67a)

---

👤 **saood06** commented on **2025-06-30** at **22:40:08**

> Aye! It fired right up this time and I was able to play with it a little and have a successful generation. 

Nice.

>It is cool how it I can mouse over the tokens to see the probabilities!

Yes, I like to turn on the "Color by probability" to be able to see low probability tokens at a glance.

It might also be useful to you for benchmarking quants or models (saving and cloning prompts).

---

👤 **ikawrakow** commented on **2025-07-02** at **08:09:57**

This is getting surprisingly little testing. Nevertheless we can merge whenever @saood06 feels it is ready and removes the "draft" label.

---

👤 **saood06** commented on **2025-07-25** at **00:55:21**

This post is from before I finished implementing and polishing the UI.

The new resizable sessions section (`All` group is always on top, and contains all prompts, number is how many prompts in that group ):
![image](https://github.com/user-attachments/assets/c52040cc-b0d6-4759-9250-36d7ee24157a)

Managing prompts from disk cache:
<img width="1881" height="306" alt="image" src="https://github.com/user-attachments/assets/343c8954-265f-4dbc-bba2-f14c8ffd82c3" />

The left panel is adjustable in width, and the entire thing is adjustable in height. (Note: total width is fixed, but total height is adjustable).

Edit: The image is outdated, renaming support and the save tab now exist.

The icons for sorting are in order: name, token count, file size, and modified date. I hope the icons make that clear but they also say what they do on hover:

<img width="135" height="62" alt="image" src="https://github.com/user-attachments/assets/27e37def-ebb9-487c-8ee0-a58f117aa379" />

The sidebar which includes the button to open what is shown above alongside Database compression management:

<img width="675" height="341" alt="image" src="https://github.com/user-attachments/assets/7ce869f0-f836-4d6f-9d95-230ce4a576eb" />

The enable button will swap to an update button once you enable compression

Custom button when clicked shows this:

<img width="676" height="93" alt="image" src="https://github.com/user-attachments/assets/a866b33d-a496-4693-84f8-d286e69c8230" />

---

👤 **saood06** commented on **2025-08-11** at **10:10:12**

>I have yet to push a commit with it because although most things are functional, there are still bugs and some missing functionality.

I have now pushed my changes (the UI for compression, KV cache manipulation, and the export all button are now implemented and fully functional). This PR is now feature complete, but leaving it in draft as it won't compile for people who do not have sqlite3 installed (fixing this is the last on my To-do list).

---

👤 **saood06** commented on **2025-08-24** at **09:22:42**

@ikawrakow 

It is finally ready for review.

@firecoperana 

This adds endpoints that you may find useful to utilize in the UI you maintain.

---

👤 **ikawrakow** started a conversation on `examples/server/CMakeLists.txt` on **2025-08-24** at **12:35:54**

This is not required, right? I don't have SQLite3 installed, so the build fails because of this line. If I remove it, build succeeds.

> 👤 **saood06** replied on **2025-08-24** at **12:40:31**
> 
> I forgot to remove this line. See a few lines down I moved it into the if block

> 👤 **saood06** replied on **2025-08-24** at **12:44:04**
> 
> Pushed a commit removing the line

---

👤 **ikawrakow** approved this pull request ✅ on **2025-08-24** at **12:51:56**