+++
title = 'jellyfin user data migration (feat. sqlite3)'
date = 2025-08-12T05:35:45+09:00
draft = false 
+++

# jellyfin의 기존 local 사용자 데이터를 SSO 계정으로 마이그레이션하기 

## 시도하게 된 계기

현재 홈랩에 구성되어있는 서비스들의 계정들을 authentik을 기반으로 한 SSO 계정으로 통합하는 작업을 하고 있다.

jellyfin과 authentik을 연결을 하려고 하는데, authentik 계정으로 새로운 jellyfin 계정을 만드는 것은 가능했지만, authentik 계정을 기존의 jellyfin 계정과 연동을 하는 것이 불가능하였다.

따라서 authentik 계정을 통해서 새로운 jellyfin 계정을 생성한 뒤, DB에서 기존 jellyfin의 계정 데이터를 새로운 계정으로 마이그레이션하는 작업을 계획하였다.

## 구체적인 방법

우선 jellyfin의 계정 데이터를 다른 계정으로 옮기기 위해서는 jellyfin이 어떤 데이터베이스를 사용하는 지를 알아야 했다. 확인하는 방법은 간단했다. 나는 jellyfin을 도커 컨테이너로 구축했는데 (linuxserver.io 이미지) 컨테이너 안에서 db 파일을 찾으면 그만이다. 확인 결과 jellyfin은 DB로 sqlite3을 사용한다는 것을 알 수 있었다.

내가 마이그레이션하고 싶은 정보는 시청 기록, 마지막으로 시청을 한 위치, 재생 위치, 오디오/자막 트랙 정보 등이었는데, 이러한 정보는 library.db의 UserDatas 테이블에 있다.

이 UserDatas 테이블에서 userid attribute가 기존 계정에 속해있는 인스턴스들을 새로운 계정의 userid로 바꿔주면 마이그레이션은 끝이다. 

각 유저들의 id를 알 수 있는 방법은 다음과 같다. Users 테이블은 jellyfin.db에 존재한다.

```
sqlite3 path/to/your/jellyfin/config/data/jellyfin.db << 'EOF'
.headers on
.mode column
.width 40 20 20 25
SELECT Id, Username, AuthenticationProviderId, LastLoginDate 
FROM Users 
ORDER BY rowid DESC;
EOF
```

이를 통해 알게 된 userid로 실제로 마이그레이션하는 방법은 다음과 같다.

```
cat > migration_script.sql << 'EOF'
BEGIN TRANSACTION;
UPDATE UserDatas 
SET UserId = 'C4A02EDD-AF3A-4BFD-8C12-45AD95516469' # 새로운 계정
WHERE UserId = '72304C19-F46C-4B46-A6CF-93C3B9FC4816'; # 기존 계정
...
-- COMMIT;
EOF
```

## 주의사항

DB 수정을 하기 전에 반드시 백업을 할 것을 추천한다. DB를 잘못 건드리면 데이터가 사라질 수 있다.

```
```
``` bash

````
```
