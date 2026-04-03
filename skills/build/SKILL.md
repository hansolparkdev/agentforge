---
name: build
description: Feature 단위로 TDD 기반 개발을 실행한다. "/build {slug} F{n}"으로 호출. /slice로 features.md가 생성되고 사용자가 확인한 후 사용한다.
---

# Build

test-writer → implementer → refactorer → reviewer → qa 순서로 에이전트를 실행하여 Feature를 완성한다.

## 실행 규칙

- 에이전트 순서를 반드시 지킨다
- 각 에이전트 호출 시 이전 산출물(파일 목록, 판정 결과)을 컨텍스트로 명시적으로 전달한다
- reviewer FAIL 시 지적 사항 전체를 implementer에게 전달하여 재호출한다 (최대 3회)
- qa FAIL 시 실패 유형별 담당 에이전트를 재호출한다 (최대 3회)
- 3회 초과 실패 시 사용자에게 에스컬레이션한다
- 각 에이전트 호출 전 진행 상황을 사용자에게 알린다

## 실행 절차

### 준비

`docs/plans/{slug}/features.md` 존재 확인. (읽지 않는다)
없으면: "Feature 목록이 없습니다. `/slice {slug}`를 먼저 실행하세요."

기본 컨텍스트:
```
slug: {slug}
feature: F{n}
기획서: docs/plans/{slug}/plan.md
features: docs/plans/{slug}/features.md
```

### 에이전트 루프

**[1/5] test-writer**
전달: 기본 컨텍스트
산출물: 테스트 파일 경로 목록

---

**[2/5] implementer**
전달: 기본 컨텍스트 + test-writer가 작성한 테스트 파일 경로 목록
산출물: 구현 파일 경로 목록, 테스트 통과 수

에스컬레이션 발생 시 → 즉시 중단, 사용자에게 보고

---

**[3/5] refactorer**
전달: implementer가 작성/수정한 파일 경로 목록
산출물: 수정된 파일 경로 목록

---

**[4/5] reviewer** (reviewer_count = 0)
전달: refactorer가 전달한 파일 경로 목록

- PASS → [5/5] qa로 진행
- FAIL → reviewer_count += 1
  - reviewer_count ≤ 3: reviewer 지적 사항 전체를 implementer에게 전달 → [2/5]부터 재실행
  - reviewer_count > 3: 에스컬레이션

---

**[5/5] qa** (qa_count = 0)
전달: 기본 컨텍스트 + 전체 수정 파일 목록

- PASS → 완료 보고
- FAIL → qa_count += 1
  - qa_count ≤ 3: 실패 유형별 담당 에이전트로 재호출
    - 커버리지 미달 → [1/5] test-writer 재실행 (테스트 추가)
    - E2E / API / 보안 실패 → [2/5] implementer 재실행 (qa 실패 내용 전달)
    - 빌드/린트 실패 → [2/5] implementer 재실행
  - qa_count > 3: 에스컬레이션

### 완료 보고

**PASS 시:**
```
Feature 완료 ✓

slug: {slug}
Feature: F{n} — {Feature 이름}
테스트: 통과
커버리지: {n}%
E2E / API: 통과 (해당 시)
보안: 통과 (해당 시)

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

다음 중 선택해주세요:
1. 문제를 해결하고 "/build {slug} F{n}" 재실행
2. docs/plans/{slug}/ambiguous.md 확인 후 스펙 보완 및 재실행
3. 이 Feature를 건너뛰고 "/build {slug} F{n+1}" 실행
```
