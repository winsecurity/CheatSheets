# Pivoting 

Tags : #portforwarding #ssh #sshuttle #chisel

Before starting you need to enable port forwarding option in your kali linux box else no method will work

to do that edit ip_forward file and put 1 
this enables to act our kali as router

```
sudo echo 1 > /proc/sys/net/ipv4/ip_forward
```

---

if you want full explanation with theory i made these videos on yt 

SSH Tunneling
```
https://www.youtube.com/watch?v=2FRn-M7LOj4
```

Pivoting with Metasploit
```
https://www.youtube.com/watch?v=I3N2_arY9Kg
```

Pivoting with Chisel
```
https://www.youtube.com/watch?v=srUUUkcYEwg
```

---

## SSH Tunneling

This method requires ssh login credentials of the victim machine

### Lab Setup

kali linux - 192.168.2.129

machine1 - 192.168.2.128
					192.168.3.128
					
machine2 - 192.168.3.129


kali linux can ping machine1 but cannot ping machine2 
machine1 have another adapter that connects to machine2
machine1 can ping machine2 using that adapter

we can forward the data from our kali to machine3 using ssh


### Local Port Forwarding

we forward our kali's port to machine3 

```
$ ssh -L 4444:192.168.3.129:80 msfadmin@192.168.2.128
```

now data coming from our machine's port 4444 to machine2 will be forwarded to machine3's port 80 

This is like one-to-one port connection

kali:4444 -> machine2 -> machine3:80


### Dynamic Port Forwarding with SOCKS

in previous local port forwarding we can forward from one port to one ip's one port only
but with dynamic port forwarding , the machine2 will forward data according to what we send

```
ssh -D 4444 msfadmin@192.168.2.128
```

now we have been forwarded
we need to edit **proxychains4.conf** 
with proxychains we throw all our data at port 4444
it will be forwarded to destination via machine2

add this line at the end
```
socks5 	127.0.0.1  4444
```

now u can do nmap scan on machine3
or any command but include proxychains before that command

```
$ proxychains nmap -p80 -Pn -sV  192.168.3.129
[proxychains] config file found: /etc/proxychains4.conf
[proxychains] preloading /usr/lib/x86_64-linux-gnu/libproxychains.so.4
[proxychains] DLL init: proxychains-ng 4.14
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-30 10:15 EDT
[proxychains] Strict chain  ...  127.0.0.1:4444  ...  192.168.3.129:80  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:4444  ...  192.168.3.129:80  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:4444  ...  192.168.3.129:80  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:4444  ...  192.168.3.129:80  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:4444  ...  192.168.3.129:80  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:4444  ...  192.168.3.129:80  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:4444  ...  192.168.3.129:80  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:4444  ...  192.168.3.129:80  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:4444  ...  192.168.3.129:80  ...  OK
Nmap scan report for 192.168.3.129
Host is up (0.0041s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.2.8 ((Ubuntu) DAV/2)
```

we can see we scanned successfully
you can also see there is **Strict Chain**
```
[proxychains] Strict chain  ...  127.0.0.1:4444  ...  192.168.3.129:80  ...  OK
```
that shows our connection

kali:4444 -> machine2 -> any_machine_it_can_scan

kali:4444 -> machine2 -> machine3:anyport

if you want to open the machine3 port 80 on browser 
then go to settings > proxy > put the proxy as 127.0.0.1 4444
choose socks5

**Limitations** : you need ssh credentials or private key 

---

## Sshuttle

sshuttle is very easy to use and it creates vpn network for all the machines machine2 connected to
its similar to dynamic port forwarding 

sshuttle -r username@machine1 subnettoadd

```
$ sudo sshuttle -r msfadmin@192.168.2.128 192.168.3.1/24
The authenticity of host '192.168.2.128 (192.168.2.128)' can't be established.
RSA key fingerprint is SHA256:BQHm5EoHX9GCiOLuVscegPXLQOsuPs+E9d/rrJB84rk.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.2.128' (RSA) to the list of known hosts.
msfadmin@192.168.2.128's password: 
Traceback (most recent call last):
  File "<string>", line 1, in <module>
  File "assembler.py", line 27, in <module>
  File "sshuttle.helpers", line 97
    except OSError as e:
                    ^
SyntaxError: invalid syntax
c : fatal: c : server died with error code 1
```

in my kali , sshuttle is giving me error
but on ur box it works for sure

**Limitations** : sshuttle will not work on windows boxes , may be in future it may work 
so its better to use sshuttle in linux only environment

---

## Chisel

chisel is my favourite because it is cross platform and doesnot require any credentials 

chisel has two modes , **client** and **server** modes

