

### S3 분석

~~~ sh


aws s3 cp s3://emrocloud-prod-access-alb/AWSLogs/720737318830/elasticloadbalancing/ap-northeast-2/2025/07/11/ . --recursive


aws s3 cp s3://emrocloud-prod-eks-elb-logs/pub-alb-service/AWSLogs/720737318830/elasticloadbalancing/ap-northeast-2/2025/07/10/ . --recursive


 zcat *.log.gz | awk -F' ' '$9 == "404" {
    split($0,f,"\"");
    split($4,ip,":");
    printf "IP: %s\tTime: %s\tRequest: %s\tResponse: %s\n", ip[1], $2, f[2], $9
}' | sort > result



zcat *.log.gz | awk -F' ' '{
    split($0,f,"\"");
    split($4,ip,":");
    printf "IP: %s\tTime: %s\tRequest: %s\tResponse: %s\n", ip[1], $2, f[2], $9
}' | sort > result2

~~~





| 과금 항목                                                 | 가격                                                       | 자세한 내용                                                                                                                                                                        |     |
| ----------------------------------------------------- | -------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --- |
| **Web ACL** 생성                                        | **$5.00 / 월** (시간 비례)                                    | 배포(ALB, API Gateway, AppSync, CloudFront 등)마다 1 개 Web ACL만 연결 가능 ([Amazon Web Services](https://aws.amazon.com/waf/pricing/ "Pricing - AWS WAF - Amazon Web Services (AWS)")) |     |
| **Rule 추가**∙ 개별 Rule∙ Rule Group / Managed Rule Group | **$1.00 / 월** (Rule 또는 그룹 1 개당, 시간 비례)                   | 예: 커스텀 8 개 + AWS Managed 1 개면 9 달러/월 ([Amazon Web Services](https://aws.amazon.com/waf/pricing/ "Pricing - AWS WAF - Amazon Web Services (AWS)"))                             |     |
| **요청 처리량**                                            | **$0.60 / 백만 Request**                                   | Web ACL이 검사한 실제 요청 수 기준 (서울 리전도 동일 단가) ([Amazon Web Services](https://aws.amazon.com/waf/pricing/ "Pricing - AWS WAF - Amazon Web Services (AWS)"))                           |     |
| 추가 WCU 사용량                                            | **$0.20 / 백만 Req (추가 500 WCU마다)**                        | 기본 1,500 WCU 초과 시 과금 ([Amazon Web Services](https://aws.amazon.com/waf/pricing/ "Pricing - AWS WAF - Amazon Web Services (AWS)"))                                             |     |
| 추가 Body 사이즈 검사                                        | **$0.30 / 백만 Req (추가 16 KB마다)**                          | 기본 16 KB 초과 본문 검사 시 과금 ([Amazon Web Services](https://aws.amazon.com/waf/pricing/ "Pricing - AWS WAF - Amazon Web Services (AWS)"))                                           |     |
| **CAPTCHA / Challenge**                               | CAPTCHA : **$4 / 10 K 시도**Challenge : **$0.40 / 1 M 응답** | CAP 시도·Challenge 응답마다 별도 과금 ([Amazon Web Services](https://aws.amazon.com/waf/pricing/?utm_source=chatgpt.com "Pricing - AWS WAF - Amazon Web Services (AWS)"))               |     |
| **Bot Control AMR**                                   | **$10 / 월 + 요청당 $1.00 / 백만**(Common : 첫 10 M 무료)         | 악성 봇 탐지용 AWS Managed Rules ([Amazon Web Services](https://aws.amazon.com/waf/pricing/ "Pricing - AWS WAF - Amazon Web Services (AWS)"))                                       |     |
| **Fraud Control AMR**(Account Takeover / Creation)    | **$10 / 월 + 요청별 차등 단가**                                  | 로그인·회원가입 보호용, 단계별 요율 적용 ([Amazon Web Services](https://aws.amazon.com/waf/pricing/ "Pricing - AWS WAF - Amazon Web Services (AWS)"))                                          |     |



~~~ sh

# 1) Web ACL 생성
aws wafv2 create-web-acl \
  --region ap-northeast-2 \
  --scope REGIONAL \
  --cli-input-json file://waf-alb-captcha.json

# ! 출력 결과에서 WebAcl.Arn 값을 복사해 둡니다.
WEB_ACL_ARN="arn:aws:wafv2:ap-northeast-2:123456789012:regional/webacl/alb-rate-limit-captcha/xxxxxxxx"

# 2) ALB에 연결
LB_ARN="arn:aws:elasticloadbalancing:ap-northeast-2:123456789012:loadbalancer/app/my-alb/50dc6c495c0c9188"

aws wafv2 associate-web-acl \
  --region ap-northeast-2 \
  --web-acl-arn "$WEB_ACL_ARN" \
  --resource-arn "$LB_ARN"



~~~

~~~

@도메인
안녕하세요
며칠 전 발생했던 웹에 대한 비정상 트래픽을 탐지하고 조치하기 위해
개발aws에 WAF rule 적용 및 테스트를 진행하려 합니다.

테스트 할 룰 내용은 아래와 같습니다.
- 특정 공인IP가 특정 도메인을 1분 60회 이상 호출하면 captcha가 발생
- devif.emrocloud.com 요청은 제외

하루정도 테스트를 위해 날짜를 조율하고 싶습니다만,
금일 진행해도 되는 지, 다른 날로 협의해야 할 지 판단을 요청 드립니다.
감사합니다.

** 영향도는 개발 비지니스 url 접속 과다 시(1분 60회이상) captcha가 발생하는 정도입니다.
~~~


~~~ sh

emrocloud-dev-web-alb

~~~