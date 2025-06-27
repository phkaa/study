#### 서비스 확인 방법
- CentOS 6
  - service [서비스이름] stop: 서비스 중지
  - service [서비스이름] start: 서비스 시작
  - service [서비스이름] restart: 서비스 재시작
  - service [서비스이름] status: 서비스 상태 확인
  - service [서비스이름] reload: 서비스의 설정파일을 리로드
- CentOS 7
  - systemctl stop [서비스이름]: 서비스 중지
  - systemctl start [서비스이름]: 서비스 시작
  - systemctl restart [서비스이름]: 서비스 재시작
  - systemctl status [서비스이름]: 서비스 상태 확인
  - systemctl reload [서비스이름]: 서비스의 설정파일을 리로드
- 톰캣 서버 버전 숨기기
  - server.xml 파일을 연다.
  - server 키의 값을 공백으로 아래와 같이 채운다.
  - 이렇게 하면 Server의 Apache-Coyote/1.1이 공백으로 출력된다.

  ```
  <Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" URIEncoding="UTF-8"
               server=" "/>
  ```

#### 톰캣 서버 Trace Method 차단하기
  - server.xml 파일을 연다.
  - allowTrace 키의 값을 false 로 아래와 같이 채운다.

  ```
  <Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" URIEncoding="UTF-8"
               server=" "
               allowTrace="false"/>
  ```

#### 톰캣 Allow Method 제한하기
  - server.xml 파일을 연다.
  - 위의 '톰캣 서버 Trace Method 차단하기' 를 적용한다.
  - web.xml 파일을 연다.
  - web.xml 파일 맨아래에 아래(</web-app> 바로위)와 같이 제한할 allow method를 작성한다.
  - 서버를 재시작 한다.

  ```
  <security-constraint>
		<display-name>Forbidden</display-name>
		<web-resource-collection>
			<web-resource-name>restricted methods</web-resource-name>
			<url-pattern>/*</url-pattern>
			<http-method>PUT</http-method>
			<http-method>HEAD</http-method>
			<http-method>HEAD</http-method>
			<http-method>DELETE</http-method>
			<http-method>TRACE</http-method>
			<http-method>COPY</http-method>
			<http-method>MOVE</http-method>
		</web-resource-collection>
		<auth-constraint />
	</security-constraint>
    <security-constraint>
        <web-resource-collection>
            <web-resource-name>Protected Context</web-resource-name>
            <url-pattern>/servlet/org.apache.catalina.servlets.DefaultServlet/*</url-pattern>
        </web-resource-collection>
        <user-data-constraint>
            <transport-guarantee>CONFIDENTIAL</transport-guarantee>
        </user-data-constraint>
    </security-constraint>
  ```

#### 톰캣 에러 페이지 커스텀
- 톰캣 설치 위치/conf/web.xml 파일을 연다.
- web.xml 파일의 맨 아래에 아래의 같이 에러 페이지를 정의 한다.
- html 파일은 톰캣 설치 위치/webapps/ROOT 안에 넣는다.

```
<web-app>
  ...
  <welcome-file-list>
      <welcome-file>index.html</welcome-file>
      <welcome-file>index.htm</welcome-file>
      <welcome-file>index.jsp</welcome-file>
  </welcome-file-list>

  <error-page>
      <error-code>404</error-code>
      <location>/error/404.html</location>
  </error-page>
  
  <error-page>
      <error-code>500</error-code>
      <location>/error/500.html</location>
  </error-page>
</web-app>
```

#### Spring Allow Method 제한하기(위의 톰캣과 같이 진행해야 한다.)
  - spring 프로젝트의 web.xml 파일을 연다.
  - `<web-app>` 안에 아래와 같이 입력한다.
  - `<servlet>` 안에 dispatchOptionsRequest 옵션을 true로 추가한다.

  ```
  <!-- 서블릿 파일 설정 -->
	<servlet>
		<servlet-name>HelloWorld</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>/WEB-INF/hw-servlet.xml </param-value>
		</init-param>
		<init-param>
			<param-name>dispatchOptionsRequest</param-name>
			<param-value>true</param-value>
		</init-param>
		<load-on-startup>1</load-on-startup>
	</servlet>

  <security-constraint>
		<display-name>Forbidden</display-name>
		<web-resource-collection>
			<web-resource-name>restricted methods</web-resource-name>
			<url-pattern>/*</url-pattern>
			<http-method>PUT</http-method>
			<http-method>HEAD</http-method>
			<http-method>HEAD</http-method>
			<http-method>DELETE</http-method>
			<http-method>TRACE</http-method>
			<http-method>COPY</http-method>
			<http-method>MOVE</http-method>
		</web-resource-collection>
		<auth-constraint>
			<role-name></role-name>
		</auth-constraint>
	</security-constraint>
  ```

