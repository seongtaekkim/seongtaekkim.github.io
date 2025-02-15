---
layout: default
title: 로드벨런서 ipvs
parent: eks network
grand_parent: eks 실습
nav_order: 4
---

`strictARP: true` : ARP 패킷을 보다 엄격하게 처리하겠다는 설정.

- stric ARP가 활성화되면, 노드의 인터페이스는 자신에게 할당된 IP주소에 대해서만 ARP 응답을 보낸다.

~~~sh
# 파일 작성
cat <<EOT> kind-svc-2w-ipvs.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
featureGates:
  "InPlacePodVerticalScaling": true
  "MultiCIDRServiceAllocator": true
nodes:
- role: control-plane
  labels:
    mynode: control-plane
    topology.kubernetes.io/zone: ap-northeast-2a
  extraPortMappings:
  - containerPort: 30000
    hostPort: 30000
  - containerPort: 30001
    hostPort: 30001
  - containerPort: 30002
    hostPort: 30002
  - containerPort: 30003
    hostPort: 30003
  - containerPort: 30004
    hostPort: 30004
  kubeadmConfigPatches:
  - |
    kind: ClusterConfiguration
    apiServer:
      extraArgs:
        runtime-config: api/all=true
    controllerManager:
      extraArgs:
        bind-address: 0.0.0.0
    etcd:
      local:
        extraArgs:
          listen-metrics-urls: http://0.0.0.0:2381
    scheduler:
      extraArgs:
        bind-address: 0.0.0.0
  - |
    kind: KubeProxyConfiguration
    metricsBindAddress: 0.0.0.0
    ipvs:
      strictARP: true
- role: worker
  labels:
    mynode: worker1
    topology.kubernetes.io/zone: ap-northeast-2a
- role: worker
  labels:
    mynode: worker2
    topology.kubernetes.io/zone: ap-northeast-2b
- role: worker
  labels:
    mynode: worker3
    topology.kubernetes.io/zone: ap-northeast-2c
networking:
  podSubnet: 10.10.0.0/16
  serviceSubnet: 10.200.1.0/24
  kubeProxyMode: "ipvs"        
EOT

# k8s 클러스터 설치
kind create cluster --config kind-svc-2w-ipvs.yaml --name myk8s --image kindest/node:v1.31.0
docker ps

# 노드에 기본 툴 설치
docker exec -it myk8s-control-plane sh -c 'apt update && apt install tree psmisc lsof wget bsdmainutils bridge-utils net-tools dnsutils ipset ipvsadm nfacct tcpdump ngrep iputils-ping arping git vim arp-scan -y'
for i in worker worker2 worker3; do echo ">> node myk8s-$i <<"; docker exec -it myk8s-$i sh -c 'apt update && apt install tree psmisc lsof wget bsdmainutils bridge-utils net-tools dnsutils ipset ipvsadm nfacct tcpdump ngrep iputils-ping arping -y'; echo; done


# kube-proxy configmap 확인
kubectl describe cm -n kube-system kube-proxy
...
mode: ipvs
ipvs: # 아래 각각 옵션 의미 조사해보자!
  excludeCIDRs: null
  minSyncPeriod: 0s
  scheduler: ""
  strictARP: true  # MetalLB 동작을 위해서 true 설정 변경 필요
  syncPeriod: 0s
  tcpFinTimeout: 0s
  tcpTimeout: 0s
  udpTimeout: 0s
...

# strictARP: true는 ARP 패킷을 보다 엄격하게 처리하겠다는 설정입니다.
## IPVS 모드에서 strict ARP가 활성화되면, 노드의 인터페이스는 자신에게 할당된 IP 주소에 대해서만 ARP 응답을 보내게 됩니다. 
## 이는 IPVS로 로드밸런싱할 때 ARP 패킷이 잘못된 인터페이스로 전달되는 문제를 방지합니다.
## 이 설정은 특히 클러스터 내에서 여러 노드가 동일한 IP를 갖는 VIP(Virtual IP)를 사용하는 경우 중요합니다.


# 노드 별 네트워트 정보 확인 : kube-ipvs0 네트워크 인터페이스 확인
for i in control-plane worker worker2 worker3; do echo ">> node myk8s-$i <<"; docker exec -it myk8s-$i ip -c route; echo; done
for i in control-plane worker worker2 worker3; do echo ">> node myk8s-$i <<"; docker exec -it myk8s-$i ip -c addr; echo; done
for i in control-plane worker worker2 worker3; do echo ">> node myk8s-$i <<"; docker exec -it myk8s-$i ip -br -c addr show kube-ipvs0; echo; done


