
Apress - Git Recipies

Git is a distributed version control system

`shallow repository`: The history of a repository is truncated to a given number of of the latest revisions

Porcelain commands: High-level commands for everyday use. `git help` lists porcelain level commands

``` bash
git add
git commit
git help
git push
```

Plumbing commands: Low level commands. `git help -a` lists plumbing level commands

#### Configure Git

To create commits within git repository, `user.name` and `user.email` settings are required. These strings will be stored in every commit. 

See/set git user name/email

``` bash
git config --global user.name ['USER NAME']
git config --global user.email ['USER EMAIL']
```

`.gitattributes`: This file contains rules that override the settings defined with the git config command
`.gitignore`: Contains a list of rules (patterns) about files or dirs that should not be committed.
#### Cloning a Repository

Two methods of cloning a repository: HTTP and SSH

To copy an entire remote repository locally

``` bash
git clone https://github.com/'repo'.git
```

To clone a local repository

```bash
git clone 'SOURCE REPO'/ 'DEST'/
```

The config file shows the origin of the repo. In this case - local

`.git/config`
```
[core]
        repositoryformatversion = 0
        filemode = true
        bare = false
        logallrefupdates = true
[remote "origin"]
        url = /root/git-recipies/02-01/jquery/
        fetch = +refs/heads/*:refs/remotes/origin/*
[branch "main"]
        remote = origin
        merge = refs/heads/main
```

`[remote "origin"]`: Stores the address passed passed to the git clone command.

The repo structure

```
.
├── .git # Git directory
├── ...
├── ... } The working (temporary) directory
├── ...
├── .git/objects # The database
├── .git/index # The list of staged files
```

The working directory is a temporary storage that contains your work. The `git` directory contains the database that stores all snapshots of the project.

To restore the contents of the working directory (if `.git` is present)

``` bash
git reset --hard
```

Clone a bare repository. Contains only the contents of the `.git` directory. Used for sync purposes only.

``` bash
git clone --bare 'SOURCE_REPO' 'DEST'
```

`bare`: Contains only the git directory
`non-bare`: Contains the git and working directories
#### Exploring Commit History

The git history consists of a series of revisions. Each revision is a snapshot for the working directory at a particular point in time. Revisions are stored in the `.git/objects` database.

Every revision is uniquely identified by its name - SHA-1 hash automatically generated from the author's name, timestamp, etc. 

To list the complete list of revisions that are accessible from the current version

``` bash
git log
```

To print simplified and shortened log information

``` bash
git log --abbrev-commit --abbrev=4 --pretty=oneline -10
```

Displays the last 10 revisions with abbreviated SHA-1 name to 4 characters

```
[root@localhost jquery]$ git log --abbrev-commit --abbrev=4 --pretty=oneline -10
d5ebb (HEAD -> main, origin/main, origin/HEAD) Build: Make middleware-mockserver not crash on reading nonexistent files
32966 Selector: Properly deprecate `jQuery.expr[ ":" ]`/`jQuery.expr.filters`
07c9 Build: Bump the github-actions group with 4 updates
19716 Build: Run tests on Node 22 & 23
d928 Docs: Align CONTRIBUTING.md with `3.x-stable`
```

To show only the simplified last revision on one line

``` bash
git log --oneline -1
```

To list revisions authored by specific author

``` bash
git log --author='AUTHOR'
```

Other options
`--since="2021-12-20"`
`--until="DATE..."`

To list contributors and their revisions. `-s` for displaying names only. `-n` sort by revs

``` bash
git shortlog [-s] [-n]
```

List files in the working directory

``` bash
git ls-files
```

#### Creating Local Repositories

Initialize the working directory as a repository or create a new one

``` bash
git init ['DIRECTORY']
```

```
Initialized empty Git repository in /root/git-recipies/03-01/.git/
```

```
.
└── .git
```

Check the status of the repo

``` bash
git status -s
```

The output returns an untracked file denoted with `??`.  This also means the repo is dirty.

```
?? agatha-christie.txt
```

Select files for revision. `-A` for the entire working directory

``` bash
git add -A ['FILE']
```

The file is in the staging area ( **staged file** ) and ready to be committed. List of staged files is stored in `.git/index` - **staging area** or **index**.

```
[root@localhost 03-01]$ git status -s
A  agatha-christie.txt
```

Create the new revision (commit the file) and store it in database

``` bash
git commit -m "COMMENT"
```

To stage and commit all files in a single command

``` bash
git commit -a -m "COMMENT"
```

Check out history

``` bash
git log --oneline
```

```
56a58d4 (HEAD -> master) Third Revision [Stephen King]
cf59a3b Second Revision [John Grisham]
c9ca92f First revision [Agatha Christie]
```

#### File States

``` bash
git status [-s]
```

`(STAGING_AREA)(WORKDIR)`
`untracked ( ?? )`:  First `?` - file is unknown in staging area. Second `?` -unknown in wd
`staged`
`unstaged`
`unmodified`

Staging commands
`git add`
`git rm`
`git mv`

To `unstage` or remove a file from the staging area

``` bash
git rm --cached -- [filename]
```

``` bash
git reset -- [filename]
```

When modifying files, the status command will return `M_` or `_M` states

`_M`:  File was not added to the staging area, `M`: file was modified in working directory
`M_`: File was added to staging area, The state of file in staging area is identical to the state in the file in the working directory

To reverse `M_` state back to `_M`

``` bash
git checkout -- [filename]
```

##### Deleting a File with git rm

To remove a file from both the **working directory** and the **staging area**, marking it for deletion in the next commit

``` bash
git rm "FILE"
```

The file enters `D_` state. Deletion has been added to the staging area. To proceed with the removal, commit the changes

To `unstage` the removal

``` bash
git reset -- [file]
```

The file enters `_D` state: The file has been deleted in the working directory but is not yet staged for deletion. Shows up as a deleted file because it is missing from the working directory compared to the last commit.

To restore the file in the working directory

``` bash
git checkout -- [file]
```

##### Delete a File with rm

Deleting a file with the linux rm command will put the file in `' D'` state - deletion unstaged. To stage the deletion and commit the change

``` bash
git commit -a -m 'COMMENT'
```

To restore the file in the working directory (if not committed). Puts the file from `' D'` to unmodified state.

``` bash
git checkout -- 'FILENAME'
```

##### Untrack an Unmodified File

To convert an unmodified file into an untracked file, i.e remove it from the snapshot only

``` bash
git rm --cached 'FILENAME'
```

This will put the file into two states `D_` and `??`, . The next commit will store the snapshot of the working directory without the file. `git status` will show it as an "untracked" file again.

``` bash
git commit -m 'COMMENT'
```

##### Renaming Files with git mv

``` bash
git mv old-name.txt new-name.txt
```

```
R  old-name.txt -> new-name.txt
```

'R ': Renaming is staged. The state of the file in the working directory == as in the staging area.

To undo the operation, unstage the renaming op and restore the old file

``` bash
git reset --
git checkout -- 'FILENAME'
```

#### Git Aliases

`~.gitconfig`
```
[alias]
    snapshot = "!f() { COMMENT=\"${*:-wip}\"; git add -A && git commit -m \"$COMMENT\"; }; f"
```

This alias will combine the git add -A and git commit -m... commands into a single `git snapshot COMMENT` command. The comment is passed to the snapshot command as arguments.

``` bash
git snapshot Sing a song of sixpence
```

```
[root@localhost 03-03]$ git log --oneline
d933c7f (HEAD -> master) Sing a song of sixpence
```

#### Restoring Revisions

`git reset` 

`git reset --hard`: With no specified commit will reset to the HEAD of the current branch. Discards working directory changes, resets the staging area to match the specified commit.
`git reset --hard <commit>`: Moves the branch pointer, clears the staging area, and resets the working directory, discarding all changes
`git reset --`: Clears the staging area. Unstages any files that were previously staged for commits. It does not change files in the working directory.

List all revisions of a repo

```
[root@localhost 03-05]$ git log --oneline
dc37c7a (HEAD -> master, origin/master, origin/HEAD) Mapping names with .mailmap
f6c731c [FR] Frere Jacques
b22b19f [EN] Little skylar, lovely little skylark
81a3e5e [FR] Alouette, gentille alouette
7b61196 Titles
f82919f [PL] Bajka iskierki
91f11f5 Internationalization: directory EN
455955e Baa, baa black sheep
d933c7f Sing a song of sixpence
```

