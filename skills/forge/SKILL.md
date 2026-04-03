---
name: forge
description: 전략 기획서 작성. "/forge {주제}"로 호출.
---

# Forge

Planner와 Critic을 순차적으로 실행하여 기획서를 완성한다. 최대 3라운드.

## 실행 규칙

- Planner → Critic 순서를 반드시 지킨다
- 각 Agent 호출은 이전 산출물 파일 경로를 컨텍스트로 전달한다
- Critic이 APPROVED를 내리면 3라운드 전이라도 즉시 종료한다
- 3라운드 후에도 REJECTED면 데드락으로 판단하고 사용자에게 중재를 요청한다
- 각 Agent 호출 전에 현재 진행 상황을 사용자에게 알린다

## 실행 절차

### 준비

**CLAUDE.md 확인:**
- 존재하지 않으면 작성을 도와준다:
  ```
  CLAUDE.md가 없습니다. 기획서 작성 전에 프로젝트 기본 정보가 필요합니다.
  아래 내용을 알려주시면 CLAUDE.md를 생성해드립니다:
  1. 기술 스택 (언어, 프레임워크)
  2. 프로젝트 유형 (Frontend / Backend / Fullstack)
  3. 코드 컨벤션 (있으면)
  ```
  사용자 응답 받으면 CLAUDE.md 생성 후 진행.
- 존재하면 기술 스택을 읽어 Planner에 전달한다.

사용자 입력에서 주제를 추출한다. 주제가 불명확하면 한 줄로 질문한다.

주제를 kebab-case slug로 변환하여 Planner에 전달한다.
- "틱택토 게임" → `tic-tac-toe`
- "다크모드" → `dark-mode`
- "로그인 기능" → `login`

기획서 경로: `docs/plans/{slug}/plan.md`

### 라운드 루프 (최대 3회)

1. 사용자에게 진행 상황 알림:
   ```
   Round {n}/3 시작 — Planner 실행 중...
   ```

2. **Planner 호출**
   - Round 1: 신규 기획서 작성
   - Round 2+: 비평 반영하여 수정
   - 전달: 주제, slug, 이전 Critic 비평 내용 (있는 경우 — 파일 아닌 텍스트로 전달)

3. 사용자에게 진행 상황 알림:
   ```
   Planner 완료 → Critic 실행 중...
   ```

4. **Critic 호출**
   - 전달: `docs/plans/{slug}/plan.md`, 라운드 번호

5. Critic 판정 확인
   - **APPROVED** → Planner를 재호출하여 features.md 생성
   - **REJECTED** → 다음 라운드 (3라운드 미만인 경우)

6. **APPROVED 후 — Planner 재호출 (features.md 생성)**
   전달: slug, `docs/plans/{slug}/plan.md`, 작업 = "features-init"

### 완료 보고

**APPROVED 시:**

```
기획 완료 ✓

주제: {topic}
slug: {slug}
최종 기획서: docs/plans/{slug}/plan.md
features: docs/plans/{slug}/features.md
라운드: {n}회

다음 단계: /build {slug} F1
```

**데드락 (3라운드 REJECTED) 시:**
```
3라운드 후에도 합의에 도달하지 못했습니다.

기획서: docs/plans/{slug}/plan.md
마지막 비평: {Critic의 마지막 REJECTED 내용 요약}

다음 중 선택해주세요:
1. "/forge {topic}" — 처음부터 다시 시작
2. 위 비평을 참고하여 기획서를 수동으로 수정 후 "/slice {slug}"
3. 현재 기획서를 그대로 사용 후 "/slice {slug}"
```