# kube-ipvs0 에 할당된 IP(기본 IP + 보조 IP들) 정보 확인 
kubectl get svc,ep -A
NAMESPACE     NAME         TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)                  AGE
default       kubernetes   ClusterIP   10.200.1.1    <none>        443/TCP                  3m8s
kube-system   kube-dns     ClusterIP   10.200.1.10   <none>        53/UDP,53/TCP,9153/TCP   3m7s

# ipvsadm 툴로 부하분산 되는 정보 확인 : 서비스의 IP와 서비스에 연동되어 있는 파드의 IP 를 확인
## Service IP(VIP) 처리를 ipvs 에서 담당 -> 이를 통해 iptables 에 체인/정책이 상당 수준 줄어듬
for i in control-plane worker worker2 worker3; do echo ">> node myk8s-$i <<"; docker exec -it myk8s-$i ipvsadm -Ln ; echo; done

## IPSET 확인
docker exec -it myk8s-worker ipset -h
docker exec -it myk8s-worker ipset -L


# iptables 정보 확인 : 정책 갯수를 iptables proxy 모드와 비교해보자
for i in filter nat mangle raw ; do echo ">> IPTables Type : $i <<"; docker exec -it myk8s-control-plane  iptables -t $i -S ; echo; done
for i in filter nat mangle raw ; do echo ">> IPTables Type : $i <<"; docker exec -it myk8s-worker  iptables -t $i -S ; echo; done
for i in filter nat mangle raw ; do echo ">> IPTables Type : $i <<"; docker exec -it myk8s-worker2 iptables -t $i -S ; echo; done
for i in filter nat mangle raw ; do echo ">> IPTables Type : $i <<"; docker exec -it myk8s-worker3 iptables -t $i -S ; echo; done

# 각 노드 bash 접속
docker exec -it myk8s-control-plane bash
docker exec -it myk8s-worker bash
docker exec -it myk8s-worker2 bash
docker exec -it myk8s-worker3 bash
----------------------------------------
exit
----------------------------------------


# mypc 컨테이너 기동 : kind 도커 브리지를 사용하고, 컨테이너 IP를 직접 지정 혹은 IP 지정 없이 배포
docker run -d --rm --name mypc --network kind --ip 172.18.0.100 nicolaka/netshoot sleep infinity
혹은
docker run -d --rm --name mypc --network kind nicolaka/netshoot sleep infinity

docker ps
~~~



### ipvs-proxy

![](/img/lb-14.png)



#### manifests

~~~sh
cat <<EOT> 3pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: webpod1
  labels:
    app: webpod
spec:
  nodeName: myk8s-worker
  containers:
  - name: container
    image: traefik/whoami
  terminationGracePeriodSeconds: 0
---
apiVersion: v1
kind: Pod
metadata:
  name: webpod2
  labels:
    app: webpod
spec:
  nodeName: myk8s-worker2
  containers:
  - name: container
    image: traefik/whoami
  terminationGracePeriodSeconds: 0
---
apiVersion: v1
kind: Pod
metadata:
  name: webpod3
  labels:
    app: webpod
spec:
  nodeName: myk8s-worker3
  containers:
  - name: container
    image: traefik/whoami
  terminationGracePeriodSeconds: 0
EOT
~~~

~~~sh
cat <<EOT> netpod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: net-pod
spec:
  nodeName: myk8s-control-plane
  containers:
  - name: netshoot-pod
    image: nicolaka/netshoot
    command: ["tail"]
    args: ["-f", "/dev/null"]
  terminationGracePeriodSeconds: 0
EOT
~~~

~~~sh
cat <<EOT> svc-clusterip.yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-clusterip
spec:
  ports:
    - name: svc-webport
      port: 9000        # 서비스 IP 에 접속 시 사용하는 포트 port 를 의미
      targetPort: 80    # 타킷 targetPort 는 서비스를 통해서 목적지 파드로 접속 시 해당 파드로 접속하는 포트를 의미
  selector:
    app: webpod         # 셀렉터 아래 app:webpod 레이블이 설정되어 있는 파드들은 해당 서비스에 연동됨
  type: ClusterIP       # 서비스 타입
EOT
~~~





#### 생성 및 확인 : IPVS Proxy Mode

