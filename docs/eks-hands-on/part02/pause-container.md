---
layout: default
title: PAUSE 컨테이너
parent: eks network
grand_parent: eks 실습
nav_order: 3
---

- 파드는 1개 이상의 컨테이너로 구성된 컨테이너의 집합이며, PAUSE 컨테이너가 Network/IPC/UTS 네임스페이스를 생성하고 유지/공유한다.





~~~sh
# '컨트롤플레인, 워커 노드 1대' 클러스터 배포 : 파드에 접속하기 위한 포트 맵핑 설정
cat <<EOT> kind-2node.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
  extraPortMappings:
  - containerPort: 30000
    hostPort: 30000
  - containerPort: 30001
    hostPort: 30001
EOT
kind create cluster --config kind-2node.yaml --name myk8s

# 툴 설치
docker exec -it myk8s-control-plane sh -c 'apt update && apt install tree jq psmisc lsof wget bridge-utils tcpdump htop git nano -y'
docker exec -it myk8s-worker        sh -c 'apt update && apt install tree jq psmisc lsof wget bridge-utils tcpdump htop -y'

# 확인
kubectl get nodes -o wide
docker ps
docker port myk8s-worker
docker exec -it myk8s-control-plane ip -br -c -4 addr
docker exec -it myk8s-worker  ip -br -c -4 addr

# kube-ops-view
helm repo add geek-cookbook https://geek-cookbook.github.io/charts/
helm install kube-ops-view geek-cookbook/kube-ops-view --version 1.2.2 --set service.main.type=NodePort,service.main.ports.http.nodePort=30000 --set env.TZ="Asia/Seoul" --namespace kube-system

# 설치 확인
kubectl get deploy,pod,svc,ep -n kube-system -l app.kubernetes.io/instance=kube-ops-view"
~~~





### 격리자원 확인

~~~sh
# [터미널1] myk8s-worker bash 진입 후 실행 및 확인
docker exec -it myk8s-worker bash
----------------------------------
systemctl list-unit-files | grep 'enabled         enabled'
containerd.service                                                                    enabled         enabled
kubelet.service                                                                       enabled         enabled
...


# 확인 : 파드내에 pause 컨테이너와 metrics-server 컨테이너, 네임스페이스 정보
# pgrep metrics-server
pstree -aclnpsS
...
	\-containerd-shim,1776 -namespace k8s.io -id 6bd147995b3a6c17384459eb4d3ceab4369329e6b57c009bdc6257b72254e1fb -address /run/containerd/containerd.sock
	|-{containerd-shim},1777
  ...
	|-pause,1797,ipc,mnt,net,pid,uts
	|-metrics-server,1896,cgroup,ipc,mnt,net,pid,uts --cert-dir=/tmp --secure-port=10250 --kubelet-insecure-tls --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname --kubelet-use-node-status-port --metric-resolution=15s
	|   |-{metrics-server},1912
  ...


# 특정 소켓 파일을 사용하는 프로세스 확인
lsof /run/containerd/containerd.sock

ss -xl | egrep 'Netid|containerd'

findmnt -A
TARGET                                                  SOURCE                 FSTYPE    OPTIONS
/                                                       overlay                overlay   rw,relatime,lowerdir=/var/lib/docker/overlay2/l/HW4BGGJ4LV6M5
...
|-/sys                                                  sysfs                  sysfs     ro,nosuid,nodev,noexec,relatime
| |-/sys/kernel/debug                                   debugfs                debugfs   rw,nosuid,nodev,noexec,relatime
| |-/sys/kernel/tracing                                 tracefs                tracefs   rw,nosuid,nodev,noexec,relatime
| |-/sys/fs/fuse/connections                            fusectl                fusectl   rw,nosuid,nodev,noexec,relatime
| |-/sys/kernel/config                                  configfs               configfs  rw,nosuid,nodev,noexec,relatime
| \-/sys/fs/cgroup                                      cgroup                 cgroup2   rw,nosuid,nodev,noexec,relatime

findmnt -t cgroup2
grep cgroup /proc/filesystems
stat -fc %T /sys/fs/cgroup/

----------------------------------
~~~



~~~sh
pstree -aclnpsS
~~~

![](/img/pause-01.png)





### containers test

~~~yaml
apiVersion: v1
kind: Pod
metadata:
  name: myweb2
spec:
  containers:
  - name: myweb2-nginx
    image: nginx
    ports:
    - containerPort: 80
      protocol: TCP

  - name: myweb2-netshoot
    image: nicolaka/netshoot
    command: ["/bin/bash"]
    args: ["-c", "while true; do sleep 5; curl localhost; done"] # 포드가 종료되지 않도록 유지합니다

  terminationGracePeriodSeconds: 0
~~~

~~~sh
 kubectl apply -f myweb2.yaml
~~~



~~~sh
kubectl exec myweb2 -c myweb2-netshoot -- ip addr
kubectl exec myweb2 -c myweb2-nginx -- apt update
kubectl exec myweb2 -c myweb2-nginx -- apt install -y net-tools
kubectl exec myweb2 -c myweb2-nginx -- ifconfig

NGINXPID=$(ps -ef | grep 'nginx -g' | grep -v grep | awk '{print $2}')
echo $NGINXPID

NETSHPID=$(ps -ef | grep 'curl' | grep -v grep | awk '{print $2}')
echo $NETSHPID
~~~

![](/img/pause-02.png)

#### ns, ip 비교

~~~sh
# 개별 컨테이너에 명령 실행 : IP 동일 확인
crictl ps
crictl ps -q
crictl exec -its <myweb2-nginx    컨테이너ID> ifconfig
crictl exec -its <myweb2-netshoot 컨테이너ID> ifconfig


# PAUSE 의 NET 네임스페이스 PID 확인 및 IP 정보 확인
lsns -t net
nsenter -t $PAUSEPID -n ip -c addr
nsenter -t $NGINXPID -n ip -c addr
nsenter -t $NETSHPID -n ip -c addr

# 2개의 네임스페이스 비교 , 아래 2112 프로세스의 정제는?
crictl inspect <myweb2-nginx    컨테이너ID> | jq
crictl inspect <myweb2-netshoot 컨테이너ID> | jq
~~~





### 자원정리

~~~sh
kubectl delete pod myweb2
kind delete cluster --name myk8s
~~~









