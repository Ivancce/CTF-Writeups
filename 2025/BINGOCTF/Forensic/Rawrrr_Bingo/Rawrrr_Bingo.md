# Writeup - Rawrrr_Bingo 
## Challenge Description

>When the bingo caller calls, we listen.

**Flag Format:** `BINGOCTF{flag}`

**Attachments:** ![B!nG0.mp3](./assets/B!nG0.mp3)

## Steps to Solve

1. Firstly, using earphones to listen to the audio, we can hear a brief hello and noises after.

2. Using `file` command to see that it is stereo audio.
```bash
└─$ file B\!nG0.raw     
B!nG0.raw: Audio file with ID3 version 2.4.0, contains: MPEG ADTS, layer III, v1, 64 kbps, 44.1 kHz, Stereo
```

3. Use Audacity for further analysis.

<img width="1870" height="363" alt="image" src="https://github.com/user-attachments/assets/441d34d9-202d-45e0-af31-101e1806850e" />

We can see that the left audio looks normal but the right audio has noises.

4. Listen to only the left audio. Mute the right audio and raise the volume to get the flag. The flag is at the later part of the audio.
<img width="181" height="148" alt="image" src="https://github.com/user-attachments/assets/574e155a-652c-4db0-a636-50dc861f66aa" />

**Flag:** `BINGOCTF{b1ng0_w1ns}`