on our kali box we use server mode and goes on listening

machine2 will connect to our chisel server using chisel binary and then will forward to other machines

download chisel linux binary for our kali and corresponding chisel binary for victim machine

```
https://github.com/jpillora/chisel/releases
```

transfer it to machine2 which we have reverse shell access

on our kali linux,run as reverse
reverse option allow incoming client to open a port on our kali and connect to destination specified

```
chisel server -p 5555 --reverse
```

on our reverseshell machine1
```
./chisel32 client 192.168.2.129:5555 R:1234:192.168.3.129:80
```

the above command connects to our kali and then 
R stands for remote host or our kali box
and whatever data comes from our box 1234 port that will be redirected to 192.168.3.129 port 80 
so go to browser and open 127.0.0.1:1234
you will see webpage of 192.168.3.129

```
$ ./chisel server -p 5555 --reverse                         
2021/05/30 23:06:57 server: Reverse tunnelling enabled
2021/05/30 23:06:57 server: Fingerprint ZHyDUORbA0i2GQhnaHza0PP3AX13BcDkehSnhlIM4is=
2021/05/30 23:06:57 server: Listening on http://0.0.0.0:5555
2021/05/30 23:07:42 server: session#1: tun: proxy#R:1234=>192.168.3.129:80: Listening
```

you can see connection 

kali:1234 -> machine1 -> machine2:80

now this is like one:one port 
we can make machine1 act as socks 

on our kali box , like always
```
chisel server -p 5555 --reverse
```

on our machine1 
```
./chisel32 client 192.168.2.129:5555 R:1234:socks
```

now machine1 acts as socks
we need to edit proxychains4.conf
add this line
```
socks5 	127.0.0.1  1234
```

now scan 192.168.3.129 with proxychains nmap
```
$ proxychains nmap -p80 -Pn -sV  192.168.3.129
[proxychains] config file found: /etc/proxychains4.conf
[proxychains] preloading /usr/lib/x86_64-linux-gnu/libproxychains.so.4
[proxychains] DLL init: proxychains-ng 4.14
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-30 23:14 EDT
[proxychains] Strict chain  ...  127.0.0.1:1234  ...  192.168.3.129:80  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1234  ...  192.168.3.129:80  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1234  ...  192.168.3.129:80  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1234  ...  192.168.3.129:80  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1234  ...  192.168.3.129:80  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1234  ...  192.168.3.129:80  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1234  ...  192.168.3.129:80  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1234  ...  192.168.3.129:80  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1234  ...  192.168.3.129:80  ...  OK
Nmap scan report for 192.168.3.129
Host is up (0.0033s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.2.8 ((Ubuntu) DAV/2)
```
now we can use any tool but put proxychains before the command

now let's add another machine 

**machine3 - 192.168.4.128**

only machine2 can ping machine3

now we compromised machine1 and machine2  
we have reverse shell on machine2
inorder to pivot through machine2 to machine3

you need to setup a chisel server on machine1 and machine2 will connect to it and acts as socks

let's do it to get more clarity 

on my kali 
```
chisel server -p 5555 --reverse
```

on machine1

```
./chisel32 client 192.168.2.129:5555 R:1234:socks
```

now grab another shell on machine1 in the way you got first shell

now we  need to run this chisel as server on machine1
```
./chisel32 server -p 6666 --reverse
```

now on machine2 
```
./chisel32 client 192.168.3.128:6666 R:7777:socks
```

now this will redirect our traffic to 192.168.4.0/24

we are almost done

we just need to edit proxychains4.conf
add these two lines
```
socks5 127.0.0.1  1234
socks5 127.0.0.1  7777
```

run proxychains nmap scan
```
$ proxychains nmap -p80 -Pn -sV  192.168.4.128 
[proxychains] config file found: /etc/proxychains4.conf
[proxychains] preloading /usr/lib/x86_64-linux-gnu/libproxychains.so.4
[proxychains] DLL init: proxychains-ng 4.14
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-30 23:43 EDT
[proxychains] Strict chain  ...  127.0.0.1:1234  ...  127.0.0.1:7777  ...  192.168.4.128:80  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1234  ...  127.0.0.1:7777  ...  192.168.4.128:80  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1234  ...  127.0.0.1:7777  ...  192.168.4.128:80  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1234  ...  127.0.0.1:7777  ...  192.168.4.128:80  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1234  ...  127.0.0.1:7777  ...  192.168.4.128:80  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1234  ...  127.0.0.1:7777  ...  192.168.4.128:80  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1234  ...  127.0.0.1:7777  ...  192.168.4.128:80  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1234  ...  127.0.0.1:7777  ...  192.168.4.128:80  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1234  ...  127.0.0.1:7777  ...  192.168.4.128:80  ...  OK
Nmap scan report for 192.168.4.128
Host is up (0.055s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.2.8 ((Ubuntu) DAV/2)
```

