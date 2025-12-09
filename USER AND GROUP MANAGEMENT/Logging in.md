
man loginctl
man lastlog

```
logname
lastlog
whoami
w
who
users
last
lastb: lists bad logins, incorrect password or other issues
```


**MOTD_FILE**: Message of The Day file. Defines welcome message on login.

man issue

- `/etc/issue` : for local logins
- `/etc/issue.net` : for SSH logins

#### User Prompt

[BASH PROMPT CUSTOMIZATION](https://tldp.org/)

The PS1 variable defines the user prompt

``` bash
echo $PS1
[\u@\h \W]\$
```

- `\u`: username
- `\h`: hostname
- `\W`: working directory
- `\T`: timestamp

See PROMPTING section in the bash man page

To customize the user prompt

``` bash
PS1="(\T)[\u@\h:\W] $ "
(07:43:52)[root@prometheus:tmp] $
```

#### Environment Variables

- `/etc/profile`
- `/etc/profile.d/`
- `~/.bash_profile`

To add a directory to the start of the PATH variable

``` bash
PATH="/home/kimchen/documents":$PATH
```

To print the PATH variable

```
echo $PATH
/home/kimchen/documents/:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/root/bin
```

To add a path to the end of the PATH

``` bash
PATH=$PATH:"/home/kimchen/documents"
```

These effects are temporary, lasting only until the end of the session. To make changes permanent, add the modifications in the `~/.bash_profile` file, + the export command to propagate the change.

`~/.bash_profile`
```
PATH=$PATH:"/home/kimchen/documents"
export PATH
```

#### Aliases

www.tldp.org/LDP/Bash-Beginners-Guide/html/ and http://tldp.org/HOWTO/Bash-Prog-Intro-HOWTO.html.

To get a listing of all aliases

``` bash
alias
```

```
alias cp='cp -i'
alias egrep='egrep --color=auto'
alias fgrep='fgrep --color=auto'
alias grep='grep --color=auto'
alias l.='ls -d .* --color=auto'
alias ll='ls -l --color=auto'
alias ls='ls --color=auto'
alias mv='mv -i'
alias rm='rm -i'
alias which='alias | /usr/bin/which --tty-only --read-alias --show-dot --show-tilde'
```

Make an alias

```
alias ll='ls -lah'
```

To delete an alias

``` bash
unalias "ALIAS"
```

