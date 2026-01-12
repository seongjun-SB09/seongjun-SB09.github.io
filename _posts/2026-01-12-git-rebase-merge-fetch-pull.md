---
layout: post
title: "git rebase vs merge, fetch vs pull 차이 정리"
---

Git을 쓰다 보면 명령어는 익숙한데  
막상 **“차이가 뭐야?”** 라고 물으면 말이 막히는 경우가 많다.

나 역시 헷갈렸던 개념 두 가지를 정리해보며 다시 한 번 상기해본다.

- `git rebase` 와 `git merge`의 차이  
- `git fetch` 와 `git pull`의 차이  

이번 글에서는 **각 명령어의 특징과 어떤 상황에서 사용하는 것이 적절한지**를 중심으로 정리한다.


## 1️⃣ git merge vs git rebase

### 🔹 git merge란?

`git merge`는 **두 브랜치의 작업 내용을 하나로 합치는 방식**이다.

```bash
git checkout main
git merge feature/login
```

이 방식은 새로운 merge commit을 생성하며,
각 브랜치의 커밋 히스토리를 그대로 보존한다.

#### 특징
- 브랜치의 작업 흐름이 명확하게 남는다
- 누가 언제 어떤 작업을 병합했는지 추적하기 쉽다

#### 장점
- 히스토리가 안전하게 보존된다
- 협업 시 예측하기 쉽다

#### 단점
- merge commit이 많아지면 히스토리가 복잡해질 수 있다

#### 사용하기 좋은 상황
- 여러 명이 함께 작업하는 협업 환경
- 이미 원격 저장소에 공유된 브랜치
- 작업 이력을 중요하게 관리해야 할 때


### 🔹 git rebase란?

`git rebase`는 **기준 브랜치를 변경해 커밋 히스토리를 다시 쌓는 방식**이다.

```bash
git checkout feature/login
git rebase main
```

이 명령을 실행하면  
`feature/login` 브랜치의 커밋들이  
`main` 브랜치 최신 커밋 뒤에 순서대로 다시 적용된다.

#### 특징
- 커밋 히스토리가 한 줄로 깔끔해진다
- 기존 커밋의 해시 값이 변경된다

#### 장점
- 로그가 매우 깔끔해진다
- 개인 브랜치 관리에 적합하다

#### 단점
- 이미 공유된 브랜치에서 사용하면 위험하다
- 충돌 발생 시 해결 난이도가 높다

#### 사용하기 좋은 상황
- 혼자 작업하는 feature 브랜치
- PR 생성 전 커밋 정리
- 아직 원격 저장소에 push 하지 않은 커밋

---

### 🔑 git merge vs git rebase 정리

| 구분 | git merge | git rebase |
|---|---|---|
| 히스토리 | 유지됨 | 다시 정렬 |
| merge commit | 생성 | 미생성 |
| 협업 안정성 | 높음 | 낮음 |
| 개인 작업 | 가능 | 매우 적합 |

📌 **정리**
- 공유된 브랜치 → `git merge
- 개인 브랜치 정리 → `git rebas`

---

## 2️⃣ git t fch vs git pull

### 🔹 git fetch란?

`git fetch`는 **원격 저장소의 변경 사항을 로컬로 가져오기만 하는 명령어**다.

```bash
git fetch origin
```

#### 특징
- 원격 저장소의 상태만 갱신된다
- 코드 병합은 직접 선택해야 한다

#### 장점
- 매우 안전하다
- 변경 사항을 미리 확인할 수 있다

#### 사용하기 좋은 상황
- 협업 중 원격 변경 사항 확인
- 바로 병합하고 싶지 않을 때
- 충돌 가능성이 있을 때


### 🔹 git pull이란?

`git pull`은  
**`git fetch` + `git merge`를 한 번에 수행하는 명령어**다.

```bash
git pull origin main
```

#### 장점
- 빠르고 간편하다
- 바로 최신 상태로 동기화된다

#### 단점
- 원치 않는 merge가 발생할 수 있다
- 변경 내용을 확인할 시간이 없다

#### 사용하기 좋은 상황
- 혼자 작업하는 프로젝트
- 변경 사항이 단순한 경우
- 빠르게 최신 상태로 맞추고 싶을 때

```bash
git diff origin/main
git merge origin/main
```

---

### 🔑 git fetch vs git pull 정리

| 구분 | git fetch | git pull |
|---|---|---|
| 코드 반영 | ❌ | ⭕ |
| 안전성 | 높음 | 상대적으로 낮음 |
| 협업 추천 | ⭕⭕ | ⭕ |

📌 **협업 시 추천 흐름**
```bash
git fetch origin
git diff origin/main
git merge origin/main
```
