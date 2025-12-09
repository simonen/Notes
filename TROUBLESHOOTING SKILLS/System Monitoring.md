
#### Using nmon

Package 
nmon

$ nmon

To save stats every N seconds
$ nmon -f -s *SECONDS* -c REFRESHES

-s: seconds between refreshing the screen
-c number of refreshes

To collect data over 1 hour, 1 second refresh rate
$ nmon -f -s 1 -c 3600

This will produce a .nmon file in the current dir 

To visualize the data, use the nmonchart tool
https://sourceforge.net/projects/nmon/files/

Unpack the file
$ tar xvf nmonchart42.tar

Run the executable against the .nmon file
\$ ./**nmonchart** *NMON_DATA*.nmon *index.html*

Put it in the document root of a web server and open it in a browser

#### Using watch

man watch

Package
`procps-ng`

watch  runs  command  repeatedly,  displaying  its  output and errors (the first screenful).  This allows you to watch the program output change over time.  By default, the program is run every 2 seconds.  By default, watch will run until interrupted.

`watch [options] command`

Watch the contents of a directory in 2s intervals

``` bash
watch 'ls -l' /"DIR"
```

```
Every 2.0s: ls -l /home/kimchen                    Thu May  9 22:19:10 2024

total 32792
-rw-r--r--. 1 root    root        1984 Apr 18 11:19 ca.cert.pem
-rw-r--r--. 1 root    root        4328 Apr 12 14:05 cacert.pem
```

#### Continuous Performance Monitoring

`Metrics collector` ->  `Metrics exporter` -> `Back-end Metrics storage` <- `Graphing`

`Collectd` - system metrics collection daemon
`Graphite` - A store to collect and graph metrics
	`carbon`: A high-performance listener for receiving time-series data
		`carbon-relay`
		`carbon-cache`
		`carbon-aggregator`
	`whisper`: A time-series database for metrics data
	`graphite-API`: An API that fetches JSON data from the time-series Whisper database
`Grafana` - Front-end for graphing metrics

####  Host Metrics with Collectd

Packages:
`collectd collectd-mysql`

Configuration file
`/etc/collectd.conf`

SELinux
`collectd_tcp_network_connect --> on`


