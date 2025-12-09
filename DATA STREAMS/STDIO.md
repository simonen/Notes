
File descriptors

```
[root@dnsserver proc]# stat /dev/fd
  File: /dev/fd -> /proc/self/fd
```

```
[root@dnsserver proc]# ll /proc/self/fd
lrwx------. 1 root root 64 Mar 27 19:55 0 -> /dev/pts/0
lrwx------. 1 root root 64 Mar 27 19:55 1 -> /dev/pts/0
lrwx------. 1 root root 64 Mar 27 19:55 2 -> /dev/pts/0
lrwx------. 1 root root 64 Mar 27 19:55 255 -> /dev/pts/0
```

The block above shows that by default all data streams are wired to the pseudo-terminal the current user is hooked up to. 

Regardless of how a SHELL is accessed, the following are always wired
- `/dev/fd/0` - STDIN
- `/dev/fd/1` - STDOUT
- `/dev/fd/2` - STDERR

`fd`: file descriptor
#### STDIN Standard In

Common STDIN locations:
* **Keyboard**: `/dev/fd/0`
* **File**: `/dev/fd/0`
* **Pipe**: `/dev/fd0`

Redirection symbols
\> STDOUT, writes input to a file
\< : STDIN, takes input
|, pipe: takes the output and passes it as input to the next function. Connects output and input streams

This will redirect the stream from the FILE, which is user@host, to the ssh command as argument

```bash
cat < FILE | ssh $(cat)
```

#### Using cat

**cat**: concatenate
Takes data from STDIN and hands it over to STDOUT

cat \[ - ] - waits on: /dev/fd/0 (STDIN) for input. Takes input from keyboard if nothing is specified

$ **cat** *FILE* - takes input directly from the file
Same as
$ cat < *FILE*

Can concatenate multiple files

``` bash
cat "FILE1" "FILE2"
```

To scroll up/down a large text file SHIFT+PAGE UP / PAGE DOWN
To scroll up/down one line at a time - CTRL + PAGE UP / PAGE DOWN
#### Using tee

tee \[ - ]
tee *FILENAME*
tee - *FILENAME*
tee -a - *FILENAME*

#### STDOUT Standard Out 

STDOUT - '/dev/fd/1'. Redirects data to

Symbols
(>|>>&1)

Locations
TTY|File|Pipe|Socket

To get a list of active pseudo terminals
$ w

```
[root@dnsserver kimchen]# w
 21:36:16 up 12:37,  3 users,  load average: 0.08, 0.03, 0.00
USER     TTY        LOGIN@   IDLE   JCPU   PCPU WHAT
kimchen  pts/0     09:01    0.00s  1.53s  0.27s sshd: kimchen [priv]
kimchen  pts/1     21:21   15:12   0.06s  0.06s -bash
kimchen  pts/2     21:29    6:38   0.05s  0.05s -bash
```

To direct a stream to a particular pseudo terminal (permissions-permitted)

``` bash
echo "STREAM" > /dev/pts/"NUMBER"
```

The stream should appear to the corresponding user's pseudo terminal STDOUT (screen)

To do this to a remote terminal over ssh

``` bash
echo "STREAM" | ssh "REMOTE" cat /dev/pts/"NUMBER"
```

#### STDERR Standard Error

Symbols

\> >> &2

`>` redirects STDOUT only by default. 
`2>` : redirects STDERR
`2>&1` : Redirects `stderr` (file descriptor 2) to `stdout` (file descriptor 1), ensuring that both the standard output and any errors are written to the same file.

Combining STD( OUT | ERR ) 

To redirect STDOUT + STDERR to a file

``` bash
$ ls 'NON_EXISTING' 'EXISTING' > 'DEST' 2>&1
```

`2> /dev/null` -> redirect only error msgs from the pts0 to nowhere.
` > /dev/null` -> redirect stdout to nowhere, show errors

Some programs redirect STDOUT and STDERR to screen by default
To redirect STDOUT to a file: COMMAND > FILE. This will still print errors to screen but write only stdout to the file.

To execute a second command only if the previous had completed successfully 
$ *COMMAND1* && *COMMAND2*

To execute a second command only if the previous had NOT executed successfully
$ *COMMAND1* || *COMMAND2*