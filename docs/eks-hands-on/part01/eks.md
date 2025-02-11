---
layout: default
title: eks 설치, public & privaet 모드
parent: eks 시작하기
grand_parent: eks 실습
nav_order: 2
---


### cloudformation 으로 aws 구조 설치

~~~sh
aws cloudformation deploy --template-file ./myeks.yaml \
     --stack-name myeks --parameter-overrides KeyName=container SgIngressSshCidr=$(curl -s ipinfo.io/ip)/32 --region ap-northeast-2
     
     
## ssh 접속

ssh root@$(aws cloudformation describe-stacks --stack-name myeks --query 'Stacks[*].Outputs[0].OutputValue' --output text)
root@@X.Y.Z.A's password: qwe123
~~~



### 베스천 자격증명 확인

~~~sh
[root@myeks-host ~]# aws configure
AWS Access Key ID [None]: A*************
AWS Secret Access Key [None]: wiN***zxq4/******************
Default region name [None]: ap-northeast-2
Default output format [None]:
[root@myeks-host ~]#
[root@myeks-host ~]# aws sts  get-caller-identity
{
    "UserId": "A*************",
    "Account": "73*********",
    "Arn": "arn:aws:iam::73*********",:user/Administrator"
}
~~~



- 기본 정보 확인

```bash
# (옵션) cloud-init 실행 과정 로그 확인
tail -f /var/log/cloud-init-output.log

# 사용자 확인
whoami

# 기본 툴 및 SSH 키 설치 등 확인
**kubectl version --client=true -o yaml**
**eksctl version**
**aws --version**
**ls /root/.ssh/id_rsa***

# 도커 엔진 설치 확인
**docker info**
```

- IAM User 자격 증명 설정 및 VPC 확인 및 **변수 지정**

```bash
# EKS 배포할 VPC 정보 확인
**aws ec2 describe-vpcs --filters "Name=tag:Name,Values=$CLUSTER_NAME-VPC" | jq**
aws ec2 describe-vpcs --filters "Name=tag:Name,Values=$CLUSTER_NAME-VPC" | jq Vpcs[]
aws ec2 describe-vpcs --filters "Name=tag:Name,Values=$CLUSTER_NAME-VPC" | jq Vpcs[].VpcId
aws ec2 describe-vpcs --filters "Name=tag:Name,Values=$CLUSTER_NAME-VPC" | jq -r .Vpcs[].VpcId
**export VPCID=$(aws ec2 describe-vpcs --filters "Name=tag:Name,Values=$CLUSTER_NAME-VPC" | jq -r .Vpcs[].VpcId)
echo "export VPCID=$VPCID" >> /etc/profile**
echo $VPCID

****# EKS 배포할 VPC에 속한 Subnet 정보 확인
aws ec2 describe-subnets --filters "Name=vpc-id,Values=$VPCID" --output json | jq
aws ec2 describe-subnets --filters "Name=vpc-id,Values=$VPCID" --output yaml

****## 퍼블릭 서브넷 ID 확인
**aws ec2 describe-subnets --filters Name=tag:Name,Values="$CLUSTER_NAME-PublicSubnet1" | jq**
aws ec2 describe-subnets --filters Name=tag:Name,Values="$CLUSTER_NAME-PublicSubnet1" --query "Subnets[0].[SubnetId]" --output text
**export PubSubnet1=$(aws ec2 describe-subnets --filters Name=tag:Name,Values="$CLUSTER_NAME-PublicSubnet1" --query "Subnets[0].[SubnetId]" --output text)
export PubSubnet2=$(aws ec2 describe-subnets --filters Name=tag:Name,Values="$CLUSTER_NAME-PublicSubnet2" --query "Subnets[0].[SubnetId]" --output text)
echo "export PubSubnet1=$PubSubnet1" >> /etc/profile
echo "export PubSubnet2=$PubSubnet2" >> /etc/profile**
echo $PubSubnet1
echo $PubSubnet2
```



### 관리형 노드 설치

컨트롤플레인 설치

```bash
# 변수 확인***
echo $AWS_DEFAULT_REGION
echo $CLUSTER_NAME
echo $VPCID
echo $PubSubnet1,$PubSubnet2

# 옵션 [터미널1] EC2 생성 모니터링
**while true; do aws ec2 describe-instances --query "Reservations[*].Instances[*].{PublicIPAdd:PublicIpAddress,PrivateIPAdd:PrivateIpAddress,InstanceName:Tags[?Key=='Name']|[0].Value,Status:State.Name}" --filters Name=instance-state-name,Values=running --output text ; echo "------------------------------" ; sleep 1; done**
aws ec2 describe-instances --query "Reservations[*].Instances[*].{PublicIPAdd:PublicIpAddress,PrivateIPAdd:PrivateIpAddress,InstanceName:Tags[?Key=='Name']|[0].Value,Status:State.Name}" --filters Name=instance-state-name,Values=running --output table

# eks 클러스터 & 관리형노드그룹 배포 전 정보 확인
**eksctl create cluster --name $CLUSTER_NAME --region=$AWS_DEFAULT_REGION --nodegroup-name=$CLUSTER_NAME-nodegroup --node-type=t3.medium \\
--node-volume-size=30 --vpc-public-subnets "$PubSubnet1,$PubSubnet2" --version 1.31 --ssh-access --external-dns-access --dry-run** | yh
****...
**vpc**:
  autoAllocateIPv6: false
  cidr: 192.168.0.0/16
  clusterEndpoints:
    privateAccess: false
    publicAccess: true
  **id**: vpc-0505d154771a3dfdf
  manageSharedNodeSecurityGroupRules: true
  nat:
    gateway: Disable
  **subnets**:
    **public**:
      ap-northeast-2a:
        az: ap-northeast-2a
        cidr: 192.168.1.0/24
        id: subnet-0d98bee5a7c0dfcc6
      ap-northeast-2c:
        az: ap-northeast-2c
        cidr: 192.168.2.0/24
        id: subnet-09dc49de8d899aeb7
****
**# eks 클러스터 & 관리형노드그룹 배포: 총 15분 소요**
**eksctl create cluster --name $CLUSTER_NAME --region=$AWS_DEFAULT_REGION --nodegroup-name=$CLUSTER_NAME-nodegroup --node-type=t3.medium \\
--node-volume-size=30 --vpc-public-subnets "$PubSubnet1,$PubSubnet2" --version 1.31 --ssh-access --external-dns-access --verbose 4**
...
023-04-23 01:32:22 [▶]  setting current-context to admin@myeks.ap-northeast-2.eksctl.io
2023-04-23 01:32:22 [✔]  **saved kubeconfig as "/root/.kube/config"**
...
```

설치 로그확인

![](/img/eks-002.png)

![](/img/eks-003.png)



접속확인

```bash
# 노드 IP 확인 및 PrivateIP 변수 지정
aws ec2 describe-instances --query "Reservations[*].Instances[*].{PublicIPAdd:PublicIpAddress,PrivateIPAdd:PrivateIpAddress,InstanceName:Tags[?Key=='Name']|[0].Value,Status:State.Name}" --filters Name=instance-state-name,Values=running --output table
kubectl get node --label-columns=topology.kubernetes.io/zone
kubectl get node --label-columns=topology.kubernetes.io/zone --selector=topology.kubernetes.io/zone=ap-northeast-2a
kubectl get node --label-columns=topology.kubernetes.io/zone --selector=topology.kubernetes.io/zone=ap-northeast-2c
**N1=$(**kubectl get node --label-columns=topology.kubernetes.io/zone --selector=topology.kubernetes.io/zone=ap-northeast-2a -o jsonpath={.items[0].status.addresses[0].address})
**N2=$**(kubectl get node --label-columns=topology.kubernetes.io/zone --selector=topology.kubernetes.io/zone=ap-northeast-2c -o jsonpath={.items[0].status.addresses[0].address})
****echo $N1, $N2
echo "export N1=$N1" >> /etc/profile
echo "export N2=$N2" >> /etc/profile

# eksctl-host 에서 노드의IP나 coredns 파드IP로 ping 테스트
ping <IP>
ping -c 1 $N1
ping -c 1 $N2

# 노드 보안그룹 ID 확인
aws ec2 describe-security-groups --filters Name=group-name,Values=*nodegroup* --query "SecurityGroups[*].[GroupId]" --output text
NGSGID=$(aws ec2 describe-security-groups --filters Name=group-name,Values=*nodegroup* --query "SecurityGroups[*].[GroupId]" --output text)
echo $NGSGID
echo "export NGSGID=$NGSGID" >> /etc/profile

# 노드 보안그룹에 eksctl-host 에서 노드(파드)에 접속 가능하게 룰(Rule) 추가 설정
aws ec2 authorize-security-group-ingress --group-id $NGSGID --protocol '-1' --cidr **192.168.1.100/32**

# eksctl-host 에서 노드의IP나 coredns 파드IP로 ping 테스트
ping -c 2 $N1
ping -c 2 $N2

# 워커 노드 SSH 접속
**ssh -i ~/.ssh/id_rsa -o StrictHostKeyChecking=no ec2-user@$N1 hostname
ssh -i ~/.ssh/id_rsa -o StrictHostKeyChecking=no ec2-user@$N2 hostname**
**ssh ec2-user@$N1
exit**
**ssh ec2-user@$N2
exit**
```



## AWS API 서버 엔드포인트 엑세스

![](/img/eks-004.png)

### public

~~~sh
while true; do dig +short $APIDNS ; echo "------------------------------" ; date; sleep 1; done
------------------------------
2025. 02. 09. (일) 01:40:42 KST
15.164.132.1
3.38.114.154
~~~

kubelet과 kube-proxy의 엔드포인트는 공인아이피로 되어있다.
(즉, 컨트롤 플레인과 노드 통신(kubelet,  kube-proxy)이 일어날 때 항상 공인아이피로 송수신된다는 것.)

~~~sh
while true; do ssh ec2-user@$N1 sudo ss -tnp | egrep 'kubelet|kube-proxy' ; echo ; ssh ec2-user@$N2 sudo ss -tnp | egrep 'kubelet|kube-proxy' ; echo "------------------------------" ; date; sleep 1; done

2025. 02. 09. (일) 01:40:59 KST
ESTAB 0      0               192.168.1.170:36268           15.164.132.1:443   users:(("kubelet",pid=2894,fd=26))
ESTAB 0      0               192.168.1.170:55092           3.38.114.154:443   users:(("kube-proxy",pid=3113,fd=9))
ESTAB 0      0      [::ffff:192.168.1.170]:10250 [::ffff:192.168.2.122]:47200 users:(("kubelet",pid=2894,fd=12))
ESTAB 0      0      [::ffff:192.168.1.170]:10250 [::ffff:192.168.2.156]:52132 users:(("kubelet",pid=2894,fd=20))

ESTAB 0      0               192.168.2.244:34406           3.38.114.154:443   users:(("kube-proxy",pid=3114,fd=9))
ESTAB 0      0               192.168.2.244:53726           15.164.132.1:443   users:(("kubelet",pid=2892,fd=26))
ESTAB 0      0      [::ffff:192.168.2.244]:10250 [::ffff:192.168.2.156]:37720 users:(("kubelet",pid=2892,fd=24))
ESTAB 0      0      [::ffff:192.168.2.244]:10250 [::ffff:192.168.2.122]:48800 users:(("kubelet",pid=2892,fd=12))
------------------------------
~~~



### public & private

![](/img/eks-005.png)

~~~sh
aws eks update-cluster-config --region $AWS_DEFAULT_REGION --name $CLUSTER_NAME --resources-vpc-config endpointPublicAccess=true,publicAccessCidrs="$(curl -s ipinfo.io/ip)/32",endpointPrivateAccess=true
{
    "update": {
        "id": "7e6c1d1e-0020-34a9-90f7-645628f32752",
        "status": "InProgress",
        "type": "EndpointAccessUpdate",
        "params": [
            {
                "type": "EndpointPublicAccess",
                "value": "true"
            },
            {
                "type": "EndpointPrivateAccess",
                "value": "true"
            },
            {
                "type": "PublicAccessCidrs",
                "value": "[\"13.125.166.72/32\"]"
            }
        ],
        "createdAt": "2025-02-09T01:40:33.081000+09:00",
        "errors": []
    }
}
~~~



~~~sh
while true; do dig +short $APIDNS ; echo "------------------------------" ; date; sleep 1; done
------------------------------
2025. 02. 09. (일) 01:40:42 KST
15.164.132.1
3.38.114.154
~~~

kubelet과 kube-proxy의 엔드포인트는 공인아이피로 되어있다.
(즉, 컨트롤 플레인과 노드 통신(kubelet,  kube-proxy)이 일어날 때 항상 공인아이피로 송수신된다는 것.)

~~~sh
while true; do ssh ec2-user@$N1 sudo ss -tnp | egrep 'kubelet|kube-proxy' ; echo ; ssh ec2-user@$N2 sudo ss -tnp | egrep 'kubelet|kube-proxy' ; echo "------------------------------" ; date; sleep 1; done

2025. 02. 09. (일) 01:40:59 KST
ESTAB 0      0               192.168.1.170:36268           15.164.132.1:443   users:(("kubelet",pid=2894,fd=26))
ESTAB 0      0               192.168.1.170:55092           3.38.114.154:443   users:(("kube-proxy",pid=3113,fd=9))
ESTAB 0      0      [::ffff:192.168.1.170]:10250 [::ffff:192.168.2.122]:47200 users:(("kubelet",pid=2894,fd=12))
ESTAB 0      0      [::ffff:192.168.1.170]:10250 [::ffff:192.168.2.156]:52132 users:(("kubelet",pid=2894,fd=20))

ESTAB 0      0               192.168.2.244:34406           3.38.114.154:443   users:(("kube-proxy",pid=3114,fd=9))
ESTAB 0      0               192.168.2.244:53726           15.164.132.1:443   users:(("kubelet",pid=2892,fd=26))
ESTAB 0      0      [::ffff:192.168.2.244]:10250 [::ffff:192.168.2.156]:37720 users:(("kubelet",pid=2892,fd=24))
ESTAB 0      0      [::ffff:192.168.2.244]:10250 [::ffff:192.168.2.122]:48800 users:(("kubelet",pid=2892,fd=12))
------------------------------
~~~

위 결과 public & private 으로 변경되는데, 베스천에서 kubctl 명령어가 더이상 실행되지 않는다.
왜냐하면 EKS owned ENI 형태로 관리되는 ENI의 시큐리티그룹에 베스천에 대한 인바운드 설정을 해놓지 않았기에 통신이 되지 않는다. (아래와 같이 처리하면 됨)

ControlPlaneSecurityGroup을 찾고, 베스천 Ip 인바운드를 Expose한다

~~~sh
CPSGID=$(aws ec2 describe-security-groups --filters Name=group-name,Values=*ControlPlaneSecurityGroup* --query "SecurityGroups[*].[GroupId]" --output text)
~~~

~~~sh
aws ec2 authorize-security-group-ingress --group-id $CPSGID --protocol '-1' --cidr 192.168.1.100/32

{
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-0e61ded354b5415b0",
            "GroupId": "sg-0e301754ec8ac3212",
            "GroupOwnerId": "738612635754",
            "IsEgress": false,
            "IpProtocol": "-1",
            "FromPort": -1,
            "ToPort": -1,
            "CidrIpv4": "192.168.1.100/32",
            "SecurityGroupRuleArn": "arn:aws:ec2:ap-northeast-2:738612635754:security-group-rule/sgr-0e61ded354b5415b0"
        }
    ]
}
~~~

- dig +short $APIDNS 
- dig 결과가 컨트롤 플레인에서 관리하는 사설아이피가 조회되기 시작한다.
- 이는, 데이터플레인과의 통신이 aws 내부적으로 일어난다는 의미

~~~sh
------------------------------
2025. 02. 09. (일) 01:56:05 KST
192.168.1.152
192.168.2.82
~~~

- node dns 역시 internal로 변경됨

~~~sh
kubectl get node -v=6
I0209 01:58:11.886004   18989 loader.go:395] Config loaded from file:  /root/.kube/config
I0209 01:58:12.857831   18989 round_trippers.go:553] GET https://4049728AFEB8F9CEA17AA2E19BD71657.gr7.ap-northeast-2.eks.amazonaws.com/api/v1/nodes?limit=500 200 OK in 961 milliseconds
NAME                                               STATUS   ROLES    AGE   VERSION
ip-192-168-1-170.ap-northeast-2.compute.internal   Ready    <none>   99m   v1.31.4-eks-aeac579
ip-192-168-2-244.ap-northeast-2.compute.internal   Ready    <none>   99m   v1.31.4-eks-aeac579
~~~

- 실제 통신은, 아래와같이 재실행 이후 적용됨을 확인

~~~sh
# 모니터링 : tcp peer 정보 변화 확인
while true; do ssh ec2-user@$N1 sudo ss -tnp | egrep 'kubelet|kube-proxy' ; echo ; ssh ec2-user@$N2 sudo ss -tnp | egrep 'kubelet|kube-proxy' ; echo "------------------------------" ; date; sleep 1; done

# kube-proxy rollout
kubectl rollout restart ds/kube-proxy -n kube-system
~~~

~~~sh
while true; do ssh ec2-user@$N1 sudo ss -tnp | egrep 'kubelet|kube-proxy' ; echo ; ssh ec2-user@$N2 sudo ss -tnp | egrep 'kubelet|kube-proxy' ; echo "------------------------------" ; date; sleep 1; done

ESTAB 0      0      192.168.1.170:44346  192.168.2.82:443   users:(("kubelet",pid=60428,fd=9))
ESTAB 0      0      192.168.1.170:57082 192.168.1.152:443   users:(("kube-proxy",pid=60264,fd=9))

ESTAB 0      0      192.168.2.244:51864 192.168.1.152:443   users:(("kube-proxy",pid=59674,fd=9))
ESTAB 0      0      192.168.2.244:54588  192.168.2.82:443   users:(("kubelet",pid=59859,fd=12))
~~~



### 허용아이피를 삭제하면 ?

- 당연히 kubectl api 통신은 된다. 애초에 데이터플레인 이 있는 서브넷에 속해있기 때문에 EKS owned ENI 을 통해 통신하고 있었기에, 허용아이피와는 무관하다. (private 모드에서 통신 가능)

- 다른 작업공간에서 kubectl 통신이 필요하다면, aws 자격증명 후에
  `aws eks update-kubeconfig --region ap-northeast-2 --name myeks` 와 같이 eks 접근정보를 작성하면
  통신이 가능하다.

  ~~~sh
  ➜  .kube kubectl get nodes
  NAME                                               STATUS   ROLES    AGE    VERSION
  ip-192-168-1-170.ap-northeast-2.compute.internal   Ready    <none>   136m   v1.31.4-eks-aeac579
  ip-192-168-2-244.ap-northeast-2.compute.internal   Ready    <none>   136m   v1.31.4-eks-aeac579
  ~~~

### private

- EKS 클러스터의 API 엔드포인트가 사설 IP(프라이빗 IP)만 사용하도록 설정됨
- EKS가 관리하는 ENI(Elastic Network Interface)에 사설 IP가 할당되어, 해당 EKS Control Plane이 위치한 서브넷 내에서만 접근이 가능
- VPC 내부나 해당 VPC와 연결된 네트워크(예: VPN, Direct Connect)에서만 API 서버로 통신할 수 있다.

![](/img/eks-006.png)

~~~sh
aws eks update-cluster-config \
  --region $AWS_DEFAULT_REGION \
  --name $CLUSTER_NAME \
  --resources-vpc-config endpointPublicAccess=false,endpointPrivateAccess=true
  
  {
    "update": {
        "id": "b7526d2d-3009-3f5e-ab64-2c255053dcf4",
        "status": "InProgress",
        "type": "EndpointAccessUpdate",
        "params": [
            {
                "type": "EndpointPublicAccess",
                "value": "false"
            },
            {
                "type": "EndpointPrivateAccess",
                "value": "true"
            },
            {
                "type": "PublicAccessCidrs",
                "value": "[\"13.125.166.72/32\"]"
            }
        ],
        "createdAt": "2025-02-09T02:03:37.283000+09:00",
        "errors": []
    }
}
~~~

당연하게도 public & private 모드에서 설정했던 로컬 작업Pc에서의 kubectl 통신은 불가하다. (proxy server는 사설아이피 대역으로 엔드포인트를 제공하기 때문)
~~~
# kubectl get nodes
Unable to connect to the server: dial tcp 192.168.2.82:443: i/o timeout
~~~


