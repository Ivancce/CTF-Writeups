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


## Step 1 - Export File in Wireshark
Knowing that I needed to export something from the pcap file. I went straight to using wireshark to seacrh for export object. I managed to find one in the http export and proceeded to export it.


`File -> Export Object -> http`


<img width="700" height="120" alt="export" src="https://github.com/user-attachments/assets/73378d35-74f2-47ec-851a-ace141fdf392" />


**Exported:** [can_you_fix_meeee.bin](./assets/can_you_fix_meeee.bin)

---

## Step 2 - Metadata Analysis
1. Run `file` command and we can see the file with .bin extension is actually a `PNG`.
```bash
└─$ file can_you_fix_meeee.bin 
can_you_fix_meeee.bin: PNG image data, 800 x 700, 8-bit/color RGBA, non-interlaced
```

2. Next, check `exiftool` and we can see many information here.
```bash
└─$ exiftool can_you_fix_meeee.bin 
ExifTool Version Number         : 13.36
File Name                       : can_you_fix_meeee.bin
Directory                       : .
File Size                       : 61 kB
File Modification Date/Time     : 2025:11:28 23:02:46-05:00
File Access Date/Time           : 2025:11:30 04:17:43-05:00
File Inode Change Date/Time     : 2025:11:28 23:02:46-05:00
File Permissions                : -rw-r--r--
File Type                       : PNG
File Type Extension             : png
MIME Type                       : image/png
Image Width                     : 800
Image Height                    : 700
Bit Depth                       : 8
Color Type                      : RGB with Alpha
Compression                     : Deflate/Inflate
Filter                          : Adaptive
Interlace                       : Noninterlaced
Exif Byte Order                 : Little-endian (Intel, II)
Bits Per Sample                 : 8 8 8
Image Description               : i want to fixxxxxx helppp.-.-.-.-.-.-.-.-.-.-.-.-.-.-.-.-.-.-.-.-.-.-.-.-BINGOCTF{keep_going!_not_this}
Orientation                     : Horizontal (normal)
X Resolution                    : 118.1098901
Y Resolution                    : 118.1098901
Resolution Unit                 : cm
Software                        : GIMP 3.0.6
Modify Date                     : 2025:11:15 07:17:02
Artist                          : BINGOCTF{fake_flag_hehe}
Offset Time                     : +08:00
User Comment                    : i want to fixxxxxx helppp.-.-.-.-.-.-.-.-.-.-.-.-.-.-.-.-.-.-.-.-.-.-.-.-BINGOCTF{keep_going!_not_this}
Color Space                     : sRGB
GPS Altitude                    : 0 m
By-line                         : BINGOCTF{fake_flag_hehe}
By-line Title                   : bob_the_fixer?
Caption-Abstract                : Photo taken at 2025. 800
Keywords                        : https://www.youtube.com/watch?v=dQw4w9WgXcQ
Object Name                     : fixfixfixfixfixfixfixfix
XMP Toolkit                     : XMP Core 4.4.0-Exiv2
Document ID                     : gimp:docid:gimp:74b250ec-7b98-4438-bcbd-78b717c4d60b
Instance ID                     : xmp.iid:9eade4ae-5a54-4122-a88e-de4a87e1026e
Original Document ID            : xmp.did:334a8723-3846-4906-a634-5293bb476c64
Api                             : 3.0
Platform                        : Windows
Time Stamp                      : 1763162475383392
Version                         : 3.0.6
Format                          : image/png
Authors Position                : bob_the_fixer?
Creator Tool                    : GIMP
Metadata Date                   : 2025:11:15 07:17:02+08:00
Web Statement                   : BINGOCTF{n0_y0u_
History Action                  : saved, saved, saved
History Changed                 : /metadata, /metadata, /
History Instance ID             : xmp.iid:40306fa7-b39b-4a47-8d61-bc77639aeea8, xmp.iid:5e41bfd9-59cf-4310-ab6e-96e0e08b6e9c, xmp.iid:81b8e514-b265-4a10-a69c-5f90fdbefdfb
History Software Agent          : GIMP 3.0.6 (Windows), GIMP 3.0.6 (Windows), GIMP 3.0.6 (Windows)
History When                    : 2025:11:15 06:54:47+08:00, 2025:11:15 07:16:51+08:00, 2025:11:15 07:21:15+08:00
Creator                         : BINGOCTF{fake_flag_hehe}
Description                     : Photo taken at 2025. 800
Subject                         : https://www.youtube.com/watch?v=dQw4w9WgXcQ
Title                           : fixfixfixfixfixfixfixfix
SRGB Rendering                  : Relative Colorimetric
Gamma                           : 2.2
White Point X                   : 0.3127
White Point Y                   : 0.329
Red X                           : 0.64
Red Y                           : 0.33
Green X                         : 0.3
Green Y                         : 0.6
Blue X                          : 0.15
Blue Y                          : 0.06
Background Color                : 255 255 255
Pixels Per Unit X               : 11811
Pixels Per Unit Y               : 11811
Pixel Units                     : meters
Comment                         : i want to fixxxxxx helppp.-.-.-.-.-.-.-.-.-.-.-.-.-.-.-.-.-.-.-.-.-.-.-.-BINGOCTF{keep_going!_not_this}
Warning                         : [minor] Trailer data after PNG IEND chunk [x2]
Image Size                      : 800x700
Megapixels                      : 0.560
Modify Date                     : 2025:11:15 07:17:02+08:00

```

