## 🗣️ [Discussion #888](https://github.com/ikawrakow/ik_llama.cpp/discussions/888) - TUI client for LLM inference?

| **Author** | `magikRUKKOLA` |
| :--- | :--- |
| **State** | ✅ **Open** |
| **Created** | 2025-11-02 |
| **Updated** | 2025-11-02 |

---

## 📄 Description

I had been using charmbracelet/mods until I'd noticed that sometimes its lagging behind the llama server while eating up a few CPU cores. https://github.com/charmbracelet/mods/issues/635

I've done some test and yeah, its extremely slow.  For example, in case I would want to output 20k ctx into the terminal it takes:

```
real    0m0.660s
user    0m0.664s
sys     0m0.102s
```

Lets compare it to some regular stuff like hightlight program with a custom nested rules for the markdown:

```lua
NestedSections = {
  { Lang="bash",        Delimiter={ [[^\s*```bash\s*$]],    [[^\s*```\s*$]] } },
  { Lang="c",           Delimiter={ [[^\s*```c\s*$]],       [[^\s*```\s*$]] } },
  { Lang="cpp2",         Delimiter={ [[^\s*```cpp\s*$]],     [[^\s*```\s*$]] } },
  { Lang="css",         Delimiter={ [[^\s*```css\s*$]],     [[^\s*```\s*$]] } },
  { Lang="html",        Delimiter={ [[^\s*```html\s*$]],    [[^\s*```\s*$]] } },
  { Lang="java",        Delimiter={ [[^\s*```java\s*$]],    [[^\s*```\s*$]] } },
  { Lang="javascript",  Delimiter={ [[^\s*```js\s*$]],      [[^\s*```\s*$]] } },
  { Lang="javascript",  Delimiter={ [[^\s*```javascript\s*$]],[[^\s*```\s*$]] } },
  { Lang="json",        Delimiter={ [[^\s*```json\s*$]],    [[^\s*```\s*$]] } },
  { Lang="lua",         Delimiter={ [[^\s*```lua\s*$]],     [[^\s*```\s*$]] } },
  { Lang="perl",        Delimiter={ [[^\s*```perl\s*$]],    [[^\s*```\s*$]] } },
  { Lang="php",         Delimiter={ [[^\s*```php\s*$]],     [[^\s*```\s*$]] } },
  { Lang="python",      Delimiter={ [[^\s*```py\s*$]],      [[^\s*```\s*$]] } },
  { Lang="python",      Delimiter={ [[^\s*```python\s*$]],  [[^\s*```\s*$]] } },
  { Lang="ruby",        Delimiter={ [[^\s*```rb\s*$]],      [[^\s*```\s*$]] } },
  { Lang="shell",       Delimiter={ [[^\s*```sh\s*$]],      [[^\s*```\s*$]] } },
  { Lang="sql",         Delimiter={ [[^\s*```sql\s*$]],     [[^\s*```\s*$]] } },
  { Lang="xml",         Delimiter={ [[^\s*```xml\s*$]],     [[^\s*```\s*$]] } },
}
```

And here is the results for the same 20k ctx:

```
time highlight -S md -O xterm256 -s xoria256 --verbose --stdout /tmp/20k

real    0m0.087s
user    0m0.078s
sys     0m0.008s
```

So its about ten times faster than mods.

And mods is all I found which is working more-or-less stable.

I mean, all the TUI client does is syntax highlighting and conversation history management.  It shouldn't be that resourse-intensive, right?  But what that actually mean?  Is there any decent and stable TUI or not?  Because I can't find it.

---

## 💬 Discussion

👤 **magikRUKKOLA** commented on **2025-11-02** at **19:49:36**

Yes, indeed, this stupid charmbracelet/mods terminal happily eats away my CPU:

```

    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
 588661 root      20   0   61664  17496  14472 S   0.3   0.0   0:00.01              `- perf
 588664 root      20   0    5588   2176   2068 S   0.0   0.0   0:00.00                  `- sleep
 115596 root      20   0   11124   6480   4416 S   0.0   0.0   0:18.07      `- bash
 581215 root      20   0 4438100  54448  19816 S  30.2   0.0   0:29.99          `- mods
```

30 seconds of CPU time while outputting a few thousands of tokens in decode.  This is pretty crazy.  Its over a minute of CPU time by now. lool

[EDIT]:
a few minutes later...
```

    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
 589935 root      20   0   61664  17564  14540 S   0.0   0.0   0:00.00              `- perf
 589938 root      20   0    5588   2160   2052 S   0.0   0.0   0:00.00                  `- sleep
 115596 root      20   0   11124   6480   4416 S   0.0   0.0   0:18.07      `- bash
 581215 root      20   0 4881516  57544  19944 S  57.9   0.0   2:13.18          `- mods
```

[EDIT2]:
```
    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
 592003 root      20   0   61664  17700  14676 S   0.7   0.0   0:00.02              `- perf
 592006 root      20   0    5588   2076   1964 S   0.0   0.0   0:00.00                  `- sleep
 115596 root      20   0   11124   6480   4416 S   0.0   0.0   0:18.07      `- bash
 581215 root      20   0 5030324  48424  20136 S 166.9   0.0   9:51.27          `- mods
```