---
layout: post
title:  "[DevOps] Linux 부시기 (DNS)"
date:   2022-07-09 08:56:53 +0900
categories: DevOps
tag: Linux Ubuntu
---

### 목차

1. ~~~`SSH`와 `FTP` 세팅 원리와 이해~~~
2. ~~~`Port forwarding` 세팅 원리와 이해~~~
3. ~~~`Linux` 에서 `Forground` 와 `Background`의 이해~~~
4. `DNS` 세팅 원리와 이해
5. `Apache` 세팅 원리와 이해
6. `Mail server` 세팅 원리와 이해


자 이전 포스팅과 이어서 `DNS` 세팅과 원리를 이해해보자 

자 오늘도 열심히 달려보자~

---

# DNS 세팅과 원리
`DNS`란 `Domain name server`로 흔히 우리가 아는 www.google.com과 같은
글자로 표현된 도메인 서버들에 대한 맵핑을 해주는 서버이다.

이전에는 도메인이 많이 없어서 로컬에 `HOST`로 그냥 맵핑에서 쓰면됬지만,
사용량이 많아짐에 따라 더 이상 로컬에서 전부 저장할 수 없어 `DNS`가 등장하게 되었다.

`DNS`를 질의하는 클라이언트에서는 아래와 같은 방식으로 동작을 하게 되는데

![이미지](/assets/images/dns_%ED%9D%90%EB%A6%84.PNG)

이때 알아둬야할 개념은 아래와 같다.

    - DNS server
    - name server
    - hosts

`DNS server`는 도메인(domain) 이름과 관련 기록을 관리, 유지, 처리하는 name server의 일종이며, DNS 쿼리(query)에 응답하는 server이다.

`name server`는 네이밍(naming) 서비스를 제공하는 모든 server인데, 일반적으로 로컬 DNS server로 알려져 있으며, DNS server를 찾는 데 사용된다.

`name server`는 domain 이름을 IP 주소(address)로 변환하며, 이를 통해 사용자는 웹 사이트의 실제 IP address 대신 domain 이름을 입력하여 웹 사이트에 액세스할 수 있다.

DNS는 `호스트(host)` 이름을 IP address로 변환하는 분산 시스템이다.

즉, `DNS server`는 기본적으로 `name server` 유형이지만, 모든 `name server`가 `DNS server`인 것은 아니다.

그리고 `DNS server`가 가장 일반적인 `name server`이므로, `name server`와 `DNS server`는 서로 바꿔 사용할 수 있다.

" 그렇다면 클라이언트에서, 질의를 했을때 이미 사전에 등록되어져 있는 도메인들은
어떤 과정을 거쳐 응답을 반환할까?" 에 대한 의문이 들었고

구글링을 통해 아래와 같은 방식으로 작동 한다는 것을 찾았다.

![이미지](/assets/images/dns_ms_1.png)

과정을 간략하게 설명하자면

    Root DNS에 처음 들어간다 -> 사전 설정된 것을 통해 TLD DNS로 진입한다
    TLD DNS에서는 도메인 도메인 대행업체에 대한 응답을보내고 -> 도메인 대행업체로 들어간다
    도메인 대행업체는 질의한 query에 대한 응답을 보낸다

이러한 과정을 거치는데 `Root DNS`와 `TLD DNS`는 상위기관에서 관리하고,
도메인 대행 업체인 `Authoritative DNS`에서는 query에 대한 ip를 보낸다

그렇다면 현실적으로 개발자로서 개발을 할때 고려할 사항은

`Authoritatives DNS`에 대한 내용이다. 이들이 어떻게 구성이 되어져 있으며
어떤 방식으로 구현하게 되는지를 알게되면 좀 더 큰 서비스를 하고있는 `Naver`와 `Kakao`와 같은 곳의 구성도 어렴풋이 이해할 수 있다는 생각이 들었다.


그래서 지금부터는 `Authoritiatives DNS` (이하 도메인서비스업체)에서 구성하는 방식을 알아볼까한다.

자 그럼 달려보자~~~

## Master-Slave 구조

---

![이미지](/assets/images/dns_ms_2.PNG)

전형적인 Master-Slave 구조의 `DNS server`이다.

`DNS`질의의 클라이언트는 `Master`이든 `Slave`이든
모두 응답을 받을 수있다.

그렇다면 왜 `Master`와 `Slave`로 구분을 해놓은 것일까?


그 이유는 `Master`에서 위에 보이는 `DNS#2`,`DNS#3`의 `Zone file`를 통합관리하여 각각의 서버에 업데이트 하기 위함이다.

즉, `Master`만 관리하여 나머지를 유지하겠다는 관점이다.


