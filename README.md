# 개인용 NAS 구축

# 1. 소개

---

집에서 취미 생활 및 데이터 보관 편의를 위한 NAS 서버 구축하며 진행한 작업 내용 보관

(케이스를 제외한 기타 부품은 이전에 사용했던 PC 부품 재활용)

- CPU : i5-4560
- RAM : 8GB+4GB
- SDD : 128GB
- HDD : 14TB
- CASE : 8Bay
- OS : Ubuntu (버젼 표기)

---

## 1.1 NAS 구축 이유

---

- 개인용 서버를 가지고 있으면 온전히 모든 권한을 활용할 수 있는 서버 관리가 가능
: 자유롭게 명령어 이용 및 리눅스 작업 관련 학습 가능
- NAS를 만들고 관리하는 것 자체가 지속적이고 유익한 취미
- 클라우드 서비스 등을 이용할 때와 비교하여 비용 관리가 용이
: 전기세 정도의 비용만 신경쓰면 되며 불필요할 때는 서버의 전원을 마음대로 내릴 수 있음
- 대용량 데이터 보관 및 이동에 용이
: 휴대폰 사진, 영상 등을 백업하거나 개인 작업 자료들을 보관하거나 이용할 때도 NAS에 올려두고 관리하면 어디에서나 쉽게 이용할 수 있음

---

## 1.2 Ubuntu 선택 이유

---

- 기존에 쓰던 CentOS의 Stable 버젼이 8에서 업데이트 지원이 종료됨. (2021년 12월 31일부)
- 이를 대체하기 위해 Rocky Linux를 최초에 설치하였으나 구축 당시에는 해당 OS에 대한 참고자료가 기존 Linux들에 비해 많지 않은 상황
- Ubuntu 역시 CentOS와 비견해도 손색없을 정도의 커뮤니티 규모와 참고자료를 가지고 있어 개인용 서버를 구축하는데 무리가 없을 것이라 판단함.
- 구축 시, 일부 명령어 차이점이 있지만 용도가 CentOS에서 쓰던 것과 1:1로 매칭되는 경우가 대부분이었기에 큰 문제는 없었음.
(대표적으로 패키지 관리 명령어 : ubuntu → apt-get / centOS → yum)
- 참고 : [Ubuntu apt(apt-get) 와 Redhat/CentOS yum 명령어 비교표 (lesstif.com)](https://www.lesstif.com/lpt/ubuntu-apt-apt-get-redhat-centos-yum-89555903.html)

---

# 2. 구축

---

## 2.1 주요 설치 프로그램

---

1. Docker : 컨테이너 형태로 여러 서비스를 관리하여 각 서비스가 요구하는 실행환경을 독립적으로 구성
2. Portainer : Docker가 지원하는 컨테이너 관리 기능을 Portainer 화면(GUI)에서 관리
3. NextCloud : 클라우드 서비스
4. Emby : 미디어 서버 서비스
5. Nginx Proxy Manager : 설치한 서비스들의 접속에 용이하도록 리버스 프록시 이용 및 SSL 인증서 관리 용이를 위해 설치

---

## 2.2 NAS 서버 구축을 위한 기본 설정

---

### 2.2.1 SSH

1) 설치

> $ sudo apt update
$ sudo apt install **openssh-server**
> 

2) 상태 확인

> $ sudo systemctl status **ssh**
> 
> 
> ● ssh.service - OpenBSD Secure Shell server
> Loaded: loaded (/lib/systemd/system/ssh.service; enabled; vendor preset: enabled)
> Active: **active** (running) since Mon 2022-08-22 12:26:38 KST; 12min ago
> 

3) 실행 (실행되어 있지 않은 상태일 때)

> $ sudo systemctl start ssh
> 

4) 종료

> $ sudo systemctl stop ssh
> 

5) 서버 시작 시 자동 실행 활성화/비활성화

> $ sudo systemctl **enable** ssh
$ sudo systemctl **disable** ssh
> 

**※ 포트 변경 (예시 : 22 → 2222)**

> $vi **/etc/ssh/sshd_config**
…
> 
> 
> 13 Include /etc/ssh/sshd_config.d/*.conf
> 14
> 15 **Port 2222**
> 
> …
> 

**※ 공개키 접속 설정**

1) 서버에 접속하고자 하는 PC의 공개키 생성

> $ ssh-keygen -t rsa
$ vi /home/{Users}/.ssh/**id_rsa.pub**
**ssh-rsa AAAAB3NzaC1yc2E…..**
> 

2) 서버에 공개키 등록

> $ **ssh-copy-id {User}@{Server Address}**
> 

3) 공개키 접근 허용 

> $ sudo vi /etc/ssh/sshd_config
…
> 
> 
> 37 #MaxSessions 10
> 38
> 39 **PubkeyAuthentication yes**
> 40
> …
> 

4) 비밀번호 접근 금지

> $ sudo vi /etc/ssh/sshd_config
…
> 
> 
> 57 # To disable tunneled clear text passwords, change to no here!
> 58 **PasswordAuthentication no**
> 59 #PermitEmptyPasswords no
> 
> …
> 

5) 공개키로 접근

> $ ssh **{User}@{Server Address}** -p **{Port}**
Welcome to Ubuntu 20.04.4 LTS (GNU/Linux 5.15.0-46-generic x86_64)
> 

---

### 2.2.2 블루투스 드라이버 설치

1) 우분투 커널 버젼 확인

> $ uname -r
5.15.0-46-generic
> 

2) 블루투스 드라이버 설치

> $ sudo cp rtl8761b_fw /usr/lib/firmware/rtl_bt/**rtl8761b_fw.bin**
> 

3) 재부팅 후 블루투스 드라이버 동작 확인

참고 : [우분투20.04에서 Bluetooth 5.0 USB 동글 설치: Realtek 8761B Chipset (tistory.com)](https://kibua20.tistory.com/170)

---

### 2.2.3 HDD 추가 설치(HDD 마운트)

1) HDD 장착 후 정상 인식 여부 확인

> $ fdisk -l
…
Disk **/dev/sdb**: 12.75 TiB, 14000519643136 bytes, 27344764928 sectors
…
> 
> 
> $ blkid
> …
> **/dev/sdb1**: UUID=**{UUID}**
> 

