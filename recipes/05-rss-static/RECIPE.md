# Recipe 05 — RSS / 정적 사이트 파생

**조합**: 피드 항목(표현) + URL(연결) + HTTP fetch(접근)
**잘 답하는 질문**: "새로 뭐 올라왔나" — 시간축 구독. 그리고 **외부 공유**
**참고 사례**: https://cdsassj00.github.io/2026-proposals-rss/ — 문서정보를 RSS로 파생한 지식검색
**필요 도구**: GitHub Pages (무료) + 정적 생성 스크립트

---

## 이 레시피의 포지션

01~04는 "나의 검색"이다. 05는 **지식을 남(사람+타인의 에이전트)이 구독하게** 만든다.
위키의 출력 채널이지 검색 엔진이 아니다. 그래서 01·02 위에 얹는 파생 레시피다.

## 에이전트 실행 절차

### 1. 소스 결정

- Recipe 01의 `wiki/*.md` frontmatter, 또는
- Recipe 02의 `wiki.db` documents 테이블

### 2. 정적 사이트 생성

`build.py` (또는 Node 스크립트) 하나로:

1. 소스에서 문서 목록 로드 (제목·요약·축·날짜·원문 링크)
2. `index.html` 생성 — 축(org/topic)별 필터 + 클라이언트 검색(단일 HTML, 의존성 최소)
3. `feed.xml` 생성 — RSS 2.0:

```xml
<item>
  <title>{제목}</title>
  <description>{요약}</description>
  <category>{topic}</category>
  <link>{원문 URL 또는 상세 페이지}</link>
  <pubDate>{date}</pubDate>
</item>
```

### 3. 배포

```bash
# GitHub Pages: docs/ 폴더 방식이 가장 단순
git add docs/ && git commit -m "build wiki site" && git push
# Settings → Pages → main /docs
```

갱신 자동화가 필요하면 GitHub Actions로 push 시 build.py 재실행.

### 4. 에이전트가 읽게 만들기

이 사이트 자체가 다른 에이전트의 지식 소스가 된다:

- 에이전트에게 `feed.xml` URL만 주면 fetch → 최신 항목 파악
- 각 item의 link를 따라가면 원문/상세 — **URL이 연결층**
- 즉, MCP 없이도 "HTTP만 되면 검색되는 위키"가 된다.
  망분리 환경이 아니라면 가장 배포 비용이 낮은 접근층이다.

### 5. 검증

- RSS 리더 또는 `curl feed.xml`로 피드 유효성 확인
- 새 세션 에이전트에게 URL만 주고 "최근 올라온 것 요약해줘" — 성공하면 완료
