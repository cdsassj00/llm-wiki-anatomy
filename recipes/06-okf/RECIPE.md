# Recipe 06 — OKF (Open Knowledge Format) 번들

**조합**: 요약 md + YAML frontmatter(표현) + 마크다운 링크(연결) + 파일읽기/카탈로그 인제스트(접근)
**잘 답하는 질문**: 01과 동일하되, **내 위키를 남의 에이전트·타 조직·구글 Knowledge Catalog가
번역 없이 읽게** 하고 싶을 때
**필요 도구**: 파일시스템만. 스펙 전체가 한 페이지다.

---

## OKF가 뭔가 (사용자 설명용)

2026년 6월 구글 클라우드가 발표한 v0.1 개방형 스펙(Apache 2.0).
카파시(Karpathy)의 LLM-wiki gist에서 시작해 AGENTS.md, CLAUDE.md,
옵시디언 vault로 각자 만들던 **LLM-wiki 패턴을 표준으로 공식화**한 것이다.

즉, 이 레포의 recipe 01이 하던 것과 본질이 같다. 차이는 딱 하나 —
**남들과 호환되는 규약을 따른다**는 것. 내 위키를 다른 팀의 에이전트가,
다른 도구가, 구글의 Knowledge Catalog가 번역 없이 읽는다.

스펙의 전부:
- 번들 = 마크다운 파일들의 디렉토리. **파일 경로 = 개념의 정체성**
- 각 파일 = 개념 하나. YAML frontmatter의 **필수 필드는 `type` 하나뿐**
  (선택: title, description, resource, tags, timestamp)
- 파일 간 일반 마크다운 링크가 지식 그래프의 엣지
- 예약 파일명 2개: `index.md`(계층 안내), `log.md`(변경 이력)

주의: "최소한으로 opinionated"가 설계 철학이라, **type에 뭘 쓸지·본문을
어떻게 쓸지는 전부 생산자 몫**이다. 그래서 온톨로지 설계(우리 레포의
ontology/)가 OKF 위에서도 여전히 필요하다 — OKF는 그릇 표준이지
내용 표준이 아니다.

## 에이전트 실행 절차

### 1. 온톨로지 → OKF 매핑

`my-ontology.yaml`의 구조를 디렉토리와 type으로 옮긴다. 강의안(트리플) 예:

```
lectures/                      # 번들 루트
├── index.md                   # 번들 안내 (예약)
├── log.md                     # 변경 이력 (예약)
├── documents/                 # 개체: 강의안
│   ├── index.md
│   └── 2026-06-11-kakao-claude-code.md
├── orgs/                      # 개체: 기관
│   └── kakao-ent.md
└── concepts/                  # 개념: 이론·실습
    ├── console.md
    └── mcp-hands-on.md
```

패턴이 달라도 원리는 같다: 계층(taxonomy)이면 디렉토리 트리가 곧 분류,
패싯이면 tags 필드가 축, 이벤트면 log.md 스타일의 연대기.

### 2. 개념 문서 작성

```markdown
---
type: lecture
title: 카카오엔터 임원 Claude Code 과정
description: Claude/Cowork/Claude Code 도구 중심 임원 교육
resource: "file:///D:/lectures/2026/kakao-ent/claude-code.pptx"
tags: [생성형AI, 바이브코딩]
timestamp: 2026-06-11T09:00:00Z
---
# 개요
[콘솔](/concepts/console.md) 개념을 [카카오엔터](/orgs/kakao-ent.md)에서 다뤘다.
실습: [MCP 실습](/concepts/mcp-hands-on.md)
```

- `resource`가 우리의 source_path다 — **원문 포인터 필수 원칙은 여기서도 유지**
- 관계는 본문의 마크다운 링크로 표현된다 (트리플의 술어는 링크 주변 문장으로)

### 3. index.md / log.md

- 각 디렉토리의 `index.md`: 하위 개념 목록 + 한 줄 설명 (에이전트의 점진 탐색용)
- 루트 `log.md`: 문서 추가·수정의 연대기 (에이전트가 "최근 뭐 바뀌었나" 파악)

### 4. 검증

- `questions` 3개를 새 세션에서 실행: 링크 그래프를 타고 답 + resource로 원문 도달
- 상호운용 확인: 구글이 공개한 정적 HTML 비주얼라이저나 링크 파서
  (frontmatter 파싱 + `](/*.md)` 링크 수집 — 20줄짜리 파이썬이면 된다)로
  번들이 규격대로 읽히는지 확인

## 01과의 관계 (선택 기준)

| | 01 Wiki md | 06 OKF |
|---|---|---|
| 나만 쓴다, 옵시디언 그래프뷰 원함 | ✅ | |
| 팀·타 조직·타 도구와 공유 | | ✅ |
| frontmatter 규약 | 자유 | type 필수 + 표준 필드 |
| 연결층 | 심링크·정션·경로 | resource 필드 + md 링크 |

01로 만들었다가 06으로 변환하는 건 스크립트 한 번이다 (frontmatter
필드명 매핑 + 위키링크→상대경로 링크). 반대도 마찬가지.
