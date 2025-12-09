
#### Merging Branches

Merge types
* A fast-forward
* Merging of two diverged branches
* Merging of multiple diverged branches

> The current branch is the branch we merge into, the branch passed to the `merge` command is the branch to be merged in.

`start repo`
```
* 85dab21 (HEAD -> feature) f3
* 5a23ffe f2
* f20f3ca f1
* e58c55f (master) m3
* 6c0ac29 m2
* 17cdc7a m1
```

In this case, the master branch is fully merged in the feature branch

##### Fast-forwarding Branches

To merge a branch into the current branch

``` bash
git merge 'BRANCH'
```

```
Updating e58c55f..85dab21
Fast-forward
 f1.txt | 1 +
 f2.txt | 1 +
 f3.txt | 1 +
 3 files changed, 3 insertions(+)
 create mode 100644 f1.txt
 create mode 100644 f2.txt
 create mode 100644 f3.txt
```

The merge command moves the `master` pointer to the place referenced by the `feature` pointer. This is called `fast-forwarding`. No new commits are created

```
* 85dab21 (HEAD -> master, feature) f3
* 5a23ffe f2
* f20f3ca f1
* e58c55f m3
* 6c0ac29 m2
* 17cdc7a m1
```

###### Undoing Fast-forward

Using `reflog`

```
85dab21 (HEAD -> master, feature) HEAD@{0}: merge feature: Fast-forward
e58c55f HEAD@{1}: checkout: moving from feature to master
```

Undo the merge

```bash
git reset --hard HEAD@{1}
```

##### Diverged Branches

The commits `m5` and `m4` were created after `f2` and `f2`

```
* cd12fef (HEAD -> master) m5
* 2fab539 m4
| * 85dab21 (feature) f2
| * f20f3ca f1
|/
* 6c0ac29 m2
* 17cdc7a m1
```

Fast-forward merge of diverging branches is not possible

##### Merging Diverged Branches

In the case where `fast-forward` merge is not possible, the merge command creates an additional revision, called `merge commit`. It contains more than one parent

> The current branch we merged into becomes the first parent of the merge commit. The tip of the branch that is merged in becomes the second parent of the merge commit.

To merge two diverging branches

```bash
git merge 'CURRENT BRANCH' 'BRANCH TO MERGE'
```

```
*   a0f4071 (HEAD -> master) Merge branch 'feature'
|\
| * 85dab21 (feature) f2
| * f20f3ca f1
* | cd12fef m4
* | 2fab539 m3
|/
* e58c55f m2
* 17cdc7a m1
```

Print merged only commits

```bash
git log --merges
```

```
commit a0f40711a2b9142e9ecc6ae5ea5e7ac03809f997 (HEAD -> master)
Merge: cd12fef 85dab2
```

To pick n-th parent for a merge commit

```bash
git log --oneline 'BRANCH'^N
```

To undo the merge

``` bash
git reset --hard BRANCH^
```

##### Avoiding Fast-forward Merge

To force the creation of a merge commit even if the merge can be done as a fast-forward

```bash
git merge --no-ff 'BRANCH'
```

```
*   b81de59 (HEAD -> master) Merge branch 'feature'
|\
| * 85dab21 (feature) f3
| * 5a23ffe f2
| * f20f3ca f1
|/
* e58c55f m3
* 6c0ac29 m2
* 17cdc7a m1
```

Now `feature` is merged into the `master` but `feature` revisions are still related to each other and the operation can be easily reverted.

##### Diverging Multiple Branches

```
* 3d331fa (HEAD -> d) d2
* 05f0828 d1
| * ca633a8 (c) c2
| * c1473f0 c1
|/
| * 9c35240 (b) b2
| * e87a20d b1
|/
| * 75fce7c (a) a2
| * 0e106fe a1
|/
| * 85dab21 (feature) f2
| * f20f3ca f1
|/
* e58c55f (master) m2
* 17cdc7a m1
```

##### Merging Multiple Branches

```bash
git merge 'BRANCH1' 'BRANCH2' 'BRANCH_N'
```

The merge commit

```
*---.   be7c4ef (HEAD -> master) Merge branches 'a', 'b', 'c' and 'd'
|\ \ \
| | | * 3d331fa (d) d2
| | | * 05f0828 d1
| | * | ca633a8 (c) c2
| | * | c1473f0 c1
| | |/
| * | 9c35240 (b) b2
| * | e87a20d b1
| |/
* | 75fce7c (a) a2
* | 0e106fe a1
* | 14acd5f delete a
* | 7112b4d a
|/

```

Each merged branch is now a parent to the master branch

```bash
git log --onelog master^N
```

The order of parent branches depends on the order of the branch names passed to the `merge` command. (a): 1, (b): 2, (c): 3, (d): 4

To undo the merge

```bash
git reset --hard master^
```
