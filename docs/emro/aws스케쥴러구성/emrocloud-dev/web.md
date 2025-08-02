

서비스등록 jenkins httpd dataog-agent

~~~
systemctl is-enabled datadog-agent
systemctl is-enabled httpd
systemctl is-enabled jenkins

~~~

~~~
[ec2-user@emrocloud-dev-web ~]$ sudo netstat -tulpn
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 127.0.0.1:6062          0.0.0.0:*               LISTEN      28177/process-agent
tcp        0      0 0.0.0.0:111             0.0.0.0:*               LISTEN      1899/rpcbind
tcp        0      0 127.0.0.1:6162          0.0.0.0:*               LISTEN      28177/process-agent
tcp        0      0 127.0.0.1:5012          0.0.0.0:*               LISTEN      28178/trace-agent
tcp        0      0 0.0.0.0:40022           0.0.0.0:*               LISTEN      9928/sshd
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      10071/master
tcp        0      0 127.0.0.1:8126          0.0.0.0:*               LISTEN      28178/trace-agent
tcp        0      0 127.0.0.1:32000         0.0.0.0:*               LISTEN      32444/java
tcp        0      0 127.0.0.1:5000          0.0.0.0:*               LISTEN      28176/agent
tcp        0      0 127.0.0.1:5001          0.0.0.0:*               LISTEN      28176/agent
tcp6       0      0 :::3343                 :::*                    LISTEN      32444/java
tcp6       0      0 :::111                  :::*                    LISTEN      1899/rpcbind
tcp6       0      0 :::80                   :::*                    LISTEN      1135/httpd
tcp6       0      0 :::8081                 :::*                    LISTEN      6447/java
tcp6       0      0 :::4434                 :::*                    LISTEN      32444/java
tcp6       0      0 :::40022                :::*                    LISTEN      9928/sshd
tcp6       0      0 :::3690                 :::*                    LISTEN      4731/httpd
udp        0      0 127.0.0.1:8125          0.0.0.0:*                           28176/agent
udp        0      0 0.0.0.0:68              0.0.0.0:*                           2125/dhclient
udp        0      0 0.0.0.0:111             0.0.0.0:*                           1899/rpcbind
udp        0      0 127.0.0.1:323           0.0.0.0:*                           10264/chronyd
udp        0      0 0.0.0.0:802             0.0.0.0:*                           1899/rpcbind
udp6       0      0 :::111                  :::*                                1899/rpcbind
udp6       0      0 ::1:323                 :::*                                10264/chronyd
udp6       0      0 fe80::e:85ff:feb6:3:546 :::*                                2259/dhclient
udp6       0      0 :::802                  :::*                                1899/rpcbind
~~~



