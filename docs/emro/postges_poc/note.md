
## 1. Replica가 처음 연결되면 “빈” 상태여도 모든 객체가 복사되나요?

- **Base backup** 단계에서 Primary의 데이터 디렉터리 전체를 복제하기 때문에, 스키마·테이블·데이터는 물론 시스템 카탈로그(역할, 권한, 테이블스페이스, 확장 등)까지 **물리적으로 동일하게** 복사됩니다.
    
- 그 이후에는 Primary의 WAL(Write-Ahead Log)을 실시간으로 받아서 변경분만 적용하는 구조라, 처음에 아무것도 없어도 Primary 클러스터 전체가 복제됩니다.
    

## 2. Replica에 접속하면 오직 SELECT만 가능한가요?

- 기본적으로 Replica(Standby)는 읽기 전용 모드(`hot_standby = on`)로 동작합니다.
    
    - **SELECT** 등 읽기 쿼리는 허용
        
    - **INSERT/UPDATE/DELETE**, DDL(`CREATE`, `DROP` 등)은 허용되지 않습니다.
        
- 다만 세션 로컬의 **임시 테이블(TEMPORARY TABLE)** 생성·조회 정도는 가능합니다.
    

## 3. Replica를 중지했다가 재시작하면 자동으로 Primary에 재연결되나요?

- 네. Replica 데이터 디렉터리에 남아 있는 `standby.signal` 파일과 `primary_conninfo` 설정을 참고해, 서비스 재시작 시 자동으로 Primary에 다시 접속하여 WAL 스트리밍을 재개합니다.
    
- 단, 중지 기간 동안 Primary에서 넘쳐난 WAL 세그먼트가 로컬에 남아 있지 않으면(예: `wal_keep_size` 부족), 복제이력(아카이브)에 접근 가능하도록 `archive_command` 설정이 필요할 수 있습니다.
    

## 4. Primary가 중단되면 어떻게 되나요?

- Replica는 계속 **읽기 전용** standby 상태로 남아 있지만, WAL을 더 받을 수 없으므로 최신 상태로의 동기화는 중단됩니다.
    
- 클라이언트의 쓰기 요청은 “Primary not available” 에러가 납니다.
    
- **가용성 확보**를 위해선 수동 또는 Patroni·repmgr 같은 툴을 이용한 **Failover(Replica 승격 → New Primary)** 절차를 거쳐야 합니다.
    

## 5. 복제 방식과 내부 기술은 무엇인가요?

- PostgreSQL 복제는 **Physical Streaming Replication** 방식이 기본입니다.
    
    1. **WAL Level**을 `replica` 이상으로 설정
        
    2. Primary의 WAL(Write-Ahead Log) 이벤트를 **TCP 스트리밍**으로 Replica에 전달
        
    3. Replica는 받은 WAL을 **즉시 재생(replay)** 하여 디스크 상태를 동기화
        
- 이 기능은 PostgreSQL 코어에 내장된 “Streaming Replication Protocol”과 `pg_basebackup` 유틸, `wal_receiver`·`wal_sender` 백그라운드 프로세스를 통해 구현되어 있습니다.