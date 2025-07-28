

~~~
[ec2-user@emrocloud-dev-elasticsearch ~]$ sudo netstat -tulpn
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      2476/master
tcp        0      0 10.100.2.200:5601       0.0.0.0:*               LISTEN      9034/node
tcp        0      0 0.0.0.0:111             0.0.0.0:*               LISTEN      1987/rpcbind
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      18595/sshd
tcp6       0      0 :::7673                 :::*                    LISTEN      2923/java
tcp6       0      0 127.0.0.1:7675          :::*                    LISTEN      2923/java
tcp6       0      0 :::111                  :::*                    LISTEN      1987/rpcbind
tcp6       0      0 :::9200                 :::*                    LISTEN      8562/java
tcp6       0      0 :::9300                 :::*                    LISTEN      8562/java
tcp6       0      0 :::22                   :::*                    LISTEN      18595/sshd
udp        0      0 0.0.0.0:68              0.0.0.0:*                           2219/dhclient
udp        0      0 0.0.0.0:111             0.0.0.0:*                           1987/rpcbind
udp        0      0 127.0.0.1:323           0.0.0.0:*                           1993/chronyd
udp        0      0 0.0.0.0:883             0.0.0.0:*                           1987/rpcbind
udp6       0      0 :::111                  :::*                                1987/rpcbind
udp6       0      0 ::1:323                 :::*                                1993/chronyd
udp6       0      0 fe80::af:e0ff:fe26::546 :::*                                2332/dhclient
udp6       0      0 :::883                  :::*                                1987/rpcbind
~~~
~~~
 sudo systemctl status elasticsearch.service
 $ sudo systemctl status kibana.service
~~~



7673 포트 서비스
~~~
/home/ec2-user/search/bin


admin-webapp-shutdown.sh  admin-webapp-startup.sh

~~~