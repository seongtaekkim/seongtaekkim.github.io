

emrocloud 7개 서버에 설정되어있는 datadog agent를 걷어태고
그 위치에 whatap을 설정한다.

추가적으로 emroCloud IF 인스턴스에도 추가하도록 한다.



## emroCloud-prod ec2 BIZ
- setenv.sh 파일삭제
- datadog agent 중지(?)
- whatap agent 다운로드 & 파일 설정
- whatap.conf & security.conf 설정

~~~ sh



~~~



## emroCloud-prod ec2 IF
- whatap agent 다운로드 & 파일 설정
- whatap.conf & security.conf 설정
~~~ sh



~~~



dns 자동화
젠킨스 코드





## emroCloud-dev bastion



## emroCloud-dev web




# emroCloud-prod worker-node 2ea



## emroCloud-prod infra-node 2ea










image pull
dockerfile 생성

catalina.sh 에 삽입
image build 
image push



~~~
aws ecr get-login-password --region ap-northeast-2 | docker login --username AWS --password-stdin 720737318830.dkr.ecr.ap-northeast-2.amazonaws.com

docker build -t tomcat .


docker tag tomcat:latest 720737318830.dkr.ecr.ap-northeast-2.amazonaws.com/tomcat:latest


docker push 720737318830.dkr.ecr.ap-northeast-2.amazonaws.com/tomcat:latest

~~~



~~~

#!/bin/sh

WHATAP_HOME=/usr/local/emrocloud/whatap  
WHATAP_JAR=`ls ${WHATAP_HOME}/whatap.agent-*.jar | sort -V | tail -1`  
JAVA_OPTS="${JAVA_OPTS} -javaagent:${WHATAP_JAR} "


security 추가
뭐또 추가 


${WHATAP_HOME}/whatap.conf

actx_meter_enabled=true
hook_method_patterns=smartsuite.app.*,smartsuite.app.*.*,smartsuite.app.*.*.*,smartsuite.app.*.*.*.*,smartsuite.app.*.*.*.*.*,smartsuite.app.*.*.*.*.*.*,smartsuite.app.*.*.*.*.*.*.*
httpc_host_meter_enabled=true
license=x605gkg867b38-x3sk1nma6qce5s-x5cl36s182n143
oshi_enabled=true
oshi_netstat_enabled=true
profile_http_header_enabled=true
profile_http_parameter_enabled=true
profile_sql_param_enabled=true
profile_sql_resource_enabled=true
sql_dbc_meter_enabled=true
tx_caller_meter_enabled=true
whatap.apdex_time=5000
whatap.server.host=13.124.11.223/13.209.172.35


security.conf

paramkey=ABCDEF
threadkill=ABCDEF
~                       

~~~


~~~ sh

RUN set -eux; \
    sed -i '1 a\
WHATAP_HOME=/usr/local/emrocloud/whatap\n\
WHATAP_JAR=$(ls ${WHATAP_HOME}/whatap.agent-*.jar | sort -V | tail -1)\n\
JAVA_OPTS="${JAVA_OPTS} -javaagent:${WHATAP_JAR}"' \
    /usr/local/emrocloud/tomcat/catalina.sh
    

~~~


~~~

https://archive.apache.org/dist/tomcat/tomcat-8/v8.5.82/bin/apache-tomcat-8.5.82.tar.gz


~~~