---
name: create-agent
description: 새로운 Claude Code subagent를 인터뷰를 통해 생성하는 skill. /create-agent 명령으로 호출되며, 사용자와 대화하면서 역할, 도구, 모델을 결정하고 .claude/agents/ 에 파일을 생성한다.
version: 0.2.0
---

# Create Agent

사용자와 인터뷰를 진행하여 새로운 Claude Code subagent 파일을 생성한다.

## 인터뷰 절차

다음 질문을 순서대로 진행한다. 각 질문은 하나씩, 사용자의 답변을 기다린 후 다음으로 넘어간다.

### Q1. 역할
"어떤 역할을 하는 agent를 만들까요? (예: 코드 리뷰, 기획 문서 작성, 테스트 생성 등)"

### Q2. 트리거 조건
"어떤 상황에서 이 agent가 자동으로 호출되길 원하나요? (예: '코드 리뷰해줘'라고 할 때)"

### Q3. 도구
사용자의 역할 설명을 바탕으로 필요한 도구를 제안하고 확인한다.

| 작업 유형 | 추천 tools |
|-----------|------------|
| 읽기/분석만 | Read, Glob, Grep |
| 파일 생성 | Read, Write, Glob |
| 코드 실행/테스트 | Read, Bash |
| 전체 | Read, Write, Bash, Glob, Grep |

"이 agent에 필요한 도구를 추천드립니다: {추천 tools}. 맞나요?"

### Q4. 모델
"작업의 복잡도에 따라 모델을 선택합니다:
- **opus** (claude-opus-4-6): 복잡한 판단, 기획, 비평
- **sonnet** (claude-sonnet-4-6): 일반 개발, 구현
- **haiku** (claude-haiku-4-5): 빠른 검증, 단순 리뷰

어떤 모델이 적합할까요?"

### Q5. 이름 확인
수집한 정보를 바탕으로 agent 이름을 제안한다. (소문자 + 하이픈, 예: `code-reviewer`, `planner`)

"agent 이름을 `{name}`으로 할까요?"

---

## 인터뷰 완료 후

### 중복 확인
`Glob(".claude/agents/*.md")`로 같은 이름의 파일이 있는지 확인한다.
존재하면 덮어쓸지 사용자에게 확인한다.

### 파일 생성
`.claude/agents/{name}.md`를 다음 포맷으로 생성한다:

```markdown
---
name: {name}
description: >
  {트리거 조건. "~할 때 사용", "~가 필요할 때 호출" 형태.}
model: {model}
tools: {tools}
---

## Role

{역할을 한 문장으로.}

## 절대 하지 않는 것

- {역할 외 작업 금지}
- {다른 agent 역할 침범 금지}

## 입력 형식

{이 agent가 받을 입력의 형태.}

## 출력 형식

{결과물을 어디에, 어떤 형식으로 남길지.}

## 완료 조건 (DoD)

다음을 모두 만족할 때만 완료:
- [ ] {사용자 요청에서 도출된 조건 1}
- [ ] {사용자 요청에서 도출된 조건 2}

## 실패 시 행동

- 조건 미충족 시 최대 2회 재시도
- 2회 초과 시 FAILED 상태로 종료
- 실패 이유를 `.claude-progress.txt`에 기록

## 에스컬레이션 조건

다음 상황에서 즉시 중단하고 human gate로 전달:
- 요구사항이 상호 모순될 때
- 필요한 파일/정보가 없을 때
- 3회 재시도 후에도 DoD 미충족
```

### 완료 보고
```
✅ Agent 생성 완료
- 파일: .claude/agents/{name}.md
- 역할: {한 줄 요약}
- 모델: {model}
- Tools: {tools}
- DoD 항목: {n}개
```