To restore the working directory to a revision

``` bash
git reset --hard ['REVISION']
```

To reset the state of the working directory to the first revision named `d933c7f`. 

``` bash
git reset --hard d933c7f
```

```
[root@localhost 03-05]$ git log --oneline
d933c7f (HEAD -> master) Sing a song of sixpence
```

Restoring revisions removes revisions from the history but they are still available in the database. The history of the repo is a subset of the information in the database. Only a valid name is needed to retrieve it.

To restore the revision denoted as `7b61196` ( Titles )

``` bash
git reset --hard 7b61196
```

```
[root@localhost 03-05]$ git log --oneline
7b61196 (HEAD -> master) Titles
f82919f [PL] Bajka iskierki
91f11f5 Internationalization: directory EN
455955e Baa, baa black sheep
d933c7f Sing a song of sixpence
```

To restore the original state of the repo, the name of the last revision is required

##### Restoring Revisions with git checkout

1. Enters a `detached HEAD` state
2. Resets the state of the working directory to the specified revision
3. Removes the history of all revisions created after the specified revision

`detached HEAD`: Special state in which you are not on any branch

Restore the working directory to the very first revision

``` bash
git checkout d933c7f
```

This changes the state of the repository into a "detached HEAD" state.

``` bash
git status [-sb]
```

```
HEAD detached at d933c7f
```

Return to the latest version and restore the normal state.

``` bash
git checkout master
```

To return information about current branch

``` bash
git status -b
```

##### Working with reflog

`reglog` is a special log that stores information about movements in the repository. Each time you create a revision, reset the repository or change the current revision otherwise, the `reflog` is updated.

``` bash
git reflog
```

``` bash
36b3e4c (HEAD -> master) HEAD@{0}: commit: dolor
c2f028d HEAD@{1}: commit: ipsum.txt
67c3673 HEAD@{2}: commit (initial): lorem
```

Restore to the lorem revision using its `reflog` name

```bash
git reset --hard HEAD@{2}
```

```
67c3673 (HEAD -> master) HEAD@{0}: reset: moving to HEAD@{2}
36b3e4c HEAD@{1}: commit: dolor
c2f028d HEAD@{2}: commit: ipsum.txt
67c3673 (HEAD -> master) HEAD@{3}: commit (initial): lorem
```

`HEAD@{0}`: Always points to the current revision. A previous revision is available as `HEAD@{1}`. This allows us to refer to previous revisions and roll back changes even if we don't know the exact revision name. Also called `dangling revisions`

#### Losing Uncommitted Changes

> Modified files that are uncommitted are lost if the working directory is rolled back to a previous state. This can be mitigated by resetting to the relative previous state using `reflog` name `HEAD@{1}` before the unsaved changes were introduced.

* `reachable objects`: Objects that are available through various symbolic references, such as `reflog`, branches and tags
* `unreachable objects`: Objects that are available only by their SHA-1 names
##### Losing Commits

Resetting to an older revision clears the revision history after the now current revision but the commit info is still present in the git database. To delete the commits after the current revision, the `reflog` must be cleaned

``` bash
git reflog expire --all --expire=now
```

The `reflog` is now empty. Revisions are now available only through their SHA-1 names. There are not symbolic names leading to revisions. To check which objects stored in the git database are accessible only by their SHA-1 names

``` bash
git prune --dry-run
```

These objects are classified as `unreachable` and will be deleted during automatic garbage collection process

```
58da9cdd7e83cc4f54ac2aef61e2cba90a1538a2 commit
61780798228d17af2d34fce4cfbdf35556832472 blob
f4b354863caa9cea99b95422c9dab70465757d87 tree
```

To manually delete unreachable objects

``` bash
git prune
```

The current revision is now permanent and deleted revisions cannot be restored

> Working in `detached HEAD` state creates unreachable revisions

#### Branches

* `.git/HEAD`: Points to the current revision. Used as parent revision during next commit. 
* `.git/refs/heads/<branch>`: Stores the commit hash of the branch's latest commit. Git uses it to track the current state of each branch.
* `.git/packed-refs`: Stores a compact list of references (packed refs) like branches, tags.. for efficiency in large repos.
##### Creating and Switching branches

A `branch` is a line of development, a pointer to an arbitrary commit in the database. Branches are independent of each other.

`tip of the branch`: Last commit in a branch

Create a new branch. If no revision is specified, defaults to HEAD

``` bash
git branch 'BRANCH_NAME' ['REVISION']
```

> New branches stem from the current branch if no additional arguments are given. The working directory of new branches will contain the files from the branch it stemmed from

To create and switch to a new branch, pointing to the master revision

``` bash
git checkout -b 'NEW_BRANCH' master
```

List branches. `*` denotes current branch

``` bash
git branch
```

```
  doc
* master
```

To print the latest commit (revision) in a branch

``` bash
git branch -v
```

To list files stored in a particular branch without switching to

```bash
git show 'BRANCH'^{tree}
```

Switch to a different branch

``` bash
git checkout 'BRANCH'
```

List the repo tree

```
git log --oneline
```

```
6f1ce08 (HEAD -> info) i3
d7ed465 i2
5ffefdc i1
e12897c (master) m3
f5b6ad4 m2
5dfddf6 m1
```

```bash
git log --oneline --graph --decorate --all
```

```
* 6f1ce08 (HEAD -> info) i3
* d7ed465 i2
* 5ffefdc i1
| * 818b756 (doc) d3
| * f779c18 d2
| * ef4bcc9 d1
|/
* e12897c (master) m3
* f5b6ad4 m2
* 5dfddf6 m1
```

To store all references, including branches in packed format in `.git/packed-refs`

``` bash
git pack-refs --all
```

This clears the `.git/refs/heads` directory.

```
# pack-refs with: peeled fully-peeled sorted
818b75667ce091e1fe066854828b4c05c279f044 refs/heads/doc
6f1ce081c69cd86ddcda88839231b71595622c99 refs/heads/info
e12897cb429edc6d6c24801ec2dc28880e746819 refs/heads/master
```

`refs/heads/<branch>`: Symbolic form of the reference. Stores the last commit SHA-1

Branches are stored in the .git directory in two different formats
* Loose format: Every branch is stored in a separate file. `refs/heads/xyz` is a path to
* Packed format: Many references are stored in a single file `.git/packed-refs`

##### Cloning Branched Repositories

Types of local branches:
* Local branches
* `Remote tracking branches`: Local copies of remote branches. They preserve the state of the remote branch as it was during the initial clone or last fetch operation. Stored in the packed-refs file. Read-only copies of remote branches.
* `Local tracking branches`: Used to publish commits in a remote branch. Connected to remote tracking branches.

Clone a remote repo

``` bash
git clone 'REMOTE REPO'/ 'LOCAL REPO'/
```

The new repository will contain only the current branch of the source repository as a local branch, as referred to in its HEAD file.

To list branches

``` bash
git branch 
```

```
* master
```

To print all branches.

``` bash
git branch -a
```

```
* master
  remotes/origin/HEAD -> origin/master # contents of the HEAD file of remoterepo
  remotes/origin/doc
  remotes/origin/info
  remotes/origin/master
```

Other `git branch` options
`-r`: List remote tracking branches
`-vv`: Print last commits

Local pointers stored in the `.git/refs/heads` directory are missing and have to be created manually with `git checkout`

Create a new local tracking branch for a remote branch with the same name

``` bash
git checkout 'BRANCH'
```

```
branch 'doc' set up to track 'origin/doc'.
Switched to a new branch 'doc'
```

```
* bar                   e12897c m3
  doc                   ed40084 [origin/doc] d5
  foo                   e12897c m3
  info                  6f1ce08 [origin/info] i3
  master                e12897c [origin/master] m3
  remotes/origin/HEAD   -> origin/master
  remotes/origin/doc    ed40084 d5
  remotes/origin/info   6f1ce08 i3
  remotes/origin/master e12897c m3
```

`* bar e12897c m3`: local branch
`doc ed40084 [origin/doc] d5`: local tracking branch
`remotes/origin/doc    ed40084 d5`: remote tracking branch
`remotes/origin/HEAD -> origin/master`: The contents of the HEAD file of the remote repository. Tells the current branch of the remote repo.

