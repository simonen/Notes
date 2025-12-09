**GREP** - General Regular Expression Parser
#### Line Anchors

Specify where in a line a particular text is located

^text  - matches lines where a text starts with ***text***
text$ - matches lines ending with *text*

**$ grep ^text** *text_file*

To search directories recursively for keywords in files. Output shows the result and the line number

``` bash
grep -nr "KEYWORD" *
```

```
[root@dbserver19 ~]# grep -nr -i root /etc/ssh/
/etc/ssh/sshd_config:38:#PermitRootLogin yes
/etc/ssh/sshd_config:90:# the setting of "PermitRootLogin without-password".
/etc/ssh/sshd_config:119:#ChrootDirectory none
```

**-i**: case insensitive. Matches upper and lower case
**-v**: result excludes keyword
**-o**: shows the result only, without printing the whole line
#### Escaping Regex

put the regex in quotes 

**$ grep '^text'** *text_file*

#### Wildcards and Multipliers

* . - matches any single character
* * - matches 0 or more of the previous characters

* r\[a, e, o, u]t - looks for rat, ret, rot, rut
/{2/} - matches exactly 2 occurrences of a character
/{1,3/} - matches from 1 to 3 occurrences of a character

#### Extended Regex

**$ grep -E** or **egrep**

* + - matches 1 or more of the previous characters
* ? - matches 0 or 1 of the previous characters

colou?r  - matches color or colour. 0 or 1 occurrences of letter "u"

$ grep 'b.\*t' - matches anything starting with 'b' and ending with 't'
$ egrep 'b.+t' - matches anything starting with 'b' , at least least 1 character in between and ending with 't'

\\b...\\b - word boundary

