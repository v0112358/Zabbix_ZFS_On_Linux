##Zabbix template for ZFS On Linux

###How to use：

- Import template into zabbix server.

- Grant permissions to zabbix user: vi /etc/sudoers.d/zabbix

        Defaults:zabbix !requiretty
        zabbix  ALL=(root)  NOPASSWD:   /sbin/zpool
        zabbix  ALL=(root)  NOPASSWD:   /sbin/zfs

- Add new configure into /etc/zabbix/zabbix_agentd.conf

        UserParameter=zfs.zpool.status,sudo /sbin/zpool status | /bin/egrep -e "DEGRADED|OFFLINE|UNAVAIL|FAILED" | /usr/bin/wc -l
        UserParameter=zfs.capacity,sudo /sbin/zpool list -H -o capacity | awk -F'%' '{print $1}'
        UserParameter=zfs.arcmaxsize,/bin/grep c_max /proc/spl/kstat/zfs/arcstats | awk '{print $3}'
        UserParameter=zfs.arcusage,/bin/grep ^size /proc/spl/kstat/zfs/arcstats | awk '{print $3}'
        UserParameter=zfs.rpoolbwread,sudo /sbin/zpool iostat rpool | stdbuf -o0 awk 'NR > 3 {print($6)}' | stdbuf -o0 sed -e 's/K/\*1024/g' -e 's/M/\*1048576/g' -e 's/G/\*1073741824/g' | bc | stdbuf -o0 awk '{printf("%d\n",$1 + 0.5)}'
        UserParameter=zfs.rpoolbwwrite,sudo /sbin/zpool iostat rpool | stdbuf -o0 awk 'NR > 3 {print($7)}' | stdbuf -o0 sed -e 's/K/\*1024/g' -e 's/M/\*1048576/g' -e 's/G/\*1073741824/g' | bc | stdbuf -o0 awk '{printf("%d\n",$1 + 0.5)}'

- Restart zabbix agent

- It tested on Debian.