To check the relationship between the cloned (remote) and local repositories

``` bash
git remote -v
```

The local repo uses the alias `origin` to point to the remote repo

```
origin  /root/git-recipies/05-01 (fetch)
origin  /root/git-recipies/05-01 (push)
```

The `.git/config` file also gives info about the remote repo

To remove the relationship between local and remote repositories

``` bash
git remote rm origin
```

The command will remove the `[remote "origin"]` entry from the repo config file and all remote tracking branches. The repo now contains local branches only

```
* bar    e12897c m3
  doc    ed40084 d5
  foo    e12897c m3
  info   6f1ce08 i3
  master e12897c m3
```

##### Committing in a Detached HEAD State

```
* ed40084 (origin/doc, doc) d5
* ef4bcc9 d1
| * 6f1ce08 (origin/info, info) i3
| * d7ed465 i2 # First parent of info i3 revision
| * 5ffefdc (HEAD) i1 # Second parent of info i3 revision
|/
* e12897c (origin/master, origin/HEAD, master) m3
* 5dfddf6 m1
```

To enter detached HEAD state at the `i1`, which is second parent of the revision pointed to by the `info` branch

``` bash
git checkout info~2
```

```
HEAD is now at 5ffefdc i1
```

Commit a new file x1 in the detached HEAD state.  The parent of x1 is now i1

```
* d115d70 (HEAD) x3
* 0907706 x2
* da6dfbd x1
| * ed40084 (origin/doc, doc) d5
| * 5304790 d4
| * ef4bcc9 d1
| | * 6f1ce08 (origin/info, info) i3
| | * d7ed465 i2
| |/
|/|
* | 5ffefdc i1
|/
* e12897c (origin/master, origin/HEAD, master) m3
* f5b6ad4 m2
* 5dfddf6 m1
```

Switching to another branch will leave the revisions as `dangling revisions`

``` bash
git checkout master
```

```
Warning: you are leaving 3 commits behind, not connected to
any of your branches:

  d115d70 x3
  0907706 x2
  da6dfbd x1
```

To retrieve the revisions into a new branch at the latest commit

``` bash
git branch xeen d115d70
```

To remove dangling revisions

``` bash
git reflog expire --all --expire=now
git prune --dry-run # to show which revisions will be lost
git fsck --unreachable # to show which revisions will be lost
```

##### Resetting a Branch

To cancel uncommitted changes to tracked files

``` bash
git reset --hard
git clean -f # removes untracked files
```

`git clean -n`: dry run

##### Switching Branches in a Dirty Repo Without Conflicts

By default, all changes are preserved even if they were staged or not. If git cannot preserve changes it will refuse to switch branches. Switching branches in a dirty repository is permitted only if the changes that are not committed do not collide with the contents of the branch you are switching to.

> The only situation when you can lose uncommitted changes without warning is when during branch switching you remove a file that wasn't present in the branch you are switching to.

##### Switching Branches in a Dirty Repo with Conflicts

Git will not allow switching to a branch that will cause loss of unsaved changed

```
error: Your local changes to the following files would be overwritten by checkout:
        i1.txt
Please commit your changes or stash them before you switch branches.
Aborting
```

###### Stashing Changes

`git stash` is a handy command when you need to temporarily set aside changes you've made in your working directory but aren't quite ready to commit them. This can be really useful if you need to switch branches or work on something else without losing your current changes.

Stash the uncommitted changes temporarily. Can be stacked.

``` bash
git stash
```

```
Saved working directory and index state WIP on info: 6f1ce08 i3
```

> Stashing saves the current state of the working directory then resets it to the last rev

```
[root@localhost 05-07]$ git log --oneline --graph --decorate --all
*   7238a5c (refs/stash) WIP on info: 6f1ce08 i3
|\
| * fa222b6 index on info: 6f1ce08 i3
|/
* 6f1ce08 (HEAD -> info, origin/info) i3
```

The default WIP message can be replaced with `git stash save "MESSAGE"`

Displays the list of stashes you have saved.

``` bash
git stash list
```

```
stash@{0}: WIP on info: 6f1ce08 i3
```

Apply the most recent stash and removes it from the stash list. Popped stashed changes are merged with the current branch

``` bash
git stash pop
```

```
CONFLICT (modify/delete): i1.txt deleted in Updated upstream and modified in Stashed changes.  Version Stashed changes of i1.txt left in tree.
```

```
## master...origin/master
DU i1.txt # conflicted file
```

Stage the file and commit

``` bash
git add i1.txt
git commit -m "i1 mod"
```

The modified file will now show in the switched branch working directory but will remain unchanged in its original branch

###### Merging Changes During Checkout

To merge modification during checkout

``` bash
git checkout -m 'BRANCH'
```

This is equivalent to executing 

``` bash
git stash
git checkout 'BRANCH'
git stash pop
```

Proceed to stage the changes and commit

##### Committing in a Wrong Branch

1. Copy the revision over to the correct branch
2. Delete the revision from the wrong branch

`git cherry-pick` is a powerful command used in Git to apply the changes introduced by specific commits from one branch into your current branch. This is particularly useful when you want to include individual commits from another branch without merging the entire branch.

To 'copy' the last commit of a branch to the current branch. Hashes and refs also work

``` bash
git cherry-pick 'BRANCH'
```

If you encounter issues or change your mind, you can abort the cherry-pick process:

```
git cherry-pick --abort
```

Go back to the wrong branch and reset it to the state of the previous commit.

``` bash
git reset --hard HEAD~
```

##### Deleting Local Branches

Branch A is `merged` into branch B if all revisions in A are also included in B.

To print the branch names merged in current branch

``` bash
git branch --merged
```

Print branches not merged in current branch

``` bash
git branch --no-merged
```

To delete a branch that is merged in current branch

``` bash
git branch -d 'BRANCH'
```

To delete a branch that is not merged

``` bash
git branch -D 'BRANCH'
```

Deleting branches simply removes the pointers. It does not modify the git database, all revisions are left intact. Branches can be retrieved using `reflog`

##### Renaming Branches

```bash
git branch -m 'OLD_BRANCH_NAME' 'NEW_BRANCH_NAME'
```

The `-M` option is a shorthand for `--move` which forces the rename, even if the new branch name already exists. This is very useful for cleaning up branch names to make them more meaningful or to correct typos.

```
* 9e5bbea (feature) three.txt
* 787424b two
* 9e4fdf0 one
| * 6f1ce08 (info) i3
| * d7ed465 i2
| * 5ffefdc i1
|/
* e12897c (HEAD -> master) m3
```

```bash
git branch -M 'feature' 'info'
```

> This will overwrite existing branch `BRANCH2` revisions with that of `BRANCH1`

```
* 9e5bbea (info) three.txt
* 787424b two
* 9e4fdf0 one
|/
* e12897c (HEAD -> master) m3
```

##### Checking out a File from a Different Branch

To retrieve the version of a specific file from a different branch (in this case, `master`) and apply it to your current working branch without changing branches entirely.

```bash
git checkout [REVISION] -- [filename]
```

Arbitrary revision identifier can be passed: `HEAD`, ancestor refs, stash refs, remote tracking branches, reflog.

`--`: Used to disambiguate command options from paths.

```bash
git checkout 'BRANCH' -- 'FILE'
```

Arbitrary versions of files stored in a database can also be displayed on your screen

``` bash
git show 'REVISION':'FILE'
```

##### Switching Branches in a Bare Repository

A bare repository doesn’t allow for direct edits or commits to files, as there is no working directory. Bare repositories are often used as remote repositories for collaboration, where multiple users push and pull changes. Example: Github

Clone a repository as a bare repository

```bash
git clone --bare 'SOURCE REPO' 'TARGET REPO'
```

Change the current branch in a bare repository by changing the ref in the HEADS file

``` bash
git symbolic-ref HEAD refs/heads/'BRANCH'
```

To clone a repository to a specific branch

```bash
git clone -b 'BRANCH' 'SOURCE REPO' 'TARGET REPO-non-bare'
git clone --bare 'BRANCH' 'SOURCE REPO' 'TARGET REPO-bare'
```

#### Rebasing Branches

Rebasing: Converting divergent branches into linear history. Skips merged commits by def

```bash
git rebase 'DEST' 'SRC'
```

##### Rebasing Divergent Branches

