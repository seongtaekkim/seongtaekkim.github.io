

~~~
[ec2-user@dev-was-0 ~]$ df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        1.9G     0  1.9G   0% /dev
tmpfs           1.9G     0  1.9G   0% /dev/shm
tmpfs           1.9G   43M  1.9G   3% /run
tmpfs           1.9G     0  1.9G   0% /sys/fs/cgroup
/dev/nvme0n1p1  100G   39G   61G  40% /
127.0.0.1:/     8.0E   29G  8.0E   1% /smartsuite
tmpfs           388M     0  388M   0% /run/user/1000
[ec2-user@dev-was-0 ~]$ cat /etc/fstab
#
UUID=e9c3e7c2-e039-4b2b-9991-955a8c7cd8c0     /           xfs    defaults,noatime  1   1
/swapfile swap swap defaults 0 0
fs-0d9d3c46f2926d4a4:/      /smartsuite                              efs    _netdev,tls,accesspoint=fsap-07b2ba2ee777ec9b6  0   0
~~~


~~~
[ec2-user@dev-was-0 ~]$ sudo netstat -tulpn
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 127.0.0.1:20526         0.0.0.0:*               LISTEN      2575/stunnel5
tcp        0      0 0.0.0.0:111             0.0.0.0:*               LISTEN      1965/rpcbind
tcp        0      0 0.0.0.0:40022           0.0.0.0:*               LISTEN      2526/sshd
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      31322/master
tcp6       0      0 :::8009                 :::*                    LISTEN      31975/java
tcp6       0      0 :::1099                 :::*                    LISTEN      5736/java
tcp6       0      0 :::111                  :::*                    LISTEN      1965/rpcbind
tcp6       0      0 :::8080                 :::*                    LISTEN      31975/java
tcp6       0      0 :::40022                :::*                    LISTEN      2526/sshd
tcp6       0      0 127.0.0.1:7005          :::*                    LISTEN      5736/java
tcp6       0      0 :::41341                :::*                    LISTEN      5736/java
tcp6       0      0 :::7070                 :::*                    LISTEN      5736/java
tcp6       0      0 127.0.0.1:8005          :::*                    LISTEN      31975/java
udp        0      0 0.0.0.0:68              0.0.0.0:*                           2187/dhclient
udp        0      0 0.0.0.0:111             0.0.0.0:*                           1965/rpcbind
udp        0      0 127.0.0.1:323           0.0.0.0:*                           1937/chronyd
udp        0      0 0.0.0.0:858             0.0.0.0:*                           1965/rpcbind
udp6       0      0 :::111                  :::*                                1965/rpcbind
udp6       0      0 ::1:323                 :::*                                1937/chronyd
udp6       0      0 fe80::67:6ff:fed7:f:546 :::*                                2294/dhclient
udp6       0      0 :::858                  :::*                                1965/rpcbind
~~~



~~~
sudo systemctl status amazon-ssm-agent.service
 sudo systemctl status amazon-efs-mount-watchdog.service
~~~





~~~
[ec2-user@dev-was-1 ~]$ sudo netstat -tulpn
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:111             0.0.0.0:*               LISTEN      1954/rpcbind
tcp        0      0 0.0.0.0:40022           0.0.0.0:*               LISTEN      13981/sshd
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      5764/master
tcp        0      0 127.0.0.1:20896         0.0.0.0:*               LISTEN      2765/stunnel5
tcp6       0      0 127.0.0.1:8005          :::*                    LISTEN      14373/java
tcp6       0      0 :::8009                 :::*                    LISTEN      14373/java
tcp6       0      0 :::1099                 :::*                    LISTEN      8318/java
tcp6       0      0 :::111                  :::*                    LISTEN      1954/rpcbind
tcp6       0      0 :::8080                 :::*                    LISTEN      14373/java
tcp6       0      0 :::40022                :::*                    LISTEN      13981/sshd
tcp6       0      0 127.0.0.1:7005          :::*                    LISTEN      8318/java
tcp6       0      0 :::7070                 :::*                    LISTEN      8318/java
tcp6       0      0 :::43647                :::*                    LISTEN      8318/java
udp        0      0 0.0.0.0:68              0.0.0.0:*                           2206/dhclient
udp        0      0 0.0.0.0:111             0.0.0.0:*                           1954/rpcbind
udp        0      0 127.0.0.1:323           0.0.0.0:*                           1956/chronyd
udp        0      0 0.0.0.0:838             0.0.0.0:*                           1954/rpcbind
udp6       0      0 :::111                  :::*                                1954/rpcbind
udp6       0      0 ::1:323                 :::*                                1956/chronyd
udp6       0      0 fe80::811:94ff:fe55:546 :::*                                2304/dhclient
udp6       0      0 :::838                  :::*                                1954/rpcbind
~~~


