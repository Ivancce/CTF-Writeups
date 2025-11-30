# Writeup - Easy Fix 
## Challenge Description

>so my lab partner decided to be "helpful" and threw my Picture iNto the Green bin, claiming it was "corrupted beyond repair." then she spent three hours at her computer going "hehe" and "trust me bro, i got this."  
she peaced out for the weekend muttering "damn, that's smaller of a size than i expected" while giggling uncontrollably, and now i'm stuck trying to figure out what she broke, what she fixed, and what she hid in the process.  
this is what i get for sharing a workspace with a chaotic person. help?


**Flag format:** `BINGOCTF{flag}`


**Attachment:** [help_fix.pcap](./assets/help_fix.pcap)


**Hints from Description**
- “Picture iNto the Green bin” → probably an image inside the PCAP
- “corrupted beyond repair” → something being corrupted
- “smaller of a size than I expected” → something about size / resolution changed, not just corruption


## Steps to solve
Knowing that I needed to export something from the pcap file. I went straight to using wireshark to seacrh for export object. I managed to find one in the http export and proceeded to export it.


`File -> Export Object -> http`


<img width="700" height="120" alt="export" src="https://github.com/user-attachments/assets/73378d35-74f2-47ec-851a-ace141fdf392" />


**Exported:** [can_you_fix_meeee.bin](./assets/can_you_fix_meeee.bin)

Next is to check the file and we can see the file with .bin extension is actually a `PNG`.
```bash
└─$ file can_you_fix_meeee.bin 
can_you_fix_meeee.bin: PNG image data, 800 x 700, 8-bit/color RGBA, non-interlaced
```
