---
layout: default
title: service
parent: eks network
grand_parent: eks 실습
nav_order: 5
---

### kind - cluster install

~~~sh
#
cat <<EOT> kind-svc-1w.yaml
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
EOT

# k8s 클러스터 설치
kind create cluster --config kind-svc-1w.yaml --name myk8s --image kindest/node:v1.31.0
docker ps

# 노드에 기본 툴 설치
docker exec -it myk8s-control-plane sh -c 'apt update && apt install tree psmisc lsof wget bsdmainutils bridge-utils net-tools ipset ipvsadm nfacct tcpdump ngrep iputils-ping arping git vim arp-scan -y'
for i in worker worker2 worker3; do echo ">> node myk8s-$i <<"; docker exec -it myk8s-$i sh -c 'apt update && apt install tree psmisc lsof wget bsdmainutils bridge-utils net-tools ipset ipvsadm nfacct tcpdump ngrep iputils-ping arping -y'; echo; done
~~~



설치 확인

~~~sh

# k8s v1.31.0 버전 확인
kubectl get node

# 노드 labels 확인
kubectl get nodes -o jsonpath="{.items[*].metadata.labels}" | grep mynode
kubectl get nodes -o jsonpath="{.items[*].metadata.labels}" | jq | grep mynode
kubectl get nodes -o jsonpath="{.items[*].metadata.labels}" | jq | grep 'topology.kubernetes.io/zone'

# kind network 중 컨테이너(노드) IP(대역) 확인 : 172.18.0.2~ 부터 할당되며, control-plane 이 꼭 172.18.0.2가 안될 수 도 있음
docker ps -q | xargs docker inspect --format '{{.Name}} {{.NetworkSettings.Networks.kind.IPAddress}}'
/myk8s-control-plane 172.18.0.4
/myk8s-worker 172.18.0.3
/myk8s-worker2 172.18.0.5
/myk8s-worker3 172.18.0.2

# 파드CIDR 과 Service 대역 확인 : CNI는 kindnet 사용
kubectl get cm -n kube-system kubeadm-config -oyaml | grep -i subnet
      podSubnet: 10.10.0.0/16
      serviceSubnet: 10.200.1.0/24
kubectl cluster-info dump | grep -m 2 -E "cluster-cidr|service-cluster-ip-range"

# feature-gates 확인 : https://kubernetes.io/docs/reference/command-line-tools-reference/feature-gates/
kubectl describe pod -n kube-system | grep feature-gates
      --feature-gates=InPlacePodVerticalScaling=true
kubectl describe pod -n kube-system | grep runtime-config
      --runtime-config=api/all=true

# MultiCIDRServiceAllocator : https://kubernetes.io/docs/tasks/network/extend-service-ip-ranges/
kubectl get servicecidr
NAME         CIDRS           AGE
kubernetes   10.200.1.0/24   2m13s


# 노드마다 할당된 dedicated subnet (podCIDR) 확인
kubectl get nodes -o jsonpath="{.items[*].spec.podCIDR}"
10.10.0.0/24 10.10.4.0/24 10.10.3.0/24 10.10.1.0/24

# kube-proxy configmap 확인
kubectl describe cm -n kube-system kube-proxy
...
mode: iptables
iptables:
  localhostNodePorts: null
  masqueradeAll: false
  masqueradeBit: null
  minSyncPeriod: 1s
  syncPeriod: 0s
...


# 노드 별 네트워트 정보 확인 : CNI는 kindnet 사용
for i in control-plane worker worker2 worker3; do echo ">> node myk8s-$i <<"; docker exec -it myk8s-$i ls /opt/cni/bin/; echo; done
for i in control-plane worker worker2 worker3; do echo ">> node myk8s-$i <<"; docker exec -it myk8s-$i cat /etc/cni/net.d/10-kindnet.conflist; echo; done
for i in control-plane worker worker2 worker3; do echo ">> node myk8s-$i <<"; docker exec -it myk8s-$i ip -c route; echo; done
for i in control-plane worker worker2 worker3; do echo ">> node myk8s-$i <<"; docker exec -it myk8s-$i ip -c addr; echo; done
for i in control-plane worker worker2 worker3; do echo ">> node myk8s-$i <<"; docker exec -it myk8s-$i ip -c -4 addr show dev eth0; echo; done

# iptables 정보 확인
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

# kind 설치 시 kind 이름의 도커 브리지가 생성된다 : 172.18.0.0/16 대역
docker network ls
docker inspect kind

# arp scan 해두기
docker exec -it myk8s-control-plane arp-scan --interfac=eth0 --localnet

# mypc 컨테이너 기동 : kind 도커 브리지를 사용하고, 컨테이너 IP를 직접 지정
docker run -d --rm --name mypc --network kind --ip 172.18.0.100 nicolaka/netshoot sleep infinity
docker ps
## 만약 kind 네트워크 대역이 다를 경우 위 IP 지정이 실패할 수 있으니, 그냥 IP 지정 없이 mypc 컨테이너 기동 할 것
## docker run -d --rm --name mypc --network kind nicolaka/netshoot sleep infinity

# 통신 확인
docker exec -it mypc ping -c 1 172.18.0.1
for i in {1..5} ; do docker exec -it mypc ping -c 1 172.18.0.$i; done
docker exec -it mypc zsh
-------------
ifconfig
ping -c 1 172.18.0.2
exit
-------------

~~~



### kube-ops-view 설치

~~~sh
helm repo add geek-cookbook https://geek-cookbook.github.io/charts/
helm install kube-ops-view geek-cookbook/kube-ops-view --version 1.2.2 --set service.main.type=NodePort,service.main.ports.http.nodePort=30000 --set env.TZ="Asia/Seoul" --namespace kube-system

# myk8s-control-plane 배치
kubectl -n kube-system edit deploy kube-ops-view
---
spec:
  ...
  template:
    ...
    spec:
      nodeSelector:
        mynode: control-plane
      tolerations:
      - key: "node-role.kubernetes.io/control-plane"
        operator: "Equal"
        effect: "NoSchedule"
---

# 설치 확인
kubectl -n kube-system get pod -o wide -l app.kubernetes.io/instance=kube-ops-view

# kube-ops-view 접속 URL 확인 (1.5 , 2 배율) : macOS 사용자
echo -e "KUBE-OPS-VIEW URL = http://localhost:30000/#scale=1.5"
echo -e "KUBE-OPS-VIEW URL = http://localhost:30000/#scale=2"
~~~

![](/img/service-17.png)







### grafana 설치

~~~sh
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

