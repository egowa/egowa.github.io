+++
title = 'NixOS에서 tailscale 설정 선언하기 (feat: headscale, sops-nix)'
date = 2025-08-18T23:56:13+09:00
draft = false 
+++

![NixOS + tailscale + ops-nix](featured.png)

## NixOS에서 tailscale 선언적 설정 및 ops-nix로 비밀을 관리

## NixOS

NixOS는 리눅스 디스트로 중 하나로, 다른 디스트로와 다르게 설정을 선언적으로 한다는 특징이 있다.

 시스템 설정을 하는 방식은 크게 2가지로 분류할 수 있는데, 절차적 방식과 선언적 방식이 그것이다. 
 
 절차적 방식은 우리가 흔히 알고 있는 것이다. 예를 들어 방의 에어컨을 키려면 어떻게 해야 할까? 리모컨을 찾아서, 리모컨을 쥐고, 에어컨을 키면 될 것이다. 즉, 에어컨을 키기 위해서 해야 하는 일련의 과정을 하나하나씩 입력하며 어떤 목적을 달성하는 것이 절차적 방식의 특징이라 할 수 있다.
 
 그렇다면 선언적 방식이란 무엇일까? 선언적 방식이란 어떤 목적을 달성하기 위해 이루어져야 할 과정들에 관심을 갖지 않고, 원하는 최종 상태를 선언함으로써 무언가를 달성하는 것을 말한다. 이렇게 들으면 뭔가 마법같은 것이라 생각할 수 있지만, 이것이 실제로 NixOS가 작동하는 방식이다.
 
 예를 들어 새로산 컴퓨터에서 스팀 게임을 하고 싶다고 하자. 그러면 일단 어떻게 해야 할까? 절차적인 방식이라면 우선 운영체제를 설치해야 한다. 그리고 그래픽 드라이버를 설치해야 한다. 그리고 스팀을 설치해야 한다. 오디오 문제가 있으면 오디오 드라이버도 다운로드해야 한다. 스팀에서 원하는 game 모드가 있다면 수동으로 설정해야 한다. (CPU 부스트 모드 등) 그리고 만약 멀티게임을 하려고 한다면 방화벽을 열어줘야할 수도 있다. 이것도 수동으로 설정해야 한다.
 
 선언적 방식으로는 어떻게 설정을 할 수 있을까? 우선, 과정에 관심을 가질 필요가 없으며, 최종적인 상태만 선언하면 된다. 일단 스팀이 설치되어야 할 것이다. 그래픽, 오디오 등 게임을 하는 데에 필요한 드라이버는 다 설치가 되어야할 것이다. 스팀의 세부 설정은 ... 해야 할 것이다. 멀티플레이를 위한 방화벽이 열려 있어야 할 것이다.. 등의 최종 결과를 선언하면 시스템은 알아서 이 결과를 만들어내기 위한 과정들을 수행한다. 이것이 선언적인 방식의 특징이다.
 
 스크립트와 선언적인 방식의 차이점은 여전히 스크립트는 절차적으로 설정을 진행한다는 것이다. 즉, 내가 원하는 목표를 달성하기 위해 해야 하는 과정들을 빠짐없이 명시해야 한다는 것이다. 그에 반해 선언적인 방식은 과정에 관심을 갖지 않아도 되며, 최종 상태만 선언하면 된다는 장점이 있다.
 
 이 선언적인 방식을 운영체제 환경 전체에서도 적용되도록 한 것이 NixOS이다. NixOS에서는 .nix 파일을 이용해서 시스템을 빌드하며, 같은 .nix 파일과 디렉토리 구조를 갖고 있다면 기기에 상관없이 동일한 환경이 구현이 된다. 즉, 재현성이 매우 높다는 것이다. 어떤 기기에서는 작동하는 것이 다른 곳에서는 작동하지 않을 까봐 걱정하지 않아도 된다는 것이다. 왜냐하면 .nix 파일은 선언적으로 코드를 작성하며, 최종 결과를 얻기 위한 과정은 NixOS에서 자동으로 수행하기 때문이다. 의존성 관리는 덤이다.
 
 ## NixOS를 통해 tailscale 설정하기
 
 홈랩을 관리하다보면 기기마다, 또는 VM 마다 설치를 해줘야 하는 프로그램이 있는데 나의 경우에는 그것이 tailscale이다. 홈랩의 기기에서 tailscale을 활성화하려면 다음의 과정을 거쳐야 한다.
 
1. tailscale을 설치한다. (GPG키를 추가하고, 저장소를 추가하고, 패키지를 설치한다.)

2. 나의 경우 headscale을 사용하므로 headscale 서버 접속을 한다. (flag를 설정해서 tailscale up을 실행하고, headscale 서버에서 승인을 한다.)

이것을 .nix 파일을 통해 선언적으로 설정을 한다면 더 이상 수동으로 tailscale 설정을 할 필요 없이 NixOS를 설치하고 nix 파일들이 있는 저장소를 clone하기만 하면 모든 기기에서 동일한 환경을 얻을 수 있을 것이다. 
 
 NixOS에서 tailscale 설정을 하려면 configuration.nix에 다음의 코드를 추가하면 된다.

```nix
services.tailscale = {
  enable = true;
  authKey = "tskey-auth-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx";  
  extraUpFlags = [
    "--login-server=https://your-headscale-server.com"
    "--accept-routes"
    "--accept-dns"
  ];
};
```

