https://www.jenkins.io/blog/2022/12/27/run-jenkins-agent-as-a-service/

### node 생성


### agent 구성 및 연결 테스트



### jenkins agent service 등록


~~~


sudo adduser --group --home /home/jenkins --shell /bin/bash jenkins
sudo adduser --home /home/jenkins --shell /bin/bash jenkins




# run jenins agent as a service
sudo mkdir -p /usr/local/jenkins-service
sudo chown jenkins /usr/local/jenkins-service




~~~




![[Pasted image 20250717152708.png]]








### copy artifacts 구성






### sudo 권한 부여
- jenkins 에서 sudo systemctl ~ 를 사용하게 하기 위해 nopassword 정책 적용

~~~

# root 계정에서 wheel 그룹부여
usermod -aG wheel was

# was계정에서 그릅확인
id

# systemctl 실행위치 확인
which systemctl

sudo visudo -f /etc/sudoers.d/jenkins-tomcat

# 추가
was ALL=(ALL) NOPASSWD: /bin/systemctl stop tomcat, /bin/systemctl start tomcat, /bin/systemctl status tomcat

# 권한 확인
ls -l /etc/sudoers.d/jenkins-tomcat
sudo visudo -c
sudo -l -U was

~~~





passkey 통신방식 webauthn

~~~

ssh-keygen -t ed25519 -f ~/.ssh/stkkim_ed25519 -C "stkkim_key"
ssh-copy-id -p 24477 -i ~/.ssh/stkkim_ed25519.pub stk.kim@192.168.6.113
ssh -p 24477 -i stkkim_ed25519 stk.kim@192.168.6.113


ㅇcmms 

공인ㅇ아이피 1개 ㅇㄹ




  
jong.hwa.kim


~~~




cloudadmin
clouddomain