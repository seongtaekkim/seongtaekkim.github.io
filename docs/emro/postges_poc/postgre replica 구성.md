
#### 개요
caidentia cloud admin 구성에 managed postgre primary/replica 구성이 필요하다. - es 등에서 읽기전용 db로 쿼리하도록 하기 위함.
postgre primary/replica 구성을 해보고 실제 데이터가 복사되는 기능 동작(WAL) 되는지 PoC 해본다.


#### 일정
07.03 postgre replica 전용 VM 생성신청 (primary와 같은 구성으로 생성)
07.07 방화벽신청




~~~ sh
sh -p 24477 stk.kim@192.168.6.120

~~~

24477, 5432


~~~
/var/lib/pgsql/17/data/postgresql.conf

# listen_addresses = 'localhost' 를 아래처럼 변경
listen_addresses = '*'          # what IP address(es) to listen on;
~~~


~~~


# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     peer
# IPv4 local connections:
host    all             all             127.0.0.1/32            scram-sha-256
host    all             all             0.0.0.0/0               md5
# IPv6 local connections:
host    all             all             ::1/128                 scram-sha-256
# Allow replication connections from localhost, by a user with the
# replication privilege.
local   replication     all                                     peer
host    replication     all             127.0.0.1/32            scram-sha-256
host    replication     all             ::1/128                 scram-sha-256



~~~


아래는 고려할 설정
~~~
# 로컬, 내부 IP 허용 (예시)
host    all           all      127.0.0.1/32      trust
host    all           all      192.168.0.0/24    md5

# 외부 허용 (모든 IPv4)
host    all           all      0.0.0.0/0         md5
~~~


방화벽 확인
~~~
firewall-cmd --list-all
sudo ss -tnlp | grep 5432
~~~




## primary 구성


##### 복제 전용 롤 생성

~~~ sql
sudo -u postgres psql -c \
  "CREATE ROLE replicator WITH REPLICATION LOGIN ENCRYPTED PASSWORD 'replicator';"

~~~


##### `postgresql.conf` 변경
- sudo vi /var/lib/pgsql/17/data/postgresql.conf
~~~ sql
# WAL level 설정 (필수)
wal_level = replica

# 동시 복제 연결 허용 수
max_wal_senders = 5

# standby 서버가 과거 WAL을 참조할 수 있도록 보관량 (MB)
wal_keep_size = 1024

# 이미 '*' 로 바인딩 돼 있어야 함; 외부접속 허용 확인
listen_addresses = '*'

~~~


##### `pg_hba.conf`에 복제 권한 추가
- sudo vi /var/lib/pgsql/17/data/pg_hba.conf
~~~ conf
# 복제용 연결 허용 (replica VM IP가 192.168.6.120)
host  replication  replicator  192.168.6.120/32  md5
~~~


~~~ sh
sudo systemctl reload postgresql-17
~~~



### replica 구성

##### PostgreSQL 서비스 중지 & 데이터 초기화
~~~ sh
sudo systemctl stop postgresql-17
sudo -u postgres rm -rf /var/lib/pgsql/17/data/*

~~~

##### Base backup 수행
~~~ sh

sudo -u postgres pg_basebackup \
  -h 192.168.6.115        \
  -D /var/lib/pgsql/17/data \
  -U replicator        \
  -Fp -Xs -P -R

~~~

![[Pasted image 20250707144210.png]]


standby.signal 파일 생성 확인

primary_conninfo 위치

##### `postgresql.conf` 확인 (Replica)
- 기본적으로 `standby.signal` 이 존재하면 hot_standby=on 이 자동 적용되지만, 명시하려면:
~~~ conf
hot_standby = on
~~~


~~~ sh

sudo systemctl start postgresql-17
sudo systemctl status postgresql-17
~~~


## 복제상태확인


##### Primary side
- `client_addr` 에 replica IP, `state` 가 `streaming` 이어야 정상.
~~~ sh
sudo -u postgres psql -c "SELECT client_addr, state, sent_lsn, write_lsn
  FROM pg_stat_replication;"

~~~

##### Replica side
- `pg_is_in_recovery()` 가 `true` 면 standby 모드 활성화.
~~~ sh
sudo -u postgres psql -c "SELECT pg_is_in_recovery(), pg_last_wal_receive_lsn();"
~~~





## 4. 애플리케이션 분리

- **쓰기/읽기 분리**
    
    - 쓰기는 primary VM의 IP/호스트명,
        
    - 읽기는 replica VM의 IP/호스트명에 접속하도록 커넥션 풀 또는 애플리케이션 설정 분리.
        
- **참고**
    
    - 비동기 복제이므로 약간의 지연(latency)이 발생할 수 있습니다.
        
    - 장애(failover) 자동화는 Patroni, repmgr 같은 툴 검토를 권장합니다.
        

이렇게 하면 primary는 읽기·쓰기, replica는 읽기 전용으로 동작하는 기본 스트리밍 복제 환경이 완성됩니다.


