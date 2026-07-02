# Recipe 04 — RAG (벡터 검색)

**조합**: 벡터 청크(표현) + 청크 메타데이터의 경로(연결) + 벡터 유사도 검색(접근)
**잘 답하는 질문**: "이거랑 비슷한 내용 어디 있었지?" — 정확한 키워드를 모를 때
**못 답하는 질문**: 관계 조인 ("콘솔을 가르친 **기관**과 그때 **실습**") — 이건 01·02가 낫다
**필요 도구**: 임베딩 모델(로컬: sentence-transformers / API: OpenAI·Voyage) + chromadb 등

---

## 언제 이 레시피를 쓰나 (에이전트 판단 기준)

`questions`가 대부분 "~비슷한", "~관련된", "~느낌의" 형태이고
개념-개체-관계 구조가 잘 안 잡히는 자료(잡다한 스크랩, 웹클립)일 때.
축이 명확한 자료(강의안, 제안서)라면 01·02를 먼저 권하라.
**RAG는 만능이 아니라 퍼지 검색 특화 조합이다.**

## 에이전트 실행 절차

### 1. 청킹 전략

- 문서 통째 임베딩 금지. 의미 단위(슬라이드/섹션/모듈)로 분할
- 청크 크기 500~1500자, 오버랩 10~15%
- **각 청크 메타데이터에 반드시 기록**: `source_path`, 가능하면 온톨로지 축 값
  (메타데이터가 연결층이다 — 이거 없으면 검색돼도 원문에 못 간다)

### 2. 인덱스 구축

```python
import chromadb
client = chromadb.PersistentClient(path="./wiki_vectors")
col = client.create_collection("wiki")
col.add(
    ids=[...],
    documents=[chunk_texts],
    metadatas=[{"source_path": p, "org": o, "topic": t} for ...],
)
```

로컬 무료 구성: chromadb 기본 임베딩(all-MiniLM) 또는
한국어 성능이 필요하면 `BM-K/KoSimCSE` 계열 sentence-transformers.

### 3. 검색 → 원문 도달

```python
res = col.query(query_texts=["콘솔 관련 실습"], n_results=5)
# res의 metadatas[i]["source_path"] → 원문 읽기
```

메타데이터 필터로 축 검색도 일부 가능: `where={"org": "삼성전자"}` —
단 이건 RAG의 본령이 아니라 보완이다.

### 4. 검증

`questions` 3개 실행. 퍼지 질문에서 top-5 안에 정답 문서가 들면 성공.
조인형 질문이 계속 실패하면 → 02(SQLite)와 하이브리드로 가라:
**축 필터는 SQL, 본문 유사도는 벡터.** 실무에서 가장 강한 조합이다.
