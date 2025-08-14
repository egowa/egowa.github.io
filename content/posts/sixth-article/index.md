+++
title = 'Tailscale 클라이언트 Headscale 서버에 연결하기'
date = 2025-08-14T04:59:45+09:00
draft = false
+++

![썸네일 사진](featured.png)

## tailscale이란 무엇인가?

tailscale은 mesh network를 구성할 수 있게 해주는 도구 중 하나다. 각 클라이언트가 속한 물리적인 네트워크에 관계 없이 하나의 network로 연결시켜준다. 

tailscale은 wireguard 기반 도구이다. 다만 wireguard 보다 강력한 점은 tailscale을 통해서 클라이언트를 연결하면 단순히 1:1 연결만 되는 것이 아니라 VPN에 속한 모든 기기들이 서로 연결되는 mesh network가 구성이 된다는 것이다. 그래서 쉽게 여러 기기들을 하나의 네트워크로 묶어줄 수 있다.

tailscale에 대한 구체적인 원리는 아래의 글에서 자세히 설명되어 있다.

[How Tailscale works](https://tailscale.com/blog/how-tailscale-works)

## headscale이란 무엇인가?

원래 tailscale은 tailscale 회사에서 호스팅하는 서버를 중심으로 메쉬망을 구성하도록 한다. tailscale의 공식 문서에 따르면, 인터넷 트래픽이 전부 tailscale 회사를 거쳐가는 것이 아니라 메쉬망 내의 기기들이 P2P 연결을 수립할 수 있도록 중계해주는 역할만 한다고 한다. P2P 연결이 불가능한 경우에는 DERP 서버를 거쳐서 연결이 되지만, 이 경우에도 packet은 암호화되며, 일반적인 두 기기간 인터넷 통신과 다를 것이 없는 보안 레벨이라고 말한다.  즉 프라이버시 상 문제는 없다고 얘기한다. 아래의 글을 통해 이에 대한 tailscale의 설명을 확인할 수 있다.

[Is my traffic routed through your servers? - tailscale](https://tailscale.com/kb/1094/is-all-traffic-routed-through-tailscale)

이런 설명에도 불구하고 불안한 마음을 떨쳐낼 수가 없거나, 또는 tailscale의 coordination server(제어 서버)를 직접 제어하고 싶은 마음이 있을 때 headscale을 사용하면 좋다.

headscale은 tailscale 시스템에서 coordination server에 해당하는 부분을 오픈소스로 재구현한 프로젝트로,  tailscale 회사가 아니라 스스로 제어 서버를 구축할 수 있도록 해준다.

## 클라이언트를 headscale 서버에 연결하기

headscale을 통해서 tailscale의 제어 서버를 대체할 수 있다. 이 글에서는 headscale 서버를 이미 구축했다고 가정한다.

우선 각 기기를 headscale 서버에 연결시키기 위해서는 tailscale을 설치해야 한다. 
제어 서버와 달리 클라이언트 부분은 오픈소스로 구현되지 않았으므로, 상용 프로그램을 설치하면 된다. 설치 방법은 시간이 지남에 따라 달라질 수 있으므로 꼭 아래의 tailscale 공식 문서를 확인하자. 여기서는 현 시점 우분투 서버 24.04 LTS 버전에 설치하는 방법만 다뤄보겠다.

[Install Tailscale](https://tailscale.com/kb/1347/installation)

### 우분투 서버에 tailscale 설치하기 

우선 GPG 공개키와 저장소를 호스트에 추가하자.

```bash
curl -fsSL https://pkgs.tailscale.com/stable/ubuntu/noble.noarmor.gpg | sudo tee /usr/share/keyrings/tailscale-archive-keyring.gpg >/dev/null # GPG 공개키(서명키) 다운로드
curl -fsSL https://pkgs.tailscale.com/stable/ubuntu/noble.tailscale-keyring.list | sudo tee /etc/apt/sources.list.d/tailscale.list # Tailscale 저장소 설정 파일 다운로드
```

Tailscale 저장소에서 최신 버전 패키지를 잘 다운받을 지 여부를 검증해보자.

```bash
apt policy tailscale # Candidate 부분과 Version table을 확인하자.
```

확인이 되었다면 이제 패키지를 설치하자.

```bash
sudo apt update # 패키지 목록 업데이트
sudo apt install tailscale # tailscale 설치
```

이제 설치가 완료되었다! 본격적으로 클라이언트를 headscale 서버에 연결해보자.

### headscale 서버에 접속하기

여기서부터는 tailscale 문서에는 적혀있지 않은 내용이다. 특별한 내용이라는 것은 아니고, headscale 공식 문서를 보면 자세히 나와 있다. 

[Register a node - headscale](https://headscale.net/stable/usage/getting-started/#register-a-node)

이 글에서는 Normal, interactive login 방법에 대해서 설명하겠다.

우선 tailscale initialization이 필요하다. 다음과 같이 하면 된다.

```console
# tailscale up --login-server <YOUR_HEADSCALE_URL> 

To authenticate, visit:

	https://<YOUR_HEADSCALE_URL>/register/RmX7d6xJNPnEAdqF6iWAjzli
```

출력되는 주소로 접속하면 다음과 같은 블럭이 있을 것이다.

```bash
headscale nodes register --user USERNAME --key RmX7d6xJNPnEAdqF6iWAjzli
```

이 명령어를 입력해주면 되는데, 도커 컨테이너로 headscale 서버를 구축한 사람은 조금 다르게 입력해줘야 한다. 차이는 크지 않다.

```console
$ docker exec -it <HEADSCALE_CONTAINER_NAME> headscale nodes register --user USERNAME --key RmX7d6xJNPnEAdqF6iWAjzli

Node <NODE_NAME> registered
```

headscale 서버 연결이 완료되었다!

## 마무리

별도의 서버가 없어 headscale을 쓰지 못한다고 하더라도 tailscale 자체가 매우 강력한 mesh network 솔루션이여서 꼭 한번 써보는 것을 추천한다. 상용 서비스는 별도의 호스팅이 필요 없다.
