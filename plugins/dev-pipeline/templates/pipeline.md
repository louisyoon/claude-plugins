# 파이프라인 레포 규칙

> dev-pipeline 플러그인 에이전트가 작업 전에 읽는다. 레포마다 다른 것만 적는다.
> 공통 섹션은 모든 에이전트가, `## {에이전트명}` 섹션은 해당 에이전트가 추가로 따른다.

## 파이프라인 설정
- tester: default            # default | on-request
- designer: on               # on | off  (on이면 UI 비중 큰 작업에서 2b 단계 실행)
- 정본 지침 문서: AGENTS.md   # 충돌 시 우선하는 문서

## 공통
- 스택: (예: React 18 + Vite, FSD, TanStack Query, Zod)
- 작업 순서: (예: entities → features → widgets → pages → MSW → stories → changeset)
- 관련 레포: (예: `../biz-api-mathpie` 백엔드 — 응답 VO 경로 `src/main/java/**/vo`)
- 언어: 한국어

## pm
- 화면/도메인 목록, 기획 문서 위치

## architect
- 레이어별 산출물 체크리스트, 스키마/DTO 위치, 캐시 키 규칙

## designer
- 디자인 토큰 파일, 공용 컴포넌트 목록 파일, variant 방식

## coder
- 반려 기준(의존 방향, import 규칙, 네이밍, 스타일)
- 자체 검증 명령 + 기준 오류 수 (예: `pnpm typecheck:teacher` 기존 35개 → 신규 0)

## tester
- 테스트 러너·위치·네이밍, 게이트 명령

## reviewer
- red flag 목록 (리뷰 반려 사유)

## qa
- 로컬 실행 방법(포트, 목업 모드, 테스트 계정), 브라우저 확인 방법

## docs
- 히스토리 문서 경로·파일명 규칙, 기능 문서 위치
