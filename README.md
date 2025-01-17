## HyperFrame-Tomcat

업로드된 바이너리는 HyperFrame Tomcat 제품 설치를 위한 파일

## 설치 파일

### Tomcat

- Version : apache-tomcat-9.0.52
- Note : [https://tomcat.apache.org/download-90.cgi](https://tomcat.apache.org/download-90.cgi)

### Tomcat-Connectors

- Version : tomcat-connectors-1.2.48
- Note : [https://tomcat.apache.org/download-connectors.cgi](https://tomcat.apache.org/download-connectors.cgi)

### Tomcat 지원 Java Version

- JDK 8 이상 지원
- OpenJDK : [https://openjdk.java.net/](https://openjdk.java.net/)

## 검증 환경

- CentOS Linux release 7.9
- CentOS Linux release 8.4
- CentOS Stream release 8
- Ubuntu 20.04.1 LTS

## 설치 및 실행

### 1) Tomcat 압축 풀기

```
$ tar -zxf apache-tomcat-9.0.52.tar.gz

```

### 2) 디렉토리 구조 확인

```
$ cd ${TOMCAT_HOME}
    hftomcat
    ├── bin
    ├── conf
    ├── lib
    ├── logs
    ├── temp
    ├── webapps
    └── work

```

### 3) Tomcat 실행

```
$ cd ${TOMCAT_HOME}/bin/
$ ./startup.sh

```

### 4) Tomcat 종료

```
$ cd ${TOMCAT_HOME}/bin/
$ ./shutdown.sh

```

## 버전 확인 1

```
$ cd ${TOMCAT_HOME}/lib
$ java -cp catalina.jar org.apache.catalina.util.ServerInfo
Server version: Apache Tomcat/9.0.52
Server built:   Jul 31 2021 04:12:17 UTC
Server number:  9.0.52.0
OS Name:        Linux
OS Version:     2.6.32-754.11.1.el6.x86_64
Architecture:   amd64
JVM Version:    1.8.0_171-b11
JVM Vendor:     Oracle Corporation

```

## 버전 확인 2

```
$ cd ${TOMCAT_HOME}/bin
$ ./version.sh
Using CATALINA_BASE:   ${TOMCAT_HOME}
Using CATALINA_HOME:   ${TOMCAT_HOME}
Using CATALINA_TMPDIR: ${TOMCAT_HOME}/temp
Using JRE_HOME:
Using CLASSPATH:       ${TOMCAT_HOME}/bin/bootstrap.jar:${TOMCAT_HOME}/bin/tomcat-juli.jar
Using CATALINA_OPTS:
Server version: Apache Tomcat/9.0.52
Server built:   Jul 31 2021 04:12:17 UTC
Server number:  9.0.52.0
OS Name:        Linux
OS Version:     3.10.0-1160.36.2.el7.x86_64
Architecture:   amd64
JVM Version:    1.8.0_302-b08
JVM Vendor:     Red Hat, Inc.

```

## 환경 설정 파일 정보

### 1) 환경 설정 파일 경로

```
${TOMCAT_HOME}/conf/server.xml

```

### 2) 환경 설정

- Port 확인 및 변경
    
    ```
    ...
    <Service name="Catalina">
        Connector port="8080" protocol="HTTP/1.1"
                 connectionTimeout="20000"
                 redirectPort="8443" />
    ...
    </Service>
    
    ```
    
- 로그 정보
    
    ```
    서버상에서 발생한 모든 내용을 기록한 Log
    $ {TOMCAT_HOME}/logs/catalina.out
    
    Tomcat에서 발생하는 Log
    $ {TOMCAT_HOME}/logs/catalina.yyyy-mm-dd.log
    
    Tomcat Manager Web App Log
    $ {TOMCAT_HOME}/logs/manager.log
    
    ```
    
- 프로토콜 별 Connector 추가 및 삭제
    
    ```
    ...
    <Service name="Catalina">
        # http Connector
        <Connector port="8080" protocol="HTTP/1.1"
                   connectionTimeout="20000"
                   redirectPort="8443" />
        # https Connector
        <Connector port="8443" protocol="HTTP/1.1" SSLEnabled="true"
                   maxThreads="150" scheme="https" secure="true"
                   clientAuth="false" sslProtocol="TLS" />
        # ajp Connector
        <Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />
    ...
    </Service>
    ...
    
    ```
    
- Web Application 추가
    
    ```
    ...
    <Host name="localhost" appBase="webapps"
          unpackWARs="true" autoDeploy="true">
          <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
                 prefix="localhost_access_log" suffix=".txt"
                 pattern="%h %l %u %t &quot;%r&quot; %s %b" />
          # https:localhost:Port/sample/, docBase = Page Directory
          <Context path = "/sample" docBase = "${TOMCAT_HOME}/sample/" />
    </Host>
    ...
    
    ```
    

## Web Server 연동

### 1.1) Apache

- https://github.com/TmaxSoftOfficial/HyperFrame-Apache 통해 Apache 설치 진행
- Apache와 Tomcat을 연동하기 위해서는 mod_jk 플러그인이 필요
    - mod-jk download : [https://github.com/TmaxSoftOfficial/HyperFrame-Apache/tree/main/binaries](https://github.com/TmaxSoftOfficial/HyperFrame-Apache/tree/main/binaries)
    <br>
- tomcat-connectors-1.2.43.src.tar.gz를 다운로드 받고 아래와 같은 명령어로 압축 해제
    
    ```
    $ tar xvfz tomcat-connectors-1.2.43-src.tar.gz
    $ cd tomcat-connectors-1.2.43-src/native
    
    ```
    
- 위의 native 폴더에서 아래의 명령어로 configure 적용
    
    ```
    $ cd ${TOMCAT_CONNECTIOR}/native/
    $ ./configure --with-apxs=${APACHE_HOME}/bin/apxs
    $ make & make install
    
    ```
    
- ${APACHE_HOME}/modules/ Directory에 mod_jk.so 생성 확인
    
    ```
    $ ls -al ${APACHE_HOME}/modules/mod_jk.so
    ```
    
- Tomcat에서 AJP 프로토콜 설정
    
    ```
    $ vi ${TOMCAT_HOME}/conf/server.xml
    
    ...
    <Service name="Catalina">
    
        <!-- Define an AJP 1.3 Connector on port 8009 -->
        <Connector protocol="AJP/1.3" address="0.0.0.0" secretRequired="false" port="8009" redirectPort="8443" />
    ...
    </Service>
    ...
    
    ```
    
- Apache의 worker 설정
    
    ```
    $ vi ${APACHE_HOME}/conf/workers.properties/
    
    worker.list=worker1
    worker.worker1.type=ajp13                # AJP1.3 프로토콜을 사용
    worker.worker1.host=${TOMCAT기동서버ip}   # 톰캣이 기동하고 있는 서버 IP를 등록
    worker.worker1.port=8009                  # 연결할 톰캣의 포트 번호
    
    ```
    
- Apache와 Tomcat 연동
    
    ```
    $ vi ${APACHE_HOME}/conf/httpd.conf
    
    ...
    LoadModule jk_module modules/mod_jk.so # LoadModule 라인에 작성
    
    <IfMoudle unixd_module>
    ...
    ...
    <IfModule jk_module>
    Include conf/mod_jk.conf # 파일 맨 아래에 작성
    </IfModule>
    ```
    
    ```
    $ vi ${APACHE_HOME}/conf/mod_jk.conf # 파일 생성
    
    JkWorkersFile /home/youngsoo/apache-2.4.48/conf/workers.properties
    JkLogFile /home/youngsoo/apache-2.4.48/logs/mod_jk.log
    JkLogLevel info
    JkLogStampFormat "[%a %b %d %H:%M:%S %Y]"
    JkShmFile /home/youngsoo/apache-2.4.48/logs/mod_jk.shm
    JkMount /* worker1
    ```
    

### 1.2) SSL 설정

- 인증기관(CA) 개인 키 발급
    
    ```
    $ mkdir ${HOME}/ssl
    $ cd ssl
    $ openssl genrsa -des3 -out server.key 2048
    
    ...
    Enter pass phrase for server.key: # 개인 키 비밀번호 생성
    Verifying - Enter pass phrase for server.key: # 비밀번호 확인
    ```
    
- 인증요청서(CSR) 생성
    
    ```
    $ openssl req -new -days 365 -key server.key -out ssl.csr
    
    ...
    Enter pass phrase for server.key: # 개인 키 비밀번호 입력
    ...
    ------
    Country Name (2 letter code) [XX]: # 모든 정보 입력 부분은 Default로 진행해도 상관 없음
    ...
    ...
    An optional company name []:
    ```
    
- 인증서(CRT) 생성
    
    ```
    $ openssl x509 -req -days 465 -in ssl.csr -signkey server.key -out ssl.crt
    
    ...
    Enter pass phrase for server.key: # 개인 키 비밀번호 입력
    ```
    
- httpd.conf 설정
    
    ```
    $ vi ${APACHE_HOME}/conf/httpd.conf
    
    # Secure (SSL/TLS) connections
    Include conf/extra/httpd-ssl.conf # 주석 해제
    ```
    
- httpd-ssl.conf 설정
    
    ```
    $ vi ${APACHE_HOME}/conf/extra/httpd-ssl.conf
    
    ...
    <VirtualHost _DEFAULT_:443>
    
    JkMount /* worker1
    
    # General setup for the virtual host
    ...
    SSLCertificateFile "${SSL_HOME}/ssl.crt"
    ...
    SSLCertificateKeyFile "${SSL_HOME}/server.key"
    ```
    
- Apache를 실행하면 비밀번호를 묻는데, 이 때 개인 키 비밀번호로 지정했던 값을 입력하면 된다.

## 2) Nginx

- 사전에 Nginx 설치가 필요 (Nginx 제품의 [README.MD](http://readme.md/) 파일 참고)
- [Nginx] ${NGINX_HOME}/conf/nginx.conf 파일 수정
    
    ```
    $ vi ${NGINX_HOME}/conf/nginx.conf
    ...
    location / {
    root             html;
    index            index.html index.htm;
    proxy_pass     http://[Tomcat IP]:[Tomcat Listen Port];
    }
    ...
    
    ```
    
- Nginx 및 Tomcat 기동
