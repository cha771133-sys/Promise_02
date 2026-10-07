# 브랜치 전략 (Git Flow)

이 저장소는 **Git Flow** 모델을 따릅니다. 모든 작업은 아래 규칙에 맞는 브랜치에서 진행하고, 규칙에 맞는 방향으로만 머지합니다.

## 1. 브랜치 종류 및 역할

| 브랜치 | 역할 | 분기 출발점 | 머지 대상 | 수명 |
|---|---|---|---|---|
| `main` | 배포용. 항상 안정적이고 배포 가능한 코드만 존재 | — | — | 영구 |
| `develop` | 개발 통합용. 완료된 기능/수정이 모이는 곳 | `main` | — | 영구 |
| `feature/기능명` | 새 기능 개발 | `develop` | `develop` | 임시 (머지 후 삭제) |
| `fix/버그명` | 버그 수정 (급하지 않은 일반 버그) | `develop` | `develop` | 임시 (머지 후 삭제) |
| `release/버전` | 배포 준비, QA, 버전 고정 | `develop` | `main` **+** `develop` | 임시 (머지 후 삭제) |
| `hotfix/이슈명` | 프로덕션 긴급 수정 | `main` | `main` **+** `develop` | 임시 (머지 후 삭제) |

```
main     ●────────────●──────────────────●───────────────▶
          \           ▲ release/1.1.0    ▲ hotfix/xxx
           \         /│                 /│
develop     ●───●───●─●───●───●───●────●─●──────────────▶
             \   \     /   /     /
feature/A     ●───●───┘   /     /
fix/B              ●─────┘     /
feature/C                 ●────┘
```

## 2. 브랜치 네이밍 규칙

- 소문자, 하이픈(`-`)으로 단어 구분: `feature/guardian-otp-login`
- 접두사는 반드시 `feature/`, `fix/`, `hotfix/`, `release/` 중 하나
- `release/`는 버전 번호 사용: `release/1.2.0` (SemVer 권장: `MAJOR.MINOR.PATCH`)
- 가능하면 이슈 번호 포함: `feature/42-pill-shape-customizer`

## 3. 작업 흐름

### 3-1. 새 기능 개발 (`feature/*`)

```bash
git checkout develop
git pull origin develop
git checkout -b feature/기능명

# ... 작업 & 커밋 ...

git push -u origin feature/기능명
# GitHub에서 feature/기능명 → develop PR 생성
# 리뷰 승인 후 머지, 브랜치 삭제
```

### 3-2. 일반 버그 수정 (`fix/*`)

`feature/*`와 동일한 흐름. `develop`에서 분기해 `develop`으로 PR.

```bash
git checkout develop
git pull origin develop
git checkout -b fix/버그명
```

### 3-3. 배포 준비 (`release/*`)

`develop`이 다음 배포에 포함될 기능으로 충분히 채워졌을 때 생성합니다. 이 브랜치에서는 **버그 수정과 배포 준비 작업만** 허용하고 새 기능 추가는 금지합니다.

```bash
git checkout develop
git pull origin develop
git checkout -b release/1.1.0

# 버전 번호(package.json 등) 업데이트, QA, 버그 수정 커밋

# 준비 완료 후:
git checkout main
git merge --no-ff release/1.1.0
git tag -a v1.1.0 -m "v1.1.0"
git push origin main --tags

git checkout develop
git merge --no-ff release/1.1.0
git push origin develop

git branch -d release/1.1.0
git push origin --delete release/1.1.0
```

### 3-4. 긴급 수정 (`hotfix/*`)

`main`에 배포된 코드에서 치명적 문제가 발견됐을 때만 사용합니다. `develop`을 거치지 않고 `main`에서 바로 분기합니다.

```bash
git checkout main
git pull origin main
git checkout -b hotfix/이슈명

# 수정 커밋

git checkout main
git merge --no-ff hotfix/이슈명
git tag -a v1.1.1 -m "v1.1.1 hotfix"
git push origin main --tags

git checkout develop
git merge --no-ff hotfix/이슈명
git push origin develop

git branch -d hotfix/이슈명
git push origin --delete hotfix/이슈명
```

> 핵심: **hotfix는 반드시 main과 develop 양쪽에 모두 반영**해야 합니다. 그렇지 않으면 다음 release 때 수정 사항이 사라집니다.

## 4. 머지 규칙

- 모든 머지는 `--no-ff` (fast-forward 금지)로 진행해 브랜치 히스토리를 남깁니다.
- `main`, `develop`에는 직접 push하지 않고 **반드시 PR을 통해서만** 반영합니다.
- `feature/*`, `fix/*`는 머지 후 로컬/원격 브랜치를 삭제합니다.
- `main`에 머지되는 시점(release, hotfix)마다 태그(`vX.Y.Z`)를 남깁니다.

## 5. 커밋 메시지

Conventional Commits 스타일을 권장합니다.

```
feat: 보호자 OTP 인증 추가
fix: QStash 중복 알림 발송 버그 수정
docs: 브랜치 전략 문서 추가
refactor: redis 클라이언트 모듈 분리
```

## 6. 요약 치트시트

| 하고 싶은 일 | 분기 위치 | 머지 위치 |
|---|---|---|
| 새 기능 | `develop` | `develop` |
| 일반 버그 수정 | `develop` | `develop` |
| 배포 준비 | `develop` | `main` + `develop` |
| 프로덕션 긴급 수정 | `main` | `main` + `develop` |