2) HDD 파티셔닝

> $ parted **/dev/sdb**
> 
> 
> $ (parted) **mklabel**
> New disk label type? **gpt**
> 
> $ (parted) **mkpart**
> Partition name? []? **1**
> File system type? [ext2]? **ext4**
> Start? **1**
> End? **14t**
> 
> $ (parted) **p**
> 
> Model: ~
> Disk /dev/sdb: 14...
> …
> Number 	Start	End	Size	File System	Name
> …
> 
> $ (parted) **q** 
> Information: You may need to update /etc/fstab
> 

3) HDD 포멧

> $ mkfs -t ext4 /dev/sda
> 

4) HDD 마운트 경로 생성

> $ mkdir **{DIR}**
> 

5) HDD 마운트 경로 등록

> $ sudo vi /etc/fstab
UUID=**{UUID} /dev/sda** ext4 defaults 0 1
> 

6) HDD 마운트

> $ sudo mount -a
> 

7) HDD 마운트 상태 확인

> $ df -h
…
/dev/sdb1 13T 0 13T 0% **{DIR}**
…
> 

 

참고 : [[Ubuntu/Linux] 대용량 디스크 마운트하기(4TB 이상 디스크 mount) (tistory.com)](https://minimin2.tistory.com/158)

---

## 2.3 Docker & Portainer 설치

---

1) Docker 패키지 설치 전 사전 작업

> $ sudo apt-get update
$ sudo apt-get install \
ca-certificates \
curl \
gnupg \
lsb-release
> 

2) Docker 패키지 설치

> $ curl -fsSL [https://download.docker.com/linux/ubuntu/gpg](https://download.docker.com/linux/ubuntu/gpg) | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
> 
> 
> $ sudo add-apt-repository "deb [arch=amd64] [https://download.docker.com/linux/ubuntu](https://download.docker.com/linux/ubuntu) $(lsb_release -cs) stable"
> $ apt-get update
> $ sudo apt-get install docker-ce docker-ce-cli [containerd.io](http://containerd.io/)
> 

3) Docker 설치 확인

> $ docker --version
Docker version 20.10.16, build aa7e414
> 
> 
> $ docker run hello-world
> Unable to find image 'hello-world:latest' locally
> latest: Pulling from library/hello-world
> 2db29710123e: Pull complete
> Digest: sha256:7d246653d0511db2a6b2e0436cfd0e52ac8c066000264b3ce63331ac66dca625
> Status: Downloaded newer image for hello-world:latest
> 
> Hello from Docker!
> This message shows that your installation appears to be working correctly.
> 
> To generate this message, Docker took the following steps:
> 
> 1. The Docker client contacted the Docker daemon.
> 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
> (amd64)
> 3. The Docker daemon created a new container from that image which runs the
> executable that produces the output you are currently reading.
> 4. The Docker daemon streamed that output to the Docker client, which sent it
> to your terminal.
> 
> To try something more ambitious, you can run an Ubuntu container with:
> $ docker run -it ubuntu bash
> 
> Share images, automate workflows, and more with a free Docker ID:
> [https://hub.docker.com/](https://hub.docker.com/)
> 
> For more examples and ideas, visit:
> [https://docs.docker.com/get-started/](https://docs.docker.com/get-started/)
> 

4) 컨테이너 설치는 아래 서비스들 설치 항목에서 서술 (compos 이용)

참고 : [Ubuntu 20.04 LTS ) Docker 설치하기 (tistory.com)](https://shanepark.tistory.com/237)
[ubuntu docker 설치시 Package 'docker-ce' has no installation candidate 해결 (tistory.com)](https://boying-blog.tistory.com/82)

### 2.3.1 Docker Volume 위치 변경

참고 : [[Docker] Data Root Directory 경로 변경 (tistory.com)](https://carpfish.tistory.com/entry/Docker-Data-Root-Directory-%EA%B2%BD%EB%A1%9C-%EB%B3%80%EA%B2%BD)
[Docker Volume 마운트 위치 변경 (tistory.com)](https://boying-blog.tistory.com/78?category=833988)

### 2.3.2 Docker 컨테이너 사이 볼륨 공유

참고 : [코끼리를 냉장고에 넣는 방법 :: [Docker] 도커 컨테이너 사이에 디렉터리 및 파일 공유하기 (tistory.com)](https://dololak.tistory.com/403)

[Docker에 vi가 안될때 (tistory.com)](https://cpdev.tistory.com/144)
[[Docker] 컨테이너 bash에 vim 설치하기 — 논리적 코딩 (tistory.com)](https://logical-code.tistory.com/123)

---

작성 중...# nas