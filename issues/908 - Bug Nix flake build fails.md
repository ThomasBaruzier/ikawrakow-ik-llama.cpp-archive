## 📌 [Issue #908](https://github.com/ikawrakow/ik_llama.cpp/issues/908) - Bug: Nix flake build fails

| **Author** | `yellowbadbeast1` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-11-06 |
| **Updated** | 2025-11-06 |

---

## 📄 Description

### What happened?

Building both the default and vulkan packages fails when using [#901](https://github.com/ikawrakow/ik_llama.cpp/issues/901) or later. (Didn't test cuda since I don't have an Nvidia card.)

### Name and Version

N/A

### What operating system are you seeing the problem on?

Linux

### Relevant log output

```shell
[user@nixos:~]$ sudo nixos-rebuild switch
[sudo] password for user:
warning: updating lock file '"/etc/nixos/flake.lock"':
• Updated input 'ik_llama':
    'github:ikawrakow/ik_llama.cpp/5b38d431ace092c63c63d3b03ec17e18dc994de5?narHash=sha256-qaktvviIN6KgzXobpe6ORyszb0bc9YErCApeRw9LYPg%3D' (2025-11-05)
  → 'github:ikawrakow/ik_llama.cpp/15159a87d45a474589602a67fed038b775494991?narHash=sha256-IBvoNDGO7koZ01kUkaC2TzlMq0tKzr5%2B8UvMXDrs1iM%3D' (2025-11-05)
building the system configuration...
error: builder for '/nix/store/xr8z5wasjrcfpzzb5l0j7ad6xzvy190p-llama-cpp-blas-0.0.0.drv' failed with exit code 1;
       last 25 log lines:
       > [167/180] Linking CXX executable bin/llama-passkey
       > [168/180] Linking CXX executable bin/llama-parallel
       > [169/180] Linking CXX executable bin/llama-mtmd-cli
       > [170/180] Linking CXX executable bin/llama-save-load-state
       > [171/180] Linking CXX executable bin/llama-sweep-bench
       > [172/180] Linking CXX executable bin/llama-speculative
       > [173/180] Linking CXX executable bin/llama-simple
       > [174/180] Linking CXX executable bin/llama-vdot
       > [175/180] Linking CXX executable bin/llama-tokenize
       > [176/180] Linking CXX executable bin/llama-q8dot
       > [177/180] Generating index_llamacpp.html.gz.hpp
       > [178/180] Generating index.html.gz.hpp
       > [179/180] Building CXX object examples/server/CMakeFiles/llama-server.dir/server.cpp.o
       > FAILED: examples/server/CMakeFiles/llama-server.dir/server.cpp.o
       > /nix/store/62zpnw69ylcfhcpy1di8152zlzmbls91-gcc-wrapper-13.3.0/bin/g++ -DGGML_USE_BLAS -DLLAMA_SHARED -DSERVER_VERBOSE=1 -I/build/source/examples -I/build/source/examples/server -I/build/source/build/examples/server -I/build/source -I/build/source/examples/server/../mtmd -I/build/source/common/. -I/build/source/common/../vendor -I/build/source/src/. -I/build/source/src/../include -I/build/source/ggml/src/../include -I/build/source/examples/mtmd/. -O3 -DNDEBUG -std=gnu++17 -MD -MT examples/server/CMakeFiles/llama-server.dir/server.cpp.o -MF examples/server/CMakeFiles/llama-server.dir/server.cpp.o.d -o examples/server/CMakeFiles/llama-server.dir/server.cpp.o -c /build/source/examples/server/server.cpp
       > /build/source/examples/server/server.cpp: In member function 'void server_context::update_slots()':
       > /build/source/examples/server/server.cpp:3140:36: error: format not a string literal and no format arguments [-Werror=format-security]
       >  3140 |                             fprintf(stdout, slot.cache_tokens.detokenize(ctx, true).c_str());
       >       |                             ~~~~~~~^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
       > /build/source/examples/server/server.cpp: In lambda function:
       > /build/source/examples/server/server.cpp:4296:20: error: format not a string literal and no format arguments [-Werror=format-security]
       >  4296 |             fprintf(stdout, prompt.get<std::string>().c_str());
       >       |             ~~~~~~~^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
       > cc1plus: some warnings being treated as errors
       > ninja: build stopped: subcommand failed.
       For full logs, run:
         nix log /nix/store/xr8z5wasjrcfpzzb5l0j7ad6xzvy190p-llama-cpp-blas-0.0.0.drv
error: 1 dependencies of derivation '/nix/store/wj65pfzvz7a1357nylmd0ks4x06i67l4-system-path.drv' failed to build
error: 1 dependencies of derivation '/nix/store/k24ff7n0d5r1426i4pbiq6gm22zalfhb-nixos-system-nixos-25.05.20251104.ca534a7.drv' failed to build
```

---

## 💬 Conversation

👤 **ikawrakow** commented on **2025-11-06** at **05:20:49**

These particular warnings are solved now on the main branch.

But more generally, unless a project has decided to treat compiler warnings as errors, warnings are warnings and not errors, irrespective of the opinion of a Nix flake.