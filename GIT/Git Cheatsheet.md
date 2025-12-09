
Set up local repo username and email

```bash
git config --local user.name 'USERNAME'
git config --local user.email 'EMAIL'
```

Set up global username and email

```bash
git config --global user.name 'USENAME'
git config --global user.email 'EMAIL'
```

List all tracked files

```bash
git ls-files
```

Clone a repo. Sets up branch tracking automatically

```bash
git clone 'SOURCE REPO'.git 'DEST'
```

Displaying commits

```bash
git log [OPTIONS]
```

`--oneline`: Displays hash and message only

Display repo tree

```bash
git log --oneline --graph --decorate --all
```

Display files only for the last 3 revisions

```bash
git log --oneline --name-only HEAD~3..HEAD
```

Open a file from a revision

```bash
git show 'REVISION':'FILE
```

Create branches

```bash
git branch 'NAME'
```

Create and checkout a branch

```bash
git checkout -b 'BRANCH'
```

Delete a branch

```bash
git branch -d 'BRANCH'
```

#### Remote Repos and Branches

Display remote-tracking branches

```bash
git remote -v
git branch -r
git branch -a -vv
```

Check the relation between local-tracking and remote-tracking branches

```bash
git status -sb
```

Create an empty initial revision

```bash
git commit --allow-empty -m 'MESSAGE'
```


