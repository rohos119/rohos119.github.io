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




## 방화벽 세팅
---

기본적으로 컴퓨터는 `Packet`을 통해서 통신을 한다.
이때 방화벽을 사용해 들어오는 `Packet` 또는 `Protocol`을 제한 또는 허용할 수 있는데,
방화벽 기능 활용하는곳은 매우 많다.

`Router`에서도 방화벽을 사용하고, `local`에서도 방화벽을 사용하며 다양하게 구성이 가능하다.

이를 구현하기 위해선 여러가지 방법이 있는데 대표적으론 `iptables`, `firewlld`, `ufw`가 있다.


`iptables`, `firewalld`는 linux기반의 모든 OS에서 사용이 가능하고, 통상적으로 `firewalld`는
기본설치가 되어져 있다. 
또한 보통 명령어를 사용하여 `CLI` 기반으로 하기가 힘드니, `GUI`를 제공하는게 일반적이다.
또한 `linux`기반에서는 그 어떤 패키지도 커널 `netfilter` 프레임워크 위에서 동작하게 된다.

즉, 방화벽 설정은 `netfilter` 모듈에 의해 결정되는 것이며, 두 패키지 모두 `netfilter`를 제어하는 것이다.

이전에는 `iptables`를 사용하였지만 `systemd`와 함께 `firewalld`가 출시되면서,
대체되어 지고있다. 그 가장 큰 이유는 `iptables`는 방화벽 규칙을 변경할 때마다 서비스를 중지 후 재시작 해야하기 때문이다. 그렇기 때문에 `OpenStack`이나 `가상화`같은 환경에서는 제약이 많다.

그럼 각각에 사용법과 설정방법을 알아보도록 하자~

자 오늘도 열심히 달려보자~~~~

### firewalld 설정  
---

`firewalld`은 IPv4, IPv6, 이더넷 브리지 및 IPSet의 방화벽(firewall) 설정을 지원하는 리눅스(linux) firewall 관리 도구이며, `linux` 커널 `netfilter` 프레임워크의 프론트엔드 역할을 하는 RHEL 7 제품군의 기본 firewall 관리 소프트웨어이다.

`ubuntu`의 기본 방화벽 시스템은 `ufw`(uncomplicated firewall)이지만, `firewalld`를 설치하여 사용할 수 있다.

이 포스트에서는 `ubuntu` 환경에서 패키지로 `firewalld`를 설치하고 설정하는 방법을 소개하겠다.


> firewalld 와 iptables를 동시에 사용하게 될경우 충돌이 나기때문에 둘중 하나만 사용해야 한다!

#### firewalld 특징 

    1. firewalld는 Runtime(실행중)설정과 Permanet(영구) 설정이 있다.
        - 즉, 말그대로 Runtime 설정을 하면 실행하는 중에만 사용, Permanet는 부팅시 설정되게끔
    2. Pre-defined Zone과 Default Zone이 있다.
        - Zone 이라고 표현하는 영역을 설정하여 관리되어 진다.
        - 각 영역별로 설정을 하면되고 영역들에 대한 설명은 아래와 같다
            - block : 모든패킷 거부, 내부->외부 반환 패킷은 허용
            - dmz : 외부->내부 패킷 거부, 외부 연결은 허용(ex ssh) 
            - drop : 외부->내부 모든 폐킷 폐기, ICMP error도 폐기, 외부로 연결만 허용
            - external : dmz와 동일하나 IP 마스커레이팅 활성화
            - home : 외부->내부 패킷 거부, dhcpv6-client,ipp-client,mdns,samba-cliet,ssh 허용
            - internal : home과 동일
            - public(Default Zone) : 외부->내부 패킷 거부, ssh,dhcpv6-client허용
            - trusted : 모든 패킷 허용
            - work : 외부->내부 패킷 거부, dhcpv6-client,ipp-client,shh 허용
  
위에 정리 한것 처럼 `firewalld`는 `Zone`이라 명명한 카테고리를 만들고,

그 카테고리를 `Permanet` or `Runtime` 이라는 옵션을 추가하여 설정을 적용한다.
각각의 `network interface` 마다 설정을 하게끔 되어있으며, 기본적으로는 `Default zone`인 `public` 으로 설정이 되어진다.

```linux
    sudo firewall-cmd --get-active-zones
```
위 명령어를 사용하면 아래와 같은 결과가 나올것이다.
![이미지](/assets/images/firewall_get_active_zone.PNG)

