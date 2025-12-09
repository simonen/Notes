
http://www.akadia.com/services/ntp_synchronize.html
http://doc.ntp.org/4.1.1/confopt.htm.
Apress – Pro Linux System Administration 2nd 2017

**RTC**: Real-Time Clock. aka Hardware clock. Should be set to UTC time
**UTC**: Coordinated Universal Time. A time that is the same everywhere on the planet
**system time**: maintained by the OS. Independent of the hardware clock. Changes to sys time are not synchronized with the hardware clock
**Local Time**: the actual time in the current time zone
**DST**: Daylight Saving Time
**Atomic Clock**: reliable hardware clock connected to the computer
**epoch time**: time in seconds after Jan 1 1970, in UTC. To work until 2037 in 32-bit systems.
#### NTP

Network Time Protocol
**chrony** and **ntpd**: full NTP implementations, time servers
**timesyncd**: lightweight time client. Uses the **SMTP**

There is a wolrd-wide network of time servers organized into **strata**
**stratum**: reliability of the time server. Lower the better. 
* 0: source of all timekeeping. A network of atomic clocks, radio receivers
* **1**:  primary timeservers directly connected to stratum 0 sources
* 2: public servers syncing with stratum 1. Connect here instead of stratum 1
* **5**: local time servers with reliable hardware clocks
* **8**: NTP server will be used only if no external service is available
* **10**: for the local clock on every node that uses NTP. Enables the server to sync time even without external source
* **15**: used by clocks that indicate they should not be used for syncing

`driftfile` directive gives the server a place to store information about the idiosyncrasies of your local system clock. It stores the clock frequency offset every hour, depending on the tolerance of drift, and uses this information when the daemon is started. If the file is not there it sets the frequency offset to zero.
Over time, it will use this information to report the time more precisely between synchronization attempts, as the daemon knows how the local clock behaves.
`iburst`: synchronize quickly after network interruptions. Sends an extra 8 packets if does 
not get an initial response

Clients are connected to pool of servers, rather than individual servers

#### Identifying NTP Clients

Three time sync daemons: **!!! Use only one at a time !!!**
* **ntpd
* **chronyd** : newer, faster and more reliable than ntpd.
* **timesyncd**

$ **systemctl -t service --all | grep { ntpd | chronyd | timesyncd }**

``` bash
timedatectl
```

```
[root@dnsserver ~]# timedatectl
                Time zone: Europe/Sofia (EET, +0200)
System clock synchronized: no
              NTP service: active
          RTC in local TZ: no
```

This shows that time is received from the RTC only
Non-systemd systems can choose between **ntpd** and **chronyd** only

> **Restoring the system from a snapshop requires chronyd.service to be restarted in order to sync time!**
#### Using chronyc for Simple Time Synchronization Client

For more information, see the following:
• https://chrony.tuxfamily.org/faq.html
• https://chrony.tuxfamily.org/comparison.html

* chrony does not support multicast or manycasts
* chrony is useful when networks are intermittent (???)
* chrony works better in congested networks and virtual hosts (???)

Package
**chrony**

**man chronyc**
**man 5 chrony.conf**

NTP configuration. Contains standard list of NTP servers
**/etc/chrony.conf** 

`prefer`: always use this server if available

`chronycd` should the be only running time daemon

``` bash
chronyc activity
```

```
200 OK
8 sources online
0 sources offline
0 sources doing burst (return to online)
0 sources doing burst (return to offline)
0 sources with unknown address
```

`chrony.conf`

```
pool 0.ubuntu.pool.ntp.org iburst
pool 1.ubuntu.pool.ntp.org iburst
pool 1.ubuntu.pool.ntp.org iburst
server LOCAL_TIME_SERVER iburst prefer
```

**chrony** has better handling of network interruptions and faster syncing after restoring connectivity

To see the active NTP servers used by the client:

``` bash
chronyc sources
```

```
210 Number of sources = 1
MS Name/IP address         Stratum Poll Reach LastRx Last sample
===========================================================================
^* dnsserver                    10   6    17    11   -112ns[-4320ns] +/-  
```

