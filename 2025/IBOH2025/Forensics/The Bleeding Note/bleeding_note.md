# Writeup - The Bleeding Note
## Challenge Description

> In the continuous war against The NeoCyberian Regime, FUN (Force Underground Network) was breached.  Our Net Watchers found unusual outbound traffic from a crucial node. They managed to get an artifact  from a memory dump believed to be a malware. They also captured the full traffic on the compromised  host moments before it went dark. It is believed that the malware contains crucial intel on which  data was being stolen.

**Flag format:** `BOH25{flag_content}`

The challenge description mentions that the malware inside the memory dump was responsible for suspicious outbound traffic and the goal is to analyze these files and recover the stolen intel. 2 files were given to download:

**Attachments:**  
1. [CapturedTraffic.pcap](./assets/CapturedTraffic.pcap)
2. [NeoEvilBeacon](./assets/NeoEvilBeacon)


## Step 1 - Initial Reconnaissance
Firstly, I used ```file FILENAME``` command to check the file.

<img width="940" height="94" alt="image" src="https://github.com/user-attachments/assets/3cba151b-6c0e-46dc-9c0a-00f3b57fb909" />

* This shows that the file is a standard PCAP network capture. 

* The timestamps are stored with microsecond precision. 

* The file uses little-endian byte order, common for Linux systems, and was captured from an Ethernet interface with a large capture buffer. 

* Little-endian means the computer stores the smallest (least significant) byte of a number first in memory. For example, A 4-byte number like 0x12345678 is stored in memory as: 78 56 34 12 (little-endian).

<img width="940" height="88" alt="image" src="https://github.com/user-attachments/assets/c7de2191-94fe-4927-b0a6-1a2ae61c27ce" />

* This file is a 64-bit little-endian Linux ELF executable designed for x86-64 systems. 

* A shared object is a Linux file (.so) that contains reusable code meant to be shared by multiple programs. 

* Statically linked means the program has all the libraries it needs already built inside it, so it does not rely on external .so files when it runs. 

* Besides that, “No section header” means the program has deliberately removed its internal map, making the file impossible to examine normally and hints it is packed or obfuscated.

After that, I used ```strings FILENAME``` command on the NeoEvilBeacon file to gain more insight and this revealed many clues and hints. 

<img width="400" height="300" alt="image" src="https://github.com/user-attachments/assets/f6206bd9-e712-4802-a694-7a583c8dcd5b" />

<img width="940" height="51" alt="image" src="https://github.com/user-attachments/assets/dd78e693-19f5-414d-b977-e63eb8246ab6" />

<img width="447" height="81" alt="image" src="https://github.com/user-attachments/assets/58e151d8-8848-40cc-bf39-7ce1744bf7d9" />

Some interesting strings found were:
1.	UPX! – hints that it could be UPX (Ultimate Packer for eXecutables) packed
2.	Vigen – hints Vigenere cipher
3.	Base64 – hints that there could be base 64 encoding
4.	An encrypted string which looks like base64 encoded data cmVtb3ZlTWVhbmRJc2hhbGx1bnBhY2s=

Next, I used ```hexdump -C FILENAME``` to get a clearer picture. hexdump -C shows the bytes of a file in hex and ASCII side-by-side, making it easier to inspect binary data and recognize patterns like headers, strings, or encoded sections.

<img width="500" height="600" alt="image" src="https://github.com/user-attachments/assets/c3aae876-fab8-4daa-9fac-e730a8c7bb76" />

The output shows that there is an inner ELF which indicates that the binary is packed or obfuscated, with the real executable payload hidden inside and only unpacked in memory at runtime.

---

## Step 2 - Base64 Decode
After that, I used CyberChef https://gchq.github.io/CyberChef/ which is a web app for encryption, encoding, compression and data analysis to decode the base64 string.

<img width="815" height="516" alt="image" src="https://github.com/user-attachments/assets/0fb52348-ecc4-47d7-b2d9-d6b654c0b928" />

This could also be done in terminal using command ```echo ‘BASE64 ENCODED STRING’ | base64 -d```

<img width="722" height="123" alt="image" src="https://github.com/user-attachments/assets/28e65db1-17ac-41e5-91ab-0628695720f6" />

This revealed that something needs to be removed and something will be unpacked.

---

