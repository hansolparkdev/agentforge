---
name: build
description: Feature 단위 TDD 개발. "/build {slug} F{n}" 또는 자연어로 호출 가능.
---

# Build

test-writer → implementer → reviewer → qa 순서로 에이전트를 실행하여 Feature를 완성한다.

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

기본 컨텍스트:
```
slug: {slug}
feature: {F{n} | 작업 설명}
기획서: docs/plans/{slug}/plan.md (있는 경우)
features: docs/plans/{slug}/features.md (있는 경우)
```

### 에이전트 루프

각 에이전트 시작 시 시각 기록: `{에이전트명}_start = 현재 시각`
각 에이전트 완료 시 시각 기록: `{에이전트명}_end = 현재 시각`

**[1/4] test-writer**
전달: 기본 컨텍스트
산출물: 테스트 파일 경로 목록

---

**[2/4] implementer**
전달: 기본 컨텍스트 + test-writer가 작성한 테스트 파일 경로 목록
산출물: 구현 파일 경로 목록, 테스트 통과 수

에스컬레이션 발생 시 → 즉시 중단, 사용자에게 보고

---

**[3/4] reviewer** (reviewer_count = 0)
전달: implementer가 작성/수정한 파일 경로 목록

- PASS → [4/4] qa로 진행
- FAIL → reviewer_count += 1
  - reviewer_count ≤ 3: reviewer 지적 사항 전체를 implementer에게 전달 → [2/4]부터 재실행
  - reviewer_count > 3: 에스컬레이션

---

**[4/4] qa** (qa_count = 0)
전달: 기본 컨텍스트 + 전체 수정 파일 목록

- PASS → features.md Tasks 업데이트 후 완료 보고
- FAIL → qa_count += 1
  - qa_count ≤ 3: 실패 유형별 담당 에이전트로 재호출
    - 커버리지 미달 → [1/4] test-writer 재실행 (테스트 추가)
    - E2E / API / 보안 실패 → [2/4] implementer 재실행 (qa 실패 내용 전달)
    - 빌드/린트 실패 → [2/4] implementer 재실행
  - qa_count > 3: 에스컬레이션

---

**qa PASS 후 — features.md Tasks 업데이트**

`docs/plans/{slug}/features.md`가 있는 경우 **반드시** 해당 Feature의 모든 Tasks를 `- [ ]` → `- [x]`로, Feature 헤더를 `### F{n}.` → `### ✅ F{n}.`으로 직접 수정한다.

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
| reviewer     | {n}s   |
| qa           | {n}s   |
| 합계          | {n}s   |

ambiguous.md 확인 필요: {있음 → docs/plans/{slug}/ambiguous.md | 없음}

다음 단계: PR을 생성하거나 다음 Feature를 시작하세요.
  다음 Feature: /build {slug} F{n+1}
  문서 갱신이 필요하면: /doc {slug}
```

**에스컬레이션 시:**
```
빌드 중단 — 사용자 판단 필요

Feature: F{n}
중단 위치: {에이전트 이름}
시도 횟수: {n}회
원인: {에러/이슈 내용}

다음 중 선택해주세요:
1. 문제를 해결하고 "/build {slug} F{n}" 재실행
2. docs/plans/{slug}/ambiguous.md 확인 후 스펙 보완 및 재실행
3. 이 Feature를 건너뛰고 "/build {slug} F{n+1}" 실행
```