#### Using chrony as a LAN Time Server

> [!NOTE] In chrony.conf
> ```
> # specify source pools
> pool 0.north-america.pool.ntp.org iburst
> pool 1.north-america.pool.ntp.org iburst
> pool 2.north-america.pool.ntp.org iburst
> 
> local stratum 10
> 
> # allowed networks
> allow 192.168.0.0/16
> allow 2001:db8::/56
> 
> driftfile /var/lib/chrony/chrony.drift
> maxupdateskew 100.0
> rtcsync
> logdir /var/log/chrony
> log measurements statistics tracking
> leapsectz right/UTC
> makestep 1 3
> ```
> 

Allow the NTP service through the firewall:

``` bash
firewall-cmd --add-service=ntp --permanent
firewall-cmd --reload
```

`chrony` Statistics

``` bash
chronyc sources -v
chronyc tracking
```

#### Using ntpd as NTP Client

man ntpd
**/etc/ntp.conf**

List syncing NTP servers

**$ ntpq -p**

#### Setting Up Time Manually with timedatectl

Time on Linux systems is managed by RTC and system time (kernel)
RTC has constant power provided by the mobo battery. When the computer starts it gets its initial time from the RTC. When network is available, it corrects the time from the upstream time servers.

**date**: manages local time
**hwclock**: manages hardware time
**timedatectl**: to manage all aspects of time

\# timedatectl set-ntp false
\# timedatectl set-time "2020-10-04 19:30:00"
$ timedatectl status

To convert epoch time into human readable time:
**$ date --date** '@1420987251

/etc/ntp/ntp.conf directives
`statistics`, would enable `loopstats`, `peerstats`, and `clockstats` reporting to files in `/var/log/ntpstats`.

`loopstats` collects information on the updates made to the local clock by the ntpd server. `peerstats` logs information about all peers—upstream servers as well as clients that use your server to synchronize.
`clockstats` writes statistical information about the local clock to the log file.
The `filegen` directive tells the daemon which file you want this statistical information written to and how often the file needs to be changed. 
`restrict default nomodify notrap nopeer noquery`

* default: wildcard, matches all addresses
* notrap: rejects control packets that get sent
* nomodify: disallows attempts to modify time on the server
* nopeer: ensures the server does not start using a connecting client as an upstream NTP server
* noquery: prevents the server from being queried for peer and other stats


#### Using date

Enables to manage system time. Does not sync hardware time

```
[gligana@rhel81 users]$ date
Tue Jan  9 18:17:51 EET 2024
```

```
[gligana@rhel81 users]$ date +%d-%m-%y
09-01-24
```

**date -s 16:03**: sets the time to 16:03

#### Using hwclock

**hwclock --systohc**: syncs current system time to hardware time
**hwclock --hctosys**: syncs current hardware time to system time

#### Using timedatectl

**NTP** works with the **chronyd.service**. The service should be active to use NTP

To enable NTP syncing:
**$ timedatectl set-ntp 1**

To set time:
**$ timedatectl set-time** *TIME*

To list NTP servers. timesynced.service only
**$ timedatectl timesync-status**`

```
root@server15:~# timedatectl timesync-status
       Server: 192.168.137.23 (192.168.137.23)
Poll interval: 1min 4s (min: 32s; max 34min 8s)
         Leap: normal
      Version: 4
      Stratum: 10
```

#### Managing Time Zone Settings
Time zone data comes from IETF.org Timezones
In order to use consistent time, regardless of time zones, Linux servers communicate in UTC. 

Three approaches to changing time zones:
**/usr/share/timezone**: contains files for each time zone
Current time zone is presented as a symbolic link 
**/etc/localtine** -> **/usr/share/zoneinfo/Europe/Sofia**

To list all timezones:
**$ timedatectl list-timezones**

To set time zone:
**$ timedatectl set-timezone** *CONTINENT/CAPITAL*

To change manually 
**$ ln -sf /usr/share/zoneinfo/** CONTINENT/CAPITAL **/etc/localtime**
-f: overwrites existing sym link

Using the **timedatectl set-timezone**: Recommended method

Using **tzselect**

