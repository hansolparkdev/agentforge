# agentforge

Claude Code의 plugin marketplace. 큐레이션된 subagent 파일을 plugin 설치 명령으로 프로젝트에 주입할 수 있다.

## 사용 방법

```
# 1. agentforge를 marketplace로 등록
/plugin marketplace add hansolparkdev/agentforge

# 2. 사용 가능한 plugin 목록 확인
/plugin list@agentforge

# 3. 원하는 agent 설치
/plugin install agent-factory@agentforge
```

설치 후 프로젝트의 `.claude/agents/agent-factory.md`가 생성된다.

## 저장소 구조

```
agentforge/
├── .claude-plugin/
│   └── marketplace.json        # Marketplace 카탈로그
└── plugins/
    └── agent-factory/          # 개별 plugin
        ├── .claude-plugin/
        │   └── plugin.json     # Plugin manifest
        └── agents/
            └── agent-factory.md  # 설치될 agent 파일
```

## 새 plugin 추가 방법

1. `plugins/{name}/` 디렉토리 생성
2. `plugins/{name}/.claude-plugin/plugin.json` 작성
3. `plugins/{name}/agents/{name}.md` 작성 (올바른 frontmatter 포함)
4. `.claude-plugin/marketplace.json`의 `plugins` 배열에 항목 추가

## Behavior Rules

- `marketplace.json`의 `name`과 plugin 디렉토리명은 일치해야 함
- agent frontmatter의 `name` 필드는 파일명(확장자 제외)과 동일해야 함
- plugin 버전은 `plugin.json`과 `marketplace.json` 모두 동일하게 유지
