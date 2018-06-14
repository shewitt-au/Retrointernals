---
title: BASIC
---

# BASIC

I didn’t initially plan writing anything on BASIC at all. When I was working on my disassembler one day it struck me it would be kind of cool to have it handle the SYS in the BASIC bootstrap so many C64 games had (mostly after they’d been cracked except for some early games from what I’ve seen). The encoding is pretty simple so I just kept on going after I’d got the SYSes to work and got it the run on entire programs, after which I tested it on some. After you see a program on the “big screen” you wonder how the hell anyone ever got these things to work with only 40 columns. Anyway, I'll document what I discovered along the way.

## Encoding

The encoding of BASIC programs is relatively straightforward. On the C64 basic programs start at the hexadecimal address $0801, although location $0800 must be zero or attempting to run the program results in a syntax error. From $0801 onwards the lines follow in line-number order. Each line consists of a header followed by a series on PETSCII characters and tokens terminated by a zero. Immediately after the terminating zero the next line begins. The header of each line consists of two 16-bit (little-endian) unsigned integers. The first of these is the line-link; this is a pointer to the start of the next line, an entry with a high byte of $00 marking the end of the program. Normally both bytes are zero, only checking the high byte is probably an optimization, but as we’ll see later you do come across machine language programs with BASIC bootstraps taking advantage of this to save one byte. These line-links form a linked list so every line of the program can be traversed and lines looked up: LIST, GOTO and GOSUB use this mechanism. The second 16-bit number is the line number. You’d think this would mean that line numbers can be in the range 0-65535, but for some reason only 0- 63999 can be used. When a GOTO or GOSUB are executed the lines are traversed using the line-links and the target line number compared to the line number member of the of the header. The structure of the rest of a line is a series of PETSCII characters and tokens. If the byte has the MSB set it is interpreted as a token and if it’s clear it’s a character. The Tokens are listed below:

**Value** | **Command**
$80 | END	
$81 | FOR	
$82 | NEXT
$83 | DATA
$84 | INPUT#
$85 | INPUT
$86 | DIM	
$87 | READ
$88 | LET	
$89 | GOTO
$8a | RUN	
$8b | IF	
$8c | RESTORE
$8d | GOSUB
$8e | RETURN
$8f | REM	
$90 | STOP
$91 | ON	
$92 | WAIT
$93 | LOAD
$94 | SAVE
$95 | VERIFY
$96 | DEF	
$97 | POKE
$98 | PRINT#
$99 | PRINT
$9a | CONT
$9b | LIST
$9c | CLR	
$9d | CMD	
$9e | SYS	
$9f | OPEN
$a0 | CLOSE
$a1 | GET	
$a2 | NEW	
$a3 | TAB(
$a4 | TO	
$a5 | FN	
$a6 | SPC(
$a7 | THEN
$a8 | NOT	
$a9 | STEP
$aa | +	
$ab | -	
$ac | *	
$ad | /	
$ae | ↑	
$af | AND	
$b0 | OR	
$b1 | >	
$b2 | =	
$b3 | <	
$b4 | SGN	
$b5 | INT	
$b6 | ABS	
$b7 | USR	
$b7 | FRE	
$b9 | POS	
$ba | SQR	
$bb | RND	
$bc | LOG	
$bd | EXP	
$be | COS	
$bf | SIN	
$c0 | TAN	
$c1 | ATN	
$c2 | PEEK
$c3 | LEN	
$c4 | STR$
$c5 | VAL	
$c6 | ASC	
$c7 | CHR$
$c8 | LEFT$
$c9 | RIGHT$
$ca | MID$
$ff | π	

Note how the TAB and SPC command come with an embedded opening bracket, the closing one is encoded as a PETSCII character. Other commands which require brackets encode both as PETSCII characters following the tokenised command (naturally with the contents of the brackets between).

### Examples

Ok, time for some examples. Let's follow tradition and start with "Hello, World!":

	10 PRINT "HELLO, WORLD!"

In memory this looks like this:

	0801:  17 08 0a 00  99 20 22 48  45 4c 4c 4f  2c 20 57 4f  52 4c 44 21  22 00 00 00

Dumped using VICE's 'ii' command (reformatted slightly):

	0801: whj@. "HELLO, WORLD!"@@@

You can see the majority of the line is simply encoded in PETSCII, including the quotes and the space after the PRINT statement before the opening quote. Let's break it down:


	0801:  17 08   ; Line-link of $0817
	0803:  0a 00   ; Line number, 10
	0805:  99      ; Here the MSB is set so it's a token, PRINT
	0806:  20      ; The space after the PRINT, PETSCII
	0807:  "HELLO, WORLD!" ; PETSCII here
	0816:  00      ; Terminated the line
	0817:  00 00   ; Line-link with high byte of zero terminated the program.
	               ; Note that this is linked to by the previous line-link.