서버 관리자가 `Master`인 DNS #1 번의 `Zone file`을 업데이트 하였을때 `Slave`인  #2,#3은 
`AXFR/IXFR`이라는 방식으로 새로운 `Zone file`을 `Master`로부터 업데이트 한다.


`AXFR`은 모든 `Zone file`을 가져가는 방식이고,

`IXFR`은 수정된 업데이트 부분만 가져가는 방식이다.

`AXFR/IXFR`또한 `53`포트를 사용한다.


그렇다면 `Slave`는 어떻게 `Master`가 업데이트 한것을 알수 있을까?

다음과 같은 방식으로 확인한다.

### SOA 레코드 기반
---

나중에 `Zone file`에 대하여 자세히 다룰 거기에
간략하게만 다뤄보자.

`Master`에서는 `Zone file`업데이트시 아래와 같은 과정을 지켜야한다.

1. `Master`는 업데이트시 `Serial`값을 기존보다 큰 값으로 업데이트
2. `Slave`는 `Refresh`에 설정된 주기마다 `Master`에게 `Zone file`을 조회한다.
3. `Slave`가 조회시 자신의 `Serial`보다 크다면, 업데이트 되었다 간주한다.
    * 만약 `Master`에게 응답이 없으면, `Retry`에 설정한 주기만큼 재실행
    * `Expire` 시간을 초과하면 더 이상 요청을 하지 않고 사용자들에게 응답하지 않는다.
4. `AXFR/IXFR` 요청으로 새로운 `Zone file`을 받아온다.


이런 과정을 거쳐 `SOA RECORD` 기반은 작동하게된다.

그러나 위 과정을 보면 알겠듯이, `Refresh`만큼 지연시간이 발생한다.

그렇기 때문에 `Notify`라는 개념이 등장했다.


### NOTIFY 기반
---

`SOA`는 `Zone file`에서 설정하는 것이였다면, 

`Notify`는 `options`에서 설정을 하게 된다.

`Notify`는 `Master`에서 `Zone file`을 업데이트 하면 모든 `Slave`에게 `Notify` 메세지를 발송하여 새로운 `Zone file`을 받아오게 한다.

이렇게 한다면 `Zone file`의 동기화 문제를 해결할 수 있다.


## Hidden-Master 구조
---

위와 같이 `Master`도 직접 클라이언트에게 질의하며,

필드를 뛰고 있다면 `Hidden-Master`구조는 비선실세이다.


즉, `DNS client`는 `Master`의 존재를 알수 없고
`Slave`만 알 수 있으며 작동원리는 `Master-salve`구조와 동일하다.


그림으로 표현하면 아래와 같다.

![이미지](/assets/images/dns_ms_3.png)


그렇다면 위에서 부터 쭉 설명해온 `Zone file`이 도대체 무엇인지, `options`가 무엇인지 알아보도록 하자



## bind9 세팅
---

`bind9`은 `Liux`기반에서 제공하는 `DNS` 관리 패키지 이다.


먼저 설치를 해보자
```linux
    # bind9 pacakge 설치
    sudo apt install bind9
    # bind9 servcie 실행 후 작동상태 확인
    sudo service bind9 start&sudo service bind9 status
```
위 명령어를 실행해서 아래와 같은 화면이 나오면
정상적으로 실행이 된것이다.


![이미지](/assets/images/bind9_start.PNG)

`bind9`은 `DNS`를 다루기에 `53` 포트를 열어주어야 한다.

```linux
    sudo ufw allow 53/tcp
    sudo ufw enable
    sudo ufw reload
```

자 그럼 `bind9`의 구성을 살펴보자

먼저 `bind9`의 설정파일은 `/etc/bind`에 있다.

![이미지](/assets/images/bind_repo.PNG)

위와 같은 구조로 되어있다.

`bind9`은 기본적으로 `named.conf` 파일에 의해 동작한다

![이미지](/assets/images/bind_name_confg.PNG)

default 설정을 보면 `name.conf.options`,`named.conf.local`,`named.conf.default-zones`가 삽입되고 설정이 되는것을 볼수 있다.

즉, `named.conf`이 메인문이 되는것이고 여러 서브 설정들을 `include` 할수 있다.


#### named.conf 파일 옵션
---

자 그럼 named.conf 파일에 설정하는 옵션들을 알아보자

- BIND9 데몬의 listen 정보를 지정 (인터페이스 주소, 포트 번호 등)

```
    listen-on port 53 { 옵션; }; // IPv4
    listen-on-v6 port 53 { 옵션; }; //IPv6

    옵션) any / none / 단일 IP주소 / CIDR
    ex) listen-on port 53 { 127.0.0.1; }; listen-on-v6 port 53 { ::1; };
    ex) listen-on port 11111 { any; }; listen-on-v6 port 53 { any; };
```

