+++
title = 'GitHub Pages Blog (with Hugo)'
date = 2025-08-09T02:42:06+09:00
draft = false
+++

# 블로그를 시작한 이유 및 구체적인 제작 과정

## 🎯 왜 블로그를 시작하게 되었나?

어떤 일을 지속적으로 해나갈 때 **진행 상황에 대한 기록**이 굉장히 중요하다고 생각한다. 그렇지 않으면 내가 해왔던 것을 까먹고, 이전에 힘들게 알게 된 것을 또 다시 알기 위해서 다시 공식 문서를 뒤지고 시간을 허비하게 된다. 이때 내가 해둔 기록들이 있으면 이 기억을 다시 되찾는 데 매우 도움이 될 것이다.

기록을 남기는 방법에는 굉장히 많은 것이 있을 것이다. 가령 노트에 연필로 기록을 남길 수도 있고, 유튜브에 기록을 할 수도 있다. 나열은 못하더라도 굉장히 많은 방법이 있다는 것은 확신할 수 있다.

나는 그 많은 방법 중에서 **블로그를 통한 텍스트 기록**의 방식을 선택했는데, 이것은 내가 영상 매체보다는 텍스트를 통한 지식 습득을 선호하기도 하고, 글을 쓰면서 생각을 정리하고 체계화하는 과정 자체가 학습에 도움이 된다고 생각하기 때문이다.

## 🤔 플랫폼 선택 과정

블로그를 써야겠다는 다짐을 하고 나니 이제 구체적으로 **어떤 플랫폼을 선택할지**가 고민이었다. 선택지는 다양했다:

- **네이버 블로그**
- **티스토리**
- **WordPress**
- **GitHub Pages**
- **직접 웹사이트 구축**

### 각 플랫폼 검토 결과

#### 1. 네이버 블로그 & 티스토리

네이버 블로그나 티스토리도 충분히 좋은 플랫폼이고 매력적이지만, 마크다운으로 글을 쓰고 버전 관리를 할 수 있는 환경을 원했다. 네이버, 티스토리에서도 가능하기는 하지만 번거롭기 때문에 선택지에서 배제했다.

#### 2. 직접 웹사이트 구축  

사실 직접 웹사이트를 구축해서 블로그를 만드는 것에 굉장히 이끌렸었는데, 웹사이트 구축하는 방법을 알아보다 보니 내가 **블로그 글을 작성해서 기록을 남기는 게 목적**이었는데 점점 **웹사이트 구축이 목적**이 되는 느낌을 받았다. 주객전도가 된 꼴이다. 그래서 이 선택지도 배제했다.

#### 3. WordPress

WordPress는 속도가 좀 느려진다고 해서 배제했다. (그만큼 글을 쓸지는 모르긴 하지만. 이 이유로 GitHub Pages에서도 Hugo를 선택하게 되었다.)

#### 🏆 최종 선택: GitHub Pages

그러다 보니 선택지가 **GitHub Pages**가 남게 되었고, 분명히 단점도 있겠지만 충분히 감내할 만하다고 생각하여 블로그 만들기를 시작하게 되었다.

## 🚀 GitHub Pages + Hugo 선택 이유

### GitHub Pages란?

GitHub에서 제공하는 **무료 정적 웹사이트 호스팅 서비스**로, `{username}.github.io`라는 이름의 저장소(repository)를 만들고 설정하면 된다.

### Hugo를 선택한 이유

우선 나는 Hugo를 써서 블로그를 개설하고자 했는데 그 이유는:

1. **속도**: 글이 누적될수록 GitHub Pages에서 기본으로 지원하는 Jekyll은 속도가 느려진다고 함
2. **트렌드**: 최근엔 Hugo가 깃허브 스타 개수도 많고, 업데이트도 더 활발하게 이루어짐

## 🛠️ 구체적인 제작 과정

### 1. GitHub Public Repository 만들기

GitHub Page로 블로그를 만드려면 github 계정이 있어야 하고 github page 용 공개 저장소를 만들어야 한다. 자세한 내용은 다음 링크에서 확인이 가능하다.

