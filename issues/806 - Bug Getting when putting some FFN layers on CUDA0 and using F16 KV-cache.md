## 📌 [Issue #806](https://github.com/ikawrakow/ik_llama.cpp/issues/806) - Bug: Getting "!!!!" when putting some FFN layers on CUDA0 and using F16 KV-cache

| **Author** | `sayap` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-09-28 |
| **Updated** | 2025-10-07 |

---

## 📄 Description

### What happened?

If I start llama-server with:
```
$ ~/repo/ik_llama.cpp/build/bin/llama-server -m /tank/models/ggml-org/gpt-oss-120b-gguf/gpt-oss-120b-mxfp4-00001-of-00003.gguf \
  -c 4096 -ngl 999 --no-mmap -ncmoe 26
...
llm_load_tensors: offloaded 37/37 layers to GPU
llm_load_tensors:        CPU buffer size = 42065.16 MiB
llm_load_tensors:  CUDA_Host buffer size =   586.82 MiB
llm_load_tensors:      CUDA0 buffer size = 17786.52 MiB
```
i.e. by putting the last 26 FFN layers on CPU, then a request like this:
```
$ cat /tmp/req.json 
{
  "messages": [
    {
      "role": "user",
      "content": "1\n2\n3\n4\n5\n6\n7\n8\n9\n10\n11\n12\n13\n14\n15\n16\n17\n18\n19\n20\n21\n22\n23\n24\n25\n26\n27\n28\n29\n30\n31\n32\n33\n34\n35\n36\n37\n38\n39\n40\n41\n42\n43\n44\n45\n46\n47\n48\n49\n50\n51\n52\n53\n54\n55\n56\n57\n58\n59\n60\n61\n62\n63\n64\n65\n66\n67\n68\n69\n70\n71\n72\n73\n74\n75\n76\n77\n78\n79\n80\n81\n82\n83\n84\n85\n86\n87\n88\n89\n90\n91\n92\n93\n94\n95\n96\n97\n98\n99\n100\n101\n102\n103\n104\n105\n106\n107\n108\n109\n110\n111\n112\n113\n114\n115\n116\n117\n118\n119\n120\n121\n122\n123\n124\n125\n126\n127\n128\n129\n130\n131\n132\n133\n134\n135\n136\n137\n138\n139\n140\n141\n142\n143\n144\n145\n146\n147\n148\n149\n150\n151\n152\n153\n154\n155\n156\n157\n158\n159\n160\n161\n162\n163\n164\n165\n166\n167\n168\n169\n170\n171\n172\n173\n174\n175\n176\n177\n178\n179\n180\n181\n182\n183\n184\n185\n186\n187\n188\n189\n190\n191\n192\n193\n194\n195\n196\n197\n198\n199\n200\n201\n202\n203\n204\n205\n206\n207\n208\n209\n210\n211\n212\n213\n214\n215\n216\n217\n218\n219\n220\n221\n222\n223\n224\n225\n226\n227\n228\n229\n230\n231\n232\n233\n234\n235\n236\n237\n238\n239\n240\n241\n242\n243\n244\n245\n246\n247\n248\n249\n250\n251\n252\n253\n254\n255\n256\n257\n258\n259\n260\n261\n262\n263\n264\n265\n266\n267\n268\n269\n270\n271\n272\n273\n274\n275\n276\n277\n278\n279\n280\n281\n282\n283\n284\n285\n286\n287\n288\n289\n290\n291\n292\n293\n294\n295\n296\n297\n298\n299\n300\n301\n302\n303\n304\n305\n306\n307\n308\n309\n310\n311\n312\n313\n314\n315\n316\n317\n318\n319\n320\n321\n322\n323\n324\n325\n326\n327\n328\n329\n330\n331\n332\n333\n334\n335\n336\n337\n338\n339\n340\n341\n342\n343\n344\n345\n346\n347\n348\n349\n350\n351\n352\n353\n354\n355\n"
    }
  ],
  "seed": 666,
  "temperature": 1,
  "top_k": 1,
  "max_tokens": 64,
  "stream": false
}

$ curl 127.0.0.1:8080/v1/chat/completions -d @/tmp/req.json
```
will result in a response of "!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!", the tokens with id=0.

The prompt can be anything, I just use a series of numbers to increase the size. Below a certain size, the issue doesn't happen.

A few things I have tried:

* Adding `-fa`: the issue still happens.

* Changing from the default of `-ctk f16 -ctv f16` to `-ctk bf16 -ctv bf16`: the issue still happens.

