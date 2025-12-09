
#### Types of files

- `-`: regular file or binary
- `d`: directory - a file that contains a list of other files
- `l`: symlink
- `c`: character device
- `b`: block device
- `s`: socket - allow interprocess communication
- `p`: named pipe

#### Listing Files and Dirs

**man ls**
**man tree**
**man stat**

To return to the last used dir

``` bash
cd -
```

List one file / dir per line, no stats

``` bash
ls -C1 or just $ ls -1
```

``` bash
[kimchen@server2 ~]$ ls -C1
CentOS-8.3.2011-x86_64-dvd1.iso
Desktop
Documents
Doknq
```

List directories on top

``` bash
ls --group-directories-first
```

Sort natural numbers. Treats numbers as integers instead of strings

``` bash
ls -v
```

Displaying dir structure using **tree**

``` bash
tree [-L 1] 
```

Displaying file and dir stats with **stat**

``` bash
stat "FILE"
```

Display a directory disk usage

``` bash
du -sh /"DIR"
```

```
[kimchen@prometheus tmp]$ du -sh /home/kimchen
113M    /home/kimchen
```
#### Creating Files and Dirs

Create a dir and all dirs above it that does not exist using -p

``` bash
mkdir -p "/PARENT_DIR/CHILD1/CHILD2"
```

``` bash
mkdir -v "DIR"
```

Setting permissions while creating a dir

``` bash
mkdir -m 0700  DIR
```

Create multiple numbered dirs, 0 to 10

``` bash
mkdir "NAME{00..10}.EXT"
or
for i in {0..10}; do mkdir "NAME"$i; done
```

To populate a file with contents at an exact file size - exactly 100MB 

``` bash
yes "STRING" | head -c 100MiB > "FILE"
```

**yes**  prints an infinite stream of a given string on separate lines, **head -c** N picks the first N megabyte worth of content in this case. **-n** N picks the first N lines

#### Moving Files and Dirs

**man mv**

Useful options:
* `-a`, archive: preserve file attributes, timestamps 
* `-n`, no-clobber: prevents overwriting destination files
* `-u`, update: overwrites destination files only if source files are newer

#### Date and Time

- `atime`: last time a file was accessed
- `ctime`: when a file was created

To display file creation time

```bash
ls -lc 
```

To display file last access time

``` bash
ls -lu
```