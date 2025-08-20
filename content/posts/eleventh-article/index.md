+++
title = 'NixOS에서 home manager 사용하기'
date = 2025-08-20T22:02:57+09:00
draft = false
+++

![home manager github 그림](featured.png)

## home manager

NixOS에서 home manager는 시스템과 사용자 환경 중에 사용자 환경을 선언적으로 구성할 수 있게 해주는 도구이다. 독립적으로 설치할 수도 있고 flake를 통해서 설치할 수도 있는데, 일반적으로 flake를 통한 설치가 권장되는데, 이는 flake가 버전 고정을 명확히 할 수 있어서 재현성을 높일 수 있기 때문이다.

configuration.nix 에서도 홈 설정을 선언적으로 구성할 수는 있지만, 바람직하지 않다. 왜냐하면 시스템 설정이라는 것은 인프라적인 부분을 구성하는 것이고 홈 설정은 말단의 사용자의 환경을 구성하는 것이니 범주도 다를 뿐더러, configuration.nix를 통해 생성된 파일들은 root가 소유하게 되어 일반 사용자가 접근하기 불편해지는 상황이 발생한다. 개인적으로 쓰려고 설치한 패키지인데 sudo 명령어를 사용하는 등의 과정을 추가해야한다면 매우 번거로울 것이다.

실질적인 불편함 외에도 설정 범위에 따라서 nix 파일을 모듈식 구조로 분리하는 것이 관리하기에도 편하다. 나중에 다른 기기로 환경을 옮기려고 할 때 새로운 기기의 상황에 따라 특정 모듈을 적용하고 특정 모듈은 적용하지 않는 식의 유연함을 챙길 수 있다. 하나의 nix 파일에서 모든 설정을 관리하면 추후 이런 설정 관련 조정이 불편해진다. 적용이 되지 않는 코드를 찾아서 주석 처리 등을 해줘야 하는데 상상만 해도 상당히 불편하다.

home manager는 사용자 환경 관리에 특화되어있는 도구이며, 그러니 권한 문제 등의 문제가 발생하지 않고, 나아가서 NixOS 설정을 모듈화하는 데에 큰 도움을 준다. 또한, home manager는 꼭 NixOS에서만 쓸 수 있는 것이 아니며, 다른 OS에서도 사용이 가능하므로 home manager를 통해서 사용자 환경을 관리하면 배포에도 매우 유리하다.

조금 더 설명을 하자면, configuration.nix에서는 단순 선언으로 설정할 수 없는 것들을 home manager를 통해서 관리할 수도 있다. 그러니 여러모로 홈 환경 관리에는 home manager를 사용하지 않을 이유가 없다.

## NixOS에서 home manager를 도입하기

그렇다면 NixOS에서 home manager를 추가해보자. 이번 글에서는 home manager를 flake module로서 설치를 해볼 것이다. 

다른 모듈을 추가하듯이, flake.nix의 inputs attribute set에 모듈에 대한 url 속성, inputs.nixpkgs.follows 속성을 정의해주면 된다. 그 다음 outputs function body에서 modules list에 home-manager.nixosModules.home-manager를 추가한다. 이번에는 추가 옵션도 같이 제공할 예정이다. 

flake.nix 에 다음의 내용을 추가하자.

```nix
{
  inputs = {
    ...
    home-manager = {
      url = "github:nix-community/home-manager/release-25.05";
      inputs.nixpkgs.follows = "nixpkgs";
    };
    ...
  };
  
  outputs = inputs: {
    nixosConfigurations.nixos = inputs.nixpkgs.lib.nixosSystem {
      modules = [
        ...
        inputs.home-manager.nixosModules.home-manager
        {
          home-manager.useGlobalPkgs = true; # home manager downloads packages via system's nixpkgs
          home-manager.useUserPackages = true; # to make pkgs downloaded to system (shared to users)
          home-manager.users.your-name = import ./home.nix; # your-name user configuration
          home-manager.extraSpecialArgs = { inherit inputs; }; # additional options
        }
      ];
    };
  };

}
```

config 부분에서 각 항목과 설정값을 둔 이유는 다음과 같다.

