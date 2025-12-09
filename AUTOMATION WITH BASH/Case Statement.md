
```
case variable in
    pattern1)
        # Commands to execute if variable matches pattern1
        ;;
    pattern2)
        # Commands to execute if variable matches pattern2
        ;;
    *)
        # Commands to execute if variable does not match any patterns
        ;;
esac
```

Using positional arguments

```
#!/bin/bash

case $1 in
    start)
        echo "Starting the service..."
        ;;
    stop)
        echo "Stopping the service..."
        ;;
    restart)
        echo "Restarting the service..."
        ;;
    *)
        echo "Usage: $0 {start|stop|restart}"
        ;;
esac
```

**$1**: This refers to the first argument passed to the script when it's executed. For example, if you run `./my_script.sh start`, then `$1` will be `start`.