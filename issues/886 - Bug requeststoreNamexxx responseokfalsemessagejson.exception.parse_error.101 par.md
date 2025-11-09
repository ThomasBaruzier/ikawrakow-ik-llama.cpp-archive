## 📌 [Issue #886](https://github.com/ikawrakow/ik_llama.cpp/issues/886) - Bug: request="storeName=xxx" response="{\"ok\":false,\"message\":\"[json.exception.parse_error.101] parse error at line 1, column 1: syntax error while parsing value - invalid literal; last read: 's'\>

| **Author** | `magikRUKKOLA` |
| :--- | :--- |
| **State** | ✅ **Open** |
| **Created** | 2025-11-01 |
| **Updated** | 2025-11-01 |

---

## 📄 Description

### What happened?

How do I dump the data into the sqlite3 database ?

Somehow I can't figure out how to send the data with the name of the store to the server for it to save the data into the database.

```bash
curl -v -X POST --data-raw '{ "storeName": "xxx" }' localhost:8080/save
```

### Name and Version

 /opt/ik_llama.cpp/ik_llama.cpp/build/bin/llama-server --version
version: 3944 (58922c23)
built with cc (Debian 15.2.0-7) 15.2.0 for x86_64-linux-gnu

### What operating system are you seeing the problem on?

_No response_

### Relevant log output

```shell
Nov 01 02:01:42 lenovo run-ik_llama.cpp.sh[538314]: VERB [      log_server_request] request | tid="139977247657984" timestamp=1761962502 request="storeName=xxx" response="{\"ok\":false,\"message\":\"[json.exception.parse_error.101] parse error at line 1, column 1: syntax error while parsing value - invalid literal; last read: 's'\>
Nov 01 02:01:56 lenovo run-ik_llama.cpp.sh[538314]: /opt/ik_llama.cpp/ik_llama.cpp/common/../vendor/nlohmann/json.hpp:22188: GGML_ASSERT(it != m_data.m_value.object->end()) failed
Nov 01 02:01:56 lenovo run-ik_llama.cpp.sh[539052]: gdb: warning: Couldn't determine a path for the index cache directory.
Nov 01 02:01:56 lenovo run-ik_llama.cpp.sh[539052]: [New LWP 538990]
Nov 01 02:01:56 lenovo run-ik_llama.cpp.sh[539052]: [New LWP 538560]
Nov 01 02:01:56 lenovo run-ik_llama.cpp.sh[539052]: [New LWP 538559]
Nov 01 02:01:56 lenovo run-ik_llama.cpp.sh[539052]: [New LWP 538558]
Nov 01 02:01:56 lenovo run-ik_llama.cpp.sh[539052]: [New LWP 538557]
...

Nov 01 02:04:21 lenovo run-ik_llama.cpp.sh[539650]: [New LWP 539180]
Nov 01 02:04:21 lenovo run-ik_llama.cpp.sh[539650]: [New LWP 539179]
Nov 01 02:04:21 lenovo run-ik_llama.cpp.sh[539650]: [New LWP 539178]
Nov 01 02:04:21 lenovo run-ik_llama.cpp.sh[539650]: [Thread debugging using libthread_db enabled]
Nov 01 02:04:21 lenovo run-ik_llama.cpp.sh[539650]: Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".
Nov 01 02:04:21 lenovo run-ik_llama.cpp.sh[539650]: __syscall_cancel_arch () at ../sysdeps/unix/sysv/linux/x86_64/syscall_cancel.S:56
Nov 01 02:04:21 lenovo run-ik_llama.cpp.sh[539650]: warning: 56        ../sysdeps/unix/sysv/linux/x86_64/syscall_cancel.S: No such file or directory
Nov 01 02:04:21 lenovo run-ik_llama.cpp.sh[539650]: #0  __syscall_cancel_arch () at ../sysdeps/unix/sysv/linux/x86_64/syscall_cancel.S:56
Nov 01 02:04:21 lenovo run-ik_llama.cpp.sh[539650]: 56        in ../sysdeps/unix/sysv/linux/x86_64/syscall_cancel.S
Nov 01 02:04:21 lenovo run-ik_llama.cpp.sh[539650]: #1  0x00007fd511899668 in __internal_syscall_cancel (a1=<optimized out>, a2=<optimized out>, a3=<optimized out>, a4=<optimized out>, a5=a5@entry=0, a6=a6@entry=4294967295, nr=202) at ./nptl/cancellation.c:49
Nov 01 02:04:21 lenovo run-ik_llama.cpp.sh[539650]: warning: 49        ./nptl/cancellation.c: No such file or directory
Nov 01 02:04:21 lenovo run-ik_llama.cpp.sh[539650]: #2  0x00007fd511899c9c in __futex_abstimed_wait_common64 (private=0, futex_word=0x7fff1d0584e0, expected=<optimized out>, op=<optimized out>, abstime=0x0, cancel=true) at ./nptl/futex-internal.c:57
Nov 01 02:04:21 lenovo run-ik_llama.cpp.sh[539650]: warning: 57        ./nptl/futex-internal.c: No such file or directory
Nov 01 02:04:21 lenovo run-ik_llama.cpp.sh[539650]: #3  __futex_abstimed_wait_common (futex_word=futex_word@entry=0x7fff1d0584e0, expected=<optimized out>, clockid=clockid@entry=0, abstime=abstime@entry=0x0, private=private@entry=0, cancel=cancel@entry=true) at ./nptl/futex-internal.c:87
Nov 01 02:04:21 lenovo run-ik_llama.cpp.sh[539650]: 87        in ./nptl/futex-internal.c
Nov 01 02:04:21 lenovo run-ik_llama.cpp.sh[539650]: #4  0x00007fd511899cfb in __GI___futex_abstimed_wait_cancelable64 (futex_word=futex_word@entry=0x7fff1d0584e0, expected=<optimized out>, clockid=clockid@entry=0, abstime=abstime@entry=0x0, private=private@entry=0) at ./nptl/futex-internal.c:139
Nov 01 02:04:21 lenovo run-ik_llama.cpp.sh[539650]: 139        in ./nptl/futex-internal.c
Nov 01 02:04:21 lenovo run-ik_llama.cpp.sh[539650]: #5  0x00007fd51189c158 in __pthread_cond_wait_common (cond=0x7fff1d0584c0, mutex=0x7fff1d058498, clockid=0, abstime=0x0) at ./nptl/pthread_cond_wait.c:426
Nov 01 02:04:21 lenovo run-ik_llama.cpp.sh[539650]: warning: 426        ./nptl/pthread_cond_wait.c: No such file or directory
Nov 01 02:04:21 lenovo run-ik_llama.cpp.sh[539650]: #6  ___pthread_cond_wait (cond=0x7fff1d0584c0, mutex=0x7fff1d058498) at ./nptl/pthread_cond_wait.c:458
Nov 01 02:04:21 lenovo run-ik_llama.cpp.sh[539650]: 458        in ./nptl/pthread_cond_wait.c
Nov 01 02:04:21 lenovo run-ik_llama.cpp.sh[539650]: #7  0x0000558fd6230017 in server_queue::start_loop() ()
Nov 01 02:04:21 lenovo run-ik_llama.cpp.sh[539650]: #8  0x0000558fd61aacdf in main ()
Nov 01 02:04:21 lenovo run-ik_llama.cpp.sh[539650]: [Inferior 1 (process 539177) detached]
```