# 파라미터 파일 생성
cat <<EOT > monitor-values.yaml
prometheus:
  prometheusSpec:
    podMonitorSelectorNilUsesHelmValues: false
    serviceMonitorSelectorNilUsesHelmValues: false
    nodeSelector:
      mynode: control-plane
    tolerations:
    - key: "node-role.kubernetes.io/control-plane"
      operator: "Equal"
      effect: "NoSchedule"


grafana:
  defaultDashboardsTimezone: Asia/Seoul
  adminPassword: 1234

  service:
    type: NodePort
    nodePort: 30002
  nodeSelector:
    mynode: control-plane
  tolerations:
  - key: "node-role.kubernetes.io/control-plane"
    operator: "Equal"
    effect: "NoSchedule"

defaultRules:
  create: false
alertmanager:
  enabled: false

EOT

# 배포
kubectl create ns monitoring
helm install kube-prometheus-stack prometheus-community/kube-prometheus-stack --version 62.3.0 -f monitor-values.yaml --namespace monitoring

# 확인
helm list -n monitoring

# Grafana 접속 계정 : admin / 1234 : macOS 사용자
echo -e "Grafana URL = http://localhost:30002"

# (참고) helm 삭제
helm uninstall -n monitoring kube-prometheus-stack
~~~

![](/img/service-18.png)




# clusterIP



## 통신 흐름

클라이언트(TestPod)가 'CLUSTER-IP' 접속 시 해당 노드의 iptables 룰(랜덤 분산)에 의해서 DNAT 처리가 되어 목적지(backend) 파드와 통신

![](/img/service-01.png)

![](/img/service-02.png)

클러스터 내부에서만 'CLUSTER-IP' 로 접근 가능 ⇒ 서비스에 DNS(도메인) 접속도 가능!

서비스(ClusterIP 타입) 생성하게 되면, apiserver → (kubelet) → kube-proxy → iptables 에 rule(룰)이 생성됨

모든 노드(마스터 포함)에 iptables rule 이 설정되므로, 파드에서 접속 시 해당 노드에 존재하는 iptables rule 에 의해서 분산 접속이 됨





## 실습 구성

![](/img/service-03.png)

목적지(backend) 파드(Pod) 생성

~~~yaml
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

클라이언트(TestPod) 생성 

~~~yaml
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

서비스(ClusterIP) 생성

~~~yaml
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

생성 확인

~~~sh
➜  service git:(master) ✗ kubectl apply -f 3pod.yaml,netpod.yaml,svc-clusterip.yaml
pod/webpod1 created
pod/webpod2 created
pod/webpod3 created
pod/net-pod created
service/svc-clusterip created

kubectl get endpoints svc-clusterip
NAME            ENDPOINTS                                AGE
svc-clusterip   10.10.1.4:80,10.10.2.4:80,10.10.3.3:80   51m
~~~





## 서비스(ClusterIP) 접속 확인

클라이언트(TestPod) Shell 에서 접속 테스트 & 서비스(ClusterIP) 부하분산 접속 확인



~~~sh
# webpod 파드의 IP 를 출력
kubectl get pod -l app=webpod -o jsonpath="{.items[*].status.podIP}"

# webpod 파드의 IP를 변수에 지정
WEBPOD1=$(kubectl get pod webpod1 -o jsonpath={.status.podIP})
WEBPOD2=$(kubectl get pod webpod2 -o jsonpath={.status.podIP})
WEBPOD3=$(kubectl get pod webpod3 -o jsonpath={.status.podIP})
echo $WEBPOD1 $WEBPOD2 $WEBPOD3

# net-pod 파드에서 webpod 파드의 IP로 직접 curl 로 반복 접속
for pod in $WEBPOD1 $WEBPOD2 $WEBPOD3; do kubectl exec -it net-pod -- curl -s $pod; done
for pod in $WEBPOD1 $WEBPOD2 $WEBPOD3; do kubectl exec -it net-pod -- curl -s $pod | grep Hostname; done
for pod in $WEBPOD1 $WEBPOD2 $WEBPOD3; do kubectl exec -it net-pod -- curl -s $pod | grep Host; done
for pod in $WEBPOD1 $WEBPOD2 $WEBPOD3; do kubectl exec -it net-pod -- curl -s $pod | egrep 'Host|RemoteAddr'; done

# 서비스 IP 변수 지정 : svc-clusterip 의 ClusterIP주소
SVC1=$(kubectl get svc svc-clusterip -o jsonpath={.spec.clusterIP})
echo $SVC1

# 위 서비스 생성 시 kube-proxy 에 의해서 iptables 규칙이 모든 노드에 추가됨 
docker exec -it myk8s-control-plane iptables -t nat -S | grep $SVC1
for i in control-plane worker worker2 worker3; do echo ">> node myk8s-$i <<"; docker exec -it myk8s-$i iptables -t nat -S | grep $SVC1; echo; done
-A KUBE-SERVICES -d 10.200.1.52/32 -p tcp -m comment --comment "default/svc-clusterip:svc-webport cluster IP" -m tcp --dport 9000 -j KUBE-SVC-KBDEBIL6IU6WL7RF

## (참고) ss 툴로 tcp listen 정보에는 없음 , 별도 /32 host 라우팅 추가 없음 -> 즉, iptables rule 에 의해서 처리됨을 확인
docker exec -it myk8s-control-plane ss -tnlp
docker exec -it myk8s-control-plane ip -c route

# TCP 80,9000 포트별 접속 확인 : 출력 정보 의미 확인
kubectl exec -it net-pod -- curl -s --connect-timeout 1 $SVC1
kubectl exec -it net-pod -- curl -s --connect-timeout 1 $SVC1:9000
kubectl exec -it net-pod -- curl -s --connect-timeout 1 $SVC1:9000 | grep Hostname
kubectl exec -it net-pod -- curl -s --connect-timeout 1 $SVC1:9000 | grep Hostname

# 서비스(ClusterIP) 부하분산 접속 확인
kubectl exec -it net-pod -- zsh -c "for i in {1..10};   do curl -s $SVC1:9000 | grep Hostname; done | sort | uniq -c | sort -nr"
kubectl exec -it net-pod -- zsh -c "for i in {1..100};  do curl -s $SVC1:9000 | grep Hostname; done | sort | uniq -c | sort -nr"
kubectl exec -it net-pod -- zsh -c "for i in {1..1000}; do curl -s $SVC1:9000 | grep Hostname; done | sort | uniq -c | sort -nr"
혹은
kubectl exec -it net-pod -- zsh -c "for i in {1..100};   do curl -s $SVC1:9000 | grep Hostname; sleep 1; done"
kubectl exec -it net-pod -- zsh -c "for i in {1..100};   do curl -s $SVC1:9000 | grep Hostname; sleep 0.1; done"
kubectl exec -it net-pod -- zsh -c "for i in {1..10000}; do curl -s $SVC1:9000 | grep Hostname; sleep 0.01; done"


