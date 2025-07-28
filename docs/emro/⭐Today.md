


## npki eks 연결구성


write
read


eks 에 

~~~

apiVersion: v1
kind: PersistentVolume
metadata:
  namespace: emrocloud-dev
  name: pv-efs-npki
spec:
  capacity:
    storage: 10Gi
  accessModes: [ReadWriteMany]
  persistentVolumeReclaimPolicy: Retain
  csi:
    driver: efs.csi.aws.com
    volumeHandle: fs-12345678::fsap-abcdef0123456789
    volumeAttributes:
      accessPointId: fsap-abcdef0123456789
  mountOptions: [tls]
  
~~~

~~~ yml

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-efs-npki
spec:
  accessModes: [ReadWriteMany]
  resources:
    requests:
      storage: 10Gi
  volumeName: pv-efs-npki

~~~


~~~
sudo yum -y install amazon-efs-utils  
sudo mkdir /mnt/efs-npki
sudo mount -t efs -o tls,accesspoint=fsap-abcdef0123456789 \
  fs-12345678:/ /mnt/efs-npki


fs-12345678 /mnt/efs-npki efs _netdev,tls,accesspoint=fsap-abcdef0123456789 

~~~


~~~

  - hostPath:
      path: /usr/local/NPKI/
      type: ""
      
# Pod (발췌)
volumes:
- name: shared
  persistentVolumeClaim:
    claimName: pvc-efs-npki
containers:
- name: app
  image: nginx:1.27
  volumeMounts:
  - name: shared
    mountPath: /usr/local/NPKI
  - 
~~~

## 에코비트

15.164.168.207
15.164.206.133

IF -> NLB IP:PORT -> ALB Nginx
- NLB EIP 생성 후 할당 필요

~~~ bash

stream {
    log_format proxy '$remote_addr [$time_local] '
                     '$protocol $status $bytes_sent $bytes_received '
                     '$session_time "$upstream_addr" "$upstream_bytes_sent" "$upstream_bytes_received" "$upstream_connect_time"';

    access_log /var/log/nginx/tcp-access.log proxy;

    upstream bsg_3200_backend {
        hash $remote_addr consistent;
        server 15.164.168.207:3200;
        server 15.164.206.133:3200;
    }

    server {
        listen 3200;
        proxy_pass bsg_3200_backend;
    }

    upstream bsg_3300_backend {
        hash $remote_addr consistent;
        server 15.164.168.207:3300;
        server 15.164.206.133:3300;
    }

    server {
        listen 3300;
        proxy_pass bsg_3300_backend;
    }
}


~~~





#### 한샘 도메인
hanssem.dev.emrocloud.com
hanssem.emrocloud.com




애니사인
ANYBUYER / 1111




PRF( Pseudo-Random Function ) SHA1 ==> Integrity SHA1
Diffie-Hellman group 2 -> DH 2,
Lifetime 28800

Phase 2
Encryption AES128
Authentication SHA1
PFS group 2
Lifetime 3600



azure -> jenkins (사설ip) 통신이 어떻게 되어야 하는가

## spLogin 리다이렉트
- 우선 중단 및 롤백하였음.

partner.dev.emrocloud.com

/spLogin.do -> 파ㅣ트너스로 리다이렉트

예외
demo.dev.emrocloud.com
imk.emrocloud.com/login.do : 구매사
imk.emrocloud.com/spLogin.do : 공급사

