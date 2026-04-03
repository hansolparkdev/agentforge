# agentforge

Claude Code 플러그인. 큐레이션된 agent와 skill을 plugin 설치 명령으로 프로젝트에 주입할 수 있다.

## 사용 방법

```
/plugin add hansolparkdev/agentforge
```

## 저장소 구조

```
agentforge/
├── .claude-plugin/
│   └── plugin.json             # Plugin manifest
├── agents/
│   ├── planner.md              # 전략 기획자 agent
│   ├── critic.md               # 비평가 agent
│   ├── test-writer.md          # 테스트 작성 agent
│   ├── implementer.md          # 구현 agent
│   ├── reviewer.md             # 코드 리뷰 agent
│   └── qa.md                   # QA agent
└── skills/
    ├── forge/SKILL.md          # /forge 스킬
    ├── build/SKILL.md          # /build 스킬
    └── doc/SKILL.md            # /doc 스킬
```

## Behavior Rules

- agent frontmatter의 `name` 필드는 파일명(확장자 제외)과 동일해야 함
- agent는 `agents/` 디렉토리에, skill은 `skills/{name}/SKILL.md` 경로에 위치
- 버전은 `plugin.json`에서만 관리
