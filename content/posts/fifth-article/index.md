+++
title = 'Understanding Ubuntu PPA'
date = 2025-08-13T20:35:50+09:00
draft = false 
+++

## PPA에 대한 이해

### PPA란 무엇인가

PPA(Personal Package Archive)는 공식 저장소 외에 개인이 직접 저장소를 관리할 수 있게 해주는 플랫폼이다. 또한 그 플랫폼 위의 각 저장소 또한 PPA라 한다. 즉,  PPA도 하나의 저장소이다.

우분투의 공식 Repository(저장소)는 안정성을 위해 패키지가 오래된 버전인 경우가 많다. 하지만 때에 따라서는 최신 버전의 패키지가 필요한 경우가 있다.

이 때에 원하는 패키지에 대한 PPA를 추가한 뒤에 설치를 하면 최신 버전의 패키지를 설치할 수 있다.

### PPA를 통한 패키지 설치 방법

PPA를 찾으려면 [Personal Package Archives for Ubuntu](https://launchpad.net/ubuntu/+ppas) 이 사이트에서 검색을 통해서 내가 찾는 패키지에 대한 PPA가 있는 지 찾을 수 있다. (아니면 패키지의 공식 문서를 보면 다운로드 방법이 있을 것인데, 거기에서 PPA를 통한 설치를 안내하는 경우도 있다.) 이렇게 해서 추가하고자 하는 PPA를 찾았다고 하자.

```bash
sudo apt update # apt repository update
sudo apt install software-properties-common # 저장소 관리 도구 패키지 설치
sudo add-apt-repository ppa:user/repo # 공식 문서나 인터넷 검색을 통해 찾은 저장소
# PPA에 대한 정보가 나오고 확인 메시지가 나옴. Enter - 추가 / ctrl-c - 취소
sudo apt update # apt repository update (새 저장소 반영)
sudo apt install package
```

PPA를 추가할 때 다음의 명령어를 사용하면 자동 확인 및 apt 업데이트를 한번에 해준다.

```
sudo add-apt-repository --yes --update ppa:ansible/ansible
```

예시로 ansible을 PPA를 통해서 다운로드해보자. ansible 공식 문서에서는  ansible/ansible 이라는 PPA를 통해 다운로드하도록 안내한다. 따라서

```bash
sudo apt update
sudo apt install software-properties-common # 이미 설치되어있으면 생략 가능
sudo add-apt-repository ppa:ansible/ansible
# Enter
sudo apt update
sudo apt install ansible
```

이 과정을 따르면 ansible 설치는 끝났다!

### 패키지를 다운로드한 저장소 확인하기

마지막으로 패키지가 의도대로 PPA에서 설치되었는지 확인이 필요하다. 이는 다음의 명령어로 가능하다.

```console
$ apt show ansible # apt 저장소에서 ansible 정보 불러오기
Package: ansible
Version: 11.8.0-1ppa~noble # PPA에서 다운로드한 것을 확인할 수 있다!
Priority: optional
Section: admin
Maintainer: Ansible Community Builds <ansible-community-builds@redhat.com>
Installed-Size: 223 MB
Depends: ansible-core, python3:any
Recommends: python3-jmespath, python3-winrm
Download-Size: 19.1 MB
APT-Manual-Installed: yes
APT-Sources: https://ppa.launchpadcontent.net/ansible/ansible/ubuntu noble/main amd64 Packages
Description: Ansible collections for ansible-core
 Batteries-included package providing a curated set of Ansible collections

N: There is 1 additional record. Please use the '-a' switch to see it # PPA 외에 공식 저장소에도 ansible가 있어서 뜨는 문구.
```

패키지를 설치하기 전에도 확인이 가능하다! 다음의 명령어로 가능하다.

```console
$ apt-cache policy ansible
ansible:
  Installed:  (none)
  Candidate: 11.8.0-1ppa~noble
  Version table:
 *** 11.8.0-1ppa~noble 500
        500 https://ppa.launchpadcontent.net/ansible/ansible/ubuntu noble/main amd64 Packages
        100 /var/lib/dpkg/status
     9.2.0+dfsg-0ubuntu5 500
        500 http://kr.archive.ubuntu.com/ubuntu noble/universe amd64 Packages
```

### PPA 목록 확인 및 PPA 삭제 방법

추가한 PPA 목록은 다음과 같은 명령어로 확인 가능하다.

```
ls /etc/apt/sources.list.d/
```

PPA 제거 방법은 다음과 같다.

```
sudo add-apt-repository --remove ppa:user/repo
```

PPA 제거를 하면서 패키지도 같이 정리하고 싶으면 다음 명령어를 사용한다.

```
sudo ppa-purge ppa:user/repo
```

이 명령어는 다음과 같이 작동한다.
1. PPA를 삭제
2. 해당 PPA를 통해 설치한 패키지는 다른 저장소의 버전으로 전환.
3. 만약 해당 패키지를 갖는 다른 저장소가 없으면 패키지를 삭제한다.

## 마무리

나 스스로는 PPA에 대해서 이 정도의 이해를 하고 있고, 추후 새로 알게 되는 정보가 있으면 글을 추가할 수 있다. 우분투를 사용한다면 PPA는 어찌 되었든 무조건 쓰게 되어 있으니 간단하게라도 내용을 숙지하고 있으면 도움이 될 것이다.