```
[root@localhost 07-01]$ git log --oneline --graph --decorate --all
* 622517e (HEAD -> master) m5
* 0baac4d m4
| * 85dab21 (feature) f3
| * 5a23ffe f2
| * f20f3ca f1
|/
* e58c55f m3
* 6c0ac29 m2
* 17cdc7a m1
```

The `rebase` command takes two arguments. Defaults to HEAD if not specified

```bash
git rebase 'BASE POINT' 'BRANCH TO REBASE'
```

Checkout to the `feature` branch and rebase it onto the master branch

```bash
git rebase master
```

In this case the current branch does not matter

```bash
git rebase master feature
```

Revisions cannot be moved from one place to another. The `feature` revisions have new hashes after rebasing. The original revisions still remain in the database as `dangling` revisions.

```
* 9f9a3ab (HEAD -> feature) f3
* da0ad27 f2
* 02b5671 f1
* 622517e (master) m5
* 0baac4d m4
* e58c55f m3
* 6c0ac29 m2
* 17cdc7a m1
```

The same can be achieved by

```bash
git branch -D feature
git checkout -b feature 'ORIGINAL_HASH'
```

Or 

```bash
git checkout -B feature 'ORIGINAL_HASH'
```

The operation can be undone by pointing the `feature` branch back to the original f3 commit, which was the tip of the `feature` branch. The hash of a commit containing the f3 commit message needs to be retrieved.

```bash
git log --walk-reflogs  --grep=f3 --pretty="%h %s %gs"
```

Placeholders
`%h`: hash
`%s`: comments
`%cd`: commit date
`%gs`: reflog message

```
9f9a3ab f3 rebase (finish): returning to refs/heads/feature
9f9a3ab f3 rebase (pick): f3
85dab21 f3 checkout: moving from master to feature
85dab21 f3 clone: from /root/git-recipies/06-01
```

Move the `feature` pointer back to the original hash

```bash
git reset --hard 85dab21
```

##### Manually Rebasing Divergent Branches

> This is for educational purposes only to show how rebasing work

```
* 3b62cbd (master) m5
* 908cfe3 m4
| * 85dab21 (HEAD -> feature) f2
| * 5a23ffe f1
|/
* 6c0ac29 m2
* 17cdc7a m1
```

Generate patches for `feature` revisions

```bash
git format-patch --ignore-if-in-upstrem master
```

```
0001-f1.patch
0002-f2.patch
0003-f3.patch
```

Enter detached HEAD state with HEAD pointing to the same revision as the master branch

```bash
git checkout `git rev-parse master`
```

`git rev-parse master`: Outputs the full SHA-1 hash of the current `master` branch's HEAD commit.

`* 3b62cbd (HEAD, master)`

Apply patches

```bash
git am.*patch
```

Move the feature branch to the current revision

```bash
git checkout -B feature
```

```
* 9a760b8 (HEAD -> feature) f2
* 6627454 f1
* 3b62cbd (master) m5
* 908cfe3 m4
* 17cdc7a m1
```

To display a range of commits. Display all revisions that are in `feature` but not in `master`.
This helps discover which set of commits were or will be moved during rebase.

```bash
git log --oneline master..feature
```

###### Using Cherry-pick for Rebase

Enter the detached HEAD state at master

```bash
git checkout `git rev-parse master`
```

Reapply the revision f1-f3 in HEAD 

```bash
git cherry-pick feature~2
git cherry-pick feature~1
git cherry-pick feature
```

Move the feature branch to the current revision

```bash
git checkout -B feature
```

##### Joining Divergent Branches into Linear History

```
* 3b62cbd (master) m5
* 908cfe3 m4
| * 85dab21 (HEAD -> feature) f2
| * f20f3ca f1
|/
* 6c0ac29 m2
* 17cdc7a m1
```

Rebase the `feature` branch onto the `master` branch and vice versa

```bash
git rebase master feature
git rebase feature master
```

```
* 39f5dde (HEAD -> master, feature) f2
* d5e4f8c f1
* 3b62cbd m5
* 908cfe3 m4
* 6c0ac29 m2
* 17cdc7a m1
```

Find common ancestors between branches. Returns the commit hash

```bash
git merge-base 'BRANCH1' 'BRANCH2'
```

List new commits introduced in branches

```bash
git log --oneline \
master feature brave-idea \
^`git merge-base master feature`\
^`git merge-base feature brave-idea`
```

To display the commit hash of a revision only

```bash
git long --format="%h" --grep='COMMIT_MSG' --all
```

##### Partial Rebasing

```
* b3e58c3 (feature) f4
* bdbe54d f3
| * 9ae5dca (brave-idea) b1
| * 1ebd82b b2
|/
* 85dab21 f2
* f20f3ca f1
| * 3b62cbd (HEAD -> master) m5
| * 908cfe3 m4
|/
* e58c55f m2
* 17cdc7a m1
```

Rebase the current branch onto a specific commit or branch (`foo`), but only with the commits that exist after a specific point (`bar`).

```bash
git rebase --onto 'foo' 'bar'
```

**`master`**: This is the new base where the `brave-idea` branch will be moved.
Take the commits in `brave-idea` that are not shared with `feature` and rebase them onto `master`
**`brave-idea`**: This is the branch you’re rebasing, which will be moved onto `master`.

```bash
git rebase --onto master feature brave-idea
```

**`feature`**: This specifies the "starting point," or the commit just before `brave-idea` diverged. Only commits in `brave-idea` that are after `feature` will be rebased.

```
* 0b57b4d (HEAD -> brave-idea) b1
* b40f42f b2
* 3b62cbd (master) m5
* 908cfe3 m4
| * b3e58c3 (feature) f4
| * 85dab21 f3
| * 5a23ffe f2
| * f20f3ca f1
|/
* e58c55f m3
* 17cdc7a m1
```

##### Creating Bulbs for Divergent Branches

```
* 3b62cbd (master) m5
* 908cfe3 m4
| * 85dab21 (HEAD -> feature) f3
| * 5a23ffe f2
| * f20f3ca f1
|/
* e58c55f m3
* 6c0ac29 m2
* 17cdc7a m1
```

Rebase the feature branch onto the master branch

```bash
git rebase master feature
```

Switch to the master branch

```bash
git checkout master
```

Merge the feature branch into master branch

```bash
git merge --no-ff feature
```

```
*   becea3a (HEAD -> master) Merge branch 'feature'
|\
| * 762bb30 (feature) f3
| * 0f757eb f2
| * faa4988 f1
|/
* 3b62cbd m5
* 908cfe3 m4
* e58c55f m3
* 6c0ac29 m2
* 17cdc7a m1
```

##### Preserving Mergers During Rebase

```
*   1a050ee (feature) Merge branch 'brave-idea' into feature
|\
| * f9c0cd9 b1
| * 422b51d b2
|/
* b3e58c3 f3
* 5a23ffe f2
* f20f3ca f1
| * 3b62cbd (HEAD -> master) m5
| * 908cfe3 m4
|/
* e58c55f m2
* 17cdc7a m1
```

```bash
git rebase --rebase-merges master feature
```

```
*   20b2197 (HEAD -> feature) Merge branch 'brave-idea' into feature
|\
| * a9828a8 b1
| * 9ca0cdd b2
|/
* fabcbe2 f3
* 65102b8 f2
* 6b48870 f1
* 3b62cbd (master) m5
* 908cfe3 m4
* 6c0ac29 m2
* 17cdc7a m1
```

#### Modifying the History

Revisions in git are permanent. Cannot be modified. In this case, modification means creating a new revision that resembles the original. 

##### Amending the Most Recent Revision

`git commit --amend`: Allows to 'modify' the most recent revision in the history by creating a new revision and updating the master branch to point to the new revision. The original revision remains dangling in the git database until it is deleted by garbage collection.

Create a file 'lorem.txt' containing text 'lorem' and commit it.

```
* fcd163d (HEAD -> master) lorem.txt
```

To introduce changes to an already committed revision, without creating a new commit

```bash
git commit --amend -m "Lorem Ipsum Dolor"
```

Author and commit names also differ. The file has been modified and has a new hash and commit message.

```
[root@localhost 08-01]$ git log --pretty=fuller
commit 8a663c171637d91b2a6dba8c67e3376d3eeada19 (HEAD -> master)
Author:     Jon DOW <jondow@example.com>
AuthorDate: Fri Nov 15 10:31:22 2024 +0200
Commit:     John Doe <john@example.com>
CommitDate: Fri Nov 15 10:36:08 2024 +0200
    Lorem Ipsum Dolor
```