이 방식은 headscale preauth key를 평문으로 설정파일에 작성하는데, git 저장소를 외부에 공개하지 않으면 상관 없겠지만, 만약 github 등의 공개 저장소에 push를 할 생각이면 절대 이렇게 설정해서는 안된다. 

공개 저장소에 .nix 파일들을 push하기 위해서는 이 preauthkey를 암호화할 필요가 있는데, 이를 위해서는 sops-nix를 사용하는 것이 가장 권장되는 방법이다.

## sops-nix

sops-nix를 이해하려면 우선 sops가 무엇인지 알아야 한다. sops란, Mozila에서 만든 암호화 도구로, 설정 파일 형태로 비밀을 암호화해서 저장한다. 따라서 git에 안심하고 커밋을 할 수 있게 된다. 자세한 작동 원리는 추후 별도의 글로 작성을 해보겠다.

### sops-nix 설정하기

sops-nix를 설치하려면 다음의 과정을 수행하면 된다. 이 글에서는 nix flake가 설정되어 있다고 가정한다.

flake.nix의 input에 sops-nix.url 속성을 정의할 것이다. 그리고 sops-nix 내부의 flake.nix가 호스트의 nixpkgs 버전을 따르도록 할 것이다. 마지막으로 output 함수의 반환 값 ( nested attribute set) 에서 모듈에 inputs.sops-nix.nixosModules.sops을 추가할 것이다. 예시 코드는 다음과 같다. 

```nix
{

  inputs = {
    nixpkgs.url = "github:nixos/nixpkgs/nixos-25.05";
    sops-nix.url = "github:Mic92/sops-nix";
    sops-nix.inputs.nixpkgs.follows = "nixpkgs";
  };

  outputs = inputs: {

    nixosConfigurations.nixos = inputs.nixpkgs.lib.nixosSystem {
      modules = [
        { nix.settings.experimental-features = ["nix-command" "flakes"]; }
        ./configuration.nix
        inputs.sops-nix.nixosModules.sops
      ];
    };

  };

}
```

이제 편집용 키와 시스템용 키를 생성하자.

```bash
# age 키 생성
mkdir -p ~/.config/sops/age
nix-shell -p age --run "age-keygen -o ~/.config/sops/age/keys.txt"

# 공개 키 확인
grep "# public key:" ~/.config/sops/age/keys.txt
# 결과: age1ql3z0hjhkyrw9xga8k0qqnqxjqsxlnlkny7qy7zs9p6lcxsl7z2s7n6z2x
```

```bash
# 시스템용 키 생성
sudo mkdir -p /var/lib/sops-nix
sudo nix-shell -p age --run "age-keygen -o /var/lib/sops-nix/key.txt"
sudo chmod 600 /var/lib/sops-nix/key.txt

# 공개 키 확인
sudo grep "# public key:" /var/lib/sops-nix/key.txt
# 결과: age1npylkntw643g4hwz96dts9y7pmvc05rtnk6sqm7fx2lefuv8tq6qd8ufca
```

.sops.yaml 파일을 만들자. 경로는 /etc/nixos에 하겠다.

```yaml
keys:
  - &admin_user age1ql3z0hjhkyrw9xga8k0qqnqxjqsxlnlkny7qy7zs9p6lcxsl7z2s7n6z2x  # 편집용 키
  - &nixos_system age1npylkntw643g4hwz96dts9y7pmvc05rtnk6sqm7fx2lefuv8tq6qd8ufca  # 시스템용 키

creation_rules:
  - path_regex: secrets/[^/]+\.(yaml|json|env|ini)$
    key_groups:
    - age:
      - *admin_user
      - *nixos_system
```

비밀 파일을 생성하자.

```bash
sudo mkdir secrets

# 비밀 파일 생성 및 편집
sudo HOME=/home/egowa nix-shell -p sops --run "sops secrets/tailscale.yaml"
```

```yaml
tailscale-authkey: {preauth_key}
```

이제 configuration.nix에서 다음과 같이 설정하자.

```nix
{
  # sops-nix 기본 설정
  sops.defaultSopsFile = ./secrets/tailscale.yaml;
  sops.age.keyFile = "/var/lib/sops-nix/key.txt";
  sops.age.generateKey = true;
  
  # 비밀 정의
  sops.secrets.tailscale-authkey = {
    mode = "0400";
    owner = "root";
  };
  
  # Tailscale 서비스에서 사용
  services.tailscale = {
    enable = true;
    authKeyFile = config.sops.secrets.tailscale-authkey.path;
    extraUpFlags = [
      "--login-server=https://your-headscale-server.com"
      "--accept-routes"
      "--accept-dns"
    ];
  };
}
```

변경사항을 git에 커밋하고, 시스템 재빌드를 하자.

```bash
# Git에 파일 추가 (암호화된 상태)
git add .sops.yaml
git add secrets/tailscale.yaml
git add configuration.nix
git commit -m "Add sops-nix configuration"

# 시스템 적용
sudo nixos-rebuild switch --flake .
```

마지막으로 확인을 해보자.

```bash
# 비밀이 복호화되었는지 확인
sudo ls -la /run/secrets/
sudo cat /run/secrets/tailscale-authkey

# Tailscale 연결 확인
sudo tailscale status
```

정상적으로 설정이 된 것을 확인할 수 있다!
