---
title: Boulder Dash Theme Score
layout: default
nosidebar: true
---

# Boulder Dash Theme Score

OK, I got the idea to score the Boulder Dash music into my head. I had a look around to see if anyone else had already done so, and they had, kind of. The man himself has a crack [here](https://www.brainjam.ca/wp/2009/11/scoring-the-boulder-dash-theme/). But with some knowledge of the limitations of the music routine it’s clear it’s not a faithful representation, more of an arrangement; the routine doesn’t do rests, or notes of any but one duration. There are other scores out there, but all of them seem to be arrangements of the theme and not faithful representations. Rectifying this lack of accurate sheet music for this corker might keep me out of trouble for a bit.

## Congratulations. I don't give a shit!

Ah... It's like that is it?

A PDF of the score can be found [here](./Boulder_Dash.pdf)

More of an SVG man? We can [accommodate](./Boulder_Dash.svg)

MIDI? [sure](./Boulder_Dash.mid)

A GitHub repository with all the [code](https://github.com/shewitt-au/BD-Music)

## Overview

The theme music use two of the SID chip's three voices. The data for the tune is exactly 256 bytes long (not including the note to SID frequency value table) and as mentioned above no rests or notes of any but one duration are supported. The tune data is simply 128 pairs of note numbers, the first value of the pair for one voice and the second for the other. If we notate the note duration as a quaver (a quarter note) the tune is 16 bars long (4/4 time) repeated twice. I think it's in F minor, and I've notated it as such, but I'm no musician.

## Tuning
I'm going to start by figuring out the tuning. The table which maps notes to SID frequency values looks like this.

<pre>
<b>MusicNoteToFreqTable</b>:
<b>8317</b>   dc 02 0a 03  3a 03 6c 03  a0 03 d2 03  12 04 4c 04
<b>8327</b>   92 04 d6 04  20 05 6e 05  b8 05 14 06  74 06 d8 06
<b>8337</b>   40 07 a4 07  24 08 98 08  24 09 ac 09  40 0a dc 0a
<b>8347</b>   70 0b 28 0c  e8 0c b0 0d  80 0e 48 0f  48 10 30 11
<b>8357</b>   48 12 58 13  80 14 b8 15  e0 16 50 18  d0 19 60 1b
<b>8367</b>   00 1d 90 1e  90 20 60 22  90 24 b0 26  00 29 70 2b
<b>8377</b>   c0 2d
</pre>

Each entry is two bytes long. We need to do two things here:

	1. Map from SID frequency value to a frequency.
	2. Map from a frequency to a note.

We'll take one at a time then tie it all together.

### Mapping from a SID frequency value to a frequency

I Googled a few resources when trying to figure out how to do this. I did briefly consider getting an oscilloscope and verifying what I'd learnt, but I decided that was probably overkill. We'll interpret consistency across multiple sources as verification enough. Anyway, here are some links if you're interested in reading up on this:


[The SID (by Stephen L. Judd)](http://www.cbmitapages.it/c64/sid1eng.htm)

[How to calculate SID frequency tables By FTC/HT](https://codebase64.org/doku.php?id=base:how_to_calculate_your_own_sid_frequency_table)

[Excact Vertical Frequency / Refresh Rate](https://csdb.dk/forums/?roomid=11&topicid=124823&firstpost=2)

I hacked together the Python code below to get the frequencies of the notes. I live in Australia, so we had PAL machines. I'm only going to consider the PAL case but I've included the code for NTSC machines.

```python
#!/usr/bin/env python3

from math import log

bd_sid_values = [
0xdc, 0x02, 0x0a, 0x03,  0x3a, 0x03, 0x6c, 0x03,  0xa0, 0x03, 0xd2, 0x03,  0x12, 0x04, 0x4c, 0x04,
0x92, 0x04, 0xd6, 0x04,  0x20, 0x05, 0x6e, 0x05,  0xb8, 0x05, 0x14, 0x06,  0x74, 0x06, 0xd8, 0x06,
0x40, 0x07, 0xa4, 0x07,  0x24, 0x08, 0x98, 0x08,  0x24, 0x09, 0xac, 0x09,  0x40, 0x0a, 0xdc, 0x0a,
0x70, 0x0b, 0x28, 0x0c,  0xe8, 0x0c, 0xb0, 0x0d,  0x80, 0x0e, 0x48, 0x0f,  0x48, 0x10, 0x30, 0x11,
0x48, 0x12, 0x58, 0x13,  0x80, 0x14, 0xb8, 0x15,  0xe0, 0x16, 0x50, 0x18,  0xd0, 0x19, 0x60, 0x1b,
0x00, 0x1d, 0x90, 0x1e,  0x90, 0x20, 0x60, 0x22,  0x90, 0x24, 0xb0, 0x26,  0x00, 0x29, 0x70, 0x2b,
0xc0, 0x2d]

def sid_frequencies():
    i = iter(bd_sid_values)
    try:
        while True:
            yield next(i)+next(i)*256
    except StopIteration:
        return

pal_const =  (256**3)/985248
ntsc_const = (256**3)/1022727

def reg_to_freq_pal(reg):
    return reg/pal_const

def reg_to_freq_ntsc(reg):
    return reg/ntsc_const

def cents_from(f, ref):
    return 1200*log(f/ref, 2)

print("PAL\n---")
for f in sid_frequencies():
    f = reg_to_freq_pal(f)
    print(reg_to_freq_pal(f))
```

Here's the output:
<pre>
PAL
---
42.986961364746094
45.68832778930664
48.507144927978516
51.44341278076171
54.49713134765624
57.433399200439446
61.191822052001946
64.59789276123047
68.70866775512695
72.70199203491211
77.04766845703125
81.62824630737305
85.97392272949219
91.37665557861328
97.01428985595703
102.88682556152342
108.99426269531249
114.86679840087889
122.38364410400389
129.19578552246094
137.4173355102539
145.40398406982422
154.0953369140625
163.2564926147461
171.94784545898438
182.75331115722656
194.02857971191406
205.77365112304685
217.98852539062497
229.73359680175778
244.76728820800778
258.3915710449219
274.8346710205078
290.80796813964844
308.190673828125
326.5129852294922
343.89569091796875
365.5066223144531
388.0571594238281
411.5473022460937
435.97705078124994
459.46719360351557
489.53457641601557
516.7831420898438
549.6693420410156
581.6159362792969
616.38134765625
653.0259704589844
687.7913818359375
</pre>

The closest value to A440 is 435.97705078124994 which is 16 cents below. A perceptible distance. If we use A435 instead this is still the closest value but this time 4 cents higher. That's more like it. We'll use this entry as our A4. Here is some info on [cents](https://en.wikipedia.org/wiki/Cent_(music)) and [A440](https://en.wikipedia.org/wiki/A440_(pitch_standard)) for the curious.

### Maping from a frequency to a note

This requires some understanding of the maths behind musical tunings. When you start looking into this you can't help but wonder how deep the rabbit hole goes. For our purposes we'll only concern ourselves with the [equal temperament](https://en.wikipedia.org/wiki/Equal_temperament) tuning system. To cut a long story short the following Python code will give us a note number measured in semitones from the reference frequency a4.

```pyhhon
base = 2**(1/12)
a4 = 435.97705078124994

def freq_to_note(f):
	return log(f/a4, base)
```

### Tieing it all together

In short this:

```python
#!/usr/bin/env python3

from math import log, floor

bd_sid_values = [
0xdc, 0x02, 0x0a, 0x03,  0x3a, 0x03, 0x6c, 0x03,  0xa0, 0x03, 0xd2, 0x03,  0x12, 0x04, 0x4c, 0x04,
0x92, 0x04, 0xd6, 0x04,  0x20, 0x05, 0x6e, 0x05,  0xb8, 0x05, 0x14, 0x06,  0x74, 0x06, 0xd8, 0x06,
0x40, 0x07, 0xa4, 0x07,  0x24, 0x08, 0x98, 0x08,  0x24, 0x09, 0xac, 0x09,  0x40, 0x0a, 0xdc, 0x0a,
0x70, 0x0b, 0x28, 0x0c,  0xe8, 0x0c, 0xb0, 0x0d,  0x80, 0x0e, 0x48, 0x0f,  0x48, 0x10, 0x30, 0x11,
0x48, 0x12, 0x58, 0x13,  0x80, 0x14, 0xb8, 0x15,  0xe0, 0x16, 0x50, 0x18,  0xd0, 0x19, 0x60, 0x1b,
0x00, 0x1d, 0x90, 0x1e,  0x90, 0x20, 0x60, 0x22,  0x90, 0x24, 0xb0, 0x26,  0x00, 0x29, 0x70, 0x2b,
0xc0, 0x2d]

def sid_frequencies():
    i = iter(bd_sid_values)
    try:
        while True:
            yield next(i)+next(i)*256
    except StopIteration:
        return

def note_to_sid(n):
    return bd_sid_values[n*2]+bd_sid_values[n*2+1]*256

names_sharp=['a{}','a{}♯','b{}','c{}','c{}♯','d{}','d{}♯','e{}','f{}','f{}♯','g{}','g{}♯']
names_flat=['a{}','b{}♭','b{}','c{}','d{}♭','d{}','e{}♭','e{}','f{}','g{}♭','g{}','a♭{}']

def index_to_name(i, sharp):
    octave = floor((i-3)/12)+5
    names = names_sharp if sharp else names_flat
    return names[i%12].format(octave)

pal_const =  (256**3)/985248
ntsc_const = (256**3)/1022727
a4 = 435.97705078124994
base = 2**(1/12)

def reg_to_freq_pal(reg):
    return reg/pal_const

def reg_to_freq_ntsc(reg):
    return reg/ntsc_const

def freq_to_note(f):
    return log(f/a4, base)

bd_val = 0x0a
for sid in sid_frequencies():
    freq = reg_to_freq_pal(sid)
    idx = round(freq_to_note(freq))
    print("${:02x} : {}".format(bd_val, index_to_name(idx, True)))
    bd_val += 1
print()
```

Gives us the note names that correspond to the note numbering Boulder Dash uses:

	$0a : f1
	$0b : f1♯
	$0c : g1
	$0d : g1♯
	$0e : a1
	$0f : a1♯
	$10 : b1
	$11 : c2
	$12 : c2♯
	$13 : d2
	$14 : d2♯
	$15 : e2
	$16 : f2
	$17 : f2♯
	$18 : g2
	$19 : g2♯
	$1a : a2
	$1b : a2♯
	$1c : b2
	$1d : c3
	$1e : c3♯
	$1f : d3
	$20 : d3♯
	$21 : e3
	$22 : f3
	$23 : f3♯
	$24 : g3
	$25 : g3♯
	$26 : a3
	$27 : a3♯
	$28 : b3
	$29 : c4
	$2a : c4♯
	$2b : d4
	$2c : d4♯
	$2d : e4
	$2e : f4
	$2f : f4♯
	$30 : g4
	$31 : g4♯
	$32 : a4
	$33 : a4♯
	$34 : b4
	$35 : c5
	$36 : c5♯
	$37 : d5
	$38 : d5♯
	$39 : e5
	$3a : f5

In music notation:

![Note values](./notes.svg)

## The Tune

Here's the data for the tune:

<pre>
<b>5fe8</b>   16 22 1d 26  22 29 25 2e  14 24 1f 27  20 29 27 30
<b>5ff8</b>   12 2a 12 2c  1e 2e 12 31  20 2c 33 37  21 2d 31 35
<b>6008</b>   16 22 16 2e  16 1d 16 24  14 20 14 30  14 24 14 20
<b>6018</b>   16 22 16 2e  16 1d 16 24  1e 2a 1e 3a  1e 2e 1e 2a
<b>6028</b>   14 20 14 2c  14 1b 14 22  1c 28 1c 38  1c 2c 1c 28
<b>6038</b>   11 1d 29 2d  11 1f 29 2e  0f 27 0f 27  16 33 16 27
<b>6048</b>   16 2e 16 2e  16 2e 16 2e  22 2e 22 2e  16 2e 16 2e
<b>6058</b>   14 2e 14 2e  14 2e 14 2e  20 2e 20 2e  14 2e 14 2e
<b>6068</b>   16 2e 32 2e  16 2e 33 2e  22 2e 32 2e  16 2e 33 2e
<b>6078</b>   14 2e 32 2e  14 2e 33 2e  20 2c 30 2c  14 2c 31 2c
<b>6088</b>   16 2e 16 3a  16 2e 35 38  22 2e 22 37  16 2e 31 35
<b>6098</b>   14 2c 14 38  14 2c 14 38  20 2c 20 33  14 2c 14 38
<b>60a8</b>   16 2e 32 2e  16 2e 33 2e  22 2e 32 2e  16 2e 33 2e
<b>60b8</b>   14 2e 32 2e  14 2e 33 2e  20 2c 30 2c  14 2c 31 2c
<b>60c8</b>   2e 32 29 2e  26 29 22 26  2c 30 27 2c  24 27 14 20
<b>60d8</b>   35 32 32 2e  2e 29 29 26  27 30 24 2c  20 27 14 20
</pre>

Now we'll take a look at the code which fetches the note data, looks up the SID values for the note and writes the values to the SID chip's frequency registers:

<pre>
<b>83ca</b>   ae 00 98   LDX <b>MusicDataIndex</b>
<b>83cd</b>   ee 00 98   INC <b>MusicDataIndex</b>
<b>83d0</b>   ee 00 98   INC <b>MusicDataIndex</b>
<b>83d3</b>   bd e9 5f   LDA <b>MusicData</b>+1,X
<b>83d6</b>   0a         ASL A
<b>83d7</b>   a8         TAY 
<b>83d8</b>   b9 03 83   LDA <b>MusicNoteToFreqTable</b>-$14,Y
<b>83db</b>   8d 00 d4   STA <b>Sid_Voice1FreqLo</b>
<b>83de</b>   b9 04 83   LDA <b>MusicNoteToFreqTable</b>-$13,Y
<b>83e1</b>   8d 01 d4   STA <b>Sid_Voice1FreqHi</b>
<b>83e4</b>   bd e8 5f   LDA <b>MusicData</b>,X
<b>83e7</b>   0a         ASL A
<b>83e8</b>   a8         TAY 
<b>83e9</b>   b9 03 83   LDA <b>MusicNoteToFreqTable</b>-$14,Y
<b>83ec</b>   8d 07 d4   STA <b>Sid_Voice2FreqLo</b>
<b>83ef</b>   b9 04 83   LDA <b>MusicNoteToFreqTable</b>-$13,Y
<b>83f2</b>   8d 08 d4   STA <b>Sid_Voice2FreqHi</b>
</pre>

Again, the music data this is simply 128 pairs of note values, one for each voice. You can see the first number of the pair is for voice 2 and the second for voice 1. Also the first note value is $0a which makes the last $3a.

We have all the pieces we need to dump the notes for both voices. Let's snap some blocks together:

```python
#!/usr/bin/env python3

from math import log, floor

bd_sid_values = [
0xdc, 0x02, 0x0a, 0x03,  0x3a, 0x03, 0x6c, 0x03,  0xa0, 0x03, 0xd2, 0x03,  0x12, 0x04, 0x4c, 0x04,
0x92, 0x04, 0xd6, 0x04,  0x20, 0x05, 0x6e, 0x05,  0xb8, 0x05, 0x14, 0x06,  0x74, 0x06, 0xd8, 0x06,
0x40, 0x07, 0xa4, 0x07,  0x24, 0x08, 0x98, 0x08,  0x24, 0x09, 0xac, 0x09,  0x40, 0x0a, 0xdc, 0x0a,
0x70, 0x0b, 0x28, 0x0c,  0xe8, 0x0c, 0xb0, 0x0d,  0x80, 0x0e, 0x48, 0x0f,  0x48, 0x10, 0x30, 0x11,
0x48, 0x12, 0x58, 0x13,  0x80, 0x14, 0xb8, 0x15,  0xe0, 0x16, 0x50, 0x18,  0xd0, 0x19, 0x60, 0x1b,
0x00, 0x1d, 0x90, 0x1e,  0x90, 0x20, 0x60, 0x22,  0x90, 0x24, 0xb0, 0x26,  0x00, 0x29, 0x70, 0x2b,
0xc0, 0x2d]

def sid_frequencies():
    i = iter(bd_sid_values)
    try:
        while True:
            yield next(i)+next(i)*256
    except StopIteration:
        return

def note_to_sid(n):
    return bd_sid_values[n*2]+bd_sid_values[n*2+1]*256

names_sharp=['a{}','a{}♯','b{}','c{}','c{}♯','d{}','d{}♯','e{}','f{}','f{}♯','g{}','g{}♯']
names_flat=['a{}','b{}♭','b{}','c{}','d{}♭','d{}','e{}♭','e{}','f{}','g{}♭','g{}','a♭{}']

def index_to_name(i, sharp):
    octave = floor((i-3)/12)+5
    names = names_sharp if sharp else names_flat
    return names[i%12].format(octave)

bd_music = [
0x16, 0x22, 0x1d, 0x26,  0x22, 0x29, 0x25, 0x2e,  0x14, 0x24, 0x1f, 0x27,  0x20, 0x29, 0x27, 0x30,
0x12, 0x2a, 0x12, 0x2c,  0x1e, 0x2e, 0x12, 0x31,  0x20, 0x2c, 0x33, 0x37,  0x21, 0x2d, 0x31, 0x35,
0x16, 0x22, 0x16, 0x2e,  0x16, 0x1d, 0x16, 0x24,  0x14, 0x20, 0x14, 0x30,  0x14, 0x24, 0x14, 0x20,
0x16, 0x22, 0x16, 0x2e,  0x16, 0x1d, 0x16, 0x24,  0x1e, 0x2a, 0x1e, 0x3a,  0x1e, 0x2e, 0x1e, 0x2a,
0x14, 0x20, 0x14, 0x2c,  0x14, 0x1b, 0x14, 0x22,  0x1c, 0x28, 0x1c, 0x38,  0x1c, 0x2c, 0x1c, 0x28,
0x11, 0x1d, 0x29, 0x2d,  0x11, 0x1f, 0x29, 0x2e,  0x0f, 0x27, 0x0f, 0x27,  0x16, 0x33, 0x16, 0x27,
0x16, 0x2e, 0x16, 0x2e,  0x16, 0x2e, 0x16, 0x2e,  0x22, 0x2e, 0x22, 0x2e,  0x16, 0x2e, 0x16, 0x2e,
0x14, 0x2e, 0x14, 0x2e,  0x14, 0x2e, 0x14, 0x2e,  0x20, 0x2e, 0x20, 0x2e,  0x14, 0x2e, 0x14, 0x2e,
0x16, 0x2e, 0x32, 0x2e,  0x16, 0x2e, 0x33, 0x2e,  0x22, 0x2e, 0x32, 0x2e,  0x16, 0x2e, 0x33, 0x2e,
0x14, 0x2e, 0x32, 0x2e,  0x14, 0x2e, 0x33, 0x2e,  0x20, 0x2c, 0x30, 0x2c,  0x14, 0x2c, 0x31, 0x2c,
0x16, 0x2e, 0x16, 0x3a,  0x16, 0x2e, 0x35, 0x38,  0x22, 0x2e, 0x22, 0x37,  0x16, 0x2e, 0x31, 0x35,
0x14, 0x2c, 0x14, 0x38,  0x14, 0x2c, 0x14, 0x38,  0x20, 0x2c, 0x20, 0x33,  0x14, 0x2c, 0x14, 0x38,
0x16, 0x2e, 0x32, 0x2e,  0x16, 0x2e, 0x33, 0x2e,  0x22, 0x2e, 0x32, 0x2e,  0x16, 0x2e, 0x33, 0x2e,
0x14, 0x2e, 0x32, 0x2e,  0x14, 0x2e, 0x33, 0x2e,  0x20, 0x2c, 0x30, 0x2c,  0x14, 0x2c, 0x31, 0x2c,
0x2e, 0x32, 0x29, 0x2e,  0x26, 0x29, 0x22, 0x26,  0x2c, 0x30, 0x27, 0x2c,  0x24, 0x27, 0x14, 0x20,
0x35, 0x32, 0x32, 0x2e,  0x2e, 0x29, 0x29, 0x26,  0x27, 0x30, 0x24, 0x2c,  0x20, 0x27, 0x14, 0x20]

def voice1():
    i = iter(bd_music)
    try:
        while True:
            next(i)
            yield next(i)-10
    except StopIteration:
        return

def voice2():
    i = iter(bd_music)
    try:
        while True:
            yield next(i)-10
            next(i)
    except StopIteration:
        return

pal_const =  (256**3)/985248
ntsc_const = (256**3)/1022727
a4 = 435.97705078124994
base = 2**(1/12)

def reg_to_freq_pal(reg):
    return reg/pal_const

def reg_to_freq_ntsc(reg):
    return reg/ntsc_const

def freq_to_note(f):
    return log(f/a4, base)

v1 = ""
for n in voice1():
    sid = note_to_sid(n)
    f = reg_to_freq_pal(sid)
    i = round(freq_to_note(f))
    v1 += index_to_name(i, True)+" "

v2 = ""
for n in voice2():
    sid = note_to_sid(n)
    f = reg_to_freq_pal(sid)
    i = round(freq_to_note(f))
    v2 += index_to_name(i, True)+" "

print("Voice 1")
print("-------")
print(v1)
print()
print("Voice 2")
print("-------")
print(v2)
```

Which spits out this:

<pre>
Voice 1
-------
f3 a3 c4 f4 g3 a3♯ c4 g4 c4♯ d4♯ f4 g4♯ d4♯ d5 e4 c5 f3 f4 c3 g3 d3♯ g4 g3
d3♯ f3 f4 c3 g3 c4♯ f5 f4 c4♯ d3♯ d4♯ a2♯ f3 b3 d5♯ d4♯ b3 c3 e4 d3 f4 a3♯
a3♯ a4♯ a3♯ f4 f4 f4 f4 f4 f4 f4 f4 f4 f4 f4 f4 f4 f4 f4 f4 f4 f4 f4 f4 f4
f4 f4 f4 f4 f4 f4 f4 d4♯ d4♯ d4♯ d4♯ f4 f5 f4 d5♯ f4 d5 f4 c5 d4♯ d5♯ d4♯
d5♯ d4♯ a4♯ d4♯ d5♯ f4 f4 f4 f4 f4 f4 f4 f4 f4 f4 f4 f4 d4♯ d4♯ d4♯ d4♯ a4
f4 c4 a3 g4 d4♯ a3♯ d3♯ a4 f4 c4 a3 g4 d4♯ a3♯ d3♯

Voice 2
-------
f2 c3 f3 g3♯ d2♯ d3 d3♯ a3♯ c2♯ c2♯ c3♯ c2♯ d3♯ a4♯ e3 g4♯ f2 f2 f2 f2 d2♯
d2♯ d2♯ d2♯ f2 f2 f2 f2 c3♯ c3♯ c3♯ c3♯ d2♯ d2♯ d2♯ d2♯ b2 b2 b2 b2 c2 c4
c2 c4 a1♯ a1♯ f2 f2 f2 f2 f2 f2 f3 f3 f2 f2 d2♯ d2♯ d2♯ d2♯ d3♯ d3♯ d2♯ d2♯
f2 a4 f2 a4♯ f3 a4 f2 a4♯ d2♯ a4 d2♯ a4♯ d3♯ g4 d2♯ g4♯ f2 f2 f2 c5 f3 f3
f2 g4♯ d2♯ d2♯ d2♯ d2♯ d3♯ d3♯ d2♯ d2♯ f2 a4 f2 a4♯ f3 a4 f2 a4♯ d2♯ a4 d2♯
a4♯ d3♯ g4 d2♯ g4♯ f4 c4 a3 f3 d4♯ a3♯ g3 d2♯ c5 a4 f4 c4 a3♯ g3 d3♯ d2♯
</pre>

## What key is it in?

A fair question. At first I tried to stare down the notes until they buckled and gave up the goods. My knowledge of music theory is limited however and any candidate key I picked still had more [accidentals](https://en.wikipedia.org/wiki/Accidental_(music)) than I felt comfortable with. I decided to whip up some code which compares the notes in the theme to all of the major keys and counts up the accidentals. With a bit of luck this would narrow down the candidates considerably. This article is getting a little code heavy and there's a bit of repetition, so I'll just show the additional code.

Firstly I added code the count the notes in the melody. All octaves of a note are counted as the same note (so a1 and a2 just count as a).

```python
class Spectrum(object):
	def __init__(self):
		self.notes = [0]*12

	def add(self, n):
		self.notes[n%12] += 1

	def __str__(self):
		s = ""
		for i in range(0, 12):
			s += names[i].ljust(2)+" : "+str(self.notes[i])+"\n"
		return s
```

Next a class to generate the notes in all of the major keys (and the notes which aren't in them):

```python
class Keys(object):
	def __init__(self):
		maj = [0, 2, 2, 1, 2, 2, 2]
		self.keys = {}
		for t in range(0, 12):
			s = []
			last = t
			for i in maj:
				s.append((last+i)%12)
				last += i
			self.keys[t] = s

	def notes_in(self, k):
		return self.keys[k]

	def notes_not_in(self, k):
		return [n for n in range(0, 12) if n not in self.keys[k]]
```

So we get a Spectrum (s below) object, feed it all the notes then run this:

```python
names = ['a', 'a♯', 'b', 'c', 'c♯', 'd', 'd♯', 'e', 'f', 'f♯', 'g', 'g♯']
keys = Keys()
for k in range(0, 12):
	acc = 0
	for nkn in keys.notes_not_in(k):
		acc += s.notes[nkn]
	print(names[k].ljust(2)+" : "+str(acc))
```

Here's the results:

<pre>
a  : 213
a♯ : 26
b  : 142
c  : 105
c♯ : 38
d  : 207
d♯ : 33
e  : 150
f  : 90
f♯ : 50
g  : 200
g♯ : 26
</pre>

We've got two stand outs here: a♯ major and g# major (normally you'd call these B♭ and A♭ major). I actually think it sounds minor. The [relative minor](https://en.wikipedia.org/wiki/Relative_key) keys which correspond are G minor and F minor. Of these I went with F minor after some more note gazing.

Given this key the accidentals should be flats:

<pre>
Voice 1
-------
f3 a3 c4 f4 g3 b3♭ c4 g4 d4♭ e4♭ f4 a♭4 e4♭ d5 e4 c5 f3 f4 c3 g3 e3♭ g4 g3 e3♭ f3 f4 c3 g3
d4♭ f5 f4 d4♭ e3♭ e4♭ b2♭ f3 b3 e5♭ e4♭ b3 c3 e4 d3 f4 b3♭ b3♭ b4♭ b3♭ f4 f4 f4 f4 f4 f4 f4
f4 f4 f4 f4 f4 f4 f4 f4 f4 f4 f4 f4 f4 f4 f4 f4 f4 f4 f4 f4 f4 e4♭ e4♭ e4♭ e4♭ f4 f5 f4 e5♭
f4 d5 f4 c5 e4♭ e5♭ e4♭ e5♭ e4♭ b4♭ e4♭ e5♭ f4 f4 f4 f4 f4 f4 f4 f4 f4 f4 f4 f4 e4♭ e4♭ e4♭
a4 f4 c4 a3 g4 e4♭ b3♭ e3♭ a4 f4 c4 a3 g4 e4♭ b3♭ e3♭ 

Voice 2
-------
f2 c3 f3 a♭3 e2♭ d3 e3♭ b3♭ d2♭ d2♭ d3♭ d2♭ e3♭ b4♭ e3 a♭4 f2 f2 f2 f2 e2♭ e2♭ e2♭ e2♭ f2 f2
f2 f2 d3♭ d3♭ d3♭ d3♭ e2♭ e2♭ e2♭ e2♭ b2 b2 b2 b2 c2 c4 c2 c4 b1♭ b1♭ f2 f2 f2 f2 f2 f2 f3
f3 f2 f2 e2♭ e2♭ e2♭ e2♭ e3♭ e3♭ e2♭ e2♭ f2 a4 f2 b4♭ f3 a4 f2 b4♭ e2♭ a4 e2♭ b4♭ e3♭ g4 e2♭
a♭4 f2 f2 f2 c5 f3 f3 f2 a♭4 e2♭ e2♭ e2♭ e2♭ e3♭ e3♭ e2♭ e2♭ f2 a4 f2 b4♭ f3 a4 f2 b4♭ e2♭
a4 e2♭ b4♭ e3♭ g4 e2♭ a♭4 f4 c4 a3 f3 e4♭ b3♭ g3 e2♭ c5 a4 f4 c4 b3♭ g3 e3♭ e2♭ 
</pre>

## Tempo

The music routine is called twice per frame but only does anything of consequence every other call. There's some weirdness going on there. Why not just call the routine once per frame instead of calling it twice and having it do nothing half of the time? Function calls aren't free.

<pre>
<b>MusicTickRoutine</b>:
<b>83ac</b>   a5 c1      LDA <b>TitleScreenFrameCountMod4</b>
<b>83ae</b>   29 01      AND #$01
<b>83b0</b>   d0 01      BNE $83b3
<b>83b2</b>   60         RTS 
</pre>

So on every other call:

<pre>
<b>83b3</b>   ad 0c 98   LDA <b>MusicTickRoutine__Voice1SustainLevel</b>
<b>83b6</b>   c9 a0      CMP #$a0
<b>83b8</b>   d0 43      BNE <b>__MusicTickRoutine__TweakCurrentNote</b>
<b>__MusicTickRoutine__NextNote</b>:
; Set voice 1&2 to triangle wave.
<b>83ba</b>   a9 10      LDA #$10
<b>83bc</b>   8d 04 d4   STA <b>Sid_Voice1Ctrl</b>
<b>83bf</b>   8d 0b d4   STA <b>Sid_Voice2Ctrl</b>
 .
 .
 .
<b>__MusicTickRoutine__TweakCurrentNote</b>:
<b>83fd</b>   ad 0c 98   LDA <b>MusicTickRoutine__Voice1SustainLevel</b>
 .
 .
 .
<b>8413</b>   ad 0c 98   LDA <b>MusicTickRoutine__Voice1SustainLevel</b>
<b>8416</b>   18         CLC 
<b>8417</b>   69 01      ADC #$01
<b>8419</b>   29 a7      AND #$a7
<b>841b</b>   8d 0c 98   STA <b>MusicTickRoutine__Voice1SustainLevel</b>
 .
 .
 .
</pre>


OK, there's some weirdness going on here too. Whether the routine advances to the next note (actually the next note for each of the two voices used) or just modulates the amplitude of voice one is determined by **MusicTickRoutine__Voice1SustainLevel**. This is initialised to $a0 before the music starts and a new note is only played when it's $a0. The lower three bits are incremented every other call but the $a in the high nybble is entirely pointless. The upshot of all this is the song advances to the next note every 8 frames. We'll use quavers (eighths notes) for the transcription. Given that PAL has around 50 frames per second this gives us a BPM of 187.5. We'll call it 188. 


## Notes on a staff bitch!

OK, OK, I getting to it. Writing music engraving software from scratch would obviously be a major undertaking and breaking out my ruler is just offensive. I fired up Google and starting searching for some free software that could do the deed. Something that had an interactive user interface was not a requirement; I needed it to consume a text based file that my software could generate. Initially I came across [VexFlow](http://www.vexflow.com/). It had limitations, but while reading various message boards trying to figure out how to resolve these issues I came discovered [LilyPond](http://lilypond.org/). Holy shit! The stuff they give away for free these days. The other building block I needed was a Python based templating system. My go to for this kind of thing is [Jinja2](https://jinja.palletsprojects.com/en/2.10.x/). So firstly we need the notes in a form that LilyPond can consume. I added this code:

```python
lilynames_sharp = ['a', 'as', 'b', 'c', 'cs', 'd', 'ds', 'e','f', 'fs', 'g', 'gs']
lilynames_flat  = ['a', 'bf', 'b', 'c', 'df', 'd', 'ef', 'e','f', 'gf', 'g', 'af']

def index_to_lily(i, sharp):
	names = lilynames_sharp if sharp else lilynames_flat
	octave = floor((i-3)/12)+2
	if octave<0:
		octave = ','*-octave
	else:
		octave = "'"*octave
	return names[i%12]+octave
```

See [here](http://lilypond.org/doc/v2.18/Documentation/notation/writing-pitches) and [there](http://lilypond.org/doc/v2.18/Documentation/notation/writing-pitches#note-names-in-other-languages) for information on these note naming conventions.

Next we need a skeletal Lilipond file and to hook up Jinja2 to fill in the blanks. The Jinja2 side of things is pretty simple:

```python
# import jinja2

def render(env, template_name, **template_vars):
    template = env.get_template(template_name)
    return template.render(**template_vars)

with open("test.ly", "w", encoding='utf-8') as of:
	v1 = ""
	for n in voice1():
		sid = note_to_sid(n)
		f = reg_to_freq_pal(sid)
		i = round(freq_to_note(f))
		v1 += index_to_lily(i, False)+"8 "
	v2 = ""
	for n in voice2():
		sid = note_to_sid(n)
		f = reg_to_freq_pal(sid)
		i = round(freq_to_note(f))
		v2 += index_to_lily(i, False)+"8 "

	env = jinja2.Environment(loader=jinja2.FileSystemLoader('.'))
	env.globals['voice1'] = v1
	env.globals['voice2'] = v2
	env.globals['key'] = r"\key f \minor"
	env.globals['tempo'] = r"\tempo 4 = 188"
	s = render(env, 'bd.ly')
	of.write(s)
```

And finally our skeleton:

<pre>
\version "2.16.0"

\header {
  title = "Boulder Dash (C64)"
  composer = "Peter Liepa"
  arranger = "Transcribed by Stephen Hewitt"
  tagline = "" % removed
}

\language english

\score
{
	\new PianoStaff \with { instrumentName = "SID chip" }
	<<
		\new Staff = "up"
		{
			<<
				\new Voice = "voice1"
				{
					\voiceOne
					&lbrace;&lbrace;key&rbrace;&rbrace;
					&lbrace;&lbrace;tempo&rbrace;&rbrace;
					\set midiInstrument = #"acoustic guitar (steel)"
					\autochange
					\repeat volta 2
					{
						&lbrace;&lbrace;voice1&rbrace;&rbrace;
					}
				}

				\new Voice = "voice2"
				{
					\voiceTwo
					&lbrace;&lbrace;key&rbrace;&rbrace;
					\set midiInstrument = #"electric guitar (jazz)"
					\autochange
					\repeat volta 2
					{

						&lbrace;&lbrace;voice2&rbrace;&rbrace;
					}
				}
			>>
		}

		\new Staff = "down"
		{
			\clef bass
			&lbrace;&lbrace;key&rbrace;&rbrace;
		}
	>>

	\layout { }
	\midi
	{
		\context
		{
			\Staff
			\remove "Staff_performer"
		}
		\context
		{
			\Voice
			\consists "Staff_performer"
		}
	}
}
</pre>

This gives us a PDF and a MIDI file.

![Boulder Dash music](./Boulder_Dash.svg)
