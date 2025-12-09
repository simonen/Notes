
- `/etc/profile` (for login shells) → may source `/etc/bashrc`
- `/etc/bashrc` (global, all users)
- `~/.bashrc` (user-specific, overrides global)

List all environment variables

``` bash
env
```

Shows all environment variables that have been exported in the shell.

``` bash
export -p
```

Set an environment variable

```bash
export ENV_VARIABLE=value
```

> Exported environment variables live only in the subshell from where they were exported  Once it exits, the variables expire. 

Remove an environment variable

```bash
unset ENVIRONMENT_VARIABLE
```

To make variables permanent 

```bash
source "SCRIPT CONTAINING ENV VARIABLES"
```

