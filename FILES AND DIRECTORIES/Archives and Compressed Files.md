
**tar** - **Tape AR**chiver - designed to stream uncompressed data to a tape

#### **Create a new archive**

``` bash
tar -cvf "FILENAME".tar "FILE(S)_TO_ARCHIVE"
```

- `-c`: Create archive
- `-v`: Verbose
- `-f`: File

Same result using data streams

``` bash
tar -cvf - "/DIR" | cat > "ARCHIVE.tar"
```

#### **Add a new file to an archive**

`-r` option

``` bash
tar -rvf "ARCHIVE.tar" "FILE_TO_ADD"
```

#### **Update files in an archive**

`-u` option

``` bash
tar -uvf "ARCHIVE.tar" "/UPDATED_FILES"
```

#### **See the contents of an archive**

`-t`, `--list`

``` bash
tar -tvf "ARCHIVE.tar"
```

Look for a file within an archive

``` bash
tar -tvf "ARCHIVE".tar "FILENAME"
```

The `--strip-components=1` option strips away the first component from each file path. 

``` bash
tar -zx --strip-components=1 -C /"DESTINATION" -f "ARCHIVE".tar.gz
```
#### Analyze the type of a file

``` bash
file "FILE|DIR"
```

#### Extracting files from archive

`-x`: Extract

``` bash
tar -xvf "FILENAME.tar" # in same dir
```

``` bash
tar -xvf "FILENAME.tar" -C "/TARGET_DIR" # (in target dir)
```

Extract a particular file from an archive to a specified location

``` bash
tar -xvf "ARCHIVE.tar" -C "/TARGET_DIR" "FILE_TO_EXTRACT"
```

#### Compressing files

``` bash
gzip | bzip2 | xz "ARCHIVE.tar"
```

##### Compressing using tar

- `-z` options (gzip)
- `-J` option (xz)
- `-j` option (bzip2)

Compress archive.tar to archive.tar.bz2

``` bash
tar cjvf "ARCHIVE.tar" 
```

#### Decompress

``` bash
gunzip | bunzip2 "COMPRESSED_ARCHIVE"
```

Using tar

``` bash
tar x{z|j|J}vf "COMPRESSED_ARCHIVE"
```

#### cpio Archives

rpm packages are compressed with the cpio archiver

#### Splitting Archives in Multiple Parts

**man split**

Package 
`coreutils`

To split a file into N parts

``` bash
split -n "NUMBER" "FILE" "FILENAME.part"
```

To split a file into parts of exact size

``` bash
split -b "SIZE" "FIRE" "FILENAME.part"
```

To re-create a file from a sequence/stream

``` bash
cat "FILENAME.part\*"  > "FILENAME"
```

To archive partitions. Partitions are always given explicitly 

``` bash
tar -cvf "ARCHIVE".tar --one-file-system "/PARTITION1" "/PARTITION2" \ --exclude "/PARTITION"
```

`--one-file-system`: Stay in local file system when creating archive. Pseudo filesystems are ignored

#### Aggregating Files with find

Look for particular files and archive them

``` bash
find "SEARCH_ROOT_DIR" -iname "*.mp4" -exec tar -rvf "ARCHIVE".tar {} \;
```

`-iname`: Makes the search pattern case insensitive
#### Transfer an Archive over SSH Using Streams

This generates a stream of data sourced at the given DIR, pushes it to ssh and executes a cat against the incoming stream, then assembles the data into a tar file onto the remote machine.

``` bash
tar -cvf - "DIR/" | ssh "REMOTE" "cat > '/REMOTE_DIR/ARCHIVE_NAME.tar'"
```