Both commits, original and amended, are available through `reflog`

```
8a663c1 (HEAD -> master) HEAD@{0}: commit (amend): Lorem Ipsum Dolor
fcd163d HEAD@{1}: commit (initial): lorem.txt
```

Undo the amendment 

```bash
git reset --hard HEAD@{1}
```

##### Removing N-most Recent Revisions

To remove the last 2 revisions from the current branch

```bash
git reset --hard HEAD~2
```

##### Squashing Many Revision Into One Revision

`starting repo`
```
* 649718b (HEAD -> master) d
* 02c86af c
* 86026ed b
* 3144996 a
```

To verify which files are included in the last revision

```bash
git show --name-only HEAD[~N]
```

```
commit 649718b5891ef13f28983818effa8d6495cb45a5 (HEAD -> master)
Author: Jon DOW <jondow@example.com>
Date:   Fri Nov 15 11:05:05 2024 +0200
    d
d.txt
```

Squash the last three commits into a single revision

```bash
git rebase -i HEAD~3
```

`vim editor`
```bash
pick 86026ed b
pick 02c86af c
pick 649718b d
# s, squash = use commit, but meld into previous commit
# f, the same as squash, but this time git will not allow modification of the comment of the resulting revision (the comment of the first revision will be used)
```

Change to

```
pick 86026ed b
fixup 02c86af c
fixup 649718b d
```

`pick 86026ed b`: Picks commit b thus it will appear in the resulting history
`fixup 02c86af c`: Squashes commit c into the previous commit b. Comments cannot be modified. The b comment is retained.
`fixup 649718b d`: Squashes d into the resulting c into b squash. 

This results in a single commit b that incorporates changes from b, c and d - last three commits, as per the rebase command

```
4545033 (HEAD -> master) b
3144996 a
```

```bash
git show --name-only HEAD
```

The updated b commit has the files from the squashed commits c and d

```
commit 45450331930d467079bbc3e3fce458ed7fa5fe78 (HEAD -> master)
Author: Jon DOW <jondow@example.com>
Date:   Fri Nov 15 11:05:05 2024 +0200
    b
b.txt
c.txt
d.txt
```

`reflog`
```
4545033 (HEAD -> master) HEAD@{0}: rebase (finish): returning to refs/heads/master
4545033 (HEAD -> master) HEAD@{1}: rebase (fixup): b
7b107fa HEAD@{2}: rebase (fixup): # This is a combination of 2 commits.
649718b HEAD@{8}: commit: d
```

To undo the operation

```bash
git reset --hard HEAD@{8}
```

##### Splitting One Revision Into Many Revisions

Split the b revision into a separate revision for each file it contains

`start repo`
```
commit b0aa79f3a32fb657c862d326b824289201dd46bd (HEAD -> master)
    b
b.txt
c.txt
d.txt
commit ba303b3f53efd24ccf21ae3b213156ca6ab34f49
    a
a.txt
```

Reset the history to revision a, but **keep working directory changes**: Any changes in your working directory will remain. This is different from a hard reset, which would discard those changes.

```bash
git reset HEAD~
```

The files are now untracked. Separate revisions can be created for each file

`git status -sb`
```
## master
?? b.txt
?? c.txt
?? d.txt
```

##### Reordering Revisions

`starting repo`
```
1e03d54 (HEAD -> master) d
b89df4d c
245fc62 b
7160d75 a
```

Perform interactive rebasing on the last 3 commits

```bash
git rebase -i HEAD~3
```

`vim`
```
pick 245fc62 b
pick b89df4d c
pick 1e03d54 d
```

Rebasing applies the patches according to their order in the editor.  Save and exit

`vim`
```
pick 1e03d54 d
pick 245fc62 b
pick b89df4d c
```

Result of `git log`

```
1955b65 (HEAD -> master) c
374e46d b
a02e2ff d
7160d75 a
```

##### Removing Several Revisions

`starting repo`
```
599e640 (HEAD -> master) f
204bfcf e
5bb37e3 d
18e3882 c
edf6f55 b
6415ab4 a
```

We want to remove commits b, d and f. b is the oldest commit and 5th from top down.

```bash
git rebase -i HEAD~5
```

`vim`
```
pick edf6f55 b
pick 18e3882 c
pick 5bb37e3 d
pick 204bfcf e
pick 599e640 f
```

Remove the entries that are not needed and leave the rest. Save and exit

```
pick 18e3882 c
pick 204bfcf e
```

The b, d and f commits are not gone from the history

```
1381eb6 (HEAD -> master) e
5c2dba0 c
6415ab4 a
```

##### Editing an Old Revision

Each commit introduces a new file $i.txt. We want to include another file in x and add a new commit after x, leave the rest intact.

```
5fe4652 (HEAD -> master) d
9ba0a19 c
08adf4d x
21db689 b
629861d a
```

Perform interactive rebase on the last 3 commits (x is 3rd)

```bash
git rebase -i HEAD~3
```

The original subcommands

`vim`
```
pick 08adf4d x
pick 9ba0a19 c
pick 5fe4652 d
```

Change the subcommand `pick` to `edit` for the corresponding commit

`vim`
```
edit 08adf4d x
pick 9ba0a19 c
pick 5fe4652 d
```

The rebasing process stops at the x commit. Git enters detached HEAD state after patch x

```
Stopped at 08adf4d...  x
----
af0e3b7 (HEAD) x
21db689 b
629861d a
```

Adjust the x commit

```bash
echo y > y.txt
git add y.txt
git commit --amend --no-edit
```

`--no-edit`: If omitted, git will open the editor again 

Create the new z commit. To finish the rebasing

```bash
git rebase --continue
```

```
3749f14 (HEAD -> master) d
bb75324 c
790b2bc z
af0e3b7 x
21db689 b
629861d a
```

To abort paused rebasing

```bash
git rebase --abort
```

##### Reverting Revisions

`starting repo`
```
39bb62a (HEAD -> master) c
e916eae b
7b6e0c7 a
```

Undo the changes introduced by the b commit

```bash
git revert --no-edit HEAD~
```

`--no-edit`: Sets the comment of the new revision to "Revert "revision comment""

A new commit Revert "b" is created. The file b.txt is now gone.

```
457748a (HEAD -> master) Revert "b"
39bb62a c
e916eae b
7b6e0c7 a
```

##### Revering Merge Commit Revisions

The f1-3 commits were part of a `feature` branch that was merged into `master` and deleted. The changes introduced by `feature` must be undone.

`starting repo`
```
* 238b5e2 (HEAD -> master) m6
*   10e47b2 Merge branch 'feature'
|\
| * 99cde1c f3
| * d1ef3b3 f2
| * b22eeac f1
* | b96d664 m5
* | cf9feb7 m4
|/
* 1f45371 m3
* 26f731a m2
* 5458c21 m1
```

The merge commit "10e47b2 Merge branch 'feature'" has two parents - 1. m5 and 2. f3.

Revert the "Merge branch 'feature'"

```bash
git revert --no-edit -m 1 HEAD~
```

`-m 1`: Keep the history stored under parent 1 (master branch) of the merge commit
 
```
[master 51f41c1] Revert "Merge branch 'feature'"
 Date: Fri Nov 15 20:24:57 2024 +0200
 3 files changed, 3 deletions(-)
 delete mode 100644 f1.txt
 delete mode 100644 f2.txt
 delete mode 100644 f3.txt
```

The revert commit is created and the `feature` files f1-3.txt are no longer in the working directory. Only the m1-6 files of the `master` branch parent are kept.

##### Cherry-picking Revisions

Get revision m4 from `master` over to `feature`

`starting repo`
```
* 3b62cbd (master) m5
* 908cfe3 m4
| * 85dab21 (HEAD -> feature) f3
| * 5a23ffe f2
| * f20f3ca f1
|/
* e58c55f m3
* 6c0ac29 m2
* 17cdc7a m1
```

From `feature` get 1st parent of `master` (m4)

```bash
git cherry-pick master~1
```

##### Squashing a Branch

Squash the f1-3 commits of `feature` branch into a single revision and add it in `master`

