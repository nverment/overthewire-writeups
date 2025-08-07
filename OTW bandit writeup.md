### level 0 -> level 1
bandit0 doesn't have that much to it. being the introductory level, the only thing needed is to ssh in and use the `cat` command to read the contents of the `readme` file, situated in the home directory.

![[image-1.png]]

for the record, and for future levels, i prefer to run these commands upon logging in:

```shell
export TERM=linux
force_color_prompt=yes
source ~/.bashrc
```

the first one allows for a 'better shell' for me since i am using `kitty` terminal, so after that i can use the arrow keys to navigate command history or clear the terminal for example. if you have regular `xterm` i believe you shouldn't need it.

the second and third make the terminal have at least some color because by default, when ssh'ing you will have a pretty much two-color theme. `force_color_prompt` enables your terminal's colors, and reloading the shell activates them.
### level 1 -> level 2
in this level, upon successfully ssh'ing into bandit1, we can `ls -la` (the `-l` flag makes it so that we can also see the permisions and owners, and the `-a` allows us to see all files, including hidden ones). that presents us with a singular flie named `-` - pretty non-standard as far as file names go.

so how can we bypass this?
of course, i tried these at first - the problem with that is of course the fact that like all commands, when cat sees a `-` it expects not a filename, but a parameter / flag.
```shell
cat -
cat '-'
cat \-
```

so, my mind went to commands like `echo 'string' > file`. the `>` operator redirects the output of `echo` to the file we specify. so why wouldn't i be able to do something similar, but in the opposite direction, using cat?

`cat < -` does, indeed, work, and returns us the password to the next level.
### level 2 -> level 3
level 2 is very similar to level 1, since we can immediately see a file called '--spaces in this filename--' under the home directory.
![[image-7.png]]

naturally, i thought i'd reuse the previous solution and i was successful - being presented with the password for the next level.
![[image-8.png]]

i should also mention that there is another solution that i know of - probably more that i don't. using `cat -- <filename>` instructs cat to ignore whatever comes after `--` as a flag, instead taking it as the filename. running that command returns the same result as the above solution.
### level 3 -> level 4 
bandit 3's home directory contains a directory called `inhere`. following its suggestion, we go 'in here' only to find it seemingly empty. or at least until we use the `ls -la` command. we can, then, see a file that claims to be hiding from us.
![[image-9.png]]

the `.` prefix, for those who don't know, is used in hidden files in linux systems. there are ways to view them from terminal (like the command i just used) or by enabling some 'view hidden files' setting in your preferred file manager. but no one hides from me. so, using `cat ...Hiding-From-You` we can successfully view the contents of the file and move onto the next level.
### level 4 -> level 5
in level 4, we can observe a directory of the same name as before, 'inhere'. we `cd` in, and are greeted with multiple files with names like `-file<number>`.  we are told the following:

The password for the next level is stored in the only human-readable file in the **inhere** directory. Tip: if your terminal is messed up, try the “reset” command.

i tend to be an 'easy solution is best' kinda person. so i just ran `cat -- \-file*`, which would display the contents of all the files in the directory. in the output, between the weird characters, there is a familiar-looking string. 
![[image-10.png]]

since there is a letter - character in the non-human readable files, and there is no delimiter in between each file's contents, we have no way of knowing exactly where the password begins or ends. in my case, i confirmed using `cat` on -file07 (a guess that proved to be correct). 

**note** i am sure the right way to do this is to use some form of the `file -readable` command. i didn't bother to do that though, so check yourself :)
### level 6 -> level 7 
ssh'ing into level 6, we see the same 'inhere' directory, this time with multiple 'maybehere' numbered directories inside it.
![[image-11.png]]

being told that we need a file that is readable, non exacutable and has a size of 1033 bytes, my first tought was to run `ls -la maybehere* | grep 1033`. although this did return a file, we can't see the directory it's in, and manually looking into each file with that name is not too time efficient.

