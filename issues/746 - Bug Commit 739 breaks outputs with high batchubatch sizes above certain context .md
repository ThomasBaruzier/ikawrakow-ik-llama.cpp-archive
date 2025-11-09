## 📌 [Issue #746](https://github.com/ikawrakow/ik_llama.cpp/issues/746) - Bug: Commit [#739](https://github.com/ikawrakow/ik_llama.cpp/issues/739) breaks outputs with high batch/ubatch sizes above certain context usage

| **Author** | `Geechan` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-09-01 |
| **Updated** | 2025-09-02 |

---

## 📄 Description

### What happened?

Hardware: 7763 Epyc, 256GB DDR4 RAM, 2x RTX 8000 Quadros, using hybrid inferencing.

When using batch sizes greater than the default values, updating to commit [#739](https://github.com/ikawrakow/ik_llama.cpp/issues/739) breaks my outputs: they become increasingly worse as the context is filled up. At 0 context things seem fine, but then turn into complete and random 'creative gibberish' even at sane context values (4k+), almost like the model is being fed garbage instead of the actual context. The effects vary per model, but it's never coherent. Reverting to [#738](https://github.com/ikawrakow/ik_llama.cpp/issues/738) fixes the issue completely, as does using the default batch/ubatch sizes.

Tested with `-2048` and`-4096` batch size/ubatch sizes, with/without the `-rtr`, `-fmoe` flags. Tested with both GLM 4.5 and DeepSeek v3.1.

For reference, my outputs look akin to this no matter the prompt, with worsening results the further in context usage you go:

GLM:

```Intention: 1.5.3. Presenting modules 1. A module of this type can be imported from a text file. 2. Input formats can include XLS, CSV, and JSON. 3. The import process can support up to 3 million records. 4. The import process is designed to be user-friendly and intuitive, ensuring that users can successfully complete the task without difficulty. 5. The function is designed to help users identify and manage their imports effectively, ensuring that all records are imported successfully and any issues are identified. 6. The function can be used for handling large volumes of records and can be used for various business needs, such as production management, sales management, and customer management. 7. The function is also designed to help users quickly and easily complete their import tasks, reducing the need for integration and making the import process more intuitive. 8. The function supports the import of multiple modules, including production management, sales management, customer management, and other modules. 9. The function supports the import of records from different sources, including XLS, CSV, JSON, and other sources. 10. The function supports the integration of multiple modules, including production management and customer management, in the same list. 11. The function supports the import of more than 3 million records, ensuring that users can handle large volumes of records without difficulty. 12. The function supports the import of projects of all categories, including production management, sales management, and other categories. 13. table of contents: 14. Chapter 1: Introduction to Product Module 15. Chapter 2: Importing Production Management Module 16. Chapter 3: Importing Sales Management Module 17. Chapter 4: Importing Menu Management Module 18. Chapter 5: Importing Customer Management Module 19. Chapter 6: Importing Other Modules 20. Chapter 7: Handling Import Errors 21. Chapter 8: Handling Large Volumes of Records 22. Chapter 9: Conclusion 23. The function is designed to handle large volumes of records and can be handled with ease, ensuring that users can complete their tasks without difficulty.```

DeepSeek:

```It's good that you're in a good mood. What a good guy. The other day I had a good time with a friend, and it was great. However, it's a little different from what I imagined. What about you? How are you doing lately? I'm very curious.```

### Name and Version

version: 3871 (d10d90ae)
built with cc (GCC) 15.2.1 20250813 for x86_64-pc-linux-gnu

(Arch Linux, compiled through the [ik-llama.cpp-cuda AUR package](https://aur.archlinux.org/packages/ik-llama.cpp-cuda).)

Compiled with `-DGGML_BLAS=OFF` and `-DGGML_SCHED_MAX_COPIES=1` flags.

### What operating system are you seeing the problem on?

Linux

### Relevant log output

```shell
llama-server \
    --n-gpu-layers 999 --threads 48 --threads-batch 64 --batch-size 4096 --ubatch-size 4096 -rtr \
    --override-tensor "blk\.(0|1|2|3|4|5|6|7|8|9|10|11|12|13|14|15|16)\.ffn_.*=CUDA0" \
    --override-tensor "blk\.(17|18|19|20|21|22|23|24|25|26|27|28|29|30)\.ffn_.*=CUDA1" \
    --override-tensor "blk\..*_exps\.=CPU" \
    --ctx-size 36864 -fa -fmoe --port 8080 --host 0.0.0.0 \
    --model "GLM-4.5-IQ4_K-00001-of-00005.gguf" --alias GLM-4.5-IQ4_K \
    --chat-template chatglm4

llama-server \
    --n-gpu-layers 999 --threads 48 --threads-batch 64 --batch-size 4096 --ubatch-size 4096 -rtr \
    --override-tensor "blk\.(0|1|2|3|4|5|6|7|8)\.ffn_.*=CUDA0" \
    --override-tensor "blk\.(9|10|11|12|13|14)\.ffn_.*=CUDA1" \
    --override-tensor "blk\..*_exps\.=CPU" \
    --ctx-size 40448 -fa -fmoe -mla 2 -amb 1024 --port 8080 --host 0.0.0.0 \
    --model "DeepSeek-V3.1-IQ3_K-00001-of-00007.gguf" --alias "Deepseek V3.1 IQ3_K"
```

---

## 💬 Conversation

👤 **sayap** commented on **2025-09-01** at **13:06:50**

Can confirm. Got this response with GLM 4.5 since [#739](https://github.com/ikawrakow/ik_llama.cpp/issues/739):
````
```python
def main():
    # The original code seems to be a mix of different languages and frameworks.
    # It's not clear what the intended functionality is.
    # However, I can provide a simple Python function that demonstrates
    # how to handle command-line arguments and print a message.
    import sys

    def print_usage():
        print("Usage: python script.py <message>")
        sys.exit(1)

    if len(sys.argv) != 2:
        print_usage()

    message = sys.argv[1]
    print(f"Message: {message}")

if __name__ == "__main__":
    main()
```
````
The prompt is for golang 😅

---

👤 **ikawrakow** commented on **2025-09-01** at **17:11:11**

Does [#748](https://github.com/ikawrakow/ik_llama.cpp/issues/748) fix it?

---

👤 **Geechan** commented on **2025-09-01** at **18:34:19**

> Does [[#748](https://github.com/ikawrakow/ik_llama.cpp/issues/748)](https://github.com/ikawrakow/ik_llama.cpp/pull/748) fix it?

Can confirm [#748](https://github.com/ikawrakow/ik_llama.cpp/issues/748) fixes the issue completely for me, even at high batch sizes.

---

👤 **magikRUKKOLA** commented on **2025-09-01** at **22:38:59**

@ikawrakow 

Can confirm the issue exist and the [#748](https://github.com/ikawrakow/ik_llama.cpp/issues/748) fixes it for a Deepseek R1 with at least batches higher than 2k.

---

👤 **ikawrakow** commented on **2025-09-02** at **04:58:49**

Closed via [#748](https://github.com/ikawrakow/ik_llama.cpp/issues/748)