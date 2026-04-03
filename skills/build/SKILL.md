---
name: build
description: TDD 기반 개발을 실행한다. "/build {slug} F{n}", "/build F1", "/build 로그인 만들어줘" 등 다양한 형태로 호출 가능. slug나 Feature 번호가 불명확하면 docs/plans/ 구조를 보고 스스로 추론한다.
---

# Build

test-writer → implementer → refactorer → reviewer → qa → documentor 순서로 에이전트를 실행하여 Feature를 완성한다.

## 호출 방식

### A. 기획서 기반
```
/build tic-tac-toe F1
/build login F2
```
`docs/plans/{slug}/features.md`의 Feature 정의를 기준으로 동작한다.

### B. 자유형 (기획서 없이)
```
/build 결제 페이지 UI 구현
```
작업 설명을 Feature 정의로 사용한다.

## 실행 규칙

- 에이전트 순서를 반드시 지킨다
- 각 에이전트 호출 시 이전 산출물(파일 목록, 판정 결과)을 컨텍스트로 명시적으로 전달한다
- reviewer FAIL 시 지적 사항 전체를 implementer에게 전달하여 재호출한다 (최대 3회)
- qa FAIL 시 실패 유형별 담당 에이전트를 재호출한다 (최대 3회)
- 3회 초과 실패 시 사용자에게 에스컬레이션한다
- 각 에이전트 호출 전후 시각을 기록하여 완료 보고에 포함한다

## 실행 절차

### 준비

**slug/Feature 번호 추론:**
입력이 정확하지 않아도 스스로 파악한다.
- `/build F1` → `docs/plans/` 아래 features.md가 하나면 그것 사용
- `/build tic-tac-toe` → slug는 tic-tac-toe, features.md에서 미완료 Feature 중 첫 번째
- `/build 로그인 만들어줘` → `docs/plans/login/features.md` 존재 확인, 없으면 자유형으로 처리
- 여러 slug가 있고 불명확하면 목록 보여주고 선택 요청

**A. 기획서 기반인 경우:**
- `docs/plans/{slug}/features.md` 존재 확인 (읽지 않는다)
- 없으면: "Feature 목록이 없습니다. `/slice {slug}`를 먼저 실행하세요."

**B. 자유형인 경우:**
- features.md 없어도 진행
- 작업 설명을 그대로 Feature 정의로 사용

`docs/project/status.md` 확인 (있으면):
- `진행 중: {slug} F{n} — {에이전트명}` 항목이 있으면 사용자에게 알린다:
  ```
  이전 빌드가 중단된 기록이 있습니다.
  중단 위치: {slug} F{n} — {에이전트명}
  이어서 진행하시겠습니까? (y/n)
  ```
  - y → 해당 에이전트부터 재개
  - n → 처음부터 시작

기본 컨텍스트:
```
slug: {slug}
feature: {F{n} | 작업 설명}
기획서: docs/plans/{slug}/plan.md (있는 경우)
features: docs/plans/{slug}/features.md (있는 경우)
```

### status.md 업데이트 규칙

각 에이전트 시작 전에 `docs/project/status.md`를 갱신한다:
```
진행 중: F{n} — {에이전트명}
```

qa PASS 후 documentor 완료 시:
```
완료: F{n} 추가
진행 중: 없음
```

### 에이전트 루프

각 에이전트 시작 시 시각 기록: `{에이전트명}_start = 현재 시각`
각 에이전트 완료 시 시각 기록: `{에이전트명}_end = 현재 시각`

**[1/6] test-writer**
전달: 기본 컨텍스트
산출물: 테스트 파일 경로 목록

---

**[2/6] implementer**
전달: 기본 컨텍스트 + test-writer가 작성한 테스트 파일 경로 목록
산출물: 구현 파일 경로 목록, 테스트 통과 수

에스컬레이션 발생 시 → 즉시 중단, 사용자에게 보고

---

**[3/6] refactorer** (조건부 실행)
수정 파일이 3개 이하이고 implementer가 "단순 구현"으로 완료한 경우 → 건너뛰고 [4/6]으로
그 외 → 실행
전달: implementer가 작성/수정한 파일 경로 목록
산출물: 수정된 파일 경로 목록

---

**[4/6] reviewer** (reviewer_count = 0)
전달: refactorer가 전달한 파일 경로 목록

- PASS → [5/6] qa로 진행
- FAIL → reviewer_count += 1
  - reviewer_count ≤ 3: reviewer 지적 사항 전체를 implementer에게 전달 → [2/6]부터 재실행
  - reviewer_count > 3: 에스컬레이션

---

**[5/6] qa** (qa_count = 0)
전달: 기본 컨텍스트 + 전체 수정 파일 목록

- PASS → [6/6] documentor로 진행
- FAIL → qa_count += 1
  - qa_count ≤ 3: 실패 유형별 담당 에이전트로 재호출
    - 커버리지 미달 → [1/6] test-writer 재실행 (테스트 추가)
    - E2E / API / 보안 실패 → [2/6] implementer 재실행 (qa 실패 내용 전달)
    - 빌드/린트 실패 → [2/6] implementer 재실행
  - qa_count > 3: 에스컬레이션

---

**[6/6] documentor**
전달: 기본 컨텍스트 + qa 결과(커버리지 수치 포함) + 전체 수정 파일 목록
산출물: 갱신된 docs/project/ 파일 목록 (index.md 포함)

documentor 완료 후 **반드시** `docs/plans/{slug}/features.md`의 해당 Feature Tasks를 모두 `- [x]`로 업데이트한다.

### 완료 보고

**PASS 시:**
```
Feature 완료 ✓

Feature: F{n} — {Feature 이름}
테스트: 통과
커버리지: {n}%
E2E / API: 통과 (해당 시)
보안: 통과 (해당 시)

에이전트 소요 시간:
| 에이전트     | 소요 시간 |
|------------|---------|
| test-writer  | {n}s   |
| implementer  | {n}s   |
| refactorer   | {n}s   |
| reviewer     | {n}s   |
| qa           | {n}s   |
| documentor   | {n}s   |
| 합계          | {n}s   |

산출물 갱신:
- docs/project/index.md
- docs/project/status.md
- docs/project/coverage.md
- {갱신된 나머지 docs/project/ 파일들}

ambiguous.md 확인 필요: {있음 → docs/plans/{slug}/ambiguous.md | 없음}

다음 단계: PR을 생성하거나 다음 Feature를 시작하세요.
  다음 Feature: /build {slug} F{n+1}
```

**에스컬레이션 시:**
```
빌드 중단 — 사용자 판단 필요

Feature: F{n}
중단 위치: {에이전트 이름}
시도 횟수: {n}회
원인: {에러/이슈 내용}

진행 상황은 docs/project/status.md에 저장되었습니다.

다음 중 선택해주세요:
1. 문제를 해결하고 "/build {slug} F{n}" 재실행
2. docs/plans/{slug}/ambiguous.md 확인 후 스펙 보완 및 재실행
3. 이 Feature를 건너뛰고 "/build {slug} F{n+1}" 실행
```