Key findings:
- many fake flags
- `BINGOCTF{n0_y0u_` in web statement which looks like the beginning of the flag
- Warning telling us that there is trailer data after IEND chunk which means we need to fix it.

3. Running `pngcheck` to double check.
```bash
└─$ pngcheck -v can_you_fix_meeee.bin 
File: can_you_fix_meeee.bin (61120 bytes)
  chunk IHDR at offset 0x0000c, length 13
    800 x 700 image, 32-bit RGB+alpha, non-interlaced
  CRC error in chunk IHDR (computed 17a3dbe4, expected db700668)
ERRORS DETECTED in can_you_fix_meeee.bin
```
The CRC is expecting `db700668`.

*What is CRC?*

>CRC = Cyclic Redundancy Check  
CRC = CRC32("IHDR" + [13 bytes of IHDR data])  
It is a 4-byte checksum that verifies the data is correct.  

4. Next, going through `strings` command, I found string that looks like the ending of the flag.
```bash
└─$ strings can_you_fix_meeee.bin | tail -n 10 
c4"ND
P4fRVF
vk@N
)X0G-^\
5>>^
3>>^
;f'f
mvBO
IEND
m3_br0}
```

Now we have `BINGOCTF{n0_y0u_` and `m3_br0}` so we need to find the middle part.

---

Step 3 - Fixing PNG

```bash
└─$ xxd can_you_fix_meeee.bin | head

00000000: 8950 4e47 0d0a 1a0a 0000 000d 4948 4452  .PNG........IHDR
00000010: 0000 0320 0000 02bc 0806 0000 00db 7006  ... ..........p.
00000020: 6800 0002 1e65 5849 6649 492a 0008 0000  h....eXIfII*....
00000030: 000d 0000 0104 0001 0000 0020 0300 0001  ........... ....
00000040: 0104 0001 0000 0020 0300 0002 0103 0003  ....... ........
00000050: 0000 00aa 0000 000e 0102 0068 0000 00b0  ...........h....
00000060: 0000 0012 0103 0001 0000 0001 0000 001a  ................
00000070: 0105 0001 0000 0018 0100 001b 0105 0001  ................
00000080: 0000 0020 0100 0028 0103 0001 0000 0003  ... ...(........
00000090: 0000 0031 0102 000b 0000 0028 0100 0032  ...1.......(...2
```

** Why PNG was broken? **

`49 48 44 52  00 00 03 20  00 00 02 BC  08 06 00 00 00  
   IHDR      width=800     height=700`

When pngcheck recomputed CRC for 800×700, it got:

- 17A3DBE4   ← WRONG

But when we recompute CRC using 800×800, the CRC matches:

- DB 70 06 68   ← CORRECT

Use HxD or https://hexed.it/ to edit the bytes and the fixed image:

`00 00 02 BC -> 00 00 03 20`


Combining the pieces together to get the flag.

**Flag:** `BINGOCTF{n0_y0u_c@nT_tr5$T_m3_br0}`
