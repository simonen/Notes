It is a good practice to start a script with a shebang
`#!/bin/bash`

**exit 0**: Exit status. 0 means OK

To request the exit status of the last command executed:

``` bash
echo $?
```

To execute a script
$ ./script
**$ bash script**

Executing a script with the **bash** command does not require execute permissions

#### Positional Parameters

**$ command** *ARGUMENT*

**$ useradd lisa**
**useradd**: command
**lisa**: argument

For Loops:

```
for i in iterable
do
	command
done
```

```
#!/bin/bash

echo you have entered $# arguments

for i in $@
do
	echo $i
done

exit 0
```

$#: returns the count of arguments entered
$@: list of arguments
$i: current argument

#### Variables

Variables are always uppercase. No space between the '='. If declared statically:
NAME=value
NAME=$variable

Dynamically, using **read**
To assign a command to a variable
NAME=$(command)

To call a variable:
$NAME

#### Conditional Loops

**if, then else**
**while**: as long as a condition is true
**until**: until a condition is true
**case**

IF statement

```
if [ -z $1 ]
then
	command
fi
```

or

```
if [ -z $1 ]; then
	command
fi
```

[ -z $1 ] - test command. -z checks if $1 is empty
[ -f $1 ] : -f checks if file exists
[ -d $1 ] : -d checks if directory exists

ELIF
```
if [ test ]; then
	command
elif [ text ]; then
	command
else
	command
fi
```

#### Logical Operators || and &&

|| : OR - executes the second part only if the first part is NOT true
&& : AND - executes and second part only if the first is true

#### FOR Loops

Iterating through lists

```
for i in item1 item2 item3; do
	command
done
```

Iterating through a defined range:
```
for (( COUNTER=100; COUNTER>0; COUNTER--)); do
	command
done
```

```
for i in {100..1}; do echo $i
done
```

#### WHILE and UNTIL

**while**: executes as long as the condition is true
**until**: executes as long as the condition is not met

as long as the user is logged, the command keeps executing
```
while users | grep username; do
	command
done
```

as long as the user is not logged in, the command keeps executing
```
until users | grep username; do
	command
done
```

#### xargs

`xargs` reads standard input and passes it as arguments to a command. Default `xargs` command is `echo`. Arguments from `stdin` are space-delimited by default

xargs \[ OPTIONS ] *COMMAND* < STDIN
STDIN | xargs \[ OPTIONS ]

EXAMPLE

To execute the **file** command against a stream of filenames, passed from stdin as args
$ **ls | xargs file**

```
[root@dnsserver ~]# ls | xargs file
/dev/stdin:               empty
07:23:51:        empty
07:24:34:        directory
2024:            directory
28:              empty
anaconda-ks.cfg: ASCII text
awkvars.out:     Algol 68 source, ASCII text
EET:             empty
hosts:           ASCII text
huizinger:       ASCII text
Mar:             empty
pen4o:           ASCII text
PM:              empty
script:          ASCII text
scrit:           Bourne-Again shell script, ASCII text executable
```

This works because **file** or **stat** can take multiple arguments at a time. 

`xargs` executes echo against the `stdin` stream when with no arguments

```
[root@dnsserver ~]# ls | xargs
- 07:23:51 07:24:34 07:24:47 2024 28 anaconda-ks.cfg awkvars.out EET files hosts huizinger Mar pen4o pi4a pingi PM script scrit
```

To put each argument on a separate line

```bash
ls | xargs | xargs -n 1
```

```
[root@dnsserver ~]# ls | xargs | xargs -n 1
-
07:23:51
07:24:34
07:24:47
2024
28
anaconda-ks.cfg
awkvars.out
EET
files
hosts
```

To put two arguments per line, etc...

```bash
ls | xargs | xargs -n 2
```

```
[root@dnsserver ~]# ls | xargs | xargs -n 2
- 07:23:51
07:24:34 07:24:47
2024 28
anaconda-ks.cfg awkvars.out
EET files
hosts huizinger
Mar pen4o
pi4a pingi
PM script
scrit
```

```
[root@dnsserver ~]# cat hosts
192.168.137.23
192.168.137.12
192.168.137.11
```

To execute the ping command against a list of IP addresses from a *hosts* file

```bash
cat hosts | xargs -n 1 ping -c 4
```

```
[root@dnsserver ~]# cat hosts | xargs -n 1 ping -c 1
PING 192.168.137.23 (192.168.137.23) 56(84) bytes of data.
64 bytes from 192.168.137.23: icmp_seq=1 ttl=64 time=0.059 ms

--- 192.168.137.23 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.059/0.059/0.059/0.000 ms
PING 192.168.137.12 (192.168.137.12) 56(84) bytes of data.
64 bytes from 192.168.137.12: icmp_seq=1 ttl=64 time=0.563 ms

--- 192.168.137.12 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.563/0.563/0.563/0.000 ms
PING 192.168.137.11 (192.168.137.11) 56(84) bytes of data.
From 192.168.137.23 icmp_seq=1 Destination Host Unreachable

--- 192.168.137.11 ping statistics ---
1 packets transmitted, 0 received, +1 errors, 100% packet loss, time 0ms
```

This works exactly as a **for loop** with one IP per line

```
for i in $(cat hosts); do ping -c 1; done
```

As the **ping** command can take a single argument (ip address) at a time, the -n 1 flag is necessary. Without it, **xargs** executes the default **echo** command first which puts all IPs on a single line and thus - as a single argument

```
[root@dnsserver ~]# cat hosts | xargs
192.168.137.23 192.168.137.12 192.168.137.11
```

To pass two arguments at a time

```
[root@dnsserver fatty]# cat pingi
1 192.168.137.12
2 192.168.137.23
```

The first field will be the passed to the -c flag for packet count and the second as the IP

```bash
cat pingi | xargs -n 2 ping -c
```

The first host will be pinged once, the second twice. 
- `-n 2`: ensures that exactly two arguments are passed at a time

#### heredoc

**heredoc** is a technique for building simple multi-line text on the fly

```
cat << EOF [> output.txt]
This is
a multi-line
script
EOF
```