`starting repo`
```
* 3b62cbd (master) m5
* 908cfe3 m4
| * 85dab21 (HEAD -> feature) f3
| * 5a23ffe f2
| * f20f3ca f1
|/
* e58c55f m3
* 6c0ac29 m2
* 17cdc7a m1
```

From `master`

```bash
git merge --squashed feature
```

The working directory now contains all changes from the feature branch and they are staged. Commit

Files f1-f3.txt are now in `master`

##### Re-using a Reverted Branch

Branch `foo-bar` was merged into the `master` and then the merge was reverted. `master` contains files \[a-d].txt 

`starting repo`
```
* fe392fd (HEAD -> master) d
* bbe9afb Revert "Merge branch 'foo-bar'"
* 3d862d3 c
*   d387827 Merge branch 'foo-bar'
|\
| * 1b184c4 (foo-bar) y
| * 0b3beb4 x
|/
* 6a2da8c b
* a946b2a a
```

`foo-bar` revisions can be listed with

```bash
git log --oneline foo-bar~2..foo-bar
```

Create the patches for the `foo-bar` revisions

```bash
git format-patch foor-bar~2..foo-bar
```

Create a temporary branch 

```bash
git checkout -b foo-bar-tmp
```

Apply the patches in it and remove them afterwards

```bash
git am *.patch
rm -rf *.patch
```

Rename the temporary branch `foo-bar-tmp` to the original name

```bash
git branch -M foo-bar-tmp foo-bar
```

Checkout to the master and merge the `foo-bar` branch again

```bash
git checkout master
git merge --no-ff -m "2nd merge of 'foo-bar'" foo-bar
```

```
*   ceb20f9 (HEAD -> master) 2nd merge of 'foo-bar'
|\
| * 3926806 (foo-bar) y
| * 162c913 x
|/
* fe392fd d
* bbe9afb Revert "Merge branch 'foo-bar'"
* 3d862d3 c
*   d387827 Merge branch 'foo-bar'
|\
| * 1b184c4 y
| * 0b3beb4 x
|/
* 6a2da8c b
* a946b2a a
```

#### Resolving Conflicts

##### Creating Conflicting Changes in Text Files

The three repos contain a file numbers.txt with different line 2, indicated by Numbers: __

`starting repo`
```
* 57942ef (fr) Numbers: deux
| * d0ece6d (HEAD -> en) Numbers: two
|/
* 8a791cd (master) Numbers: 1 2 3
```

```
[root@localhost 09-03]$ git show en:numbers.txt
1
two
3
[root@localhost 09-03]$ git show fr:numbers.txt
1
deux
3
```
##### Resolving Textual Conflict After Merging

Resolving merge conflicts:
1. Editing a file
2. Staging the file with `git add [FILENAME]`
3. Finishing the merge with `git commit --no-edit`

Merge the `fr` branch into `en`

```bash
git merge fr
```

```
Auto-merging numbers.txt
CONFLICT (content): Merge conflict in numbers.txt
Automatic merge failed; fix conflicts and then commit the result.
```

```
[root@localhost 09-02]$ git status -sb
## en
UU numbers.txt 
```

`UU`: "Updated but Unmerged". Conflicted files. 

Merging is paused until conflict is resolved

The contents of the numbers.txt file after the merge, indicating a merge conflict

```
1
<<<<<<< HEAD
two
=======
deux
>>>>>>> fr
3
```

- **`<<<<<<< HEAD`**:
    
    - Indicates the start of the conflicting section.
    - The content below it (`two`) is the version from the **current branch** (the branch you were on when you ran the merge).

- **`=======`**:
    
    - Divides the two conflicting changes.

- **`>>>>>>> fr`**:
    
    - Marks the end of the conflict.
    - The content above it (`deux`) is the version from the **branch being merged in** (in this case, the `fr` branch).

- **Lines Outside the Conflict Markers**:
    
    - Lines like `1` and `3` are unchanged and appear in both versions.

Resolve the conflict by manually editing the file to the proper version. For example:

```
1
two - deux
3
```

Stage the file and resume the paused merge

```bash
git add numbers.txt
git commit --no-edit
```

`git commit --abort`: To abort the merge

To restore versions

```bash
git checkout { --ours | --theirs } 'FILENAME'
```

`--ours`: The file version as in the current branch. Equivalent to `git checkout en numbers.txt`
`--theirs`: The version as from the branch that is being merged

The contents of the file is restored but the conflict is still not resolved, status remains `UU`

```bash
git checkout merge 'FILENAME'
```

Conflicts are now denoted with <<< ours and >>> theirs. 

```
1
<<<<<<< ours
two
=======
deux
>>>>>>> theirs
3
```

To show the original line before the branches `en` and `fr` diverged

```bash
git checkout --conflict=diff3 numbers.txt
```

```
1
<<<<<<< ours
two
||||||| base
2
=======
deux
>>>>>>> theirs
3
```

##### Resolving Textual Conflict After Rebasing

Resolving merge conflicts:
1. Editing a file
2. Staging the file with `git add [FILENAME]`
3. Finishing the merge with `git rebase --continue`

Rebase the `en` onto `fr` branch

```bash
git rebase fr en
```

Rebasing fails and is paused. The conflict needs to be resolved before resuming

```
CONFLICT (content): Merge conflict in numbers.txt
error: could not apply d0ece6d... Numbers: two
```

```
1
<<<<<<< HEAD
deux
=======
two
>>>>>>> d0ece6d (Numbers: two)
3
```

```
* (no branch, rebasing en)
  en
  fr
  master
```

Edit the file to the correct version

```
1
deux - two
3
```

Stage the file and resume the rebase

```bash
git add numbers.txt
git rebase --continue
```

`git show en:numbers.txt`
```
1
deux - two
3
```

The meaning of `--ours` and `--theirs` is reversed is this case because rebasing starts with the checkout of the latest commit of the base branch `fr`

##### Resolving a Binary Conflict During Merging

Create an empty commit with empty message

```bash
git commit --allow-empty --allow-empty-message -m ' '
```

`master`: Empty commit, empty message
`a`: Contains a file `picture.jpg` of an exterior
`b`: Contains a file `picture.jpg` of an interior

`starting repo`
```
* a477522 (b) Interior
| * ec62d5b (HEAD -> a) Exterior
|/
* 0f1d6a2 (master)
```

Merge `b` into `a`

```
warning: Cannot merge binary files: exterior.jpg (HEAD vs. b)
Auto-merging exterior.jpg
CONFLICT (add/add): Merge conflict in exterior.jpg
Automatic merge failed; fix conflicts and then commit the result.
```

`git status -sb`
```
## a
AA picture.jpg
```

Checkout to the proper file version

```bash
git checkout [--ours | --theirs] 'FILENAME'
```

or 

```bash
git checkout a 'FILENAME'
git checkout b 'FILENAME'
```

Resolve the conflict an finish the merge

```bash
git add 'FILENAME'
git commit --no-edit
```

##### Resolving a Binary Conflict During Rebasing

`a` and `b` contain a file `picture.jpg`

```
* 4d16eec (HEAD, b) Exterior
| * f638596 (a) Interior
|/
* 5872cbf (master)
```

Rebase `a` onto `b`

```bash
git rebase b a
```

```
warning: Cannot merge binary files: picture.jpg (HEAD vs. f638596 (Interior))
Auto-merging picture.jpg
CONFLICT (add/add): Merge conflict in picture.jpg
error: could not apply f638596... Interior
```

`git status -sb`
```
## HEAD (no branch)
AA picture.jpg
```

Restore the proper file version

```bash
git checkout --theirs picture.jpg (branch a)
```

Stage and resume

```bash
git add picture.jpg
git rebase --continue
```

##### Forcing a Binary Mode During a Merge

Create a file `.gitattributes` in the work directory containing the line `filename binary`. Likewise, a binary file can be treated as text with `filename text`

`.gitattributes`
```
numbers.txt binary
```

Merging now produces

```
warning: Cannot merge binary files: numbers.txt (HEAD vs. fr)
Auto-merging numbers.txt
CONFLICT (content): Merge conflict in numbers.txt
```

Filename can be treated as a pattern. `*.txt binary`, `bindir/ text`, etc. Attributes can be applied to every pattern as well

`*.txt text -merge eol=crlf`

##### Co-working with a Central Repository

