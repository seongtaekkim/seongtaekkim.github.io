---
layout: default
title: local에서 k8s 설치하기
parent: eks 시작하기
grand_parent: eks 실습
nav_order: 1
---

# local > K8s



### install

~~~sh
sudo apt-get update
sudo apt-get install -y docker.io
apt-get update
sudo apt-get install -y docker.io
apt-get install -y docker.io
apt-get install sudo

systemctl start docker
systemctl enable docker
usermod -aG docker ens4

curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
~~~

### 설정파일

~~~sh
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        system-reserved: memory=1Gi
- role: worker
  kubeadmConfigPatches:
  - |
    kind: JoinConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        system-reserved: memory=2Gi
- role: worker
  kubeadmConfigPatches:
  - |
    kind: JoinConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        system-reserved: memory=2Gi
~~~



### 실행

~~~
root@ens4:~# kind create cluster --config kind-config.yaml
Creating cluster "kind" ...
 ✓ Ensuring node image (kindest/node:v1.27.3) 🖼
 ✓ Preparing nodes 📦 📦 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing CNI 🔌
 ✓ Installing StorageClass 💾
 ✓ Joining worker nodes 🚜
Set kubectl context to "kind-kind"
You can now use your cluster with:

kubectl cluster-info --context kind-kind

Have a nice day! 👋
~~~



### 테라폼 설치

~~~
# Terraform 1.5.5 버전 다운로드 (버전은 필요에 따라 변경)
curl -LO https://releases.hashicorp.com/terraform/1.5.5/terraform_1.5.5_linux_amd64.zip

# 다운로드한 zip 파일 압축 해제
unzip terraform_1.5.5_linux_amd64.zip

# terraform 바이너리를 /usr/local/bin에 이동하여 PATH에 포함
sudo mv terraform /usr/local/bin/
~~~



### 테라폼 이용해서 실행

~~~
root@ens4:~# terraform apply

Terraform used the selected providers to generate the following execution plan. Resource actions are
indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # null_resource.kind_cluster will be created
  + resource "null_resource" "kind_cluster" {
      + id = (known after apply)
    }

Plan: 1 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

null_resource.kind_cluster: Creating...
null_resource.kind_cluster: Provisioning with 'local-exec'...
null_resource.kind_cluster (local-exec): Executing: ["/bin/sh" "-c" "kind create cluster --config=kind-config.yaml"]
null_resource.kind_cluster (local-exec): Creating cluster "kind" ...
null_resource.kind_cluster (local-exec):  • Ensuring node image (kindest/node:v1.27.3) 🖼  ...
null_resource.kind_cluster (local-exec):  ✓ Ensuring node image (kindest/node:v1.27.3) 🖼
null_resource.kind_cluster (local-exec):  • Preparing nodes 📦 📦 📦   ...
null_resource.kind_cluster (local-exec):  ✓ Preparing nodes 📦 📦 📦
null_resource.kind_cluster (local-exec):  • Writing configuration 📜  ...
null_resource.kind_cluster (local-exec):  ✓ Writing configuration 📜
null_resource.kind_cluster (local-exec):  • Starting control-plane 🕹️  ...
null_resource.kind_cluster: Still creating... [10s elapsed]
null_resource.kind_cluster (local-exec):  ✓ Starting control-plane 🕹️
null_resource.kind_cluster (local-exec):  • Installing CNI 🔌  ...
null_resource.kind_cluster (local-exec):  ✓ Installing CNI 🔌
null_resource.kind_cluster (local-exec):  • Installing StorageClass 💾  ...
null_resource.kind_cluster (local-exec):  ✓ Installing StorageClass 💾
null_resource.kind_cluster (local-exec):  • Joining worker nodes 🚜  ...
null_resource.kind_cluster: Still creating... [20s elapsed]
null_resource.kind_cluster: Still creating... [30s elapsed]
null_resource.kind_cluster (local-exec):  ✓ Joining worker nodes 🚜
null_resource.kind_cluster (local-exec): Set kubectl context to "kind-kind"
null_resource.kind_cluster (local-exec): You can now use your cluster with:

