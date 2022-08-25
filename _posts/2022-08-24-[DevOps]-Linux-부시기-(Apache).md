---
layout: post
title:  "[DevOps] Linux 부시기 (Apache)"
date:   2022-08-24 08:56:53 +0900
categories: DevOps
tag: Linux Ubuntu Apache
---
자 이전 까지 했던것과 더불어서 오늘은 `Apache`에 대해 알아보자

1. ~~`SSH`와 `FTP` 세팅 원리와 이해~~
2. ~~`Port forwording` 세팅 원리와 이해~~
3. ~~`Linux` 에서 `Forground` 와 `Background`의 이해~~
4. ~~`DNS` 세팅 원리와 이해~~
5. `Apache` 세팅 원리와 이해
6. `Mail server` 세팅 원리와 이해
    

자 오늘도 열심히 달려보자~

---

# Apache 세팅 원리와 이해

`Apache`는 `Web server`의 일종이다.

`Web server`란 서버 프로그램으로, 시스템 소프트웨어라고 불리며, 클라이언트에게 `HTML`문서로 된 페이지를 전달하는 역할을 한다.


즉, `HTTP` 프로토콜을 이용해 `Apache`는 저장되어져 있는 `HTML` 페이지를 `Server` -> `Client`로 전달하는 역할을 한다.

이 중간에 `WAS`를 사용하여 `JSP`를 처리하고 `Static Resource`인 `HTML`에 포함하여 보내기도 한다. 이 과정에서 `Web server`는 `HTML`을 `WAS`에 보내게 된다.

`Web server`의 종류는 `Apache`, `Nginx`, `IIS`등 여러가지가 있고, `Apache`가 가장 인기가 많으며 `Nginx` 사용량도 많다.

프로젝트를 진행하면서 `Apache` ,`Nginx` 모두 사용해 보았는데 사실 크게 차이를 느끼지는 못했다. 실제 유저가 많은 사이트의 경우 `Apache`의 `MPM`을 활용하여 구성하는 것과 `Nginx`의 `Thread pool`과 비교했을때 어떤것이 더 좋다는 경험해 보지 못했다.....


제발! 큰 회사여 날 뽑아가시오 저런거 다 해보고싶습니다 ...ㅜ



## `Apache` 세팅

---
먼저 아래의 명령어를 사용하여 `Apache` 패키지를 다운 받는다.

```linux
sudo apt install apache2
```
설치가 완료 되었으면 아래 명령어를 쳐서 동작 하고 있는지 확인을 한다.

```linux
sudo systemctl status apache2
```

자 그럼 `Apache` 설정파일은 무엇이 있고, 각 설정파일이 의미하는 것이 무엇인지
자세히 알아보도록 하자.

먼저 아래의 설정 파일의 구조를 살펴보자
![이미지](/assets/images/apache2_config.PNG)

`Apache`는 설정 파일에 `directives(지시문)`을 작성하여 넣으면 된다. 그러나 설정 파일이 하나가 아니라, 여러 개로 쪼개져서 만들어져 있어서, 필요한 설정을 필요한 곳에 넣어야 한다.

그렇기 때문에, 각 설정파일이 의미하는 것이 무엇인지 어떤곳에 어떤 설정을 해야하는지 충분히 이해할 필요가 있다.

- `apache2.conf`: 메인 설정 파일, 아파치 전체에 적용되는 설정을 포함한다.
- `ports.conf`: TCP port에 대한 설정
- `conf-available`과 `conf-enabled`: `conf-available` 안에는 설정 파일인 conf 파일들이 들어 있다. `conf-enabled` 안에다가 적용하고 싶은 설정을 심볼릭 링크 해주면 그 설정이 활성화된다.
- `mods-available`과 `mods-enabled`: 위와 비슷한데 모듈을 로드하고 설정하는데 필요한 설정 파일들이 들어 있다.
- `sites-available`과 `sites-enabled`: 이 부분은 조금 중요한데, 아파치는 포트번호와 호스트네임에 따라서 마치 서버가 여러 개가 돌아가는 것처럼 각기 다른 설정을 주어 구분할 수 있는 기능을 가지고 있는데, 이걸 Virtual Host라고 한다. 이 폴더는 각 Virtual Host에 대한 설정들을 담고 있다.