Structure:
1. `shared-repo`: Central bare repository. Create with `git init --bare shared-repo`
2. `johns-repo`: Non-bare user repo
3. `sarahs-repo`: Non-bare user repo

Configure local user settings

```bash
git config --local user.name john
git config --local user.email john@example.com
```

`johns-repo`
```
* f60d979 (HEAD -> master) a3
* ccf9c16 a2
* 9d1a849 a1
```

Add the remote repo to the config

```bash
git remote add origin ../shared-repo
```

Push the local `master` branch to the remote repository, aliased as `origin`

```bash
git push -u origin master
```

`-u (or --set-upstream)`:  Links the local `master` branch to the remote `origin/master`. 

`shared-repo/refs/heads/master`
```
f60d97971985f48a4f13230f6f461cec86eeacef
```

`johns-repo/.git/refs`
```
└── refs
│       ├── heads
│       │   └── master     # f60d97971985f48a4f13230f6f461cec86eeacef
│       └── remotes
│           └── origin
│               └── master # f60d97971985f48a4f13230f6f461cec86eeacef
```

John's repo contains a local tracking branch `master` and a remote-tracking branch `origin/master`

`git branch -a -vv`
```
* master                f60d979 [origin/master] a3
  remotes/origin/master f60d979 a3
```

Display info about the remote repository, aliased as `origin`

```bash
git remote show origin
```

```
* remote origin
  Fetch URL: /root/git-recipies/10-02/shared-repo
  Push  URL: /root/git-recipies/10-02/shared-repo
  HEAD branch: master
  Remote branch:
    master tracked
  Local branch configured for 'git pull':
    master merges with remote master
  Local ref configured for 'git push':
    master pushes to master (local out of date)
```

Clone the central repo for user `sarah`

```bash
git clone ../shared-repo
```

Cloning the repo sets up the remote-tracking branches automatically. To download the revisions from the remote `master` branch

```bash
git pull origin master
```

To push own `master` revisions to the shared repo

```bash
git push origin master
```

If the local repo is missing revisions from the remote repo, pushing new revisions will fail

```
 ! [rejected]        master -> master (fetch first)
error: failed to push some refs to '../shared-repo/
```

The local repo must first be synced with the remote repo before pushing commits

The local `master` branch contains three revisions that are not included in `origin/master` tracking branch

`git status -sb`
```
## master...origin/master [ahead 3]
```
##### Generating (n-1) Merge Commits for One Commit

n: Number of developers

##### Keeping the History Linear

`shared-repo`
```
* 82b8001 (HEAD -> master) John j3
* ef68e3b John j2
* 8010ad8 John j1
* 1081fd8 John i2
* 6e00890 John i1
```

`marks-repo`
```
* 753128c (HEAD -> master) Mark m2
* 9269a6e Mark m1
* 1081fd8 (origin/master, origin/HEAD) John i2
* 6e00890 John i1
```

In case of divergent branches, the pushing user must first update his master branch 

```bash
git fetch origin
```

`marks-repo`
```
* 753128c (HEAD -> master) Mark m2
* 9269a6e Mark m1
| * 82b8001 (origin/master, origin/HEAD) John j3
| * ef68e3b John j2
| * 8010ad8 John j1
|/
* 1081fd8 John i2
* 6e00890 John i1
```

`git status -sb`
```
## master...origin/master [ahead 2, behind 3]
```

Now Mark knows that his master and `origin/master` branches have diverged.

`ahead 2`: Master branch has 2 revisions that are not included in `origin/master`
`behind 3`: Local master branch is missing 3 revisions from the `origin/master`

To keep the history linear, Mark needs to rebase his `master` branch on top of the `origin/master`, fetched from the remote repo

```bash
git rebase origin/master master
```

The history is linear

`git log --oneline --graph --decorate --all`
```
* cc3480e (HEAD -> master, origin/master) Mark m2
* 68251e3 Mark m1
* 82b8001 John j3
* ef68e3b John j2
* 8010ad8 John j1
* 1081fd8 John i2
* 6e00890 John i1
```

`git pull` == `git fetch && git merge`
`git pull -r` == `git fetch && git rebase`

##### Accessing Remote Branches

`git status -sb`, `git branch -a -vv` only work for local branches. They do not access remote branches. To catch up with remote branches, they must first be fetched.

To ascertain if a branch is synced

```bash
git fetch
git status -sb
```
##### Tracking Branches

The `fetch` command fetches all remote branches and stores them as remote-tracking branches.

Ways to set up branch tracking

```bash
git branch --set-upstream-to=origin/'REMOTE BRANCH' 'LOCAL-TRACKING BRANCH'
```

```bash
git push -u origin master
```

```bash
git config branch...
```

or Editing the `.git/config` file

To convert a local-tracking branch into an ordinary local branch

``` bash
git config --unset branch.'BRANCH'.remote
git config --unset branch.'BRANCH'.merge
```

##### Working with Remote Branches

To send a local branch to a remote branch under a different name, converting it to a local-tracking branch

```bash
git push -u 'REMOTE NAME' 'LOCAL-BRANCH-NAME':'REMOTE-BRANCH-NAME
```

Check if a branch can be safely removed

```bash
git branch --merged
```

To remove a remote branch. Deletes the remote-tracking branch. Doesn't delete the local-tracking branch in local repo.

```bash
git push origin :'REMOTE-BRANCH'
```

To remove stale remote tracking branches

```bash
git remote prune
```

##### Using Remote Branches for Contributions

##### 
#### The Reset Command

A repository consists of:
* The working directory
* The staging area
* The database

The .git/HEAD file points to one of the commits stored in the database

In any given point in time, the repository operates on three different snapshots:

1. The working directory
2. The Staging area
3. The snapshot stored in the revision pointed by HEAD

When the repository is clean all three snapshots are identical. 

The two-letter code `XY` printed by the `git status -sb` command tells the difference 
between the three snapshots

`X`: Tells the difference between the third snapshot HEAD and the second snapshot staging area
`Y`: Tells the difference between the second snapshot (staging area) and the first snapshot (working directory)

` M`: The file in the working directory differs from that stored in the staging area, but staging area is identical to HEAD
`M `: The file in the working directory is identical to the file in the staging area. The file stored in the staging area differs from that stored in HEAD

After committing, all three snapshots are synchronized

`git reset [OPTION]`:
* `--soft`: The command influences only HEAD
* `--mixed`: Default. Applies to HEAD and staging area
* `--hard`: Applies to all three snapshots

`git reset --hard [REVISION]`:

* Modification of HEAD: Moves the branch pointer (HEAD) to a specific commit.
* Modification of the staging area: store snapshot that is now stored in HEAD and store it in the staging area
* Resets the branch pointer, staging area (index), and working directory to match the specified revision. All uncommitted changes are discarded.

> **This operation is destructive!**: All changes in your working directory and staging area are permanently lost unless you have them saved elsewhere (e.g., stashed or committed).

`git reset --mixed [REVISION]`

* Moves the branch pointer (HEAD) to the specified revision 
* Updates the staging area (index) to match the specified commit, but it does not affect your working directory.  Unstages any changes.
* Any changes in your working directory remain as uncommitted changes.

`git reset --soft [REVISION]`

* Moves the branch pointer (HEAD) to the specified revision
* Does not change the staging area (index)
* Does not change the working directory

**Key Effect**: It "undos" commits while keeping changes staged for recommit.

When to Use `--soft`

- **Undo a commit but keep changes staged**: Use `--soft` when you realize you committed too soon and want to make further changes or adjust your commit message.
- **Move commits between branches**: Reset to a point in history, and then reapply the staged changes to another branch.

Undo the last commit but keep changes staged:

```bash
git reset --soft HEAD~1
```
#### Mapping Names

To remap the author name of all commits by `name@mail.com` to a new name

`repo/.mailmap`
```
NAME <name@mail.com>
```

To change both name and email of commits by certain author

```
NEW_NAME <new_email> <original_email>
```

#### Git Directory Structure

##### `.git/`

`config`: The main repo configuration file
`HEAD`: Reference to the current branch
`refs/heads`: It stores **references to the latest commit hashes** for all local branches in the repository.
`refs/remotes`: Stores **references to remote-tracking branches**, which are local representations of the branches from a remote repository. Holds files for each branch on that remote. Each file stores the commit hash that the corresponding remote branch points to. On the remote repository, these branches are stored in `refs/heads`. In order not to cause collisions with our local `refs/heads` directory, they are stored in their own `refs/remotes` dir.
`refs/tags`

