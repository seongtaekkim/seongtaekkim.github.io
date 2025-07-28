

~~~


vi /etc/systemd/system/admin-webapp.service


[Unit]
Description=tomcat8(admin-webapp)
After=network.target syslog.target

[Service]
Type=forking
#Environment=JAVA_HOME=
PIDFile=/home/ec2-user/search/app/admin-webapp/pid/admin-webapp.pid


User=ec2-user
Group=elasticsearch

ExecStart=/home/ec2-user/search/bin/admin-webapp-startup.sh
ExecStop=/home/ec2-user/search/bin/admin-webapp-shutdown.sh


UMask=0007
RestartSec=10s
Restart=on-success

TimeoutStartSec=450s
TimeoutStopSec=60s

[Install]
WantedBy=multi-user.target
~                                                                                                                                                                                                                                                                                       ~                                    


~~~


