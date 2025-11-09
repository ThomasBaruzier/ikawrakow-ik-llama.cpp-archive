## 🔀 [Pull Request #661](https://github.com/ikawrakow/ik_llama.cpp/pull/661) - Fix Qwen3 content extraction breaking code formatting

| **Author** | `iSevenDays` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `tool-calls-bug-fix` |
| **Target Branch** | `main` |
| **Created** | 2025-07-29 |
| **Updated** | 2025-08-07 |
| **Merged** | 2025-08-07 |

---

## 📄 Description

- [x] I have read the [contributing guidelines](https://github.com/ggerganov/llama.cpp/blob/master/CONTRIBUTING.md)
- Self-reported review complexity:
  - [x] Low
  - [ ] Medium
  - [ ] High


## Summary

Fixes a bug in Qwen3 content extraction where aggressive whitespace cleanup was breaking proper code formatting in non-tool-call streaming output.

## Problem

The `qwen3::extract_content_during_parsing()` function was using an aggressive regex pattern that collapsed multiple newlines:
```cpp
content = std::regex_replace(content, std::regex(R"(\n\s*\n)"), "\n");
```

This broke:
- **PEP 8 compliance**: 2 empty lines between Python functions were collapsed to 1
- **Code formatting**: Any intentional whitespace structure was lost
- **Streaming output**: Users requesting properly formatted code received malformed results

## Root Cause

The regex `R"(\n\s*\n)"` replaces any sequence of newline + whitespace + newline with a single newline, which is overly aggressive for content that should preserve formatting.

## Solution

✅ **Replace aggressive cleanup with gentle trimming**
- Remove `std::regex_replace(content, std::regex(R"(\n\s*\n)"), "\n")`
- Replace with `string_strip(content)` following original llama.cpp patterns
- Only trim leading/trailing whitespace, preserve internal formatting

✅ **Add proper include**
- Add `#include "../../common/common.h"` for `string_strip` access

✅ **Add regression test**
- New `test_qwen3_whitespace_preservation()` validates PEP 8 compliance
- Ensures 2 empty lines between functions are preserved

## Testing

- ✅ **PEP 8 compliance**: 2 empty lines between functions preserved
- ✅ **Tool call parsing**: All Qwen3 tests continue to pass
- ✅ **No regressions**: Existing functionality maintained
- ✅ **Follows patterns**: Matches original llama.cpp whitespace handling

## Example

**Before (broken):**
```python
def celsius_to_fahrenheit(celsius):
    return celsius * 9/5 + 32
def fahrenheit_to_celsius(fahrenheit):  # Missing empty lines\!
    return (fahrenheit - 32) * 5/9
```

**After (fixed):**
```python
def celsius_to_fahrenheit(celsius):
    return celsius * 9/5 + 32


def fahrenheit_to_celsius(fahrenheit):  # Proper PEP 8 spacing preserved\!
    return (fahrenheit - 32) * 5/9
```

## Files Changed

- `examples/server/parsers/qwen3_parser.hpp` - Fix whitespace handling
- `tests/test-function-calls.cpp` - Add regression test

## Test Results

```
🧹 Testing Qwen3 Whitespace Preservation Fix:
✅ PASS: Qwen3: PEP 8 double empty lines preserved
✅ PASS: Qwen3: Content not empty after processing
✅ PASS: Qwen3: Function content preserved
✅ PASS: Qwen3: Second function preserved
```

---

## 💬 Conversation

👤 **gopinath87607** commented on **2025-08-02** at **03:11:22**

hi i tested this pr in vs code continue extension with recent unsolth quant qwen 30b a3b  coder but its still breaking when its calling second time tool and main  repo not even working

update

 looks like model itself there is a error let me check again

---

👤 **gopinath87607** commented on **2025-08-02** at **03:50:08**

https://huggingface.co/unsloth/Qwen3-Coder-30B-A3B-Instruct-GGUF/discussions/1

kindly check this discussion 

update this pull
https://github.com/ggml-org/llama.cpp/pull/14962

---

👤 **iSevenDays** commented on **2025-08-02** at **08:16:51**

@gopinath87607 here is my latest branch updates
https://github.com/ikawrakow/ik_llama.cpp/pull/670
I found some issues and differences between llama.cpp and ik_llama.cpp and fixed them. That's probably not everything, but works much stable.

---

👤 **Danmoreng** commented on **2025-08-02** at **10:08:30**

Apparently qwen3 coder uses a different tool calling structure with XML instead of json, llama.cpp currently ads the functionality here: https://github.com/ggml-org/llama.cpp/issues/15012

---

👤 **gopinath87607** commented on **2025-08-02** at **11:01:33**

> @gopinath87607 here is my latest branch updates [#670](https://github.com/ikawrakow/ik_llama.cpp/issues/670) I found some issues and differences between llama.cpp and ik_llama.cpp and fixed them. That's probably not everything, but works much stable.

thanks mate i will check it do we need to use any special tag for tool activation like jinja

---

👤 **ubergarm** commented on **2025-08-02** at **14:34:54**

Some more discussion on similar qwen with toolcalling stuff here: https://huggingface.co/bartowski/Qwen_Qwen3-235B-A22B-Instruct-2507-GGUF/discussions/1

---

👤 **ikawrakow** approved this pull request ✅ on **2025-08-07** at **05:21:55**