- useGlobalPkgs: 시스템과 동일한 nixpkgs 사용한다. (메모리 효율성, 일관성)
- useUserPackages: 패키지를 시스템 프로파일에 설치한다. (가비지 컬렉션 효율성)
- extraSpecialArgs: flake inputs를 home.nix에서 사용 가능하게 한다.

flake.nix 를 작성했으면 이제 home.nix 파일을 만들자. (home-manager.users.your-name = import ./home.nix; 여기서 home.nix의 경로가 정해졌다. 만약 다른 경로로 변경했다면 변경한 주소로 가서 home.nix를 만들자.

이번 글에서는 neovim, vscode, alacritty를 home manager를 통해서 설정해보겠다.

```nix

# home.nix
{ config, pkgs, inputs, ... }:

{
  home.username = "egowa";
  home.homeDirectory = "/home/egowa";

  home.stateVersion = "25.05";

  home.packages = with pkgs; [
    neovim
    vscode
  ];

  programs = {
    home-manager.enable = true;

    alacritty = { # enable을 쓰면 단순한 패키지 설치를 넘어 기본값으로 되어있는 설정들도 같이 적용이 된다. 여러모로 enable이 가능하면 enable을 사용하는 것이 편하다.
    enable = true;
    };
  };

  services = {
    ssh-agent.enable = true;
  };
}
```

home.stateVersion = "25.05"; 의 옵션의 경우엔 하위 호환성을 위한 것이다. 예를 들어 Home Manager 버전이 올라가면서 패키지의 enable 기본 설정값이 달라질 수 있는데, home.stateVersion을 고정해두면 25.05 Home Manager 버전에서의 패키지 enable 기본 값을 사용할 수 있어 호환 문제가 없어진다. 현재 패키지가 갖고 있는 설정만 건드릴 수 있다.

home.nix 파일을 작성했다면 바로 git 커밋을 하기 전에 문법이 옳은 지를 먼저 테스트하자. 다음의 명령어로 가능하다. 

```console

$ nix-instantiate --parse /etc/nixos/flake.nix

```

별도의 오류가 뜨지 않으면 문법 문제는 없다는 것이다. flake는 git으로 추적되는 파일만 고려하므로 git 커밋을 하자.

```bash
git add .
git commit -m "Add home-manager configuration"
```

만약 git 명령어에서 권한 오류가 발생한다면:
`sudo chown -R $USER:$USER /etc/nixos` 을 입력하자.
(선언적이고 재현가능한 방식은 아니지만, 테스트 하기엔 유용하고, 마이그레이션 시에도 명령어 한 줄만 입력하면 된다. 기억하기도 어렵지 않다.)

그 다음 빌드 테스트만 해보자 (실제 적용은 아직 하지 않는다.) 

```bash
sudo nixos-rebuild dry-build --flake .
```

문제가 없으면 빌드 및 switch 까지 완료하자.

```bash
sudo nixos-rebuild switch --flake .
```

빌드가 완료되고 나면 실제로 프로그램들이 설치되고 작동하는지 확인해보자.

```bash
nvim
code
alacritty
```

정상적으로 프로그램이 실행되는 것을 알 수가 있다!

## 마무리

이번 글에서는 NixOS에서 flake과 같이 가장 많이 쓰이는 모듈인 home manager를 소개했다. home manager 를 통해서 홈 환경을 선언적으로 하고, 모듈화를 하고, 풍부한 옵션들을 활용해서 NixOS 환경을 더 NixOS 답게 할 수 있는 도구이니 꼭 다들 한번씩 써봤으면 좋겠다. 

서두에서 언급했듯이 home manager는 NixOS에서만 사용가능한 것이 아니니 가볍게 시도해볼 수 있다. 다만 이 글에서 소개한 flake를 통한 방법이 아닌 다른 방법으로 설치를 해야 한다.

단순히 패키지 설정만 건드릴 수 있는 게 아니라 다른 사람들이 github 등에 올린 dotfile 등을 home manager로 쉽게 가져와서 적용을 해볼 수도 있으니 이런 것도 시도해보면 좋을 듯 하다.

