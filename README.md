# agentforge

Claude Code plugin marketplace. 기획부터 TDD 개발까지 전 과정을 자동화하는 에이전트 세트.

## 설치

```
# 1. agentforge를 marketplace로 등록
/plugin marketplace add hansolparkdev/agentforge

# 2. plugin 설치 (project scope 선택)
/plugin install agentforge@agentforge
```

설치하면 `~/.claude/plugins/cache/`에 파일이 저장되고, 프로젝트의 `.claude/settings.json`에 활성화 등록된다.

## 전체 워크플로우

```
/forge {주제}          # 전략 기획서 작성
  └── docs/plans/plan.md 생성
  └── docs/project/architecture.md 초기화

/slice                 # Feature 단위로 분해
  └── docs/plans/features.md 생성

/build F1              # Feature 단위 TDD 개발
/build F2
...
```

## Skills

### /forge {주제}

Planner → Critic 루프로 전략 기획서를 작성한다. 최대 3라운드.

```
/forge 틱택토 게임
/forge SNS 플랫폼
/forge 회의록 자동화 SaaS
```

**산출물:**
```
docs/plans/
├── plan.md              # 최종 기획서 (8개 섹션)
├── critique-r1.md       # 1라운드 비평
└── critique-r2.md       # 2라운드 비평 (있는 경우)

docs/project/
└── architecture.md      # 시스템 아키텍처 초기 문서
```

**기획서 구조:**

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

---

### /slice

기획서를 독립적인 Feature 단위로 분해한다.

```
/slice
```

**산출물:**
```
docs/plans/
└── features.md    # Feature 목록 (수동 편집 후 /build 사용)
```

features.md 확인 후 `/build`로 넘어간다.

---

### /build F{n}

Feature 하나를 TDD로 완성한다. 에이전트 파이프라인을 순서대로 실행한다.

```
/build F1
/build F2
```

**에이전트 파이프라인:**
```
test-writer → implementer → refactorer → reviewer → qa → documentor
```

**실패 시 자동 재시도:**
- reviewer FAIL → implementer 재실행 (최대 3회)
- qa FAIL → 실패 유형별 담당 에이전트 재실행 (최대 3회)
- 3회 초과 → 사용자 에스컬레이션

**중단 재개:**
`docs/project/status.md`에 진행 상황이 저장된다. `/build F{n}` 재실행 시 중단 위치부터 이어서 진행할지 선택할 수 있다.

**산출물 (Feature 완료마다 갱신):**
```
docs/project/
├── status.md        # 진행 현황, 완료된 Feature 목록
├── architecture.md  # 시스템 아키텍처
├── api.md           # API 명세
├── schema.md        # DB 스키마
├── components.md    # 컴포넌트/모듈 구조
├── sequence.md      # 시퀀스 다이어그램
├── security.md      # 보안 검토 결과
├── coverage.md      # 테스트 커버리지 이력
└── decisions.md     # 기술 결정 이력 (모호한 스펙 선판단 기록)
```

## Agents

| 에이전트 | 역할 | 모델 | 호출 방식 |
|---------|------|------|---------|
| planner | 전략 기획서 작성 (8개 섹션) | opus-4-6 | /forge |
| critic | 기획서 비평 및 APPROVED/REJECTED 판정 | opus-4-6 | /forge |
| slicer | 기획서를 Feature 단위로 분해 | sonnet-4-6 | /slice |
| test-writer | 실패하는 단위 테스트 작성 (Red) | sonnet-4-6 | /build |
| implementer | 테스트 통과 최소 구현 (Green) | opus-4-6 | /build |
| refactorer | 동작 유지하며 코드 품질 개선 | sonnet-4-6 | /build |
| reviewer | 보안·정확성·컨벤션 코드 리뷰 | sonnet-4-6 | /build |
| qa | 커버리지·E2E·API·보안 검증 | sonnet-4-6 | /build |
| documentor | Feature 완료 후 docs/project/ 갱신 | sonnet-4-6 | /build, /forge |

## 저장소 구조

```
agentforge/
├── .claude-plugin/
│   ├── marketplace.json
│   └── plugin.json
├── agents/
│   ├── planner.md
│   ├── critic.md
│   ├── slicer.md
│   ├── test-writer.md
│   ├── implementer.md
│   ├── refactorer.md
│   ├── reviewer.md
│   ├── qa.md
│   └── documentor.md
└── skills/
    ├── forge/
    │   └── SKILL.md
    ├── slice/
    │   └── SKILL.md
    └── build/
        └── SKILL.md
```
