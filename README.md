# louis-plugins

개인 Claude Code 플러그인 마켓플레이스.

## dev-pipeline
`pm → architect → (designer) → coder → (tester) → reviewer → qa → docs` 서브에이전트 파이프라인.
에이전트는 공통 규칙만 담고, 레포별 규칙은 각 레포의 `.claude/pipeline.md`에서 읽는다.

### 설치
```
/plugin marketplace add louisyoon/claude-plugins
/plugin install dev-pipeline@louis-plugins
```

레포에 커밋해 두고 열 때 자동 안내받으려면 `.claude/settings.json`:
```json
{
  "extraKnownMarketplaces": {
    "louis-plugins": { "source": { "source": "github", "repo": "louisyoon/claude-plugins" } }
  },
  "enabledPlugins": { "dev-pipeline@louis-plugins": true }
}
```

로컬 개발(설치 없이 테스트):
```
claude --plugin-dir ~/Desktop/claude-plugins/plugins/dev-pipeline
```

### 레포 적용
1. `plugins/dev-pipeline/templates/pipeline.md`를 레포의 `.claude/pipeline.md`로 복사해 채운다.
2. 레포의 `.claude/agents/`를 삭제한다(플러그인 에이전트와 중복 방지).
3. `CLAUDE.md`의 "작업 오케스트레이션" 절은 `/dev-pipeline:pipeline` skill로 대체되므로 지워도 된다.

### 사용
- `/dev-pipeline:pipeline MP-007 진행해줘` 또는 "이 기획안 파이프라인으로 진행해줘"
- 에이전트 직접 호출: `dev-pipeline:coder` 등 (`/agents`에서 확인)

### 갱신
레포 push 후 `/plugin marketplace update louis-plugins`.
