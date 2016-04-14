# Joining
Trusty `ls -la` reveals a binary name `printfile` in our home directory. The setguid bit
is set for the owning user, which means that this process will run as `levaithan3`.
To pass this level it looks like I need to print out the contents of 
`/etc/leviathan_pass/leviathan3` file which is only readable by the user `levaithan3`.
The binary is just shy of 8kb, so it will probably be pretty simple to reverse. 
Running `strings` on the binary reveals a couple of
interesting strings right away. The first is `"/bin/cat %s"`. I see a usage string as
well, and a failure message. I judge that it is probably safe to run this binary, so
I run it without any arguments. It exits after printing the usage string I saw earlier.
Again I failed to check the exit code the binary, which is becoming somewhat of a theme.
I try supplying the filename as the first argument

I then disassemble the binary using `radare2 printfile`. I then enter visual mode by
entering `V` at the prompt, and then entering `d` to run the disassembler.

It appears that if the `access(const char* pathname, int mode)` call succeeds, i.e.
the file exists, then the user entered filename is passed directly to a format
string unfiltered for special characters. That format string is evaluated directly by
`/bin/sh`.

The password is `Ahdiemoo1j`.
