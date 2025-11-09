## 🔀 [Pull Request #798](https://github.com/ikawrakow/ik_llama.cpp/pull/798) - Port mdmd from mainline + Qwen2/2.5-VL support

| **Author** | `ikawrakow` |
| :--- | :--- |
| **State** | 🔀 **Merged** |
| **Source Branch** | `ik/add_mtmd` |
| **Target Branch** | `main` |
| **Created** | 2025-09-25 |
| **Updated** | 2025-09-28 |
| **Merged** | 2025-09-27 |

---

## 📄 Description

This PR is a port of mainline's `mtmd` library and multi-modal command line tool `llama-mtmd-cli`, along with implementation of Qwen2/2.5-VL support.

Based on my own testing it seems fully functional.

Please test and provide feedback!

---

### Original WIP description

This is WIP to port `mdmd` and `mdmd-cli` from mainline.

Current state: ~compiles, but not functional (missing several `ggml` ops)~
* [x] `mtmd`-related ops have been added to the CPU and CUDA back-ends
* [x] The `mtmd` library and `mtmd-cli` tools have been ported
* [x] Support for Qwen2/2.5-VL was added
* [x] Tests with the example image `examples/mtmd/test-1.jpeg` produce a meaningful response

Here an example run with the CPU back-end:
```
./bin/llama-mtmd-cli -m Qwen2.5Vl-7.6B-Q9_0gguf --mmproj mmproj-qwen2.5vl --image ../examples/mtmd/test-1.jpeg -c 8192 -n 8192 -s 5678 -t 16 -fa -p " "
...
load_hparams: model size:         1291.40 MiB
load_hparams: metadata size:      0.18 MiB
alloc_compute_meta:        CPU compute buffer size =     3.60 MiB
encoding image slice...
image slice encoded in 2042 ms
decoding image batch 1/1, n_tokens_batch = 414
image decoded (batch 1/1) in 1210 ms
The image shows the front page of The New York Times from July 21, 1969. The headline reads
"MEN WALK ON MOON: ASTRONAUTS LAND ON PLAIN; COLLECT ROCKS, PLANT FLAG."
This historic front page announces the Apollo 11 moon landing, a significant event in human history.
The page includes a photograph of the lunar module and a report on the astronauts' activities on
the moon. The New York Times was one of the first major newspapers to cover this event, providing
detailed reports and images of the historic mission.

llama_print_timings:        load time =     766.57 ms
llama_print_timings:      sample time =       4.76 ms /   122 runs   (    0.04 ms per token, 25603.36 tokens per second)
llama_print_timings: prompt eval time =    3494.36 ms /   425 tokens (    8.22 ms per token,   121.62 tokens per second)
llama_print_timings:        eval time =   15474.56 ms /   121 runs   (  127.89 ms per token,     7.82 tokens per second)
llama_print_timings:       total time =   19437.51 ms /   546 tokens
```


The same thing with mainline:
```
load_hparams: model size:         1291.40 MiB
load_hparams: metadata size:      0.18 MiB
alloc_compute_meta:        CPU compute buffer size =     3.60 MiB
main: loading model: ../../ik_llama.cpp/ncuda/junk.bin
encoding image slice...
image slice encoded in 4553 ms
decoding image batch 1/1, n_tokens_batch = 414
image decoded (batch 1/1) in 2729 ms

The image shows the front page of The New York Times from July 21, 1969. The headline reads
"MEN WALK ON MOON" in large, bold letters, indicating a significant event. Below the headline,
there are two photographs, though the details of the images are not clear from this description.
The article discusses the historic landing of astronauts on the moon, mentioning that they landed
on a plain, collected rocks, and planted a flag. The article is dated Monday, July 21, 1969, which
corresponds to the day of the Apollo 11 moon landing. The newspaper is priced at 10 cents, which
was the cost of a New York Times in 1969.


llama_perf_context_print:        load time =     909.82 ms
llama_perf_context_print: prompt eval time =    7653.99 ms /   425 tokens (   18.01 ms per token,    55.53 tokens per second)
llama_perf_context_print:        eval time =   19592.87 ms /   153 runs   (  128.06 ms per token,     7.81 tokens per second)
llama_perf_context_print:       total time =   27709.67 ms /   578 tokens
llama_perf_context_print:    graphs reused =          0
```

Interesting to see that image encoding/decoding in `ik_llama.cpp` is 2X the speed of mainline without me having done anything related to this part of the calculation.

### TODO

* [ ] Testing by others
* [x] The conversation mode does not seem to be working - works, it was just a matter of `LOG` vs `LOG_TEE`
* [ ] Implement more vision models - preferences?

### More observations

* Tested with a bunch of additional images
* Seems to work fine on CUDA
* Interestingly enough, given a portion of a photograph of my passport, it correctly recognizes my place of birth even though only 5 out of 12 letters are visible. It hallucinates my birth year, which is not visible on the photograph.
* ~Something is not quite right on the CPU when the number of image tokens is larger than the u-batch size. For the passport photo it generates 1036 image tokens, and I get gibberish unless I set `u-batch = batch = 2048`. This is strange because if something was wrong with setting the token positions (different than usual for Qwen2-VL), it shouldn't be working on CUDA either, but it does.~ Fixed with last commit
* ~Something is not quite right also on CUDA. The first image works fine, but any attempt to add a second image to the conversation leads to gibberish.~ Fixed with last commit

---

## 💬 Conversation

👤 **Ph0rk0z** commented on **2025-09-27** at **14:35:27**

So does using it with the server require more implementations?

---

👤 **ikawrakow** commented on **2025-09-27** at **14:40:09**

Nothing has been done for the server. You have a command line tool. You can use it like this
```
./bin/llama-mtmd-cli -m model --mmproj mmproj $other_args
...
> /image some_image.jpg
> Describe the image
...
```
(see the `mtmd` documentation)

For the server I'm hoping someone else will do a PR.

---

👤 **Ph0rk0z** commented on **2025-09-27** at **16:44:19**

Ok. That clears it up. I have still yet to use the command line tools :P

We have progress!

---

👤 **mcm007** commented on **2025-09-28** at **14:52:00**

Impressive!

Tested: Qwen2.5-VL-7B, gemma-3-12b and pixtral-12b on two systems with CPU+iGPU.
- All 3 models are working OK.
- Vulkan build is working as well; `encoding image slice...` (with `-ngl 0`...) is 2x faster than the CPU build.
- .png and .jpg are working.
- mmproj-f16.gguf is acepted but not mmproj-Q8_0.gguf

`version: 3899 (3d4977cb)`

---

👤 **ikawrakow** commented on **2025-09-28** at **17:11:53**

> mmproj-f16.gguf is acepted but not mmproj-Q8_0.gguf

Does mainline support quantized mmproj files? To me it looked like the convolution and `im2col` kernels only work with `f16/f32` tensors.

---

👤 **mcm007** commented on **2025-09-28** at **18:49:21**

Yes, [mmproj-Qwen2.5-VL-7B-Instruct-Q8_0.gguf](https://huggingface.co/ggml-org/Qwen2.5-VL-7B-Instruct-GGUF/blob/main/mmproj-Qwen2.5-VL-7B-Instruct-Q8_0.gguf) produces good results in mainline; it seems it needs `--no-mmproj-offload` otherwise it produces some random text each time.