the next thought that came to mind is to stop trying to make everything 'easier' for me and just do it the proper way:
`find . ! -executable -size 1033c`
the `find` command finds (surprise) files with the specifications we give it - in this case, we look for a non-executable (notice the space between ! and the flag!!) as well a size of 1033 bytes, which we can then `cat` to get the next password.

### level 6 -> level 7
a bit of a change of pace in level 6, as bandit tells us the file is stored "somewhere on the server". ominous. truly, running `ls` in the home directory returns us nothing. so what other option but to look "somewhere in the server". 

we can run `ls -la /` to view all the directories, but still seemingly no files. we have no other option but to use `find` again - this time we are looking for a file 33 bytes in size, owned by user bandit7 and group bandit6. we can put this together: 
`bandit6@bandit:~$ find / -user bandit7 -size 33c -group bandit6 
but the result looks something like this:
![[image-12.png]]

so, after scrolling to see if i can find the one file we were able to read (to no avail), i found out there is a way to redirect errors and warnings so that they don't show up on the screen. 
`bandit6@bandit:~$ find / -user bandit7 -size 33c -group bandit6 2>/dev/null`

`/dev/null` is always empty. the data sent there is always discarded, so it serves as a great place to redirect unwanted output. with that addition, we can now see the file that is returned, once again using `cat` to read its contents.
### level 7 -> level 8 
there is honestly not a lot i can say about this level - we are told there is a password in the (very lengthy) data.txt file, next to the word 'millionth'. a simple `grep` will do the trick.

![[image-13.png]]
### level 8 -> level 9
for level 8, the goal is described as follows:

The password for the next level is stored in the file **data.txt** and is the only line of text that occurs only once.

so, using my analytical skills, i thought i'd get out easy again and ran `sort data.txt` which would return the file, but sorted, hoping it would be easy to spot the one line that occurs only once.
![[image-14.png]]

yeah.. the output is 1001 lines. i am **not** reading all that. after some more consideration, i looked at what the `uniq` command does. turns out, running it with a `-u` flag, it returns the unique (surprise) lines in a file. the end result looks like this:
`bandit8@bandit:~$ sort data.txt | uniq -u`

and we are ready to move onto level 9.
### level 9 -> level 10 
from now on, i will start putting each level's description first, so i don't have to paraphrase it. for level 9 it is as follows:

The password for the next level is stored in the file **data.txt** in one of the few human-readable strings, preceded by several ‘=’ characters.

since i don't know any commands that can help with this, i needed to look into what each of the recommended ones does. the answer is `string` - it pretty much shows us all the human readable characters in a file. so, we can simply run:
`strings data.txt` 

and we are off to the next one.
### level 10 -> level 11 
The password for the next level is stored in the file **data.txt**, which contains base64 encoded data.

looks like this is easy enough as well. we can see a command called base64 - same name as the supposed encoding the file `data.txt` is in. the syntax is pretty straightforward - the flag `-d` decodes the encoded text / file. so we run:
`base64 -d data.txt`

and we get the password to the next level.

### level 11 -> level 12
The password for the next level is stored in the file data.txt, where all lowercase (a-z) and uppercase (A-Z) letters have been rotated by 13 positions.

when i was a weird little child, i was very interesting in codes. i think i just didn't want anyone to know what i was writing. cesar cipher was one of the first ones i 'discovered' myself, only to later find that it had already been invented some thousands of years ago. alas, seeing the string inside data.txt rang a bell.

looking at the string `7x16WNeHIi5YkIhWsfFIqoognUTyj9Q4`, even knowing that it is rotated by 13 positions, i just copied it and pasted it in my beloved site dcode (https://www.dcode.fr/caesar-cipher). the site bruteforces by default, showing all possible rotations, which can and will be useful in the future (in krypton especially). in this case, we can just copy the value next to +13 or set the shift number to 13 manually.