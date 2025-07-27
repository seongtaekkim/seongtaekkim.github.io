

GNS3 노드 & veth 구성
![[baddass01.png]]



최종 네트워크 구성도 (Underlay + Overlay)
~~~ sh

                         (Underlay /30 IP + OSPF)
                   +--------------------------------+
                   |  Spine / RR  (_seongtki-1)     |
                   |  eth0 10.1.1.1/30              |
                   |  eth1 10.1.1.5/30              |
                   +----------+----------+----------+
                              |          |
          /30 10.1.1.0/30     |          |  /30 10.1.1.4/30
                              |          |
                   +----------+          +----------+
                   |                                |
          Leaf‑1 (_seongtki-2)                Leaf‑2 (_seongtki-3)
          eth0 10.1.1.2/30                    eth1 10.1.1.6/30
          eth1 —— br0 —— vxlan10              eth0 —— br0 —— vxlan10
               \                               /
               Host‑1                      Host‑2
          (20.1.1.1/24)                 (20.1.1.2/24)

~~~





## Step 1 – Underlay 인터페이스에 IP 부여

|노드|명령 (Linux)|완료되는 것|검증|
|---|---|---|---|
|RR|`ip addr add 10.1.1.1/30 dev eth0`  <br>`ip addr add 10.1.1.5/30 dev eth1`|P2P 링크 IP 설정 시작|`ip -br addr` 로 주소 확인|
|Leaf‑1|`ip addr add 10.1.1.2/30 dev eth0`|RR↔Leaf‑1 링크 완성|`ping -c1 10.1.1.1`|
|Leaf‑2|`ip addr add 10.1.1.6/30 dev eth1`|RR↔Leaf‑2 링크 완성|`ping -c1 10.1.1.5`|

> **검증 결과** : 각 Leaf 가 **자신과 RR 사이 /30 주소**로 ping 성공 → 케이블 정상.

---

## Step 2 – OSPF Area 0 활성 (Underlay 라우팅)


`# 모든 노드에서 vtysh -c 'configure terminal' \       -c 'router ospf' \       -c 'network 0.0.0.0/0 area 0' \       -c 'exit'`

|완료되는 것|검증|
|---|---|
|RR·Leaf 들이 **OSPF Neighbor** 로 붙고, /30 경로를 교환|RR 또는 Leaf 에서 `vtysh -c "show ip ospf neighbor"` → State `FULL` 2개|

---

## Step 3 – Loopback 주소 설정

| 노드     | 명령                              | 검증                               |
| ------ | ------------------------------- | -------------------------------- |
| RR     | `ip addr add 1.1.1.1/32 dev lo` | `ping -c1 1.1.1.1` (자체 loopback) |
| Leaf‑1 | `ip addr add 1.1.1.2/32 dev lo` | Leaf‑1 → RR `ping -c1 1.1.1.1`   |
| Leaf‑2 | `ip addr add 1.1.1.3/32 dev lo` | Leaf‑2 ↔ RR ping                 |
|        |                                 |                                  |

> **OSPF** 가 Loopback /32 경로도 배포하므로 세 노드 모두 서로의 _1.1.1.x_ 로 ping 이 돼야 함.

---

## Step 4 – BGP 프로세스 기동 (AS 1)

### 4‑1 RR (Route‑Reflector) 설정

~~~ sh

vtysh <<'EOF'
configure terminal
  router bgp 1
    bgp router-id 1.1.1.1
    neighbor ibgp peer-group
    neighbor ibgp remote-as 1
    neighbor ibgp update-source lo
    bgp listen range 1.1.1.0/29 peer-group ibgp
  exit
EOF

~~~

### 4‑2 Leaf‑1 / Leaf‑2


~~~ sh

# Leaf 2
vtysh <<'EOF'
configure terminal
  router bgp 1
    bgp router-id 1.1.1.2 
    neighbor 1.1.1.1 remote-as 1
    neighbor 1.1.1.1 update-source lo
  exit
EOF



# Leaf 3
vtysh <<'EOF'
configure terminal
  router bgp 1
    bgp router-id 1.1.1.3
    neighbor 1.1.1.1 remote-as 1
    neighbor 1.1.1.1 update-source lo
  exit
EOF

~~~

|완료되는 것|검증|
|---|---|
|**iBGP 세션** (TCP 179, Loopback↔Loopback) 수립|각 노드 `vtysh -c "show bgp summary"` → `Estab` 2 / 1|

---

## Step 5 – EVPN Address‑Family + Route‑Reflector 기능

### 5‑1 RR

~~~ bash

vtysh -c 'configure terminal' \
      -c 'router bgp 1' \
      -c 'address-family l2vpn evpn' \
      -c 'neighbor ibgp activate' \
      -c 'neighbor ibgp route-reflector-client' \
      -c 'exit-address-family'


~~~

### 5‑2 Leaf‑1 / Leaf‑2



~~~ sh

vtysh -c 'configure terminal' \
      -c 'router bgp 1' \
      -c 'address-family l2vpn evpn' \
      -c 'neighbor 1.1.1.1 activate' \
      -c 'advertise-all-vni' \
      -c 'exit-address-family'

