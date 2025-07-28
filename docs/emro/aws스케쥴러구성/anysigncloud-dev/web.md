



#### filesystem
~~~
[ec2-user@anysign-web ~]$ df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        1.9G     0  1.9G   0% /dev
tmpfs           1.9G   12K  1.9G   1% /dev/shm
tmpfs           1.9G  250M  1.7G  13% /run
tmpfs           1.9G     0  1.9G   0% /sys/fs/cgroup
/dev/nvme0n1p1  100G   55G   46G  55% /
tmpfs           389M     0  389M   0% /run/user/1000
~~~



#### process info
~~~

[ec2-user@anysign-web ~]$ sudo netstat -tulpn
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 127.0.0.1:9229          0.0.0.0:*               LISTEN      2662/gitlab-workhor
tcp        0      0 0.0.0.0:3342            0.0.0.0:*               LISTEN      2663/nginx: master
tcp        0      0 0.0.0.0:111             0.0.0.0:*               LISTEN      1909/rpcbind
tcp        0      0 127.0.0.1:9168          0.0.0.0:*               LISTEN      14592/puma 4.3.5.gi
tcp        0      0 127.0.0.1:8080          0.0.0.0:*               LISTEN      2658/puma 4.3.5.git
tcp        0      0 127.0.0.1:8082          0.0.0.0:*               LISTEN      2760/sidekiq 5.2.9
tcp        0      0 127.0.0.1:9236          0.0.0.0:*               LISTEN      2694/gitaly
tcp        0      0 0.0.0.0:40022           0.0.0.0:*               LISTEN      2479/sshd
tcp        0      0 127.0.0.1:3000          0.0.0.0:*               LISTEN      2669/grafana-server
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      2443/master
tcp        0      0 0.0.0.0:8060            0.0.0.0:*               LISTEN      2663/nginx: master
tcp        0      0 127.0.0.1:32000         0.0.0.0:*               LISTEN      15533/java
tcp        0      0 127.0.0.1:9121          0.0.0.0:*               LISTEN      2674/redis_exporter
tcp        0      0 127.0.0.1:9090          0.0.0.0:*               LISTEN      2671/prometheus
tcp        0      0 127.0.0.1:9187          0.0.0.0:*               LISTEN      2677/postgres_expor
tcp        0      0 127.0.0.1:9093          0.0.0.0:*               LISTEN      2679/alertmanager
tcp        0      0 127.0.0.1:9100          0.0.0.0:*               LISTEN      2665/node_exporter
tcp6       0      0 :::3343                 :::*                    LISTEN      15533/java
tcp6       0      0 :::111                  :::*                    LISTEN      1909/rpcbind
tcp6       0      0 ::1:9168                :::*                    LISTEN      14592/puma 4.3.5.gi
tcp6       0      0 :::80                   :::*                    LISTEN      6260/httpd
tcp6       0      0 :::8081                 :::*                    LISTEN      2570/java
tcp6       0      0 :::4434                 :::*                    LISTEN      15533/java
tcp6       0      0 :::40022                :::*                    LISTEN      2479/sshd
tcp6       0      0 :::9094                 :::*                    LISTEN      2679/alertmanager
tcp6       0      0 :::3690                 :::*                    LISTEN      5896/httpd
udp        0      0 0.0.0.0:68              0.0.0.0:*                           2125/dhclient
udp        0      0 0.0.0.0:111             0.0.0.0:*                           1909/rpcbind
udp        0      0 127.0.0.1:323           0.0.0.0:*                           1922/chronyd
udp        0      0 0.0.0.0:802             0.0.0.0:*                           1909/rpcbind
udp6       0      0 :::5353                 :::*                                2570/java
udp6       0      0 :::111                  :::*                                1909/rpcbind
udp6       0      0 ::1:323                 :::*                                1922/chronyd
udp6       0      0 fe80::7e:28ff:fedc::546 :::*                                2254/dhclient
udp6       0      0 :::802                  :::*                                1909/rpcbind
udp6       0      0 :::9094                 :::*                                2679/alertmanager
udp6       0      0 :::33848                :::*                                2570/java

~~~




#### process cmd
~~~

