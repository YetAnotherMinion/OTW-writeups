# Intro

useful information:
- use mktemp -d to create a random, user writeable working directory because
 each user`s home directory is not writeable
- read access in /tmp/ and /proc/ is disabled (what about write access?)
- ALSR has been turned off, so that points to us having to write some shell
code in later levels
- compiled with
  - m32: 32bit architecture
  - fno-stack-protector: Not exactly how this works, but I think stack protector
    puts canaries between frames.
  - Wl, -z, norelo: No idea what these flags do at all

- Useful tools have been given to us:
  - peda (https://github.com/longld/peda.git) in /usr/local/peda/
    - No idea what this one does.
  - gdbinit (https://github.com/gdbinit/Gdbinit) in /usr/local/gdbinit/
    - Hopefully some useful extension to gdb, this tool makes it look like
      mucking about with the memory of running processes might be necessary
  - pwntools (https://github.com/Gallopsled/pwntools) in /usr/src/pwntools/
    - No idea what these are or how to use.
  - radare2 (http://www.radare.org/) should be in $PATH
    - I know this is the open source version of IDA, but I have no clue how to
      use it. Time to learn!

# Jumping in
$ ssh leviathan0@leviathan.labs.overthewire.org
$ password = leviathan0
Using our trusty ls -al we discover a hidden directory in the home directory of
the user leviathan0. This directory is named `.backup`, and belongs to the
leviathan1 user. Our `leviathan0` user is the group allowed to acces this directory.
Descending into `.backup`, we find a file named `bookmarks.html`, and a directory
named `wget123`. The `wget123` directory belongs to `leviathan1`, and its group
belongs to `leviatan1` as well. Its group setgid bit is set as well as its
execute bit, making the group permissions look like `rws`. SetGID means when
another user creates a file or directory under the setgid directory, the
new file will have its group set as the group of the directory owner, instead
of the group of the user who creates it. This means if we create files within
the wget directory then by default `leviathan1` will be able to see them.

Catting the `bookmarks.html` file, I notice that there is an alphabetically sorted
list of URLs with some metadata about those URLs. First I get an idea of how many
of those URLs are in that file using `wc -l bookmarks.html`. Yikes, there are 1399
lines in that file! I definently do not want to anaylze this file by hand. Given
we are interested in getting the password for leviathan1, the first thing
we can do is `grep` through the file for any mention of leviathan.
`grep leviathan bookmarks.html`
It matches on bookmark entry that contains a URL to leviathan.labs.overthewire.org/passwordus.html. The title text of that url gives the password for `leviathan1` is
`rioGegei8m`. Interestingly the ADD_DATE for that bookmark is 1155384634, or
12 August 2006 12:10:34pm (UTC)
