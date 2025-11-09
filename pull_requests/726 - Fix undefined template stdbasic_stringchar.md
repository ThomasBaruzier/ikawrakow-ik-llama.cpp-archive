## 🔀 [Pull Request #726](https://github.com/ikawrakow/ik_llama.cpp/pull/726) - Fix undefined template std::basic_string<char>

| **Author** | `mohangk` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `main` |
| **Target Branch** | `main` |
| **Created** | 2025-08-25 |
| **Updated** | 2025-08-25 |
| **Merged** | 2025-08-25 |

---

## 📄 Description

Getting this error when compiling on Mac with clang 17 

Simple fix, add the string header in src/llama-impl.h

- [x] I have read the [contributing guidelines](https://github.com/ggerganov/llama.cpp/blob/master/CONTRIBUTING.md)
- Self-reported review complexity:
  - [x ] Low
  - [ ] Medium
  - [ ] High

Error that I was getting before this fix


> [ 22%] Built target build_info
> In file included from /Users/mohan.krishnan/Workspace/ik_llama.cpp/src/llama-model-loader.cpp:1:
> In file included from /Users/mohan.krishnan/Workspace/ik_llama.cpp/src/llama-model-loader.h:4:
> /Users/mohan.krishnan/Workspace/ik_llama.cpp/src/llama-impl.h:48:15: error: implicit instantiation of undefined template 'std::basic_string<char>'
>    48 |     if (search.empty()) {
>       |               ^
> /Library/Developer/CommandLineTools/usr/bin/../include/c++/v1/iosfwd:248:32: note: template is declared here
>   248 |     class _LIBCPP_TEMPLATE_VIS basic_string;
>       |                                ^
> In file included from /Users/mohan.krishnan/Workspace/ik_llama.cpp/src/llama-model-loader.cpp:1:
> In file included from /Users/mohan.krishnan/Workspace/ik_llama.cpp/src/llama-model-loader.h:4:
> /Users/mohan.krishnan/Workspace/ik_llama.cpp/src/llama-impl.h:51:17: error: implicit instantiation of undefined template 'std::basic_string<char>'
>    51 |     std::string builder;
>       |                 ^
> /Library/Developer/CommandLineTools/usr/bin/../include/c++/v1/iosfwd:248:32: note: template is declared here
>   248 |     class _LIBCPP_TEMPLATE_VIS basic_string;
>       |                                ^
> In file included from /Users/mohan.krishnan/Workspace/ik_llama.cpp/src/llama-model-loader.cpp:1:
> In file included from /Users/mohan.krishnan/Workspace/ik_llama.cpp/src/llama-model-loader.h:4:
> /Users/mohan.krishnan/Workspace/ik_llama.cpp/src/llama-impl.h:52:22: error: implicit instantiation of undefined template 'std::basic_string<char>'
>    52 |     builder.reserve(s.length());
>       |                      ^
> ...
>

---

## 💬 Conversation

👤 **ikawrakow** approved this pull request ✅ on **2025-08-25** at **08:33:55**

Thanks!