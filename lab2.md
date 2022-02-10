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
