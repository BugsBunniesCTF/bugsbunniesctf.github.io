---
author: HDhotdog
ctf: nahamcon-ctf-2022
type: writeup
title: Call me Picasso
tags: forensics
---

Call me Picasso provided a mystery.zip which contained an .iq file containing [I/Q data](https://en.wikipedia.org/wiki/In-phase_and_quadrature_components). 
There are various tools to convert this data into a simple wave audio file, for example [sox](https://github.com/chirlu/sox).

```bash

sox -e float -t raw -r 44100 -b 32 -c 1 raw_data.iq -t wav -e float -b 32 -c 1 -r 44100 out.wav
```

First I opened the .wav file in vlc but it is over an hour long and only consists of noise. 
But the challenge name is "Call me Picasso" so it has to do something with paintings, amirite? Therefore I took a look at the spectrogram using Sonic Visualiser and indeed there was a picture of John Hammond and the flag. 

![](https://cdn.discordapp.com/attachments/969952004648607754/970261401438412851/unknown32.png)

And now the real challenge began, actually reading the flag. Afterwards I learned that you can change contrasts and colors in Sonic Visualiser but by then I already managed to read it correctly.

flag{c56b87fdb0ffe5ad2e9dccc806562da8}
