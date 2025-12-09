
#### Monitoring Temperatures, Fans, and Voltages with lm-sensors

Package
`lm-sensors`

Calibrate lm-sensors to hardware

``` bash
sensors-detect
```

The modules will be loaded after restart. To load them immediately 

\# **systemctl restart systemd-modules-load.service**

To get sensor data

``` bash
sensors
```

```
coretemp-isa-0000
Adapter: ISA adapter
Package id 0: +42.0°C (high = +86.0°C, crit = +96.0°C)
Core 0: +34.0°C (high = +86.0°C, crit = +96.0°C)
Core 1: +35.0°C (high = +86.0°C, crit = +96.0°C)
Core 2: +32.0°C (high = +86.0°C, crit = +96.0°C)
Core 3: +31.0°C (high = +86.0°C, crit = +96.0°C)
```

To get updated status every 2 seconds. CTRL + C to stop

``` bash
watch -d sensors
```

Set different update interval

``` bash
watch -d -n 'SECONDS' sensors
```
