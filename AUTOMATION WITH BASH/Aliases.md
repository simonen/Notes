
Create an alias

```bash
alias 'ALIAS'='COMMAND'
```

``` bash
alias 'cobbler stat'='systemctl status cobbler'
```

Remove an alias

``` bash
unalias 'ALIAS'
```

Create an alias as a function

`/etc/bashrc`
``` bash
cobbler() {
    case "$1" in
        stat)
            systemctl status cobbler
            ;;
        *)
            echo "Usage: cobbler {stat}"
            ;;
    esac
}
```

#### Environment Variables

Export an environment variable. The variable name is in all caps.

``` bash
export 'VARIABLE'='value'
```

To list exported variables

``` bash
export -p
```

To remove a variable

```
unset VARIABLE
```