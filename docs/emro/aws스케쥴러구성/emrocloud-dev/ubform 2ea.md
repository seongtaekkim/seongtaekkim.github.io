종료시간 2~3분
실행시간 2~3분

서비스등록: 톰캣
마운트: efs

서비스확인
https://seahsteel.dev.emrocloud.com/UBIFORM/UView5/index.jsp?formName=gr_rpt_info&projectName=SEAHSTEEL

~~~
[ec2-user@ip-10-100-2-199 ~]$ sudo netstat -tulpn
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:40022           0.0.0.0:*               LISTEN      2162/sshd: /usr/sbi
tcp        0      0 127.0.0.1:20588         0.0.0.0:*               LISTEN      2201/stunnel
tcp6       0      0 127.0.0.1:8005          :::*                    LISTEN      2014/java
tcp6       0      0 :::40022                :::*                    LISTEN      2162/sshd: /usr/sbi
tcp6       0      0 :::8080                 :::*                    LISTEN      2014/java
udp        0      0 127.0.0.1:323           0.0.0.0:*                           2190/chronyd
udp        0      0 10.100.2.199:68         0.0.0.0:*                           1967/systemd-networ
udp6       0      0 ::1:323                 :::*                                2190/chronyd
udp6       0      0 fe80::4:5cff:feb8:d:546 :::*                                1967/systemd-networ
~~~



~~~
[ec2-user@ip-10-100-2-199 ~]$ df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        4.0M     0  4.0M   0% /dev
tmpfs           475M     0  475M   0% /dev/shm
tmpfs           190M   20M  171M  11% /run
/dev/xvda1       10G  5.6G  4.4G  56% /
tmpfs           475M   32K  475M   1% /tmp
/dev/xvda128     10M  1.3M  8.7M  13% /boot/efi
127.0.0.1:/     8.0E  363M  8.0E   1% /data/ubstorm2
tmpfs            95M     0   95M   0% /run/user/1000
~~~
~~~
[ec2-user@ip-10-100-2-199 ~]$ cat /etc/fstab
#
UUID=3d668946-b9eb-4d6b-acbd-da437d796ee8     /           xfs    defaults,noatime  1   1
UUID=1753-28B3        /boot/efi       vfat    defaults,noatime,uid=0,gid=0,umask=0077,shortname=winnt,x-systemd.automount 0 2
fs-005b11b94f799366e:/ /data/ubstorm2 efs _netdev,tls,accesspoint=fsap-00e113af31fd53776 0 0
/swapfile swap swap defaults 0 0
~~~

~~~
sudo systemctl is-enabled tomcat
~~~





~~~
[ec2-user@ip-10-100-3-199 ~]$ sudo netstat -tulnp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 127.0.0.1:20130         0.0.0.0:*               LISTEN      2241/stunnel
tcp        0      0 0.0.0.0:40022           0.0.0.0:*               LISTEN      2191/sshd: /usr/sbi
tcp        0      0 127.0.0.1:20885         0.0.0.0:*               LISTEN      2242/stunnel
tcp6       0      0 :::40022                :::*                    LISTEN      2191/sshd: /usr/sbi
tcp6       0      0 :::8080                 :::*                    LISTEN      3800/java
tcp6       0      0 127.0.0.1:8005          :::*                    LISTEN      3800/java
udp        0      0 10.100.3.199:68         0.0.0.0:*                           1966/systemd-networ
udp        0      0 127.0.0.1:323           0.0.0.0:*                           2223/chronyd
udp6       0      0 fe80::8a8:3cff:fe83:546 :::*                                1966/systemd-networ
udp6       0      0 ::1:323                 :::*                                2223/chronyd
~~~



~~~
[ec2-user@ip-10-100-3-199 ~]$ cat /etc/fstab
#
UUID=3d668946-b9eb-4d6b-acbd-da437d796ee8     /           xfs    defaults,noatime  1   1
UUID=1753-28B3        /boot/efi       vfat    defaults,noatime,uid=0,gid=0,umask=0077,shortname=winnt,x-systemd.automount 0 2
fs-005b11b94f799366e:/ /data/ubstorm2 efs _netdev,tls,accesspoint=fsap-00e113af31fd53776 0 0
/swapfile swap swap defaults 0 0
~~~


~~~
[ec2-user@ip-10-100-3-199 ~]$ df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        4.0M     0  4.0M   0% /dev
tmpfs           475M     0  475M   0% /dev/shm
tmpfs           190M   37M  154M  20% /run
/dev/xvda1       10G  5.6G  4.4G  57% /
tmpfs           475M   32K  475M   1% /tmp
/dev/xvda128     10M  1.3M  8.7M  13% /boot/efi
127.0.0.1:/     8.0E   29G  8.0E   1% /data/ubstorm
127.0.0.1:/     8.0E  363M  8.0E   1% /data/ubstorm2
tmpfs            95M     0   95M   0% /run/user/1000
~~~