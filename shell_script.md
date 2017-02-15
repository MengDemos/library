##Find directory that has the specific file
```sh
find . -type f -name '*f*' | sed -r 's|/[^/]+$||' |sort |uniq
```
The above finds all files below the current directory (.) that are regular files (-type f) and have f somewhere in their name (-name '*f*'). Next, sed removes the file name, leaving just the directory name. Then, the list of directories is sorted (sort) and duplicates removed (uniq).

The sed command consists of a single substitute. It looks for matches to the regular expression /[^/]+$ and replaces anything matching that with nothing. The dollar sign means the end of the line.  [^/]+' means one or more characters that are not slashes. Thus, /[^/]+$ means all characters from the final slash to the end of the line. In other words, this matches the file name at the end of the full path. Thus, the sed command removes the file name, leaving unchanged the name of directory that the file was in.

**Simplifications**

Many modern sort commands support a -u flag which makes uniq unnecessary:
```sh
find . -type f -name '*f*' | sed -r 's|/[^/]+$||' |sort -u
```
Also, if your find command supports it, it is possible to have find print the directory names directly. This avoids the need for sed:
```sh
find . -type f -name '*f*' -printf '%h\n' | sort -u
```
**More robust version (Requires GNU tools)**

The above versions will be confused by file names that include newlines. A more robust solution is to do the sorting on nul-terminated strings:
```sh
find . -type f -name '*f*' -printf '%h\0' | sort -zu | sed -z 's/$/\n/'
```
##Search all the shell file in the directory and run command
```sh
#note: run shellcheck on all the shell scripts files in current directory. 'grep -Rin' return matching result with line number, e.g. 'path/file:1:#!/bin/bash'

grep -Rin --exclude-dir=.git '^#!.*bash\|^#!.*sh\|^#!.*ksh\|^#!.*dash' ./ \
    | grep ':1:' \
    | awk -F ':' '{print $1}' \
    | xargs $COMMAND --format=checkstyle | sed "s|name='./|name='$(pwd)/|"  || true

```
