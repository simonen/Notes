
ssh: for private repositories, authenticated users
http(s), git: for public read-only access

Inspect repositories with a web browser. Git web apps:
`Gitweb`
`Cgit`

`Gitolite`: Additional layers on top of git that is hosted with ssh. Allows to grant or revoke read, write and forced write privileges.


#### Hosting Git Repositories Over SSH

Create a git user (repository)

```bash
adduser -s /usr/bin/git-shell -c Git git
```

Create a user and an SSH key pair on the client

```bash
adduser -c 'USER DESC' 'USER'
su - 'USER'
mkdir -m 0750 /home/'USER'/.ssh
ssh-keygen -t rsa -C peter@ldap.cc -N "" -f /home/peter/.ssh/id_rsa
```

Copy the public key to the remote repository git user

Clone a repository over SSH (client)

```bash
git clone ssh://'USER'@'HOST'/'REPO-DIR'
```

or

```bash
git clone 'USER'@'HOST':/'REPO-DIR'
```

> SSH options cannot be passed to the git command. Use `~/.ssh/config`. Anonymous SSH access is not allowed

Clone the arbitrary public repo from github.com using the SSH protocol. All clone commands include `git@github.com`

```bash
git clone git@github.com:jquery/jquery.git
```

Commit a revision and push

```bash
git push -u origin master
```

#### Hosting Git Repositories with Git Daemon

Create a directory for the public repo

```bash
mkdir /srv/git
```

Create a repo

```bash
git init --bare /srv/git/my-public-repo.git
```

Mark the repository as exportable

```bash
touch /srv/git/my-public-repo.git/git-daemon-export-ok
```

Set the permissions

```bash
chmod -R 0755 /srv/git
```

##### Configure and enable the `git-daemon`

Create the `systemd` service file for `git-daemon`

`/etc/systemd/system/git-daemon.service`
```
[Unit]
Description=Git Daemon
After=network.target

[Service]
ExecStart=/usr/libexec/git-core/git-daemon --reuseaddr --base-path=/srv/git/ --export-all --enable=receive-pack --verbose
User=git
Group=git
Restart=always

[Install]
WantedBy=multi-user.target
```

`--base-path=/srv/git/`: Restricts access to repositories under `/srv/git/`.
`--export-all`: Allows all repositories (with `git-daemon-export-ok`) to be served.
`--enable=receive-pack`: Enables pushing to the repository (optional). !!!

Reload `systemd` and start the service

```bash
systemctl daemon-reload
systemctl enable --now git-daemon
```

Allow through the firewall

```bash
firewall-cmd --add-service=git --permanent && firewall-cmd --reload
```

SElinux

```bash
setsebool nis_enabled 1
```

**`nis_enabled`**: This SELinux boolean allows services, like `git-daemon`, to create and manage network connections for accepting and listening on TCP sockets.

Clone the repository

```bash
git clone git://'HOST'/my-public-repo.git
```

#### Hosting Git Repositories Over HTTP

On the server

```
/var/www/html/
└── 03-01.git
    ├── branches
    ├── config
    ├── description
    ├── HEAD
    ├── hooks
    ├── info
    ├── objects
    ├── packed-refs
    └── refs
```

Install `httpd`, allow it through the firewall, SElinux

```bash
dnf install -y httpd && systemctl enable --now httpd
firewall-cmd --add-service=httpd --permanent; firewall-cmd --reload
setsebool -P httpd_can_network_connect 1
restorecon -R /var/www/html
```

Clone the repo (client)

```bash
git clone http://'REPO_ADDR'/03-01.git
```

#### Github

A new repository `12-04` is created on Github. Publish a project to the empty repo

```bash
git remote add origin git@github.com:'USERNAME'/'REPO-DIR'.git
git push -u origin master
```

#### Sending Pull Requests

> When contributing to projects hosted on Github, always use branches! Do not work in the master branch!

Pull requests repositories

* Original repository hosted on Github
* Your fork hosted on Github
* Locally cloned original repository linked to the fork

Pull request workflow

* You commit, merge and rebase in your local repository only
* You push the changes from local repo to the fork
* Send pull requests from your fork to the original repo
* Fetch the latest updates from the original repository to the local repository

Create a clone of the repo stored locally

```bash
git clone git@github.com:'ORGANIZATION'/'REPO'.git
```

Add the remote URL aliased by me, pointing to the fork

```bash
git remote add 'MY-ALIAS' git@github.com:'MY-USERNAME'/'FORKED-REPO-NAME'
```

```
[gina@ldap 12-06]$ git remote -v
my      git@github.com:simonen/vivid-12-06.git (fetch) <- forked repo
my      git@github.com:simonen/vivid-12-06.git (push)
origin  git@github.com:vivid-pictures/12-06.git (fetch) <- original repo
origin  git@github.com:vivid-pictures/12-06.git (push)
```

Create a new branch, add commits and push the changes

```bash
git checkout -b 'BRANCH'
git commit -m ...
git push -u 'MY-ALIAS' HEAD
```

The branch is now available in the fork

To append new commits to an existing pull request

```bash
git push -f 'MY-ALIAS' 'MY-BRANCH'
```

`-f`: Necessary if during the update phase the revisions were rebased. If no new revisions in the original master branch, the -f flag can be omitted.