~~~
[ec2-user@emrocloud-dev-web ~]$ ps -efww
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0  2023 ?        00:52:54 /usr/lib/systemd/systemd --system --deserialize 23
root         2     0  0  2023 ?        00:00:09 [kthreadd]
root         4     2  0  2023 ?        00:00:00 [kworker/0:0H]
root         6     2  0  2023 ?        00:00:00 [mm_percpu_wq]
root         7     2  0  2023 ?        00:19:06 [ksoftirqd/0]
root         8     2  0  2023 ?        01:54:29 [rcu_sched]
root         9     2  0  2023 ?        00:00:00 [rcu_bh]
root        10     2  0  2023 ?        00:04:04 [migration/0]
root        11     2  0  2023 ?        00:01:06 [watchdog/0]
root        12     2  0  2023 ?        00:00:00 [cpuhp/0]
root        13     2  0  2023 ?        00:00:00 [cpuhp/1]
root        14     2  0  2023 ?        00:01:21 [watchdog/1]
root        15     2  0  2023 ?        00:04:21 [migration/1]
root        16     2  0  2023 ?        00:24:30 [ksoftirqd/1]
root        18     2  0  2023 ?        00:00:00 [kworker/1:0H]
root        19     2  0  2023 ?        00:00:00 [kdevtmpfs]
root        20     2  0  2023 ?        00:00:00 [netns]
root       115     2  0  2023 ?        00:00:31 [khungtaskd]
root       178     2  0  2023 ?        00:00:00 [oom_reaper]
root       179     2  0  2023 ?        00:00:00 [writeback]
root       180     2  0  2023 ?        00:00:32 [kcompactd0]
root       182     2  0  2023 ?        00:00:00 [ksmd]
root       183     2  0  2023 ?        00:07:13 [khugepaged]
root       184     2  0  2023 ?        00:00:00 [crypto]
root       185     2  0  2023 ?        00:00:00 [kintegrityd]
root       186     2  0  2023 ?        00:00:00 [kblockd]
root       295     2  0  2023 ?        00:00:00 [md]
root       298     2  0  2023 ?        00:00:00 [edac-poller]
root       346     2  0 16:10 ?        00:00:00 [kworker/1:2]
root       349     2  0 16:10 ?        00:00:00 [kworker/0:0]
root       410     2  0  2023 ?        00:00:29 [kauditd]
root       416     2  0  2023 ?        00:46:31 [kswapd0]
root       548     2  0  2023 ?        00:00:00 [kthrotld]
root       593     2  0  2023 ?        00:00:00 [kstrp]
root       620     2  0  2023 ?        00:00:00 [ipv6_addrconf]
apache    1135 31474  0 08:15 ?        00:00:05 /usr/sbin/httpd -DFOREGROUND
root      1212     2  0  2023 ?        00:00:00 [nvme-wq]
root      1321     2  0  2023 ?        00:00:00 [xfsalloc]
root      1322     2  0  2023 ?        00:00:00 [xfs_mru_cache]
root      1324     2  0  2023 ?        00:00:00 [xfs-buf/nvme0n1]
root      1325     2  0  2023 ?        00:00:00 [xfs-data/nvme0n]
root      1326     2  0  2023 ?        00:00:00 [xfs-conv/nvme0n]
root      1327     2  0  2023 ?        00:00:00 [xfs-cil/nvme0n1]
root      1328     2  0  2023 ?        00:00:00 [xfs-reclaim/nvm]
root      1329     2  0  2023 ?        00:00:00 [xfs-log/nvme0n1]
root      1330     2  0  2023 ?        00:00:00 [xfs-eofblocks/n]
root      1331     2  0  2023 ?        02:29:33 [xfsaild/nvme0n1]
root      1332     2  0  2023 ?        00:04:46 [kworker/0:1H]
root      1396     1  0  2023 ?        02:11:47 /usr/lib/systemd/systemd-journald
root      1409     2  0  2023 ?        00:00:00 [ena]
root      1780     2  0  2023 ?        00:05:57 [kworker/1:1H]
root      1870     2  0  2023 ?        00:00:00 [rpciod]
root      1871     2  0  2023 ?        00:00:00 [xprtiod]
root      1874     1  0  2023 ?        00:01:54 /sbin/auditd
rpc       1899     1  0  2023 ?        00:00:53 /sbin/rpcbind -w
dbus      1909     1  0  2023 ?        00:45:04 /usr/bin/dbus-daemon --system --address=systemd: --nofork --nopidfile --systemd-activation
libstor+  1914     1  0  2023 ?        00:01:28 /usr/bin/lsmd -d
root      1915     1  0  2023 ?        00:21:11 /usr/lib/systemd/systemd-logind
root      1931     1  0  2023 ?        00:00:00 /usr/sbin/gssproxy -D
root      2125     1  0  2023 ?        00:00:27 /sbin/dhclient -q -lf /var/lib/dhclient/dhclient--eth0.lease -pf /var/run/dhclient-eth0.pid -H ip-10-100-0-100 eth0
root      2259     1  0  2023 ?        00:01:19 /sbin/dhclient -6 -nw -lf /var/lib/dhclient/dhclient6--eth0.lease -pf /var/run/dhclient6-eth0.pid eth0 -H ip-10-100-0-100
root      2445  9928  0 16:13 ?        00:00:00 sshd: ec2-user [priv]
ec2-user  2462  2445  0 16:13 ?        00:00:00 sshd: ec2-user@pts/0
ec2-user  2463  2462  0 16:13 pts/0    00:00:00 -bash
root      2564     1  0  2023 ttyS0    00:00:00 /sbin/agetty --keep-baud 115200,38400,9600 ttyS0 vt220
root      2565     1  0  2023 tty1     00:00:00 /sbin/agetty --noclear tty1 linux
root      3831     2  0 16:15 ?        00:00:00 [kworker/1:1]
csvn      4731 16351  0 Jun23 ?        00:00:41 /usr/local/csvn/bin/httpd -f /usr/local/csvn/data/conf/httpd.conf -k start
root      5441     2  0 Jul19 ?        00:00:02 [kworker/u4:0]
root      5985     2  0 16:19 ?        00:00:00 [kworker/u4:1]
root      6421     1  0  2024 ?        00:00:00 /usr/lib/systemd/systemd-udevd
jenkins   6447     1  0 Jun23 ?        01:10:50 /usr/bin/java -Djava.awt.headless=true -jar /usr/share/java/jenkins.war --webroot=%C/jenkins/war --httpPort=8081 --prefix=/jenkins
postfix   6582 10071  0 16:20 ?        00:00:00 cleanup -z -t unix -u
postfix   6584 10071  0 16:20 ?        00:00:00 trivial-rewrite -n rewrite -t unix -u
postfix   6586 10071  0 16:20 ?        00:00:00 local -t unix
root      6587     2  0 16:20 ?        00:00:00 [kworker/0:1]
postfix   6588 10071  0 16:20 ?        00:00:00 bounce -z -t unix -u
root      6589     2  0 16:20 ?        00:00:00 [kworker/1:3]
root      6607 27054  0 16:20 ?        00:00:00 sleep 1
ec2-user  6608  2463  0 16:20 pts/0    00:00:00 ps -efww
apache    6786 31474  0 15:27 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache    6940 31474  0 15:28 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache    7009 31474  0 15:28 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache    7084 31474  0 15:28 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache    7408 31474  0 15:28 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache    7423 31474  0 15:28 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
csvn      9506 16351  0 Jun10 ?        00:04:44 /usr/local/csvn/bin/httpd -f /usr/local/csvn/data/conf/httpd.conf -k start
csvn      9888 16351  0 Jul19 ?        00:00:05 /usr/local/csvn/bin/httpd -f /usr/local/csvn/data/conf/httpd.conf -k start
csvn      9889 16351  0 Jul19 ?        00:00:05 /usr/local/csvn/bin/httpd -f /usr/local/csvn/data/conf/httpd.conf -k start
root      9928     1  0  2024 ?        00:00:00 /usr/sbin/sshd -D
root     10071     1  0  2024 ?        00:03:53 /usr/libexec/postfix/master -w
postfix  10073 10071  0  2024 ?        00:01:06 qmgr -l -t unix -u
root     10126     1  0  2024 ?        00:00:00 /usr/sbin/lvmetad -f
root     10207     1  0  2024 ?        01:22:15 /usr/sbin/rsyslogd -n
chrony   10264     1  0  2024 ?        00:13:37 /usr/sbin/chronyd -F 2
root     10318     1  0  2024 ?        00:01:28 /usr/sbin/crond -n
root     10349     1  0  2024 ?        00:19:43 /usr/sbin/irqbalance --foreground
root     10474     1  0  2024 ?        00:00:00 /usr/sbin/atd -f
rngd     10503     1  0  2024 ?        00:14:11 /sbin/rngd -f --fill-watermark=0 --exclude=jitter
root     10643     1  0  2024 ?        00:33:11 /usr/bin/amazon-ssm-agent
root     10693 10643  0  2024 ?        06:28:04 /usr/bin/ssm-agent-worker
root     12609     2  0 03:17 ?        00:00:01 [kworker/u4:2]
root     13193 31474  0 Jul20 ?        00:00:02 /usr/sbin/rotatelogs -l logs/access_log.%Y%m%d 86400
root     13194 31474  0 Jul20 ?        00:00:46 /usr/sbin/rotatelogs -l logs/mod_jk.log.%Y%m%d 86400
csvn     13257 16351  0 Jun11 ?        00:01:16 /usr/local/csvn/bin/httpd -f /usr/local/csvn/data/conf/httpd.conf -k start
root     16351     1  0 Mar14 ?        00:05:26 /usr/local/csvn/bin/httpd -f /usr/local/csvn/data/conf/httpd.conf -k start
root     16352 16351  0 Mar14 ?        00:00:00 /usr/local/csvn/bin/rotatelogs -l /usr/local/csvn/data/logs/error_%Y_%m_%d_%H_%M_%S.log 86400
root     16353 16351  0 Mar14 ?        00:01:22 /usr/local/csvn/bin/rotatelogs -l /usr/local/csvn/data/logs/access_%Y_%m_%d_%H_%M_%S.log 86400
root     16354 16351  0 Mar14 ?        00:00:09 /usr/local/csvn/bin/rotatelogs -l /usr/local/csvn/data/logs/subversion_%Y_%m_%d_%H_%M_%S.log 86400
postfix  17716 10071  0 14:53 ?        00:00:00 pickup -l -t unix -u
root     20390     2  0 15:50 ?        00:00:00 [kworker/0:2]
csvn     22196 16351  0 Jul16 ?        00:00:07 /usr/local/csvn/bin/httpd -f /usr/local/csvn/data/conf/httpd.conf -k start
csvn     22206 16351  0 Jul16 ?        00:00:06 /usr/local/csvn/bin/httpd -f /usr/local/csvn/data/conf/httpd.conf -k start
csvn     24831 16351  0 Jun11 ?        00:05:01 /usr/local/csvn/bin/httpd -f /usr/local/csvn/data/conf/httpd.conf -k start
apache   25056 31474  0 08:54 ?        00:00:04 /usr/sbin/httpd -DFOREGROUND
csvn     26422 16351  0 Jun11 ?        00:02:00 /usr/local/csvn/bin/httpd -f /usr/local/csvn/data/conf/httpd.conf -k start
root     27054     1  0 16:01 ?        00:00:00 /bin/bash /usr/bin/log4j-cve-2021-44228-hotpatch -w 1800 -m 10
root     28175     1  0  2024 ?        1-09:09:15 /opt/datadog-agent/embedded/bin/system-probe run --config=/etc/datadog-agent/system-probe.yaml --pid=/opt/datadog-agent/run/system-probe.pid
dd-agent 28176     1  0  2024 ?        3-06:12:31 /opt/datadog-agent/bin/agent/agent run -p /opt/datadog-agent/run/agent.pid
dd-agent 28177     1  0  2024 ?        1-20:11:21 /opt/datadog-agent/embedded/bin/process-agent --cfgpath=/etc/datadog-agent/datadog.yaml --sysprobe-config=/etc/datadog-agent/system-probe.yaml --pid=/opt/datadog-agent/run/process-agent.pid
dd-agent 28178     1  0  2024 ?        1-07:30:35 /opt/datadog-agent/embedded/bin/trace-agent --config /etc/datadog-agent/datadog.yaml --pidfile /opt/datadog-agent/run/trace-agent.pid
root     30499     2  0 16:06 ?        00:00:00 [kworker/1:0]
root     31474     1  0  2024 ?        00:28:21 /usr/sbin/httpd -DFOREGROUND
apache   31798 31474  0 06:26 ?        00:00:06 /usr/sbin/httpd -DFOREGROUND
apache   31824 31474  0 06:26 ?        00:00:06 /usr/sbin/httpd -DFOREGROUND
csvn     31910 16351  0 Jun10 ?        00:03:55 /usr/local/csvn/bin/httpd -f /usr/local/csvn/data/conf/httpd.conf -k start
csvn     32442     1  0 Jun24 ?        00:27:52 /usr/local/csvn/bin/./wrapper-linux-x86-64 /usr/local/csvn/bin/../data/conf/csvn-wrapper.conf wrapper.syslog.ident=csvn wrapper.pidfile=/usr/local/csvn/bin/../data/run/csvn.pid wrapper.name=csvn wrapper.displayname=CSVN Console wrapper.daemonize=TRUE wrapper.statusfile=/usr/local/csvn/bin/../data/run/csvn.status wrapper.java.statusfile=/usr/local/csvn/bin/../data/run/csvn.java.status wrapper.script.version=3.5.26
csvn     32444 32442  0 Jun24 ?        01:04:43 /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.392.b08-2.amzn2.0.1.x86_64/bin/java -XX:MaxPermSize=128m -Djetty.home=../appserver -Djetty.port=3343 -Djetty.ssl.port=4434 -Xms64m -Xmx512m -Djava.library.path=../lib -classpath ../lib/wrapper.jar -Dwrapper.key=Cf_9USDYKRi_y-8T -Dwrapper.port=32000 -Dwrapper.disable_console_input=TRUE -Dwrapper.pid=32442 -Dwrapper.version=3.5.26 -Dwrapper.native_library=wrapper -Dwrapper.arch=x86 -Dwrapper.service=TRUE -Dwrapper.cpu.timeout=10 -Dwrapper.jvmid=1 org.tanukisoftware.wrapper.WrapperJarApp ../appserver/start.jar
~~~



~~~
[ec2-user@emrocloud-dev-web ~]$ df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        1.9G     0  1.9G   0% /dev
tmpfs           1.9G  520K  1.9G   1% /run
tmpfs           1.9G     0  1.9G   0% /sys/fs/cgroup
/dev/nvme0n1p1  100G   59G   42G  59% /

~~~





![[Pasted image 20250725192857.png]]


![[Pasted image 20250725193420.png]]



~~~
/usr/local/csvn/bin/httpd -f /usr/local/csvn/data/conf/httpd.conf -k start
~~~



~~~

sudo /home/jenkins/svn/csvn/bin/csvn start

sudo /usr/local/csvn/bin/csvn start
~~~