---
name: build
description: Feature 단위로 TDD 기반 개발을 실행한다. "/build {slug} F{n}"으로 호출. /slice로 features.md가 생성되고 사용자가 확인한 후 사용한다.
---

# Build

test-writer → implementer → refactorer → reviewer → qa 순서로 에이전트를 실행하여 Feature를 완성한다.

## 실행 규칙

- 에이전트 순서를 반드시 지킨다
- 각 에이전트는 이전 에이전트의 산출물을 컨텍스트로 전달받는다
- reviewer FAIL 시 implementer로 돌아간다 (최대 3회)
- qa FAIL 시 실패 유형에 따라 담당 에이전트로 돌아간다 (최대 3회)
- 3회 초과 실패 시 사용자에게 에스컬레이션한다
- 각 에이전트 호출 전 진행 상황을 사용자에게 알린다

## 실행 절차

### 준비

`docs/plans/{slug}/features.md` 존재 확인. (읽지 않는다)
없으면: "Feature 목록이 없습니다. `/slice {slug}`를 먼저 실행하세요."

전달 컨텍스트 준비:
```
slug: {slug}
feature: F{n}
기획서: docs/plans/{slug}/plan.md
features: docs/plans/{slug}/features.md
```

### 에이전트 루프

```
[1/5] test-writer 실행 중...
  → 실패하는 테스트 작성

[2/5] implementer 실행 중...
  → 테스트 통과까지 구현 (3-strike rule)
  → 에스컬레이션 발생 시 즉시 중단, 사용자에게 보고

[3/5] refactorer 실행 중...
  → 코드 품질 개선

[4/5] reviewer 실행 중...
  → PASS → 다음 단계
  → FAIL → implementer 재실행 (최대 3회)
         → 3회 초과 시 에스컬레이션

[5/5] qa 실행 중...
  → PASS → 완료
  → FAIL → 실패 유형별 담당 에이전트 재실행 (최대 3회)
          → 3회 초과 시 에스컬레이션
```

### 완료 보고

**PASS 시:**
```
Feature 완료 ✓

slug: {slug}
Feature: F{n} — {Feature 이름}
테스트: 통과
커버리지: {n}%
보안: 통과

ambiguous.md 확인 필요: {있음/없음}
docs/plans/{slug}/ambiguous.md

다음 단계: PR을 생성하거나 다음 Feature를 시작하세요.
  다음 Feature: /build {slug} F{n+1}
```

**에스컬레이션 시:**
```
빌드 중단 — 사용자 판단 필요

Feature: F{n}
중단 위치: {에이전트 이름}
원인: {에러 내용}

다음 중 선택해주세요:
1. 문제를 해결하고 "/build {slug} F{n}" 재실행
2. ambiguous.md를 확인하고 스펙 보완 후 재실행
3. 이 Feature를 건너뛰고 "/build {slug} F{n+1}" 실행
```
