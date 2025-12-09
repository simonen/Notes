
* Anyone who can read a file can copy it
* root and privileged users can read any file
* Permissions and ownership are functions of the filesystem and can be bypassed by reading a storage device from another system. Just mount the storage device with root privilege

**man stat**

To list just the file / dir permissions and ownership

``` bash
stat --format=%a:%A:%U:%G "FILE/DIR"
```

`755:drwxr-xr-x:root:root`

755: octal notation
drwxr-xr-x: symbolic notation