#### 아파치 버전 숨기는 방법
- httpd.conf 파일을 연다.
- ServerSignature 값을 Off 로 변경한다.
- ServerTokens 값을 Prod 로 변경한다.
  - Prod: 웹서버의 이름만 노출
  - Major: 웹서버의 이름과 Major 버전번호만 노출
  - Minor: 웹서버의 이름과 Minor 버전번호까지 노출
  - Min: 웹서버의 이름과 Minimum 버전까지 노출
  - OS: 웹서버의 이름과 버전, 운영체제까지 노출
  - Full: 최대한의 정보를 모두 노출

#### 아파치 커스텀 에러 페이지
- httpd.conf 파일을 연다.
- Alias /error/ "/var/www/error/" 이 부분을 찾는다.
- 해당 부분아래에 주석으로 된 ErrorDocument 가 존재한다.
- 주석을 해제하고 에러 페이지를 넣는다.
- HTML 파일은 /var/www/error 이라고 정의되어 있는데 해당 폴더에 넣는다.

```
Alias /error/ "/var/www/error/"

<IfModule mod_negotiation.c>
<IfModule mod_include.c>
    <Directory "/var/www/error">
        AllowOverride None
        Options IncludesNoExec
        AddOutputFilter Includes html
        AddHandler type-map var
        Order allow,deny
        Allow from all
        LanguagePriority en es de fr
        ForceLanguagePriority Prefer Fallback
    </Directory>

    ErrorDocument 400 /error/400.html
    ErrorDocument 401 /error/401.html
    ErrorDocument 403 /error/403.html
    ErrorDocument 404 /error/404.html
    ErrorDocument 405 /error/405.html
#    ErrorDocument 408 /error/HTTP_REQUEST_TIME_OUT.html.var
#    ErrorDocument 410 /error/HTTP_GONE.html.var
#    ErrorDocument 411 /error/HTTP_LENGTH_REQUIRED.html.var
#    ErrorDocument 412 /error/HTTP_PRECONDITION_FAILED.html.var
#    ErrorDocument 413 /error/HTTP_REQUEST_ENTITY_TOO_LARGE.html.var
#    ErrorDocument 414 /error/HTTP_REQUEST_URI_TOO_LARGE.html.var
#    ErrorDocument 415 /error/HTTP_UNSUPPORTED_MEDIA_TYPE.html.var
    ErrorDocument 500 /error/500.html
#    ErrorDocument 501 /error/HTTP_NOT_IMPLEMENTED.html.var
#    ErrorDocument 502 /error/HTTP_BAD_GATEWAY.html.var
#    ErrorDocument 503 /error/HTTP_SERVICE_UNAVAILABLE.html.var
#    ErrorDocument 506 /error/HTTP_VARIANT_ALSO_VARIES.html.var

</IfModule>
</IfModule>
```

#### 아파치 Method 제한하기
- httpd.conf 파일을 연다.
- Location 부분을 추가하여 제한할 메소드를 정의한다.
- LimitExcept는 허용가능한 메소드이고 Limit는 허용하지 않는 메소드 이다.
- TraceEnable Off 도 추가한다.(trace method는 Limit 가 아닌 TraceEnable로 Off 하면 된다.)

```
<Location />
	<LimitExcept GET POST OPTION PUT>
		Order allow,deny
    		Allow from all
	</LimitExcept>
#    <Limit TRACE>
#        Order allow,deny
#            Allow from all
#    </Limit>
</Location>

TraceEnable off
```

### CentOS7 기준

#### 자바 설치
  - rpm 파일 위치: /data/download/jdk-8u192-linux-x64.rpm
  - 설치 방법:
    - RPM 파일 위치로 이동
    - rpm -ivh jdk-8u192-linux-x64.rpm 입력
    - java -version 입력
    - 자바 버전의 정보가 정상적으로 나온다면 설치 완료된것임.
    - 환경변수 등록하기
    - vi /etc/profile.d/java.sh 입력
    - 아래 입력
    
    ```
    #!/bin/bash
    JAVA_HOME=/usr/java/jdk1.8.0_192-amd64/
    PATH=$JAVA_HOME/bin:$PATH
    export PATH JAVA_HOME
    export CLASSPATH=.
    ```

    - chmod +x /etc/profile.d/java.sh 입력(권한 설졍)
    - source /etc/profile.d/java.sh 입력
    - 환경 변수 설정 완료

