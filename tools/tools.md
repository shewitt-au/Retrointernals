---
title: Tools
---

# Tools

This is the tools page where you can find the software I use for my reverse engineering.

## Negentropy

My weapon of choice. This is a disassembler for the C64 that I'm working on written in Python. This is my first real Python program and is still a work in progress, but nevertheless it's still quite useful. The repo can be found [here](https://github.com/shewitt-au/negentropy). Here's a list of some of the features currently implemented:
* Disassemble 6502 machine code
* List BASIC programs
* Annotate the generated output with labels and comments provided in configuration files
* Generate links
* Generate images for character sets

In addition to Python's standard library the following modules are used:
* Jinja2 - used for templates. Currently only HTML is supported (there are other incomplete templates)
* Pillow - for image generation
* Lark - for parsing