~~~
[ec2-user@dev-was-1 ~]$ df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        1.9G     0  1.9G   0% /dev
tmpfs           1.9G     0  1.9G   0% /dev/shm
tmpfs           1.9G   45M  1.9G   3% /run
tmpfs           1.9G     0  1.9G   0% /sys/fs/cgroup
/dev/nvme0n1p1  100G   37G   64G  37% /
127.0.0.1:/     8.0E   29G  8.0E   1% /smartsuite
tmpfs           388M     0  388M   0% /run/user/1000
[ec2-user@dev-was-1 ~]$ cat /etc/fstab
#
UUID=e9c3e7c2-e039-4b2b-9991-955a8c7cd8c0     /           xfs    defaults,noatime  1   1
/swapfile swap swap defaults 0 0
fs-0d9d3c46f2926d4a4:/      /smartsuite                              efs    _netdev,tls,accesspoint=fsap-07b2ba2ee777ec9b6  0   0
~~~











~~~


[Unit]
Description=Apache Tomcat emroCloud BIZ
After=network.target

[Service]
Type=forking

User=was
Group=was
UMask=0007


# 환경 변수 (필요 시)
Environment="JAVA_HOME=/usr/lib/jvm/java"
Environment="CATALINA_BASE=/usr/local/tomcat_dev"
Environment="CATALINA_HOME=/usr/local/tomcat_dev"

# 시작/중지 스크립트
ExecStart=/bin/bash /usr/local/tomcat_dev/bin/startup.sh
ExecStop=/bin/bash /usr/local/tomcat_dev/bin/shutdown.sh
SuccessExitStatus=143

# 보안 샌드박싱
Restart=on-failure               # 비정상 종료 시 자동 재가동
ProtectSystem=full               # /usr·/etc read-only
ProtectHome=yes                  # /home/* 차단
PrivateTmp=yes                   # 개별 /tmp
NoNewPrivileges=yes              # setuid 역이용 차단
LimitNOFILE=65535                # 열 수 있는 FD 상향


[Install]
WantedBy=multi-user.target

~~~


~~~

[Unit]
Description=Apache Tomcat emroCloud IF
After=network.target

[Service]
Type=forking

User=was
Group=was
UMask=0007


# 환경 변수 (필요 시)
Environment="JAVA_HOME=/usr/lib/jvm/java"
Environment="CATALINA_BASE=/usr/local/tomcat_if"
Environment="CATALINA_HOME=/usr/local/tomcat_if"

# 시작/중지 스크립트
ExecStart=/bin/bash /usr/local/tomcat_if/bin/startup.sh
ExecStop=/bin/bash /usr/local/tomcat_if/bin/shutdown.sh
SuccessExitStatus=143

# 보안 샌드박싱
Restart=on-failure               # 비정상 종료 시 자동 재가동
ProtectSystem=full               # /usr·/etc read-only
ProtectHome=yes                  # /home/* 차단
PrivateTmp=yes                   # 개별 /tmp
NoNewPrivileges=yes              # setuid 역이용 차단
LimitNOFILE=65535                # 열 수 있는 FD 상향


[Install]
WantedBy=multi-user.target

~~~


~~~

[was@dev-was-0 ~]$ ps -ef |grep tomcat
was       4445     1  1 Jul24 ?        00:09:36 /usr/bin/java -Djava.util.logging.config.file=/usr/local/tomcat_dev/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Djava.awt.headless=true -server -Xms1024m -Xmx1024m -XX:NewSize=128m -XX:MaxNewSize=128m -XX:PermSize=128m -XX:MaxPermSize=128m -XX:+DisableExplicitGC -Dignore.endorsed.dirs= -classpath /usr/local/tomcat_dev/bin/bootstrap.jar:/usr/local/tomcat_dev/bin/tomcat-juli.jar -Dcatalina.base=/usr/local/tomcat_dev -Dcatalina.home=/usr/local/tomcat_dev -Djava.io.tmpdir=/usr/local/tomcat_dev/temp org.apache.catalina.startup.Bootstrap start
df
was       4723     1  0 Jul24 ?        00:05:33 /usr/bin/java -Djava.util.logging.config.file=/usr/local/tomcat_if/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Djdk.tls.ephemeralDHKeySize=2048 -Djava.protocol.handler.pkgs=org.apache.catalina.webresources -Dorg.apache.catalina.security.SecurityListener.UMASK=0027 -Dignore.endorsed.dirs= -classpath /usr/localtomcat_if/bin/bootstrap.jar:/usr/local/tomcat_if/bin/tomcat-juli.jar -Dcatalina.base=/usr/local/tomcat_if -Dcatalina.home=/usr/local/tomcat_if -Djava.io.tmpdir=/usr/local/tomcat_if/temp org.apache.catalina.startup.Bootstrap start
root     28860 28742  0 04:43 pts/2    00:00:00 grep --color=auto tomcat
~~~



~~~
sudo systemctl start tomcat_dev.service

~~~