> 만일 53번이 아니라 타 포트를 사용할 경우, 여기서 포트 번호를 변경

> Public한 서비스의 경우 일반적으로 any를 기재한다.


- DNS 서비스를 할 Client 대역을 지정
```
    allow-query { 옵션; };

    옵션) any / none / 단일 IP주소 / CIDR
    ex) allow-query { any; };
    ex) allow-query { 192.168.0.0/16; 10.0.0.0/8 };
    ex) allow-query { 127.0.0.1; 1.2.3.4; 5.6.7.8; 9.10.11.12; }
```
> Public한 서비스인 경우 any를 기재
> 특정 Client에게 제공하는 서비스인 경우 CIDR 혹은 단일 IP주소를 기재

- recursiv DNS 서버 or authoritative DNS 서버 설정
```
    recursion 옵션;

    옵션) yes / no
    ex) recursion no;

    yes -> 해당 DNS 서버를 recursive DNS 서버로 설정할 경우에 사용 (cache DNS server)
    no -> 해당 DNS 서버를 authoritative DNS 서버로 설정할 경우 사용 (일반적)

    주의) 만일 Public 서비스를 제공할 경우, 반드시 no를 선택할 필요가 있다. (매우 중요)
    DNS의 경우 scanning이 워낙 빈번하기 때문에,
    만일 실수로 여기서 yes를 했다가는 전 세계 수많은 PC들이 해당 네임서버에 자신의 도메인 네임과는 
    전혀 관계없는 내용을 마구 질의할 것이다.
    심지어, 해당 서버가 DNS 관련 공격에 연관될 수도 있다. (직접 당하든 Relay 되든)

    잘 모르면 no를 선택한다. (반드시!!!!!!!)
    yes는 DNS 시스템과 BIND9를 잘 이해한 다음에 사용할 필요가 있다.
```

- 해당 DNS 서버에서 DNSSEC을 사용할 지 선택
```
    dnssec-enable 옵션;
    dnssec-validation 옵션;

    옵션) yes / no 잘 모르면 yes 선택 추천
    ex) dnssec-enable yes;
    ex) dnssec-validation yes;
```

- Zone-Transfer를 허용할지 정하는 옵션(Master - Slave 구성시)
```
    allow-transfer { 옵션; };

    옵션) any / none / 단일 IP주소 / CIDR
    ex) allow-transfer {none;};
    
    Zone-Transfer는 Master-Slave 구성 외에는 사용할 일이 사실상 없다. 
    자신의 Zone File 원본을 밖으로 유출되기를 원하는 서버관리자는 없을 것이다. 
    자신의 Zone File을 보호하기 위해 none을 선택하는 것을 강추한다.
    만일 Master-Slave 구성을 할 경우(Zone-Transfer가 필요한 경우), 
    여기가 아닌 하단의 /etc/named.rfc1912.zones 파일에서 Zone-Transfer 설정을 할 것을 추천한다.

    잘 모르면 none을 선택한다. (매우 권장함)
```


- 현 BIND9의 버전을 감추는 옵션

```
    version "UNKNOWN";

    ex) version "UNKNOWN";

    BIND9의 버전이 밖에 노출될 경우 좋을 것이 하나도 없다.
    (참고로, dig version.bind CHAOS TXT @IP주소 명령어로 외부에서 BIND9 버전 조회 시도를 할 수 있다)
```

- DNS NOTIFY에 대한 옵션 (Master - Slave 구성시에 주로 사용)
```
    notify 옵션;

    옵션) yes / no / explicit / master-only
    (선택사항) also-notify { 단일 IP주소 / CIDR };
    ex) notify yes; // 일반적인 구성
    ex) also-notify { 1.2.3.4; }; notify yes; // Zone의 NS List + 1.2.3.4에게 NOTIFY 발송
    ex) also-notify { 1.2.3.4; 5.6.7.8; }; notify explicit; // 1.2.3.4, 5.6.7.8에게만 NOTIFY 발송
    ex) notify no; // NOTIFY 미발송


    yes (기본값) 선택시, Zone의 NS 리스트에 있는 주소 + also-notify에 기재한 주소로 NOTIFY 메시지를 발송한다.
    단, SOA 레코드의 MNAME Field(주로 Primary Name Server)에 있는 주소로는 NOTIFY를 발송하지 않는다.
    no 선택시, NOFITY 메시지는 발송되지 않는다.
    master-only 선택시, 본인이 Master인 Zone File만 NOTIFY 메시지를 발송한다.
    explicit 선택시, 사용자가 also-notify로 지정한 리스트에게만 NOTIFY 메시지가 발송된다.

    (잘 모르면 yes를 선택하는 것을 추천)
    (주로 Hidden-Master 설정시 explicit 옵션을 사용한다)
```

