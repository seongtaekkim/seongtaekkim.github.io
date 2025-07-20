
postgres replica 구성
캘린더 구성 정리 + 설명
인프라 구성 정리 + 설명
SES 






### 도메인

메일주소를 받지 못해 ㅇㅜ

rmi -> rest 변경 이후 urlencode 된 내용을 읽지 못해 base64로 인코딩 하거나
alb를 nlb로 변경하는 방법을 고려

특정서비스 트랜잭션 시간이 60초를 초과하여 120초로 늘려달라는 요청


운영중이 맞는지 ㅇ아닌지 화ㅑㄱ인하는 테넌트확욘요청

es측에 스케줄러 적용할건데 껏다켣 ㅗㅗ디는지 요청

cli는 jenkins등 서드파티 이용해서 하는걸 고려3



### QA ES CPIFA-92
[https://qasearchadm.emrocloud.com:7673/login.do](https://qasearchadm.emrocloud.com:7673/login.do)
OSS 버전으로 설치했기 때문에 elasticsearch 접속을 위한 계정은 따로 없습니다.

* 서비스 포트
 - elasticsearch: 9200
 - kibana: 5601
 - search-admin-webapp: 7673



### SKST - 한앤코 30호 (SK스패셜티)

- 52.79.83.145 (개발서버) => 개발 NAT
- 54.180.195.50 (운영서버) => 운영 NAT



### 용어사전 - 192.168.6.94
stk.kim / emro1004 
intellij pluglin token
eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJzdGsua2ltIiwicm9sZSI6IlVTRVIiLCJpYXQiOjE3NTIxMTYwMjZ9.NIGrmYOHkglWpUHN3uFbvEwIAfj0UyWnKAh1GoxK8lA


### 7.20 PM 작업목록 

emrocloud prod eks 업그레이드
emrocloud prod eks node개수 축소



### opensearch 자원삭제 검토

~~~ sh
kubectl get node -L 
kubectl get node -L role
kubectl get node -l role=worker


kubectl get pod -A --show-labels
~~~


~~~ sh

ENDPOINT_HOST=vpc-emrocloud-prod-npzvwvz7kx23ootmi7wvdazdtm.ap-northeast-2.es.amazonaws.com

# 전체 인덱스 문서 수·최종 업데이트 시각
curl -s -H "X-Amz-Content-Sha256: UNSIGNED-PAYLOAD" \
  -H "Host: ${ENDPOINT_HOST}" \
  --aws-sigv4 "aws:amzn-opensearch:ap-northeast-2" \
  "${ENDPOINT}/_cat/indices?h=index,docs.count,store.size,creation.date.string,health" | sort
# 특정 인덱스의 통계
curl -s --aws-sigv4 "aws:amzn-opensearch:ap-northeast-2" \
  "${ENDPOINT}/my-index-*/_stats?pretty&human" | jq '.indices[].total.docs'




curl -v --aws-sigv4 "aws:es:ap-northeast-2" -w '\nHTTP_STATUS=%{http_code}\n' "https://vpc-emrocloud-prod-npzvwvz7kx23ootmi7wvdazdtm.ap-northeast-2.es.amazonaws.com/_cluster/health"


~~~


### deepsecurity agent 설치 현황

emrocloud-prod-was-1a
emrocloud-prod-web-1a
anysign-prod-was
anysign-prod-web


딥시큐리티 ds 설치ㅠㅏㄹ때 메가존에 문의가 필요함 
- 운영에서 이중화가 필요할지도

### jenkns vm 업데이트 ? 



### whatap 

화요일 저녁 운영emrocloud 설치
was, IF, eks(?)



### caidentia cloud admin devops


jenkins 플러그인 등 기술들 김성훈 프로께 설명회 부탁 