
- `console`: The screen the user is looking at
- `terminal`: the command-line environment that is opened on the console and provides access to the shell ( typically bash), where commands are issued
- `prompt`: \[user@host directory]$ 

[Customize the prompt](https://tldp.org/HOWTO/Bash-Prompt-HOWTO/)
#### Virtual Terminals

`virtual terminal` - offers possibility to open multiple terminals in an non-graphical environment

Change the virtual terminal

``` bash
chvt N 
```

- `/dev/tty#` - devices to which virtual terminals are linked
- `pts` - pseudo terminal services

```
[student@localhost ~]$ w
 16:25:38 up  3:13,  2 users,  load average: 0.00, 0.01, 0.05
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
kimchen  pts/0    192.168.137.9    16:20    5:18   0.11s  0.11s -bash
student  pts/1    192.168.137.1    16:25    1.00s  0.13s  0.02s w
```

Both users are connected via SSH

Show login history (uses the `/var/log/wmtp` log)

``` bash
last
```

To leave a graphical session and enter a console
`CTRL + Fn..`

To return to graphical session
`ALT + Fn`

#### Change Prompt Colors

Works on CentOS 9, ubuntu 24.04

Green prompt \[root@dhcp ~]#

```
PS1='\[\e[0;32m\][\u@\h \W]\$ \[\e[m\]'
```

- `\[\e[0;32m\]`: This starts the **green** color for the prompt. \[1;32m\\] - light green
- `[\u@\h \W]\$`: Displays the **prompt** (`[username@hostname current_directory]$ or #`).
- `\[\e[m\]`: This **resets** the color to the terminal's default.
	- `\e[31m`: Red
	- `\e[32m`: Green
	- `\e[33m`: Yellow
	- `\e[34m`: Blue
	- `\e[35m`: Magenta
	- `\e[36m`: Cyan
- `\u` represents the username.
- `\h` represents the hostname.
- `\w` represents the current working directory.

Apply

``` bash
source ~/.bashrc # per user
source /etc/bashrc # global
```

`/etc/bashrc`
```bash
if [ "$UID" -eq 0 ]; then
    PS1='\[\e[0;36m\][\u@\h \W]# \[\e[m\]'
else
    PS1='\[\e[0;32m\][\u@\h \W]$ \[\e[m\]'
fi
```

`[root@centos10 ~]#`