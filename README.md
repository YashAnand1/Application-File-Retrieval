<div align = "center">

# Retrieving Application Details
</div>

## Index
- [Tools Utilised](#Tools-Utilised)
- [Listing Executed Commands From Listening Programs](#Listing-Executed-Commands-From-Listening-Programs)
- [Finding Package Names From Binaries](#Finding-Package-Names-From-Binaries)
- [Finding Data And Conf Directories Using Dlocate](#Finding-Data-And-Conf-Directories-Using-Dlocate)
- [Finding Remaining Logs From Lsof And Syslog](#Finding-Remaining-Logs-From-Lsof-And-Syslog)
- [Finding Remaining Configurations From Strace](#Finding-Remaining-Configurations-From-Strace)

## Tools Utilised
- For finding details of Listening Programs
    - `Netstat`
        -`sudo netstat -tulnp | awk '{print $7}'| grep / | cut -f1 -d/ | sort -u`
    - `ps -aux`
        - `sudo netstat -tulnp | awk '{print $7}' | grep / | cut -d/ -f1 | sort -u | while read pid; do ps -aux | awk -v pid="$pid" '$2 == pid {print $11}' | sort -u; done | sort -u`

- For retrieving the required files 
    - `lsof | grep <listening program from netstat>` 
    - `strace -f <executed command from ps>` (f for child-procs)
    - `dlocate <pkg>, dlocate --conf <pkg>, dlocate --lsdir <pkg>`
    - `grep <service from netstat> /var/log/syslog`

## Listing Executed Commands From Listening Programs
- Retrieve Listening Programs & Process Details using **Netstat & Ps**:
    - Under the 'COMMAND' column, look for the executed commands
    - If they contain binary, config, log and data directory paths, save them
```
sudo netstat -tulnp | awk '{print $7}' | grep / | cut -d/ -f1 | sort -u | while read pid; do ps -aux | awk -v pid="$pid" '$2 == pid {print}' | sort -u; done

Output:
root           1  0.1  0.1 169772 14120 ?        Ss   15:34   0:06 /sbin/init splash
root        1286  0.0  0.1  37900 12376 ?        Ss   15:34   0:00 /usr/sbin/cupsd -l
postgres    1539  0.0  0.3 224164 30648 ?        Ss   15:34   0:00 /usr/lib/postgresql/16/bin/postgres -D /var/lib/postgresql/16/main -c config_file=/etc/postgresql/16/main/postgresql.conf
libvirt+    1814  0.0  0.0  11256   516 ?        S    15:34   0:00 /usr/sbin/dnsmasq --conf-file=/var/lib/libvirt/dnsmasq/default.conf --leasefile-ro --dhcp-script=/usr/lib/libvirt/libvirt_leaseshelper
nobody      1962  0.2  1.9 163256 154244 ?       Ss   15:34   0:11 /usr/sbin/merecat -sn /var/www
statd       1979  0.0  0.0   4640  2116 ?        Ss   15:34   0:00 /sbin/rpc.statd
root        2014  0.0  0.0   5008   444 ?        Ss   15:34   0:00 /usr/sbin/rpc.mountd
mysql       2180  1.2  4.8 2204164 392448 ?      Ssl  15:34   1:06 /usr/sbin/mysqld
root        3497  0.0  0.0  43104  4868 ?        Ss   15:34   0:00 /usr/lib/postfix/sbin/master -w
systemd+     774  0.2  0.1  20492 13184 ?        Ss   15:34   0:11 /lib/systemd/systemd-resolved
nobody      8380  0.4  3.7 316856 302568 ?       Ss   15:41   0:21 /usr/sbin/merecat
prometh+     965  0.0  0.1 1170244 13620 ?       Ssl  15:34   0:00 /usr/bin/prometheus-node-exporter
```

## Finding Package Names From Binaries 
- Use binaries from previous step to find their package name:
    - These packages will be used with `dlocate`
```
sudo netstat -tulnp | awk '{print $7}' | grep / | cut -d/ -f1 | sort -u | while read pid; do ps -aux | awk -v pid="$pid" '$2 == pid {print $11}' | sort -u; done | xargs -I {} dpkg -S {} | cut -f1 -d:

Output:
systemd-sysv
cups-daemon
postgresql-16
dnsmasq-base
merecat
nfs-common
netperf
nfs-kernel-server
mysql-community-server-core
postfix
systemd-resolved
prometheus-node-exporter
```

## Finding Data And Conf Directories Using Dlocate  
-  The`dlocate` tool helps with viewing dpkg information & acts as an alternative for `dpkg -L` or `dpkg -S`:
```
dlocate --lsdir systemd-sysv
dlocate --conf systemd-sysv

dlocate --lsdir cups-daemon
dlocate --conf cups-daemon

dlocate --lsdir dnsmasq-base
dlocate --conf dnsmasq-base

dlocate --lsdir merecat
dlocate --conf merecat

dlocate --lsdir nfs-common
dlocate --conf nfs-common

dlocate --lsdir netperf
dlocate --conf netperf

dlocate --lsdir nfs-kernel-server
dlocate --conf nfs-kernel-server

dlocate --lsdir mysql-community-server-core 
dlocate --conf mysql-community-server-core

dlocate --lsdir postfix 
dlocate --conf postfix

dlocate --lsdir systemd-resolved
dlocate --conf systemd-resolved

dlocate --lsdir prometheus-node-exporter
dlocate --conf prometheus-node-exporter
```

## Finding Remaining Logs From Lsof And Syslog   
- Using the `list opened files` tool, the remaining logs were retrieved:
```
sudo lsof | grep postgres | grep log
sudo lsof | grep mysqld | grep log
cat /var/log/syslog
```

## Finding Remaining Configurations From Strace   
- Utilised the OpenAt() Syscalls from the strace output for finding remaining files from each executed command of the remaining Programs:
```
sudo strace -o mysql.txt -f /usr/sbin/mysqld
strace -f /usr/lib/postgresql/16/bin/postgres -D /var/lib/postgresql/16/main -c config_file=/etc/postgresql/16/main/postgresql.conf
```

