---
layout: post
title:  "[DevOps] Linux 부시기 (SSH)"
date:   2022-07-01 08:56:53 +0900
categories: DevOps
tag: Linux Ubuntu
---

이전에 Ecommerce 작업을 진행하면서 웹서비스를을 Linux 서버에 올릴때 항상 고생을 했다.
한번 시작 할때마다 나는 삽질을 계속한다.... 이게 너무싫어서 좀 정확하게 공부를 해놓으려고 한다.

이번 삽질의 목표!
기본적으로 웹서비스를 배포하게 될때 필요한 원리와 필요한 정보들을 정리하고자 한다. 정리내용은 아래와 같다


1. `SSH`와 `FTP` 세팅 원리와 이해
2. `Port forwording` 세팅 원리와 이해
3. `Linux` 에서 `Forground` 와 `Background`의 이해
4. `DNS` 세팅 원리와 이해
5. `Apache` 세팅 원리와 이해
6. `Mail server` 세팅 원리와 이해


위 내용을 공부하기 위해서 나는 `VMware`를 쓸지... `virtual Box`를 한참을 고민하다
그래도 무료인 `virtual Box`를 하기로 했다.
    

자 오늘도 열심히 달려보자~

---

# SSH와 FTP 세팅 원리와 이해

먼저 기본적으로 파일을 업로드 및 접속을 할때 가장 필요한 기능들이며

기본적으로 클라우드,Linux,centOS의 경우 기본적으로 세팅이 되어져 있다.

그러나 세상살이 어떻게 될지 모르고, 잘 안되는 경우도 있으니 정리를 해보자.



## SSH 세팅

---
먼저 아래의 명령어를 사용하여 `openssh-server` 패키지를 다운 받는다.

```linux
sudo apt install openssh-server
```
설치가 완료 되었으면 아래 명령어를 쳐서 동작 하고 있는지 확인을 한다.

```linux
sudo systemctl status ssh
```

아래와 같이 `active`가 나오면 정상 작동 중인것이다.

![이미지](/assets/images/sudo_system_status_ssh.PNG)


그 이후 먼저 `SSH` 가 시스템이 재부팅 될때 자동으로 실행 가능하게 만들어 준후 재실행한다.
```linux
 # ssh를 시작시 자동실행
 sudo systemctl enable ssh
 # ssh 재실행  
 sudo systemctl ssh restart
```

이렇게 설치까지 완료 되었으면 `22번` 포트를 열어준다. `ufw`는 방화벽을 설정하는 명령어이다.

```linux
# ssh 사용포트인 22번 열어주기
sudo ufw allow ssh
# ssh 사용포트인 22번 열어주기 명령어 다른 버전
sudo ufw allow 22/tcp
# 방화벽을 시작시 자동실행
sudo ufw enable
```

이렇게 하면 얼추 끝났다. 그렇면 제대로 열려있는지 확인을 해보자

```linux
# network의 port 열려 있는지 확인하는 명령어
netstat -tnlp
```
![이미지](/assets/images/ssh_netstat.PNG)

위와 같이 `22번` port가 열리면 셋팅이 완료된 것이다.
그러나 기본적으로 이렇게 세팅을 할 경우에는 `any` 접근이 설정 된것이다.
특정 사용자 or ip만 접근을 가능하게 하는 등의 설정은 `sshd_config`에서 설정을 해야한다. 자세한 부분은 아래에서 다루겠다.


## SSH의 작동흐름
---

`SSH`가 작동하는 흐름은 아래와 같다

    1. 클라이언트가 호스트에게 연결을 한다 (TCP/22포트 사용)
    2. 호스트는 클라이언트의 SSH 접속 요청을 받고, 자신의 공개키를 클라이언트에게 전송하고, 클라이언트는 서버로부터 받은 공개키를 로컬에 저장한다.
    3. 클라이언트와 호스트는 여러 Prameter를 송수신하며 보안 채널(세션)을 만든다.
    4. 클라이언트가 호스트와 연결된다.

