## HyperFrameOE-Tomcat

업로드된 바이너리는 HyperFrame Open Edition Tomcat 제품 설치를 위한 파일

## 설치 파일

### Tomcat

* Version : apache-tomcat-9.0.52
* Note : https://tomcat.apache.org/download-90.cgi

### Tomcat-Connectors

* Version : tomcat-connectors-1.2.48
* Note : https://tomcat.apache.org/download-connectors.cgi

## 설치 및 실행

### 1) Tomcat 압축 풀기

    $ tar -zxf apache-tomcat-9.0.52.tar.gz

### 2) Port 확인 및 변경

    $ vi ${TOMCAT_HOME}/conf/server.xml
    ...
    <Service name="Catalina">
        <Connector port="80" protocol="HTTP/1.1"
                   connectionTimeout="20000"
                   redirectPort="8443" />
    ...
    </Service>

### 3) Tomcat 실행

    $ cd ${TOMCAT_HOME}/bin/
    $ ./startup.sh
    
### 4) Tomcat 종료

    $ cd ${TOMCAT_HOME}/bin/
    $ ./shutdown.sh

## 디렉토리 구조

    $ cd /home/username/apache-tomcat/
        apache-tomcat
        ├── bin
        │    ├── bootstrap.jar       <---------------- tomcat 서버가 구동될 때 사용되는 main( ) 메소드 포함
        │    ├── tomcat-juil.jar
        │    ├── common-daemon.jar
        │    ├── catalina.sh         <---------------- CATALINA 서버의 제어 스크립트
        │    ├── ciphers.sh
        │    ├── configtest.sh       <---------------- CATALINA 서버의 설정 스크립트
        │    ├── daemon.sh
        │    ├── makebase.sh
        │    ├── setclasspath.sh     <---------------- JAVA_HOME 또는 JRE_HOME이 세팅되지 않았을 경우 세팅
        │    ├── shutdown.sh         <---------------- CATALINA 서버를 중지하는 스크립트
        │    ├── startup.sh          <---------------- CATALINA 서버를 시작하는 스크립트
        │    ├── tool-wrapper.sh
        │    └── version.sht
        ├── conf
        │    ├── catalina.policy
        │    ├── catalina.properties	
        │    ├── context.xml         <---------------- 세션, 쿠키 저장 경로등을 지정하는 설정 파일
        │    ├── logging.properties	
        │    ├── server.xml          <---------------- Tomcat 설정에서 가장 중요, Service, Connertor 등과 같은 주요 기능 설정 가능
        │    ├── tomcat-users.xml    <---------------- Tomcat의 manager 기능을 사용하기 위해 사용자 권한을 설정	
        │    └── web.xml             <---------------- Tomcat의 환경설정 파일	   
        └── common

## 버전 확인

    $ cd ${TOMCAT_HOME}/home/username/apache-tomcat/lib
    $ java -cp catalina.jar org.apache.catalina.util.ServerInfo
    Server version: Apache Tomcat/9.0.52
    Server built:   Jul 31 2021 04:12:17 UTC
    Server number:  9.0.52.0
    OS Name:        Linux
    OS Version:     2.6.32-754.11.1.el6.x86_64
    Architecture:   amd64
    JVM Version:    1.8.0_171-b11
    JVM Vendor:     Oracle Corporation

## 로그 정보

### 1) 로그 경로

    # 서버상에서 발생한 모든 내용을 기록한 Log
    $ {TOMCAT_HOME}/logs/catalina.out
    
    # Tomcat에서 발생하는 Log
    $ {TOMCAT_HOME}/logs/catalina.yyyy-mm-dd.log
    
    # Tomcat Manager Web App Log
    $ {TOMCAT_HOME}/logs/manager.log 

## 환경 설정 파일 정보

### 1) 환경 설정 파일 경로

    $ {TOMCAT_HOME}/conf/server.xml
    
### 2) 환경 설정

* Port 변경

      ...
      <Service name="Catalina">
          Connector port="80" protocol="HTTP/1.1"
                   connectionTimeout="20000"
                   redirectPort="8443" />
      ...
      </Service>  

* 프로토콜 별 Connector 추가 및 삭제

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

* Web Application 추가
    
      ...
      <Host name="localhost" appBase="webapps"
            unpackWARs="true" autoDeploy="true">
            <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
                   prefix="localhost_access_log" suffix=".txt"
                   pattern="%h %l %u %t &quot;%r&quot; %s %b" />
            # https:localhost:Port/sample/, docBase = Page Directory 
            <Context path = "/sample" docBase = "/home/minjun/tomcat/server1/sample/" />
      </Host>
      ...


## Web Server 연동

### 1) Apache

* Apache와 Tomcat을 연동하기 위해서는 mod-jdk 플러그인이 필요

* tomcat-connectors-1.2.43.src.tar.gz를 다운로드 받고 아래와 같은 명령어로 압축해제

      $ tar xvfz tomcat-connectors-1.2.43-src.tar.gz
      $ cd tomcat-connectors-1.2.43-src/native

* 위의 native 폴더에서 아래의 명령어로 configure를 적용

      $ cd ${TOMCAT_CONNECTIOR}/native/
      $ ./configure --with-apxs=${APACHE_HOME}/bin/apxs
      $ make & make install

* ${APACHE_HOME}/modules/ Directory mod_jk.so 생성확인

* Tomcat에서 AJP 프로토콜 설정

      $ vi ${TOMCAT_HOME}/conf/ server.xml
      ...
      <Service name="Catalina">
          <Connector protocol="AJP/1.3" address="::1" port="8009" redirectPort="8443" />
      ...
      </Service>
      ...

* Apache의 worker 설정

      $ vi ${APACHE_HOME}/conf/workers.properties/
      worker.list=worker1
      worker.worker1.type=ajp13		      # AJP1.3 프로토콜을 사용
      worker.worker1.host=localhost	      # 톰캣은 local에서 돌고 있습니다.
      worker.worker1.port=8009	  	      # 연결할 톰캣의 포트 번호

* Apache와 Tomcat 연동

      $ vi ${APACHE_HOME}/conf/httpd.conf
      LoadModule jk_module            modules/mod_jk.so
      <IfModule mod_jk.c>
              JkWorkersFile /usr/local/victolee/apache2.0.64/conf/workers.properties    # 실행파일
              JkLogFile /usr/local/victolee/apache2.0.64/logs/mod_jk.log                # 로그 경로
              JkLogLevel info                                                           # 로그레벨 설정
              JkLogStampFormat "[%a %b %d %H:%M:%S %Y]"                                 # 로그 포맷
              JkShmFile /usr/local/victolee/apache2.0.64/logs/mod_jk.shm                # 공유파일
              JkMount /*.jsp worker1                                                    # /*.jsp 파일은 worker1에게 넘긴다         
      </IfModule>
      
## 2) Nginx

* nginx 종료

      $ ./nginx -s stop

* [Nginx] ${NGINX_HOME}/conf/nginx.conf 파일 수정

      $ vi ${NGINX_HOME}/conf/nginx.conf
      ...
      location / {
      root             html;
      index            index.html index.htm;
      proxy_pass     http://[Tomcat IP]:[Tomcat Listen Port];
      }
      ...
      
* Nginx 및 Tomcat 기동 