null_resource.kind_cluster (local-exec): kubectl cluster-info --context kind-kind

null_resource.kind_cluster (local-exec): Have a nice day! 👋
null_resource.kind_cluster: Creation complete after 37s [id=5176424687883728826]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
~~~



### k8s 엔드포인트 확인

~~~sh
root@ens4:~# docker ps
CONTAINER ID   IMAGE                  COMMAND                  CREATED          STATUS          PORTS                       NAMES
fb8992a8caba   kindest/node:v1.27.3   "/usr/local/bin/entr…"   10 minutes ago   Up 10 minutes                               kind-worker
c9af0e2ee5a2   kindest/node:v1.27.3   "/usr/local/bin/entr…"   10 minutes ago   Up 10 minutes   127.0.0.1:38707->6443/tcp   kind-control-plane
03c2b1076001   kindest/node:v1.27.3   "/usr/local/bin/entr…"   10 minutes ago   Up 10 minutes                               kind-worker2


# 직접확인
docker exec -it c9af0e2ee5a2  bash
root@kind-control-plane:/# curl -k https://localhost:6443/version
{
  "major": "1",
  "minor": "27",
  "gitVersion": "v1.27.3",
  "gitCommit": "25b4e43193bcda6c7328a6d147b1fb73a33f1598",
  "gitTreeState": "clean",
  "buildDate": "2023-06-15T00:36:28Z",
  "goVersion": "go1.20.5",
  "compiler": "gc",
  "platform": "linux/amd64"
}


# 포트로 확인
root@ens4:~/.kube# cat config | grep server
    server: https://127.0.0.1:38707

root@ens4:~/.kube# curl -k https://127.0.0.1:38707/version
{
  "major": "1",
  "minor": "27",
  "gitVersion": "v1.27.3",
  "gitCommit": "25b4e43193bcda6c7328a6d147b1fb73a33f1598",
  "gitTreeState": "clean",
  "buildDate": "2023-06-15T00:36:28Z",
  "goVersion": "go1.20.5",
  "compiler": "gc",
  "platform": "linux/amd64"
}
~~~





## 기본 사용

선언형 멱등성 알아보기 실습

~~~sh
# 터미널1 (모니터링)
watch -d 'kubectl get pod'

# 터미널2
# Deployment 배포(Pod 3개)
kubectl create deployment my-webs --image=gcr.io/google-samples/kubernetes-bootcamp:v1 --replicas=3
kubectl get pod -w

# 파드 증가 및 감소
kubectl scale deployment my-webs --replicas=6 && kubectl get pod -w
kubectl scale deployment my-webs --replicas=3
kubectl get pod

# 강제로 파드 삭제 : 바라는상태 + 선언형에 대한 대략적인 확인! ⇒ 어떤 일이 벌어지는가? 
kubectl delete pod --all && kubectl get pod -w
kubectl get pod

# 실습 완료 후 Deployment 삭제
kubectl delete deploy my-webs
~~~



~~~sh
Every 2.0s: kubectl get pod                                              kind-control-plane: Sat Feb  8 13:06:39 2025

NAME                       READY   STATUS    RESTARTS   AGE
my-webs-684fdf4675-726kp   1/1     Running   0          47s
my-webs-684fdf4675-kpc6k   1/1     Running   0          47s
my-webs-684fdf4675-l52qt   1/1     Running   0          14s
my-webs-684fdf4675-rq229   1/1     Running   0          14s
my-webs-684fdf4675-vd2mw   1/1     Running   0          47s
my-webs-684fdf4675-zhrdt   1/1     Running   0          14s
~~~



### 서비스/디플로이먼트(mario 게임) 배포 테스트 with AWS CLB

~~~sh
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
  type: NodePort
EOT
kubectl apply -f mario.yaml

kubectl get deploy,svc,ep mario
~~~











