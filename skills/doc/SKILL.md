---
name: doc
description: 프로젝트 문서 갱신. "/doc {slug}" 또는 "/doc {slug} F{n}"으로 호출.
---

# Doc

documentor 에이전트를 실행하여 `docs/project/` 산출물을 갱신한다.
/build 파이프라인과 분리되어 원하는 시점에 수동으로 실행한다.

## 호출 방식

```
/doc tic-tac-toe
/doc tic-tac-toe F3
/doc tic-tac-toe F1 F2 F3
```

## 실행 규칙

- `docs/plans/{slug}/features.md`가 없으면 실행을 중단한다
- Feature 번호를 지정하지 않으면 features.md에서 완료된(✅) Feature 전체를 대상으로 한다
- 이미 문서화된 Feature를 재실행해도 덮어쓰지 않고 갱신만 한다

## 실행 절차

### 준비

slug를 추론한다:
- `/doc tic-tac-toe` → slug = tic-tac-toe
- `/doc` → `docs/plans/` 아래 features.md가 하나면 자동 선택, 여러 개면 목록 보여주고 선택 요청

대상 Feature 결정:
- Feature 번호 지정 시: 해당 Feature만
- 미지정 시: features.md에서 `✅`가 붙은 완료 Feature 전체

### documentor 호출

**전달:**
```
slug: {slug}
작업: feature-complete
대상 Feature: [{F{n}, ...}]
기획서: docs/plans/{slug}/plan.md
features: docs/plans/{slug}/features.md
```

### 완료 보고

```
문서 갱신 완료 ✓

slug: {slug}
대상 Feature: {F{n}, ...}

갱신된 문서:
- docs/project/index.md
- docs/project/status.md
- {갱신된 나머지 파일들}
```
