# Hunting Licence write up by Om Honrao

## Challenge Description
```
STOP! Adventurer, have you got an up to date relic hunting license? If you don't, you'll need to take the exam again before you'll be allowed passage into the spacelanes!
```

## Attachments
```bash
- rev_hunting_licence.zip
    ./licence
```

## Solution
Reversing my best category. I started with some basic scans like `file`, `strings` and `ltrace`. 

```bash
$ file license 
license: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=5be88c3ed329c1570ab807b55c1875d429a581a7, for GNU/Linux 3.2.0, not stripped
```

Strings did not give anything good i was about to open the code in Ghidra to analyse but then i saw `ltrace` which gave me some interesting output! It outputs the correct passwords needed to input!

```bash
$ ltrace ./license 
puts("So, you want to be a relic hunte"...So, you want to be a relic hunter?
)                                                                                  = 35
puts("First, you're going to need your"...First, you're going to need your license, and for that you need to pass the exam.
)                                                                                  = 82
puts("It's short, but it's not for the"...It's short, but it's not for the faint of heart. Are you up to the challenge?! (y/n)
)                                                                                  = 85
getchar(0x7fb50b2f87e0, 0x1d632a0, 0, 0x7fb50b218077y
)                                                                        = 121
readline("Okay, first, a warmup - what's t"...Okay, first, a warmup - what's the first password? This one's not even hidden: PasswordNumeroUno
)                                                                              = "PasswordNumeroUno"
strcmp("PasswordNumeroUno", "PasswordNumeroUno")                                                                             = 0
free(0x1d7c3c0)                                                                                                              = <void>
readline("Getting harder - what's the seco"...Getting harder - what's the second password? P4ssw0rdTw0
)                                                                              = "P4ssw0rdTw0"
strcmp("P4ssw0rdTw0", "P4ssw0rdTw0")                                                                                         = 0
free(0x1d7c3c0)                                                                                                              = <void>
readline("Your final test - give me the th"...Your final test - give me the third, and most protected, password: ThirdAndFinal!!!
)                                                                              = "ThirdAndFinal!!!"
strcmp("ThirdAndFinal!!!", "ThirdAndFinal!!!")                                                                               = 0
free(0x1d7c3c0)                                                                                                              = <void>
puts("Well done hunter - consider your"...Well done hunter - consider yourself certified!
)                                                                                  = 48
+++ exited (status 0) +++
```
Great! we have everything we need to pass the exam at nc Docker instance. Now we just need to input the correct passwords and we are done with some extra things! I don't think i should explain the rest of the challenge. 

```bash
What is the file format of the executable?
> elf
[+] Correct!

What is the CPU architecture of the executable?
> x86-64
[+] Correct!

What library is used to read lines for user answers? (`ldd` may help)
> libreadline.so.8
[+] Correct!

What is the address of the `main` function?
> 0x401172
[+] Correct!

How many calls to `puts` are there in `main`? (using a decompiler may help)
> 5
[+] Correct!

What is the first password?
> PasswordNumeroUno
[+] Correct!

What is the reversed form of the second password?
> 0wTdr0wss4P
[+] Correct!

What is the real second password?
> P4ssw0rdTw0
[+] Correct!

What is the XOR key used to encode the third password?
> 17
[-] Wrong Answer.
What is the XOR key used to encode the third password?
> 19
[+] Correct!

What is the third password?
> ThirdAndFinal!!!
[+] Correct!

[+] Here is the flag: `HTB{l1c3ns3_4cquir3d-hunt1ng_t1m3!}`
```

There we go we get the flag!

## Flag
```
HTB{l1c3ns3_4cquir3d-hunt1ng_t1m3!}
``` 

## Contact info

This challenge was solved by Om Honrao (Discord ID: Inv1s1bl3#7047). If you have any questions or feedback, feel free to reach out to me.