you can see the chain from 1234 -> 7777 and then to .4 subnet

kali:1234 -> machine1:7777 -> machine2 -> machine3:anyport

if you have other machines you can repeat the same process 
now machine1,2,3 acts as socks

---

## Metasploit

there is route command u can add route but
metasploit have modules which add route to the subnet of compromised machine
this autoroute is similar to route add command

**autoroute** post module
```
post/multi/manage/autoroute 
```

use that one 
set options
```
set SESSION 1
set SUBNET 192.168.3.0
run
```

```
msf6 post(multi/manage/autoroute) > run

[!] SESSION may not be compatible with this module.
[*] Running module against metasploitable.localdomain
[*] Searching for subnets to autoroute.
[+] Route added to subnet 192.168.3.0/255.255.255.0 from host's routing table.
[+] Route added to subnet 192.168.2.0/255.255.255.0 from host's routing table.

```

now use any module to scan 192.168.3.129

```
auxiliary/scanner/portscan/tcp
```

```
msf6 auxiliary(scanner/portscan/tcp) > run

[+] 192.168.3.129:        - 192.168.3.129:23 - TCP OPEN
[+] 192.168.3.129:        - 192.168.3.129:22 - TCP OPEN
[+] 192.168.3.129:        - 192.168.3.129:21 - TCP OPEN
[+] 192.168.3.129:        - 192.168.3.129:25 - TCP OPEN
[+] 192.168.3.129:        - 192.168.3.129:53 - TCP OPEN
[+] 192.168.3.129:        - 192.168.3.129:80 - TCP OPEN
[+] 192.168.3.129:        - 192.168.3.129:111 - TCP OPEN
[+] 192.168.3.129:        - 192.168.3.129:139 - TCP OPEN
[+] 192.168.3.129:        - 192.168.3.129:445 - TCP OPEN
[+] 192.168.3.129:        - 192.168.3.129:513 - TCP OPEN
[+] 192.168.3.129:        - 192.168.3.129:514 - TCP OPEN
[+] 192.168.3.129:        - 192.168.3.129:512 - TCP OPEN
[*] 192.168.3.129:        - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```
we can see the open ports

**meterpreter portfwd**

in meterpreter session we can use portfwd command to forward to destination
```
meterpreter > portfwd -h
Usage: portfwd [-h] [add | delete | list | flush] [args]


OPTIONS:

    -L <opt>  Forward: local host to listen on (optional). Reverse: local host to connect to.
    -R        Indicates a reverse port forward.
    -h        Help banner.
    -i <opt>  Index of the port forward entry to interact with (see the "list" command).
    -l <opt>  Forward: local port to listen on. Reverse: local port to connect to.
    -p <opt>  Forward: remote port to connect to. Reverse: remote port to listen on.
    -r <opt>  Forward: remote host to connect to.
```

now lets forward to 192.168.3.129

```
meterpreter > portfwd add -l 4444 -p 80 -r 192.168.3.129
[*] Local TCP relay created: :4444 <-> 192.168.3.129:80
```

now go to browser  and open 127.0.0.1:4444
you can see webpage of 192.168.3.129

now its like one:one port

lets use socks module in metasploit
make sure route is added else this will not work 

```
use auxiliary/server/socks_proxy 
```

now you can change SRVPORT 
```
run
```
now socks server is up 

edit /etc/proxychains4.conf
```
socks5  127.0.0.1  1080
```
now do proxychains nmap scan

```
$ proxychains nmap -p80 -Pn -sV  192.168.3.129
[proxychains] config file found: /etc/proxychains4.conf
[proxychains] preloading /usr/lib/x86_64-linux-gnu/libproxychains.so.4
[proxychains] DLL init: proxychains-ng 4.14
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-31 00:38 EDT
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  192.168.3.129:80  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  192.168.3.129:80  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  192.168.3.129:80  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  192.168.3.129:80  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  192.168.3.129:80  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  192.168.3.129:80  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  192.168.3.129:80  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  192.168.3.129:80  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  192.168.3.129:80  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  192.168.3.129:80  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  192.168.3.129:80  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  192.168.3.129:80  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  192.168.3.129:80  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  192.168.3.129:80  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  192.168.3.129:80  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  192.168.3.129:80  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  192.168.3.129:80  ...  OK
Nmap scan report for 192.168.3.129
Host is up (0.0099s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.2.8 (DAV/2)
```


