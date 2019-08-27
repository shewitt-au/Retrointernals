# Boulder Dash Theme Music

Ok, I got the idea to score the Boulder Dash music into my head. I had a look around to see if anyone else had already done so, and they had, kind of. The man himself has a crack [here](https://www.brainjam.ca/wp/2009/11/scoring-the-boulder-dash-theme/). But with some knowledge of the limitations of the music routine it’s clear it’s not a faithful representation, more of an arrangement; the routine doesn’t do rests, or notes of any but one duration. There are other scores out there, but all of them seem to be arrangements of the theme and not faithful representations. Let's rectify that.

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

The music routine uses this table to map from note values to SID frequency values. Each entry is two bytes long. We need to do two things here:

	1. Map from SID frequency value to a frequency.
	2. Map from a frequency to a note.

We'll take one at a time then tie it all together.

### Mapping from a SID frequency value to a frequency

I Googled a few resources when trying to figure out how to do this. I did briefly consider getting an oscilloscope and verifying what I learnt, but I decided that was probably overkill. We'll interpret consistency across multiple sources as verification enough. Anyway, here are some links if you're interested in reading up on this:


[The SID (by Stephen L. Judd)](http://www.cbmitapages.it/c64/sid1eng.htm)

[How to calculate SID frequency tables By FTC/HT](https://codebase64.org/doku.php?id=base:how_to_calculate_your_own_sid_frequency_table)

[Excact Vertical Frequency / Refresh Rate](https://csdb.dk/forums/?roomid=11&topicid=124823&firstpost=2)

I hacked together the Python code below to get the frequencies of the notes. I live in Australia, so we had PAL machines. I'm only going to consider the PAL case but I've included the code for NTSC machines.

```python
#!/usr/bin/env python3

bd_sid_values = [
0xdc, 0x02, 0x0a, 0x03,  0x3a, 0x03, 0x6c, 0x03,  0xa0, 0x03, 0xd2, 0x03,  0x12, 0x04, 0x4c, 0x04,
0x92, 0x04, 0xd6, 0x04,  0x20, 0x05, 0x6e, 0x05,  0xb8, 0x05, 0x14, 0x06,  0x74, 0x06, 0xd8, 0x06,
0x40, 0x07, 0xa4, 0x07,  0x24, 0x08, 0x98, 0x08,  0x24, 0x09, 0xac, 0x09,  0x40, 0x0a, 0xdc, 0x0a,
0x70, 0x0b, 0x28, 0x0c,  0xe8, 0x0c, 0xb0, 0x0d,  0x80, 0x0e, 0x48, 0x0f,  0x48, 0x10, 0x30, 0x11,
0x48, 0x12, 0x58, 0x13,  0x80, 0x14, 0xb8, 0x15,  0xe0, 0x16, 0x50, 0x18,  0xd0, 0x19, 0x60, 0x1b,
0x00, 0x1d, 0x90, 0x1e,  0x90, 0x20, 0x60, 0x22,  0x90, 0x24, 0xb0, 0x26,  0x00, 0x29, 0x70, 0x2b,
0xc0, 0x2d]

def chunks():
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

print("PAL\n---")
for f in chunks():
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

The closest value to A440 is 435.97705078124994 which is 16 cents below. A perceptible distance. If we use A435 instead this is still the closest value but this time 4 cents higher. That's more like it. Well use this entry as our A4. Here is some info on [cents](https://en.wikipedia.org/wiki/Cent_(music)) and [A440](https://en.wikipedia.org/wiki/A440_(pitch_standard)) for the curious.

### Maping from a frequency to a note

This requires some understanding of the maths behind musical tunings. When you start looking into this you can help but wonder how deep the rabbit hole goes. For our purposes we'll only concern ourselves with the [equal temperament](https://en.wikipedia.org/wiki/Equal_temperament) tuning system. To cut a long story short the following Python code will give us a note number measured in semitones from the reference frequency a4.

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

def chunks():
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

bd_val = 0x14
for sid in chunks():
	freq = reg_to_freq_pal(sid)
	idx = round(freq_to_note(freq))
	print("${:02x} : {}".format(bd_val, index_to_name(idx, True)))
	bd_val += 1
print()
```

Gives us the note names that correspond to the note numbering Boulder Dash uses:

	$14 : f1
	$15 : f1♯
	$16 : g1
	$17 : g1♯
	$18 : a1
	$19 : a1♯
	$1a : b1
	$1b : c2
	$1c : c2♯
	$1d : d2
	$1e : d2♯
	$1f : e2
	$20 : f2
	$21 : f2♯
	$22 : g2
	$23 : g2♯
	$24 : a2
	$25 : a2♯
	$26 : b2
	$27 : c3
	$28 : c3♯
	$29 : d3
	$2a : d3♯
	$2b : e3
	$2c : f3
	$2d : f3♯
	$2e : g3
	$2f : g3♯
	$30 : a3
	$31 : a3♯
	$32 : b3
	$33 : c4
	$34 : c4♯
	$35 : d4
	$36 : d4♯
	$37 : e4
	$38 : f4
	$39 : f4♯
	$3a : g4
	$3b : g4♯
	$3c : a4
	$3d : a4♯
	$3e : b4
	$3f : c5
	$40 : c5♯
	$41 : d5
	$42 : d5♯
	$43 : e5
	$44 : f5

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

Again, this is simply 128 pairs of note values, one for each voice. The first number of the pair is the note for voice 2 and the second the note for voice 1. $14 is the first note value and $44 the last (assuming a tuning as described above that f1 and f5 respectively). Increasing a note value by one moves up  a semitone.

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


![Boulder Dash music](./Boulder_Dash.svg)
