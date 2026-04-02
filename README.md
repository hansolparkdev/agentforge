# agentforge

Claude Code plugin marketplace. `/plan` 스킬 하나로 McKinsey/BCG 수준의 전략 기획서를 자동 작성한다.

Planner가 기획서를 쓰고, Critic이 비평하고, Planner가 수정하는 루프를 최대 3라운드 반복한다.

## 설치

```
# 1. agentforge를 marketplace로 등록
/plugin marketplace add hansolparkdev/agentforge

# 2. plugin 설치
/plugin install agentforge@agentforge
```

설치 후 프로젝트에 다음 파일이 생성된다:
- `.claude/agents/planner.md`
- `.claude/agents/critic.md`
- `.claude/skills/plan/SKILL.md`

## 사용법

```
/plan 틱택토 게임
/plan SNS 플랫폼
/plan 회의록 자동화 SaaS
```

실행 흐름:

```
/plan {주제}
  └── Round 1
  │     ├── Planner → docs/plans/{topic}-plan.md 생성
  │     └── Critic  → docs/plans/{topic}-critique-r1.md 생성
  │           ├── APPROVED → 완료
  │           └── REJECTED → Round 2
  └── Round 2~3 (최대)
        ├── Planner → 비평 반영하여 기획서 수정
        └── Critic  → 재비평
              ├── APPROVED → 완료
              └── 3라운드 REJECTED → 사용자 중재 요청
```

## 산출물

```
docs/plans/
├── {topic}-plan.md              # 최종 기획서
├── {topic}-critique-r1.md      # 1라운드 비평
├── {topic}-critique-r2.md      # 2라운드 비평 (있는 경우)
└── {topic}-critique-r3.md      # 3라운드 비평 (있는 경우)
```

### 기획서 구조

| 섹션 | 내용 |
|------|------|
| Executive Summary | 핵심 결론 먼저 (Pyramid Principle) |
| 상황 분석 (SCQA) | Situation → Complication → Question → Answer |
| Issue Tree | MECE 분해, ASCII 다이어그램 |
| 전략적 권고사항 | 3-5개, 선택 근거 포함 |
| 시스템 아키텍처 | 기술 스택, 컴포넌트, 데이터 모델, API |
| 구현 로드맵 | Phase별, 파일/모듈 단위 체크리스트 |
| 위험 요인 & 대응책 | 수준(낮음/중간/높음) + 대응 |
| 완료 조건 (DoD) | 검증 가능한 조건 5개 이상 |

## Agents

### planner

전략 기획자. 비즈니스 맥락과 기술 아키텍처를 모두 포함한 기획서를 작성한다.

- **모델:** claude-opus-4-6
- **도구:** Read, Glob, Grep, WebSearch, WebFetch, Write
- **호출 방식:** `/plan` 스킬을 통해서만 실행. 직접 호출하지 않는다.

### critic

비평가. Planner가 작성한 기획서를 전략과 아키텍처 두 관점에서 비평하고 통과/반려를 판정한다.

- **모델:** claude-opus-4-6
- **도구:** Read, Glob, Grep, WebSearch, WebFetch, Write
- **호출 방식:** `/plan` 스킬을 통해서만 실행. 직접 호출하지 않는다.
- **통과 기준:** 8개 섹션 중 재작성 필요(❌) 0개, 보완 필요(⚠️) 2개 이하

## Skills

### /plan

Planner-Critic 루프를 오케스트레이션하는 스킬.

- 최대 3라운드
- Critic이 APPROVED를 내리면 즉시 종료
- 3라운드 후에도 REJECTED면 사용자에게 중재 요청

## 저장소 구조

```
agentforge/
├── .claude-plugin/
│   └── marketplace.json
└── plugins/
    └── agentforge/
        ├── .claude-plugin/
        │   └── plugin.json
        ├── agents/
        │   ├── planner.md
        │   └── critic.md
        └── skills/
            └── plan/
                └── SKILL.md
```