~~~sh
# 생성
kubectl apply -f 3pod.yaml,netpod.yaml,svc-clusterip.yaml

# 파드와 서비스 사용 네트워크 대역 정보 확인 
kubectl cluster-info dump | grep -m 2 -E "cluster-cidr|service-cluster-ip-range"

# 확인
kubectl get pod -owide
kubectl get svc svc-clusterip
kubectl describe svc svc-clusterip
kubectl get endpoints svc-clusterip
kubectl get endpointslices -l kubernetes.io/service-name=svc-clusterip

# 노드 별 네트워트 정보 확인 : kube-ipvs0 네트워크 인터페이스 확인
## ClusterIP 생성 시 kube-ipvs0 인터페이스에 ClusterIP 가 할당되는 것을 확인
for i in control-plane worker worker2 worker3; do echo ">> node myk8s-$i <<"; docker exec -it myk8s-$i ip -c addr; echo; done
for i in control-plane worker worker2 worker3; do echo ">> node myk8s-$i <<"; docker exec -it myk8s-$i ip -br -c addr show kube-ipvs0; echo; done
for i in control-plane worker worker2 worker3; do echo ">> node myk8s-$i <<"; docker exec -it myk8s-$i ip -d -c addr show kube-ipvs0; echo; done

# 변수 지정
CIP=$(kubectl get svc svc-clusterip -o jsonpath="{.spec.clusterIP}")
CPORT=$(kubectl get svc svc-clusterip -o jsonpath="{.spec.ports[0].port}")
echo $CIP $CPORT

# ipvsadm 툴로 부하분산 되는 정보 확인
## 10.200.1.216(TCP 9000) 인입 시 3곳의 목적지로 라운드로빈(rr)로 부하분산하여 전달됨을 확인 : 모든 노드에서 동일한 IPVS 분산 설정 정보 확인
## 3곳의 목적지는 각각 서비스에 연동된 목적지 파드 3개이며, 전달 시 출발지 IP는 마스커레이딩 변환 처리
docker exec -it myk8s-control-plane ipvsadm -Ln -t $CIP:$CPORT
for i in control-plane worker worker2 worker3; do echo ">> node myk8s-$i <<"; docker exec -it myk8s-$i ipvsadm -Ln -t $CIP:$CPORT ; echo; done

# ipvsadm 툴로 부하분산 되는 현재 연결 정보 확인 : 추가로 --rate 도 있음
docker exec -it myk8s-control-plane ipvsadm -Ln -t $CIP:$CPORT --stats
for i in control-plane worker worker2 worker3; do echo ">> node myk8s-$i <<"; docker exec -it myk8s-$i ipvsadm -Ln -t $CIP:$CPORT --stats ; echo; done

docker exec -it myk8s-control-plane ipvsadm -Ln -t $CIP:$CPORT --rate
for i in control-plane worker worker2 worker3; do echo ">> node myk8s-$i <<"; docker exec -it myk8s-$i ipvsadm -Ln -t $CIP:$CPORT --rate ; echo; done


# iptables 규칙 확인 : ipset list 를 활용
docker exec -it myk8s-control-plane iptables -t nat -S | grep KUBE-CLUSTER-IP

# ipset list 정보를 확인 : KUBE-CLUSTER-IP 이름은 아래 6개의 IP:Port 조합을 지칭
# 예를 들면 ipset list 를 사용하지 않을 경우 6개의 iptables 규칙이 필요하지만, ipset 사용 시 1개의 규칙으로 가능
docker exec -it myk8s-control-plane ipset list KUBE-CLUSTER-IP
Name: KUBE-CLUSTER-IP
Type: hash:ip,port
Revision: 7
Header: family inet hashsize 1024 maxelem 65536 bucketsize 12 initval 0x6343ff52
Size in memory: 456
References: 3
Number of entries: 5
Members:
10.200.1.1,tcp:443
10.200.1.10,tcp:53
10.200.1.10,udp:53
10.200.1.245,tcp:9000
10.200.1.10,tcp:9153
~~~

![](/img/lb-15.png)









### IPVS 정보 확인 및 서비스 접속 확인

![](/img/lb-16.png)

~~~sh
for i in control-plane worker worker2 worker3; do echo ">> node myk8s-$i <<"; docker exec -it myk8s-$i ipvsadm -Ln -t $CIP:$CPORT ; echo; done

