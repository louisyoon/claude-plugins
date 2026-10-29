---
name: pipeline
description: 기능 개발·버그 수정·설계 변경 요청을 pm → architect → (designer) → coder → (tester) → reviewer → qa → docs 서브에이전트 파이프라인으로 진행할 때 사용한다. "MP-007 진행해줘", "이 기획안 구현해줘", "PRD부터 해줘", "파이프라인 돌려" 같은 요청, 또는 _workspace/ 산출물을 이어서 진행할 때.
---

# dev-pipeline 오케스트레이션

메인 세션(사용자와 대화하는 에이전트)이 오케스트레이터다. 별도 orchestrator 서브에이전트 없이 Agent 도구로 아래 서브에이전트를 직접 호출한다. 에이전트 이름은 플러그인 접두사를 붙인다: `dev-pipeline:pm`, `dev-pipeline:architect`, `dev-pipeline:designer`, `dev-pipeline:coder`, `dev-pipeline:tester`, `dev-pipeline:reviewer`, `dev-pipeline:qa`, `dev-pipeline:docs`.

## 0. 시작 전
1. 레포 루트의 `.claude/pipeline.md`를 읽는다. 없으면 `CLAUDE.md`·`AGENTS.md`로 대신하고, 사용자에게 `templates/pipeline.md` 기반으로 만들지 한 번 제안한다.
2. pipeline.md의 `## 파이프라인 설정`(tester 기본값, designer 사용 조건, 추가 게이트)을 확인한다.
3. `_workspace/00_input/request.md`에 사용자 요청 원문을 저장한다(디렉터리 없으면 생성).

## 1. 순서

```
1. pm        → _workspace/01_pm_prd.md
2. architect → _workspace/02_architect_design.md
2b. designer → _workspace/02b_designer_spec.md   (pipeline.md에서 designer 사용 + UI 비중 큰 작업일 때만)
   [사람 승인 게이트 — 설계 요약을 사용자에게 제시하고 승인받는다]
3. coder     → 소스 + _workspace/03_coder_summary.md
4. tester    → _workspace/04_tester_report.md (TestsPassed 게이트)
   - pipeline.md `tester: on-request`면 사용자가 테스트를 명시 요청할 때만 실행
   - `tester: default`면 coder 다음 실행
5. reviewer  → _workspace/05_reviewer_report.md (APPROVED 게이트)
6. qa        → _workspace/06_qa_report.md (BLOCKER 없음 게이트)
7. docs      → _workspace/07_docs_summary.md (히스토리 문서는 사용자가 원할 때만)
```

## 2. 단계 진행 허락 (필수)
- PM·Architect 산출 후: 요약 + 미확정/결정 필요 항목을 사용자에게 제시하고 답을 받는다(AskUserQuestion 사용 가능). 답은 `_workspace/00_input/decisions.md`에 기록.
- **coder 이후 단계(tester·reviewer·qa·docs)는 사용자 허락 없이 자동 실행하지 않는다.** 각 단계 결과를 보고하고 "리뷰 진행할까요?"처럼 다음 단계를 제안만 한다. 사용자가 "바로 진행", "끝까지 해"라고 명시한 경우에만 이어서 실행.
- 게이트 실패로 coder에게 되돌리는 수정 루프도 사용자에게 먼저 보고한다.

## 3. 원칙
- 실질 작업(PRD, 설계, 구현, 테스트, 리뷰, QA, 문서화)은 직접 하지 않고 서브에이전트에게 위임한다. 예외: 사소한 질문 답변, 파일 존재·사용처 조회, 로그 한두 줄 원인 설명, 문구·조건 한두 줄 수정.
- 각 서브에이전트 호출 직전 무엇을 누구에게 위임하는지 사용자에게 한 줄로 알린다.
- 각 단계 호출 전 이전 단계 산출물이 실제로 있는지 확인한다.
- `_workspace/`에 이전 진행 상태가 있으면 마지막 완료 단계 다음부터 재개한다. 완료 단계를 다시 돌리지 않는다.
- **게이트 실패**: tester 실패 → coder, reviewer CHANGES_REQUESTED → coder, qa BLOCKER → 원인 주체(coder/architect). 같은 게이트 3회 초과 시 멈추고 사용자에게 보고.
- **서브에이전트 종료는 리더 책임**: SendMessage로 완료/차단 보고를 받으면 즉시 TaskStop으로 kill한다. 후속 작업은 새로 스폰하거나 SendMessage로 재개한다.
- 사용자가 실행 중인 tester를 중단시키면 끝까지 기다리지 말고 즉시 TaskStop.
- 커밋·푸시는 사용자가 명시 요청할 때만 한다.

## 4. _workspace 절대 규칙
- `_workspace/`를 절대 아카이브하지 않는다 — 하위 폴더로 옮기기, 이름 바꾸기, 백업 복사, 삭제 모두 금지. 새 작업을 시작해도 마찬가지다.
- 새 작업은 같은 파일명이면 그냥 덮어쓴다(`00_input/request.md`, `01_pm_prd.md` 등). 새 작업이 쓰지 않는 기존 파일은 그대로 둔다. 이 처리 방식을 사용자에게 묻지 않는다.
- `_workspace/`는 `.gitignore` 대상이어야 한다. 없으면 사용자에게 추가를 제안한다.
