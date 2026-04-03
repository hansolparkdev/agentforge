---
name: doc
description: 코드베이스를 읽어 docs/project/ 문서를 생성/갱신. "/doc" 또는 "/doc {slug}"로 호출.
---

# Doc

코드베이스를 직접 읽어서 `docs/project/` 문서를 생성하거나 갱신한다.
기존 문서가 없어도 되고, 언제 실행해도 현재 코드 상태를 반영한다.

## 호출 방식

```
/doc
/doc tic-tac-toe
/doc tic-tac-toe F3
```

## 실행 절차

### 준비

slug 추론:
- `/doc {slug}` → 해당 slug 사용
- `/doc` → `docs/plans/` 아래 plan.md가 하나면 자동 선택, 여러 개면 목록 보여주고 선택 요청

대상 Feature:
- Feature 번호 지정 시: 해당 Feature만 반영
- 미지정 시: 전체 코드베이스 기준으로 갱신

### 코드베이스 탐색

다음을 읽어 현재 상태를 파악한다:

1. `CLAUDE.md` — 기술 스택, 컨벤션
2. `docs/plans/{slug}/plan.md` (있으면) — 원래 의도 파악
3. `docs/plans/{slug}/features.md` (있으면) — Feature 목록 및 완료 현황
4. 실제 소스 파일 전체 — Glob으로 탐색 후 읽기
   - 구현 파일 (src/, app/, lib/ 등)
   - 테스트 파일 (\_\_tests\_\_/, tests/, spec/ 등)
   - 설정 파일 (vite.config, tsconfig, package.json 등)
5. `docs/plans/{slug}/ambiguous.md` (있으면) — 선판단 기록

### 문서 생성/갱신

코드베이스를 기반으로 아래 파일을 **현재 상태 그대로** 작성한다.
파일이 이미 있으면 전체 교체하지 않고 변경된 부분만 갱신한다.

#### `docs/project/index.md`

```markdown
# 프로젝트 인덱스

> 마지막 업데이트: {날짜}
> 커버리지: {측정값 또는 N/A}

## 기술 스택
{한 줄 요약}

## 아키텍처
{레이어/구조 2-3줄 요약}

핵심 파일:
- {파일 경로} — {역할 한 줄}
...

## API
{엔드포인트 목록 — 없으면 "해당 없음"}

## DB 스키마
{테이블/컬렉션 목록 — 없으면 "해당 없음"}

## 디자인 시스템 (UI 포함 시)
{CSS 방법론, 색상 변수, 브레이크포인트}

## 보안
{인증/인가 방식, 주요 보안 조치}

## 완료된 Feature
{features.md 기준 ✅ Feature 목록 — 없으면 코드 기반으로 추론}

## 미확인 선판단
{ambiguous.md 미확인 항목 — 없으면 "없음"}
```

#### 필요 시 추가 생성

코드베이스에 해당 내용이 있을 때만 생성한다:

| 조건 | 생성 파일 |
|------|---------|
| API 라우터/핸들러 존재 | `docs/project/api.md` |
| DB 모델/스키마 존재 | `docs/project/schema.md` |
| 컴포넌트/모듈 다수 | `docs/project/components.md` |
| ambiguous.md 선판단 항목 | `docs/project/decisions.md` |

### 완료 보고

```
문서 갱신 완료 ✓

기준: 코드베이스 직접 탐색
탐색 파일: {n}개

생성/갱신된 문서:
- docs/project/index.md
- {그 외 생성된 파일들}

커버리지: {측정값 또는 미측정}
미확인 선판단: {n}개 (docs/project/decisions.md)
```
