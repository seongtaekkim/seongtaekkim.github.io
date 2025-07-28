대상 서버는 emroCloud-dev 15개.

anysigncloud-dev
- was
- web
- oss
- rds 1ea

emrocloud-dev
- was 2ea
- if 2ea
- web 1ea
- nginx 1ea
- eks node 2ea
- ubform 2ea
- oss 1ea
- rds 2ea


작업 전 선행
- 실행 프로세스(+port) 및 명령어 확인 
- 서비스 등록 확인
- EC2 스냅 샷 (백업)
- 서비스확인 방법 (url)
- mount 정보 (+ fstat)






~~~
 sudo netstat -tulpn
ps -efww
~~~




~~~
[root@anysign-web systemd]# systemctl is-enabled jenkins
jenkins.service is not a native service, redirecting to /sbin/chkconfig.
Executing /sbin/chkconfig jenkins --level=5
enabled
[root@anysign-web systemd]# chkconfig --list jenkins

Note: This output shows SysV services only and does not include native
      systemd services. SysV configuration data might be overridden by native
      systemd configuration.

      If you want to list systemd services use 'systemctl list-unit-files'.
      To see services enabled on particular target use
      'systemctl list-dependencies [target]'.

jenkins         0:off   1:off   2:on    3:on    4:on    5:on    6:off
~~~

