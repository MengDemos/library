##String contains another string
```shell
test "${string#*$word}" != "$string" && echo "$word found in $string"
```
