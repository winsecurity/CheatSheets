# Windows Registry

Tags : #regedit


Registry is hierarchial database that stores configuration and settings of OS , Users , Hardware, Software , Networking etc

Registry is stored on harddisk called **Hives** 
Whenever we boot our pc , all these Hives are loaded into memory and thus all the settings like wallpaper , appearance , networking etc are automatically loaded

Registry contains **keys** and **values** 
these keys can contain multiple subkeys 

There are main 5 Hives in Registry 

**HKEY_CLASSES_ROOT** (HKCR)
**HKEY_CURRENT_USER** (HKCU)
**HKEY_LOCAL_MACHINE** (HKLM)
**HKEY_USERS** (HKU)
**HKEY_CURRENT_CONFIG** (HKCC)

---

## Hives

Lets see each one of them

## HKEY_CLASSES_ROOT

This Hive contains all core components of OS
like drag-drop , copy operations , shortcuts etc
It also contain information like **Open with**
like when u open a file with a psd extension it automatically opens with photoshop
that information will be stored in this hive

## HKEY_CURRENT_USER

This hive contains keys related to currently loggedin user
like user's wallpaper , colors , cursor and all other settings
It also contains ControlPanel and software installed on User's Desktop

The structure is as follows

![[Pasted image 20210705085515.png]]

HKCU is just link to HKU 
if you open HKEY_USERS there you can see exact structure under your current user's SID

The current user's registry configuration is stored in harddisk at the path

```
C:\Users\username\NTUSER.DAT
```


## HKEY_LOCAL_MACHINE

This hive contains all information about softwares installed  , hardware , bios , security , password hashes etc

The structure is as follows

![[Pasted image 20210705085559.png]]

these SAM , SECURITY , SYSTEM , BCD are stored in 
```
C:\Windows\System32\Config
```

SAM and SECURITY are hidden even from **administrator** user
they are accessible only by **SYSTEM** user

Download psexec from microsoft sysinternals suite
Run cmd as administrator
Now navigate to that psexec folder and execute
```
>psexec.exe -sid cmd.exe
```

a new command prompt will open with SYSTEM privileges
in that type
```
> regedit.exe
```
close any opened regedit

now you will have access to SAM and SECURITY

if we have administrator access we can dump SAM SYSTEM and SECURITY

```
reg save hklm\sam e:\sam
```

## HKEY_USERS

This hive contain all users' information
in this keys contain SID (security identifer)
of users 

![[Pasted image 20210705121710.png]]

first three are inbuilt accounts used by services like IIS , Kerberos etc

remaining lengthy ones are actual users
it doesnot show all users but only active users

to know our own sid
```
whoami /all
```


## HKEY_CURRENT_CONFIG

this acts as shortcut to 
```
Computer\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Hardware Profiles\Current
```

it stores hardware information like printers , etc

--- 

## Data Types

**REG_SZ** : this data type stores common ascii characters

**REG_BINARY** : this data type stores data in the form of 0's and 1's but regedit shows us in hex format for our convenience

**REG_DWORD** : this data type stores data in the form of 4 bytes (2 words) that means 32 bits.
Generally it is used to store boolean values 

**REG_EXPAND_SZ** : this data type stores string like a file path . when parsed by application it parses it as file path 

**REG_MULTI_SZ** : this stores multiple strings as one string . Each string is separated by NULL byte \x00

