---
layout: post
title:  "[DevOps] Linux 부시기 (Master-Slave 만들기)"
date:   2022-07-12 08:56:53 +0900
categories: DevOps
tag: Linux Ubuntu
---

### 목표

1. VM 3개를 생성하여, Master-Slave 구조를 만든다
2. 설정된 Master-Slave 구조가 동작하는지 확인한다.
    - Master의 Zone file SOA record 방법
    - Master의 Notify 기능 구현
3. Master-Slave 구조를 -> Hidden-Master 구조로 변경

---

# 작동 원리 설명

먼저 VM -> Virtual Box Vm의 구조를 살펴보자

VM은 현재 PC에 여러 가상 OS를 올려 구동하게 되는데, 이때 `Network`는 현재 PC를 통해 구동이 되게 되어있다.

때문에 가상 사설 IP를 사용하게되고, `Virtual Box`는 현재 PC에 네트워크가 연결되어 있게 된다.


`Virtual Box`에 `IP`는 `Gateway`의 역할을 하게되고, 나머지 `IP`들은 할당을 해줘야 한다.


그렇기 때문에 `DHCP`기능을 활성화 하여,
`VM` 이 추가될때마다 `IP`를 자동으로 설정 되게 하게끔 하여야 충돌이 나지 않는다.

구조는 아래 이미지를 참고하자~

![이미지](/assets/images/dns_practice.PNG)


## DHCP 설정하기

---

먼저 Virtaul Box의 메인 창에서

`파일 > 호스트네트워크관리자`를 클릭하면
아래와 같은 화면이 나올것이다.

![이미지](/assets/images/dhcp_vm.PNG)

위에 보이는 `DHCP 사용함`, `수동으로 어뎁터 설정`을 하면 기본 설정이 될것이다.

이후 , 
    
    1. DHCP 서버(D) 를 클릭
    2. 형광펜 표시된 부분 수정
        - 서버주소 : DHCP를 활용할 주소
        - 서브넷마스크 : 마스킹 할 값
        - 최저 주소 한계 : DHCP할당 시작값
        - 최대 주소 한계 : DHCP할당 최대값

위 부분을 수정을 한다.

![이미지](/assets/images/dhcp_vm2.PNG)

이렇게 할경우

DHCP 서버주소 = 192.168.56.100을 활용하여,

192.168.56.101~254까지 IP를 할당한다는 설정이다.

### VM 내부 설정
---

위와 같이 실행후에, `VM`에 접속을 해서

`Network`를 보게 되면 아래와 같이 두개의 `Network interface` 가 설정이 되어있을 것이다.

![이미지](/assets/images/dhcp_ifconfig.PNG)

이때, 이미 `DHCP`를 활용하였기 때문에 자동적으로 `IP`가 할당이 되어 있을 것이다.

그러나, `DNS server`로서 사용하고 위해

`enp0s8`에 대하여 설정을 변경하여 고정`IP`를 부여하여 사용해야 한다.

그렇다면 고정 IP를 활용할껀데 왜 `DHCP`를 사용했는가? 

이유는 `192.168.56.0/24` 대역으로 변경하기 위함이다.

![이미지](/assets/images/dhcp_network.PNG)

위에 보이는 네트워크 표시를 눌러서 setting으로
들어가자, 들어가서 `enp0s8`에 대한 설정 표시를 눌러 설정을 하자

![이미지](/assets/images/dhcp_network2.PNG)

위 이미지처럼 원하는 `IP`,`NetMask`,`Gateway` 설정을 한다.

이후 다시 아래화면의 `toogle`을 클릭해 껏다 다시킨다.

![이미지](/assets/images/dhcp_network.PNG)

사실 이미 나는 설정을 해놓고 포스팅을 하는 상태라... 이해해주길 바란다...ㅎ


자 그럼 `DHCP` 설정도 완료했고, 본격적으로

`DNS server` 세팅을 시작해보자

`DNS server`를 만들기 위해선 아래와 같은 파일을 수정해야한다.

    1. resolv.conf
    2. hostname 수정
    3. Zone file 수정
    4. options 파일 수정
    5. 방화벽 설정


## resolv.conf 수정 
---

자 처음 resolv.conf 파일을 열면 아래와 같이 나온다.

![이미지](/assets/images/dns_resolv_conf.PNG)

우리는 `Masetr` server을 만들기때문에
`nameserver`가 현재 PC가 되어야 한다

때문에 `nameserver`에 `127.0.0.1`을 추가한다.

![이미지](/assets/images/dns_resolv_conf2.PNG)

이때 `options`는 어떤 network interface로 통신을 할건지를 나타내고

`search`는 `ROOT DNS`나 `TLD dns`를 의미하며 본인 `hosts`에 없는 것을 질의하는 곳을 의미한다.

그렇다면, 우리는 `Master`서버를 만들것이기 때문에 결국에 질의를 보내주는 쪽이 되어야 하고, `nslookup`을 실행했을때 `server`에는 현재 `IP`가 뜨게금 하는것이 목표이다.

예를들어 `local dns`를 활용하는 `hosts`를 수정해서 테스트를 해보면

![이미지](/assets/images/dns_hosts.PNG)

이렇게 `Server:127.0.0.53`을 활용해서 query에 대하여 answer하는것을 확인 할 수 있다.

위에서 말했듯 우리의 목표는

저 `Server : 192.168.56.101`이 되는것이 목표이다.

## hostname 수정
---

`hostname`은 현재 PC의 이름을 뜻한다

```linux
    sudo vi /etc/hostname

    ## 아래내용 입력
    ns.beomseokmaster.kr
```

이렇게 현재 PC를 ns.beomseokmaster.kr로
바꾸었고 
> 설정이 되기 위해선 VM을 재부팅을 해야한다 

![이미지](/assets/images/dns_hostname2.PNG)

이렇게 되면 잘 설정이 된것이다.


## Zone file 수정
---

먼저 `named.conf.default-zones`를 수정을 해서 사용할 건지, `Zone file`을 새로 생성해서 사용할건지

결정을 해야하는데... 나는 그냥

`named.conf.default-zones`를 수정해서 하기로 했다.


```linux
    sudo vi /etc/bind/named.conf.default-zones
```

위 명령어를 실행해서 파일을 열어보면 아래와 같다.

![이미지](/assets/images/dns_default_zones.PNG)

이 가장 밑에 아래와 같이 추가한다.

```linux
zone "beomseokmaster.kr"{
    type master; # 마스터 dns로 설정
    file "/etc/bind/db.master.zone"; # 적용할 zone file
}
```

이렇게 설정을 하면 사용할 `Zone file`을 `db.master.zone`으로

설정을 한 것이기 때문에, `db.master.zone`파일을 새로 만들어 주어야 한다.


```linux
    sudo vi /etc/bind/db.master.zone
```
을 실행후에 아래와 같이 작성한다.

![이미지](/assets/images/dns_zone_file.PNG)

위 파일의 의미는 아래와 같이 맵핑 한다는 뜻이다.

    dns1.beomseokmaster.kr -> 192.168.56.102
    dns2.beomseokmaster.kr -> 192.168.56.103
    www.beomseokmaster.kr -> dns1.beomseokmaster.kr -> 192.168.56.102
    test.beomseokmaster.kr -> 1.1.1.1
    dev.beomseokmaster.kr -> 2.2.2.2

### options 파일 수정










