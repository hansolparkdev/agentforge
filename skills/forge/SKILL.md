---
name: forge
description: 주제를 기획한다. "/forge {주제}"로 호출하면 Planner와 Critic이 협력하여 전략 기획서를 작성한다. 기획서가 필요할 때, 새 기능/서비스/프로젝트를 시작할 때 사용한다.
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

사용자 입력에서 주제를 추출한다. 주제가 불명확하면 한 줄로 질문한다.

```
주제: {topic}
기획서 경로: docs/plans/{topic}-plan.md
```

### 라운드 루프 (최대 3회)

**각 라운드:**

1. 사용자에게 진행 상황 알림
   ```
   Round {n}/3 시작
   Planner 실행 중...
   ```

2. **Planner 호출**
   - Round 1: 신규 기획서 작성
   - Round 2+: Critic 비평 문서를 반영하여 기획서 수정
   - 전달 컨텍스트: 주제, 이전 비평 파일 경로 (있는 경우)

3. 사용자에게 진행 상황 알림
   ```
   Planner 완료 → Critic 실행 중...
   ```

4. **Critic 호출**
   - 방금 작성된 기획서 비평
   - 전달 컨텍스트: 기획서 파일 경로, 현재 라운드 번호

5. Critic 판정 확인
   - **APPROVED** → 루프 종료, 완료 보고
   - **REJECTED** → 다음 라운드 진행 (3라운드 미만인 경우)

### 완료 보고

**APPROVED 시:**
```
기획 완료 ✓

주제: {topic}
최종 기획서: docs/plans/{topic}-plan.md
라운드: {n}회
비평 이력: docs/plans/{topic}-critique-r1.md ~ r{n}.md

다음 단계: 기획서를 바탕으로 구현을 시작하려면 architect 또는 개발 명령을 실행하세요.
```

**데드락 (3라운드 REJECTED) 시:**
```
3라운드 후에도 합의에 도달하지 못했습니다.

기획서: docs/plans/{topic}-plan.md
마지막 비평: docs/plans/{topic}-critique-r3.md

중재가 필요합니다. 다음 중 선택해주세요:
1. "/forge {topic}" — 처음부터 다시 시작
2. 마지막 비평 문서를 확인 후 수동으로 수정
3. 현재 기획서를 그대로 사용
```
