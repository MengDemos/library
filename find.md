---
title: Find Files and Do\ Stuff\ <br/>with\ GNU\ find
author: Serge Y. Stroobandt
date: Copyright 2014--2017, $(BYNCSA)
description: An extensive collection of real-world GNU find examples, ready to use as a cheat sheet. Includes explanations.
...


0. [Home](../../index/en/index.html)
1. [Information Technology](../../it/en/index.html)
2. [Command Line](../../cli/en/index.html)
3. find


#Introduction
| Do yourself a favour and [have a quick look at `man find`](http://linux.die.net/man/1/find).
| Done? Let us continue then with some examples for fun and profit...


#When not to use find


##Gain speed with `locate`
When looking for a hard to find file or directory that already exists for some while on your drive, do not use `find` but use `locate` instead.
```bash
$ locate filename
```
The reason is **queries with `locate` execute much faster than with `find`.**
Unlike `find`, `locate` uses a previously created database to perform the search. This database is updated periodically from `cron`.
Nevertheless, there are many instances where the use of `find` is required, albeit because `find` offers an `-exec` clause. Numerous examples are given below.


##Unleash the power of `globstar`
Here is a real-life example where I unleashed the power of `globstar`.
The Xubuntu icon would not appear on the XFCE application menu 
after installing Xubuntu on a system which already had a `\home` directory 
made by a preceding XFCE distribution that lost my favour.
I knew the icon file name would start with `xu`, end in `.png` and that it would be located somewhere in the "unique system resources" folder named `\usr`.

Finding this file with `find` would require a rather intricate [regular expression](http://en.wikipedia.org/wiki/Regular_expression). However, things turn out much simpler when you uncomment or add `shopt -s globstar` in `~/.bashrc`.
Now you can do the following:
```bash
$ cd /usr
/usr $ locate **/xu*.png
/usr $ ls **/xu*.png
```
Both commands yielded precisely nine full file paths on my system, among which the desired `/usr/share/pixmaps/xubuntu-logo-menu.png`. Interestingly, the brute-force **`ls` happens to be almost 36% faster** than the database look-up of `locate`:

```bash
/usr $ time locate **/xu*.png
4.2s
/usr $ time ls **/xu*.png
2.7s
```

##List only hidden files\ & directories in\ the\ current\ directory
```bash
ls -d .*
```

As a long list:
```bash
ls -dl .*
```

Stricter, without `.` nor `..`:
```bash
ls -dl .!(|.)
```

##List any text file containing a\ specific\ string
This can be done with a recursive `grep` command.
Notice the uncharacteristic **small case `-r`.**
**Do not add any file specification** like `*.txt`.
Otherwise, `grep -r` will search only in the working directory!
```bash
grep -r 'class="fixed"'
```


#List specific text files containing a\ specific\ string
If you are indifferent to the text file name pattern, use [`grep -r`](#list-any-text-file-containing-aspecificstring) instead.
If not, the following command lists the path and name of text files ending in `.desktop` and containing the text `searchstring`:
```bash
find . -name *.desktop -exec grep -l searchstring {} \;
```
The option `-l` suppresses the normal output of `grep` and prints the file names with matches instead.
Instead, use option `-H` if both the file name and the found string in its context needs to be shown.
```bash
find . -name *.desktop -exec grep -H searchstring {} \;
```


#Find files without descending into\ directories
With `-maxdepth 1`, `find` will not descend into underlying directories.
```bash
find . -maxdepth 1 -type f -name '*.md'
```


#Recursively count files except\ hidden\ ones
```bash
find . -type f -not -path '*/\.*' |wc -l
```


#Find and list directories
The following command will find all directories called `doc`, print their name preceded by an empty line and list their contents in a detailed, human-readable format. This example also shows how **multiple commands can be executed with `find`** by adding an `-exec` for each command.
```bash
$ find . -iname doc -exec echo -e \\n{} \; -exec ls -lh {} \;
```


#Remove many files
As strange as it may sound, `rm *` has its limits as for the number of files that it may delete.
Upon hitting this limit, the `rm` command will complain along the lines of:
```
-bash: /usr/bin: Argument list too long
```
Luckily, in situations like this, the `find` command comes to the rescue.
```bash
$ sudo find . -type f -delete
```


#Copy found files to\ another directory
Here is an example of searching files with `hf` in the file name and copying those to a directory called `hf/`.
Matching file names in both the current and subdirectories will be found.
```bash
$ find . -type f -iname '*hf*' -print -exec cp {} hf/ \;
```



#Change ownership recursively
If not too many files and folders are involved this can be done using `chown -R` recursively:
```bash
$ sudo chown -R serge:family foldername
```

Otherwise, use:
```bash
$ sudo find . -print -exec chown serge:family {} \;
```


#Change directory permissions recursively
From time to time, I am still tempted to make the naive mistake of recursively changing the permissions of entire directory trees with `chmod -R 770 *`. **This is plain wrong of course, as files within the sub-directories will also be made executable.** The end result is a major security issue.

The correct way of tackling this problem is:
```bash
$ sudo find . -type d -print -exec chmod 770 {} \;
```


#Change file permissions recursively
Similar to the previous code snippet, but then for files. Again, one cannot simply do `chmod -R 640 *`, as **this would render directories unbrowsable** because of a failing execution bit.
```bash
$ sudo find . -type f -print -exec chmod 640 {} \;
```


#Prevent others from deleting files in\ your\ folder

##Sticky bit
If on a shared drive, a directory get its *sticky bit* set, it becomes **an [append-only directory](https://en.wikipedia.org/wiki/Sticky_bit#Usage).** Putting it more accurately; it becomes a directory in which files may only be removed or renamed by a user if the user has write permission for the directory *and* the user is also the owner of the file, the owner of the directory, or root.

**Without the directory *sticky bit*, any user with group write and execute permissions for the directory can rename or delete files, even when that user is not the owner of those files.**

**A *sticky bit* on files is of no relevance** to the Linux kernel.

The *sticky bit* is set by adding the octal value `1000` to the usual octal absolute mode number:
```bash
sudo find . -type d -print -exec chmod 1770 {} \;
```

A directory which is **executable by `others`,** and which has the *sticky bit* set, will have the final `x` changed to a lower case `t` when listed by `ls -l`:
```
drwxrwxrwx   â†’   drwxrwxrwt
```
A directory which is **not executable by `others`,** but with the *sticky bit* set, will have the final `-` changed to a capital `T` when listed by `ls -l`:
```
drwxrwx---   â†’   drwxrwx--T
```

##Inherit the group ID
Setting the sticky bit to a directory becomes even more useful **when newly created subdirectories automatically inherent the same group ownership** of their parent directory.
To *set the group ID upon execution* (`setgid` or SGID), add the octal value `3000` to the usual octal absolute mode number:
```bash
sudo find . -type d -print -exec chmod 3770 {} \;
```

A directory which is **executable by `others`,** and which has both the *sticky bit* and the *SGID* set, will have the group `x` changed to a lower case `s` and the final `x` changed to a lower case `t` when listed by `ls -l`:
```
drwxrwxrwx   â†’   drwxrwsrwt
```
A directory which is **not executable by `others`,** but with both the *sticky bit* and the *SGID* set, will have the group `x` changed to a lower case `s` and the final `-` changed to a capital `T` when listed by `ls -l`:
```
drwxrwx---   â†’   drwxrws--T
```


#Find recently modified files recursively
Hidden files are ignored using `-not -path '*/\.*'`.
More files are returned by changing the number at the end of the `|head -n 10` pipe.
```bash
$ find . -type f -not -path '*/\.*' -printf '%TY.%Tm.%Td %THh%TM %Ta %p\n' |sort -nr |head -n 10
2017.01.28 07h27 Sat ./find/en/index.html
2017.01.28 07h27 Sat ./find/en/find.md
2017.01.28 07h00 Sat ./propagation/en/propagation.md
2017.01.27 11h26 Fri ./propagation/images/owf.svg
2017.01.27 11h24 Fri ./propagation/en/index.html
```

#Find recently modified directories recursively
`Permission denied` error messages are [filtered out](https://stackoverflow.com/a/40336333/2192488).
More directories are returned by changing the number at the end of the `|head -n 10` pipe.
```bash
$ { find . -type d -not -path '*/\.*' -printf '%TY.%Tm.%Td %THh%TM %Ta %p\n' 2> >(grep -v 'Permission denied' >&2); } |sort -nr |head -n 10
```

#Find more recent files
Calling `find` with the `-newer now` will return objects that have been more recently modified than the file called `now`.
```bash
$ touch now
$ find . -type f -newer now
```


#Find empty files
```bash
$ find . -type f -name *.bib -size 0
```


#Find and delete empty\ files
```bash
$ find . -type f -name *.bib -size 0 -delete
```


#Find symbolic\ links
```bash
$ find . -type l
```


#Detailed listing of what\ find\ found
Continuing from the previous example:
```bash
$ find . -type l -ls
```
This will print all details about the found symbolic links, including their target.


#Find symbolic links to\ a\ specific\ target
```bash
$ find . -lname link_target
```
Note that `link_target` may contain wildcard characters.


#Find broken symbolic links
```bash
$ find -L . -type l -ls
```
The `-L` option instructs find to follow symbolic links, unless when broken.


#Find\ & replace broken symbolic\ links
```bash
$ find -L . -type l -delete -exec ln -s new_target {} \;
```


#Rename files recursively
Here is a convoluted example which finds all files ending in `_a4.pdf` and renames them to ending in `.a4.pdf`.
Note that using multiple `-exec` additions would not work here. Instead, one resorts to a single `-exec sh -c '...$1...' _ {} \;`.
```bash
$ find . -name "*_a4.pdf" -exec sh -c 'mv "$1" "$(echo "$1" |sed s/_a4.pdf\$/.a4.pdf/)"' _ {} \;
```
Add an `echo` in front of the `mv` to test out the command first:
```bash
$ find . -name "*_a4.pdf" -exec sh -c 'echo mv "$1" "$(echo "$1" |sed s/_a4.pdf\$/.a4.pdf/)"' _ {} \;
```


#Count directories
Finally, a little extra about listing and counting only directories ---and nothing else--- in the current directory.
By now, you might think you would need `find` for that. Surprisingly, `ls` and `wc` will do just fine.
List only directories in the current folder with
```bash
$ ls -dl */
```
Piping through `wc -l` will count the number of lines printed by the preceding list command.
```bash
$ ls -dl */ |wc -l
```


#Find directories with a\ makefile\ link
On rare occasions, all pages on this website need to be rebuilt. 
This happens normally only for structural maintenance purposes; e.g. when the text at the bottom of each page changes.
The following command finds all directories containing a symbolic link called `makefile` and runs the `make` command there.
```bash
$ find . -type l -name makefile -exec sh -c 'make --always-make --directory=$(dirname {})' \;
```


#List the character encoding of text\ files
```bash
$ find . -type f -iname *.txt -exec file -i {} \;
```

#Change the character encoding of text\ files to\ UTFâ€‘8
The character encoding of all matching text files gets detected automatically and all matching text files are converted to `utf-8` encoding:
```bash
$ find . -type f -iname *.txt -exec sh -c 'iconv -f $(file -bi "$1" |sed -e "s/.*[ ]charset=//") -t utf-8 -o converted "$1" && mv converted "$1"' -- {} \;
```
To perform these steps, a sub shell `sh` is used with `-exec`, running a one-liner with the `-c` flag, and passing the filename as the positional argument `"$1"` with `-- {}`. In between, the `utf-8` output file is temporarily named `converted`.

<!--#Edit found files-->
<!--Below command will recursively find all files named `filename` and open them in `gvim` for editing.-->
<!--```bash-->
<!--$ find . -type f -name filename -print -exec gvim {} \;-->
<!--```-->