보면 `network interface`인 enp0s3, enp0s8에 각각 default인 home이 들어가져 있다.




#### firewalld 설치
---

현재 `ubuntu` 의 vm을 사용하고 있기에 먼저 설치를 하고 버전을 확인해보자

```linux
    sudo apt install firewalld
    sudo firewall-cmd --version
    # 0.4.4.5
```
설치를 하게되면 `firewalld`는 `/usr/lib/firewalld`와 `/etc/firewalld`에 설치가 된다

`/usr/lib/firewalld`에는 기본 구성, `/etc/firewalld`에는 설정파일과 규칙이 저장되어 있다.

`firewalld`는 `CLI` 기반인 `firewall-cmd`가 있고 `friewall-config`가 있다.

내장이 되어있을 경우에는 전부 설치가 되어 있지만, 현재 `ubuntu`에서는 firewall-config를 따로 설치를 해야한다.

```linux
    # 패키지 다운로드
    sudo apt install firewall-config
    # firewall-config 실행
    sudo firewall-config
```

위 명령어를 실행하게 되면 아래와 같은 GUI가 나올 것이다.
![이미지](/assets/images/firewall-config.PNG)

이런식으로 설치해서 사용하게 되면 아주 편하게 `PortForwading`과 여러 설정들을 할 수 있다.

그렇지만 `firewall-cmd` 또한 알아 두어야 하기에 명령어 사용방법과 특성에 대해 알아보자.


#### firewall-cmd 사용법
---

`friewall-cmd`의 명령어와 간단한 예시이다.

```linux
    #방화벽 상태 
    firewall-cmd --state 

    #방화벽 리로드 / 설정 변경 후엔 꼭 리로드 
    firewall-cmd --reload 

    #설정된 목록 
    firewall-cmd --list-all 

    #포트 추가/제거 --permanent 설정 유지 속성 and 서비스 추가 
    firewall-cmd --permanent --add-service=ftp #서비스 삭제 firewall-cmd --permanent --remove-service=ftp # 포트 추가 firewall-cmd --permanent --add-port=21/tcp #포트 삭제 firewall-cmd --permanent --remove-port=21/tcp 

    #아이피 허용 
    firewall-cmd --add-source=192.168.1.1 

    #아이피 / 포트 허용 firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address=192.168.1.1 port port="80" protocol="tcp" accept' 
     
    #포트 범위로 열기
    firewall-cmd --permanent --add-port=8000-8100/tcp
```

명령어를 사용하는 방법은 정말 다양하고, 사실 다 기억하지 못한다....

이런게 있다 정도만 알고 있고 필요할때 찾아서 쓰자.....


### iptables 설정
---

`iptables`의 간단한 명령어만 정리하고, 왠만하면 쓰지말자...

```liunx
    sudo iptables -L (현재 적용 룰 확인)
    sudo iptables -F (초기화)
    sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT  (tcp/22 port 허용)
    sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT  (tcp/80 port 허용)
    sudo iptables -A INPUT -p udp --dport 1024 -j ACCEPT  (udp/1024 port 허용)


    sudo iptables -A INPUT -p tcp --dport 21 -j DROP  (tcp/21 port 거부)
    sudo iptables -A INPUT -p udp --dport 53 -j DROP  (udp/53 port 거부)
```


### ufw 설정
---

`ufw`를 가장 많이 사용하는데, 이미 익숙한 명령어이니 그냥 정리용~~

```linux
    sudo apt-get install ufw (설치)
    sudo ufw reset (초기화)

    sudo ufw enable ( 원격 접속 시 SSH 포트를 열고 재부팅 필요 - sudo ufw allow ssh)

    sudo ufw status (상태확인)

    sudo ufw allow 22/tcp (tcp/22 port 허용)
    sudo ufw allow 80/udp (udp/80 port 허용)
    sudo ufw allow form 192.168.1/24(주소 허용 서브넷마스크: 24)

    sudo ufw deny 53/tcp (tcp/53 port 거부)
    sudo ufw deny 21/tcp (tcp/21 port 거부)

```


자 오늘도 열심히 달려왔다. 사실 빌드하고 진짜 짜증나는 부분이 이 포트포워딩이였다

말도 안되게 로컬에선 잘되다가 서버에만 올리면 안되고... 진짜 속상하고 너무 열받았는데

그래도 뭔가 정리하니 속이 쉬원해진다 ~~~




































