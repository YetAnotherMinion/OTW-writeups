# Jumping in
With our trusty `ls -al` we find an executable binary named `check` in our 
home directory. I see that this binary is owned by `leviathan2`, which is
the user I want to login as next. The binary will execute with the
permissions of `leviathan2` which will be perfect for reading the password
for `leviathan2` I see is present in `/etc/leviathan_pass/`. I could run
the `check` program to see what it does, but I have developed a healthy
suspicsion of running arbitrary binaries. True its not my system, and it
is unlikly that the binary is a gameover bomb given that this is level 2.
My first step is to `cat` the program since I saw from `ls -al` that is
is a very small program. The program is very short, and right away I can
see the magic string "ELF". Next I run `strings` against the binary.
There are several interesting strings I see. I see the string "/bin/sh"
inside, maybe this challenge has us write a simple stack smashing shell
code. If there is a matching call to `system` inside that is pointing
to some garbage string then this challenge is likely a classic stack
smashing exploit. I also see the odd strings "sex", "lovegod" and "etsecr".
Now feeling more secure about running the program, I run `check` and find
it asks for a password (which explains the "password" string). I then hit
enter and the program prints "Wrong password\nGood Bye" and exits. 
I neglect to check the exit code of the process using `$?` but it
ends up not mattering. 

I then choose to disassemble the binary using `radare2` because I am
not familiar with this tool, and now is as good time as any to start learning.
I run `$ radare2 check` and get a prompt telling me "Welcome to IDA 10.0" and
then a hex address prompt. I then use `pd` to print the disassembly. 

## Relevant portion of disassembly
---------------
```asm
    0x0804852d    55             push ebp                                       
    0x0804852e    89e5           mov ebp, esp                                 
    0x08048530    83e4f0         and esp, 0xfffffff0                          
    0x08048533    83ec30         sub esp, 0x30                                
    0x08048536    65a114000000   mov eax, dword gs:[0x14]      ; [0x14:4]=1   
    0x0804853c    8944242c       mov dword [esp + 0x2c], eax                  
    0x08048540    31c0           xor eax, eax                                 
    0x08048542    c74424187365.  mov dword [esp + 0x18], 0x786573 ; [0x786573:
    0x0804854a    c74424257365.  mov dword [esp + 0x25], 0x72636573 ; [0x72636
    0x08048552    66c744242965.  mov word [esp + 0x29], 0x7465 ; [0x7465:2]=0x
    0x08048559    c644242b00     mov byte [esp + 0x2b], 0                     
    0x0804855e    c744241c676f.  mov dword [esp + 0x1c], 0x646f67 ; [0x646f67:
    0x08048566    c74424206c6f.  mov dword [esp + 0x20], 0x65766f6c ; [0x65766
    0x0804856e    c644242400     mov byte [esp + 0x24], 0                     
    0x08048573    c70424808604.  mov dword [esp], str.password: ; [0x8048680:4
    0x0804857a    e841feffff     call sym.imp.printf           ;[2]           
    0x0804857f    e84cfeffff     call sym.imp.getchar          ;[3]           
    0x08048584    88442414       mov byte [esp + 0x14], al                    
    0x08048588    e843feffff     call sym.imp.getchar          ;[3]           
    0x0804858d    88442415       mov byte [esp + 0x15], al                    
    0x08048591    e83afeffff     call sym.imp.getchar          ;[3]           
    0x08048596    88442416       mov byte [esp + 0x16], al                    
    0x0804859a    c644241700     mov byte [esp + 0x17], 0                     
    0x0804859f    8d442418       lea eax, [esp + 0x18]         ; 0x18  ; "0...
    0x080485a3    89442404       mov dword [esp + 4], eax                     
    0x080485a7    8d442414       lea eax, [esp + 0x14]         ; 0x14         
    0x080485ab    890424         mov dword [esp], eax                         
    0x080485ae    e8fdfdffff     call sym.imp.strcmp           ;[4]           
    0x080485b3    85c0           test eax, eax                                
,=< 0x080485b5    750e           jne 0x80485c5                 ;[5]           
|   0x080485b7    c704248b8604.  mov dword [esp], str._bin_sh  ; [0x804868b:4]
|   0x080485be    e83dfeffff     call sym.imp.system           ;[6]           
,==< 0x080485c3    eb0c           jmp 0x80485d1                 ;[7]           
|`-> 0x080485c5    c70424938604.  mov dword [esp], str.Wrong_password__Good_Bye
|    0x080485cc    e81ffeffff     call sym.imp.puts             ;[8]           
`--> 0x080485d1    b800000000     mov eax, 0                                   
    0x080485d6    8b54242c       mov edx, dword [esp + 0x2c]   ; [0x2c:4]=0x28
    0x080485da    653315140000.  xor edx, dword gs:[0x14]                     
,=< 0x080485e1    7405           je 0x80485e8                  ;[9]
```


looking at the assembly, we piece together the inital buffer

```
esp+: 18|19|1a|1b|1c|1d|1e|1f|20|21|22|23|24|25|26|27|28|29|2a|2b

raw:  00|78|65|74|00|64|6f|67|65|76|6f|6c|00|72|63|65|73|74|65|00

text: \0  x  e  s \0  d  o  g  e  v  o  l \0  r  c  e  s  t  e \0
```
we then see where printf is called with the password prompt
and see that right after that getch is called 3 times. The 3 bytes
of input are written into a buffer starting at [esp+14] and then null
 terminated. Then [esp+14] and [esp+18] are pushed onto the stack and
 strcmp is called. Looking at the buffer starting at [esp+18] we
just translated into ASCII, we see that the `check` program password
 is `sex`. Entering this password gives us a shell `/bin/sh` which we
also saw from the disassembly. Once I have a shell I look for the password
in the `/etc/leviathan_pass/` directory where I see a `leviathan2` file. I cat
that `cat /etc/leviathan_pass/leviathan2` and see the password is `ougahZi8Ta`