#### 톰캣 설치
  - ZIP 파일 위치: /data/download/apache-tomcat-8.5.41.zip
  - 설치 방법:
    - ZIP 파일 위치로 이동
    - unzip ./apache-tomcat-8.5.41.zip -d /data/ 입력
    - /data/apache-tomcat-8.5.41 라는 폴더안에 압축히 해제되어있다.
    - apache-tomcat-8.5.41 폴더명을 원하는 폴더명으로 변경한다.(현재 tomcat8 로 변경함)
    - /data/tomcat8/conf 의 server.xml 에서 8080 포트에 URIEncoding="UTF-8" 를 추가한다.
    - /data/tomcat8/bin 으로 이동한다
    - chmod 700 *.sh 명령어 실행한다(startup.sh 를 실행하면 명령거부가 나오기 때문이다.)
    - /data/tomcat8/bin 폴더에 setenv.sh 를 생성하여 힙메모리를 설정한다. 톰캣의 기본 자바 힙 메모리는 메모리의 1/4 이므로 조정을 해준다.

    ```
    export CATALINA_OPTS="$CATALINA_OPTS -Xms512m"
    export CATALINA_OPTS="$CATALINA_OPTS -Xmx2048m"
    ```

    - 서비스를 등록을 준비한다.
    - cd /etc/systemd/system 입력
    - vi /etc/systemd/system/tomcat.service 입력
    - 아래의 사항을 입력 후 저장

    ```
    [Unit] 
    Description=tomcat 8
    After=network.target syslog.target
    
    [Service] 
    Type=forking 
    Environment="/data/tomcat8" 
    User=root 
    Group=root 
    ExecStart=/data/tomcat8/bin/startup.sh 
    ExecStop=/data/tomcat8/bin/shutdown.sh 
    
    [Install] 
    WantedBy=multi-user.target
    ```

    - systemctl enable tomcat.service 입력(서비스 등록)
    - 톰캣 시작 중지 명령어

    ```
    시작: service tomcat start
    종료: service tomcat stop
    ```

#### MariaDB 설치
  - sudo yum update 입력(yum 업데이트 진행)
  - 서버를 재시작 한다.
  - vi /etc/yum.repos.d/MariaDB.repo 입력 후 아래의 사항을 저장한다.

  ```
  [mariadb]
  name = MariaDB
  baseurl = http://yum.mariadb.org/10.4/centos7-amd64
  gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
  gpgcheck=1
  ```

  - yum install MariaDB 입력
  - 설치 완료 후 rpm -qa | grep MariaDB 입력 후 아래와 같이 나오면 정상적인 설치 완료(버전은 다를 수 있음)

  ```
  MariaDB-compat-10.4.5-1.el7.centos.x86_64
  MariaDB-client-10.4.5-1.el7.centos.x86_64
  MariaDB-common-10.4.5-1.el7.centos.x86_64
  MariaDB-server-10.4.5-1.el7.centos.x86_64
  ```
  - systemctl start mariadb 입력하여 MariaDB 시작
    - systemctl stop mariadb MariaDB 중지
  - netstat -anp | grep 3306 입력하여 정상적으로 실행이 되었는지 확인
  - mysql -u root -p 입력하여 접속(처음 비밀번호는 없음)
  - vi /etc/my.cnf 입력 후 맨 아래에 아래의 정보를 입력한다.(UTF 8 설정)

  ```
  [client]
  default-character-set = utf8

  [mysqld]
  init_connect = "SET collation_connection = utf8_general_ci"
  init_connect = "SET NAMES utf8"
  character-set-server = utf8
  collation_server = utf8_general_ci
  lower_case_table_names = 1

  [mysql]
  default-character-set = utf8
  ```

  - mysql -u root -p 입력하여 접속
  - show variables like 'c%'; 입력 후 아래와 같이 나온다면 UTF8 설정 완료

  ```
  +----------------------------------+----------------------------+
  | Variable_name                    | Value                      |
  +----------------------------------+----------------------------+
  | character_set_client             | utf8                       |
  | character_set_connection         | utf8                       |
  | character_set_database           | utf8                       |
  | character_set_filesystem         | binary                     |
  | character_set_results            | utf8                       |
  | character_set_server             | utf8                       |
  | character_set_system             | utf8                       |
  | character_sets_dir               | /usr/share/mysql/charsets/ |
  | check_constraint_checks          | ON                         |
  | collation_connection             | utf8_general_ci            |
  | collation_database               | utf8_general_ci            |
  | collation_server                 | utf8_general_ci            |
  | column_compression_threshold     | 100                        |
  | column_compression_zlib_level    | 6                          |
  | column_compression_zlib_strategy | DEFAULT_STRATEGY           |
  | column_compression_zlib_wrap     | OFF                        |
  | completion_type                  | NO_CHAIN                   |
  | concurrent_insert                | AUTO                       |
  | connect_timeout                  | 10                         |
  | core_file                        | OFF                        |
  +----------------------------------+----------------------------+
  ```

  - 비밀번호와 계정 1개를 생성 한다. 아래 참조

  ```
  # root 계정 비밀번호 지정
  use mysql
  set password=password('비밀번호');
  FLUSH PRIVILEGES;

  # root 계정 외부 접속
  use mysql
  grant all privileges on *.* to 'root'@'%' identified by '비밀번호';
  FLUSH PRIVILEGES;

  # eda 계정 외부 접속 생성
  use mysql
  grant all privileges on *.* to 'eda'@'%' identified by '비밀번호';
  FLUSH PRIVILEGES;

  grant all privileges on *.* to 'eda'@'%';
  FLUSH PRIVILEGES;

  # eda 계정 내부 접속 생성
  use mysql
  grant all privileges on *.* to 'eda'@'localhost' identified by '비밀번호';
  FLUSH PRIVILEGES;

  grant all privileges on *.* to 'eda'@'localhost';
  FLUSH PRIVILEGES;
  ```
