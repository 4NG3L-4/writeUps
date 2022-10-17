# Lab 3 - The Evil's Den
## *Challenge Description:*
- [MemLabs repo: Lab 3](https://github.com/stuxnet999/MemLabs/tree/master/Lab%203)
- [Image Link](https://mega.nz/#!2ohlTAzL!1T5iGzhUWdn88zS1yrDJA06yUouZxC-VstzXFSRuzVg)
- Tools: Volatility Framework 2.6.1, Steghide
- [Volatility cheat sheet](https://downloads.volatilityfoundation.org/releases/2.4/CheatSheet_v2.4.pdf)
## *Solution:*
1. General Analysis:
```
    vol.py -f MemoryDump_Lab3.raw imageinfo
        ...
        Suggested Profile(s) : Win7SP1x86_23418, Win7SP0x86, Win7SP1x86_24000, Win7SP1x86
        ...
```

```
    vol.py -f MemoryDump_Lab3.raw --profile=Win7SP1x86_23418 pslist
        ...
        0x8449d890 explorer.exe
        0xbc21b9f0 TrustedInstall
        0x9c6b0970 notepad.exe
        ...
```
2. Since Internet Explorer were running, let's check its history
```
    vol.py -f MemoryDump_Lab3.raw --profile=Win7SP1x86_23418 iehistory
        ...
        URL: hello@file:///C:/Users/hello/Desktop/evilscript.py
        ...
        Location: Visited: hello@file:///C:/Users/hello/Desktop/vip.txt
        ...
        Location: Visited: hello@file:///C:/Users/hello/Desktop/suspision1.jpeg
        ...
```

3. The evil script: 
```
import sys
import string

def xor(s):

        a = ''.join(chr(ord(i)^3) for i in s)
        return a


def encoder(x):

        return x.encode("base64")


if __name__ == "__main__":

        f = open("C:\\Users\\hello\\Desktop\\vip.txt", "w")

        arr = sys.argv[1]

        arr = encoder(xor(arr))

        f.write(arr)

        f.close()
```
We can understand that it xores "vip.txt" content with "3" then encodes it with base64. If we reverse it we get: 
> inctf{0n3_h4lf
4.  Now we move to the jpeg analysis, the description provided us with a hint which is using steghide
```
    steghide extract -sf  suspison1.jpeg
    Enter passphrase: inctf{0n3_h4lf
    wrote extracted data to "secret text".
```
And by reading the secret text we get the second part of the flag.
## *Flag*
inctf{0n3_h4lf_1s_n0t_3n0ugh}
