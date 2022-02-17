## Buffer overflow with ASLR(random stack aderess)

## Problem
+ Without ASLR, we can find out exact address of the user input.
+ It's quite easy to find out the address and inject malicious code to that spot, and set return address to it.
+ But, with ASLR, it's really hard to get exact address of the stack. Because it changes whenever executing.

## Main idea
+ When calling a function with an user input argument, there should be the input argument in the stack frame. 
+ If we can found the address of the argument in ths stack, and then we can use it to execute out malicious code.

## Steps
1. Find out where the break points are. There are two break points. Before calling strcpy and after calling.
2. Run gdb and set break points.
3. Run with the random argument you can notice.
4. Find the input address from the stack using x/72xw $esp.





objdump -D ans_check6 | less
objdump -D ans_check6 | grep -B3 ret | grep -A1 pop


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
