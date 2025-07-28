

~~~

[ec2-user@anysign-was 17518]$ ps -fp 17518
UID        PID  PPID  C STIME TTY          TIME CMD
was      17518     1  0  2021 ?        4-09:03:26 java -Xmx512m -classpath /usr/local/engn001/scouter/server/scouter-server-boot.jar scouter.boot.Boot ./lib
[ec2-user@anysign-was 17518]$ ps -fp 20218
UID        PID  PPID  C STIME TTY          TIME CMD
was      20218     1  0  2024 ?        15:22:16 java -classpath /usr/local/engn001/scouter/agent.host/scouter.host.jar scouter.boot.Boot ./lib
~~~



~~~
which systemctl

sudo visudo -f /etc/sudoers.d/jenkins-tomcat

# 추가
was ALL=(ALL) NOPASSWD: /bin/systemctl stop tomcat, /bin/systemctl start tomcat, /bin/systemctl status tomcat

# 권한 확인
ls -l /etc/sudoers.d/jenkins-tomcat
sudo visudo -c
sudo -l -U was
~~~


~~~


[Unit]
Description=Apache Tomcat anysignCloud BIZ
After=network.target

[Service]
Type=forking

User=was
Group=wasgroup
UMask=0007


# 환경 변수 (필요 시)
Environment="JAVA_HOME=/usr/lib/jvm/java"
Environment="CATALINA_BASE=/usr/local/tomcat_dev"
Environment="CATALINA_HOME=/usr/local/tomcat_dev"

# 시작/중지 스크립트
ExecStart=/usr/bin/bash /usr/local/tomcat_dev/bin/startup.sh
ExecStop=/usr/bin/bash /usr/local/tomcat_dev/bin/shutdown.sh
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
Description=Apache Tomcat anysignCloud IF
After=network.target

[Service]
Type=forking

User=was
Group=wasgroup
UMask=0007


# 환경 변수 (필요 시)
Environment="JAVA_HOME=/usr/lib/jvm/java"
Environment="CATALINA_BASE=/usr/local/tomcat_if"
Environment="CATALINA_HOME=/usr/local/tomcat_if"

# 시작/중지 스크립트
ExecStart=/usr/bin/bash /usr/local/tomcat_if/bin/startup.sh
ExecStop=/usr/bin/bash /usr/local/tomcat_if/bin/shutdown.sh
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
