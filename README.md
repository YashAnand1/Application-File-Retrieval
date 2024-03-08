<div align = "center">

# Retrieving Application Details
</div>

## Index
- [Introduction](#Introduction)
- [Finding Listening Programs](#Finding-Listening-Programs)
- [Finding Process Details With PS](#Finding-Process-Details-With-PS)
- [Listing Files With LSOF](#Listing-Files-With-LSOF)
- [Retrieving Files From SysCalls Using STRACE](#Retrieving-Files-From-SysCalls-Using-STRACE)
- [Finding Package Names Using DPKG](#Finding-Package-Names-Using-DPKG)
- [Locating Most Files Using DLOCATE](#Locating-Most-Files-Using-DLOCATE)
- [Retrieving Remaining Logs From SysLog](#Retrieving-Remaining-Logs-From-SysLog)
- [Retrieved Files](#Retrieved-Files)

## Introduction
As per the assigned task, I was required to find the Binary, Log, Configuration and Data directory paths of the Listening Programs from my system. Since [I have been able to retrieve the required files](https://docs.google.com/spreadsheets/d/1JQPDfufZKjkzZ16GkwGl2jKMo-j2qa4alAY_r9u-H6Y/edit#gid=0), this document has been created for the reference of others and mine for how I was able to retrieve these files. 

The tools utilised for retrieving such files were as follows:
- [Strace](https://opensource.com/article/19/10/strace#:~:text=A%20system%20call%20is%20a,understand%20how%20system%20calls%20work.)
- [Dlocate](https://github.com/craig-sanders/dlocate)
- [Lsof](https://www.geeksforgeeks.org/lsof-command-in-linux-with-examples/)
- [Netstat](https://www.lifewire.com/netstat-command-2618098)
- [Ps](https://www.digitalocean.com/community/tutorials/linux-ps-command)
- [Dpkg](https://phoenixnap.com/kb/dpkg-command)

Out of the above 2 tools, `lsof` and `strace` proved to be the most powerful in my experience. In the next sections, I have documented how these tool were utilised for being able to retrieve the required files as per my requirements. 

## Finding Listening Programs
1. `Netstat`: `sudo netstat -tlnp | awk '{print $7}'| grep / | cut -f1 -d/ | sort -u` 

```
1
1286
1539
1814
1962
1979
2014
2180
3497
774
8380
965
```                 

- `sudo netstat -tlnp`
    - Helps with displaying the PIDs of Listening Programs  
    - `-tlnp`: TCP+Listening+Numeric+Programs 
- `awk '{print $7}'| grep / | cut -f1 -d/` 
    - Helps printing PIDs of Listening Programs 
    - `awk '{print $7}': Prints the seventh column of 'PIDs/Program Name'
    - `grep / | cut -f1 -d/`: Filtering is done on the basis of `PID/Program` & First field for getting PID is printed before every "/"
    - `sort -u`: The output is uniquely sorted to remove duplicate outputs 

## For retrieving the required files 

### Finding Process Details With PS

`ps -aux`: `sudo netstat -tulnp | awk '{print $7}' | grep / | cut -d/ -f1 | sort -u | while read pid; do ps -aux | awk -v pid="$pid" '$2 == pid {print}'; done`
```
libvirt+    1814  0.0  0.0  11256  1320 ?        S    Mar07   0:00 /usr/sbin/dnsmasq --conf-file=/var/lib/libvirt/dnsmasq/default.conf --leasefile-ro --dhcp-script=/usr/lib/libvirt/libvirt_leaseshelper
mysql       2180  0.5  2.3 2204164 189296 ?      Ssl  Mar07   6:14 /usr/sbin/mysqld
nobody      1962  0.1  1.7 163256 142340 ?       Ss   Mar07   1:15 /usr/sbin/merecat -sn /var/www
nobody      8380  0.2  3.5 316856 281704 ?       Ss   Mar07   2:25 /usr/sbin/merecat
postgres    1539  0.0  0.2 224164 17476 ?        Ss   Mar07   0:02 /usr/lib/postgresql/16/bin/postgres -D /var/lib/postgresql/16/main -c config_file=/etc/postgresql/16/main/postgresql.conf
prometh+     965  0.0  0.0 1170244 3500 ?        Ssl  Mar07   0:00 /usr/bin/prometheus-node-exporter
root           1  0.0  0.1 171096 12624 ?        Ss   Mar07   0:19 /sbin/init splash
root        1286  0.0  0.0  37900  7152 ?        Ss   Mar07   0:00 /usr/sbin/cupsd -l
root        2014  0.0  0.0   5008   304 ?        Ss   Mar07   0:00 /usr/sbin/rpc.mountd
root        3497  0.0  0.0  43104  2156 ?        Ss   Mar07   0:00 /usr/lib/postfix/sbin/master -w
statd       1979  0.0  0.0   4640  1832 ?        Ss   Mar07   0:00 /sbin/rpc.statd
systemd+     774  0.0  0.1  21020  8228 ?        Ss   Mar07   0:46 /lib/systemd/systemd-resolved
```
    - `while read pid; do ps -aux | awk -v pid="$pid" '$2 == pid {print}'; done'
        - `while read pid; do ps -aux`: Initiating looping for entering each PID into pid variable & prints all listing processes in system
        - `awk -v pid="$pid" '$2 == pid {print}'; done`: Output of `ps -aux` is filtered based on the second column of PID & entire output is printed - Loop closed with `; done`
    - Detailed output can be accessed from [here]()

### Listing Files With LSOF
`lsof | grep <listening program from netstat>` 
```
rsyslogd    1075   1139 rs:main            syslog    6u     unix 0xffff9c57896a6000       0t0      28075 /var/spool/postfix/dev/log type=DGRAM (UNCONNECTED)
master      3497                             root  cwd       DIR                8,2      4096    2658612 /var/spool/postfix
master      3497                             root  txt       REG                8,2     47496    9192559 /usr/lib/postfix/sbin/master
master      3497                             root  mem       REG                8,2    303320    9192545 /usr/lib/postfix/libpostfix-util.so
master      3497                             root  mem       REG                8,2    318952    9192542 /usr/lib/postfix/libpostfix-global.so
master      3497                             root   10uW     REG                8,2        33    2658969 /var/spool/postfix/pid/master.pid
master      3497                             root   11uW     REG                8,2        33    2658970 /var/lib/postfix/master.lock
qmgr        3510                          postfix  cwd       DIR                8,2      4096    2658612 /var/spool/postfix
qmgr        3510                          postfix  rtd       DIR                8,2      4096          2 /
qmgr        3510                          postfix  txt       REG                8,2     67968    9192560 /usr/lib/postfix/sbin/qmgr
[. . .]
```
    - `lsof`: Displays "List of opened files" and filters it on the basis of Listening Program Name
    - Detailed output can be accessed from [here]()

### Retrieving Files From SysCalls Using STRACE
`strace -f <executed command from ps>` or `strace -o output.txt -fe openat <executed command from ps>`
```
537291 openat(AT_FDCWD, "/usr/lib/mysql/private/glibc-hwcaps/x86-64-v3/libssl.so.3", O_RDONLY|O_CLOEXEC) = -1 ENOENT (No such file or directory)
537291 openat(AT_FDCWD, "/usr/lib/mysql/private/glibc-hwcaps/x86-64-v2/libssl.so.3", O_RDONLY|O_CLOEXEC) = -1 ENOENT (No such file or directory)
537291 openat(AT_FDCWD, "/usr/lib/mysql/private/libssl.so.3", O_RDONLY|O_CLOEXEC) = -1 ENOENT (No such file or directory)
537291 openat(AT_FDCWD, "/usr/sbin/../lib/mysql/private/glibc-hwcaps/x86-64-v3/libssl.so.3", O_RDONLY|O_CLOEXEC) = -1 ENOENT (No such file or directory)
537291 openat(AT_FDCWD, "/usr/sbin/../lib/mysql/private/glibc-hwcaps/x86-64-v2/libssl.so.3", O_RDONLY|O_CLOEXEC) = -1 ENOENT (No such file or directory)
537291 openat(AT_FDCWD, "/usr/sbin/../lib/mysql/private/libssl.so.3", O_RDONLY|O_CLOEXEC) = -1 ENOENT (No such file or directory)
537291 openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
537291 openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libssl.so.3", O_RDONLY|O_CLOEXEC) = 3
[. . .]
```
    - Saving all OpenAt SysCalls into an output file
        - `strace -o output.txt`: Save the output of strace into a file
        - `-fe openat`: `f` for child-processes & `e` for retrieving only OpenAt SysCall as the expression

### Finding Package Names Using DPKG
`sudo netstat -tulnp | awk '{print $7}' | grep / | cut -d/ -f1 | sort -u | while read pid; do ps -aux | awk -v pid="$pid" '$2 == pid {print $11}' | sort -u; done | xargs -I {} dpkg -S {} | cut -f1 -d:`
```
systemd-sysv
cups-daemon
postgresql-16
dnsmasq-base
merecat
nfs-common
nfs-kernel-server
mysql-community-server-core
postfix
systemd-resolved
merecat
prometheus-node-exporter
```
    - For retrieving the package name from the binaries
        -  `xargs -I {} dpkg -S {}`: [`xargs -I`](https://stackoverflow.com/questions/68088351/what-is-the-syntax-for-xargs-i-flag-argument) captures the output and places it after [`dpkg -S`](https://phoenixnap.com/kb/dpkg-command) for Searching (-S) its package names 
        - `cut -f1 -d:`: For retrieving only the package names out of `packageName: packageBinary` output

### Locating Most Files Using DLOCATE
`dlocate <pkg>, dlocate --conf <pkg>, dlocate --lsdir <pkg>`
```
$ dlocate postfix
postfix: /.
postfix: /etc
postfix: /etc/init.d
postfix: /etc/init.d/postfix
postfix: /etc/insserv.conf.d
postfix: /etc/insserv.conf.d/postfix
postfix: /etc/network
postfix: /etc/network/if-down.d
postfix: /etc/network/if-down.d/postfix
postfix: /etc/network/if-up.d
postfix: /etc/network/if-up.d/postfix
postfix: /etc/postfix
postfix: /etc/postfix/dynamicmaps.cf.d
[. . . ]

$ dlocate --conf postfix
/etc/init.d/postfix
/etc/insserv.conf.d/postfix
/etc/network/if-down.d/postfix
/etc/network/if-up.d/postfix
/etc/postfix/post-install
/etc/postfix/postfix-files
/etc/postfix/postfix-script
/etc/ppp/ip-down.d/postfix
/etc/ppp/ip-up.d/postfix
/etc/resolvconf/update-libc.d/postfix
/etc/rsyslog.d/postfix.conf
/etc/ufw/applications.d/postfix

$ dlocate --lsdir postfix
/.
/etc
/etc/init.d
/etc/insserv.conf.d
/etc/network
/etc/network/if-down.d
/etc/network/if-up.d
/etc/postfix
/etc/postfix/dynamicmaps.cf.d
/etc/postfix/postfix-files.d
/etc/postfix/sasl
/etc/ppp
/etc/ppp/ip-down.d
/etc/ppp/ip-up.d
[. . .]
```
    - From the previous step, the package names are placed after each command for retrieving the required files
        - `dlocate <pkg>`: To "list records that match either package or files names" - Records matching package name are displayed
        - `dlocate --conf <pkg>`: To "list conffiles in package"
        - `dlocate --lsdir <pkg>`: To "list only the directories in package"

### Retrieving Remaining Logs From SysLog
`grep <service from netstat> /var/log/syslog`
```
$ grep rpc.statd /var/log/syslog
2024-03-05T12:57:51.802190+05:30 yashanand systemd[1]: Stopping rpc-statd.service - NFS status monitor for NFSv2/3 locking....
2024-03-05T12:57:52.064752+05:30 yashanand systemd[1]: rpc-statd.service: Deactivated successfully.
2024-03-05T12:57:52.065052+05:30 yashanand systemd[1]: Stopped rpc-statd.service - NFS status monitor for NFSv2/3 locking..
2024-03-05T12:58:43.677337+05:30 yashanand systemd[1]: Starting rpc-statd.service - NFS status monitor for NFSv2/3 locking....
2024-03-05T12:58:43.715569+05:30 yashanand rpc.statd[2523]: Version 2.6.2 starting
2024-03-05T12:58:43.715683+05:30 yashanand rpc.statd[2523]: Flags: TI-RPC 
2024-03-05T12:58:43.725150+05:30 yashanand systemd[1]: Started rpc-statd.service - NFS status monitor for NFSv2/3 locking..
2024-03-05T12:58:43.847260+05:30 yashanand systemd[1]: Starting rpc-statd-notify.service - Notify NFS peers of a restart...
[. . .]
```
    - The remaining logs yet to be retrieved for the Listening Programs can be filtered out from `/var/log/syslog`
    - Some applications by default store their logs in here but it is also the "common file for logs" - Studied more on it from [here](https://www.logicmonitor.com/blog/what-is-syslog#:~:text=Syslog%2C%20an%20abbreviation%20for%20system,different%20parts%20of%20the%20system.)

## Retrieved Files

The Log, Binary, Configuration and Data paths that I was able to retrieve using these tools have been presented below for reference. 

| S. No | Listening Service | Package Name | Binary File/Directory | Configuration File/Directory | Log File/Directory | Data File/Directory |
|-------|-------------------|--------------|-----------------------|------------------------------|--------------------|---------------------|
| 1     | init              | systemd-sysv | /sbin/init            | N/A                          | /var/log/syslog    | N/A                 |
| 2     | cupsd             | cups-daemon  | /usr/sbin/cupsd       | "/etc/apparmor.d/usr.sbin.cupsd<br>/etc/cups/cups-files.conf<br>/etc/init.d/cups<br>/etc/logrotate.d/cups-daemon<br>/etc/pam.d/cups<br>/etc/ufw/applications.d/cups" | /var/log/cups       | /usr/share/cups     |
| 3     | postgres          | postgresql-16| /usr/lib/postgresql/16/bin/postgres | etc/postgresql/16/main/postgresql.conf | /var/log/postgresql/postgresql-16-main.log | /var/lib/postgresql/16/main |
| 4     | dnsmasq           | dnsmasq-base | /usr/sbin/dnsmasq     | /etc/dbus-1/system.d/dnsmasq.conf | /var/log/syslog     | /var/lib/misc       |
| 5     | merecat           | merecat      | /usr/sbin/merecat     | /etc/merecat.conf            | /var/log/syslog     | /var/www            |
| 6     | rpc.statd         | nfs-common   | /sbin/rpc.statd       | "/etc/default/nfs-common<br>/etc/init.d/nfs-common<br>/etc/request-key.d/id_resolver.conf" | /var/log/syslog     | /var/lib/nfs        |
| 7     | rpc.mountd        | nfs-kernel-server | /usr/sbin/rpc.mountd | /etc/init.d/nfs-kernel-server | /var/log/syslog     | /var/lib/nfs        |
| 8     | mysqld            | mysql-community-server-core | /usr/sbin/mysqld | /etc/mysql/conf.d/          | /var/log/mysql/error.log | /usr/lib/mysql      |
| 9     | master            | postfix      | /usr/lib/postfix/sbin/master | "/etc/init.d/postfix<br>/etc/insserv.conf.d/postfix<br>/etc/network/if-down.d/postfix<br>/etc/network/if-up.d/postfix<br>/etc/postfix/post-install<br>/etc/postfix/postfix-files<br>/etc/postfix/postfix-script<br>/etc/ppp/ip-down.d/postfix<br>/etc/ppp/ip-up.d/postfix<br>/etc/resolvconf/update-libc.d/postfix<br>/etc/rsyslog.d/postfix.conf<br>/etc/ufw/applications.d/postfix" | /var/log            | /var/lib/postfix    |
| 10    | systemd-resolve   | systemd-resolved | /lib/systemd/systemd-resolved | /etc/systemd/resolved.conf | /var/log/syslog     | "/lib/systemd<br>/lib/systemd/system<br>/usr/lib/sysusers.d<br>/usr/lib/tmpfiles.d" |
| 11    | prometheus-node   | prometheus-node-exporter | /usr/bin/prometheus-node-exporter | "/etc/default/prometheus-node-exporter<br>/etc/init.d/prometheus-node-exporter<br>/etc/logrotate.d/prometheus-node-exporter" | /var/log/prometheus | /var/lib/prometheus/node-exporter |