아래 그림을 참조하자~

![이미지](/assets/images/ssh_flow.PNG)

## SSH 설정 방법
---
일단 기본적으로 `SSH` TCP 통신에 기반한다.
흔히 알려진 known port 인 `22`번 포트를 사용하고, 인증 및 암호화를 진행한다.


인증에는 `비대칭키 방식`, 
서버간 통신에 대한 암호화에는 `대칭키 방식`을 사용한다.


이런 부분을 설명하는 이유는 서버를 설정할때
`SSH key`을 사용하여 관리해야 하고, 각 사용자마다 어떻게 진입하는지 알아야 안전하게 서버를 관리 할 수 있기 때문이다.

---

### SSH key 설정

아래는 클라이언트가 원격접속지(호스트)들의 공개키를 저장하고 있는 경로이다.


    ※ 클라이언트 사용자의 known_hosts 경로(Default)

    <Linux>
    일반계정  : /home/USER/.ssh/known_hosts
    root 계정 : /root/.ssh/known_hosts		

    <Window>
    사용자    : %USERNAME%\.ssh\known_hosts


위 경로를 통해서 최초 `SSH` 접속시에는 아래와 같은 과정을 거치며 호스트에 대한 인증을 수행한다

    클라이언트에서 난수, 해시값 생성
    -> 호스트의 공개키로 암호 후 전송
    -> 호스트에서 개인키로 복호화
    -> 클라이언트에 호스트가 복호환 값을 전송
    -> 클라이언트는 저장된 난수와 비교
    -> 일치할 경우 올바른 서버로 인식

위 부분은 서버와 호스트간에 자동적으로 수행되는 과정이다. 그렇기 때문에 크게 고민할 필요없고,

이런 과정을 거친다~ 정도만 인지하고 있자


그럼 가장 핵심인 `SSH key` 에 대해 알아보자

```linux
ssh-keygen -t [rsa,dsa,ecdsa,ed25519] -b
# -t(type) : 알고리즘 선택 옶션
# -b(bit) : 키의 크기(bit) 결정
# 기본적으로 보통 ssh-keygen -t rsa 로 생성한다
```


위 명령어는 `openssh-server` 패키지에 자동적으로
포함되어 있다. 기본적으로 우리가 `Ubuntu`에 `SSH`를 설정할때 설치하는 그 패키지 이다.


위 명령어를 실행하면 아래처럼 나온다.

![이미지](/assets/images/ssh_keygen.jpg)

빨간 및줄 쳐놓은 첫번째는 `SSH key`를 저장할 `directory` 를 묻는다.

 입력을 안할 시에는 `default = /home/사용자계정/.ssh/id_rsa` 이다. 이것은 개인 key를 저장하는 곳이고

두번째는 `passphrase`는 `salt`값이다 즉, `passphrase`를 넣으면 한번 더 암호화가 된다.
강력한 암호화를 원한다면 넣는 것을 추천한다.

세번째는 `SSH pubilc key` 가 저장된 곳이다
동일하게 위에 key를 저장할 `directory`에 저장이 되며 `default = /home/사용자계정/.ssh/id_rsa` 이다. public key는 `개인키+passphrase`가 합쳐져서 만들어진다.

---

### SSH ssh_config, sshd_config 옵션 설정


이런 `SSH`가 작동을 하기 위해서 항상 클라이언트의 연결을 대기하는 프로세스가 있어야 한다. 그렇기에 `SSH Daemon`이 항상 `Background`에서 돌아가고 있으며 `SSH Daemon`의 설정 파일은 `sshd_config` 이다


즉, `SSH`는 두가지 설정파일이 존재하며 용도는 아래와 같다
> ssh_config (나가는 설정) , sshd_config (들어오는 설정)