[www.imarketauction.com/login.do](https://flow.emro.co.kr/www.imarketauction.com/login.do) : 공급사
[www.imarketauction.com/spLogin.do](https://flow.emro.co.kr/www.imarketauction.com/spLogin.do) : 공급사


~~~

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    alb.ingress.kubernetes.io/group.name: emrocloud
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/target-group-attributes: stickiness.enabled=true,stickiness.lb_cookie.duration_seconds=12000
    alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:ap-northeast-2:664127938624:certificate/91c36160-a915-46e3-a4af-6dc8f5bff9c1
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTP": 80}, {"HTTPS":443}]'
    alb.ingress.kubernetes.io/actions.ssl-redirect: >
      {"Type":"redirect",
       "RedirectConfig":{
         "Protocol":"HTTPS",
         "Host":"partner.dev.emrocloud.com",
         "Path":"/spLogin.do",
         "StatusCode":"HTTP_301"}}
    alb.ingress.kubernetes.io/conditions.ssl-redirect: >
      [
        {
          "Field":"host-header",
          "HostHeaderConfig":{
            "Values": ["dev.emrocloud.com","*.dev.emrocloud.com"]
          }
        }
      ]
  name: redirect-to-partner-ing
  namespace: emrocloud-dev
spec:
  rules:
    - http:
        paths:
          - path: /spLogin.do
            pathType: Exact
            backend:
              service:
                name: ssl-redirect
                port:
                  name: use-annotation
                  

~~~

dualstack.emrocloud-dev-web-alb-809693765.ap-northeast-2.elb.amazonaws.com
k8s-emrocloud-0c80f488fd-820303567.ap-northeast-2.elb.amazonaws.com
internal-k8s-emrocloudinternal-431a644be6-1667243781.ap-northeast-2.elb.amazonaws.com

emrocloud.scheduler@gmail.com
Reboot2025#$


vm 신청할때 개인퍼블릭 키 
포트 변경
18080 18081
18080 18081
램 증설요청 
4 -> 8
이중화
톰캣 2개 띄워서


jenkins agent 할때 고려사항
- scaleout 될때 어케 인지할거냐 -내용검토
- wegbsocket systemd 
- https://www.jenkins.io/blog/2022/12/27/run-jenkins-agent-as-a-service/
- stash 쓰면 안되는 이유 https://www.jenkins.io/doc/pipeline/steps/workflow-basic-steps/?utm_source=chatgpt.com
- copy 아티펙트 검토
- 

#### Zoraxy ?? 뭐시기 구성해야함
npm 과 다르게 타겟을 두개이상 설정할 수 있어서 로드벨런서 구성이 가능함
계정추가 방법 알아봐야함


frontdoor dns cli 보고 알려준다고 함
db 고가용성 + readonly 테스트 후 알려준다고함
db write / db readonly 서버 write로 교체할 때에  7분 20 분 소요 된다고 함
- dns 는 그대로임
- dns/read, dns/write 형태로 가상dns ?로 구분된다고 함


#### poc 에 jenkins / agent 구성
- jenkins에서 s3에 업로드 후에cli명령어로 deploy하는 기존 형태는 보안에 상대적으로 취약하여 이를 해결하고자 jenkins / agent 배포방식을 poc해보려고 함.



#### 그검봐야 함 코드 오늘 그거 젠킨스
- 코드 분석
- 실제 그루비 코드를 볼수 없어 해당 프로젝트 뷰 권한요청도 해야 함 (caidentia_devops ? ) - 요청 중
- 7.16 caidentia-devops 조회권한 획득 (by 김성훈프로)


#### whatap 설치준비
- BIZ ec2
- IF ec2
- BIZ eks


##### k8s 배포를 위해 tomcat image pull & 설정을 포함하여 재 빌드 후 push



#### waf 내용정리 필요
- aws 문서 참고
- jira

#### 스케쥴러 구성 작성

스케쥴 설계 및 구성 
- 주중 07시 Start, 20시 Stop
- 월,화 23시 Stop
- 구글캘린더 Stop 시간 설정
- 공휴일 Stop
- 강제 Start/Stop 기능 적용 (Cli && Jenkins)

워크플로우 스크립트 작성
- EC2 및 RDS 시작 프로세스
- EC2 및 RDS 종료 프로세스



#### 미사용 테넌트 메일로 요청




#### DB삭제요청

문서작성




#### Azure Database for PostgreSQL 에서의 제약
https://learn.microsoft.com/en-us/answers/questions/812177/azure-database-for-postgresql-tablespaces
https://learn.microsoft.com/ko-kr/azure/postgresql/flexible-server/concepts-limits
os 접근권한 없고, 테이블스페이스 생성 불가.
~~~ sh

~~~
