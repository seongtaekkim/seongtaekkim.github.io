

~~~ groovy

def BRANCH = 'develop'
def filename = null
def tomcat_home = "/usr/local/tomcat_dev"
def dev0 = [:]
dev0.name = "emroCloud-dev-biz-0 WAS"
dev0.host = "cloud.emro.co.kr"
dev0.port = 1080
dev0.user = "ec2-user"
dev0.allowAnyHosts = true
dev0.fileTransfer = 'scp'
def dev1 = [:]
dev1.name = "emroCloud-dev-biz-1 WAS"
dev1.host = "cloud.emro.co.kr"
dev1.port = 1081
dev1.user = "ec2-user"
dev1.allowAnyHosts = true
dev1.fileTransfer = 'scp'

pipeline {
    agent {
        node {
            label "master"
        }
    }
    
    tools {
        maven "Maven3.9"
        jdk "JDK8"
    }
    
    options {
        buildDiscarder logRotator( 
            numToKeepStr: '7'
        )
    }

    stages {
        stage('Git Clone') {
            steps {
                // Get some code from a GitHub repository
                dir('cloud-parent') {
                    git branch: "${BRANCH}", credentialsId: 'cloud-git-credential', url: 'http://gitea:3000/Cloud-Division/cloud-parent.git'
                }
                dir('cloud-common') {
                    git branch: "${BRANCH}", credentialsId: 'cloud-git-credential', url: 'http://gitea:3000/Cloud-Division/cloud-common.git'
                }
                dir('cloud-supplier') {
                    git branch: "${BRANCH}", credentialsId: 'cloud-git-credential', url: 'http://gitea:3000/Cloud-Division/cloud-supplier.git'
                }
                dir('cloud-buyer') {
                    git branch: "${BRANCH}", credentialsId: 'cloud-git-credential', url: 'http://gitea:3000/Cloud-Division/cloud-buyer.git'
                }
                dir('cloud-webapp') {
                    git branch: "${BRANCH}", credentialsId: 'cloud-git-credential', url: 'http://gitea:3000/Cloud-Division/cloud-webapp.git'
                }
            }
        } // stage end 
        
        stage('Build ec2 war') {
            steps {
                echo "Workspace directory : ${env.WORKSPACE}"
                
                // Run Maven on a Unix agent.
                dir('cloud-parent') {
                    sh "mvn -Dmaven.test.failure.ignore=true -D https.protocols=TLSv1.2 -P deploy -P dev clean install"
                }
            }
            
            post {
                // If Maven was able to run the tests, even if some of the test
                // failed, record the test results and archive the jar file.
                success {
                    script {
                        // war 파일명 확인
                        filename = sh(script: "ls cloud-webapp/target/*.war | head -1", returnStdout: true).trim()
                    }
                    // stash(name: "ROOT.war", includes: 'cloud-webapp/target/*.war')
                    archiveArtifacts "${filename}"
                }
            }
        } // stage end 
        
        stage('Deploy dev0') {
            steps {
                withCredentials([aws(credentialsId: 'aws-emrocloud-dev')]) {
                    
                    // 1. aws target group 해제
                    echo "################################ AWS deregister..."
                    sh 'aws elbv2 deregister-targets --target-group-arn arn:aws:elasticloadbalancing:ap-northeast-2:664127938624:targetgroup/emrocloud-dev-was-8080-http2/fe167627e46d0e70 --targets Id=10.100.2.100,Port=8080 --region ap-northeast-2'
                    
                }
                
                withCredentials([sshUserPrivateKey(credentialsId: 'emroCloud-dev-ssh-key', keyFileVariable: 'keyFile')]) {
                    script {
                        // 2. SSH 접속을 위한 keyFile 위치를 withCredentials로 받아와 변수에 삽입 후 사용
                        dev0.identityFile = keyFile
                        
                        // 3. tomcat 중지
                        echo "################################ Tomcat Stop..."
                        // $에 대한 escape 문제로 tomcat_home을 사용할 수 없음 
                        //sshCommand remote: dev0, command: 'sudo su - was -c "ps -ef | grep tomcat_dev | grep java | grep -v grep | awk \'{print \\$2}\'"'
                        //sshCommand remote: dev0, command: 'sudo su - was -c "sleep 30; /usr/local/tomcat_dev/bin/shutdown.sh; sleep 30; ps -ef | grep tomcat_dev | grep java | grep -v grep | awk \'{print \\$2}\' | xargs kill -9; rm -rf /usr/local/tomcat_dev/webapps/ROOT*"'
                        
                        def command0 = '''
                            sudo su - was -c '
                                sleep 30;
                                sudo systemctl stop tomcat_dev.service;
                                sleep 30;
                                ps -ef | grep tomcat_dev | grep java | grep -v grep | awk "{print \$2}" | xargs kill -9;
                                rm -rf /usr/local/tomcat_dev/webapps/ROOT*
                            '
                        '''
                        
                        sshCommand remote: dev0, command: command0
                        
                        // 4. war 파일 전송
                        echo "################################ War Transfer..."
                        //sshPut remote: dev0, from: "${filename}", into: "${tomcat_home}/webapps/scptest.txt"
                        sshPut remote: dev0, from: "${filename}", into: "${tomcat_home}/webapps/ROOT.war"
                        
                        // 5. tomcat 시작
                        echo "################################ Tomcat Start..."
                        sshCommand remote: dev0, command: "sudo su - was -c 'sudo systemctl start tomcat_dev.service'"
                        
                    }
                }
                
                withCredentials([aws(credentialsId: 'aws-emrocloud-dev')]) {
                    
                    // 6. aws target group 등록
                    echo "################################ AWS register..."
                    sh 'aws elbv2 register-targets --target-group-arn arn:aws:elasticloadbalancing:ap-northeast-2:664127938624:targetgroup/emrocloud-dev-was-8080-http2/fe167627e46d0e70 --targets Id=10.100.2.100,Port=8080 --region ap-northeast-2'
                    
                    // 7. HealthCheck
                    echo "################################ WAS HealthCheck..."
                    sh '''
                    cnt=0
                    while [ $cnt -lt 20 ]
                    do
                      OUTPUT=$(aws elbv2 describe-target-health --target-group-arn arn:aws:elasticloadbalancing:ap-northeast-2:664127938624:targetgroup/emrocloud-dev-was-8080-http2/fe167627e46d0e70 --targets Id=10.100.2.100,Port=8080 --output text --query 'TargetHealthDescriptions[0].TargetHealth' --region ap-northeast-2)
                      # echo $OUTPUT
                      if [ "$OUTPUT" = "healthy" ]
                      then
                        exit 0
                      fi
                      sleep 30
                      cnt=$(expr $cnt + 1)
                    done
                    exit 1
                    '''
                }
            }
        } // stage end 
        
        stage('Deploy dev1') {
            steps {
                withCredentials([aws(credentialsId: 'aws-emrocloud-dev')]) {
                    
                    // 1. aws target group 해제
                    echo "################################ AWS deregister..."
                    sh 'aws elbv2 deregister-targets --target-group-arn arn:aws:elasticloadbalancing:ap-northeast-2:664127938624:targetgroup/emrocloud-dev-was-8080-http2/fe167627e46d0e70 --targets Id=10.100.3.100,Port=8080 --region ap-northeast-2'
                    
                }
                
                withCredentials([sshUserPrivateKey(credentialsId: 'emroCloud-dev-ssh-key', keyFileVariable: 'keyFile')]) {
                    script {
                        // 2. SSH 접속을 위한 keyFile 위치를 withCredentials로 받아와 변수에 삽입 후 사용
                        dev1.identityFile = keyFile
                        
                        // 3. tomcat 중지
                        echo "################################ Tomcat Stop..."
                        //sshCommand remote: dev1, command: 'sudo su - was -c "ps -ef | grep tomcat_dev | grep java | grep -v grep | awk \'{print \\$2}\'"'
                        //sshCommand remote: dev1, command: 'sudo su - was -c "sleep 30; /usr/local/tomcat_dev/bin/shutdown.sh; sleep 30; ps -ef | grep tomcat_dev | grep java | grep -v grep | awk \'{print \\$2}\' | xargs kill -9; rm -rf /usr/local/tomcat_dev/webapps/ROOT*"'
                        
                        def command1 = '''
                            sudo su - was -c '
                                sleep 30;
                                sudo systemctl stop tomcat_dev.service;
                                sleep 30;
                                ps -ef | grep tomcat_dev | grep java | grep -v grep | awk "{print \$2}" | xargs kill -9;
                                rm -rf /usr/local/tomcat_dev/webapps/ROOT*
                            '
                        '''
                        
                        sshCommand remote: dev1, command: command1
                        
                        // 4. war 파일 전송
                        echo "################################ War Transfer..."
                        //sshPut remote: dev1, from: "${filename}", into: "${tomcat_home}/webapps/scptest.txt"
                        sshPut remote: dev1, from: "${filename}", into: "${tomcat_home}/webapps/ROOT.war"
                        
                        // 5. tomcat 시작
                        echo "################################ Tomcat Start..."
                        sshCommand remote: dev1, command: "sudo su - was -c 'sudo systemctl start tomcat_dev.service'"
                        
                    }
                }
                
                withCredentials([aws(credentialsId: 'aws-emrocloud-dev')]) {
                    
                    // 6. aws target group 등록
                    echo "################################ AWS register..."
                    sh 'aws elbv2 register-targets --target-group-arn arn:aws:elasticloadbalancing:ap-northeast-2:664127938624:targetgroup/emrocloud-dev-was-8080-http2/fe167627e46d0e70 --targets Id=10.100.3.100,Port=8080 --region ap-northeast-2'
                    
                    // 7. HealthCheck
                    echo "################################ WAS HealthCheck..."
                    sh '''
                    cnt=0
                    while [ $cnt -lt 20 ]
                    do
                      OUTPUT=$(aws elbv2 describe-target-health --target-group-arn arn:aws:elasticloadbalancing:ap-northeast-2:664127938624:targetgroup/emrocloud-dev-was-8080-http2/fe167627e46d0e70 --targets Id=10.100.3.100,Port=8080 --output text --query 'TargetHealthDescriptions[0].TargetHealth' --region ap-northeast-2)
                      # echo $OUTPUT
                      if [ "$OUTPUT" = "healthy" ]
                      then
                        exit 0
                      fi
                      sleep 30
                      cnt=$(expr $cnt + 1)
                    done
                    exit 1
                    '''
                }
            } // step end 
        } // stage end 
    } // stages end 
}

~~~