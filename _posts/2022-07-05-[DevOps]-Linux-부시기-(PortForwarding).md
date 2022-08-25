---
layout: post
title:  "[DevOps] Linux 부시기 (PortForwarding)"
date:   2022-07-05 08:56:53 +0900
categories: DevOps
tag: Linux Ubuntu
---

### 목차

1. ~~~`SSH`와 `FTP` 세팅 원리와 이해~~~
2. `Port forwarding` 세팅 원리와 이해
3. `Linux` 에서 `Forground` 와 `Background`의 이해
4. `DNS` 세팅 원리와 이해
5. `Apache` 세팅 원리와 이해
6. `Mail server` 세팅 원리와 이해


자 이전 포스팅과 이어서 `Port forwarding` 세팅과 원리를 이해해보자 

자 오늘도 열심히 달려보자~

---

# Port forwarding 세팅과 원리

`Port forwarding` 이란 어느 한 포트로 들어온 요청을 다른 포트로 전달해주는 것을 말한다.

이를 이해하기 위해서 `Port`의 개념을 이해해야 하는데

`Port`란 IP 내에서 애플리케이션 상호 구분(프로세스 구분)을 위해 사용하는 번호이며,
포트 숫자는 IP 주소가 가리키는 PC에 접속할 수 있는 통로(채널)을 의미한다.
터미널에서 `React`를 실행하면 나타나는 화면에는, 로컬 PC의 IP 주소인 127.0.0.1:3000으로 접속해서 확인하는 경우와 마찬가지이다.

이와 같이 `protocol` 또는 `TCP`를 사용하는 Application이 데이터를 주고받는
통로를 정하여 상호 간섭이나, 수정, 탈취등을 예방할 수 있다.

흔히 아는 `well known port` 는 `0-1024`까지 이며 `1025-65,535`는 잘 알려지지 않는 포트로 각 Application에 맞게 할당 할 수 있다.


흔히 `Web Server`를 구축할때, `FrontEnd` 에서 `API`를 `axios`를 사용하여 직접 `call` 하는 상황이 발생하는 경우 `Backend`는 다른 포트를 사용하기 때문에 기본적으로 `Cross Origin`에 위배되어 실행이 안되는 경우가 발생한다. 

때문에 이럴땐 2가지의 방법이 있는데
    
    1. Port Forwarding 을 통한 해결법
    2. Cros Origin 설정을 통해 특정 주소는 허용하는 방법
   
그 중에 통상 2번으로 간단하게 구현하긴 하지만, 크고 복잡한 서버의 구성이라면

필수불가결 하게 `Port forwarding`을 해야만 하는 경우가 발생 할 수 있다.


## SFTP 세팅
---

`SFTP`는 `SSH`기반 `FTP`이다. 그렇기에 우선 `SSH`가 우선적으로 설정이 되어있다는 가정 하에 진행이 되어져야 한다.

`SSH`가 이미 설치되어져 있다면 `SFTP`는 `sshd_config` 파일을 수정하여 설정 할 수 있다.

```linux
# sshd_config 파일에서 아래 설정 두개를 추가한다
Subsystem sftp /usr/lib/openssh/sftp-server
PasswordAuthentication yes
```

위 처럼 설정을 하게되면 `SFTP`를 사용가능하게 되며,
`ID/PW` 기반으로 인증을 한다는 내용이다.

동일하게 `SSH key`를 활용하여도 접속이 가능하며
클라이언트는 `/home/USER/.ssh/authorized_keys`에
키를 저장해 접속하면 접속이 이루어 진다.

window 사용자는 `File_zlia` 를 사용하는 것이 원할 하며 해당 사용내용은 아래 블로그를 참조하기 바란다.

[File_Zila SSH key 이용해서 접속하기](https://technfin.tistory.com/entry/SSH-Key%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%B4%EC%84%9C-SFTP%EB%A1%9C-%EC%84%9C%EB%B2%84%EC%97%90-%EC%A0%91%EC%86%8D%ED%95%98%EA%B8%B0-Filezilla-sFTP-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%A8-%EC%9D%B4%EC%9A%A9)


## FTP 세팅
---

`FTP`는 명령어를 전달하는 `21` 포트와 데이터를 전달하는 `20`(혹은 랜덤) 포트가 있다. 흔히 `passive`, `active` 모드로 표현된다.

갑자기 이 얘기를 왜 했냐 .... 결국 `FTP`는 2개의 포트가 열여 있어야 된다는 얘기를 하고 싶어서 썼다 !


자 그럼 설치를 해보자 

```linux
# ubuntu 에서 제공하는 패키지
sudo apt install vsftpd
```

위 명령어를 실행하게 되면 설정파일이 하나 생긴다.

`/etc/vsftpd.conf` 라는 파일이 생기는데 여기서 설정을 해준다
옵션은 아래와 같다
    vsftpd.conf 옵션
   - anonymous_enable: anonymous(익명) 사용자의 접속을 허가할지 설정
   - local_enable: 로컬 사용자의 접속 허가 여부를 설정
   - write_enable: 로컬 사용자가 저장, 삭제, 디렉터리 생성 등의 명령을 실행하게 할 것인지 설정
   (anonymous 사용자는 해당 없음)
   - anon_upload_enable: anonymous 사용자의 파일 업로드 허가 여부를 설정
   - anon_mkdir_write_eanble: anonymous 사용자의 디렉터리 생성 허가 여부를 설정
   - dirlist_enable: 접속한 디렉터리의 파일 리스트를 보여줄지 설정
   - download_enable: 다운로드의 허가 여부를 설정
   - listen_port: FTP 서비스의 포트 번호를 설정(기본: 21번)
   - deny_file: 업로드를 금지할 파일을 지정(예: deny_file={*.mpg,*.mpeg,*.avi})
   - hide_file: 보여주지 않을 파일을 지정(예: hide_file={*.gif,*.jpg,*.png})
   - max_clients: FTP 서버의 동시 최대 접속자 수를 지정
   - max_per_ip: 1개 PC가 동시에 접속할 수 있는 접속자 수를 지정


각 사용하고자 하는 설정을 추가하고 vsftpd를 실행시킨다.
```liux
# vsftpd 실행
sudo systemctl start vsftpd
# vsftpd 재시작시 자동실행
sudo systemctl enable vsftpd
# 방화벽 설정
sudo ufw allow 21/tcp
sudo ufw allow 20/tcp
sudo ufw reload
```

이렇게 하면 설정이 완료된다. `FileZile`나 기본 명령어를 사용하면 된다.

이상 끝!






