📖 **GitHub Pages 빠른 시작**: [https://docs.github.com/ko/pages/quickstart](https://docs.github.com/ko/pages/quickstart)

### 2. Hugo 설치

Hugo를 통해서 GitHub Pages를 만들려면 우선 호스트에 Hugo를 설치해야 한다.

**설치 방법:**

- 패키지 매니저를 통해서 다운로드
- GitHub를 통해서 직접 다운로드

OS 별 구체적인 설치 방법은 다음 링크를 확인하면 된다.

📖 **Hugo Installation**: [https://gohugo.io/installation/](https://gohugo.io/installation/)

### 3. 저장소 설정

1. GitHub 저장소를 클론
2. 클론한 폴더에서 Hugo 초기화

```bash
hugo new site . --force
```

### 4. 개발 서버 실행

```bash
# 개발 서버 실행 (draft도 볼 수 있게 하는 옵션)
hugo server -D

# 빌드 테스트  
hugo --minify

# GitHub에 푸시
git add .
git commit -m "Add new post"
git push origin main
```

### 5. GitHub Actions 자동화

GitHub Actions을 이용하면 `git push origin main`을 할 때 바로 GitHub Pages가 빌드되도록 자동화할 수 있다.

`.github/workflows/hugo.yml` 파일을 생성하여 설정한다.

```yaml
# Hugo 사이트를 GitHub Pages에 배포하는 워크플로우
name: Deploy Hugo site to Pages

on:
  # main 브랜치에 push할 때마다 실행
  push:
    branches: ["main"]
  
  # 수동으로도 실행 가능
  workflow_dispatch:

# GitHub Pages에 배포하기 위한 권한 설정
permissions:
  contents: read
  pages: write
  id-token: write

# 동시에 여러 배포가 실행되지 않도록 제한
concurrency:
  group: "pages"
  cancel-in-progress: false

# 기본 쉘 설정
defaults:
  run:
    shell: bash

jobs:
  # 빌드 작업
  build:
    runs-on: ubuntu-latest
    steps:
      # 1. 소스 코드 체크아웃 (submodule 포함)
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive  # 테마 submodule도 함께 가져오기

      # 2. Hugo 설치
      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v3
        with:
          hugo-version: 'latest'
          extended: true  # Hugo Extended 버전 사용

      # 3. GitHub Pages 설정
      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v5

      # 4. Hugo로 사이트 빌드
      - name: Build with Hugo
        env:
          # 프로덕션 환경 설정
          HUGO_ENVIRONMENT: production
          HUGO_ENV: production
        run: |
          hugo \
            --minify \
            --baseURL "${{ steps.pages.outputs.base_url }}/"

      # 5. 빌드 결과물을 GitHub Pages용 아티팩트로 업로드
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public

  # 배포 작업
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deploy.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build  # build 작업 완료 후 실행
    steps:
      # 6. GitHub Pages에 배포
      - name: Deploy to GitHub Pages
        id: deploy
        uses: actions/deploy-pages@v4
```

## 🎨 테마 설정: Blowfish

블로그를 효과적으로 개설하려면 **테마 설정이 필수적**이다. 이 글에서는 **Blowfish 테마**를 사용한다.

### Git Submodule을 이용한 테마 적용

테마를 적용하는 방법 중 좋은 것은 **Git의 submodule**을 사용하는 것이다.

```bash
# 테마 추가
git submodule add https://github.com/nunocoracao/blowfish.git themes/blowfish
```

#### Submodule의 장점

이렇게 하면 GitHub Pages 저장소 안에 Blowfish 테마 저장소를 포함시켜서 관리할 수 있다:

- ✅ **최신 버전으로의 업데이트가 용이**
- ✅ **Git 추적 기록이 지저분해지지 않음**
- ✅ 내 저장소에서는 Blowfish 테마와 관련해서 어떤 버전의 커밋을 참조하는지에 대한 정보만 추적
- ✅ 구체적인 파일들의 변화는 추적되지 않음

#### Submodule 관리 명령어

```bash
# submodule 업데이트
git submodule update --remote themes/blowfish

# 모든 submodule 업데이트  
git submodule update --remote

# 업데이트 후 커밋
git add themes/blowfish
git commit -m "Updated theme"
```

#### Submodule 삭제하기

```bash
# 1. .gitmodules에서 해당 섹션 삭제
# 2. .git/config에서 해당 섹션 삭제
# 3. staged 파일 제거
git rm --cached themes/ananke
# 4. 실제 폴더 삭제
rm -rf themes/ananke
# 5. 커밋
git commit -m "Remove theme submodule"
```

> **참고**: 최신 Hugo에서는 Hugo Modules도 권장되나, GitHub Pages와의 호환성을 고려하면 Git Submodule이 더 안정적이다.

### 설정 파일 구성

Hugo에서 설정 파일을 관리하는 방식은 두 가지가 있다:

1. **단일 파일 방식**
2. **별도 설정 디렉토리 하에 모듈화된 여러 설정 파일 방식** ⭐

Blowfish 테마 가이드에서 제시하는 방법은 **2번**이다.

#### 설정 파일 마이그레이션

1. Hugo를 처음 초기화하면 프로젝트 루트 디렉토리에 단일 설정 파일이 생성됨
2. 이 파일을 삭제
3. `./config/_default/`에 다중 설정 파일을 생성

#### 예시 파일 복사

설정을 쉽게 할 수 있도록 Blowfish에서는 이미 설정 변수와 예시들을 적어둔 예시 파일들이 존재한다.

**복사 과정:**

```bash
# 예시 파일 위치: /themes/blowfish/config/_default/
# 복사 대상: /config/_default/
```

각 파일별 설정 항목들의 의미와 구체적인 설정 과정 및 그 외 blowfish 관련 모든 내용은 **공식 Blowfish 테마 문서**에 자세히 설명되어 있다.

📖 **공식 Blowfish 문서 페이지**: [https://blowfish.page/](https://blowfish.page/)

## ✍️ 첫 글 작성해보기

여기까지 왔으면 이제 제법 그럴싸한 블로그가 만들어졌을 것이다. 이제 첫 글을 써보자!

```bash
# 새 글 생성
hugo new posts/my-first-post.md

# 글 작성 후 미리보기
hugo server -D

# 만족스러우면 배포
git add .
git commit -m "Add first blog post"
git push origin main
```

## 📝 추가로 설정할 내용들

다음과 같은 내용들은 별도로 설정이 필요하다:

- GitHub Page에서 GitHub Action 사용 설정 (Settings → Pages → Build and deployment에서 Source를 GitHub Actions으로 변경)
- 로컬에서 Git 설정 (SSH키 설정이나 Personal Access Token 설정 - 비밀번호 인증 지원을 이제 안하므로)

## 🎉 마무리

GitHub Pages와 Hugo를 이용한 블로그 개설이 완료되었다. 이제 꾸준히 글을 쓰며 지식을 기록하고 공유할 수 있는 플랫폼이 마련되었다.

### 다음 계획

- [ ] 블로그 디자인 커스터마이징
- [ ] SEO 최적화 설정  
- [ ] 댓글 시스템 추가
- [ ] 정기적인 포스팅 습관 만들기

---

**참고 자료:**

- [Hugo 공식 문서](https://gohugo.io/)
- [Blowfish 테마 문서](https://blowfish.page/)
- [GitHub Pages 가이드](https://pages.github.com/)