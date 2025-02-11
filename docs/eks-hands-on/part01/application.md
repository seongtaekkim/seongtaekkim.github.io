---
layout: default
title: pod 테스트하기
parent: eks 시작하기
grand_parent: eks 실습
nav_order: 3
---



### 노드에 배포된 컨테이너 정보 확인 Containerd clients 3종 : ctr, nerdctl, crictl

~~~
(Administrator@myeks:N/A) [root@myeks-host ~]# ssh ec2-user@$N1 ctr --version

ctr github.com/containerd/containerd 1.7.25
(Administrator@myeks:N/A) [root@myeks-host ~]# ssh ec2-user@$N1 ctr
NAME:
   ctr -
        __
  _____/ /______
 / ___/ __/ ___/
/ /__/ /_/ /
\___/\__/_/

containerd CLI


# 네임스페이스 확인
ssh ec2-user@$N1 sudo ctr ns list
NAME   LABELS
k8s.io

# 컨테이너 리스트 확인
for i in $N1 $N2; do echo ">> node $i <<"; ssh ec2-user@$i sudo ctr -n k8s.io container list; echo; done
CONTAINER                                                           IMAGE                                                                                          RUNTIME
28b6a15c475e32cd8777c1963ba684745573d0b6053f80d2d37add0ae841eb45    602401143452.dkr.ecr-fips.us-east-1.amazonaws.com/eks/pause:3.5                                io.containerd.runc.v2
4f266ebcee45b133c527df96499e01ec0c020ea72785eb10ef63b20b5826cf7c    602401143452.dkr.ecr-fips.us-east-1.amazonaws.com/eks/pause:3.5                                io.containerd.runc.v2
...

# 컨테이너 이미지 확인
for i in $N1 $N2; do echo ">> node $i <<"; ssh ec2-user@$i sudo ctr -n k8s.io image list --quiet; echo; done
...

# 태스크 리스트 확인
for i in $N1 $N2; do echo ">> node $i <<"; ssh ec2-user@$i sudo ctr -n k8s.io task list; echo; done
...

## 예시) 각 테스크의 PID(3706) 확인
ssh ec2-user@$N1 sudo ps -c 3706
PID CLS PRI TTY      STAT   TIME COMMAND
3099 TS   19 ?       Ssl    0:01 kube-proxy --v=2 --config=/var/lib/kube-proxy-config/config --hostname-override=ip-192-168-1-229.ap-northeast-2.compute.internal
~~~



#### ECR 퍼블릭 Repository 사용 : 퍼블릭 Repo 는 설정 시 us-east-1 를 사용

```bash
# 퍼블릭 ECR 인증
aws ecr-public get-login-password --region us-east-1 | docker login --username AWS --password-stdin public.ecr.aws

cat /root/.docker/config.json | jq

# 퍼블릭 Repo 기본 정보 확인
aws ecr-public describe-registries --region us-east-1 | jq

{
  "registries": [
    {
      "registryId": "738612635754",
      "registryArn": "arn:aws:ecr-public::738612635754:registry/738612635754",
      "registryUri": "",
      "verified": false,
      "aliases": []
    }
  ]
}

# 퍼블릭 Repo 생성
NICKNAME=seongtki
aws ecr-public create-repository --repository-name $NICKNAME/nginx --region us-east-1

# 생성된 퍼블릭 Repo 확인
aws ecr-public describe-repositories --region us-east-1 | jq
REPOURI=$(aws ecr-public describe-repositories --region us-east-1 | jq -r .repositories[].repositoryUri)
echo $REPOURI
public.ecr.aws/o2n8a7w3/seongtki/nginx


# 이미지 태그
docker pull nginx:alpine
docker images
docker tag nginx:alpine $REPOURI:latest
docker images

# 이미지 업로드
docker push $REPOURI:latest

# 파드 실행
kubectl run mynginx --image $REPOURI


kubectl get pod
NAME                     READY   STATUS              RESTARTS   AGE
mario-6d8c76fd8d-2mx7k   1/1     Running             0          10m
mynginx                  0/1     ContainerCreating   0          3s

kubectl delete pod mynginx

****# 퍼블릭 이미지 삭제
aws ecr-public batch-delete-image \
      --repository-name $NICKNAME/nginx \
      --image-ids imageTag=latest \
      --region us-east-1

# 퍼블릭 Repo 삭제
aws ecr-public delete-repository --repository-name $NICKNAME/nginx --force --region us-east-1
```



서비스/디플로이먼트(mario 게임) 배포 테스트 with AWS CLB

~~~yaml
# 터미널1 (모니터링)
watch -d 'kubectl get pod,svc'

# 수퍼마리오 디플로이먼트 배포
cat <<EOT > mario.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mario
  labels:
    app: mario
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mario
  template:
    metadata:
      labels:
        app: mario
    spec:
      containers:
      - name: mario
        image: pengbai/docker-supermario
---
apiVersion: v1
kind: Service
metadata:
   name: mario
spec:
  selector:
    app: mario
  ports:
  - port: 80
    protocol: TCP
    targetPort: 8080
  type: LoadBalancer
EOT
kubectl apply -f mario.yaml

# 배포 확인 : CLB 배포 확인
kubectl get deploy,svc,ep mario

# 마리오 게임 접속 : CLB 주소로 웹 접속
kubectl get svc mario -o jsonpath={.status.loadBalancer.ingress[0].hostname} | awk '{ print "Maria URL = http://"$1 }'

(Administrator@myeks:N/A) [root@myeks-host ~]# kubectl get svc mario -o jsonpath={.status.loadBalancer.ingress[0].hostname} | awk '{ print "Maria URL = http://"$1 }'
Maria URL = http://a3fbbb85ed6df4e0288fd3bf4944cfc3-734390217.ap-northeast-2.elb.amazonaws.com
~~~

![](/img/eks-001.png)







