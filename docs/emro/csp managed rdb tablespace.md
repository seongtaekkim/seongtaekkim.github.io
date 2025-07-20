


# aws
테이블스페이스 생성할 땡 물리적인 디렉터리 분리가 안된다
aws에서 관리하는 같은 파일시스템 내부에서 폴더 단위로만 생성이 가능함.


https://docs.aws.amazon.com/ko_kr/AmazonRDS/latest/UserGuide/PostgreSQL.Concepts.General.FeatureSupport.Tablespaces.html

![[Pasted image 20250718092157.png]]

# azure
테이블스페이스 생성 지원을 하지 않는다.
azuresu 만 생성이 가능하고 이는 azure 의 관리형 계정으로서
사용자는 해당 권한으로 에스컬레이션이 불가하다.

https://learn.microsoft.com/ko-kr/azure/postgresql/flexible-server/concepts-limits](https://learn.microsoft.com/ko-kr/azure/postgresql/flexible-server/concepts-limits)

[https://learn.microsoft.com/en-us/answers/questions/812177/azure-database-for-postgresql-tablespaces](https://learn.microsoft.com/en-us/answers/questions/812177/azure-database-for-postgresql-tablespaces)



![[Pasted image 20250718092622.png]]


