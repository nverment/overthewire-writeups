### level 0 -> level 1
bandit0 doesn't have that much to it. being the introductory level, the only thing needed is to ssh in and use the `cat` command to read the contents of the `readme` file, situated in the home directory.

![alt text](https://raw.githubusercontent.com/nverment/overthewire-writeups/main/images/image-1.png)

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

![alt text](https://raw.githubusercontent.com/nverment/overthewire-writeups/main/images/image-7.png)

naturally, i thought i'd reuse the previous solution and i was successful - being presented with the password for the next level.

![alt text](https://raw.githubusercontent.com/nverment/overthewire-writeups/main/images/image-8.png)

i should also mention that there is another solution that i know of - probably more that i don't. using `cat -- <filename>` instructs cat to ignore whatever comes after `--` as a flag, instead taking it as the filename. running that command returns the same result as the above solution.
### level 3 -> level 4 
bandit 3's home directory contains a directory called `inhere`. following its suggestion, we go 'in here' only to find it seemingly empty. or at least until we use the `ls -la` command. we can, then, see a file that claims to be hiding from us.

![alt text](https://raw.githubusercontent.com/nverment/overthewire-writeups/main/images/image-9.png)

the `.` prefix, for those who don't know, is used in hidden files in linux systems. there are ways to view them from terminal (like the command i just used) or by enabling some 'view hidden files' setting in your preferred file manager. but no one hides from me. so, using `cat ...Hiding-From-You` we can successfully view the contents of the file and move onto the next level.
### level 4 -> level 5
in level 4, we can observe a directory of the same name as before, 'inhere'. we `cd` in, and are greeted with multiple files with names like `-file<number>`.  we are told the following:

The password for the next level is stored in the only human-readable file in the **inhere** directory. Tip: if your terminal is messed up, try the “reset” command.

i tend to be an 'easy solution is best' kinda person. so i just ran `cat -- \-file*`, which would display the contents of all the files in the directory. in the output, between the weird characters, there is a familiar-looking string. 

![alt text](https://raw.githubusercontent.com/nverment/overthewire-writeups/main/images/image-10.png)

since there is a letter - character in the non-human readable files, and there is no delimiter in between each file's contents, we have no way of knowing exactly where the password begins or ends. in my case, i confirmed using `cat` on -file07 (a guess that proved to be correct). 

**note** i am sure the right way to do this is to use some form of the `file -readable` command. i didn't bother to do that though, so check yourself :)
### level 6 -> level 7 
ssh'ing into level 6, we see the same 'inhere' directory, this time with multiple 'maybehere' numbered directories inside it.

![alt text](https://raw.githubusercontent.com/nverment/overthewire-writeups/main/images/image-11.png)

being told that we need a file that is readable, non exacutable and has a size of 1033 bytes, my first tought was to run `ls -la maybehere* | grep 1033`. although this did return a file, we can't see the directory it's in, and manually looking into each file with that name is not too time efficient.

the next thought that came to mind is to stop trying to make everything 'easier' for me and just do it the proper way:
`find . ! -executable -size 1033c`
the `find` command finds (surprise) files with the specifications we give it - in this case, we look for a non-executable (notice the space between ! and the flag!!) as well a size of 1033 bytes, which we can then `cat` to get the next password.

### level 6 -> level 7
a bit of a change of pace in level 6, as bandit tells us the file is stored "somewhere on the server". ominous. truly, running `ls` in the home directory returns us nothing. so what other option but to look "somewhere in the server". 

we can run `ls -la /` to view all the directories, but still seemingly no files. we have no other option but to use `find` again - this time we are looking for a file 33 bytes in size, owned by user bandit7 and group bandit6. we can put this together: 
`bandit6@bandit:~$ find / -user bandit7 -size 33c -group bandit6 
but the result looks something like this:

![alt text](https://raw.githubusercontent.com/nverment/overthewire-writeups/main/images/image-12.png)

so, after scrolling to see if i can find the one file we were able to read (to no avail), i found out there is a way to redirect errors and warnings so that they don't show up on the screen. 
`bandit6@bandit:~$ find / -user bandit7 -size 33c -group bandit6 2>/dev/null`

`/dev/null` is always empty. the data sent there is always discarded, so it serves as a great place to redirect unwanted output. with that addition, we can now see the file that is returned, once again using `cat` to read its contents.
### level 7 -> level 8 
there is honestly not a lot i can say about this level - we are told there is a password in the (very lengthy) data.txt file, next to the word 'millionth'. a simple `grep` will do the trick.

![alt text](https://raw.githubusercontent.com/nverment/overthewire-writeups/main/images/image-13.png)

### level 8 -> level 9
for level 8, the goal is described as follows:

The password for the next level is stored in the file **data.txt** and is the only line of text that occurs only once.

so, using my analytical skills, i thought i'd get out easy again and ran `sort data.txt` which would return the file, but sorted, hoping it would be easy to spot the one line that occurs only once.

![alt text](https://raw.githubusercontent.com/nverment/overthewire-writeups/main/images/image-14.png)

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

the only thing i did is copied the string `Gur cnffjbeq vf 7k16JArUVv5LxVuJfsSVdbbtaHGlw9D4` and pasted it in dcode (https://www.dcode.fr/caesar-cipher). the site bruteforces by default, showing all possible rotations, which can and will be useful in the future (in krypton especially). in this case, we can just copy the value next to +13 or set the shift number to 13 manually.

### level 12 -> level 13
level 12 was probably the most tedious out of all for me at least. overthewire tells us that there is a data.txt file that has been compressed multiple time. my first thought was to learn what a hexdump is.

Hexdump is a utility that displays the contents of binary files in hexadecimal, decimal, octal, or ASCII. It’s a utility for inspection and can be used for data recovery, reverse engineering, and programming.
(https://opensource.com/article/19/8/dig-binary-files-hexdump)

following the challenge's hints, it would be a good idea to copy the file into a location where we can mess with it, given that we don't have permission to do that in the home directory. so, we will first run `cp data.txt /tmp/tempeh/' (or whatever your directory is called. i am personally running out of 'temp' jokes to make). now we have a copy of data.txt that we can experiment on.

let's see the theory we looked up in practice. we can see that the xxd command makes a hexdump or does the reverse by looking at its man page. running the command on the data.txt file outputs the following.

```shell
bandit12@bandit:~$ xxd data.txt 
00000000: 3030 3030 3030 3030 3a20 3166 3862 2030  00000000: 1f8b 0
00000010: 3830 3820 3833 6339 2038 3736 3820 3032  808 83c9 8768 02
00000020: 3033 2036 3436 3120 3734 3631 2033 3232  03 6461 7461 322
00000030: 6520 202e 2e2e 2e2e 2e2e 682e 2e64 6174  e  .......h..dat
00000040: 6132 2e0a 3030 3030 3030 3130 3a20 3632  a2..00000010: 62
00000050: 3639 2036 6530 3020 3031 3365 2030 3263  69 6e00 013e 02c
00000060: 3120 6664 3432 2035 6136 3820 3339 3331  1 fd42 5a68 3931
```

alright, looks like there is no readable text there. first, we can follow the hint and revert the hexdump of the file using `xxd -r data.txt newdata` - we save the output as newdata and take a look at it:

```shell
bandit12@bandit:/tmp/tempeh$ cat newdata 
�ɇhdata2.bin>��BZh91AY&SY�=�����������������������?���w��߿߰1h @�A���@� ��CMhѠhA��:z�0�"�`��E�@Kk��O~�y�lTe���Y��$i�D��!��C���	j���H5��H��!��%N�o�D:T��rJA�z´`
X�=�� �~�8�3T�N�Y�t]qKdh��.�+'�ih*���֛�T.���W�'G>\��l�d��!����
                                                             �wJ""9�
��%����8-��0L��r�I�jCxB^Ӻ�p��
```
looks promising. let's look at what kind of file it is with `file newdata`:
```shell
bandit12@bandit:/tmp/tempeh$ file newdata 
newdata: gzip compressed data, was "data2.bin", last modified: Mon Jul 28 19:03:31 2025, max compression, from Unix, original size modulo 2^32 574
```

interesting - we are getting somewhere. since the file is compressed with gzip, we can also use the same command to reverse the compression. **however**, gzip only works on files with the appropriate suffix - so we first need to rename the file to `newdata.gz`. after running `gzip -d` on the file, we get the file `newdata`. `file` reveals to us that this has been compressed with bzip2, having block size = 900k. 

`bzip2 -d -9 newdata` will do the trick - we use the decompress flag and set the block size to 900k (help page says `-1 .. -9 set block size to 100k .. 900k`) and get the file `newdata.out`.

safe to say the new file is still not readable. yet another gzip compressed file, and by following the same method as above, we get the file newdata which is **also** compressed, this time using tar. `tar -xf newdata` decompresses the file into `data5.bin`. from then on i will provide a code snipped with the steps i followed because honestly it is repetitive and tiring.

```shell
bandit12@bandit:/tmp/tempeh$ file data5.bin 
data5.bin: POSIX tar archive (GNU)
bandit12@bandit:/tmp/tempeh$ tar -xf data5.bin
bandit12@bandit:/tmp/tempeh$ ls
data5.bin  data6.bin  data.txt  newdata
bandit12@bandit:/tmp/tempeh$ file data6.bin 
data6.bin: bzip2 compressed data, block size = 900k
bandit12@bandit:/tmp/tempeh$ bzip2 -d -9 data6.bin 
bzip2: Can\'t guess original name for data6.bin -- using data6.bin.out
bandit12@bandit:/tmp/tempeh$ ls
data5.bin  data6.bin.out  data.txt  newdata
bandit12@bandit:/tmp/tempeh$ file data6.bin.out 
data6.bin.out: POSIX tar archive (GNU)
bandit12@bandit:/tmp/tempeh$ tar -xf data6.bin.out
bandit12@bandit:/tmp/tempeh$ ls
data5.bin  data6.bin.out  data8.bin  data.txt  newdata
bandit12@bandit:/tmp/tempeh$ file data8.bin 
data8.bin: gzip compressed data, was "data9.bin", last modified: Mon Jul 28 19:03:31 2025, max compression, from Unix, original size modulo 2^32 49
bandit12@bandit:/tmp/tempeh$ mv data8.bin data8.gz
bandit12@bandit:/tmp/tempeh$ gzip -d data8.gz 
bandit12@bandit:/tmp/tempeh$ ls
data5.bin  data6.bin.out  data8  data.txt  newdata
bandit12@bandit:/tmp/tempeh$ cat data8
The password is ...
```

oof. there we have it. when i say this genuinely stressed me out because i thought each decompress or unzip would be the last one. thankfully, we got the password and we are ready to move onto the next one.