# conntrack 확인
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

# (참고) Link layer 에서 동작하는 ebtables
ebtables -L
~~~

반복해서 실행을 해보면, SVC1 IP로 curl 접속 시 3개의 파드로 대략 33% 정도로 부하분산 접속됨을 확인

~~~sh
➜  service git:(master) ✗ kubectl exec -it net-pod -- zsh -c "for i in {1..1000}; do curl -s $SVC1:9000 | grep Hostname; done | sort | uniq -c | sort -nr"
    340 Hostname: webpod1
    330 Hostname: webpod3
    330 Hostname: webpod
~~~



### 각 워커노드에서 패킷 덤프 확인

~~~sh
# 방안1 : 1대 혹은 3대 bash 진입 후 tcpdump 해둘 것
docker exec -it myk8s-worker bash
docker exec -it myk8s-worker2 bash
docker exec -it myk8s-worker3 bash
----------------------------------
# nic 정보 확인
ip -c link
ip -c route
ip -c addr

# tcpdump/ngrep : eth0 >> tcp 9000 포트 트래픽은 왜 없을까? iptables rule 동작 그림을 한번 더 확인하고 이해해보자
## ngrep 네트워크 패킷 분석기 활용해보기 : 특정 url 호출에 대해서만 필터 등 깔끔하게 볼 수 있음 - 링크
tcpdump -i eth0 tcp port 80 -nnq
tcpdump -i eth0 tcp port 80 -w /root/svc1-1.pcap
tcpdump -i eth0 tcp port 9000 -nnq
ngrep -tW byline -d eth0 '' 'tcp port 80'

# tcpdump/ngrep : vethX
VETH1=<각자 자신의 veth 이름>
tcpdump -i $VETH1 tcp port 80 -nn
tcpdump -i $VETH1 tcp port 80 -w /root/svc1-2.pcap
tcpdump -i $VETH1 tcp port 9000 -nn
ngrep -tW byline -d $VETH1 '' 'tcp port 80'

exit
----------------------------------

# 방안2 : 노드(?) 컨테이너 bash 직접 접속하지 않고 호스트에서 tcpdump 하기
docker exec -it myk8s-worker tcpdump -i eth0 tcp port 80 -nnq
VETH1=<각자 자신의 veth 이름> # docker exec -it myk8s-worker ip -c route
docker exec -it myk8s-worker tcpdump -i $VETH1 tcp port 80 -nnq

# 호스트PC에 pcap 파일 복사 >> wireshark 에서 분석
docker cp myk8s-worker:/root/svc1-1.pcap .
docker cp myk8s-worker:/root/svc1-2.pcap .
~~~











## IPTABLES 정책 확인

![](/img/service-04.png)

~~~sh
# 컨트롤플레인에서 확인 : 너무 복잡해서 리턴 트래픽에 대해서는 상세히 분석 정리하지 않습니다.
docker exec -it myk8s-control-plane bash
----------------------------------------

# iptables 확인
iptables -t filter -S
iptables -t nat -S
iptables -t nat -S | wc -l
iptables -t mangle -S

# iptables 상세 확인 - 매칭 패킷 카운트, 인터페이스 정보 등 포함
iptables -nvL -t filter
iptables -nvL -t nat
iptables -nvL -t mangle

# rule 갯수 확인
iptables -nvL -t filter | wc -l
iptables -nvL -t nat | wc -l

# 규칙 패킷 바이트 카운트 초기화
iptables -t filter --zero; iptables -t nat --zero; iptables -t mangle --zero

# 정책 확인 : 아래 정책 내용은 핵심적인 룰(rule)만 표시했습니다!
iptables -t nat -nvL

