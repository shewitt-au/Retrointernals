# Boulder Dash Theme Music

Ok, I got the idea to score the Boulder Dash music it into my head. I had a look around to see if anyone else had already done so, and they had, kind of. The man himself has a crack [here](https://www.brainjam.ca/wp/2009/11/scoring-the-boulder-dash-theme/). But with some knowledge of the limitations of the music routine it’s clear it’s not a faithful representation, more of an arrangement; the routine doesn’t do rests, or notes of any but one duration. There are other scores out there, but all of them seem to be arrangements of the theme and not faithful representations. Let's rectify that.

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

The music player routine uses note numbers to index into this table to get the SID values used to play the note of the desired frequency. Each entry is two bytes long. I hacked together the Python code below to get the frequencies of the notes. I live in Australia, so we had PAL machines. I'm only going to consider the PAL case but I've included the code for NTSC machines.

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

## Tempo

The music routine is called once per frame but only does anything every other frame. There's some weirdness going on here. Why not just call the routine on every other frame instead of calling it every frame and having it do nothing half of the time? Function calls aren't free.

<pre>
<b>MusicTickRoutine</b>:
<b>83ac</b>   a5 c1      LDA <b>TitleScreenFrameCountMod4</b>
<b>83ae</b>   29 01      AND #$01
<b>83b0</b>   d0 01      BNE $83b3
<b>83b2</b>   60         RTS 
</pre>

So on every other frame:

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

OK, there's some weirdness going on here too. Whether the routine advances to the next note (actually the next note for each of the two voices used) or just modulates the amplitude of voice one is determined by **MusicTickRoutine__Voice1SustainLevel**. This is initialised to $a0 before the music starts and a new note is only played when it's $a0. The lower three bits are incremented every other frame but the $a in the high nybble is entirely pointless. The upshot of all this is the song advances to the next "beat" every 16 frames. We'll use quavers (eighths notes) for the transcription. Given that PAL has around 50 frames per second this gives us a BPM of 187.5. We'll call it 188. 

![Boulder Dash music](./Boulder_Dash.svg)