위 파일중 available,enabled 파일은 심볼릭 링크를 해주어야 한다 이를 간단하게 실행시켜주는 명령어는 아래와 같다.

    모듈 활성화/비활성화 명령 : a2enmod/a2dismod
    가상 호스트 사이트 활성화/비활성화 명령 : a2ensite/a2dissite
    환경 구성 활성화/비활성화 :  a2enconf/a2disconf

## apache2.conf 설정

`apache2.conf` 는 기본적인 전역변수가 설정되어져 있다. 전역변수를 `directive`라고 표현하는데, 

각 전역변수에 대하여 설정을 함으로써 필요한 기능과 파일의 위치등을 설정할 수 있다.

`directive` 들은 아래와 같다!


- ServerRoot : apache2의 파일 위치  
  * (default : /etc/apache2)
- Main DocumentRoot : HTML 파일이 위치한 곳
  * 보통은 /var/www/DOMAIN 으로 파일을 형성하여 관리한다. 이 파일의 위치를 넣는다.
- Main ErrorLog : error 파일이 위치한 곳
  * Main DocumentRoot와 동일하게 errorlog에 대하여 특정 파일을 생성해서 관리한다.
- Mutex : 모든 또는 지정된 뮤텍스에 대한 뮤텍스 메커니즘 및 잠금 파일 디렉토리를 구성
  * ( default : $ )
  * APACHE_RUN_DIR 변수에서 설정한 값을 불러오며 APACHE_RUN_DIR는 envvars 파일에서 설정되어 있다.
- DefaultRuntimeDir : 서버 런타임 파일들의 기본 디렉터리
  * ( default : $ )
  * APACHE_RUN_DIR 변수에서 설정한 값을 불러오며 APACHE_RUN_DIR는 envvars 파일에서 설정되어 있다.
- PidFile : 서버가 데몬의 프로세스 ID를 기록하는 파일
  * ( default : $ )
  * APACHE_RUN_DIR 변수에서 설정한 값을 불러오며 APACHE_RUN_DIR는 envvars 파일에서 설정되어 있다.
- TimeOut : 서버가 특정 이벤트를 기다리는 최대 시간(초)
  - ( default : 60)
- MaxKeepAliveRequests : KeepAlive가 켜져있을 때 지속적 연결 중에 연결 당 허용할 최대 요청 수
  - ( default : 100 )
- KeepAliveTimeout : 서버가 지속적 연결에 대한 후속 요청을 기다리는 시간
  - ( default : 5 )
- MaxRequestWorkers : 동시 처리 최대 연결 수
- MaxConnectionsPerChild : 개별 자식 서버 프로세스가 처리 할 연결 수에 대한 제한
- User : 서버가 요청에 응답할 사용자ID
  * ( default : $ )
  * APACHE_RUN_DIR 변수에서 설정한 값을 불러오며 APACHE_RUN_DIR는 envvars 파일에서 설정되어 있다.
- Group : 서버가 요청에 응답할 그룹 ID
- HostnameLookups : 클라이언트 IP 주소에 대한 DNS 조회 활성화
  - ( default : Off)
- ErrorLog : error log 가 쌓이는 곳
  - ( default : $/error.log)
  - APACHE_LOG_DIR 변수가 지정한 디렉터리 안의 error.log 파일에 기록한다.