`.git/refs/`
```
 refs
    ├── heads   # refs to latest commit hashes for all local branches
    ├── remotes # refs to remote-tracking branches
    │   └── origin # name alias for the remote repository
    │       ├── clean # remote tracking branch
    │       ├── doc
    │       ├── legacy
    │       ├── master
    └── tags
        ├── 0.0.1
```

- **Remote-Tracking Branches**:
    
    - These are not local branches but references to the state of branches in the remote repository.
    - For example, `origin/main` represents the state of the `main` branch in the remote `origin` at the last fetch or pull.
- **Updates**:
    
    - Remote-tracking branches are updated when you fetch or pull changes from the remote repository.
    - For example, after `git fetch origin`, `.git/refs/remotes/origin/main` will point to the latest commit on the `main` branch in the remote.
- **No Direct Edits**:
    
    - You typically don't interact with `.git/refs/remotes` directly. Instead, you work with remote-tracking branches via commands like `git fetch`, `git pull`, and `git push`.

Send revisions from local to remote

```bash
git push 'REMOTE REPO' 'LOCAL BRANCH'
```

Download revisions from remote to local and merge them into the appropriate local branch

```bash
git pull 'REMOTE REPO' 'LOCAL BRANCH'
```

```bash
git pull origin master
```

- **Fetches Updates from `origin/master`**:
    
    - The `git pull` command combines two operations:
        - `git fetch`: Downloads the latest changes from the `master` branch on the `origin` remote repository.
        - `git merge`: Merges the fetched changes into your current branch (if there are new commits on the remote).
- **Updates Your Local `master` Branch**:
    
    - If you are currently on the `master` branch, this command integrates changes from `origin/master` into your local `master`.

**Review Changes Before Merging**:

- Use `git fetch` and `git log` to review updates before merging:

``` bash
git fetch origin 
git log ..origin/master` 
```

##### Rewriting History with `git push -f`

Squash several commits into one

`git log --oneline`
```
93aec7b (HEAD -> new-web-interface, origin/new-web-interface) c
ef94485 b
42ce70c a
```

```bash
git rebase -i HEAD~3
```

```
reword 42ce70c
fixup ef94485
fixup 42ce70c
```

Save, put a comment and exit

Republish the revisions

```bash
git push -f origin new-web-interface
```

`-f`: By default `git push` succeeds only for fast-forwarding updates. This forces git to rewrite history
#### Git Fetch

It's a safe way to update your local view of the remote repository. Use `git fetch` regularly to keep your repository up-to-date without merging or modifying your code.

```bash
git fetch [remote] [refspec]
```

- **Downloads Remote Changes**:
    - Updates your remote-tracking branches (e.g., `origin/main`) with the latest state of the corresponding branches in the remote repository.
    - It does not affect your local branches or files.

- **No Automatic Merge**:
    - Unlike `git pull`, `git fetch` does not merge the changes into your current branch. You decide when and how to incorporate the fetched changes.

`-p`: Prunes non-existent remote-tracking branches

#### Git Diff

`diff` compares the working directory and the staging area. Compares lines by default

```bash
git diff ['COMMIT']
```

```
diff --git a/numbers.txt b/numbers.txt
index c9e9e05..f1f2be8 100644
--- a/numbers.txt
+++ b/numbers.txt
@@ -1,10 +1,8 @@
 one
 two
 three
-four
-five
-six
-seven
+foo
+bar
 eight
 nine
 ten
```

`@@ -<old-range> +<new-range> @@` `-+starting line, span`

The `@@` delimiters indicate a chunk of changes. Git splits the file into manageable sections when displaying differences, each chunk is preceded by a header like this.

`@@ -1,10 +1,8 @@`
The changes start at line 1 in both the old and new files.
10 lines from the old file were reduced to 8 lines in the new file due to modifications.

Compare branches

```bash
git diff master 'BRANCH' [-- 'FILE']
```

```bash
git diff --word-diff
```

To find the revisions in which a given file was modified

```bash
git log --name-only master 'BRANCH' [-- 'FILE']
```

To compare the files in the staging area to the file stored in HEAD

```bash
git diff --staged
```

If the changes have been committed, compare the latest version to the previous

``` bash
git diff --unified=1 HEAD~ HEAD
```

```
diff --git a/numbers.txt b/numbers.txt
index c9e9e05..f1f2be8 100644
--- a/numbers.txt
+++ b/numbers.txt
@@ -3,6 +3,4 @@ two
 three
-four
-five
-six
-seven
+foo
+bar
 eight
```

`--unified=1`: number of lines preceding and following the changed content
`--check`: Outputs changes that can be regarded as problem with white chars. 
#### Line Endings and Conversions

`LF`: encoded as `\n` strings (Linux)
`CRLF`: encoded as `\r\n` (Windows)

To verify line endings in a file

```bash
hexdump -c 'FILE'
```

To commit without line conversion

```bash
git config --local core.autocrlf false
```

>If `core.autocrlf` is set to `true` line endings will be converted automatically LF=>CRLF at checkout. The staging area, however, will still use the original line endings.

To recreate all files in the working directory and in the staging area to match the line endings in the HEAD revision:

Remove all tracked files

```bash
git ls-files | xargs rm
```

```
[root@gitlab 13-03]$ git status -sb
## master...origin/master
 D linux/abcd.txt
 D mixed/abcd.txt
 D windows/abcd.txt
```

Remove the staging area

```bash
rm .git/index
```

```
[root@gitlab 13-03]$ git status -sb
## master...origin/master
D  linux/abcd.txt
D  mixed/abcd.txt
D  windows/abcd.txt
```

Turn off conversion of new lines

```bash
git config --local auto.crlf false
```

Recreate the working directory and staging area

```bash
git reset --hard
```

Now the staging area and the working directory contain the files exactly as they were stored in the HEAD revision. Line endings are as originally created

##### Define Line Endings for Individual Files and Directories

Guarantee that cloning the repository will have the exact line endings, regardless of user settings

`.gitattributes`
```
* eol=lf
windows/* eol=crlf
mixed/* -text
```

`eol=lf`: Forces git to always check checkout all files using LF line endings. 
`eol=crlf`: Exception to the previous rule
`-text`: Turns off all conversion of line endings in the specified directory

#### Ignoring Automatically Generated Files

Ignore files in the `tmp/` directory and its subdirectories, and files with `.abc` extension

`.gitignore`
```
/tmp/
.abc
```

Such files will not show as untracked in git status

Three types of settings

`.gitingore`: File committed into the directory. Affects only the particular repository
Global `.gitingore`: This file resides in your home directory. Affects all repositories. It is a private file and is not committed into the repository.
`.git/info/exclude`: This file is stored in `.git/` directory. It is a private file, not shared with others. Affects only one repository: the one that contains the file

#### Tags

`.git/refs/tags`: Tags are stored here

Git allows to label arbitrary revisions with two types of tags:
* Annotated: Stored in the repo database as objects. Contain:
	* Author
	* Date of tag creation
	* Comment
	* SHA-1 of the tagged revision
* Lightweight: Contain just the SHA-1 of the tagged revision

Create an annotated tag

```bash
git tag -a 'VERSION' -m 'MESSAGE'
```

Create a lightweight tag

```bash
git tag 'TAG_NAME'
```

To remove a tag

```bash
git tag -d 'TAG'
```

`.git/refs/tags/` 
```
temp-version  
v1.2.3
```

To list all tags

```bash
git tag
```

To show tag info

```bash
git show -s 'TAG'
```

To check the most recent annotated tag

```bash
git describe
```

To publish tags

```bash
git push --tags
```

To sync tags between local and remote repo

```bash
git fetch --tags
```

To delete a remote tag

```bash
git push origin :remote-branch-to-delete
git push origin :refs/tags/remote-tag-to-delete
```
#### Export and Archive Git Projects

Create a `.gitattributes` file with a list of files or dirs to exclude from archives

`.gitattributes`
```
<FILE_OR_DIR> export-ignore
```

Tag the revision

```bash
git tag -a ["VERSION"] -m 'VERSION'
```

To archive a project with a specific revision

```bash
git archive --format=zip --output="ARCHIVE.zip" ["VERSION"]
```

