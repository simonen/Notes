
man `wget`

Download recursively directories. This will download the Routing/ dir only but will create the dir structure all the way to the root dir + the host (www.website.com)

``` bash
 wget [-r] [-np] [-nH] [--cut-dir=N] --reject "index.html*" https://'SERVER.COM'/'LEVEL1'/'LEVEL2'/'LEVEL_N'/'TARGET_DIR'/
```

- `-r`: Recursive download
- `-np`: No parent. Do not traverse levels above the target dir
- `-nH`: No host. Do not include the server's hostname
- `--cut-dir=N`: Do not include the first N dir levels in the downloaded directory tree

```
Removing Routing/Public Routing Labs/EIGRP/index.html?C=N&O=A.tmp since it should be rejected.
```

To download just the target directory and its contents and all levels below

```bash
 wget -r -nH -np --cut-dir='N' --reject "index.html*" https://'SERVER.COM'/'LEVEL1'/'LEVEL2'/'LEVEL_N'/'TARGET_DIR'/
```

