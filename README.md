# git lfs repository

`ivi/`, `nts/` 소스 전체 아카이브 저장소.
이후 수정 없이 보관 목적이므로 두 폴더의 모든 파일을 Git LFS로 관리한다.

## 사전 준비

```bash
# git-lfs 패키지 설치 (Rocky Linux)
sudo dnf install git-lfs
```

## LFS 등록 절차

```bash
# 1. 현재 저장소에 LFS 활성화
#    .git/hooks 에 LFS용 훅(pre-push 등)을 설치한다. 저장소마다 최초 1회만 실행하면 된다.
git lfs install

# 2. 두 폴더 전체를 LFS 추적 대상으로 지정
#    "**" 는 하위 디렉토리를 포함한 모든 파일을 의미한다.
#    실행하면 추적 규칙이 .gitattributes 파일에 기록된다.
git lfs track "ivi/**"
git lfs track "nts/**"

# 3. 추적 규칙 파일을 스테이징
#    .gitattributes 가 커밋에 포함되어야 clone 하는 쪽에서도 같은 규칙이 적용된다.
git add .gitattributes

# 4. 두 폴더를 스테이징
#    2번의 규칙이 먼저 등록된 상태이므로, 이 시점에 추가되는 파일들은
#    실제 내용 대신 LFS 포인터(oid/size 텍스트)로 git에 저장되고
#    원본은 .git/lfs/objects/ 에 보관된다.
git add ivi nts

# 5. 커밋
git commit -m "add ivi, nts source archive"

# 6. 원격 저장소로 푸시
#    LFS 객체가 먼저 LFS 서버로 업로드된 뒤 커밋(포인터)이 푸시된다.
git push origin main
```

## 등록 확인

```bash
# LFS로 관리되는 파일 목록 확인 (많으면 wc -l 로 개수만 확인)
git lfs ls-files | wc -l

# 스테이징/푸시 대기 중인 LFS 객체 상태 확인
git lfs status

# 특정 파일이 포인터로 저장됐는지 확인 (oid, size 텍스트가 보이면 정상)
git show HEAD:ivi/ivi-xtiv/dat/A301S_3102 | head -3
```

## clone 시 참고

```bash
# 일반 clone: 포인터와 함께 LFS 원본 파일까지 자동으로 받는다 (전체 약 860MB)
git clone <repo-url>

# 원본 없이 포인터만 받기 (목록 확인 등 가볍게 받을 때)
GIT_LFS_SKIP_SMUDGE=1 git clone <repo-url>

# 포인터만 받은 상태에서 나중에 원본 내려받기
git lfs pull
```

## add LFS 파일

```bash
# 새로운 LFS 파일을 추가할 때는 일반적인 git add 명령을 사용하면 된다.
git add <file>

# 확인
git lfs status
git lfs ls-files | grep "ivi/some/path/new_file"

# 이후 커밋 및 푸시는 기존과 동일
git commit -m "add new LFS file"
git push origin main
```


## 기타

- `*.o`, `*.a` 빌드 산출물은 `.gitignore` 로 제외되어 관리하지 않는다.
- 추적 규칙은 `.gitattributes` 에서 확인/수정할 수 있다.
