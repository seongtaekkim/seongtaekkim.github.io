

#### 세션 상한


~~~ sh
max_connections=1000
shared_buffers=1024

~~~


~~~ sh

# 1) 커널 파라미터 (SHM / SEM) & FD 상한
cat <<'EOF' | sudo tee /etc/sysctl.d/99-postgres.conf
kernel.shmmax = 1073741824
kernel.shmall = 262144
kernel.sem    = 32768 2147483647 2147483646 512   # 1000 세션도 커버
EOF
sudo sysctl --system

# 2) 파일 디스크립터
cat <<'EOF' | sudo tee /etc/security/limits.d/99-postgres.conf
postgres soft nofile 40000
postgres hard nofile 40000
EOF
sudo systemctl edit --full postgresql-17   # [Service] LimitNOFILE=40000 추가
sudo systemctl daemon-reload

# 3) Postgres 파라미터
sudo -u postgres psql -c "ALTER SYSTEM SET max_connections = 1000;"
sudo systemctl restart postgresql-17


~~~

| 파라미터                                          | 산출 로직                                         | 공식(또는 근사치)                                            | 200 세션 적용 예                                              | 1 000 세션 적용 예                                                              |
| --------------------------------------------- | --------------------------------------------- | ----------------------------------------------------- | -------------------------------------------------------- | -------------------------------------------------------------------------- |
| `shared_buffers` _(참고)_                       | OS RAM 일부를 PostgreSQL 버퍼 캐시로 확보               | `RAM × 0.25 ≈ 25 %`(RAM이 64 GiB↓라면 25 % 전후가 일반적)      | 4 GiB × 0.25 = **1 GiB**                                 | 4 GiB × 0.25 = **1 GiB** → 메모리 부족 우려로 예시에서는 512 MiB로 축소                    |
| **`kernel.shmmax`**                           | 하나의 SHM 세그먼트 최대 크기 ≥ `shared_buffers`         | `shmmax ≥ shared_buffers`                             | **1 GiB** (1 073 741 824 B)                              | **1 GiB** (같은 값)※ 실제 shared_buffers 512 MiB라면 512 MiB로 내려도 무방              |
| **`kernel.shmall`**                           | 시스템 전체 SHM page 합계 ≥ `shared_buffers / 4 KiB` | `shmall ≥ shmmax ÷ PAGE_SIZE`PAGE_SIZE(대부분) = 4 096 B | 1 GiB ÷ 4 096 = 262 144 page → **262 144**               | **262 144**                                                                |
| **`kernel.sem`**`SEMMSL SEMMNS SEMMNI SEMOPM` | 첫값(SEMMSL) ≥ 세션당 세마포어 16개                     | `SEMMSL ≥ max_connections × 16`                       | 200 × 16 = 3 200 → 근사 상한 **4 096**                       | 1 000 × 16 = 16 000 → **32 768** (여유치) 나머지 3개는 매우 큰 기본값(2¹³¹ – 1 등) 그대로 사용 |
| **`nofile`** _(soft/hard + `LimitNOFILE`)_    | 백엔드 1개가 열 FD ≈ 30개(소켓·WAL·테이블 등)              | `nofile ≥ (max_connections × 30) × 1.3`(30 % 여유분)     | 200 × 30 = 6 000 → 6 000 × 1.3 ≈ 7 800 → **10 000**으로 올림 | 1 000 × 30 = 30 000 → 30 000 × 1.3 ≈ 39 000 → **40 000**                   |
