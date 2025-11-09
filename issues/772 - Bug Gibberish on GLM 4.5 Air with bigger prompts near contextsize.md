## 📌 [Issue #772](https://github.com/ikawrakow/ik_llama.cpp/issues/772) - Bug: Gibberish on GLM 4.5 Air (with bigger prompts near contextsize?)

| **Author** | `naxan6` |
| :--- | :--- |
| **State** | ❌ **Closed** |
| **Created** | 2025-09-10 |
| **Updated** | 2025-09-24 |

---

## 📄 Description

### What happened?

I think something broke in between `46968d4ab15c00397c236376864ab6203379beae `and `b66cecca4525550fddf609bb875e58a4fad5f0ce`
I'm mostly using Thireus' builds, but do also see the same error with a version self-built out of the now most current commit `d323871ba9da2e9889161541c30dba5cd680bd09`

### What I run:
`.\llama-server.exe -c 60000 -m C:/Users/XXXXX/.lmstudio/models/ubergarm/GLM-4.5-Air-GGUF/GLM-4.5-Air-IQ4_KSS-00001-of-00002.gguf --no-mmap --mlock -sm none --host 0.0.0.0 --port 8080 --jinja --top-k 0 -ctk q8_0 -ctv q8_0 -b 4096 -ub 4096 -fa -no-fug -ngl 99 -t 14 -ot blk.44.ffn_gate_exps.=CUDA0 -ot blk.44.ffn_up_exps.=CUDA0 -ot output=CUDA0 -ot ffn_.*_exps.=CPU`

PS. `-no-fug` was my latest try, before that i always had added `-fmoe`, which worked.
### Scenario:
What I do is asking for errors in a 200kchar long "lorem ipsum", resulting in around 56kT.


### What I get:
**With up to** `46968d4ab15c00397c236376864ab6203379beae`, I got something meaningful like 
```
Der Text endet abrupt mit "Lorem ipsum 1". Der Satz ist unvollständig.
```

**But with every never Version I get some glibberish  like** 
```

https， https的用法 china https列叶 China
https_地种 [...probaby neverending]
```
or
```
!h, CDATA = CDATA, ,,c c, , , , ,,, ,, , ,c所 , , , ,d,h,c 享受在 ,c,d ( ,c,c ,c d, ,,,c ,, ,n ,n1,d,hc,n (n不可在,,n(c,n (在,, d,d,hd,d,hd,h,n,d,n,n,n ,n不可nc d (n,h,n,n h,n,n�d,hd 不可n 不可n不可( ,,n d,n,, ,n不可nc n n 不可n ,h,n (,d,n ,n不可n ,n(n不可n�n,n h不可n n n n d,n n n n n n n n n nn (n 不可n nn n n n n n,nn ,n�n n nn不可n n nn n�n nn nn n�n nn n n�,,n nn nn nn n,n�nn ,n ,nn n n n,nn ,nn nn nn nn nn nn nn nn n(nn n n,n nn nn n =,,n
```

### Sidenote:

- it seems to work for both versions with a shortened prompt with a 95kchar long "lorem ipsum", resulting in around 25kT.
- I've seen the same broken behaviour with calls from kilocode or the like - as soon as the prompt gets bigger.

Any idea, maybe some parameter that fixes it? Or did something break?
Thanks

### Name and Version

GOOD b4125:
.\llama-server.exe --version           
version: 1 (f23b10c)                                                                                                              
built with MSVC 19.44.35215.0 for 
Should be commit 46968d4ab15c00397c236376864ab6203379beae

BROKEN b4129:
.\llama-server.exe --version        
version: 1 (ce1cc30)                                                                                                              
built with MSVC 19.44.35215.0 for
Should be commit b66cecca4525550fddf609bb875e58a4fad5f0ce

### What operating system are you seeing the problem on?

Windows

### Relevant log output

```shell

```

---

## 💬 Conversation

👤 **kirnat** commented on **2025-09-11** at **08:56:31**

I  experienced something similar with full fat GLM 4.5 at longer context, it worked with smaller batch sizes. But I need to verify to be sure this is still the case.

---

👤 **naxan6** commented on **2025-09-11** at **09:12:13**

some additional info:
- Also happens with Q4_0.
- Mainline llama.cpp works with Q4_0 without a problem (but is slower).
- query_25k.txt works in current  ik_llama.cpp
- query_36k.txt is broken in current ik_llama.cpp
- both work with older commit 46968d4ab15c00397c236376864ab6203379beae

[query_25k.txt](https://github.com/user-attachments/files/22271733/query_25k.txt)

[query_36k.txt](https://github.com/user-attachments/files/22271734/query_36k.txt)

---

👤 **ikawrakow** commented on **2025-09-23** at **12:52:32**

So, [#739](https://github.com/ikawrakow/ik_llama.cpp/issues/739) broke things (see [#746](https://github.com/ikawrakow/ik_llama.cpp/issues/746)), so was reverted in [#748](https://github.com/ikawrakow/ik_llama.cpp/issues/748).  It does not work on that commit? (commit 62f5382c2b321e66d8b62e9e4db43072af71c36e)

---

👤 **ikawrakow** commented on **2025-09-23** at **14:27:13**

OK, does [#790](https://github.com/ikawrakow/ik_llama.cpp/issues/790) fix the issue?

---

👤 **naxan6** commented on **2025-09-24** at **06:57:58**

I can confirm its fixed now with Thireus build [main-b4158-7793fe2] - which corresponds to [#790](https://github.com/ikawrakow/ik_llama.cpp/issues/790).

Thanks!