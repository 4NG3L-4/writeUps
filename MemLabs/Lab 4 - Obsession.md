# Lab 3 - Obsession
## *Challenge Description:*
- [MemLabs repo: Lab 4](https://github.com/stuxnet999/MemLabs/tree/master/Lab%204)
- "My system was recently compromised. The Hacker stole a lot of information but he also deleted a very important file of mine. I have no idea on how to recover it. The only evidence we have, at this point of time is this memory dump. Please help me."
- [Image Link](https://mega.nz/#!Tx41jC5K!ifdu9DUair0sHncj5QWImJovfxixcAY-gt72mCXmYrE)
- Tools: Volatility Framework 2.6.1
- [Volatility cheat sheet](https://downloads.volatilityfoundation.org/releases/2.4/CheatSheet_v2.4.pdf)
## *Solution:*
1. General Analysis:
```
    vol.py -f MemoryDump_Lab4.raw imageinfo
        ...
        Suggested Profile(s) : Win7SP1x64, Win7SP0x64, Win2008R2SP0x64, Win2008R2SP1x64_24000, Win2008R2SP1x64_23418, Win2008R2SP1x64, Win7SP1x64_24000, Win7SP1x64_23418
        ...
```

```
    vol.py -f MemoryDump_Lab4.raw --profile=Win7SP1x64 pslist
        ...
        0xfffffa8000eaeb30 explorer.exe           3012
        0xfffffa8000f18b30 StikyNot.exe           2432
        ...
```
2. Checking Internet explorer history:
```
    vol.py -f MemoryDump_Lab4.raw --profile=Win7SP1x64 iehistory
        ...
        Location: Visited: eminem@file:///C:/Users/eminem/Downloads/galf.jpeg
        ...
        Location: Visited: eminem@https://volatilevirus.home.blog/comments/feed
        ...
        Location: Visited: eminem@file:///C:/Users/eminem/Desktop/secrets.txt
        ...
        Location: Visited: eminem@file:///C:/Users/eminem/Desktop/flag.txt.txt
        ...
        Location: Visited: eminem@file:///C:/Users/eminem/Desktop/Important.txt
        ...
        Location: :2019062920190630: eminem@file:///C:/Users/eminem/Desktop/Flag%20not%20here.bmp
        ...
        Location: :2019062920190630: SlimShady@file:///C:/Users/SlimShady/Desktop/Important.txt
        ...

```
3. For now we can conclude there's two users eminem and SlimShady. Since, We don't have any hint about who's the hacker (Assuming the who wasn't impoortant). We proceed in dumping the files mentioned in the history. Nothing interesting except a hidden "LMAO.txt" in galf.jpeg that says:
> Move on bro, there is nothing here :)
4.  Trusting it, we move to StikyNot.exe analysis 
```
    vol.py -f MemoryDump_Lab4.raw --profile=Win7SP1x64 memdump -p 2432 -D .
        ************************************************************************
        Writing StikyNot.exe [  2432] to 2432.dmp
```
And by looking for interesting strings we get:
> The clipboard plugin works well but it doesn't give the flag :P
5. Back to the description, we look for deleted files:
```
    vol.py -f MemoryDump_Lab4.raw --profile=Win7SP1x64 mftparser > mft.txt         
```
Using a text editor and looking for files names listed before we get:
```
$FILE_NAME
Creation                       Modified                       MFT Altered                    Access Date                    Name/Path
------------------------------ ------------------------------ ------------------------------ ------------------------------ ---------
2019-06-27 13:14:13 UTC+0000 2019-06-27 13:14:13 UTC+0000   2019-06-27 13:14:13 UTC+0000   2019-06-27 13:14:13 UTC+0000   Users\SlimShady\Desktop\Important.txt

$OBJECT_ID
Object ID: 7726a550-d498-e911-9cc1-0800275e72bc
Birth Volume ID: 80000000-b800-0000-0000-180000000100
Birth Object ID: 99000000-1800-0000-690d-0a0d0a0d0a6e
Birth Domain ID: 0d0a0d0a-0d0a-6374-0d0a-0d0a0d0a0d0a

$DATA
0000000000: 69 0d 0a 0d 0a 0d 0a 6e 0d 0a 0d 0a 0d 0a 63 74   i......n......ct
0000000010: 0d 0a 0d 0a 0d 0a 0d 0a 66 7b 31 0d 0a 0d 0a 0d   ........f{1.....
0000000020: 0a 5f 69 73 0d 0a 0d 0a 0d 0a 5f 6e 30 74 0d 0a   ._is......_n0t..
0000000030: 0d 0a 0d 0a 0d 0a 5f 45 51 75 34 6c 0d 0a 0d 0a   ......_EQu4l....
0000000040: 0d 0a 0d 0a 5f 37 6f 5f 32 5f 62 55 74 0d 0a 0d   ...._7o_2_bUt...
0000000050: 0a 0d 0a 0d 0a 0d 0a 0d 0a 0d 0a 5f 74 68 31 73   ..........._th1s
0000000060: 5f 64 30 73 33 6e 74 0d 0a 0d 0a 0d 0a 0d 0a 5f   _d0s3nt........_
0000000070: 6d 34 6b 65 0d 0a 0d 0a 0d 0a 5f 73 33 6e 0d 0a   m4ke......_s3n..
0000000080: 0d 0a 0d 0a 0d 0a 73 33 7d 0d 0a 0d 0a 47 6f 6f   ......s3}....Goo
0000000090: 64 20 77 6f 72 6b 20 3a 50                        d.work.:P

***************************************************************************
```
## *Flag*
inctf{1_is_n0t_EQu4l_7o_2_bUt_th1s_d0s3nt_m4ke_s3ns3}
