
# GDB Cheatsheet

# Preserves Debug Symbols

```
gcc  -ggdb  filename.c
``` 

# Loading Binary into gdb

```
gdb binary_file
```

## or

```
(gdb) file binary_file
```


# List Source Code

```
(gdb) list
```
## or list from particular line or func

```
list 10  
```

# Running Binary in GDB

```
(gdb) run 
```

## with arguments

```
(gdb) run arg1 arg2 arg3
```

# Help 

```
(gdb) help
```



# Extracting Symbol Table

```
objcopy --only-keep-debug  binary_with_debugsymbols  debug_symbols
```


# Removing Symbol Table

```
strip --strip-debug binary_file
```
## and to strip everything

```
strip --strip-debug --strip-unneeded binary_file 
```

# Reading Symbol Table 

## inside gdb

```
(gdb) symbol-file debug_symbols
```

## from commandline

```
objcopy --add-gnu-debuglink=debug_symbols
```

# Change syntax

## setting to intel
```
(gdb) set disassembly-flavor intel
```
## setting to at&t

```
(gdb) set disassembly-flavor att
```


# List Symbol Table from Binary

```
nm binary_file
```

# Tracing System and Library Calls

```
strace binary_file
strace -e read,write binary_file

ltrace binary_file
```

# Set Breakpoint

```
(gdb) break line_number
(gdb) break function_name
(gdb) break *address
```

# Delete Breakpoint

```
(gdb) delete breakpoint_number
```

# Getting info

```
(gdb) info functions
(gdb) info variables
(gdb) info breakpoints
(gdb) info registers
(gdb) info scope function_name  #lists variables in that function
```

# Print values 

```
(gdb) print a
(gdb) print /x $eax
```

# Examine 

## syntax 
```
(gdb) x/format address
```

## more detailly

```
(gdb) x/number_of,representation,size  address
```

## display 5 bytes in hexadecimal

```
(gdb) x/5xb address
```

## display 1 word in floating point

```
(gdb) x/1fw address
```


## display 2 bytes in characters of variable a

```
(gdb) x/2cb &a
```

## display 10 instructions after address

```
(gdb) x/10i address
```

## display the string 

```
(gdb) x/s argv[0]
```

# Disassemble functions

```
(gdb) disassemble function_name
```

# Step through the program


## step through program but dont go inside functions

```
(gdb) step 
```

## step through each instruction , go inside function

```
(gdb) stepi
```

## step 5 instructions exactly

```
(gdb) step 5
```

## step at machine instructions level

```
(gdb) next 
```

## step through functions at machine instructions level

```
(gdb) nexti 
```

# Setting values

## set variable or register 

```
(gdb) set $eip = address
```

```
(gdb) set $eax = value
```

## setting values in address

```
(gdb) set {data_type} address = value
(gdb) set {int} 0x12345678 = 9
```

# Call a function

```
(gdb) call function_name
```

# Continue the Program 

```

(gdb) continue
```

# GDB Hooks

## hooks are user defined commands , when there is command 'temp' , whenever the command is executed , if there is any hook named 'hook-temp' then commands in that hook will gets executed 

## let's say if my program stops then some commands should execute 

## show registers automatically when program stops

```
(gdb) define hook-stop
> info registers
> x/8cb $esp
> end
```



# Continue only the Function

```
(gdb) fin
```


# Quit the gdb

```
(gdb) quit 
```

# Installing PEDA Plugin

## peda plugin is like automated hook that displays registers , stack and instructions

```
git clone https://github.com/longld/peda.git ~/peda
echo "source ~/peda/peda.py" >> ~/.gdbinit
```

## for other peda commands check out its github 

[check here](https://github.com/longld/peda)