## Step 3 - UPX Unpacking
UPX stands for Ultimate Packer for eXecutables and is used to compress executable files and libraries using command line across multiple operating systems.

To unpack the file, I used ```upx -d FILENAME``` to unpack the file. 

<img width="940" height="278" alt="image" src="https://github.com/user-attachments/assets/2b2ed226-5ee6-4f34-b588-244b1d10531e" />

However, the output revealed that 0 files were unpacked. So, this means that the file is not actually UPX packed but could be packed in a different way. And earlier when I ran file command, there were no statement of UPX packed.

---

## Step 4 - Execution of NeoEvilBeacon File
To save time, I decided to run the file and see if it is really packed with something.

I used command ```chmod +x FILENAME``` to make the file executable and  ```./FILENAME``` to run the file.

<img width="783" height="300" alt="image" src="https://github.com/user-attachments/assets/f6677e45-6733-4a98-a036-46697f777413" />

To my surprise, this revealed that the malware unpacks itself at runtime and the Base64 “removeMeandIshallunpack” string was not required. The UPX hints were also misdirections. 

* This program prints its entire configuration. 

* It shows that a C2 (Command and Control server) connection is initializing which means that the malware is preparing to communicate with its operator. 

* Besides that, it shows that the domain name is c2.malcorp.bizarre and it exfiltrates using DNS (Domain Name System) queries. 

* Furthermore, it shows that the data is encoded with Vigenère cipher with key “owned’ and later base64. These information gives me clear indication of where to look inside the network capture file.

---

## Step 5 - Analyzing the Network capture file
Firstly, I used Wireshark tool which is a free and open-source network protocol analyzer used to capture and inspect live network traffic or analyze saved data file to analyze the file. I used command wireshark FILENAME to run wireshark.

Based on the findings before this, I searched for c2.malcorp.bizarre and filtered using ```dns.qry.name contains “c2.malcorp.bizarre”```.

<img width="940" height="269" alt="image" src="https://github.com/user-attachments/assets/aa0b4cef-aaa9-400e-85de-323a01e0c5cb" />

After looking through, I noticed a pattern of short 3-character labels used in each DNS query with the final label ending in an equal sign, which could be base64 encoded data.

<img width="437" height="506" alt="image" src="https://github.com/user-attachments/assets/c194508b-0a60-4f2b-b1e3-32562ece6156" />

---

## Step 6 - Reconstructing Decoded Data
To see a clearer picture, I used tshark command to extract it and save the output into a text file: ```tshark -r CapturedTraffic.pcap -Y 'dns.qry.name contains "c2.malcorp.bizarre"' -T fields -e dns.qry.name > dns.txt```
-	-r  for read packets 
-	-Y to apply a display filter
-	-T fields to output specific fields instead of full packet information
-	-e dns.qry.name to print only the domain requested
-	> dns.txt to save the output into a file

Tshark is the command-line version of Wireshark that allows filtering and extracting network traffic directly from PCAP files using terminal commands.
 
<img width="400" height="595" alt="image" src="https://github.com/user-attachments/assets/2da5a179-fa7f-4b8a-a1ea-d434c8eb5328" />

Then, I deleted the duplicates and manually merged the pieces of base64 data to get ```UEtVMjV7ZzJfazQ1X3AzM2pfcTNnMGczcn0=```.

---

## Step 7 - Flag Retrieval
Next I used command line to decode it ```echo 'UEtVMjV7ZzJfazQ1X3AzM2pfcTNnMGczcn0=' | base64 -d```.

<img width="600" height="105" alt="image" src="https://github.com/user-attachments/assets/b8f199ad-14d4-41a9-b0ab-7b836865f86b" />

Decoded string: ```PKU25{g2_k45_p33j_q3g0g3r}```. This looks like a flag as it contains the flag format …25{…}.

From the previous information by running the program, I knew that the data was encoded with Vigenère cipher first then base 64. To get the flag, I needed to decode in reverse. I did the base64 decode and the next step is to decode with Vigenère cipher with key ‘owned. For this, I used CyberChef again. 

<img width="700" height="500" alt="image" src="https://github.com/user-attachments/assets/6afe097e-d10e-4da0-a68e-64addc187bef" />

**Flag:** ```BOH25{c2_h45_b33n_d3c0d3d}```



