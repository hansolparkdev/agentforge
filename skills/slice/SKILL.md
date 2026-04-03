---
name: slice
description: 기획서를 Feature 단위로 분해. "/slice {slug}"로 호출.
---

# Slice

완성된 기획서를 읽고 Slicer 에이전트를 실행하여 `features.md`를 생성한다.

## 실행 절차

### 1단계 — 기획서 존재 확인

`docs/plans/{slug}/plan.md` 파일이 존재하는지 확인한다. (읽지 않는다)
없으면: "기획서를 먼저 작성해주세요. `/forge {주제}`를 실행하세요."

### 2단계 — Slicer 호출

Slicer 에이전트를 호출한다.
전달: slug, `docs/plans/{slug}/plan.md`

### 3단계 — 완료 보고

```
슬라이싱 완료 ✓

features.md: docs/plans/{slug}/features.md

features.md를 확인하고 준비되면:
/build {slug} F1
```
