# Lab2 - Hexa editor, Readelf, start using gdb

## date
```
date --date=”4 hours ago”
date --date=”2 years ago” +%Y%m%d
```

## md5sum 
* Checksum: “a digit representing the sum of the correct digits in a piece of stored or transmitted digital data, against which later comparisons can be made to detect errors in the data.”
```
[02/10/22]seed@VM:Byeongchan$ md5sum date
782bfc4c88e6fa1ba4fc29bc980bbfe5  date
```

## vimdiff /bin/date my_date
```
vimdiff /bin/date my_date
```

## Objdump
* An executable program on Unix-like system. We can use it as a disassembler to view an executable in assembly form.

```
objdump -d {loc}/date | less
objdump -xtrds {loc}/date | less


[-d|--disassemble[=symbol]]
[-D|--disassemble-all]
[-x|--all-headers]
[-t|--syms]
[-r|--reloc]
[-d|--disassemble[=symbol]]
[-s|--full-contents]
```


## hexa editor
* ghex or bless

## readelf
* What is readelf? readelf is a program for displaying various information about object files on Unix-like systems such as objdump. 

```
readelf -l {loc}/date : it shows program headers
readelf -S {loc}/date : it shows section headers
readelf -W -s {loc}/date : it shows symbols
readelf -x 16 {loc}/date : it shows in hexadecimal
```
