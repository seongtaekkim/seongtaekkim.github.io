---
layout: default
title: eks ingress
parent: eks network
grand_parent: eks 실습
nav_order: 3
---


# Ingress 요약

Nginx 인그레스 컨트롤러  : 외부에서 인그레스로 접속 시 Nginx 인그레스 컨트롤러 파드로 인입되고, 이후 애플리케이션 파드의 IP로 직접 통신

![](/img/ingress-01.png)



## 클러스터 내부를 외부에 노출 - 발전 단계



1. 파드 생성 : K8S 클러스터 내부에서만 접속

![](/img/ingress-02.png)





2. 서비스(Cluster Type) 연결 : K8S 클러스터 내부에서만 접속 

- 동일한 애플리케이션의 다수의 파드의 접속을 용이하게 하기 위한 서비스에 접속

![](/img/ingress-03.png)



3. 서비스(NodePort Type) 연결 : 외부 클라이언트가 서비스를 통해서 클러스터 내부의 파드로 접속

- 서비스(NodePort Type)의 일부 단점을 보완한 서비스(LoadBalancer Type) 도 있습니다!

![](/img/ingress-04.png)



4. 인그레스 컨트롤러 파드를 배치 : 서비스 앞단에 HTTP 고급 라우팅 등 기능 동작을 위한 배치

- 인그레스(정책)이 적용된 인그레스 컨트롤러 파드(예. nginx pod)를 앞단에 배치하여 고급 라우팅 등 기능을 제공

![](/img/ingress-05.png)





5. 인그레스 컨트롤러 파드 이중화 구성 :  Active(Leader) - Standby(Follower) 로 Active 파드 장애에 대비

![](/img/ingress-06.png)



6. 인그레스 컨트롤러 파드를 외부에 노출 : 인그레스 컨트롤러 파드를 외부에서 접속하기 위해서 노출(expose)

- 인그레스 컨트롤러 노출 시 서비스(NodePort Type) 보다는 좀 더 많은 기능을 제공하는 서비스(LoadBalancer Type)를 권장합니다 (80/443 포트 오픈 시)

![](/img/ingress-07.png)



7. 인그레스와 파드간 내부 연결의 효율화 방안 : 인그레스 컨트롤러 파드(Layer7 동작)에서 서비스 파드의 IP로 직접 연결