[ec2-user@anysign-web ~]$ ps -efww
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0  2021 ?        04:42:47 /usr/lib/systemd/systemd --switched-root --system --deserialize 22
root         2     0  0  2021 ?        00:00:21 [kthreadd]
root         4     2  0  2021 ?        00:00:00 [kworker/0:0H]
root         6     2  0  2021 ?        00:00:00 [mm_percpu_wq]
root         7     2  0  2021 ?        01:11:17 [ksoftirqd/0]
root         8     2  0  2021 ?        10:42:15 [rcu_sched]
root         9     2  0  2021 ?        00:00:00 [rcu_bh]
root        10     2  0  2021 ?        00:00:56 [migration/0]
root        11     2  0  2021 ?        00:02:47 [watchdog/0]
root        12     2  0  2021 ?        00:00:00 [cpuhp/0]
root        13     2  0  2021 ?        00:00:00 [cpuhp/1]
root        14     2  0  2021 ?        00:03:00 [watchdog/1]
root        15     2  0  2021 ?        00:00:50 [migration/1]
root        16     2  0  2021 ?        01:05:14 [ksoftirqd/1]
root        18     2  0  2021 ?        00:00:00 [kworker/1:0H]
root        19     2  0  2021 ?        00:00:00 [kdevtmpfs]
root        20     2  0  2021 ?        00:00:00 [netns]
root       111     2  0  2021 ?        00:02:12 [khungtaskd]
root       179     2  0  2021 ?        00:00:00 [oom_reaper]
root       180     2  0  2021 ?        00:00:00 [writeback]
root       182     2  0  2021 ?        00:00:02 [kcompactd0]
root       183     2  0  2021 ?        00:00:00 [ksmd]
root       184     2  0  2021 ?        01:01:47 [khugepaged]
root       185     2  0  2021 ?        00:00:00 [crypto]
root       186     2  0  2021 ?        00:00:00 [kintegrityd]
root       188     2  0  2021 ?        00:00:00 [kblockd]
root       293     2  0  2021 ?        00:00:00 [md]
root       296     2  0  2021 ?        00:00:00 [edac-poller]
root       408     2  0  2021 ?        00:00:30 [kauditd]
root       414     2  0  2021 ?        02:55:16 [kswapd0]
root       546     2  0  2021 ?        00:00:00 [kthrotld]
root       591     2  0  2021 ?        00:00:00 [kstrp]
root       618     2  0  2021 ?        00:00:00 [ipv6_addrconf]
root      1203     2  0  2021 ?        00:00:00 [nvme-wq]
root      1316     2  0  2021 ?        00:00:00 [xfsalloc]
root      1317     2  0  2021 ?        00:00:00 [xfs_mru_cache]
root      1319     2  0  2021 ?        00:00:00 [xfs-buf/nvme0n1]
root      1320     2  0  2021 ?        00:00:00 [xfs-data/nvme0n]
root      1321     2  0  2021 ?        00:00:00 [xfs-conv/nvme0n]
root      1322     2  0  2021 ?        00:00:00 [xfs-cil/nvme0n1]
root      1323     2  0  2021 ?        00:00:00 [xfs-reclaim/nvm]
root      1324     2  0  2021 ?        00:00:00 [xfs-log/nvme0n1]
root      1325     2  0  2021 ?        00:00:00 [xfs-eofblocks/n]
root      1326     2  0  2021 ?        05:50:19 [xfsaild/nvme0n1]
root      1327     2  0  2021 ?        00:10:09 [kworker/1:1H]
root      1392     1  0  2021 ?        00:31:08 /usr/lib/systemd/systemd-journald
root      1405     2  0  2021 ?        00:00:00 [ena]
root      1407     1  0  2021 ?        00:00:00 /usr/sbin/lvmetad -f
root      1411     1  0  2021 ?        00:00:00 /usr/lib/systemd/systemd-udevd
root      1700     1  0  2022 ?        13:05:57 /home/jenkins/svn/csvn/bin/./wrapper-linux-x86-64 /home/jenkins/svn/csvn/bin/../data/conf/csvn-wrapper.conf wrapper.syslog.ident=csvn wrapper.pidfile=/home/jenkins/svn/csvn/bin/../data/run/csvn.pid wrapper.name=csvn wrapper.displayname=CSVN Console wrapper.daemonize=TRUE wrapper.statusfile=/home/jenkins/svn/csvn/bin/../data/run/csvn.status wrapper.java.statusfile=/home/jenkins/svn/csvn/bin/../data/run/csvn.java.status wrapper.lockfile=/var/lock/subsys/csvn wrapper.script.version=3.5.26
jenkins   1730  2570  0  2022 ?        5-11:21:40 /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.282.b08-1.amzn2.0.1.x86_64/bin/java -Xmx1024m -XX:MaxPermSize=512m -cp /var/lib/jenkins/plugins/maven-plugin/WEB-INF/lib/maven35-agent-1.12-alpha-1.jar:/var/lib/jenkins/tools/hudson.tasks.Maven_MavenInstallation/maven_3.6.0/boot/plexus-classworlds-2.5.2.jar:/var/lib/jenkins/tools/hudson.tasks.Maven_MavenInstallation/maven_3.6.0/conf/logging jenkins.maven3.agent.Maven35Main /var/lib/jenkins/tools/hudson.tasks.Maven_MavenInstallation/maven_3.6.0 /var/cache/jenkins/war/WEB-INF/lib/remoting-3.27.jar /var/lib/jenkins/plugins/maven-plugin/WEB-INF/lib/maven35-interceptor-1.12-alpha-1.jar /var/lib/jenkins/plugins/maven-plugin/WEB-INF/lib/maven3-interceptor-commons-1.12-alpha-1.jar 39525
jenkins   1731  2570  0  2022 ?        5-11:23:21 /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.282.b08-1.amzn2.0.1.x86_64/bin/java -Xmx1024m -XX:MaxPermSize=512m -cp /var/lib/jenkins/plugins/maven-plugin/WEB-INF/lib/maven35-agent-1.12-alpha-1.jar:/var/lib/jenkins/tools/hudson.tasks.Maven_MavenInstallation/maven_3.6.0/boot/plexus-classworlds-2.5.2.jar:/var/lib/jenkins/tools/hudson.tasks.Maven_MavenInstallation/maven_3.6.0/conf/logging jenkins.maven3.agent.Maven35Main /var/lib/jenkins/tools/hudson.tasks.Maven_MavenInstallation/maven_3.6.0 /var/cache/jenkins/war/WEB-INF/lib/remoting-3.27.jar /var/lib/jenkins/plugins/maven-plugin/WEB-INF/lib/maven35-interceptor-1.12-alpha-1.jar /var/lib/jenkins/plugins/maven-plugin/WEB-INF/lib/maven3-interceptor-commons-1.12-alpha-1.jar 36101
root      1847     2  0  2021 ?        00:07:47 [kworker/0:1H]
root      1870     2  0  2021 ?        00:00:00 [rpciod]
root      1871     2  0  2021 ?        00:00:00 [xprtiod]
root      1874     1  0  2021 ?        00:02:04 /sbin/auditd
root      1900     1  0  2021 ?        00:43:37 /usr/sbin/irqbalance --foreground --hintpolicy=subset
libstor+  1902     1  0  2021 ?        00:03:20 /usr/bin/lsmd -d
root      1903     1  0  2021 ?        00:28:46 /sbin/rngd -f
dbus      1908     1  0  2021 ?        00:54:53 /usr/bin/dbus-daemon --system --address=systemd: --nofork --nopidfile --systemd-activation
rpc       1909     1  0  2021 ?        00:02:07 /sbin/rpcbind -w
root      1914     1  0  2021 ?        00:00:00 /usr/sbin/gssproxy -D
chrony    1922     1  0  2021 ?        00:02:25 /usr/sbin/chronyd
root      1925     1  0  2021 ?        00:26:21 /usr/lib/systemd/systemd-logind
root      2125     1  0  2021 ?        00:01:12 /sbin/dhclient -q -lf /var/lib/dhclient/dhclient--eth0.lease -pf /var/run/dhclient-eth0.pid -H anysign-web eth0
root      2254     1  0  2021 ?        00:02:48 /sbin/dhclient -6 -nw -lf /var/lib/dhclient/dhclient6--eth0.lease -pf /var/run/dhclient6-eth0.pid eth0 -H anysign-web
root      2443     1  0  2021 ?        00:04:34 /usr/libexec/postfix/master -w
postfix   2445  2443  0  2021 ?        00:00:58 qmgr -l -t unix -u
root      2479     1  0  2021 ?        00:00:01 /usr/sbin/sshd -D
root      2484     1  0  2021 ?        02:00:03 /usr/sbin/rsyslogd -n
root      2486     1  0  2021 ?        00:40:06 /usr/bin/amazon-ssm-agent
root      2493     1  0  2021 ?        00:00:02 /usr/sbin/atd -f
root      2496     1  0  2021 ?        00:03:31 /usr/sbin/crond -n
jenkins   2516  2570  0  2022 ?        5-02:19:52 /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.282.b08-1.amzn2.0.1.x86_64/bin/java -Xmx1024m -XX:MaxPermSize=512m -cp /var/lib/jenkins/plugins/maven-plugin/WEB-INF/lib/maven35-agent-1.12-alpha-1.jar:/var/lib/jenkins/tools/hudson.tasks.Maven_MavenInstallation/maven_3.6.0/boot/plexus-classworlds-2.5.2.jar:/var/lib/jenkins/tools/hudson.tasks.Maven_MavenInstallation/maven_3.6.0/conf/logging jenkins.maven3.agent.Maven35Main /var/lib/jenkins/tools/hudson.tasks.Maven_MavenInstallation/maven_3.6.0 /var/cache/jenkins/war/WEB-INF/lib/remoting-3.27.jar /var/lib/jenkins/plugins/maven-plugin/WEB-INF/lib/maven35-interceptor-1.12-alpha-1.jar /var/lib/jenkins/plugins/maven-plugin/WEB-INF/lib/maven3-interceptor-commons-1.12-alpha-1.jar 40535
root      2520     1  0  2021 ttyS0    00:00:00 /sbin/agetty --keep-baud 115200,38400,9600 ttyS0 vt220
root      2521     1  0  2021 tty1     00:00:00 /sbin/agetty --noclear tty1 linux
jenkins   2570     1  0  2021 ?        6-07:29:24 /etc/alternatives/java -Dcom.sun.akuma.Daemon=daemonized -Djava.awt.headless=true -DJENKINS_HOME=/var/lib/jenkins -jar /usr/lib/jenkins/jenkins.war --logfile=/var/log/jenkins/jenkins.log --webroot=/var/cache/jenkins/war --daemon --httpPort=8081 --debug=5 --handlerCountMax=100 --handlerCountMaxIdle=20 --prefix=/jenkins
root      2631     1  0  2021 ?        00:16:01 runsvdir -P /opt/gitlab/service log: ...........................................................................................................................................................................................................................................................................................................................................................................................................
root      2633  2631  0  2021 ?        00:00:05 runsv redis
root      2634  2631  0  2021 ?        00:00:04 runsv gitaly
root      2635  2631  0  2021 ?        00:00:05 runsv postgresql
root      2636  2631  0  2021 ?        00:00:04 runsv puma
root      2637  2631  0  2021 ?        00:00:05 runsv sidekiq
root      2638  2631  0  2021 ?        00:00:05 runsv gitlab-workhorse
root      2639  2631  0  2021 ?        00:00:05 runsv nginx
root      2640  2631  0  2021 ?        00:00:21 runsv logrotate
root      2641  2631  0  2021 ?        00:00:05 runsv node-exporter
root      2642  2631  0  2021 ?        00:00:05 runsv gitlab-exporter
root      2643  2634  0  2021 ?        00:16:57 svlogd /var/log/gitlab/gitaly
git       2644  2634  0  2021 ?        03:59:36 /opt/gitlab/embedded/bin/gitaly-wrapper /opt/gitlab/embedded/bin/gitaly /var/opt/gitlab/gitaly/config.toml
root      2645  2631  0  2021 ?        00:00:05 runsv redis-exporter
root      2646  2631  0  2021 ?        00:00:05 runsv prometheus
root      2647  2631  0  2021 ?        00:00:05 runsv alertmanager
root      2648  2637  0  2021 ?        00:02:14 svlogd /var/log/gitlab/sidekiq
root      2649  2631  0  2021 ?        00:00:05 runsv postgres-exporter
root      2650  2633  0  2021 ?        00:01:29 svlogd -tt /var/log/gitlab/redis
git       2651  2637  0  2021 ?        00:20:34 ruby /opt/gitlab/embedded/service/gitlab-rails/bin/sidekiq-cluster -e production -r /opt/gitlab/embedded/service/gitlab-rails -m 50 --timeout 25 *
gitlab-+  2652  2633  1  2021 ?        21-01:08:19 /opt/gitlab/embedded/bin/redis-server 127.0.0.1:0
root      2653  2631  0  2021 ?        00:00:04 runsv grafana
root      2654  2635  0  2021 ?        00:00:04 svlogd -tt /var/log/gitlab/postgresql
gitlab-+  2655  2635  0  2021 ?        00:13:48 /opt/gitlab/embedded/bin/postgres -D /var/opt/gitlab/postgresql/data
root      2656  2640  0  2021 ?        00:00:04 svlogd -tt /var/log/gitlab/logrotate
root      2657  2636  0  2021 ?        00:00:04 svlogd -tt /var/log/gitlab/puma
git       2658  2636  0  2021 ?        06:40:48 puma 4.3.5.gitlab.3 (unix:///var/opt/gitlab/gitlab-rails/sockets/gitlab.socket,tcp://127.0.0.1:8080) [gitlab-puma-worker]
root      2660  2638  0  2021 ?        00:00:04 svlogd /var/log/gitlab/gitlab-workhorse
root      2661  2639  0  2021 ?        00:00:04 svlogd -tt /var/log/gitlab/nginx
git       2662  2638  0  2021 ?        06:41:44 /opt/gitlab/embedded/bin/gitlab-workhorse -listenNetwork unix -listenUmask 0 -listenAddr /var/opt/gitlab/gitlab-workhorse/sockets/socket -authBackend http://localhost:8080 -authSocket /var/opt/gitlab/gitlab-rails/sockets/gitlab.socket -documentRoot /opt/gitlab/embedded/service/gitlab-rails/public -pprofListenAddr  -prometheusListenAddr localhost:9229 -secretPath /opt/gitlab/embedded/service/gitlab-rails/.gitlab_workhorse_secret -logFormat json -config config.toml
root      2663  2639  0  2021 ?        00:00:00 nginx: master process /opt/gitlab/embedded/sbin/nginx -p /var/opt/gitlab/nginx
root      2664  2641  0  2021 ?        00:00:04 svlogd -tt /var/log/gitlab/node-exporter
gitlab-+  2665  2641  0  2021 ?        1-17:47:18 /opt/gitlab/embedded/bin/node_exporter --web.listen-address=localhost:9100 --collector.mountstats --collector.runit --collector.runit.servicedir=/opt/gitlab/sv --collector.textfile.directory=/var/opt/gitlab/node-exporter/textfile_collector
root      2666  2642  0  2021 ?        00:00:04 svlogd -tt /var/log/gitlab/gitlab-exporter
root      2668  2653  0  2021 ?        00:00:04 svlogd -tt /var/log/gitlab/grafana
gitlab-+  2669  2653  0  2021 ?        02:34:05 /opt/gitlab/embedded/bin/grafana-server -config /var/opt/gitlab/grafana/grafana.ini
root      2670  2646  0  2021 ?        00:00:09 svlogd -tt /var/log/gitlab/prometheus
gitlab-+  2671  2646  0  2021 ?        12-01:19:09 /opt/gitlab/embedded/bin/prometheus --web.listen-address=localhost:9090 --storage.tsdb.path=/var/opt/gitlab/prometheus/data --config.file=/var/opt/gitlab/prometheus/prometheus.yml
root      2673  2645  0  2021 ?        00:00:04 svlogd -tt /var/log/gitlab/redis-exporter
gitlab-+  2674  2645  0  2021 ?        11:51:07 /opt/gitlab/embedded/bin/redis_exporter --web.listen-address=localhost:9121 --redis.addr=unix:///var/opt/gitlab/redis/redis.socket
root      2676  2649  0  2021 ?        00:00:05 svlogd -tt /var/log/gitlab/postgres-exporter
gitlab-+  2677  2649  0  2021 ?        11-04:36:30 /opt/gitlab/embedded/bin/postgres_exporter --web.listen-address=localhost:9187 --extend.query-path=/var/opt/gitlab/postgres-exporter/queries.yaml
root      2678  2647  0  2021 ?        00:00:04 svlogd -tt /var/log/gitlab/alertmanager
gitlab-+  2679  2647  0  2021 ?        23:00:51 /opt/gitlab/embedded/bin/alertmanager --web.listen-address=localhost:9093 --storage.path=/var/opt/gitlab/alertmanager/data --config.file=/var/opt/gitlab/alertmanager/alertmanager.yml
git       2694  2644  0  2021 ?        2-12:06:59 /opt/gitlab/embedded/bin/gitaly /var/opt/gitlab/gitaly/config.toml
gitlab-+  2719  2663  0  2021 ?        00:24:32 nginx: worker process
gitlab-+  2720  2663  0  2021 ?        00:03:43 nginx: worker process
gitlab-+  2721  2663  0  2021 ?        00:08:11 nginx: cache manager process
gitlab-+  2746  2655  0  2021 ?        00:10:51 postgres: checkpointer
gitlab-+  2747  2655  0  2021 ?        00:16:25 postgres: background writer
gitlab-+  2748  2655  0  2021 ?        00:17:06 postgres: walwriter
gitlab-+  2749  2655  0  2021 ?        00:10:15 postgres: autovacuum launcher
gitlab-+  2750  2655  0  2021 ?        03:02:01 postgres: stats collector
gitlab-+  2751  2655  0  2021 ?        00:01:01 postgres: logical replication launcher
gitlab-+  2756  2655  0  2021 ?        4-16:52:50 postgres: gitlab-psql gitlabhq_production [local] idle
git       2760  2651  2  2021 ?        43-22:33:12 sidekiq 5.2.9 queues:authorized_project_update:authorized_project_update_project_create,authorized_project_update:authorized_project_update_project_group_link_create,authorized_project_update:authorized_project_update_user_refresh_over_user_range,authorized_project_update:authorized_project_update_user_refresh_with_low_urgency,auto_devops:auto_devops_disable,auto_merge:auto_merge_process,chaos:chaos_cpu_spin,chaos:chaos_db_spin,chaos:chaos_kill,chaos:chaos_leak_mem,chaos:chaos_sleep,container_repository:cleanup_container_repository,container_repository:delete_container_repository,cronjob:admin_email,cronjob:analytics_instance_statistics_count_job_trigger,cronjob:authorized_project_update_periodic_recalculate,cronjob:ci_archive_traces_cron,cronjob:ci_platform_metrics_update_cron,cronjob:ci_schedule_delete_objects_cron,cronjob:container_expiration_policy,cronjob:environments_auto_stop_cron,cronjob:expire_build_artifacts,cronjob:gitlab_usage_ping,cronjob:import_export_project_cleanup,cronjob:import_stuck_projec
git       2783  2694  0  2021 ?        2-01:44:07 ruby /opt/gitlab/embedded/service/gitaly-ruby/bin/gitaly-ruby 2694 /var/opt/gitlab/gitaly/internal_sockets/ruby.0
git       2785  2694  0  2021 ?        2-02:20:10 ruby /opt/gitlab/embedded/service/gitaly-ruby/bin/gitaly-ruby 2694 /var/opt/gitlab/gitaly/internal_sockets/ruby.1
gitlab-+  3154  2655  0  2021 ?        00:00:00 postgres: gitlab gitlabhq_production [local] idle
gitlab-+  3662  2655  0  2021 ?        00:00:00 postgres: gitlab gitlabhq_production [local] idle
gitlab-+  4061  2655  0  2021 ?        00:00:00 postgres: gitlab gitlabhq_production [local] idle
git       4891  2658  0 Jul09 ?        00:16:22 puma: cluster worker 1: 2658 [gitlab-puma-worker]
gitlab-+  4903  2655  0 Jul09 ?        00:00:00 postgres: gitlab gitlabhq_production [local] idle
jenkins   5896  6951  0  2024 ?        00:01:02 /home/jenkins/svn/csvn/bin/httpd -f /home/jenkins/svn/csvn/data/conf/httpd.conf -k start
jenkins   5903  6951  0  2024 ?        00:01:06 /home/jenkins/svn/csvn/bin/httpd -f /home/jenkins/svn/csvn/data/conf/httpd.conf -k start
jenkins   5908  6951  0  2024 ?        00:01:03 /home/jenkins/svn/csvn/bin/httpd -f /home/jenkins/svn/csvn/data/conf/httpd.conf -k start
root      6260     1  0  2022 ?        00:58:58 /usr/sbin/httpd -DFOREGROUND
jenkins   6951     1  0  2021 ?        01:03:11 /home/jenkins/svn/csvn/bin/httpd -f /home/jenkins/svn/csvn/data/conf/httpd.conf -k start
jenkins   6952  6951  0  2021 ?        00:00:01 /home/jenkins/svn/csvn/bin/rotatelogs -l /home/jenkins/svn/csvn/data/logs/error_%Y_%m_%d_%H_%M_%S.log 86400
jenkins   6953  6951  0  2021 ?        00:06:57 /home/jenkins/svn/csvn/bin/rotatelogs -l /home/jenkins/svn/csvn/data/logs/access_%Y_%m_%d_%H_%M_%S.log 86400
jenkins   6954  6951  0  2021 ?        00:00:09 /home/jenkins/svn/csvn/bin/rotatelogs -l /home/jenkins/svn/csvn/data/logs/subversion_%Y_%m_%d_%H_%M_%S.log 86400
apache    8334  6260  0 Jul22 ?        00:00:38 /usr/sbin/httpd -DFOREGROUND
apache    8347  6260  0 Jul22 ?        00:00:37 /usr/sbin/httpd -DFOREGROUND
apache    8348  6260  0 Jul22 ?        00:00:37 /usr/sbin/httpd -DFOREGROUND
apache    8349  6260  0 Jul22 ?        00:00:37 /usr/sbin/httpd -DFOREGROUND
apache    8540  6260  0 Jul22 ?        00:00:36 /usr/sbin/httpd -DFOREGROUND
apache    8543  6260  0 Jul22 ?        00:00:37 /usr/sbin/httpd -DFOREGROUND
apache    8555  6260  0 Jul22 ?        00:00:37 /usr/sbin/httpd -DFOREGROUND
apache    8599  6260  0 Jul22 ?        00:00:38 /usr/sbin/httpd -DFOREGROUND
apache    8602  6260  0 Jul22 ?        00:00:38 /usr/sbin/httpd -DFOREGROUND
gitlab-+ 13928  2655  0 Jul11 ?        00:55:39 postgres: gitlab gitlabhq_production [local] idle
gitlab-+ 13931  2655  1 Jul11 ?        03:09:27 postgres: gitlab gitlabhq_production [local] idle
git      14592  2642  8 Apr02 ?        9-01:17:15 puma 4.3.5.gitlab.3 (tcp://localhost:9168) [gitlab-exporter]
gitlab-+ 14612  2655  0 Apr02 ?        00:58:14 postgres: gitlab gitlabhq_production [local] idle
gitlab-+ 14613  2655  0 Apr02 ?        01:07:59 postgres: gitlab gitlabhq_production [local] idle
gitlab-+ 14614  2655  1 Apr02 ?        1-07:02:34 postgres: gitlab gitlabhq_production [local] idle
root     15533  1700  0  2022 ?        2-04:07:55 /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.282.b08-1.amzn2.0.1.x86_64/jre/bin/java -XX:MaxPermSize=128m -Djetty.home=../appserver -Djetty.port=3343 -Djetty.ssl.port=4434 -Xms64m -Xmx512m -Djava.library.path=../lib -classpath ../lib/wrapper.jar -Dwrapper.key=LinDVRLWN-Wh7kFQ -Dwrapper.port=32000 -Dwrapper.disable_console_input=TRUE -Dwrapper.pid=1700 -Dwrapper.version=3.5.26 -Dwrapper.native_library=wrapper -Dwrapper.arch=x86 -Dwrapper.service=TRUE -Dwrapper.cpu.timeout=10 -Dwrapper.jvmid=2 org.tanukisoftware.wrapper.WrapperJarApp ../appserver/start.jar
root     16331     2  0 02:58 ?        00:00:00 [kworker/u4:2]
gitlab-+ 17136  2655  0 Jul14 ?        00:05:48 postgres: gitlab gitlabhq_production [local] idle
gitlab-+ 17138  2655  0 Jul14 ?        00:00:00 postgres: gitlab gitlabhq_production [local] idle
jenkins  17969  2570  0  2022 ?        4-11:06:38 /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.282.b08-1.amzn2.0.1.x86_64/bin/java -Xmx1024m -XX:MaxPermSize=512m -cp /var/lib/jenkins/plugins/maven-plugin/WEB-INF/lib/maven35-agent-1.12-alpha-1.jar:/var/lib/jenkins/tools/hudson.tasks.Maven_MavenInstallation/maven_3.6.0/boot/plexus-classworlds-2.5.2.jar:/var/lib/jenkins/tools/hudson.tasks.Maven_MavenInstallation/maven_3.6.0/conf/logging jenkins.maven3.agent.Maven35Main /var/lib/jenkins/tools/hudson.tasks.Maven_MavenInstallation/maven_3.6.0 /var/cache/jenkins/war/WEB-INF/lib/remoting-3.27.jar /var/lib/jenkins/plugins/maven-plugin/WEB-INF/lib/maven35-interceptor-1.12-alpha-1.jar /var/lib/jenkins/plugins/maven-plugin/WEB-INF/lib/maven3-interceptor-commons-1.12-alpha-1.jar 45133
jenkins  19547  6951  0 May28 ?        00:00:26 /home/jenkins/svn/csvn/bin/httpd -f /home/jenkins/svn/csvn/data/conf/httpd.conf -k start
root     19879  2640  0 03:18 ?        00:00:00 /bin/sh /opt/gitlab/embedded/bin/gitlab-logrotate-wrapper
root     21609 19879  0 03:28 ?        00:00:00 sleep 3000
apache   21970  6260  0 Jul17 ?        00:02:19 /usr/sbin/httpd -DFOREGROUND
jenkins  22025  6951  0 May26 ?        00:00:22 /home/jenkins/svn/csvn/bin/httpd -f /home/jenkins/svn/csvn/data/conf/httpd.conf -k start
jenkins  22026  6951  0 May26 ?        00:00:30 /home/jenkins/svn/csvn/bin/httpd -f /home/jenkins/svn/csvn/data/conf/httpd.conf -k start
root     23248     2  0 03:38 ?        00:00:00 [kworker/u4:1]
gitlab-+ 24039  2655  0 03:42 ?        00:00:00 postgres: gitlab gitlabhq_production [local] idle
jenkins  25685  6951  0  2024 ?        00:05:47 /home/jenkins/svn/csvn/bin/httpd -f /home/jenkins/svn/csvn/data/conf/httpd.conf -k start
jenkins  26322  6951  0  2024 ?        00:06:17 /home/jenkins/svn/csvn/bin/httpd -f /home/jenkins/svn/csvn/data/conf/httpd.conf -k start
gitlab-+ 26326  2655  0 03:56 ?        00:00:00 postgres: gitlab gitlabhq_production [local] idle
root     26650     2  0 03:57 ?        00:00:00 [kworker/1:0]
root     26996     2  0 04:00 ?        00:00:00 [kworker/0:1]
root     26998     2  0 04:00 ?        00:00:00 [kworker/0:3]
root     27623     2  0 04:03 ?        00:00:00 [kworker/1:1]
root     27767  2479  0 04:04 ?        00:00:00 sshd: ec2-user [priv]
ec2-user 27777 27767  0 04:04 ?        00:00:00 sshd: ec2-user@pts/0
ec2-user 27778 27777  0 04:04 pts/0    00:00:00 -bash
jenkins  28313  2570  0  2022 ?        4-15:10:09 /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.282.b08-1.amzn2.0.1.x86_64/bin/java -Xmx1024m -XX:MaxPermSize=512m -cp /var/lib/jenkins/plugins/maven-plugin/WEB-INF/lib/maven35-agent-1.12-alpha-1.jar:/var/lib/jenkins/tools/hudson.tasks.Maven_MavenInstallation/maven_3.6.0/boot/plexus-classworlds-2.5.2.jar:/var/lib/jenkins/tools/hudson.tasks.Maven_MavenInstallation/maven_3.6.0/conf/logging jenkins.maven3.agent.Maven35Main /var/lib/jenkins/tools/hudson.tasks.Maven_MavenInstallation/maven_3.6.0 /var/cache/jenkins/war/WEB-INF/lib/remoting-3.27.jar /var/lib/jenkins/plugins/maven-plugin/WEB-INF/lib/maven35-interceptor-1.12-alpha-1.jar /var/lib/jenkins/plugins/maven-plugin/WEB-INF/lib/maven3-interceptor-commons-1.12-alpha-1.jar 45247
root     28414  6260  0  2023 ?        00:02:52 /usr/sbin/rotatelogs -l logs/access_log.%Y%m%d 86400
root     28416  6260  0  2023 ?        00:16:12 /usr/sbin/rotatelogs -l logs/mod_jk.log.%Y%m%d 86400
postfix  28524  2443  0 04:08 ?        00:00:00 pickup -l -t unix -u
root     28625     2  0 04:09 ?        00:00:00 [kworker/1:2]
root     28767     2  0 04:10 ?        00:00:00 [kworker/1:3]
gitlab-+ 28803  2655  0 04:10 ?        00:00:00 postgres: gitlab gitlabhq_production [local] idle
git      29104  2658  0 May29 ?        00:56:26 puma: cluster worker 0: 2658 [gitlab-puma-worker]
gitlab-+ 29112  2655  0 May29 ?        00:00:00 postgres: gitlab gitlabhq_production [local] idle
ec2-user 29444 27778  0 04:13 pts/0    00:00:00 ps -efww
jenkins  32114  6951  0 Jul20 ?        00:00:00 /home/jenkins/svn/csvn/bin/httpd -f /home/jenkins/svn/csvn/data/conf/httpd.conf -k start
jenkins  32115  6951  0 Jul20 ?        00:00:00 /home/jenkins/svn/csvn/bin/httpd -f /home/jenkins/svn/csvn/data/conf/httpd.conf -k start

~~~



~~~
[root@anysign-web ~]# sudo gitlab-ctl status
run: alertmanager: (pid 2679) 129689364s; run: log: (pid 2678) 129689364s
run: gitaly: (pid 2644) 129689365s; run: log: (pid 2643) 129689365s
run: gitlab-exporter: (pid 14592) 9868363s; run: log: (pid 2666) 129689365s
run: gitlab-workhorse: (pid 2662) 129689365s; run: log: (pid 2660) 129689365s
run: grafana: (pid 2669) 129689365s; run: log: (pid 2668) 129689365s
run: logrotate: (pid 29342) 1318s; run: log: (pid 2656) 129689365s
run: nginx: (pid 2663) 129689365s; run: log: (pid 2661) 129689365s
run: node-exporter: (pid 2665) 129689365s; run: log: (pid 2664) 129689365s
run: postgres-exporter: (pid 2677) 129689365s; run: log: (pid 2676) 129689365s
run: postgresql: (pid 2655) 129689365s; run: log: (pid 2654) 129689365s
run: prometheus: (pid 2671) 129689365s; run: log: (pid 2670) 129689365s
run: puma: (pid 2658) 129689365s; run: log: (pid 2657) 129689365s
run: redis: (pid 2652) 129689365s; run: log: (pid 2650) 129689365s
run: redis-exporter: (pid 2674) 129689365s; run: log: (pid 2673) 129689365s
run: sidekiq: (pid 2651) 129689365s; run: log: (pid 2648) 129689365s
~~~

~~~
[root@anysign-web ~]# free -h
              total        used        free      shared  buff/cache   available
Mem:           3.8G        3.3G        128M        101M        380M        123M
Swap:          7.8G        3.7G        4.1G
~~~


~~~
[root@anysign-web ec2-user]# sudo gitlab-ctl stop
ok: down: alertmanager: 0s, normally up
ok: down: gitaly: 0s, normally up
ok: down: gitlab-exporter: 0s, normally up
ok: down: gitlab-workhorse: 1s, normally up
ok: down: grafana: 0s, normally up
ok: down: logrotate: 1s, normally up
ok: down: nginx: 0s, normally up
ok: down: node-exporter: 0s, normally up
ok: down: postgres-exporter: 1s, normally up
ok: down: postgresql: 0s, normally up
ok: down: prometheus: 1s, normally up
ok: down: puma: 0s, normally up
ok: down: redis: 0s, normally up
ok: down: redis-exporter: 1s, normally up
ok: down: sidekiq: 0s, normally up
~~~



~~~

sudo gitlab-ctl status

sudo gitlab-ctl status

# 전체 재시작 (구성 변경 반영)
sudo gitlab-ctl restart



 systemctl status gitlab-runsvdir.service
  systemctl disable gitlab-runsvdir.service
~~~


~~~

//3690
 sudo /home/jenkins/svn/csvn/bin/httpd -f /home/jenkins/svn/csvn/data/conf/httpd.conf -k start

/home/jenkins/svn/csvn/bin -> README

// 3343 4434
sudo /home/jenkins/svn/csvn/bin/csvn start
~~~