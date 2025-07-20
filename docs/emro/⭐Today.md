
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
