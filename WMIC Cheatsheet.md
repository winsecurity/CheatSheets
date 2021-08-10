# WMI Console

Tags : #wmic


WMI is windows management interface . Its used to retrieve information of windows computers in local network

WMIC is console edition of WMI that uses command line , there are powershell and also programming language support for these wmi 

type wmic /? to get all options available for wmi
```
wmic /?
```

---

 ## To know about BIOS information 
 
```
wmic bios
```

if you want to truncate output or want only brief information you can use `list brief`
```
wmic bios list brief
```

if we want to query the remote computer 
we need to specify /node switch and /user and /password 
```
wmic /node:10.2.2.3 /user:arrow /password:arrowverse bios list brief
```

you can select particular columns also
```
wmic bios get manufacturer,name
```

you can get all available switches for particular class using /?
```
wmic bios /? 
wmic bios get /?
```

---

## Knowing about computersystem

```
wmic cpu list brief
```

to know about manufacturer , model name ,desktop name etc
```
wmic computersystem
wmic computersystem list brief
```

to know about users of the computer
```
wmic desktop get name
```


## Knowing about disk drives and partitions

lets view all physical harddrives connected
```
wmic diskdrive list brief
wmic diskdrive get partitions
```

now we can see partitions , lets get those information too
```
wmic partition list brief
```

this command gives much more information
```
wmic partition get name,size,index,type,bootable,primarypartition
```


## Knowing Environment variables

wmic environment gives all the variables and paths
```
wmic environment list brief
```

you can see mappings of **TEMP,Path**


## Knowing Groups and users information

lets get users information
```
wmic useraccount list brief
```

and groups information
```
wmic group list brief
```


## Knowing about operating system

```
wmic os
wmic os list brief
```

this command gives useful information
```
wmic os get buildnumber,buildtype,csname,countrycode,currenttimezone,lastbootuptime,manufacturer,name,numberofusers,ostype,serialnumber,version /format:list
```



