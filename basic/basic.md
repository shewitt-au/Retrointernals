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

### Example - Hello, World!

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
	0816:  00      ; Terminates the line
	0817:  00 00   ; Line-link with high byte of zero terminated the program.
	               ; Note that this is linked to by the previous line-link.

### Example - Sometimes a Cigar Is Just a Cigar

	10 PRINT"{clear}"

Note that the '{clear}' denotes a control character, it would appear as a love heart in reverse video. Anyone that's used a C64 will know that when you type in quotes keys that would normally do something, like clear the screen in this example, make funny symbols. When ran this program would clear the screen but when listed a character is printed. Let's have a look at it.

	0801:  0a 08 0a 00  99 22 93 22  00 00 00   ....."."...

We have the line-link, line number, the line terminating zero and a two-byte NULL line-link which ends the program. That leaves this:


	0805:  99 22 93 22

$99 is the token for PRINT; $22, PETSCII for a quote; $93, which looks like the token for LOAD (but isn't in this case); and $22, the closing quotes. The normal rule of a byte with its MSB set being a token doesn't apply within quotes, if if did this program would look like this:

	10 PRINT"LOAD"

This allows PETSCII characters greater than $7f in quotes.

### Example - Nothing

OK, so how is an empty BASIC program encoded. I'll fill $0801-$9fff with $ff, do a NEW and take a look. The addresses $2b-$2c & $2d-$2e come in handy. The first two contain the start of the program and the second two the byte following the end.

	002b:  01 08
	002d:  03 08

So an empty program starts at $0801 and the last byte is at $0802. We'll dump a bit more after the end.

	0801:  00 00 ff ff

Our $ff's are still there, the NEW did indeed only seem to touch $0801-$0802. An empty program is encoded with a line-link of zero (any number with a high byte of zero would do).

### Example - Fibonacci numbers

OK, for the final example we'll use a more elaborate program: calculating Fibonacci numbers.

	10 A=1
	20 B=1
	30 PRINT A, B,
	40 N=A+B
	50 PRINT N,
	60 A=B:B=N
	70 GOTO40

Here's what it looks like in memory:

	0801:  09 08 0a 00  41 b2 31 00  11 08 14 00  42 b2 31 00   ....A.1.....B.1.
	0811:  1d 08 1e 00  99 20 41 2c  20 42 2c 00  27 08 28 00   ..... A, B,.'.(.
	0821:  4e b2 41 aa  42 00 30 08  32 00 99 20  4e 2c 00 3c   N.A.B.0.2.. N,.<
	0831:  08 3c 00 41  b2 42 3a 42  b2 4e 00 44  08 46 00 89   .<.A.B:B.N.D.F..
	0841:  34 30 00 00  00                                      40...

We'll follow the line-links and analyse it line by the. Here's the first two bytes:

	0801:  09 08

Since the line-link points to the next line the first occupies $0801-$0808

	0801:  09 08 0a 00  41 b2 31 00

The first two bytes are the line-link as described earlier, and the second two the line number. Note that the space after the line number and before the contents of the line itself is not encoded but added by the code for LIST. The last byte is the terminating zero, so that leaves this:

	0805:  41 b2 31

Or like this with VICE's 'ii' command:

	0805: A.1

Straight away you'll notice that both the variable name and the number that's assigned to it are simply PETSCII. I'll be the first to admit that the primitiveness of this surprised me! In-between is $b2, this is a token as the MSB is set, in this case the token for '='.

Ok, now to find the bounds on the second line. Following the line-link above:

	0809:  11 08

The second line is at $0809-$0810. We'll just dump that using 'ii' as it's basically the same as the last.

	0809: qht@B.1@

The third line should be more interesting.

	30 PRINT A, B,

Again following the line-link from the previous line to find its bounds:

	0811:  1d 08

This one can be found at $0811-$081c.

	0811:  1d 08 1e 00  99 20 41 2c  20 42 2c 00

We have all the usual suspects here, the line-link is $081d, the line number is $1e or 30 in decimal and at the end the terminating zero. This leaves the following:

	0815:  99 20 41 2c  20 42 2c

And in using 'ii':

	0815: . A, B,

Actually, nothing to see here! Apart from the token for PRINT it's all PETSCII. Get used to that.

Now for line 40. At this point I'm starting to wish I picked a smaller program, but we'll push on.

	40 N=A+B

Two bytes from the previous line-link:

	081d:  27 08

Our live lives at $081d-$0827.

	081d:  27 08 28 00  4e b2 46 aa  42 00                   '.(.N.A.B.

Again, all the usual suspects. The only points on interest are $b2 and $aa, the tokens for '=' and '+' respectively.

Moving to line 50:

	50 PRINT N,

	0827:  30 08 32 00  99 20 4e 2c  00                         0.2.. N,.

Same old same old.

Line 60:

	60 A=B:B=N

	0830:  3c 08 3c 00  41 b2 42 3a  42 b2 4e 00                <.<.A.B:B.N.

About the only thing of interest if the colon: it's simply PETSCII. Here it's interpreted as a separator but in a REM statement it's just a colon. It depends on the context and not explicit in the binary format.

I'm going to skip over the last line, the GOTO. You may not be shocked to learn the line number is encoded as PETSCII, although I was.