- BIND의 응답 최대 한도를 걸어, 한도 이상의 요청이 오면 그 요청을 제한(일반적으로 DROP)하게 하는 조치

```
    rate-limit 

    해당 포스팅 참고
    https://dev.dwer.kr/2020/03/bind-9-rate-limiting.html
```

### Zone File 옵션

`Zone file`은 `named.conf.default-zones`은 그대로 두고 보통 `DOMAIN.zone`으로 생성한다.

예시를 들자면 `exampe.con.zone`으로 설정을 한다.

먼저 작성시 규칙 사항이다.

  - 모든 시간의 단위는 초(second) 이다.
  단, M(Minute), H(Hour), D(Day), W(Week) 등을 사용할 수 있다.

    ex) 300 -> 300초 == 5분
    
    10M -> 10분
    
    2H -> 2시간
    
    3D -> 3일
    
    4W -> 4주

  - 주석은 세미콜론(;) 으로 한다.

  - 호스트명 기재시, 맨 뒤에 점(.)을 꼭 붙여야 한다.
  붙이지 않으면 {기재한 내용 + ORIGIN}으로 간주된다.

    ex) ORIGIN이 example.com.일때,

    www -> 실제로 www.example.com. 으로 작동
    
    example.com -> 실제로 example.com.example.com. 으로 작동
    
    www.example.com. -> 실제로 www.example.com. 으로 작동

  - 만일 호스트명으로 공백을 기재하려는 경우, @ 을 기재한다.
    
    ex) ORIGIN이 example.com.일때, @ -> example.com. 으로 작동

  - Zone Apex 경우에만 CNAME 레코드를 사용할 수 없다. 반드시 CNAME 레코드 대신 A 레코드를 사용해야 한다.
  타 레코드에서는 타 host명을 기재해도 된다.
  (즉, CNAME은 root domain에서 사용할 수 없다)

    ex) ORIGIN이 example.com.일때,

    @    300    IN    CNAME    host.com.; 
    
    올바르지 않은 문법
    
    @    300    IN    A            1.2.3.4 ; 
    
    옳은 문법
    
    @    300    IN    MX          mail.out.com. ; 
    옳은 
    문법

  - 레코드에 별도의 시간을 명시하지 않은 경우 $TTL을 따른다.

  - 만일 NS의 호스트명이 자신의 $ORIGIN 호스트명에 속해있을 경우,
  해당 호스트명에 해당하는 A레코드가 명시적으로 해당 Zone File에 기재되어야 한다.

    ex) ORIGIN이 example.com. 이고 네임서버가(NS 레코드가) ns1.example.com. 인 경우,

    ns1    IN    A    10.20.30.40
    
    이 반드시 Zone File에 있어야 한다.

그렇다면 아래 예시를 통해서, 실제 옵션들이 의미하는 것이 무엇인지 살펴보자

![이미지](/assets/images/dns_example.png)

```
    TTL: Time to live. 
    캐시에 zone 정보가 남아있는 데이터 유효 기간
    
    @: root 도메인
    
    SOA Record: zone에 대한 필수적인 정보를 가지고 있는 레코드
    
    NS Record: 해당 도메인의 IP 주소를 찾기 위해 가야 할 네임 서버 정보를 담고 있는 레코드
    
    A Record: 호스트 네임에 주어진 도메인에 매핑되는 IPv4 형식의 IP 주소를 저장하는 레코드
    
    AAAA Record: 호스트 네임에 주어진 도메인에 매핑되는 IPv6 형식의 IP 주소를 저장하는 레코드

    @ IN SOA ns.bind9.kr. master.yes.kr.
    ns.bind9.kr. : 해당 zone의 마스터 네임 서버
    -> master.yes.kr. : zone 관리자 연락처
    관리자 연락처의 경우에는 전자 메일 주소로, 첫 번째 .(온점)이 @로 대치됩니다. 예시의 경우에는 전자 메일 주소가 master@yes.kr 이 되는 셈입니다.
    
    CNAME Record: 도메인 네임(alias name)을 다른 이름(canonical name)으로 매핑시키는 레코드

```

자 이상 해당 옵션들의 의미와 설정파일들

그리고 원리에 대해 알아보았다. 

다음포스팅에서는 실제 `virtual box`를 활용하여,
`Master`서버와 `Slave`서버를 만들고

차후에 `Apache`에 도메인을 사용하여 접속할 수 있도록 설정할 예정이다.


오늘은 이만~ 그만달리자 힘들다.....

