- 인그레스 컨트롤러 파드는 K8S API서버로부터 서비스의 엔드포인트 정보(파드 IP)를 획득 후 바로 파드의 IP로 연결 - [링크](https://kubernetes.github.io/ingress-nginx/user-guide/miscellaneous/#why-endpoints-and-not-services)
- 지원되는 인그레스 컨트롤러 : Nginx, Traefix 등 현재 대부분의 인그레스 컨트롤러가 지원함

![](/img/ingress-08.png)





# 인그레스(Ingress) 소개

인그레스 소개 : 클러스터 내부의 서비스(ClusterIP, NodePort, Loadbalancer)를 외부로 노출(HTTP/HTTPS) - Web Proxy 역할



#### 인그레스 기능 : HTTP(서비스) 부하분산 , 카나리 업그레이드

![](/img/ingress-09.png)



#### 인그레스 컨트롤러 : 인그레스의 실제 동작 구현은 인그레스 컨트롤러(Nginx, Kong 등)가 담당

![](/img/ingress-10.png)



#### 인그레스 + 인그레스 컨트롤러(Nginx) 기능 : HTTP(서비스) 부하분산 , 카나리 업그레이드 , HTTPS 처리(TLS 종료)

![](/img/ingress-11.png)







# Nginx 인그레스 컨트롤러 설치



~~~sh
# Ingress-Nginx 컨트롤러 생성
cat <<EOT> ingress-nginx-values.yaml
controller:
  service:
    type: NodePort
    nodePorts:
      http: 30080
      https: 30443
  nodeSelector:
    kubernetes.io/hostname: "k3s-s"
  metrics:
    enabled: true
  serviceMonitor:
      enabled: true
EOT

helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

kubectl create ns ingress
helm install ingress-nginx ingress-nginx/ingress-nginx -f ingress-nginx-values.yaml --namespace ingress --version 4.11.2

# 확인
kubectl get all -n ingress
kc describe svc -n ingress ingress-nginx-controller

# externalTrafficPolicy 설정
kubectl patch svc -n ingress ingress-nginx-controller -p '{"spec":{"externalTrafficPolicy": "Local"}}'

# 기본 nginx conf 파일 확인
kc describe cm -n ingress ingress-nginx-controller
kubectl exec deploy/ingress-nginx-controller -n ingress -it -- cat /etc/nginx/nginx.conf

# 관련된 정보 확인 : 포드(Nginx 서버), 서비스, 디플로이먼트, 리플리카셋, 컨피그맵, 롤, 클러스터롤, 서비스 어카운트 등
kubectl get all,sa,cm,secret,roles -n ingress
kc describe clusterroles ingress-nginx
kubectl get pod,svc,ep -n ingress -o wide -l app.kubernetes.io/component=controller

# 버전 정보 확인
POD_NAMESPACE=ingress
POD_NAME=$(kubectl get pods -n $POD_NAMESPACE -l app.kubernetes.io/name=ingress-nginx --field-selector=status.phase=Running -o name)
kubectl exec $POD_NAME -n $POD_NAMESPACE -- /nginx-ingress-controller --version
~~~









# 인그레스(Ingress) 실습 및 통신 흐름 확인



- 컨트롤플레인 노드에 인그레스 컨트롤러(Nginx) 파드를 생성, NodePort 로 외부에 노출
- 인그레스 정책 설정 : Host/Path routing, 실습의 편리를 위해서 도메인 없이 IP로 접속 설정 가능

![](/img/ingress-12.png)

![](/img/ingress-13.png)







## 디플로이먼트와 서비스를 생성

svc1-pod.yaml

~~~yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy1-websrv
spec:
  replicas: 1
  selector:
    matchLabels:
      app: websrv
  template:
    metadata:
      labels:
        app: websrv
    spec:
      containers:
      - name: pod-web
        image: nginx
---
apiVersion: v1
kind: Service
metadata:
  name: svc1-web
spec:
  ports:
    - name: web-port
      port: 9001
      targetPort: 80
  selector:
    app: websrv
  type: ClusterIP
~~~

svc2-pod.yaml

~~~yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy2-guestsrv
spec:
  replicas: 2
  selector:
    matchLabels:
      app: guestsrv
  template:
    metadata:
      labels:
        app: guestsrv
    spec:
      containers:
      - name: pod-guest
        image: gcr.io/google-samples/kubernetes-bootcamp:v1
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: svc2-guest
spec:
  ports:
    - name: guest-port
      port: 9002
      targetPort: 8080
  selector:
    app: guestsrv
  type: NodePort
~~~

svc3-pod.yaml

~~~yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy3-adminsrv
spec:
  replicas: 3
  selector:
    matchLabels:
      app: adminsrv
  template:
    metadata:
      labels:
        app: adminsrv
    spec:
      containers:
      - name: pod-admin
        image: k8s.gcr.io/echoserver:1.5
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: svc3-admin
spec:
  ports:
    - name: admin-port
      port: 9003
      targetPort: 8080
  selector:
    app: adminsrv
~~~

생성


~~~sh
# 생성
kubectl taint nodes k3s-s role=controlplane:NoSchedule
kubectl apply -f svc1-pod.yaml,svc2-pod.yaml,svc3-pod.yaml

# 확인 : svc1, svc3 은 ClusterIP 로 클러스터 외부에서는 접속할 수 없다 >> Ingress 는 연결 가능!
kubectl get pod,svc,ep
~~~

![](/img/ingress-14.png)





## 인그레스(정책) 생성 - [링크](https://kubetm.github.io/k8s/08-intermediate-controller/ingress/)



![](/img/ingress-15.png)

ingress1

~~~yaml
cat <<EOT> ingress1.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-1
  annotations:
    #nginx.ingress.kubernetes.io/upstream-hash-by: "true"
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: svc1-web
            port:
              number: 80
      - path: /guest
        pathType: Prefix
        backend:
          service:
            name: svc2-guest
            port:
              number: 8080
      - path: /admin
        pathType: Prefix
        backend:
          service:
            name: svc3-admin
            port:
              number: 8080
EOT
~~~



~~~sh
# 모니터링
watch -d 'kubectl get ingress,svc,ep,pod -owide'

# 생성
kubectl apply -f ingress1.yaml

# 확인
kubectl get ingress
kc describe ingress ingress-1
...
Rules:
  Host        Path  Backends
  ----        ----  --------
  *           
              /        svc1-web:80 ()
              /guest   svc2-guest:8080 ()
              /admin   svc3-admin:8080 ()
...

# 설정이 반영된 nginx conf 파일 확인
kubectl exec deploy/ingress-nginx-controller -n ingress -it -- cat /etc/nginx/nginx.conf

~~~

![](/img/ingress-16.png)



## 인그레스를 통한 내부 접속

- Nginx 인그레스 컨트롤러를 통한 접속(HTTP 인입) 경로 : 인그레스 컨트롤러 파드에서 서비스 파드의 IP로 직접 연결

![](/img/ingress-17.png)





인그레스(Nginx 인그레스 컨트롤러)를 통한 접속(HTTP 인입) 확인

- **HTTP 부하분산 & PATH 기반 라우팅, 애플리케이션 파드에 연결된 서비스는 Bypass**



~~~sh

#
kubectl get ingress
NAME        CLASS   HOSTS   ADDRESS        PORTS   AGE
ingress-1   nginx   *       10.10.200.24   80      3m44s
 
kubectl describe ingress ingress-1 | sed -n "5, \$p"
Rules:
  Host        Path   Backends
  ----        ----   --------
  *           /      svc1-web:80 ()
              /guest svc2-guest:8080 ()
              /admin svc3-admin:8080 ()


# 접속 로그 확인 : kubetail 설치되어 있음 - 출력되는 nginx 의 로그의 IP 확인
kubetail -n ingress -l app.kubernetes.io/component=controller

-------------------------------
# 자신의 집 PC에서 인그레스를 통한 접속 : 각각 
echo -e "Ingress1 sv1-web URL = http://$(curl -s ipinfo.io/ip):30080"
echo -e "Ingress1 sv2-guest URL = http://$(curl -s ipinfo.io/ip):30080/guest"
echo -e "Ingress1 sv3-admin URL = http://$(curl -s ipinfo.io/ip):30080/admin"

# svc1-web 접속
MYIP=<EC2 공인 IP>
MYIP=13.124.93.150
curl -s $MYIP:30080

# svvc2-guest 접속
curl -s $MYIP:30080/guest
curl -s $MYIP:30080/guest
for i in {1..100}; do curl -s $MYIP:30080/guest ; done | sort | uniq -c | sort -nr

# svc3-admin 접속 > 기본적으로 Nginx 는 라운드로빈 부하분산 알고리즘을 사용 >> Client_address 와 XFF 주소는 어떤 주소인가요?
curl -s $MYIP:30080/admin
curl -s $MYIP:30080/admin | egrep '(client_address|x-forwarded-for)'
for i in {1..100}; do curl -s $MYIP:30080/admin | grep Hostname ; done | sort | uniq -c | sort -nr


# (옵션) 디플로이먼트의 파드 갯수를 증가/감소 설정 후 접속 테스트 해보자
~~~

![](/img/ingress-18.png)



노드에서 아래 패킷 캡처 확인 : flannel vxlan, 파드 간 통신 시 IP 정보 확인

~~~sh
#
ngrep -tW byline -d ens5 '' udp port 8472 or tcp port 80

#
tcpdump -i ens5 udp port 8472 -nn

# vethY는 각자 k3s-s 의 가장 마지막 veth 를 지정
tcpdump -i vethY tcp port 8080 -nn
tcpdump -i vethY tcp port 8080 -w /tmp/ingress-nginx.pcap

# 자신의 PC에서 k3s-s EC2 공인 IP로 pcap 다운로드
scp ubuntu@<k3s-s EC2 공인 IP>:/tmp/ingress-nginx.pcap ~/Downloads
scp ubuntu@52.78.155.19:/tmp/ingress-nginx.pcap ~/Downloads
~~~







## Nginx 분산 알고리즘 변경

- nginx 는 기본 RR 라운드 로빈 이지만, IP-Hash 나 Session Cookie 설정으로 대상 유지 가능 - [링크](https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/configmap/#load-balance)

~~~yaml
  annotations:
    nginx.ingress.kubernetes.io/upstream-hash-by: "true"
~~~

확인

~~~sh
# mypc
for i in {1..100}; do curl -s $MYIP:30080/admin | grep Hostname ; done | sort | uniq -c | sort -nr
while true; do curl -s --connect-timeout 1 $MYIP:30080/admin | grep Hostname ; date "+%Y-%m-%d %H:%M:%S" ; echo "--------------" ; sleep 1; done

# 아래 ingress 설정 중 IP-Hash 설정 > # 주석 제거
sed -i 's/#nginx.ingress/nginx.ingress/g' ingress1.yaml
kubectl apply -f ingress1.yaml

# 접속 확인
for i in {1..100}; do curl -s $MYIP:30080/admin | grep Hostname ; done | sort | uniq -c | sort -nr
while true; do curl -s --connect-timeout 1 $MYIP:30080/admin | grep Hostname ; date "+%Y-%m-%d %H:%M:%S" ; echo "--------------" ; sleep 1; done

# 다시 원복(라운드 로빈) > # 주석 추가
sed -i 's/nginx.ingress/#nginx.ingress/g' ingress1.yaml
kubectl apply -f ingress1.yaml

# 접속 확인
for i in {1..100}; do curl -s $MYIP:30080/admin | grep Hostname ; done | sort | uniq -c | sort -nr
while true; do curl -s --connect-timeout 1 $MYIP:30080/admin | grep Hostname ; date "+%Y-%m-%d %H:%M:%S" ; echo "--------------" ; sleep 1; done
~~~

삭제

~~~sh
kubectl delete deployments,svc,ingress --all
~~~





## AWS Ingress (ALB) 모드

-  인스턴스 모드 : AWS ALB(Ingress)로 인입 후 각 워커노드의 NodePort 로 전달 후 IPtables 룰(SEP)에 따라 파드로 분배

![](/img/ingress-19.png)



- IP 모드 : nginx ingress controller 동작과 유사하게 AWS LoadBalancer Controller 파드가 kube api 를 통해서 파드의 IP를 제공받아서 AWS ALB 에 타켓(파드 IP)를 설정

![](/img/ingress-20.png)





## Host 기반 라우팅



![](/img/ingress-21.png)



~~~yaml
cat <<EOT> ingress2.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-2
spec:
  ingressClassName: nginx
  rules:
  - host: kans.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: svc3-admin
            port:
              number: 8080
  - host: "*.kans.com"
    http:
      paths:
      - path: /echo
        pathType: Prefix
        backend:
          service:
            name: svc3-admin
            port:
              number: 8080
EOT
~~~

생성

~~~sh
# 터미널1
watch -d 'kubectl get ingresses,svc,ep,pod -owide'

# 도메인 변경
MYDOMAIN1=staek.com
sed -i "s/kans.com/$MYDOMAIN1/g" ingress2.yaml

# 생성
kubectl apply -f ingress2.yaml,svc3-pod.yaml

# 확인
kubectl get ingress
kubectl describe ingress ingress-2

kubectl describe ingress ingress-2 | sed -n "5, \$p"
Ingress Class:    nginx
Default backend:  <default>
Rules:
  Host         Path  Backends
  ----         ----  --------
  staek.com
               /   svc3-admin:8080 ()
  *.staek.com
               /echo   svc3-admin:8080 ()
Annotations:   <none>
Events:        <none>
~~~

#### 인그레스(Nginx 인그레스 컨트롤러)를 통한 접속(HTTP 인입) 확인





~~~yaml
# 로그 모니터링
kubetail -n ingress -l app.kubernetes.io/component=controller

# (옵션) ingress nginx 파드 vethY 에서 패킷 캡처 후 확인 해 볼 것

------------
# 자신의 PC 에서 접속 테스트
# svc3-admin 접속 > 결과 확인 : 왜 접속이 되지 않는가? HTTP 헤더에 Host 필드를 잘 확인해보자!
curl $MYIP:30080 -v
curl $MYIP:30080/echo -v

# mypc에서 접속을 위한 설정
## /etc/hosts 수정 : 도메인 이름으로 접속하기 위해서 변수 지정
## 윈도우 C:\Windows\System32\drivers\etc\hosts
## 맥 sudo vim /etc/hosts
MYDOMAIN1=<각자 자신의 닉네임의 도메인>
MYDOMAIN2=<test.각자 자신의 닉네임의 도메인>
MYDOMAIN1=staek.com
MYDOMAIN2=test.staek.com
echo $MYIP $MYDOMAIN1 $MYDOMAIN2

echo "$MYIP $MYDOMAIN1" | sudo tee -a /etc/hosts
echo "$MYIP $MYDOMAIN2" | sudo tee -a /etc/hosts
cat /etc/hosts | grep $MYDOMAIN1

# svc3-admin 접속 > 결과 확인
curl $MYDOMAIN1:30080 -v
curl $MYDOMAIN1:30080/admin
curl $MYDOMAIN1:30080/echo
curl $MYDOMAIN1:30080/echo/1

curl $MYDOMAIN2:30080 -v
curl $MYDOMAIN2:30080/admin
curl $MYDOMAIN2:30080/echo
curl $MYDOMAIN2:30080/echo/1
curl $MYDOMAIN2:30080/echo/1/2

## (옵션) /etc/hosts 파일 변경 없이 접속 방안
curl -H "host: $MYDOMAIN1" $MYIP:30080
~~~





## 카나리 업그레이드

canary-svc1-pod.yaml

~~~yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dp-v1
spec:
  replicas: 3
  selector:
    matchLabels:
      app: svc-v1
  template:
    metadata:
      labels:
        app: svc-v1
    spec:
      containers:
      - name: pod-v1
        image: k8s.gcr.io/echoserver:1.5
        ports:
        - containerPort: 8080
      terminationGracePeriodSeconds: 0
---
apiVersion: v1
kind: Service
metadata:
  name: svc-v1
spec:
  ports:
    - name: web-port
      port: 9001
      targetPort: 8080
  selector:
    app: svc-v1
~~~

canary-svc2-pod.yaml

~~~yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dp-v2
spec:
  replicas: 3
  selector:
    matchLabels:
      app: svc-v2
  template:
    metadata:
      labels:
        app: svc-v2
    spec:
      containers:
      - name: pod-v2
        image: k8s.gcr.io/echoserver:1.6
        ports:
        - containerPort: 8080
      terminationGracePeriodSeconds: 0
---
apiVersion: v1
kind: Service
metadata:
  name: svc-v2
spec:
  ports:
    - name: web-port
      port: 9001
      targetPort: 8080
  selector:
    app: svc-v2
~~~



~~~yaml
# 터미널1
watch -d 'kubectl get ingress,svc,ep,pod -owide'

# 생성
curl -s -O https://raw.githubusercontent.com/gasida/NDKS/main/7/canary-svc1-pod.yaml
curl -s -O https://raw.githubusercontent.com/gasida/NDKS/main/7/canary-svc2-pod.yaml
kubectl apply -f canary-svc1-pod.yaml,canary-svc2-pod.yaml

# 확인
kubectl get svc,ep,pod

# 파드 버전 확인: 1.13.0 vs 1.13.1
for pod in $(kubectl get pod -o wide -l app=svc-v1 |awk 'NR>1 {print $6}'); do curl -s $pod:8080 | egrep '(Hostname|nginx)'; done

Hostname: dp-v1-8684d45558-9kb59
	server_version=nginx: 1.13.0 - lua: 10008
Hostname: dp-v1-8684d45558-drwzf
	server_version=nginx: 1.13.0 - lua: 10008
Hostname: dp-v1-8684d45558-hzcth
	server_version=nginx: 1.13.0 - lua: 10008
	
	
for pod in $(kubectl get pod -o wide -l app=svc-v2 |awk 'NR>1 {print $6}'); do curl -s $pod:8080 | egrep '(Hostname|nginx)'; done


Hostname: dp-v2-7757c4bdc-b8bkt
	server_version=nginx: 1.13.1 - lua: 10008
Hostname: dp-v2-7757c4bdc-ggx8l
	server_version=nginx: 1.13.1 - lua: 10008
Hostname: dp-v2-7757c4bdc-zfxfc
	server_version=nginx: 1.13.1 - lua: 10008
	
~~~

![](/img/ingress-22.png)

canary-ingress1.yaml

~~~yaml
cat <<EOT> canary-ingress1.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-canary-v1
spec:
  ingressClassName: nginx
  rules:
  - host: kans.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: svc-v1
            port:
              number: 8080
EOT
~~~

canary-ingress2.yaml

~~~yaml
cat <<EOT> canary-ingress2.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-canary-v2
  annotations:
    nginx.ingress.kubernetes.io/canary: "true"
    nginx.ingress.kubernetes.io/canary-weight: "10"
spec:
  ingressClassName: nginx
  rules:
  - host: kans.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: svc-v2
            port:
              number: 8080
EOT
~~~







~~~yaml
# 터미널1
watch -d 'kubectl get ingress,svc,ep'

# 도메인 변경
MYDOMAIN1=staek.com
sed -i "s/kans.com/$MYDOMAIN1/g" canary-ingress1.yaml
sed -i "s/kans.com/$MYDOMAIN1/g" canary-ingress2.yaml

# 생성
kubectl apply -f canary-ingress1.yaml,canary-ingress2.yaml

# 로그 모니터링
kubetail -n ingress -l app.kubernetes.io/component=controller

# 접속 테스트
curl -s $MYDOMAIN1:30080
curl -s $MYDOMAIN1:30080 | grep nginx

# 접속 시 v1 v2 버전별 비율이 어떻게 되나요? 왜 이렇게 되나요?
for i in {1..100};  do curl -s $MYDOMAIN1:30080 | grep nginx ; done | sort | uniq -c | sort -nr
for i in {1..1000}; do curl -s $MYDOMAIN1:30080 | grep nginx ; done | sort | uniq -c | sort -nr
while true; do curl -s --connect-timeout 1 $MYDOMAIN1:30080 | grep Hostname ; echo "--------------" ; date "+%Y-%m-%d %H:%M:%S" ; sleep 1; done

# 비율 조정 >> 개발 배포 버전 전략에 유용하다!
kubectl annotate --overwrite ingress ingress-canary-v2 nginx.ingress.kubernetes.io/canary-weight=50

# 접속 테스트
for i in {1..100};  do curl -s $MYDOMAIN1:30080 | grep nginx ; done | sort | uniq -c | sort -nr
for i in {1..1000}; do curl -s $MYDOMAIN1:30080 | grep nginx ; done | sort | uniq -c | sort -nr

# (옵션) 비율 조정 << 어떻게 비율이 조정될까요?
kubectl annotate --overwrite ingress ingress-canary-v2 nginx.ingress.kubernetes.io/canary-weight=100
for i in {1..100};  do curl -s $MYDOMAIN1:30080 | grep nginx ; done | sort | uniq -c | sort -nr

# (옵션) 비율 조정 << 어떻게 비율이 조정될까요?
kubectl annotate --overwrite ingress ingress-canary-v2 nginx.ingress.kubernetes.io/canary-weight=0
for i in {1..100};  do curl -s $MYDOMAIN1:30080 | grep nginx ; done | sort | uniq -c | sort -nr
~~~



아래와 같이 기본 비율과 `canary-weight=50` 비율 차이를 보자.

![](/img/ingress-23.png)

~~~sh
kubectl annotate --overwrite ingress ingress-canary-v2 nginx.ingress.kubernetes.io/canary-weight=50
~~~

![](/img/ingress-24.png)



삭제

~~~sh
kubectl delete deployments,svc,ingress --all
~~~





## HTTPS 처리 (TLS 종료) - [링크](https://kubernetes.github.io/ingress-nginx/examples/tls-termination/)



![](/img/ingress-25.png)



svc-pod.yaml

~~~yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-https
  labels:
    app: https
spec:
  containers:
  - name: container
    image: k8s.gcr.io/echoserver:1.6
  terminationGracePeriodSeconds: 0
---
apiVersion: v1
kind: Service
metadata:
  name: svc-https
spec:
  selector:
    app: https
  ports:
  - port: 8080
~~~

ssl-termination-ingress.yaml

~~~yaml
cat <<EOT> ssl-termination-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: https
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - kans.com
    secretName: secret-https
  rules:
  - host: kans.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: svc-https
            port:
              number: 8080
EOT
~~~





~~~sh
# 서비스와 파드 생성
curl -s -O https://raw.githubusercontent.com/gasida/NDKS/main/7/svc-pod.yaml
kubectl apply -f svc-pod.yaml

# 도메인 변경
MYDOMAIN1=<각자 자신의 닉네임의 도메인> 예시) gasida.com
MYDOMAIN1=kans.com
echo $MYDOMAIN1
sed -i "s/kans.com/$MYDOMAIN1/g" ssl-termination-ingress.yaml

# 인그레스 생성
kubectl apply -f ssl-termination-ingress.yaml

# 인증서 생성
# openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=dkos.com/O=dkos.com"mkdir key && cd key
MYDOMAIN1=kans.com
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=$MYDOMAIN1/O=$MYDOMAIN1"
tree

# Secret 생성
kubectl create secret tls secret-https --key tls.key --cert tls.crt

# Secret 확인 
kubectl get secrets secret-https
kubectl get secrets secret-https -o yaml

-------------------
# 자신의 PC 에서 접속 확인 : PC 웹브라우저
# 접속 확인 : -k 는 https 접속 시 : 접속 포트 정보 확인
curl -Lk https://$MYDOMAIN1:30443

## (옵션) /etc/hosts 파일 변경 없이 접속 방안
curl -Lk -H "host: $MYDOMAIN1" https://$MYDOMAIN1:30443
~~~

![](/img/ingress-26.png)

![](/img/ingress-27.png)

Nginx SSL Termination 패킷 확인 : 중간 172.16.29.11 이 nginx controller

~~~sh
# 패킷 캡처 명령어 참고
export IngHttp=$(kubectl get service -n ingress-nginx ingress-nginx-controller -o jsonpath='{.spec.ports[0].nodePort}')
export IngHttps=$(kubectl get service -n ingress-nginx ingress-nginx-controller -o jsonpath='{.spec.ports[1].nodePort}')
tcpdump -i <nginx 파드 veth> -nnq tcp port 80 or tcp port 443 or tcp port 8080 or tcp port $IngHttp or tcp port $IngHttps
tcpdump -i <nginx 파드 veth> -nn  tcp port 80 or tcp port 443 or tcp port 8080 or tcp port $IngHttp or tcp port $IngHttps -w /tmp/ingress.pcap
~~~



삭제

~~~sh
kubectl delete pod,svc,ingress --all
helm uninstall -n ingress ingress-nginx
~~~

~~~sh
# CloudFormation 스택 삭제
aws cloudformation delete-stack --stack-name mylab

# [모니터링] CloudFormation 스택 상태 : 삭제 확인
while true; do 
  date
  AWS_PAGER="" aws cloudformation list-stacks \
    --stack-status-filter CREATE_IN_PROGRESS CREATE_COMPLETE CREATE_FAILED DELETE_IN_PROGRESS DELETE_FAILED \
    --query "StackSummaries[*].{StackName:StackName, StackStatus:StackStatus}" \
    --output table
  sleep 1
done
~~~

























