- LogLevel : ErrorLog의 기록 수준 설정
  - ( default : warn)
  - emerg : 최고 위험 등급의 긴급 상황 발생 시
  - alert : 즉시 문제 해결 조치가 필요한 상황 발생 시
  - crit : 위 두 단계 정도까지는 아니지만 치명적인 시스템 문제 발생 시
  - error : 오류 발생 시
  - warn : 주의가 필요한 문제 발생 시
  - notice : 일반적이긴 하지만 꽤 중요한 상황 발생시
  - info : 서버 운용 상 필요한 정보 제공 필요 시
  - debug : 디버깅 작업 수준
  - trace1 : 추적 메시지
  - trace2 : 추적 메시지
  - trace3 : 추적 메시지
  - trace4 : 추적 메시지
  - trace5 : 추적 메시지
  - trace6 : 추적 메시지
  - trace7 : 대량의 데이터를 덤프하는 추적 메시지
  - trace8 : 대량의 데이터를 덤프하는 추적 메시지
  - 특정 파일에 대하여 설정 값을 변경가능 `<Directory>` 태그 사용하여
- IncludeOptional : 서버 구성 파일 외에 추가 서버 구성 파일 포함
- Include : 서버 구성 파일 외에 추가 서버 구성 파일 포함 
- LogFormat : 로그 파일에 사용할 형식을 설정 
  - ( default : "%h %l %u %t \"%r\" %>s %b" )
  - %h : 원격 호스트네임
  - %l : 연격 로그네임
  - %u : 요청이 인증된 원격 사용자
  - %t : 수신한 요청 시각
  - %r : 요청의 첫 줄
  - %s : 상태
  - %O : 헤더를 포함한 전송 바이트 수
  - %b : HTTP 헤더를 제외한 응답 크기 (바이트)