~~~

|완료되는 것|검증|
|---|---|
|**EVPN NLRI 교환 채널** 준비, RR 가 reflect client 지정|`vtysh -c "show bgp l2vpn evpn summary"` → AF‑Specific State `Established(0/0/0)`|

> 아직 VNI 경로가 없으므로 NLRI 카운트는 0.

---

## Step 6 – Leaf 에 VXLAN VNI 10 + 브리지 생성

### 공통 명령 (Leaf‑1 예)

~~~ sh

ip link add br0 type bridge
ip link set br0 up

ip link add vxlan10 type vxlan id 10 dstport 4789
ip link set vxlan10 up

# Leaf‑1: 호스트 NIC = eth1       Leaf‑2: 호스트 NIC = eth0
brctl addif br0 vxlan10
brctl addif br0 eth1


~~~


|완료되는 것|검증|
|---|---|
|**데이터플레인** 준비: VNI 10, 브리지 도메인 형성|`bridge link` 또는 `brctl show` – br0 포트로 eth?·vxlan10|

---

## Step 7 – 호스트 IP 설정

~~~ sh

# Host‑1 컨테이너
ip addr add 20.1.1.1/24 dev eth1   # (Leaf‑1 쪽 veth)
# Host‑2 컨테이너
ip addr add 20.1.1.2/24 dev eth0   # (Leaf‑2 쪽 veth)


~~~

|완료되는 것|검증|
|---|---|
|호스트가 **VNI 10 세그먼트** 에 참여|같은 Leaf 내 ping: Host‑1 → 20.1.1.1 Loopback OK|

---

## Step 8 – 첫 MAC 학습 & EVPN Type 2 전파 확인

1. **Host‑1 → Host‑2** 로 **Ping** 시도 → 실패해도 OK (ARP 트리거용).
    
2. Leaf‑1 에서
    
    ~~~ sh
    bridge fdb show dev vxlan10
	~~~
    
    → **MAC‑A(Host‑1)** 가 `self` 로 학습.
    
3. RR `vtysh -c "show bgp l2vpn evpn route-type 2"`  
    → 방금 학습된 MAC 이 **Type 2 NLRI** 로 표출.
    

|완료되는 것|검증|
|---|---|
|EVPN 컨트롤플레인 작동 확인|Leaf‑2 도 동일 명령 실행 → `dst 1.1.1.2` 로 원격 MAC 등록|

---

## Step 9 – 최종 Ping (Host‑1 → Host‑2)


~~~ sh

ping -c 3 20.1.1.2

~~~

- Leaf‑2 `bridge fdb show dev vxlan10` 에 **MAC‑A / dst 1.1.1.2** 가 존재    
- `tcpdump -nn -i any udp port 4789`(Leaf‑1) → VXLAN 캡슐 확인

---


| step | 완료된 기능          | 확인 명령                                          |
| ---- | --------------- | ---------------------------------------------- |
| 1    | 링크IP 부여         | `ip -br addr`, `ping` /30                      |
| 2    | OSPF Neighbor   | `show ip ospf neighbor`                        |
| 3    | Loopback 경로     | `ping 1.1.1.x`                                 |
| 4    | iBGP 세션         | `show bgp summary`                             |
| 5    | EVPN AF + RR    | `show bgp l2vpn evpn summary`                  |
| 6    | VNI10 브리지       | `brctl show`, `ip link show vxlan10`           |
| 7    | 호스트 접속          | 호스트 ping(동일 Leaf)                              |
| 8    | Type 2 전파       | `bridge fdb show`, `show bgp ... route-type 2` |
| 9    | End‑to‑End Ping | `ping 20.1.1.2` & VXLAN 캡처                     |






- **Underlay**
    
    - RR eth0 ↔ Leaf‑1 eth0 (10.1.1.0/30)
        
    - RR eth1 ↔ Leaf‑2 eth1 (10.1.1.4/30)
        
- **Overlay**
    
    - Leaf‑1 eth1 ↔ Host‑1 eth1
        
    - Leaf‑2 eth0 ↔ Host‑2 eth0
        
    - 두 Leaf 의 eth? + vxlan10 이 **br0** 에 묶여 VNI 10 의 L2 브리지 역할


| 논리적 이름                      | 컨테이너 Hostname     | 역할                     |
| --------------------------- | ----------------- | ---------------------- |
| **Spine / Route‑Reflector** | `_seongtki-1`     | iBGP EVPN RR + OSPF 백본 |
| **Leaf‑1 / VTEP‑1**         | `_seongtki-2`     | VNI 10 참여, Host‑1 수용   |
| **Host‑1**                  | `host_seongtki-1` | 20.1.1.1/24(PC A)      |
| **Leaf‑2 / VTEP‑2**         | `_seongtki-3`     | VNI 10 참여, Host‑2 수용   |
| **Host‑2**                  | `host_seongtki-2` | 20.1.1.2/24(PC B)      |
