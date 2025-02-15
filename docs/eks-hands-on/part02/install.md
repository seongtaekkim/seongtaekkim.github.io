---
layout: default
title: aws에서 eks 구성
parent: eks network
grand_parent: eks 실습
nav_order: 1
---



~~~sh
# yaml 파일 다운로드
curl -O https://s3.ap-northeast-2.amazonaws.com/cloudformation.cloudneta.net/K8S/myeks-2week.yaml

# 배포
aws cloudformation deploy --template-file ./myeks.yaml \
     --stack-name myeks --parameter-overrides KeyName=container SgIngressSshCidr=$(curl -s ipinfo.io/ip)/32 --region ap-northeast-2


# CloudFormation 스택 배포 완료 후 운영서버 EC2 IP 출력
aws cloudformation describe-stacks --stack-name myeks --query 'Stacks[*].Outputs[*].OutputValue' --output text

# 운영서버 EC2 에 SSH 접속
ssh -i <ssh 키파일> ec2-user@$(aws cloudformation describe-stacks --stack-name myeks --query 'Stacks[*].Outputs[0].OutputValue' --output text)
~~~





### eks deploy

~~~sh
#
export CLUSTER_NAME=myeks

# myeks-VPC/Subnet 정보 확인 및 변수 지정
export VPCID=$(aws ec2 describe-vpcs --filters "Name=tag:Name,Values=$CLUSTER_NAME-VPC" --query 'Vpcs[*].VpcId' --output text)
echo $VPCID

export PubSubnet1=$(aws ec2 describe-subnets --filters Name=tag:Name,Values="$CLUSTER_NAME-Vpc1PublicSubnet1" --query "Subnets[0].[SubnetId]" --output text)
export PubSubnet2=$(aws ec2 describe-subnets --filters Name=tag:Name,Values="$CLUSTER_NAME-Vpc1PublicSubnet2" --query "Subnets[0].[SubnetId]" --output text)
export PubSubnet3=$(aws ec2 describe-subnets --filters Name=tag:Name,Values="$CLUSTER_NAME-Vpc1PublicSubnet3" --query "Subnets[0].[SubnetId]" --output text)
echo $PubSubnet1 $PubSubnet2 $PubSubnet3


-------------------------------------------


# ssh 퍼블릭 키 경로 지정
SshPublic=~/.ssh/eks-demo.pub
echo $SshPublic
eksctl create cluster --name $CLUSTER_NAME --region=ap-northeast-2 --nodegroup-name=ng1 --node-type=t3.medium --nodes 3 --node-volume-size=30 --vpc-public-subnets "$PubSubnet1","$PubSubnet2","$PubSubnet3" --version 1.31 --with-oidc --external-dns-access --full-ecr-access --alb-ingress-access --node-ami-family AmazonLinux2023 --ssh-access --ssh-public-key $SshPublic --dry-run > myeks.yaml
~~~



배포

~~~sh
# kubeconfig 파일 경로 위치 지정 : 
export KUBECONFIG=$HOME/kubeconfig
혹은 각자 편한 경로 위치에 파일 지정
export KUBECONFIG=~/Downloads/kubeconfig

# 배포
eksctl create cluster -f myeks.yaml --verbose 4
~~~



### ec2 접속

~~~sh
ssh -i ~/.ssh/container.pem ec2-user@$(aws cloudformation describe-stacks --stack-name myeks --query 'Stacks[*].Outputs[0].OutputValue' --output text)
~~~



### 노드접속

~~~
export MNSGID=sg-077fac2b212f084ae
~~~

~~~
# 인스턴스 공인 IP 확인
aws ec2 describe-instances --query "Reservations[*].Instances[*].{InstanceID:InstanceId, PublicIPAdd:PublicIpAddress, PrivateIPAdd:PrivateIpAddress, InstanceName:Tags[?Key=='Name']|[0].Value, Status:State.Name}" --filters Name=instance-state-name,Values=running --output table

# 인스턴스 공인 IP 변수 지정
export N1=<az1 배치된 EC2 공인 IP>
export N2=<az2 배치된 EC2 공인 IP>
export N3=<az3 배치된 EC2 공인 IP>
export N1=43.203.169.0
export N2=13.125.28.29
export N3=13.125.255.7
echo $N1, $N2, $N3

# ping 테스트
ping -c 2 $N1
ping -c 2 $N2

# *nodegroup-ng1* 포함된 보안그룹 ID
export MNSGID=<각자 자신의 관리형 노드 그룹(EC2) 에 보안그룹 ID>
export MNSGID=sg-075e2e6178557c95a

# 해당 보안그룹 inbound 에 자신의 집 공인 IP 룰 추가
aws ec2 authorize-security-group-ingress --group-id $MNSGID --protocol '-1' --cidr $(curl -s ipinfo.io/ip)/32

# 해당 보안그룹 inbound 에 운영서버 내부 IP 룰 추가
aws ec2 authorize-security-group-ingress --group-id $MNSGID --protocol '-1' --cidr 172.20.1.100/32

# AWS EC2 관리 콘솔에서 EC2에 보안 그룹에 inbound rule 에 추가된 규칙 정보 확인


# ping 테스트
ping -c 2 $N1
ping -c 2 $N2

# 워커 노드 SSH 접속
ssh -i <SSH 키> -o StrictHostKeyChecking=no ec2-user@$N1 hostname
for i in $N1 $N2 $N3; do echo ">> node $i <<"; ssh -o StrictHostKeyChecking=no ec2-user@$i hostname; echo; done

ssh ec2-user@$N1
exit
ssh ec2-user@$N2
exit
ssh ec2-user@$N2
exit

------------------
# 운영서버 EC2에서 접속 시

## 인스턴스 공인 IP 변수 지정
export N1=<az1 배치된 EC2 내부 IP>
export N2=<az2 배치된 EC2 내부 IP>
export N3=<az3 배치된 EC2 내부 IP>
export N1=192.168.1.186
export N2=192.168.2.123
export N3=192.168.3.174
echo $N1, $N2, $N3

## ping 테스트
ping -c 2 $N1
ping -c 2 $N2
~~~









자원삭제

~~~sh
eksctl delete cluster --name $CLUSTER_NAME

aws cloudformation delete-stack --stack-name myeks
~~~

