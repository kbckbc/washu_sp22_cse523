

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
