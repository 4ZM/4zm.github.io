---
date: 2024-12-25
slug: elf-shenanigans
categories:
  - Shenanigans
  - Toolchains
---

# ELF Shenanigans - Holiday Special

Did you know that it's possible to use ANSI color codes and emojis in the section names of yor ELF binaries? I didn't. So, I had to try, and `objdump` output has never been prettier. 🎄

<!-- more -->

<pre><p style="text-align: center; margin-top: 0px; margin-bottom: 4pt;">•  •  •</p></pre>

As hackers, we some times get a *"what if I just..."* moment. A delightful spark of inspiration and mischief that can not be ignored. I found my self pondering if it would be possible to put [ANSI Codes](https://en.wikipedia.org/wiki/ANSI_escape_code) into ELF section names. Could we perhaps replace `.text` section name with some color and spice in the output of `objdump`?

It turns out, that works like a charm and I have never seen a better looking symbol table!

<center>![console](xmas.gif)</center>

The assembler complains about silly section names, but putting control characters in a linker script works just fine.

```
MEMORY {
  XMAS (rx) : ORIGIN = 0xDEC25000, LENGTH = 0x1000
}

SECTIONS
{
  "🎄 ^[[31;49m▓▒░^[[0m ^[[95;1;5m🌟^[[0m ^[[31m░▒▓^[[92;41;1mXMAS^[[31;49m▓▒░^[[0m ^[[95;1;5m🌟^[[0m ^[[31;49m░▒▓^[[0m 🎄" : {
    *(.text*)
  } > XMAS
}

ENTRY(xmas)
```

I've put some sample code over at [github.com/4ZM/elf-shenanigans](https://github.com/4ZM/elf-shenanigans) to reproduce the demo.

If you can't get enough of linker scripts or if you want to learn more about ELFs, my other article "[A Simple ELF](../2024-12-25-a-simple-elf/a-simple-elf.md)" might be helpful.

What else could we do to the `objdump` output? Multi-line ASCII art? Playing the terminal bell? Blinking text? Happy exploring, and happy holidays!

<pre><p style="text-align: center; margin-top: 0px; margin-bottom: 4pt;">•  •  •</p></pre>
