

21시 55분 스탑 거의 즉시삭제
22시 01분 스타트 - > 노드 레디까지 1분정도 소요
- 22시 10분 데모, ct 첫 팟 running 확인
- 22시 02분 npki running & 파일생성확인
emrocloud demo
emrocloud dev 
pending 확인

npki 데본팟 삭제확인
-> 깃랩이 뜨기전에 뜨면 파일생성안됨을 유의해야함


인프라는 17분 걸림 
중간에 노드그룹도 죽여서 죽었는 지 단독으로만 저렇게 죽는지는 체크해봐야해.


카프카
카프카유아이
깃랩
데시보드

32분 스타드
10분 최소 걸림 

카프카 이미지 오류로 인한 변경
 image: docker.io/bitnami/os-shell:11-debian-11-r4


=> 인프라를 띄우고 완료되면 비지니스 띄우던지
아니면 둘다 띄우고 엔피케이아이 재실행 하던지 선택이 필요함

kubectl rollout restart ds npki-worker -n kube-system