- AccessFileName : 분산 구성 파일 지정
- IncludeOptional : 서버 구동 시 추가 구성 지시문 활성화 
  - (default : conf-enabled/*.conf, sites-enabled/*.conf) 

정말 정말 많다..... 하지만 이것을 기록해놓고 나중에 필요할때 쓰자.. 후.. 힘들었다.

## ports.conf 설정

```linux
     1  # If you just change the port or add more ports here, you will likely also
     2	# have to change the VirtualHost statement in
     3	# /etc/apache2/sites-enabled/000-default.conf
     4	
     5	Listen 80
     6	
     7	<IfModule ssl_module> 
     8		Listen 443
     9	</IfModule>
    10	<IfModule mod_gnutls.c>    
    11      Listen 443
    12	</IfModule>
    13		
    14	
    15	
    16	# vim: syntax=apache ts=4 sw=4 sts=4 sr noet

```
보통은 이런식으로 설정이 되어져 있다.

>기타 태그가 없는 `Listen`으로 되어져 있는 것들은 클라이언트의 요청을 수신할 포트를 지정한다. 

위에서 설명했듯이 `sites-available`과 `sites-enabled` 파일에 의해 호스트에 따라
각기 다른 서버를 구성할 수 있다고 하였다. 즉 포트를 다르게 해도 동일하게 구성이 가능하다.

예를 들어 개발모드에서 사용하는 포트를 `999`라고 가정하고 실제 서비스를 `443`에서 구현할때,
각각 작동한다는 뜻이다. 이러한 개념은 매우 중요하며, 실제 배포 및 서비스 개발할때 필수 적으로 알아야 할 개념인 것 같다.

`Spring`에서도 개발모드, 배포모드를 다르게 설정해서 하는데 두 개를 한 서버에서 구동시킨다면
포트 분리를 통해 한 서버에서 동일하게 작업이 가능 할 것 같다는 생각이 든다.

> `<IfMoudel>` 태그의 경우 안에 특정 모듈에 대한 port를 설정할 수 있으며 위에서 얘기했듯이
각각 다른 기능을 수행 할 수 있다.

## sites-available ( VirtualHost ) 설정

드디어 가장 중요한 개념에 들어왔다! 반 왔다... 후 .. .달려보자 ~

`Apache`에서 가장 중요한 가상호스트를 설정하는 디렉토리이다.

가상호스트는 위에서 설명했듯이 한 개의 물리적인 서버에
여러 서비스를 `Apache`를 통해 운영 할 경우 사용하거나,
`Django`의 경우 `wsgi` 연동할 경우 사용한다.

그럼 `Django`를 `Apache`로 연동할때를 예시로 살펴보자.

만약 단일 서비스라면 `000-default.conf`에 수정하여 사용하면 되지만, 만약 여러개의 서비스를 해야한다면 site-available에 여려개의 `.conf` 파일을 만들어 심링크를 등록해주면 된다.

일단 각설하고 단일 서비스를 운용할때 `000-default.conf`로 설정하는 법을 살펴보자

```linux
<VirtualHost *: 지정할 포트(ex 999)>
        ServerName 도메인이름
        DocumentRoot /var/www/html
<Directory /home/djnago/backend/프로젝트이름/프로젝트이름>
        <Files wsgi.py>
               Require all granted
        </Files>
</Directory>

# admin페이지 css를 불러오기 위해 아래 경로 추가해줘야함
<Directory /home/django/venv/lib/python3.6/site-packages/django/contrib/admin/static>
        Require all granted
</Directory>

    WSGIDaemonProcess 도메인이름 python-path=/home/django/venv/lib/python3.6/site-packages
    WSGIProcessGroup 도메인이름
    WSGIScriptAlias / /home/django/backend/프로젝트이름/프로젝트이름/wsgi.py process-group=도메인이름
    
    # admin페이지 css를 불러오기 위해 아래 경로 추가해줘야함
    Alias /static /home/django/frontend/build/static
</VirtualHost>

```

이런식으로 설적을 하게된다. 즉 `<VirtualHost>`태그 안에 묶여, 여러 설정을 하게되는데
보통은 저정도만 하면 연동이 된다.


그렇다면 `.conf` 파일을 여러개 등록 하는 방법을 알아보자~~

먼저 `/etc/apache2/sites-available` 에 새로운 `.conf`파일을 만들어보자

```linux
    sudo vi /etc/apache2/sites-available/django.conf
    sudo vi /etc/apache2/sites-avialable/django-dev.conf
```

위 처럼 2개의 conf 파일을 만들었다고 가정하고


```linux
<VirtualHost *: 지정할 포트(django.conf -> 80 , django-def -> 9900)>
        ServerName 도메인이름
        DocumentRoot /var/www/html
<Directory /home/djnago/backend/프로젝트이름/프로젝트이름>
        <Files wsgi.py>
               Require all granted
        </Files>
</Directory>

# admin페이지 css를 불러오기 위해 아래 경로 추가해줘야함
<Directory /home/django/venv/lib/python3.6/site-packages/django/contrib/admin/static>
        Require all granted
</Directory>

    WSGIDaemonProcess 도메인이름 python-path=/home/django/venv/lib/python3.6/site-packages
    WSGIProcessGroup 도메인이름
    WSGIScriptAlias / /home/django/backend/프로젝트이름/프로젝트이름/wsgi.py process-group=도메인이름
    
    # admin페이지 css를 불러오기 위해 아래 경로 추가해줘야함
    Alias /static /home/django/frontend/build/static
</VirtualHost>

```

위에서 설정한 파일을 각각 포트 번호만 다르게 하여 저장을 한다.
이후 sites-available에 심볼릭 링크를 넣어주기 위해 아래와 같은 명령어를 실행하자

```linux
    # 새로작성한 .conf 파일 등록
    sudo /etc/apache2/sites-available/a2ensite django.conf
    sudo /etc/apache2/sites-available/a2ensite django-dev.conf
    # 000-default.conf 비활성화
    sudo /etc/apache2/sites-available/a2dissite 000-default.conf
```


이렇게 하게 되면 심볼릭 링크가 설정이 되면서 포트번호에 따라서 각각의 서비스로 진입이 가능하게 될것이다.

이후 사용한 포트에 대하여 방화벽 설정을 해주기 위해 아래명령어를 실행하자.

```linux
    sudo ufw allow (django에서 지정한 포트)/tcp
    sudo ufw allow (django-dev에서 지정한 포트)/tcp
    sudo ufw enable
    sudo ufw reload
```

방화벽 설정이 완료 되었다면, 재실행 시키면 작동된다.

```linux
 sudo service apache2 restart
```

아아아 이제 끝이 보인다 좀만 힘내보자 !

## mods-available 설정

`mods-available`은 위에서 설명했듯이 여러 모듈을 관리하게 된다.

`Apache`에서 기본적으로 내장되어져 있는 모듈은 아래와 같다.

```liux
  beom@wwww:/etc/apache2/mods-available# ls
  actions.load          authz_owner.load   dir.conf         log_forensic.load    proxy_ftp.load
  alias.load            authz_user.load    dir.load         mem_cache.conf       proxy_http.load
  asis.load             autoindex.load     disk_cache.conf  mem_cache.load       rewrite.load
  auth_basic.load       cache.load         disk_cache.load  mime.load            setenvif.load
  auth_digest.load      cern_meta.load     dump_io.load     mime_magic.conf      sick-hack-to-update-modules
  authn_alias.load      cgi.load           env.load         mime_magic.load      speling.load
  authn_anon.load       cgid.conf          expires.load     mod_python.load      ssl.conf
  authn_dbd.load        cgid.load          ext_filter.load  negotiation.load     ssl.load
  authn_dbm.load        charset_lite.load  file_cache.load  perl.load            status.load
  authn_default.load    dav.load           filter.load      php5.conf            suexec.load
  authn_file.load       dav_fs.conf        headers.load     php5.load            unique_id.load
  authnz_ldap.load      dav_fs.load        ident.load       proxy.conf           userdir.conf
  authz_dbm.load        dav_lock.load      imagemap.load    proxy.load           userdir.load
  authz_default.load    dbd.load           include.load     proxy_ajp.load       usertrack.load
  authz_groupfile.load  deflate.conf       info.load        proxy_balancer.load  version.load
  authz_host.load       deflate.load       ldap.load        proxy_connect.load   vhost_alias.load
```

정말 많은 모듈이 있고 어떻게 설정하느냐는 각 서비스에 맞게 설정을 하면 될 것 같다.

그 중에 이번에 예시로 다룰 것은 `userdir.conf`이다.

`userdir.conf`는 만약 처음 `Apache`를 install 했을때 기본 디렉토리가 `/var/www/html`로
되어져있을때, `Rootdir`을 새로 지정할 수가 있다.

먼저 아래 명령어를 사용하여 `userdir` 모듈을 활성화 한다.

```linux
    # userdir 모듈 활성화
    sudo a2enmod userdir 
    # apache2 재실행
    sudo systemctl stop apache2
    sudo systemctl start apache2
```

이전에는 `sudo service` 명령어로 재시작을 했다면,

설정파일을 변경 하였을때 `systemctl`을 사용하여 재시작 해야한다고 한다.

정확한 이유는 `Forground`와 `Background`의 차이인것으로 추정만 된다.

나중에 시간될때 확인해 봐야겠다....

그럼 `userdir.conf` 파일을 확인해 보자.

```linux
<IfModule mod_userdir.c>
    UserDir public_html // 사용자 웹서버 루트디렉터리 설정
    UserDir disable root // 루트사용자 홈 디렉터리에 대한 접근 차단

    <Directory /home/*/public_html> // 사용자 웹서버 루트디렉터리의 기본설정

<IfModule mod_userdir.c>
    UserDir public_html // 사용자 웹서버 루트디렉터리 설정
    UserDir disable root // 루트사용자 홈 디렉터리에 대한 접근 차단

        <Directory /home/*/public_html> // 사용자 웹서버 루트디렉터리의 기본설정
             AllowOverride FileInfo AuthConfig Limit Indexes
             Options MultiViews Indexes SymLinksIfOwnerMatch IncludesNoExec
             Require method GET POST OPTIONS
        </Directory>
</IfModule>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
```

이렇게 `UserDir`에 새로운 경로를 만들어주고
`UserDir disable root`를 사용하여 루트사용자에 대한 접근 권한을 막음으로서 보안성을 높인다.


이렇게 여러방법이 있고, 정말 많은 설정을 `Apache`에선 할 수 있다.

긴 길이었지만 일단 오늘은 이만!



























