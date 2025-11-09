## 🔀 [Pull Request #670](https://github.com/ikawrakow/ik_llama.cpp/pull/670) - [DRAFT] Function call updates 

| **Author** | `iSevenDays` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Source Branch** | `development` |
| **Target Branch** | `main` |
| **Created** | 2025-08-02 |
| **Updated** | 2025-09-26 |

---

## 📄 Description

- [x] I have read the [contributing guidelines](https://github.com/ggerganov/llama.cpp/blob/master/CONTRIBUTING.md)
- Self-reported review complexity:
  - [ ] Low
  - [ ] Medium
  - [x] High

---

## 💬 Conversation

👤 **gopinath87607** commented on **2025-08-02** at **10:59:36**

will check this and update you

---

👤 **gopinath87607** commented on **2025-08-03** at **10:06:31**

[299/300] Building CXX object examples...akeFiles/llama-server.dir/server.cpp.o
FAILED: examples/server/CMakeFiles/llama-server.dir/server.cpp.o 
/usr/bin/c++ -DGGML_USE_CUDA -DSERVER_VERBOSE=1 -I/home/gopi/ik_llama.cpp/examples -I/home/gopi/ik_llama.cpp/examples/server -I/home/gopi/ik_llama.cpp/build/examples/server -I/home/gopi/ik_llama.cpp -I/home/gopi/ik_llama.cpp/common/. -I/home/gopi/ik_llama.cpp/src/. -I/home/gopi/ik_llama.cpp/src/../include -I/home/gopi/ik_llama.cpp/ggml/src/../include -O3 -DNDEBUG -std=gnu++17 -MD -MT examples/server/CMakeFiles/llama-server.dir/server.cpp.o -MF examples/server/CMakeFiles/llama-server.dir/server.cpp.o.d -o examples/server/CMakeFiles/llama-server.dir/server.cpp.o -c /home/gopi/ik_llama.cpp/examples/server/server.cpp
/home/gopi/ik_llama.cpp/examples/server/server.cpp: In member function ‘void server_context::send_final_response(server_slot&)’:
/home/gopi/ik_llama.cpp/examples/server/server.cpp:1773:23: error: expected primary-expression before ‘<<’ token
 1773 |                       << " tool_calls, first_id='" << slot.current_msg.tool_calls[0].id << "'" << std::endl;
      |                       ^~
ninja: build stopped: subcommand failed.
gopi@gopi-X99:~/ik_llama.cpp$ 

i am getting this error 

here is what claude said 

This is a C++ compilation error in the llama.cpp server code. The error is occurring on line 1773 of server.cpp where there's a syntax issue with stream operations.
The error message indicates:
error: expected primary-expression before '<<' token
This typically happens when there's a missing left-hand side operand for the stream insertion operator (<<). Looking at the problematic line:
cpp<< " tool_calls, first_id='" << slot.current_msg.tool_calls[0].id << "'" << std::endl;
The issue is that this line starts with <<, which means it should be continuing a stream operation from the previous line, but something is wrong with the syntax.
To fix this, you need to check the lines immediately before line 1773:

Missing stream object: The previous line might be missing a stream object like std::cout, std::cerr, or a logging macro
Missing semicolon: The previous line might be missing a semicolon, causing this line to be interpreted incorrectly
Broken line continuation: There might be a syntax issue in how the multi-line stream operation is structured

Common fixes:

Add the missing stream object (if the previous line doesn't have one):
cppstd::cout << " tool_calls, first_id='" << slot.current_msg.tool_calls[0].id << "'" << std::endl;

Or if it's part of a logging system, it might need something like:
cppLOG() << " tool_calls, first_id='" << slot.current_msg.tool_calls[0].id << "'" << std::endl;

Check the previous line to ensure it properly starts the stream operation.

To see the exact issue, you could look at lines 1770-1775 in the file:
bashsed -n '1770,1775p' /home/gopi/ik_llama.cpp/examples/server/server.cpp
This will show you the context around the error line so you can see what's missing.

---

👤 **ikawrakow** commented on **2025-08-20** at **04:33:05**

@iSevenDays  This PR is still in draft. Do you intend to finish it?

---

👤 **ikawrakow** commented on **2025-09-26** at **16:25:03**

This PR hasn't seen any progress for two months and is now outdated -> closing.