# 변수 지정
CIP=$(kubectl get svc svc-clusterip -o jsonpath="{.spec.clusterIP}")
CPORT=$(kubectl get svc svc-clusterip -o jsonpath="{.spec.ports[0].port}")
echo $CIP $CPORT

# 컨트롤플레인 노드에서 ipvsadm 모니터링 실행 : ClusterIP 접속 시 아래 처럼 연결 정보 확인됨
watch -d "docker exec -it myk8s-control-plane ipvsadm -Ln -t $CIP:$CPORT --stats; echo; docker exec -it myk8s-control-plane ipvsadm -Ln -t $CIP:$CPORT --rate"

--------------------------

# 서비스 IP 변수 지정 : svc-clusterip 의 ClusterIP주소
SVC1=$(kubectl get svc svc-clusterip -o jsonpath={.spec.clusterIP})
echo $SVC1

# TCP 80,9000 포트별 접속 확인 : 출력 정보 의미 확인
kubectl exec -it net-pod -- curl -s --connect-timeout 1 $SVC1:9000
kubectl exec -it net-pod -- curl -s --connect-timeout 1 $SVC1:9000 | grep Hostname
kubectl exec -it net-pod -- curl -s --connect-timeout 1 $SVC1:9000 | grep Hostname

# 서비스(ClusterIP) 부하분산 접속 확인 : 부하분산 비률 확인
kubectl exec -it net-pod -- zsh -c "for i in {1..10};   do curl -s $SVC1:9000 | grep Hostname; done | sort | uniq -c | sort -nr"
kubectl exec -it net-pod -- zsh -c "for i in {1..100};  do curl -s $SVC1:9000 | grep Hostname; done | sort | uniq -c | sort -nr"
kubectl exec -it net-pod -- zsh -c "for i in {1..1000}; do curl -s $SVC1:9000 | grep Hostname; done | sort | uniq -c | sort -nr"
혹은
kubectl exec -it net-pod -- zsh -c "for i in {1..100};   do curl -s $SVC1:9000 | grep Hostname; sleep 1; done"
kubectl exec -it net-pod -- zsh -c "for i in {1..100};   do curl -s $SVC1:9000 | grep Hostname; sleep 0.1; done"
kubectl exec -it net-pod -- zsh -c "for i in {1..10000}; do curl -s $SVC1:9000 | grep Hostname; sleep 0.01; done"

# 반복 접속
kubectl exec -it net-pod -- zsh -c "while true; do curl -s --connect-timeout 1 $SVC1:9000 | egrep 'Hostname|RemoteAddr|Host:'; date '+%Y-%m-%d %H:%M:%S' ; echo '--------------' ;  sleep 1; done"
~~~

#### conntrack 확인

~~~sh
docker exec -it myk8s-control-plane bash
----------------------------------------
conntrack -h
conntrack -E
conntrack -C
conntrack -S
conntrack -L --src 10.10.0.6 # net-pod IP
conntrack -L --dst $SVC1     # service ClusterIP
exit
----------------------------------------

# (참고) IPVS hash 확인
ipvsadm -Ln
ipvsadm -Ln -t <CluterIP>:<Port>
ipvsadm -Ln -t <NodeIP>:<NodePort>

# (옵션) IPVS current IPVS connections
ipvsadm -Ln -c

# (옵션) IPVS statistics information
ipvsadm -Ln --stats
ipvsadm -Ln --stats -t <NodeIP>:$NPORT
watch -d ipvsadm -Ln --stats -t <NodeIP>:$NPORT

# (옵션) IPVS rate information
ipvsadm -Ln --rate
ipvsadm -Ln --rate -t <NodeIP>:$NPORT
watch -d ipvsadm -Ln --rate -t <NodeIP>:$NPORT

# iptabels 확인 >> 서비스 정책/룰 추가 확인해보자! , 정책 갯수를 iptables proxy 모드와 비교해보자
for i in filter nat mangle raw ; do echo ">> IPTables Type : $i <<"; docker exec -it myk8s-control-plane  iptables -t $i -S ; echo; done
for i in filter nat mangle raw ; do echo ">> IPTables Type : $i <<"; docker exec -it myk8s-worker  iptables -t $i -S ; echo; done
for i in filter nat mangle raw ; do echo ">> IPTables Type : $i <<"; docker exec -it myk8s-worker2 iptables -t $i -S ; echo; done
for i in filter nat mangle raw ; do echo ">> IPTables Type : $i <<"; docker exec -it myk8s-worker3 iptables -t $i -S ; echo; done
~~~







