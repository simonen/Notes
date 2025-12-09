#### Remote Repositories and Synchronization

##### Manual Cloning

1. Initialization
2. Definition of a remote
3. Downloading the git database and remote tracking branches
4. Creating the appropriate branches

Initiate a new repository

```bash
git init [DIR]
```

Add the URL of the remote repository. Link the repository to the remote source using:

```bash
git remote add origin 'URL'.git
```

**`origin`**: The alias name for the remote. You can use any name instead of `origin`, but `origin` is a convention.

This will add the following lines

`.git/config`
```
[remote "origin"]
        url = https://github.com/creationix/js-git.git
        fetch = +refs/heads/*:refs/remotes/origin/*
```

`fetch = 'respec'`
`fetch = +refs/heads/[REMOTE_BRANCH]:refs/remotes/ALIAS/[REMOTE-TRACKING-BRANCH]`
`fetch = +refs/heads/*:refs/remotes/origin/*`: Defines a `refspec`. This means that all files in the `.git/refs/heads` directory in the remote repo aliased as `origin` are mapped 1:1 to the local directory `.git/refs/remotes`. This mapping is used during the `git fetch` and `git pull` operations.

Verify the remote

```bash
git remote -v
```

```
origin  https://github.com/creationix/js-git.git (fetch)
origin  https://github.com/creationix/js-git.git (push)
```

Fetch the repository data. Download the commits and object from the remote repo. This downloads all the data but doesn’t create any working tree files yet.

```bash
git fetch 'ALIAS or URL' [--no-tags] [refspec]
```

The fetch operation defaults to downloading the branches as specified in the `refspec` in `.git/config`

```bash
git fetch origin
```

```
remote: Enumerating objects: 2677, done.
From https://github.com/creationix/js-git
 * [new branch]      clean      -> origin/clean
 * [new branch]      doc        -> origin/doc
 * [new branch]      legacy     -> origin/legacy
 * [new branch]      master     -> origin/master
 * [new branch]      phase2     -> origin/phase2
 * [new tag]         0.4.0      -> 0.4.0
```

The remote-tracking branches are now in place

`.git/refs/remotes`
```
│   └── refs
│       └── remotes      # remote-tracking branches
│           └── origin
│               ├── clean
│               ├── doc
│               ├── legacy
│               ├── master
│               └── phase2
```

If the remote-tracking branches are deleted, the next `git fetch` will update the `refs/remotes` directory and bring them back.

Create a new local branch (`master`) tracking the remote branch (`origin/master`).

```bash
git checkout -b master origin/master
```

`.git/refs/heads`
```
└── refs
    ├── heads
    │   ├── master
```

To ensure that the local `master` branch points to the same revision as the remote `master`
the hash in `.git/refs/heads/master` must be identical to `.git/refs/remotes/origin/master`

Set the local branch to track the remote branch for easier pulls and pushes

```bash
git branch --set-upstream-to=origin/'REMOTE-BRANCH' 'LOCAL_BRANCH'
```

If we have mapped the remote `master` branch to our local `master` branch

`git branch -a -vv`
```
* master                8132153 [origin/master]
```

This command creates the following entry

`.git/config`
```
[branch "master"]
        remote = origin
        merge = refs/heads/master
```

This setup allows **automatic branch tracking** between the local `master` branch and its corresponding remote branch (`origin/master`).

The same can be achieved by using two operations

```bash
git config branch.'LOCAL_BRANCH'.remote 'REMOTE_ALIAS'(origin)
git config branch.'LOCAL_BRANCH'.merge refs/heads/'REMOTE_BRANCH'
```

`git config --unset ...` : To unset

Set the default branch for the remote repository

```bash
git symbolic-ref [options] <name> [<ref>]
```

- **`<name>`**: The name of the symbolic reference to query or update (e.g., `HEAD`).
- **`<ref>`**: The reference to point to (e.g., `refs/heads/master`).

```bash
git symbolic-ref refs/remotes/origin/HEAD refs/remotes/origin/master
```

The command will create a HEAD file in the `.git/refs/remotes/origin` containing a symbolic reference pointing to `refs/remotes/origin/master`. This sets up the default branch for the remote repo.

To delete a symbolic reference

```bash
git symbolic-ref -d refs/remotes/origin/HEAD
```

To read a symbolic reference. Paths are relative to `.git/`

```bash
git symbolic-ref HEAD
```
