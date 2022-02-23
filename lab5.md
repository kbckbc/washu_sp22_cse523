## Buffer overflow attack(turn off ASLR, turn off NX)

## Overview
* So far, in lab3 & lab4, we did a buffer overflow attack using stack executable.
* But it's not the case in real situation. Linux systems usually make it impossible to run programs from stack memory.
* We injected malicious code in the stack from previous practice, but in this practice, we're going to call system function which is in the libc library.
* Most programs, including ans_check7, rely on the C standard library, libc. The return-to-libc method we discussed in the lecture explains how we can pass command line arguments to the system() function in the linked libc library to spawn a new shell, without requiring the ability to execute code on the stack.
* With this senario, we're going to make a payload like below
* PAYLOAD = PADDING, &system(), &exit_path, &cmd_string

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

### Step2 - Find out the address of the system function

### Step3 - Find out the address of the exit function

### Step4 - Find out the address of the '/bin/bash' string in the env variables

### Step5 - Make the payload and execute

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