iptables -v --numeric --table nat --list PREROUTING | column -t
Chain PREROUTING (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination
  778 46758 KUBE-SERVICES  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* kubernetes service portals */

iptables -v --numeric --table nat --list KUBE-SERVICES | column
# 바로 아래 룰(rule)에 의해서 서비스(ClusterIP)를 인지하고 처리를 합니다
Chain KUBE-SERVICES (2 references)
 pkts bytes target                     prot opt in     out     source               destination
   92  5520 KUBE-SVC-KBDEBIL6IU6WL7RF  tcp  --  *      *       0.0.0.0/0            10.105.114.73        /* default/svc-clusterip:svc-webport cluster IP */ tcp dpt:9000

iptables -v --numeric --table nat --list KUBE-SVC-KBDEBIL6IU6WL7RF | column
watch -d 'iptables -v --numeric --table nat --list KUBE-SVC-KBDEBIL6IU6WL7RF'

SVC1=$(kubectl get svc svc-clusterip -o jsonpath={.spec.clusterIP})
kubectl exec -it net-pod -- zsh -c "for i in {1..100};   do curl -s $SVC1:9000 | grep Hostname; sleep 1; done"

# SVC-### 에서 랜덤 확률(대략 33%)로 SEP(Service EndPoint)인 각각 파드 IP로 DNAT 됩니다!
## 첫번째 룰에 일치 확률은 33% 이고, 매칭되지 않을 경우 아래 2개 남을때는 룰 일치 확률은 50%가 됩니다. 이것도 매칭되지 않으면 마지막 룰로 100% 일치됩니다
Chain KUBE-SVC-KBDEBIL6IU6WL7RF (1 references)
 pkts bytes target                     prot opt in     out     source               destination
   38  2280 KUBE-SEP-6TM74ZFOWZXXYQW6  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* default/svc-clusterip:svc-webport */ statistic mode random probability 0.33333333349
   29  1740 KUBE-SEP-354QUAZJTL5AR6RR  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* default/svc-clusterip:svc-webport */ statistic mode random probability 0.50000000000
   25  1500 KUBE-SEP-PY4VJNJPBUZ3ATEL  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* default/svc-clusterip:svc-webport */

iptables -v --numeric --table nat --list KUBE-SEP-<각자 값 입력>
Chain KUBE-SEP-6TM74ZFOWZXXYQW6 (1 references)
 pkts bytes target     prot opt in     out     source               destination
   38  2280 DNAT       tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            /* default/svc-clusterip:svc-webport */ tcp to:172.16.158.3:80

iptables -v --numeric --table nat --list KUBE-SEP-354QUAZJTL5AR6RR | column -t
Chain KUBE-SEP-6TM74ZFOWZXXYQW6 (1 references)
 pkts bytes target     prot opt in     out     source               destination
   29  1500 DNAT       tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            /* default/svc-clusterip:svc-webport */ tcp to:172.16.184.3:80

iptables -v --numeric --table nat --list KUBE-SEP-PY4VJNJPBUZ3ATEL | column -t
Chain KUBE-SEP-6TM74ZFOWZXXYQW6 (1 references)
 pkts bytes target     prot opt in     out     source               destination
   25  1740 DNAT       tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            /* default/svc-clusterip:svc-webport */ tcp to:172.16.34.3:80

iptables -t nat --zero
iptables -v --numeric --table nat --list POSTROUTING | column; echo ; iptables -v --numeric --table nat --list KUBE-POSTROUTING | column
watch -d 'iptables -v --numeric --table nat --list POSTROUTING; echo ; iptables -v --numeric --table nat --list KUBE-POSTROUTING'
# POSTROUTE(nat) : 0x4000(2진수로 0100 0000 0000 0000, 10진수 16384) 마킹 되어 있지 않으니 RETURN 되고 그냥 빠져나가서 SNAT 되지 않는다!
Chain KUBE-POSTROUTING (1 references)
 pkts bytes target     prot opt in     out     source               destination
  572 35232 RETURN     all  --  *      *       0.0.0.0/0            0.0.0.0/0            mark match ! 0x4000/0x4000
    0     0 MARK       all  --  *      *       0.0.0.0/0            0.0.0.0/0            MARK xor 0x4000
    0     0 MASQUERADE  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* kubernetes service traffic requiring SNAT */ random-fully

iptables -t nat -S | grep KUBE-POSTROUTING
-A KUBE-POSTROUTING -m mark ! --mark 0x4000/0x4000 -j RETURN
-A KUBE-POSTROUTING -j MARK --set-xmark 0x4000/0x0
-A KUBE-POSTROUTING -m comment --comment "kubernetes service traffic requiring SNAT" -j MASQUERADE --random-fully
...

exit
----------------------------------------

# 위 서비스 생성 시 kube-proxy 에 의해서 iptables 규칙이 모든 노드에 추가됨을 한번 더 확이
docker exec -it myk8s-control-plane iptables -v --numeric --table nat --list KUBE-SVC-KBDEBIL6IU6WL7RF
...

for i in control-plane worker worker2 worker3; do echo ">> node myk8s-$i <<"; docker exec -it myk8s-$i iptables -v --numeric --table nat --list KUBE-SVC-KBDEBIL6IU6WL7RF; echo; done
...
~~~

![](/img/service-05.png)





## 파드 1개 장애 발생 시 동작 확인

모니터링

~~~sh
# 터미널1 >> ENDPOINTS 변화를 잘 확인해보자!
watch -d 'kubectl get pod -owide;echo; kubectl get svc,ep svc-clusterip;echo; kubectl get endpointslices -l kubernetes.io/service-name=svc-clusterip'

# 터미널2
SVC1=$(kubectl get svc svc-clusterip -o jsonpath={.spec.clusterIP})
kubectl exec -it net-pod -- zsh -c "while true; do curl -s --connect-timeout 1 $SVC1:9000 | egrep 'Hostname|IP: 10'; date '+%Y-%m-%d %H:%M:%S' ; echo ;  sleep 1; done"
혹은
kubectl exec -it net-pod -- zsh -c "for i in {1..100};  do curl -s $SVC1:9000 | grep Hostname; done | sort | uniq -c | sort -nr"
~~~

파드 하나 삭제 후 확인

~~~sh
# (방안1) 파드3번 삭제 >> 서비스의 엔드포인트가 어떻게 변경되는지 확인 하자!, 지속적인 curl 접속 결과 확인!, for 문 실행 시 결과 확인!, 절체 시간(순단) 확인!
kubectl delete pod webpod3

# (방안1) 결과 확인 후 다시 파드 3번 생성 >> 서비스 디스커버리!
kubectl apply -f 3pod.yaml

---------------------------------
# (방안2) 파드3번에 레이블 삭제
kubectl get pod --show-labels

## 레이블(라벨)의 키값 바로 뒤에 하이픈(-) 입력 시 해당 레이블 삭제됨! >> 레이블과 셀렉터는 쿠버네티스 환경에서 매우 많이 활용된다!
kubectl label pod webpod3 app-
kubectl get pod --show-labels

# (방안2) 결과 확인 후 파드3번에 다시 레이블 생성
kubectl label pod webpod3 app=webpod
~~~

k8s 클러스터 외부(mypc)에서는 서비스(ClusterIP)로 접속이 불가능

~~~sh
docker ps
docker exec -it mypc ping -c 1 172.18.0.2
docker exec -it mypc curl <SVC1_IP>:9000
docker exec -it mypc curl <Pod IP>:80
~~~





## sessionAffinity: ClientIP

sessionAffinity: ClientIP : 클라이언트가 접속한 목적지(파드)에 고정적인 접속을 지원 - [k8s_Docs](https://kubernetes.io/docs/reference/networking/virtual-ips/#session-affinity)

만약 클라이언트의 요청을 매번 동일한 목적지 파드로 전달하기 위해서 세션어피니티가 필요하다.

클라이언트가 서비스를 통해 최초 전달된 파드에 대한 연결상태를 기록해두고, 이후 동일한 클라이언트가 서비스에 접속 시 연결상태 정보를 확인하여 최초 전달된 파드, 즉 동일한 혹은 마치 고정적인 파드로 전달할 수 있다,.

![](/img/service-06.png)



~~~sh
# 기본 정보 확인
kubectl get svc svc-clusterip -o yaml
kubectl get svc svc-clusterip -o yaml | grep sessionAffinity

# 반복 접속
kubectl exec -it net-pod -- zsh -c "while true; do curl -s --connect-timeout 1 $SVC1:9000 | egrep 'Hostname|IP: 10|Remote'; date '+%Y-%m-%d %H:%M:%S' ; echo ;  sleep 1; done"

# sessionAffinity: ClientIP 설정 변경
kubectl patch svc svc-clusterip -p '{"spec":{"sessionAffinity":"ClientIP"}}'
혹은
kubectl get svc svc-clusterip -o yaml | sed -e "s/sessionAffinity: None/sessionAffinity: ClientIP/" | kubectl apply -f -

#
kubectl get svc svc-clusterip -o yaml
...
  sessionAffinity: ClientIP
  sessionAffinityConfig:
    clientIP:
      timeoutSeconds: 10800
...

# 클라이언트(TestPod) Shell 실행
kubectl exec -it net-pod -- zsh -c "for i in {1..100};  do curl -s $SVC1:9000 | grep Hostname; done | sort | uniq -c | sort -nr"
kubectl exec -it net-pod -- zsh -c "for i in {1..1000}; do curl -s $SVC1:9000 | grep Hostname; done | sort | uniq -c | sort -nr"
~~~

![](/img/service-07.png)



## 서비스(ClusterIP) 부족한 점

- 클러스터 외부에서는 서비스(ClusterIP)로 접속이 불가능 ⇒ `NodePort` 타입으로 외부에서 접속 가능!

- IPtables 는 파드에 대한 헬스체크 기능이 없어서 문제 있는 파드에 연결 가능 ⇒ 서비스 사용, 파드에 `Readiness Probe` 설정으로 파드 문제 시 서비스의 엔드포인트에서 제거되게 하자! ← 이 정도면 충분한가? 혹시 부족한 점이 없을까?

- 서비스에 연동된 파드 갯수 퍼센트(%)로 랜덤 분산 방식, 세션어피니티 이외에 다른 분산 방식 불가능 ⇒ `IPVS` 경우 다양한 분산 방식(알고리즘) 가능
  - 목적지 파드 다수가 있는 환경에서, 출발지 파드와 목적지 파드가 동일한 노드에 배치되어 있어도, 랜덤 분산으로 다른 노드에 목적지 파드로 연결 가능






# nodeport












## 통신 흐름

외부 클라이언트가 `노드IP:NodePort` 접속 시 해당 노드의 iptables 룰에 의해서 SNAT/DNAT 되어 목적지 파드와 통신 후 리턴 트래픽은 최초 인입 노드를 경유해서 외부로 되돌아감



ClusterIp 서비스 생성과 동일하게 NodePort  서비스도 생성 시 kube-proxy에 의해서 모든 노드에 iptables규칙이 설정됨.

노드에 진입한 트래픽은 iptables에 의해 서비스에 연결된 pod들에 부하분산되어 접속됨.

![](/img/service-08.png)

![](/img/service-09.png)![스크린샷 2024-09-29 오전 7.08.42](/Users/staek/Library/Application Support/typora-user-images/스크린샷 2024-09-29 오전 7.08.42.png)

- 외부에서 클러스터의 '서비스(NodePort)' 로 접근 가능 → 이후에는 Cluster IP 통신과 동일!
- 모드 노드(마스터 포함)에 iptables rule 이 설정되므로, 모든 노드에 NodePort 로 접속 시 iptables rule 에 의해서 분산 접속이 됨
- Node 의 모든 Loca IP(Local host Interface IP : loopback 포함) 사용 가능 & Local IP를 지정 가능
- 쿠버네티스 NodePort 할당 범위 기본 (30000-32767) 









## 실습 구성

목적지(backend) 디플로이먼트(Pod) 파일 생성

~~~yaml
cat <<EOT> echo-deploy.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy-echo
spec:
  replicas: 3
  selector:
    matchLabels:
      app: deploy-websrv
  template:
    metadata:
      labels:
        app: deploy-websrv
    spec:
      terminationGracePeriodSeconds: 0
      containers:
      - name: kans-websrv
        image: mendhak/http-https-echo
        ports:
        - containerPort: 8080
EOT
~~~

서비스(NodePort) 파일 생성

~~~Yaml
cat <<EOT> svc-nodeport.yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-nodeport
spec:
  ports:
    - name: svc-webport
      port: 9000        # 서비스 ClusterIP 에 접속 시 사용하는 포트 port 를 의미
      targetPort: 8080  # 타킷 targetPort 는 서비스를 통해서 목적지 파드로 접속 시 해당 파드로 접속하는 포트를 의미
  selector:
    app: deploy-websrv
  type: NodePort
EOT
~~~



~~~yaml
# 생성
kubectl apply -f echo-deploy.yaml,svc-nodeport.yaml

# 모니터링
watch -d 'kubectl get pod -owide;echo; kubectl get svc,ep svc-nodeport'

# 확인
kubectl get deploy,pod -o wide

# 아래 31493은 서비스(NodePort) 정보!
kubectl get svc svc-nodeport
NAME           TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
svc-nodeport   NodePort   10.200.1.167   <none>        9000:30396/TCP   9s

kubectl get endpoints svc-nodeport
NAME           ENDPOINTS                                      AGE
svc-nodeport   10.10.1.5:8080,10.10.2.6:8080,10.10.3.4:8080   17s

# Port , TargetPort , NodePort 주의
kubectl describe svc svc-nodeport
Name:                     svc-nodeport
Namespace:                default
Labels:                   <none>
Annotations:              <none>
Selector:                 app=deploy-websrv
Type:                     NodePort
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.200.1.167
IPs:                      10.200.1.167
Port:                     svc-webport  9000/TCP
TargetPort:               8080/TCP
NodePort:                 svc-webport  30396/TCP
Endpoints:                10.10.2.6:8080,10.10.3.4:8080,10.10.1.5:8080
Session Affinity:         None
External Traffic Policy:  Cluster
Internal Traffic Policy:  Cluster
Events:                   <none>
~~~





## 서비스(NodePort) 접속 확인

외부 클라이언트(mypc 컨테이너)에서 접속 테스트 & 서비스(NodePort) 부하분산 접속 확인



~~~yaml
# NodePort 확인 : 아래 NodePort 는 범위내 랜덤 할당으로 실습 환경마다 다릅니다
kubectl get service svc-nodeport -o jsonpath='{.spec.ports[0].nodePort}'
30396

# NodePort 를 변수에 지정
NPORT=$(kubectl get service svc-nodeport -o jsonpath='{.spec.ports[0].nodePort}')
echo $NPORT

# 현재 k8s 버전에서는 포트 Listen 되지 않고, iptables rules 처리됨
for i in control-plane worker worker2 worker3; do echo ">> node myk8s-$i <<"; docker exec -it myk8s-$i ss -tlnp; echo; done

## (참고) 아래처럼 예전 k8s 환경에서 Service(NodePort) 생성 시, TCP Port Listen 되었었음
root@myk8s-control-plane:/# ss -4tlnp | egrep "(Process|$NPORT)"
State  Recv-Q Send-Q Local Address:Port  Peer Address:PortProcess
LISTEN 0      1024      127.0.0.11:35413      0.0.0.0:*
LISTEN 0      4096       127.0.0.1:34021      0.0.0.0:*    users:(("containerd",pid=104,fd=10))
LISTEN 0      4096       127.0.0.1:10248      0.0.0.0:*    users:(("kubelet",pid=692,fd=12))
LISTEN 0      4096      172.18.0.4:2379       0.0.0.0:*    users:(("etcd",pid=638,fd=9))
LISTEN 0      4096       127.0.0.1:2379       0.0.0.0:*    users:(("etcd",pid=638,fd=8))
LISTEN 0      4096      172.18.0.4:2380       0.0.0.0:*    users:(("etcd",pid=638,fd=7))

# 파드 로그 실시간 확인 (웹 파드에 접속자의 IP가 출력)
kubectl logs -l app=deploy-websrv -f


# 외부 클라이언트(mypc 컨테이너)에서 접속 시도를 해보자

# 노드의 IP와 NodePort를 변수에 지정
## CNODE=<컨트롤플레인노드의 IP주소>
## NODE1=<노드1의 IP주소>
## NODE2=<노드2의 IP주소>
## NODE3=<노드3의 IP주소>
CNODE=172.18.0.4
NODE1=172.18.0.2
NODE2=172.18.0.3
NODE3=172.18.0.5

NPORT=$(kubectl get service svc-nodeport -o jsonpath='{.spec.ports[0].nodePort}')
echo $NPORT

# 서비스(NodePort) 부하분산 접속 확인
docker exec -it mypc curl -s $CNODE:$NPORT | jq # headers.host 주소는 왜 그런거죠?
for i in $CNODE $NODE1 $NODE2 $NODE3 ; do echo ">> node $i <<"; docker exec -it mypc curl -s $i:$NPORT; echo; done

# 컨트롤플레인 노드에는 목적지 파드가 없는데도, 접속을 받아준다! 이유는?
docker exec -it mypc zsh -c "for i in {1..100}; do curl -s $CNODE:$NPORT | grep hostname; done | sort | uniq -c | sort -nr"
docker exec -it mypc zsh -c "for i in {1..100}; do curl -s $NODE1:$NPORT | grep hostname; done | sort | uniq -c | sort -nr"
docker exec -it mypc zsh -c "for i in {1..100}; do curl -s $NODE2:$NPORT | grep hostname; done | sort | uniq -c | sort -nr"
docker exec -it mypc zsh -c "for i in {1..100}; do curl -s $NODE3:$NPORT | grep hostname; done | sort | uniq -c | sort -nr"

# 아래 반복 접속 실행 해두자
docker exec -it mypc zsh -c "while true; do curl -s --connect-timeout 1 $CNODE:$NPORT | grep hostname; date '+%Y-%m-%d %H:%M:%S' ; echo ;  sleep 1; done"


# NodePort 서비스는 ClusterIP 를 포함
# CLUSTER-IP:PORT 로 접속 가능! <- 컨트롤노드에서 아래 실행 해보자
kubectl get svc svc-nodeport
NAME           TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
svc-nodeport   NodePort   10.111.1.238   <none>         9000:30158/TCP   3m3s

CIP=$(kubectl get service svc-nodeport -o jsonpath="{.spec.clusterIP}")
CIPPORT=$(kubectl get service svc-nodeport -o jsonpath="{.spec.ports[0].port}")
echo $CIP $CIPPORT
docker exec -it myk8s-control-plane curl -s $CIP:$CIPPORT | jq

# mypc에서 CLUSTER-IP:PORT 로 접속 가능할까?
docker exec -it mypc curl -s $CIP:$CIPPORT


# (옵션) 노드에서 Network Connection
conntrack -E
conntrack -L --any-nat

# (옵션) 패킷 캡쳐 확인
tcpdump..
~~~

![](/img/service-10.png)







## IPTABLES 정책 확인

![](/img/service-11.png)

컨트롤플레인 노드 - iptables 분석 (핵심 정책 확인)

~~~sh
docker exec -it myk8s-control-plane bash
----------------------------------------

# 패킷 카운트 초기화
iptables -t nat --zero


PREROUTING 정보 확인
iptables -t nat -S | grep PREROUTING
-A PREROUTING -m comment --comment "kubernetes service portals" -j KUBE-SERVICES
...


# 외부 클라이언트가 노드IP:NodePort 로 접속하기 때문에 --dst-type LOCAL 에 매칭되어서 -j KUBE-NODEPORTS 로 점프!
iptables -t nat -S | grep KUBE-SERVICES
-A KUBE-SERVICES -m comment --comment "kubernetes service nodeports; NOTE: this must be the last rule in this chain" -m addrtype --dst-type LOCAL -j KUBE-NODEPORTS
...


# KUBE-NODEPORTS 에서 KUBE-EXT-# 로 점프!
## -m nfacct --nfacct-name localhost_nps_accepted_pkts 추가됨 : 패킷 flow 카운팅 - 카운트 이름 지정 
NPORT=$(kubectl get service svc-nodeport -o jsonpath='{.spec.ports[0].nodePort}')
echo $NPORT

iptables -t nat -S | grep KUBE-NODEPORTS | grep <NodePort>
iptables -t nat -S | grep KUBE-NODEPORTS | grep $NPORT
-A KUBE-NODEPORTS -d 127.0.0.0/8 -p tcp -m comment --comment "default/svc-nodeport:svc-webport" -m tcp --dport 30898 -m nfacct --nfacct-name localhost_nps_accepted_pkts -j KUBE-EXT-VTR7MTHHNMFZ3OFS
-A KUBE-NODEPORTS -p tcp -m comment --comment "default/svc-nodeport:svc-webport" -m tcp --dport 30898 -j KUBE-EXT-VTR7MTHHNMFZ3OFS

# (참고) nfacct 확인
nfacct list
## nfacct flush # 초기화


## KUBE-EXT-# 에서 'KUBE-MARK-MASQ -j MARK --set-xmark 0x4000/0x4000' 마킹 및 KUBE-SVC-# 로 점프!
# docker exec -it mypc zsh -c "while true; do curl -s --connect-timeout 1 $CNODE:$NPORT | grep hostname; date '+%Y-%m-%d %H:%M:%S' ; echo ;  sleep 1; done" 반복 접속 후 아래 확인
watch -d 'iptables -v --numeric --table nat --list KUBE-EXT-VTR7MTHHNMFZ3OFS'
iptables -t nat -S | grep "A KUBE-EXT-VTR7MTHHNMFZ3OFS"
-A KUBE-EXT-VTR7MTHHNMFZ3OFS -m comment --comment "masquerade traffic for default/svc-nodeport:svc-webport external destinations" -j KUBE-MARK-MASQ
-A KUBE-EXT-VTR7MTHHNMFZ3OFS -j KUBE-SVC-VTR7MTHHNMFZ3OFS


# KUBE-SVC-# 이후 과정은 Cluster-IP 와 동일! : 3개의 파드로 DNAT 되어서 전달
iptables -t nat -S | grep "A KUBE-SVC-VTR7MTHHNMFZ3OFS -"
-A KUBE-SVC-VTR7MTHHNMFZ3OFS -m comment --comment "default/svc-nodeport:svc-webport" -m statistic --mode random --probability 0.33333333349 -j KUBE-SEP-Q5ZOWRTVDPKGFLOL
-A KUBE-SVC-VTR7MTHHNMFZ3OFS -m comment --comment "default/svc-nodeport:svc-webport" -m statistic --mode random --probability 0.50000000000 -j KUBE-SEP-MMWCMKTGOFHFMRIZ
-A KUBE-SVC-VTR7MTHHNMFZ3OFS -m comment --comment "default/svc-nodeport:svc-webport" -j KUBE-SEP-CQTAHW4MAKGGR6M2


POSTROUTING 정보 확인
# 마킹되어 있어서 출발지IP를 접속한 노드의 IP 로 SNAT(MASQUERADE) 처리함! , 최초 출발지Port는 랜덤Port 로 변경
iptables -t nat -S | grep "A KUBE-POSTROUTING"
-A KUBE-POSTROUTING -m mark ! --mark 0x4000/0x4000 -j RETURN  # 0x4000/0x4000 되어 있으니 여기에 매칭되지 않고 아래 Rule로 내려감
-A KUBE-POSTROUTING -j MARK --set-xmark 0x4000/0x0
-A KUBE-POSTROUTING -m comment --comment "kubernetes service traffic requiring SNAT" -j MASQUERADE --random-fully


# docker exec -it mypc zsh -c "while true; do curl -s --connect-timeout 1 $CNODE:$NPORT | grep hostname; date '+%Y-%m-%d %H:%M:%S' ; echo ;  sleep 1; done" 반복 접속 후 아래 확인
watch -d 'iptables -v --numeric --table nat --list KUBE-POSTROUTING;echo;iptables -v --numeric --table nat --list POSTROUTING'

exit
----------------------------------------
~~~

서비스(NodePort) 생성 시 kube-proxy 에 의해서 iptables 규칙이 모든 노드에 추가되는지 확인

~~~sh
#
NPORT=$(kubectl get service svc-nodeport -o jsonpath='{.spec.ports[0].nodePort}')
docker exec -it myk8s-control-plane iptables -t nat -S | grep KUBE-NODEPORTS | grep $NPORT
...

for i in control-plane worker worker2 worker3; do echo ">> node myk8s-$i <<"; docker exec -it myk8s-$i iptables -t nat -S | grep KUBE-NODEPORTS | grep $NPORT; echo; done
...
~~~



## externalTrafficPolicy 설정

externalTrafficPolicy: Local : NodePort 로 접속 시 해당 노드에 배치된 파드로만 접속됨, 이때 SNAT 되지 않아서 외부 클라이언트 IP가 보존됨!

![](/img/service-12.png)

클라이언트 가상머신에서 노드1의 ip에 NodePort로 접속할 때, 노드1에 배포된 파드1로 접속된다.

이후 반복접속 시도하면 노드1의 파드1로만 접속된다.

externalTrafficPolicy 설정은 다른 노드에 배포된 파드로는 접속되지 않는다.

![](/img/service-13.png)





외부 클라이언트의 IP 주소(아래 출발지IP: 50.1.1.1)가 노드의 IP로 SNAT 되지 않고 서비스(backend) 파드까지 전달됨!

![](/img/service-14.png)



externalTrafficPolicy:Local 설정 시 통신흐름

![](/img/service-15.png)

[1] client 에서 파드가 배포되어있는 노드1에 NodePort로 접속시도

[2] 노드에 iptables NAT 테이블 규칙과 매핑되어 목적지 IP와 목적지 Ports는 변환되지만 출발지 IP와 Port는 변환되지 않고 목적지 파드에 전달된다. (파드 입장에서 클라이언트 ip가 그대로 존재)



~~~sh
# 기본 정보 확인
kubectl get svc svc-nodeport -o json | grep 'TrafficPolicy"'
  externalTrafficPolicy: Cluster
  internalTrafficPolicy: Cluster

# 기존 통신 연결 정보(conntrack) 제거 후 아래 실습 진행하자! : (모든 노드에서) conntrack -F
for i in control-plane worker worker2 worker3; do echo ">> node myk8s-$i <<"; docker exec -it myk8s-$i conntrack -F; echo; done
kubectl delete -f svc-nodeport.yaml
kubectl apply -f svc-nodeport.yaml

# externalTrafficPolicy: local 설정 변경
kubectl patch svc svc-nodeport -p '{"spec":{"externalTrafficPolicy": "Local"}}'
kubectl get svc svc-nodeport -o json | grep 'TrafficPolicy"'
	"externalTrafficPolicy": "Local",
  "internalTrafficPolicy": "Cluster",

# 파드 3개를 2개로 줄임
kubectl scale deployment deploy-echo --replicas=2

# 파드 존재하는 노드 정보 확인
kubectl get pod -owide

# 파드 로그 실시간 확인 (웹 파드에 접속자의 IP가 출력)
kubectl logs -l app=deploy-websrv -f


# 외부 클라이언트(mypc)에서 접속 시도

# 노드의 IP와 NodePort를 변수에 지정
## CNODE=<컨트롤플레인노드의 IP주소>
## NODE1=<노드1의 IP주소>
## NODE2=<노드2의 IP주소>
## NODE3=<노드3의 IP주소>
CNODE=172.18.0.4
NODE1=172.18.0.2
NODE2=172.18.0.3
NODE3=172.18.0.5

## NodePort 를 변수에 지정 : 서비스를 삭제 후 다시 생성하여서 NodePort가 변경되었음
NPORT=$(kubectl get service svc-nodeport -o jsonpath='{.spec.ports[0].nodePort}')
echo $NPORT

# 서비스(NodePort) 부하분산 접속 확인 : 파드가 존재하지 않는 노드로는 접속 실패!, 파드가 존재하는 노드는 접속 성공 및 클라이언트 IP 확인!
docker exec -it mypc curl -s --connect-timeout 1 $CNODE:$NPORT | jq
for i in $CNODE $NODE1 $NODE2 $NODE3 ; do echo ">> node $i <<"; docker exec -it mypc curl -s --connect-timeout 1 $i:$NPORT; echo; done

# 목적지 파드가 배치되지 않은 노드는 접속이 어떻게? 왜 그런가?
docker exec -it mypc zsh -c "for i in {1..100}; do curl -s $CNODE:$NPORT | grep hostname; done | sort | uniq -c | sort -nr"
docker exec -it mypc zsh -c "for i in {1..100}; do curl -s $NODE1:$NPORT | grep hostname; done | sort | uniq -c | sort -nr"
docker exec -it mypc zsh -c "for i in {1..100}; do curl -s $NODE2:$NPORT | grep hostname; done | sort | uniq -c | sort -nr"
docker exec -it mypc zsh -c "for i in {1..100}; do curl -s $NODE3:$NPORT | grep hostname; done | sort | uniq -c | sort -nr"

# 파드가 배치된 노드에 NodePort로 아래 반복 접속 실행 해두자
docker exec -it mypc zsh -c "while true; do curl -s --connect-timeout 1 $NODE1:$NPORT | grep hostname; date '+%Y-%m-%d %H:%M:%S' ; echo ;  sleep 1; done"
혹은
docker exec -it mypc zsh -c "while true; do curl -s --connect-timeout 1 $NODE2:$NPORT | grep hostname; date '+%Y-%m-%d %H:%M:%S' ; echo ;  sleep 1; done"
혹은
docker exec -it mypc zsh -c "while true; do curl -s --connect-timeout 1 $NODE3:$NPORT | grep hostname; date '+%Y-%m-%d %H:%M:%S' ; echo ;  sleep 1; done"

# (옵션) 노드에서 Network Connection
conntrack -E
conntrack -L --any-nat
# 패킷 캡쳐 확인
~~~

![](/img/service-16.png)













## 서비스(NodePort) 부족한 점

외부에서 노드의 IP와 포트로 직접 접속이 필요함 → 내부망이 외부에 공개(라우팅 가능)되어 보안에 취약함  ⇒ LoadBalancer 서비스 타입으로 외부 공개 최소화 가능!

클라이언트 IP 보존을 위해서, externalTrafficPolicy: local 사용 시 파드가 없는 노드 IP로 NodePort 접속 시 실패 ⇒ LoadBalancer 서비스에서 헬스체크(Probe) 로 대응 가능!



## 파드 간 속도 측정

iperf3 : 서버 모드로 동작하는 단말과 클라이언트 모드로 동작하는 단말로 구성해서 최대 네트워크 대역폭 측정 - TCP, UDP, SCTP 지원



macos

~~~sh
# iperf3 설치 
brew install iperf3

# iperf3 테스트 1 : TCP 5201, 측정시간 10초
iperf3 -s # 서버모드 실행
iperf3 -c 127.0.0.1 # 클라이언트모드 실행

# iperf3 테스트 2 : TCP 80, 측정시간 5초
iperf3 -s -p 80
iperf3 -c 127.0.0.1 -p 80 -t 5

-----------------------------------------------------------
Server listening on 80 (test #1)
-----------------------------------------------------------
Accepted connection from 127.0.0.1, port 58402
[  5] local 127.0.0.1 port 80 connected to 127.0.0.1 port 58403
[ ID] Interval           Transfer     Bitrate
[  5]   0.00-1.00   sec  12.8 GBytes   110 Gbits/sec
[  5]   1.00-2.00   sec  12.8 GBytes   110 Gbits/sec
[  5]   2.00-3.00   sec  13.3 GBytes   114 Gbits/sec
[  5]   3.00-4.01   sec  14.6 GBytes   125 Gbits/sec
[  5]   4.01-5.00   sec  13.5 GBytes   116 Gbits/sec
[  5]   5.00-5.00   sec  20.9 MBytes   128 Gbits/sec

# iperf3 테스트 3 : UDP 사용, 역방향 모드(-R)
iperf3 -s 
iperf3 -c 127.0.0.1 -u -b 100G

# iperf3 테스트 4 : 역방향 모드(-R)
iperf3 -s 
iperf3 -c 127.0.0.1 -R

# iperf3 테스트 5 : 쌍방향 모드(-R)
iperf3 -s 
iperf3 -c 127.0.0.1 --bidir

# iperf3 테스트 6 : TCP 다중 스트림(30개), -P(number of parallel client streams to run)
iperf3 -s 
iperf3 -c 127.0.0.1 -P 2 -t 30
~~~



k8s 속도 측정

~~~sh
# 배포
kubectl apply -f https://raw.githubusercontent.com/gasida/PKOS/main/aews/k8s-iperf3.yaml

# 확인 : 서버와 클라이언트가 다른 워커노드에 배포되었는지 확인
kubectl get deploy,svc,pod -owide

# 서버 파드 로그 확인 : 기본 5201 포트 Listen
kubectl logs -l app=iperf3-server -f
~~~





1. TCP 5201, 측정시간 5초

   ```bash
   # 클라이언트 파드에서 아래 명령 실행
   kubectl exec -it deploy/**iperf3-client** -- **iperf3 -c iperf3-server -t 5**
   
   # 서버 파드 로그 확인 : 기본 5201 포트 Listen
   kubectl logs -l **app=iperf3-server** -f
   
   Server listening on 5201 (test #1)
   -----------------------------------------------------------
   Accepted connection from 10.10.2.7, port 55190
   [  5] local 10.10.1.6 port 5201 connected to 10.10.2.7 port 55192
   [ ID] Interval           Transfer     Bitrate
   [  5]   0.00-1.00   sec  4.70 GBytes  40.4 Gbits/sec
   [  5]   1.00-2.00   sec  5.11 GBytes  43.9 Gbits/sec
   [  5]   2.00-3.00   sec  4.94 GBytes  42.4 Gbits/sec
   [  5]   3.00-4.00   sec  5.07 GBytes  43.6 Gbits/sec
   [  5]   4.00-5.00   sec  4.93 GBytes  42.4 Gbits/sec
   [  5]   5.00-5.00   sec  1.75 MBytes  40.1 Gbits/sec
   - - - - - - - - - - - - - - - - - - - - - - - - -
   [ ID] Interval           Transfer     Bitrate
   [  5]   0.00-5.00   sec  24.8 GBytes  42.5 Gbits/sec                  receiver
   ```

2. UDP 사용, 역방향 모드(-R)

   ```bash
   # 클라이언트 파드에서 아래 명령 실행
   kubectl exec -it deploy/**iperf3-client** -- **iperf3 -c iperf3-server -u -b 20G**
   
   # 서버 파드 로그 확인 : 기본 5201 포트 Listen
   kubectl logs -l **app=iperf3-server** -f
   ```

3. TCP, 쌍방향 모드(-R)

   ```bash
   # 클라이언트 파드에서 아래 명령 실행
   kubectl exec -it deploy/**iperf3-client** -- **iperf3 -c iperf3-server -t 5 --bidir**
   
   # 서버 파드 로그 확인 : 기본 5201 포트 Listen
   kubectl logs -l **app=iperf3-server** -f
   ```

4. TCP 다중 스트림(30개), -P(number of parallel client streams to run)

   ```bash
   # 클라이언트 파드에서 아래 명령 실행
   kubectl exec -it deploy/**iperf3-client** -- **iperf3 -c iperf3-server -t 10 -P 2**
   
   # 서버 파드 로그 확인 : 기본 5201 포트 Listen
   kubectl logs -l **app=iperf3-server** -f
   ```

https://seongtaekkim.github.io/blog/categories/serivce-clusterip/

https://seongtaekkim.github.io/blog/categories/serivce-nodeport/




