#### Docker 설치
  - sudo yum update 입력(yum 업데이트 진행)
  - yum -y install docker docker-registry 입력
  - systemctl enable docker.service 입력(부팅시에 자동 실행 등록)
  - systemctl start docker.service 입력(도커 실행)
    - systemctl stop docker.service 도커 중지
  - systemctl status docker.service 입력(도커 스테이터스 확인)
  - 그외 도커 명령어
  
  ```
  # 도커 버전 화인
  docker -v

  # 도커 실행 명령어
  docker run [OPTIONS] IMAGE[:TAG|@DIGEST] [COMMAND] [ARG...]

  옵션	설명
    -d	detached mode 흔히 말하는 백그라운드 모드
    -p	호스트와 컨테이너의 포트를 연결 (포워딩)
    -v	호스트와 컨테이너의 디렉토리를 연결 (마운트)
    -e	컨테이너 내에서 사용할 환경변수 설정
    –name	컨테이너 이름 설정
    –rm	프로세스 종료시 컨테이너 자동 제거
    -it	-i와 -t를 동시에 사용한 것으로 터미널 입력을 위한 옵션
    –link	컨테이너 연결 [컨테이너명:별칭]

  # 도커 이미지 찾기
  sudo docker search <Name>

  # 다운로드 이미지 확인
  sudo docker images

  # 도커 컨테이너 확인
  sudo docker ps -a

  # 도커 컨테이너 삭제
  sudo docker rm <컨테이너ID>

  # 도커 컨테이너 접속
  sudo docker exec -it <컨테이너 이름> /bin/bash

  # 그외 명령어 및 문서
  https://docs.docker.com/
  ```

#### 도커 컨테이너에서 vim 설치 방법
- docker exec 명령어를 이용하여 도커 컨테이너에 접속한다.
- apt-get update 입력
- apt-get install vim 입력하여 vim 설치

