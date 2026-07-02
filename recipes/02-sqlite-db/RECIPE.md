# Recipe 02 — SQLite DB화

**조합**: 스키마 레코드(표현) + 외래키·경로 필드(연결) + SQL 쿼리(접근)
**잘 답하는 질문**: 정확한 조인·필터·집계 — "2025년 공공기관 강의 전부", "주제×기관 교차"
**필요 도구**: sqlite3 (Python 표준 내장, 별도 설치 불필요)

---

## 에이전트 실행 절차

### 1. 스키마 생성 — 온톨로지를 테이블로 변환

`my-ontology.yaml`의 entities는 테이블로, relations는 외래키/조인 테이블로 옮긴다. 강의안 예시:

```sql
CREATE TABLE documents (
  id INTEGER PRIMARY KEY,
  title TEXT NOT NULL,
  org TEXT,                 -- 축: 강의기관
  topic TEXT,               -- 축: 주제
  date TEXT,
  summary TEXT NOT NULL,    -- 표현층: 요약
  source_path TEXT NOT NULL -- 연결층: 원문 절대경로. NOT NULL — 포인터 없는 레코드 금지
);

CREATE TABLE concepts (     -- 개념 (theory/practice의 개별 항목)
  id INTEGER PRIMARY KEY,
  name TEXT UNIQUE,
  kind TEXT CHECK(kind IN ('theory','practice','module'))
);

CREATE TABLE doc_concepts ( -- 관계: 문서 ↔ 개념
  doc_id INTEGER REFERENCES documents(id),
  concept_id INTEGER REFERENCES concepts(id),
  PRIMARY KEY (doc_id, concept_id)
);

-- 전문검색 (FTS5): 요약 텍스트 퍼지 검색용
CREATE VIRTUAL TABLE doc_fts USING fts5(title, summary, content='documents', content_rowid='id');
```

### 2. 적재 스크립트 작성

`ingest.py` 하나로 완결한다:

1. `input_dir` 스캔 → 텍스트 추출
2. LLM(에이전트 자신)이 문서별로 요약 + 축 값 + 개념어 추출
3. INSERT (documents, concepts, doc_concepts, doc_fts)
4. 재실행 시 `source_path` 기준 upsert (중복 적재 방지)

### 3. 검증 쿼리 — 검색 계약 그대로

"콘솔을 가르친 기관과 실습 내용"이 조인 한 방에 나와야 한다:

```sql
SELECT d.org, d.title, d.date, d.summary, d.source_path
FROM documents d
JOIN doc_concepts dc ON dc.doc_id = d.id
JOIN concepts c ON c.id = dc.concept_id
WHERE c.name = '콘솔';
```

`my-ontology.yaml`의 `questions` 3개를 각각 SQL로 번역해 실행하고,
결과의 `source_path`로 원문을 열어 읽을 수 있는지까지 확인한다.

---

## RDBMS와 뭐가 다른가 (사용자 설명용)

구조는 같다. 다른 건 **조인을 사람이 짜지 않는다**는 것.
에이전트가 자연어 질문을 받아 SQL로 번역하고, 결과의 summary로 1차 판단,
source_path로 원문 정독. 복잡한 키 스키마가 의미 기반 검색으로 대체된다.

## 다음 단계

이 DB를 어떤 세션·어떤 도구에서든 쓰고 싶으면 → `recipes/03-mcp-server`
