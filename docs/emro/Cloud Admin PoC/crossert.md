

jira에 올라와있는 NPKI와 CrosscertLIB64 를 서버에 복사
~~~
# local
scp -P 24477 .\CrossCertLIB64.tar stk.kim@192.168.6.113:~/
scp -P 24477 .\CCLicense.key stk.kim@192.168.6.113:~/

# linux server
tar -xvf CrossCertLIB64.tar

cp -R CrossCertLIB64 NPKI /usr/local/
~~~


해당 파일들을 톰캣을 실행하는 계정으로 맞추어준다
~~~
chown -R was:was /usr/local/NPKI
chown -R was:was /usr/local/CrossCertLIB64
~~~

동적라이브러리들은 실행 가능 하게 설정
~~~
cd /usr/local/CrossCertLIB64
chmod +x *.so*
~~~


bash_profile에 환경설정 등록
~~~

if [ -f ~/.bashrc ]; then
        . ~/.bashrc
fi


LD_LIBRARY_PATH=/usr/local/CrossCertLIB64

export LD_LIBRARY_PATH
    
~~~






## 동적라이브러리 의존성문제

rocky 9.5 버전에서 특정 so 파일이 요구하는 라이브러리가 없음이 확인됨.

![[Pasted image 20250718151726.png]]


![[Pasted image 20250718151706.png]]

rocky 8.10 버전에서는 존재하는데, 상위버전이라 판단되는 so파일로 링크를 걸어주었음.
![[Pasted image 20250718151902.png]]



링크를 걸어 아래와 같이 확인이 가능함.
~~~
# root권한
cd /usr/local/CrossCertLIB64
ln -s /usr/lib64/liblber.so.2.0.200 liblber-2.4.so.2
ln -s /usr/lib64/libsasl2.so.3 libsasl2.so.2
# unlink /usr/local/CrossCertLIB64/libsasl2.so.2
~~~
![[Pasted image 20250718152036.png]]

ldd로 의존성을 확인하였고고
![[Pasted image 20250718152124.png]]

프로그램 동작에 이상 없음을 확인함.
~~~
java -classpath .:crosscert_2.2.jar jCert
~~~
![[Pasted image 20250718152147.png]]


#### 방화벽 신청


~~~

1. 외부 접속 테스트(공인인증기관 LDAP 서버)
1.1. 텔넷 으로 아래와 같이 도메인으로 접속 테스트
telnet dir.crosscert.com 389
telnet ldap.signgate.com 389
telnet dir.signkorea.com 389
telnet ds.yessign.or.kr 389
telnet ldap.tradesign.net 389 

1.2 도메인으로 접속이 안될 때 IP로 접속 테스트
telnet 211.192.169.180 389
telnet 211.35.96.26 389
telnet 210.207.195.77 389
telnet 203.233.91.35 389
telnet 203.242.205.156 389 

~~~