* Changing to `-ncmoe 27` (1 fewer FFN layer on CUDA0): the issue doesn't happen.

* Changing to `-ncmoe 25` (1 more FFN layer on CUDA0): the issue doesn't happen.

* Adding `-fa -ctk q8_0 -ctv q8_0`: the issue doesn't happen.

This suggests either something is wrong with non-quantized KV cache, or something is wrong with my 3090 😅 

I first experienced the issue with GLM-4.5, and then I tried to come up with the minimal and hopefully reproducible steps above with gpt-oss-120b.

### Name and Version

$ ~/repo/ik_llama.cpp/build/bin/llama-server --version
version: 3899 (3d4977cb)
built with cc (Gentoo 14.3.1_p20250801 p4) 14.3.1 20250801 for x86_64-pc-linux-gnu

### What operating system are you seeing the problem on?

Linux

### Relevant log output

```shell

```

---

## 💬 Conversation

👤 **ikawrakow** commented on **2025-09-29** at **06:52:07**

Thank you for the detailed bug report.

My GPU has 16 GiB of VRAM, so I cannot use -ncmoe 26 to reproduce for debugging. Does the issue occur with a larger even number of layers left on the CPU? (e.g., `-ncmoe 32`)

---

👤 **sayap** commented on **2025-10-07** at **14:18:35**

Confirmed to be a hardware issue. Getting a lot of errors when running `memtest_vulkan`:
```
Error found. Mode INITIAL_READ, total errors 0x61F out of 0x34000000 (0.00017962%)
Errors address range: 0x4329754D0..=0x4CABB49D3  iteration:1
values range: 0xFF64E0D7..=0x000855D7   FFFFFFFF-like count:0    bit-level stats table:
         0x0 0x1  0x2 0x3| 0x4 0x5  0x6 0x7| 0x8 0x9  0xA 0xB| 0xC 0xD  0xE 0xF
SinglIdx  78       46    |  73       79    |                 |                 
TogglCnt     276  549 564| 178             |                 |                 
1sInValu                 |                 |   2   3   22  49| 117 172  253 270
   0x1?  239 208  125  59|  33  14    1    |                 |                 

Standard 5-minute test of 1: Bus=0xC5:00 DevId=0x2204   24GB NVIDIA GeForce RTX 3090
      1 iteration. Passed  0.0528 seconds  written:   19.5GB 801.1GB/sec        checked:   22.8GB 799.2GB/sec
Error found. Mode INITIAL_READ, total errors 0x623 out of 0x34000000 (0.00018007%)
Errors address range: 0x4329754D0..=0x4CAA7DDD3  iteration:2
values range: 0xFFF20757..=0x002027FD   FFFFFFFF-like count:0    bit-level stats table:
         0x0 0x1  0x2 0x3| 0x4 0x5  0x6 0x7| 0x8 0x9  0xA 0xB| 0xC 0xD  0xE 0xF
SinglIdx  69       68    |  58       70    |                 |                 
TogglCnt     265  530 545| 231             |                 |                 
1sInValu                 |                 |       3   16  55| 111 185  237 264
   0x1?  245 191  125  77|  45  11    6    |                 |                 

Error found. Mode INITIAL_READ, total errors 0x625 out of 0x34000000 (0.00018030%)
Errors address range: 0x4329754D0..=0x4CAA7DDD3  iteration:3
values range: 0xFF5C3CD7..=0x001E2175   FFFFFFFF-like count:0    bit-level stats table:
         0x0 0x1  0x2 0x3| 0x4 0x5  0x6 0x7| 0x8 0x9  0xA 0xB| 0xC 0xD  0xE 0xF
SinglIdx  70      115    | 111       73    |                 |                 
TogglCnt     369  737 387|  80             |                 |                 
1sInValu                 |                 |                3|   4  32   61 105
   0x1?  224 269  297 226| 174 112   40  22|   3   1         |                 

Error found. Mode INITIAL_READ, total errors 0x611 out of 0x34000000 (0.00017801%)
Errors address range: 0x4329754D0..=0x4CABB49D3  iteration:4
values range: 0xFFAB38F5..=0x00701EF5   FFFFFFFF-like count:0    bit-level stats table:
         0x0 0x1  0x2 0x3| 0x4 0x5  0x6 0x7| 0x8 0x9  0xA 0xB| 0xC 0xD  0xE 0xF
SinglIdx  99       63    |  77       83    |                 |                 
TogglCnt     322  648 477| 106             |                 |                 
1sInValu                 |                 |   1   1    4  12|  24  54  113 189
   0x1?  255 289  219 178| 115  68   21   9|   1             |
```