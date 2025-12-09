#### Cron for Recurring Tasks

**crond** service consists of:
* **crond** daemon: checks every minute if there is work to do. c
* **cron configuration**: which consists of multiple files that work together to ensure tasks are executed in the right time

**crond** does no need to be reloaded when a conf change is made. It does it self-checks every minute. 

#### Cron Timing

**man 5 crontab**

`* * * * *`

From left to right
**minute**: 0-59
**hour**: 0-23
**day**-of-month: 1-31
**month**: 1-12 (or month name)
**day-of-week**: 0-7 (Sunday is 0 or 7) or day names

"`*`"   - wildcard. Refers to any value

`* 11 * * *` : every minute between 11:00 : 11:59
`0 11 * * 1-5` :  at 11 a. m every day of the week Mon-Fri
`0 7-18 * * 1-5*` : every hour at 7:00, 8:00, 9:00...18:00, Mon-Fri
`0 */2 2 12 5` : every 2 hours on the hour Dec 2nd and every Fri in Dec.

#### Cron Configuration Files

**/etc/crontab**: main config file for cron. Not changed directly. For overview only

Cron files used for configuration
* *cron* files in **/etc/cron.d**
* scripts in **/etc/cron.daily**, **cron.hourly**, **cron.weekly**, **cron.monthly**
* user-specific files created with **crontab -e** command

To test cron execution, min of 3 minute margin is recommended, as cron checks config every minute.

To get a list of cron schedules

``` bash
crontab -l
```

-u "USER" will list another user's cron jobs.

To create user-specific cron jobs (logged as the same user):

``` bash
crontab -e
```

To edit another user's crontab if logged as **root**

``` bash
crontab -e -u "USER"
```

**crontab -e** creates a temporary file in **/var/spool/cron** for the specific user, and is activated when the file has been saved. **Files should NOT be edited directly!**  

Cronjobs can be added to **/etc/cron.d** directory instead. Filename does not matter, just the syntax in them.

```
[kimchen@rhel9 cron.d]$ cat 0hourly
# Run the hourly jobs
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root
01 * * * * root run-parts /etc/cron.hourly
```

To remove your own or another user's cron

``` bash
crontab [-u "USER"] -r
```

File starts with setting Environment variables which should be considered for the specific job.
The job is specified after that:
`01 * * * *` : specifies the execution time of the job - 1 min after every hour, every day of month, every month, every day of week
**root**: job will be executed as user root
**run-parts**: the command that should be executed
**/etc/cron.hourly**: command argument for the **run-parts** command

**run-parts** finds and executes all executable files in the specified directory

Here the command would be run at 2 minutes past the hour between the hours of 12 a.m. and 4 a.m. and between 12 p.m. and 4 p.m.

`2 0-4,12-16 1 * * root run-parts /etc/cron.monthly`
#### Anacron

**anacron** executes tasks relatively timed to certain events. Does not use absolute timing.

**man 8 anacron**
**man anacrontab**

**/etc/anacrontab**

```
period in days    delay in minutes    job-identifier    command
```

The following directories contain scripts that will be automatically executed, without any further configuration, as the name suggests: once a day, once a month, etc. Exact timing is not configured. Taken care of by **anacron**

* /etc/cron.hourly
* /etc/cron.daily
* /etc/cron.weekly
* /etc/cron.monthly

```
[kimchen@rhel9 ~]$ cat /etc/anacrontab
# /etc/anacrontab: configuration file for anacron

# See anacron(8) and anacrontab(5) for details.

SHELL=/bin/sh
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root
# the maximal random delay added to the base delay of the jobs
RANDOM_DELAY=45
# the jobs will be started during the following hours only
START_HOURS_RANGE=3-22

#period in days   delay in minutes   job-identifier   command
1       5       cron.daily              nice run-parts /etc/cron.daily
7       25      cron.weekly             nice run-parts /etc/cron.weekly
@monthly 45     cron.monthly            nice run-parts /etc/cron.monthly
```

#### Cron Security

To limit which user is allowed to schedule cron jobs, put users in the 
* **/etc/cron.allow**:  to allow
* **/etc/cron.deny**: to deny

**Only ONE of the files should exist at a time!**
**If neither file exists, only root can use cron jobs**

#### At

Used to execute tasks only once.

To get an overview
**$ atq**

To remove a job
**$ atrm** *TASK_NUMBER*




