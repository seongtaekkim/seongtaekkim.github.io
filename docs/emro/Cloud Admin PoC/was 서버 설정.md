
- jdk17 설치
- tomcat 10.1 설치
- was 계정 생성
- logrotate 구성
- 방화벽 구성


#### caidentia cloud admin poc 개발환경 

~~~ sh

# jdk

sudo dnf list --installed "java-*openjdk*"
sudo dnf remove java-1.8.0-openjdk*

sudo dnf install -y java-17-openjdk java-17-openjdk-devel



~~~ 


~~~ sh

# 안하는걸로 (CD 정책에 따라 외부에서 ssh 등 접속할 수 있어야 하므로, 로그인불가 시스템계정은 잘 사용하지 않는 추세라고 함)
# 그룹 먼저 생성
groupadd --system was

# 로그인 불가·시스템 전용 계정 생성
useradd --system --gid was \
        --home-dir /opt/was \
        --shell /usr/sbin/nologin was
        

~~~


~~~ sh


# 1) 새 버전 다운로드·압축 해제
curl -O https://dlcdn.apache.org/tomcat/tomcat-10/v10.1.43/bin/apache-tomcat-10.1.43.tar.gz
tar -xf apache-tomcat-10.1.43.tar.gz -C /data
mv /data/apache-tomcat-10.1.43 /data/tomcat-10.1.43

# 2) 최초 심볼릭 링크 생성 (없으면)
ln -s /data/tomcat-10.1.43 /data/tomcat

# 3) 소유권/퍼미션 적용
### 밑에 정책적용 안하고 그냥 tomcat 폴더전체를 was:was로 고정하였음
sudo chown -R was:was /data/tomcat-10.1.43
# bin 디렉터리와 실행 스크립트들만 root:was 750 로
sudo chown root:was /data/tomcat-10.1.43/bin
sudo chown root:was /data/tomcat-10.1.43/bin/*.sh
sudo chmod 750     /data/tomcat-10.1.43/bin
sudo chmod 750     /data/tomcat-10.1.43/bin/*.sh

# 링크가 가리키는 최신 버전에도 동일하게
sudo chown root:was /data/tomcat/bin



# 4) 서비스 재시작
systemctl restart tomcat


~~~



~~~ sh
[Unit]
Description=Apache Tomcat
After=network.target

[Service]
Type=forking

User=was
Group=was
UMask=0007


# 환경 변수 (필요 시)
# java home은 심볼릭 링하는 java폴더로 지정
Environment="JAVA_HOME=/usr/lib/jvm/java"
Environment="CATALINA_BASE=/data/tomcat"
Environment="CATALINA_HOME=/data/tomcat"

# 시작/중지 스크립트
ExecStart=/usr/bin/bash /data/tomcat/bin/startup.sh
ExecStop=/usr/bin/bash /data/tomcat/bin/shutdown.sh
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


http 막기
 
- firealld 허용정책
	- 443, 24477 허용
	- 8080: 특정아이피에 대해서만 허용 
~~~ sh

firewall-cmd --list-all
# firewall-cmd --remove-port=8080/tcp --permanent
#firewall-cmd --reload

firewall-cmd --add-port=8080/tcp --permanent
firewall-cmd --reload

sudo firewall-cmd --permanent --remove-rich-rule="rule family='ipv4' source address='192.168.6.59/32' port port='8080' protocol='tcp' accept"

~~~



logrotate 설정
~~~ sh

/data/tomcat/logs/catalina.out {
    copytruncate
    daily
    rotate 30
    compress
    missingok
    notifempty
    delaycompress
}
sudo logrotate -d /etc/logrotate.d/tomcat
~~~


java home 확인
~~~ sh

alternatives --display java
cd /usr/lib/jvm
ls -als
~~~




#### 톰캣 webapps 폴더 실제 ROOT 디렉토리 변경
- ${TOMCAT_HOME}/conf/server.xml
- appBase 위치를 절대경로로 다른 위치에 배치하여 관리포인트 분리하도록 고려
~~~

  <Host name="localhost"  appBase="webapps"
		unpackWARs="true" autoDeploy="true">
            
~~~



#### ${TOMCAT_HOME}/work

| 목적              | 내용                                                                                              |
| --------------- | ----------------------------------------------------------------------------------------------- |
| **JSP 임시 컴파일**  | `*.jsp`를 서블릿(`*.java → *.class`)으로 변환해 두는 자리입니다. 변경 여부를 감지해 자동 재컴파일을 하므로, 실제 서비스 중에는 반드시 필요합니다. |
| **세션 직렬화 (선택)** | `PersistentManager`를 쓸 때 세션을 디스크에 저장하는 위치로도 쓰입니다.                                               |
| **캐시·임시 파일**    | EL 식 캐시, 서블릿 임시 파일 등 Tomcat 내부 컴포넌트가 만드는 각종 tmp 파일이 여기에 놓입니다.                                   |
|                 |                                                                                                 |
| 디렉토리 설명         |                                                                                                 |
| Catalina        | 가상 호스트별 디렉터리                                                                                    |
|                 |                                                                                                 |
|                 |                                                                                                 |




~~~
netstat -tulpn

~~~
