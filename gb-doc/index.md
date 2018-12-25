---
layout: page
title: Gameboy Emulators
---

This page contain links to documentation on Game Boy emulation.

## RGB Rust gameboy Emulator

This project is a minimalist [gameboy emulator][sgb] in [rust].

## Nintendo Gameboy CPU (LR35902) instruction set

A [table with all the opcode][gameboy-opcodes] of the nintendo gameboy CPU.
Also include the flags affected by each instruction, and their timings.
It is a rework of Imran Nazar's opcode table.

## Blargg's testrom set

A [set of test-roms made by Blargg][blargg-roms] to check the cpu instructions behavior, timing,
acess to memory, and sound implementation.

Mirror available here:

  * [readme.txt](./data/blargg/readme.txt)
  * [cgb_sound.zip](data/blargg/cgb_sound.zip)
  * [cpu_instrs.zip](data/blargg/cpu_instrs.zip)
  * [dmg_sound.zip](data/blargg/dmg_sound.zip)
  * [halt_bug.zip](data/blargg/halt_bug.zip)
  * [instr_timing.zip](data/blargg/instr_timing.zip)
  * [interrupt_time.zip](data/blargg/interrupt_time.zip)
  * [mem_timing.zip](data/blargg/mem_timing.zip)
  * [mem_timing-2.zip](data/blargg/mem_timing-2.zip)
  * [oam_bug.zip](data/blargg/oam_bug.zip)

## How Do I Write an Emulator?

[How Do I Write an Emulator?][emulator-writing] is a general introduction to developping an emulator,
written by Daniel Boris in 1999. It is still very accurate.
This text is independent of the platform targeted, and apply to a wide range of consoles.

## Imran Nazar series

Imran Nazar's blog posts on [writing a Game Boy emulator in javascript][imran-nazar].

[rust]: https://en.wikipedia.org/wiki/Rust_(programming_language)
[sgb]: https://github.com/jeremycochoy/sgb
[gameboy-opcodes]: gameboy-opcodes.html
[blargg-roms]: http://gbdev.gg8.se/wiki/articles/Test_ROMs
[emulator-writing]: http://www.atarihq.com/danb/files/emu_vol1.txt
[imran-nazar]: http://imrannazar.com/GameBoy-Emulation-in-JavaScript:-The-CPU