그렇다면 나머지 설정은 사용설정에 맞게 구성을 하면된다.  `sshd_config`의 환결설정 변수들의 설명은 아래와 같다~

    ① 기본설정 
    Protocol 2 
    openssh는 프로토콜 버전을 
    원하는 대로 선택할 수 있습니다. protocol 2로 설정에는 서버는 버전 2로만 작동하기 때문에 ssh1을 사용해 접속을 요청하는 
    클라이언트를 받아 들일 수 
    없다. protocol 1로 설정해서 가동시킬 경우에는 버전 
    2를 사용하는 ssh2 사용자의 요청을 받아 들일 수 없다. 보안상 protocol 1 은 사용하지 않습니다. 
   
      
    KeyRegenerationInterval 3600 
    서버의 키는 한번 접속이 이루어진 뒤에 자동적으로 다시 만들어진다. 다시 만드는 목적은 
    나중에 호스트의 세션에 있는 키를 캡처해서 암호를 해독하거나 훔친 키를 사용하지 못하도록 하기 위함 위함입니다. 
    값이 0이면 키는 다시 만들어지지 않습니다. 
    기본값은 3600초입니다. 이 값은 자동으로 
    키를 재생성하기 전까지 서버가 대기할 시간을 초단위로 정의합니다. 
      
    ServerKeyBits 1024 
    서버 키에서 어느 정도의 비트 수를 사용할지 정의합니다. 최소값은 512이고 디폴트 값은 768입니다. 
      
    SyslogFacility AUTH 
    /etc/syslog.conf에서 정의한 
    로그 facility 코드입니다. 가능한 값은 DAEMON, USER, AUTH, LOCAL0, LOCAL1, LOCAL2, LOCAL3, LOCAL4, LOCAL5, 
    LOCAL6, LOCAL7입니다. 기본값은 
    AUTH입니다. Facilith란 메시지를 생성하는 하위 시스템을 말합니다. 
      
    LogLevel INFO 
    로그 레벨을 지정하는 것입니다. 가능한 값은 QUIET, 
    FATAL, ERROR, INFO, VERBOSE 그리고 DEBUGS입니다. 
       

    LoginGraceTime 600 
    유저의 로그인이 성공적으로 이루어지지 않았을 때 이 시간 후에 서버가 연결을 끊는 시간입니다. 
    값이 0 이면 제한 시간이 없으며 기본값은 
    600초입니다. 
      
    strictModes yes 
    사용자의 홈 디렉토리인 /home/username의 권한 값 등을 체크하도록 설정되어 
    있는 지시자 입니다. 
      
    RSAAuthentication yes 
    RSA 인증의 시도여부를 
    정의합니다. ssh1 프로토콜에만 사용하기 위해 예약된 것으로, 

    ssh1을 사용하고 운영상 
    보다 안전하게 운영하려면 이 옵션을 yes로 설정해야 합니다. 

    RSA는 인증을 하기 
    위해 ssh-keygen 유틸리티에 의해 생성된 공개키 와 비밀키 쌍을 사용합니다. 현재 문서에서는 보안상 ssh1 프로토콜을 사용하지 않으므로 주석 
    처리합니다. 
      
    PubkeyAuthentication yes 
    AuthorizedKeysFile 
    .ssh/authorized_keys 
    ssh에서 제공하는 
    인증에는 공개키 인증과 암호 인증법 이렇게 두가지가 있는데, 공개키인증을 사용할 것인지에 대해 
    설정하는 것입니다. 공개키 인증 사용시 공개키가 있어야 하므로 더욱 안전합니다. 
      
    RhostsAuthentication no 
    sshd가 rhosts 기반의 인증을 사용할 것인지 여부를 정의합니다. rhosts 
    인증은 안전하지 못하므로 ‘no’로 합니다. 
      
    IgnoreRhosts yes 
    IgnoreRhosts’ 명령은 
    인증시 rhosts와 shosts 파일의 사용여부를 
    정의합니다. 
    보안상의 이유로 인증할 때 rhosts와 shosts 
    파일을 사용하지 않도록 합니다. 
      
    RhostsRSAAuthentication no 
    rhost나 /etc/hosts.equiv파일이 있으면 이것을 사용해 인증합니다. 
    이것은 보안상 별로 안 좋은 방법이기 때문에 허용하지 않습니다. RSA 호스트 인증과 
    맞추어 rhosts 인증의 사용여부를 정의합니다. 
      
    HostbasedAuthentication no 
    호스트 기반의 인증 허용 여부를 결정합니다. 

      
    IgnoreUserKnownHosts yes 
    ssh 데몬이 RhostsRSAAuthentication 과정에서 각 사용자의 
    $HOME/.ssh/known_hosts를 
    무시할 것인지 여부를 정의합니다. rhosts 파일을 허용하지 않았으므로 yes로 설정하는 것이 안전합니다. 
      
    PasswordAuthentication yes   
    패스워드 인증을 허용합니다. 이 옵션은 프로토콜 버전 
    1과 2 모두 적용됩니다. 인증할 때 암호기반 
    인증방법의 사용 여부를 결정합니다. 강력한 보안을 위해 이옵션은 항상 "no"로 설정해야합니다. 

      
    PermitEmptyPasswords no 
    패스워드 인증을 할 때 서버가 비어있는 패스워드를 인정하는 것입니다. 기본 값은 no입니다. 
      
    X11Forwarding no 
    원격에서 X11 포워딩을 허용하는 것입니다. 
    이 옵션을 yes로 설정하면 xhost보다 
    안전한 방법으로 원격에 있는 X프로그램을 사용할 수 있습니다. 
      
    PrintMotd yes  

    사용자가 로그인 하는 경우 /etc/motd (the message of the day) 
    파일의 내용을 보여줄 것인지 여부결정. ssh 로그인을 환영하는 메시지나 혹은 공지사항 
    같은 것을 적어 놓으면 됩니다. 
      
    Subsystem   
    sftp    
    /usr/libexec/openssh/sftp-server 
    sftp는 프로토콜 
    버전 2에서 사용되는 것으로서 ssh와 같이 ftp의 보안을 강화하기 위해 사용되는 보안 
    ftp프로그램입니다. 
    openssh를 설치하면 
    /usr/local/ssh/libexec/sftp-server파일이 설치됩니다. 
    이것은 sftp 서버용 프로그램입니다. 
    클라이언트 sftp프로그램은 설치되지 않습니다. 
    따라서 서버로 일단 가동시키고 원도용 ssh클라이언트 프로그램이나 SSH2를 설치하면 sftp를 사용할 수 있습니다. 
      
    CheckMail (yes/no) 
    사용자가 로그인할 때 새메일이 도착했음을 알리도록 하는 기능을 설정하며 기본값은 yes로 
    되어있습니다. 
      
    Cipher (ciher) 
    세션을 암호화 할 때 사용할 방법을 명시해 줍니다 

    (idea,des,3des,blowfish,arcfour 또는 없음) 
      
    ForwardAgent 
    인증 대리인이 포워드되어야 하는지를 명시 합니다. 
      
    KeepAlive yes 
    RequireReverseMapping no 
    클라이언트에게 aliive메시지를 보낼 것인지 명시하는데 접속하는 곳의 도 메인이 
    revers Mapping 이 되는지를 활인하여 접속을 허가할지 안할지를 지정합니다. 실제로 internet상에 호스트 들중 revers mapping 이 안되는 호스트가 상 당수이므로 되도록 
    no로 설정할 것을 권장합니다. 만약 여러분이 사용하시는 
    host가 revers mapping 이 확실히 되면 보안상 yes로 하는것이 좋겠지만 revers mapping이 되지 않으면 
    접속이 불가능 하므로 조심하십시오. 
      
    PasswordAuthentication (yes/no) 

    패스워드 기반의 인증방법을 사용할 것인지를 명시 합니다. 
      
      
    PubkeyAuthentication (yes/no) 
    인증 순서를 지정합니다. 
      
    ② 보안설정 
      
    Port 22 
    ssh가 사용할 기본 
    포트를 지정합니다. 포트 변경 시 /etc/services 
    파일에서 sshd 관련 포트 역시 변경할 포트로 변경해 주어야 합니다. 
      
    AllowUsers root nextline    
    로그인 허락할 계정 nextline와 root 
    두 계정에게만 로그인 허용 합니다. 
      
    PermitRootLogin no 
    root 로그인 허용여부를 
    결정하는 것입니다. yes, no, without-password를 사용할 수 있습니다. 현재 no로 되어 있기 때문에 직접  root로 접속이 불가능합니다. 이옵션을 yes 로 하기보다는 일반계정으로 로그인후 su 명령으로 root로 전환하는 것이 보안상 안전합니다. 
      
    ListenAddress 0.0.0.0 
    sshd가 귀를 기울일 
    주소를 정해줍니다. 0.0.0.0은 모든 곳으로 부터 접속 을 받아들이겠다는 의미입니다. 하지만 패키징을 할때 어떻게 한것인지는 모르겠지만 
    tcp-wrapper의 영향을 받아서 hosts.deny에서 막혀 있으면 접속이 
    안되니 hosts.allow와 
    hosts.deny에서 sshd2 항목으로 제어를 할수가 있습니다. 
      
    AllowedAuthentications   
    publickey,password 
    Sshd2가 제공하는 
    인증은 password와 publickey 그리고 hostbased 방식이 있는데, 기본 적으로 public,password가 사용됩니다. 이는 순서대로 인증하는 
    방법을 보여주는데, 먼저 publickey로 
    인증하고, 두 번째로 password로 인증한다는 
    의미입니다. 
      
    DenyUsers  
    nextline, 3737 
    접근을 거부할 로컬의 유저를 지정합니다. 위 설정은 
    nextline 및 uid가 3737인 
    계정으로 ssh 접속 시도할 경우 접근이 거부됩니다. 

      
    DenyGroups 
    명시된 그룹은 ssh서비스에 접근할 수 없도록 하는 기능을 합니다. 

    (DenyGroups sysadmin accounting) 와일드카드가 지원되며 공백 
    문자로 그룹을 구분합니다. 
      
    DenyHosts 
    명시된 호스트는 ssh서비스에 접근할 수 없도록 하는 
    기능입니다. 
    (Deny Hosts shell.ourcompany.net).호스트 IP를 쓰거나 호스트 명을 쓸 수 있으며 와 일드 카드가 지원되고 공백 문자로 호스트를 구분합니다. 
      
    AllowHosts   
    1.2.3.0/24 192.168.1.3 
    로그인을 허가할 IP 또는 IP 대역을 
    지정합니다. 여러 개일 경우에는 공란이나 "," 로 
    구분하여 나열하면 되고 도메인 이름일 경우에는 reverse mapping이 제공되어야 
    합니다. 
      
    AllowGroups 
    ssh서비스에 접근 가능한 
    그룹을 명시합니다. (예 : AllowGroups sysadmin 
    accounting) 와일드카드가 지원되며 공백문자로 그룹을 구분합니다. 

      
    MaxConnections   
    0 
    최대 몇개의 접속을 허락할지를 지정합니다. 0은 제한을 하지 않습니다. 
      
    PasswordGuesses   
    3 
    암호인증 방식으로 인증할 때 최대 몇 차례 시도를 허용할 것인지 지정합니다. 
      
    ssh1Compatibility   
    no 
    클라이언트가 ssh1만 지원할 경우 ssh1 
    데몬을 실행할 것인지 여부를 지정합니다. ssh1은 보안상 취약하므로 no로 하는 것이 
    좋습니다



이상 끝! 다음은 FTP 조져보자~




