#### Grafana + Telegraf + Influxdb를 활용한 모니터링
- sudo docker pull samuelebistoletti/docker-statsd-influxdb-grafana 입력하여 컨테이너 다운로드
- docker run --ulimit nofile=66000:66000 -d -it --name docker-statsd-influxdb-grafana -p 3003:3003 -p 3004:8888 -p 8086:8086 -p 22022:22 -p 8125:8125/udp samuelebistoletti/docker-statsd-influxdb-grafana:latest 입력하여 실행(run 명령어는 도커 컨테이너 생성 및 실행 동시에 진행)
- 211.253.25.87:3003 접속한다.(기본 아이디/패스워드는 root/root)
- 아래의 3개를 대시보드 다운로드 한다.
  - [javastacksummary_rev1.json](./file/javastacksummary_rev1.json)
  - [service-mysql-metrics_rev1.json](./file/service-mysql-metrics_rev1.json)
  - [linux-system-overview_rev1.json](./file/linux-system-overview_rev1.json)
  - [그외 대시보드 종료](https://grafana.com/dashboards/)

- 좌측 + 아이콘 클릭 후 import 메뉴 진입
- 우측 Upload .json file 버튼 클릭하여 다운받은 2개의 json 파일을 각각 선택한다.
- telegraf 에서influxDB를 선택 후 Import 버튼 클릭하면 그래피 Import 완료

#### Telegraf 에서 Mysql Metric 정보 수집 방법
- docker exec 명령어를 이용하여 도커 컨테이너에 접속한다.
- vim /etc/telegraf/telegraf.conf 입력
- [[inputs.mysql]] 을 찾는다
- 주석을 전부 해제하고 servers 부분을 아래와 같이 입력한다.

```
[[inputs.mysql]]

servers = ["mysql_user_id:password@tcp(172.27.0.14:3306)/"]
```

- 저장하고 도커 컨테이너를 재시작한다.(docker restart <컨테이너 ID>)
- 도커 컨테이너에 접속을 한다.
- influx 입력해서 인플럭스DB에 접속한다.
- 접속이 완료되면 use telegraf 입력
- show measurements 입력해서 mysql 정보를 수집하고 있는지 데이터를 확인한다.

#### Telegraf 에서 Jolokia 를 활용하여 Java Metric 정보 수집 방법
- JMX
  - JVM 상태를 모니터링 할때 간단하게 사용하는 API
- jolokia
  - 원격지에서  JMX에 접근을 목적으로 사용되는 Java Specification Request(JSR 160)을 구현해 놓은 Agent 이며, Client  와 Agent 간의 통신은 HTTP/JSON으로 요청, JSON 으로 정보를 받는다.
  - 자바 metric 정보를 influxDB에 넣을 수 있도록 도와 준다.
  - 실행방법
    - https://jolokia.org/download.html 사이트에서 jolokai-war-1.6.2.war 을 다운받아 이름을 jolokia.war 로 변경 후 톰캣 webapps 위치에 옮긴다.
    - tomcat 이 설치된 폴더로 이동 후 conf/tomcat-users.xml 파일을 vim 으로 연다.
    - 아래와 같이 수정을 한다.

    ```
    <role rolename="tomcat"/>
    <role rolename="role1"/>
    <role rolename="jolokia"/>
    <user username="jolokia" password="암호" roles="tomcat,role1,jolokia"/>
    ```

    - 톰캣을 재시작 한다.
    - 브라우저에서 http://127.0.0.1:8080/jolokia 로 접속을 하면 브라우저에 아래의 같이 정보가 응답 된다.

    ```
    {"request":{"type":"version"},"value":{"agent":"1.6.2","protocol":"7.2","config":{"listenForHttpService":"true","maxCollectionSize":"0","authIgnoreCerts":"false","agentId":"127.0.0.1-11165-657a5304-servlet","agentType":"servlet","policyLocation":"classpath:\/jolokia-access.xml","agentContext":"\/jolokia","mimeType":"text\/plain","discoveryEnabled":"false","streaming":"true","historyMaxEntries":"10","allowDnsReverseLookup":"true","maxObjects":"0","debug":"false","serializeException":"false","detectorOptions":"{}","dispatcherClasses":"org.jolokia.http.Jsr160ProxyNotEnabledByDefaultAnymoreDispatcher","maxDepth":"15","authMode":"basic","authMatch":"any","canonicalNaming":"true","allowErrorDetails":"true","realm":"jolokia","includeStackTrace":"true","useRestrictorService":"false","debugMaxEntries":"100"},"info":{"product":"tomcat","vendor":"Apache","version":"8.5.41"}},"timestamp":1615515409,"status":200}
    ```
  - jolokia 실행이 완료 된 후
    - docker exec 명령어를 이용하여 도커 컨테이너에 접속한다.
    - vim /etc/telegraf/telegraf.conf 입력
    - telegraf.conf 에서 [[inputs.jolokia]] 를 찾아서 아래와 같이 주석을 해제 하고 입력한다.

    ```
    [[inputs.jolokia]]
    context = "/jolokia/"

    [[inputs.jolokia.servers]]
    name = "tomcat8-server-01"
    host = "127.0.0.1"
    port = "8080"
    username = "jolokia"
    password = "tomcat-users 에서 설정한 암호

    ```

    - 저장하고 도커 컨테이너를 재시작한다.(docker restart <컨테이너 ID>)
    - 도커 컨테이너에 접속을 한다.
    - influx 입력해서 인플럭스DB에 접속한다.
    - 접속이 완료되면 use telegraf 입력
    - show measurements 입력해서 jolokia 정보를 수집하고 있는지 데이터를 확인한다.
    - [그라파나 대시보드](https://grafana.com/grafana/dashboards/4503) 접속하여 그라파나에 적용한다.
    - Jolokia 만 셋팅을 하였으니 Jolokia 부분만 남기고 삭제 해서 대시보드를 확인하면 된다.