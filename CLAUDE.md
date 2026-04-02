# agentforge

Claude Code의 plugin marketplace. 큐레이션된 agent와 skill을 plugin 설치 명령으로 프로젝트에 주입할 수 있다.

## 사용 방법

```
# 1. agentforge를 marketplace로 등록
/plugin marketplace add hansolparkdev/agentforge

# 2. 사용 가능한 plugin 목록 확인
/plugin list@agentforge

# 3. plugin 설치
/plugin install agentforge@agentforge
```

## 저장소 구조

저장소 루트가 곧 plugin 루트다.

```
agentforge/
├── .claude-plugin/
│   ├── marketplace.json        # Marketplace 카탈로그
│   └── plugin.json             # Plugin manifest
├── agents/
│   ├── planner.md              # 전략 기획자 agent
│   └── critic.md               # 비평가 agent
└── skills/
    └── forge/
        └── SKILL.md            # /forge 스킬
```

## Behavior Rules

- `marketplace.json`의 `name`과 `plugin.json`의 `name`은 일치해야 함
- agent frontmatter의 `name` 필드는 파일명(확장자 제외)과 동일해야 함
- plugin 버전은 `plugin.json`과 `marketplace.json` 모두 동일하게 유지
- agent는 `agents/` 디렉토리에, skill은 `skills/{name}/SKILL.md` 경로에 위치
