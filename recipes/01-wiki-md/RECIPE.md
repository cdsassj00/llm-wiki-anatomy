# Recipe 01 — Wiki md + 심링크/정션 + Obsidian 그래프

**조합**: 요약 md(표현) + 심링크·정션·경로(연결) + 에이전트 파일읽기(접근)
**잘 답하는 질문**: 개념→개체→관계 탐색, 그래프로 지식 지형 보기
**필요 도구**: 파일시스템만. Obsidian은 그래프뷰용 (선택)

---

## 에이전트 실행 절차

### 1. 입력 스캔

`my-ontology.yaml`의 `input_dir`를 재귀 스캔하고 문서 목록을 만든다.
pdf/pptx/docx는 텍스트를 추출한다 (Python: pypdf, python-pptx, python-docx 등).

### 2. 문서별 요약 md 생성

각 문서마다 `templates/summary-template.md` 형식으로 요약 md를 만든다.

- frontmatter에 **온톨로지의 개념·개체 값을 반드시 채운다** (추출 불가면 `null`)
- `source_path`에 원문 절대경로를 기록한다 — **생략 시 이 레시피는 실패다**
- 본문 요약은 3~7 불릿. 원문 대체가 아니라 검색 미끼임을 기억할 것
- 파일명: `wiki/{org}/{date}-{제목-슬러그}.md`

### 3. 원문 포인터 연결 (연결층 — 이 레시피의 핵심)

위키는 요약만 저장한다. 에이전트가 검색된 요약에서 **원문까지 이동해 읽으려면**
경로가 살아있어야 한다. OS별로:

```bash
# Windows — 정션 (관리자 권한 불필요, 폴더 단위)
mklink /J "wiki\_sources" "D:\lectures"

# Windows — 심링크 (개발자모드 또는 관리자, 파일 단위 가능)
mklink "wiki\_sources\file.pptx" "D:\lectures\file.pptx"

# macOS / Linux
ln -s ~/lectures wiki/_sources
```

권한 문제로 링크 생성이 안 되면 `link_mode: path`로 두고
frontmatter의 `source_path`만으로 연결한다 (에이전트는 이것만으로도 읽는다).

### 4. 관계 링크 생성

요약 md들 사이에 `[[위키링크]]`를 건다:

- 같은 `topic`끼리 → 주제 MOC(Map of Content) 문서 생성, 링크
- 같은 `org`끼리 → 기관별 목록 문서 생성, 링크
- `theory`/`practice`의 개념어 → 개념 노트 생성(`wiki/concepts/콘솔.md`), 역링크

이 링크 구조가 Obsidian 그래프뷰에서 지식 지형으로 보인다.

### 5. (선택) Obsidian 연결

`wiki/` 폴더를 Obsidian vault로 열도록 안내한다.
Obsidian CLI가 있으면 vault 등록까지 자동화한다.

### 6. 검증

`my-ontology.yaml`의 `questions` 3개를 실제로 수행한다:

```
grep/글롭으로 frontmatter 검색 → 관련 요약 md 발견
→ source_path의 원문 열어 읽기 → 질문에 답변
```

3개 모두 (a) 요약 검색 성공 + (b) 원문 도달 성공이면 완료.

---

## 이후 사용법 (사용자 안내용)

어떤 세션에서든:

```
wiki/ 폴더에서 "콘솔" 관련 강의안 찾아서, 원문까지 읽고
2시간짜리 새 모듈로 재조합해줘
```

에이전트가 frontmatter 검색 → 요약 파악 → source_path로 원문 읽기 → 조립.
이것이 `/wiki` 스킬의 원형이다.
